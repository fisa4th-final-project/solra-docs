## 📌 개요

본 문서는 Kubernetes 클러스터 내 **NGINX Ingress Controller**를 Helm Chart 기반으로 설치하고, **Argo CD를 통해 GitOps 방식으로 자동 배포 및 동기화**한 과정을 기술합니다.

---

## 1. 🎯 목표

- Ingress Controller를 Helm Chart로 배포
- Argo CD를 통해 지속적 배포 및 상태관리 수행
- 외부 접근을 위한 NodePort 방식으로 노출

---

## 2. 📁 디렉토리 구조 (Git 기준)

```
infra-gitops/
└── apps/
    └── ingress-nginx/
        ├── ingress-nginx-app.yaml   # Argo CD Application
        └── values.yaml              # Helm 설정 파일

```

---

## 3. 📦 Helm values.yaml 설정

`infra-gitops/apps/ingress-nginx/values.yaml`

```yaml
controller:
  service:
    type: NodePort
    nodePorts:
      http: 30002
      https: 30443

```

> NodePort 방식으로 외부 접속 가능하도록 구성

---

## 4. 🧾 Argo CD Application 정의

`infra-gitops/apps/ingress-nginx/ingress-nginx-app.yaml`

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ingress-nginx-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: <https://kubernetes.github.io/ingress-nginx>
    chart: ingress-nginx
    targetRevision: 4.10.0
    helm:
      valueFiles:
        - values.yaml
  destination:
    server: <https://kubernetes.default.svc>
    namespace: ingress-nginx
  syncPolicy:
    automated:
      selfHeal: true
      prune: true
    syncOptions:
      - CreateNamespace=true

```

---

## 5. ⚙️ 배포 절차 요약

|단계|명령어/행동|
|---|---|
|1. Git 커밋 및 Push|`values.yaml`, `ingress-nginx-app.yaml` 작성 후 Git 저장소에 push|
|2. Argo CD 등록|`kubectl apply -f ingress-nginx-app.yaml`|
|3. Argo CD Sync|`argocd app sync ingress-nginx-app --prune --force`|
|4. 상태 확인|`kubectl get svc -n ingress-nginx` `kubectl get pods -n ingress-nginx`|

---

## 6. 🧪 접속 테스트

### 1) 확인 명령어

```bash
kubectl get svc -n ingress-nginx
```

예상 출력:

```
NAME                           TYPE       PORT(S)
ingress-nginx-app-controller   NodePort   80:30002/TCP, 443:30443/TCP
```

### 2) 외부 접속

```bash
curl -I http://<NodeIP>:30002
```

또는 브라우저에서:

```
http://<NodeIP>:30002
```

> 결과: 404 Not Found (정상 - Ingress 리소스 없음을 의미)

---

## 7. 🛠 문제 해결 경험

|문제|조치|
|---|---|
|`values.yaml` 적용 안됨|`valueFiles: values.yaml` 누락으로 Helm default 값만 사용됨|
|`LoadBalancer` 타입으로 생성됨|Helm values 미적용 상태 → NodePort 수동 override 또는 수정 후 sync|
|Application stuck at "deleting"|`kubectl patch ... -p '{"metadata":{"finalizers":null}}'`로 finalizer 강제 제거|

---

## 8. ✅ 최종 결과

- Ingress Controller: `ingress-nginx-app` 이름으로 배포
- 서비스: `NodePort` 방식 (80 → 30002, 443 → 30443)
- Argo CD에서 상태: `Synced`, `Healthy`
- 브라우저 및 curl로 외부 접속 확인 완료

---

## 9. 📎 참고 정보

- Helm Chart repo: [https://kubernetes.github.io/ingress-nginx](https://kubernetes.github.io/ingress-nginx)
- Helm Chart 버전: `4.10.0`
- GitOps 방식 적용: Argo CD + Git 리포 연동

---