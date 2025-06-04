## 🧭 개요
- **배포 대상**: Jenkins
- **배포 방식**: Helm Chart
- **자동화 도구**: Argo CD (GitOps)
- **접근 방식**: NodePort (30003)
- **배포 환경**: 온프레미스 Kubernetes 클러스터 (`solra-jenkins` 네임스페이스)
---
## 📁 디렉토리 구조 (Git 저장소 기준
```
solra/
└── deploy/
    └── app/
        └── jenkins/
            ├── Chart.yaml         # 공식 Jenkins Helm Chart
            ├── values.yaml        # 사용자 정의 설정
            ├── templates/         # Helm 템플릿
            └── jenkins-app.yaml   # Argo CD Application 정의

```
---
## ⚙️ Step 1. Jenkins Helm Chart 다운로드 및 Git 등록
```bash
helm repo add jenkins <https://charts.jenkins.io>
helm repo update
helm pull jenkins/jenkins --untar
mv jenkins/ ./deploy/app/jenkins/

```
→ `jenkins` Helm chart 전체를 Git 저장소에 포함시켜 **커스터마이징 가능하게 구성**.

---
## 🛠 Step 2. `values.yaml` 설정 (NodePort + 계정 설정 포함)
```yaml
controller:
  serviceType: NodePort
  nodePort: 30003

  admin:
    username: "admin"
    password: "admin123"
    userKey: jenkins-admin-user
    passwordKey: jenkins-admin-password
    createSecret: true

  installPlugins:
    - kubernetes
    - workflow-aggregator
    - git
    - gitlab-plugin
    - blueocean

  persistence:
    enabled: true
    size: 10Gi
    storageClass: "local-path"

agent:
  enabled: true
  resources:
    requests:
      cpu: 500m
      memory: 512Mi
    limits:
      cpu: 1
      memory: 1Gi

```
---
## 🚀 Step 3. Argo CD Application 정의 (`jenkins-app.yaml`)

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: solra-jenkins-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: <https://github.com/fisa4th-final-project/solra.git>
    targetRevision: dev
    path: deploy/app/jenkins
    helm:
      valueFiles:
        - values.yaml
  destination:
    server: <https://kubernetes.default.svc>
    namespace: solra-jenkins
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true

```
---
## 🔁 Step 4. Git 커밋 및 Argo CD 앱 동기화
```bash
git add deploy/app/jenkins
git commit -m "feat: add Argo CD Helm-based Jenkins deployment"
git push origin dev

argocd app sync solra-jenkins-app
```

> Argo CD가 Git 저장소의 Helm chart와 values.yaml을 기반으로 Jenkins 자동 설치
---
## 🔍 Step 5. 배포 확인
### Pod 확인
```bash
kubectl get pods -n solra-jenkins
```
### NodePort 서비스 확인
```bash
kubectl get svc -n solra-jenkins
```
출력 예:
```
jenkins   NodePort   10.43.XXX.XXX   <none>   8080:30003/TCP   1m
```
---
## 🔐 Step 6. Jenkins 초기 로그인
```
접속 주소: http://<클러스터 노드 IP>:30003
ID: admin
PW: admin123
```
> ID/PW는 values.yaml에서 생성된 Secret을 통해 Helm이 자동으로 설정함.
---
