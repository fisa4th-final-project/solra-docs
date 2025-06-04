# ✨ 개요

이 문서는 Kubernetes 클러스터에 Harbor를 **Helm Chart**와 **Argo CD**를 통해 GitOps 방식으로 배포하고, 자체 서명 인증서를 기반으로 **HTTPS + Ingress 환경**을 구성하며 발생하는 보안 이슈 및 배포 오류를 해결한 전체 과정을 정리한 것입니다.

---

## 🌐 환경 정보

|항목|값|
|---|---|
|Kubernetes 버전|v1.28.15 (Ubuntu 24.04)|
|Helm Chart 버전|`ingress-nginx` v4.10.0 / `harbor` 최신|
|Helm 버전|v3.x|
|배포 방식|Argo CD 기반 GitOps|
|인증서 방식|OpenSSL로 생성한 self-signed TLS|

---

## 🧱 1단계: Ingress-NGINX Controller 배포 (Argo CD)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ingress-nginx-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: '<https://kubernetes.github.io/ingress-nginx>'
    chart: ingress-nginx
    targetRevision: 4.10.0
    helm:
      parameters:
        - name: controller.service.type
          value: NodePort
        - name: controller.service.nodePorts.http
          value: "30002"
        - name: controller.service.nodePorts.https
          value: "30443"
        - name: controller.allowSnippetAnnotations
          value: "true"
  destination:
    server: '<https://kubernetes.default.svc>'
    namespace: ingress-nginx
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
    syncOptions:
      - CreateNamespace=true

```

```bash
argocd app sync ingress-nginx-app --prune --force

```

---

## 🔐 2단계: Harbor용 인증서 생성 및 Secret 등록

```bash
openssl req -x509 -nodes -days 365 \\
  -newkey rsa:2048 \\
  -keyout harbor.key \\
  -out harbor.crt \\
  -subj "/CN=solra.harbor.fisa/O=harbor"

kubectl create namespace solra-harbor

kubectl create secret tls harbor-tls \\
  --cert=harbor.crt \\
  --key=harbor.key \\
  -n solra-harbor

```

---

## ⚙️ 3단계: Harbor values.yaml 설정 (핵심 설정)

```yaml
expose:
  type: ingress
  tls:
    enabled: true
    secretName: harbor-tls
  ingress:
    className: nginx
    hosts:
      core: solra.harbor.fisa
    annotations:
      nginx.ingress.kubernetes.io/ssl-redirect: "true"
      nginx.ingress.kubernetes.io/hsts: "false"
      nginx.ingress.kubernetes.io/hsts-include-subdomains: "false"
      nginx.ingress.kubernetes.io/hsts-preload: "false"
      nginx.ingress.kubernetes.io/configuration-snippet: |
        more_clear_headers "Strict-Transport-Security";
      nginx.ingress.kubernetes.io/server-snippet: |
        more_clear_headers "Strict-Transport-Security";

externalURL: <https://solra.harbor.fisa>
harborAdminPassword: Harbor12345

```

> ⚠️ 운영 환경에서는 harborAdminPassword를 Secret으로 분리할 것.

---

## 🚀 4단계: Harbor Argo CD Application 배포

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: solra-harbor-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: '<https://your.repo.url>'
    path: deploy/harbor
    helm:
      valueFiles:
        - values.yaml
  destination:
    server: '<https://kubernetes.default.svc>'
    namespace: solra-harbor
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
    syncOptions:
      - CreateNamespace=true

```

```bash
argocd app sync solra-harbor-app --prune --force

```

---

## 🧪 트러블슈팅 이슈 및 해결 내역

|문제|원인|해결 방법|
|---|---|---|
|`Strict-Transport-Security` 헤더 차단|Chrome의 HSTS 프리로드 도메인 사용 (`harbor.com`)|비-HSTS 도메인 (`solra.harbor.fisa`) 사용|
|HSTS 제거 안됨|단순 `hsts: false` 설정으로는 제거되지 않음|`configuration-snippet`, `server-snippet` 설정 추가|
|Helm 배포 실패|`allowSnippetAnnotations` 미설정|Ingress-NGINX 재배포 시 해당 옵션 활성화|
|여전히 접속 불가|Chrome 내부 HSTS 캐시|`chrome://net-internals/#hsts`에서 domain 삭제 후 브라우저 재시작|

---

## 🔐 Harbor 접속 정보

|항목|정보|
|---|---|
|접속 주소|`https://solra.harbor.fisa:30443`|
|기본 ID|`admin`|
|비밀번호|`Harbor12345` (또는 values.yaml 설정값)|

---

## 🔄 이후 추천 작업

- ✅ Harbor 프로젝트 생성 및 권한 설정
- ✅ Docker CLI로 이미지 push/pull 테스트
- ✅ GitHub Actions 또는 Jenkins와 연동한 CI/CD 구축
- ✅ GitOps 배포 도구와 Harbor Registry 연계
- ✅ Trivy, Clair, Notary 등 보안 컴포넌트 설정
- ✅ 인증서 → Let's Encrypt 또는 사설 CA로 교체

---

## ✅ 마무리 요약

이 문서는 Helm과 Argo CD를 이용하여 Kubernetes 위에 Harbor를 배포하고, HTTPS 환경 구성 및 HSTS 관련 문제를 포함한 전체 과정을 정리한 실무 가이드입니다. GitOps 기반 운영체계에서 Harbor를 안정적으로 도입하기 위한 설정 예시와 트러블슈팅 경험이 모두 담겨 있습니다.

> 🔒 운영 환경에서는 반드시 신뢰된 인증서(예: Let's Encrypt)를 사용하고, 관리자 계정 정보는 Secret으로 분리 관리하세요.

---