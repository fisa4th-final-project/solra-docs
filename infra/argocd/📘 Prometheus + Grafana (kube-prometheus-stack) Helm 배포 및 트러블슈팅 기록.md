## 1. 📌 개요

이 문서는 Helm Chart를 활용해 `kube-prometheus-stack`을 설치하고, Argo CD 기반 GitOps 방식으로 모니터링 스택을 운영하는 과정을 기록한 문서입니다. 설치 중 발생한 문제와 해결 방법, 수집되는 메트릭 및 Grafana 대시보드 구성도 포함됩니다.


## 2. 🛠 설치 절차

### ① Helm Chart 설치

```bash
helm repo add prometheus-community <https://prometheus-community.github.io/helm-charts>
helm repo update
helm pull prometheus-community/kube-prometheus-stack --untar

```

### ② CRD 수동 설치

Helm `--skip-crds` 옵션을 사용한 경우, CRD는 별도로 설치해야 합니다. `crds/` 디렉토리가 비어있기 때문에 공식 저장소에서 직접 다운로드하여 적용합니다.

```bash
curl -O <https://raw.githubusercontent.com/prometheus-operator/prometheus-operator/main/example/prometheus-operator-crd/monitoring.coreos.com_prometheuses.yaml>

# yq를 사용한 사전 정리
yq -i 'del(.metadata.annotations)' monitoring.coreos.com_prometheuses.yaml
yq -i 'del(.spec.versions[].schema)' monitoring.coreos.com_prometheuses.yaml
yq -i '.spec.versions[].schema.openAPIV3Schema = {"type": "object", "x-kubernetes-preserve-unknown-fields": true}' monitoring.coreos.com_prometheuses.yaml

kubectl apply -f monitoring.coreos.com_prometheuses.yaml

```

---

## 3. ⚙️ Helm + Argo CD 배포 구성

### `values.yaml` 주요 설정

```yaml
grafana:
  enabled: true
  service:
    type: NodePort
    nodePort: 32000

prometheus:
  enabled: true
  service:
    type: NodePort
    nodePort: 32001
  prometheusSpec:
    storageSpec:
      volumeClaimTemplate:
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 10Gi
          storageClassName: local-path

prometheus-node-exporter:
  enabled: true

kube-state-metrics:
  enabled: true

```

### Argo CD Application 리소스 정의

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: solra-monitoring-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: <GitHub 저장소 URL>
    path: deploy/app/monitoring
    targetRevision: dev
    helm:
      valueFiles:
        - values.yaml
      skipCrds: true
  destination:
    server: <https://kubernetes.default.svc>
    namespace: solra-monitoring
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true

```

---

## 4. 🐞 트러블슈팅 이력

|문제|원인|해결 방법|
|---|---|---|
|CRD 적용 실패|`metadata.annotations` 길이 초과|`.metadata.annotations` 필드 제거|
|Prometheus 리소스 인식 실패|CRD 누락|CRD 수동 다운로드 및 적용|
|PVC Pending|`standard` StorageClass 없음|`local-path`로 지정 후 재배포|
|Prometheus Pod CrashLoopBackOff|`empty duration string` 오류|설정 누락 확인 및 Helm 템플릿 렌더링으로 원인 분석|

---

## 5. 📊 수집 메트릭 및 Grafana 대시보드

### 🔍 주요 수집 소스

- **Node Exporter**: CPU, 메모리, 디스크 I/O, 네트워크 등 시스템 메트릭
- **Kube State Metrics**: Kubernetes 리소스 상태 (Pod, Deployment 등)
- **Prometheus 자체 메트릭**: 알림 상태, rule 평가, scrape 상태 등

### 📈 기본 Grafana 대시보드

- Node Exporter Full
- Kubernetes / Compute Resources / Namespace (CPU, Memory)
- Kubernetes / Networking / Namespace (Bandwidth, Packets)
- Prometheus 2.0 Overview

> 모든 대시보드는 Grafana의 /var/lib/grafana/dashboards 디렉토리에서 자동 로딩됩니다.

---

## 6. 🔐 Grafana 로그인 정보

```bash
kubectl get secret --namespace solra-monitoring solra-monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 --decode

```

- ID: `admin`
- 비밀번호: 위 명령어로 확인
- 접속 주소: `http://<NodeIP>:32000`

---

## 7. 🧪 유용한 점검 명령어

```bash
# PVC 상태 확인
kubectl get pvc -n solra-monitoring

# Prometheus 리소스 상태
kubectl get prometheus -n solra-monitoring -o yaml

# Operator 로그 확인
kubectl logs deploy/<operator-name> -n solra-monitoring

# Helm 템플릿 렌더링 (설정 점검)
helm template solra-monitoring ./deploy/app/monitoring/charts/kube-prometheus-stack -f ./deploy/app/monitoring/values.yaml

```

---

## 8. ✅ 운영 팁 및 권장사항

- Helm 설정 변경 전 `helm template` 명령어로 미리 렌더링 결과 확인
- PVC가 필요한 경우 StorageClass 존재 여부를 먼저 확인
- CRD는 chart 버전에 맞게 수동으로 설치하거나 신중하게 skip 처리
- Grafana 대시보드는 JSON 내보내기 기능으로 백업 및 커스터마이징 가능

---