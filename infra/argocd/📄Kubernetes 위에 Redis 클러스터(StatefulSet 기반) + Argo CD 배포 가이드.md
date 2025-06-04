## ✨ 개요

본 문서는 Kubernetes 환경에서 `StatefulSet + Headless Service` 기반으로 Redis 2노드를 배포하고, **수동으로 Redis Cluster를 초기화**하여 클러스터링하는 과정을 정리한 실무 중심의 가이드입니다. 배포 및 설정 관리는 Argo CD를 통한 GitOps 방식으로 수행합니다.

---

## 🌐 환경 정보

|항목|값|
|---|---|
|Kubernetes 버전|v1.28.15|
|Redis 버전|`redis:7.2-alpine`|
|Redis 방식|클러스터 모드 (2 노드)|
|구성 방식|StatefulSet + Headless Service + ConfigMap|
|GitOps 도구|Argo CD|
|데이터 저장|PVC 1Gi (Pod 별로 자동 생성)|

---

## 🗂️ Git 디렉토리 구조 예시

```bash
redis-gitops/
└── redis-cluster/
    ├── redis-configmap.yaml         # redis.conf 정의
    ├── redis-headless-service.yaml  # 클러스터용 Headless Service
    ├── redis-statefulset.yaml       # StatefulSet 정의
    └── solra-redis-app.yaml         # Argo CD Application 정의

```

---

## ⚙️ 1. Redis 리소스 정의

### 📄 1-1. ConfigMap (`redis-configmap.yaml`)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-config
  namespace: redis
data:
  redis.conf: |
    port 6379
    cluster-enabled yes
    cluster-config-file nodes.conf
    cluster-node-timeout 5000
    appendonly yes
    protected-mode no

```

> Redis 클러스터 구성을 위한 필수 설정 포함: cluster-enabled, nodes.conf, appendonly 등.

---

### 📄 1-2. Headless Service (`redis-headless-service.yaml`)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: redis
spec:
  ports:
    - port: 6379
      name: redis
  clusterIP: None
  selector:
    app: redis

```

> Redis 노드 간 직접 통신을 위한 Headless Service (clusterIP: None) 구성.

---

### 📄 1-3. StatefulSet (`redis-statefulset.yaml`)

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
  namespace: redis
spec:
  serviceName: "redis"
  replicas: 2
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: redis:7.2-alpine
          command: ["redis-server", "/data/redis.conf"]
          ports:
            - containerPort: 6379
          volumeMounts:
            - name: redis-data
              mountPath: /data
            - name: config
              mountPath: /data/redis.conf
              subPath: redis.conf
      volumes:
        - name: config
          configMap:
            name: redis-config
  volumeClaimTemplates:
    - metadata:
        name: redis-data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi

```

> 각 Redis 노드에 독립적인 PVC와 ConfigMap 기반 설정 파일을 주입합니다.

---

## 🚀 2. Argo CD Application 정의

### 📄 `solra-redis-app.yaml`

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: redis-cluster
  namespace: argocd
spec:
  project: default
  source:
    repoURL: <https://github.com/fisa4th-final-project/solra-redis.git>
    targetRevision: dev
    path: redis-cluster
  destination:
    server: <https://kubernetes.default.svc>
    namespace: redis
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true

```

> Argo CD를 통해 Git 저장소로부터 Redis 구성 파일을 자동 배포/동기화합니다.

---

## 🔗 3. Redis 클러스터 수동 초기화

### ✅ 클러스터 생성 명령어 실행

1. 하나의 Redis Pod에 접속:

```bash
kubectl exec -it redis-0 -n redis -- ash

```

1. 클러스터 생성 명령 실행:

```bash
redis-cli --cluster create \\
  redis-0.redis.redis.svc.cluster.local:6379 \\
  redis-1.redis.redis.svc.cluster.local:6379 \\
  --cluster-replicas 0

```

1. 확인 요청에 `yes` 입력:

```
>>> Performing hash slots allocation on 2 nodes...
>>> Trying to optimize slaves allocation for anti-affinity
[OK] All 16384 slots covered.
>>> Do you want to proceed with the proposed cluster configuration (yes/no)? yes

```

1. 클러스터 상태 확인:

```bash
redis-cli -c -h redis-0.redis.redis.svc.cluster.local cluster nodes

```

> 두 노드가 각자 myself / connected로 표시되면 정상입니다.

---

## 📈 4. 배포 상태 점검

```bash
kubectl get pods -n redis
kubectl get svc -n redis
kubectl get pvc -n redis

```

- `redis-0`, `redis-1` Pod 상태 확인
- `redis` Headless Service 정상 여부 확인
- 각 Pod에 PVC가 바인딩(`Bound`)되어 있는지 점검

---

## 🔒 5. 보안 및 운영 팁

|항목|권장 사항|
|---|---|
|Redis 인증|`requirepass`, `masterauth` 설정 필요 시 `redis.conf`에 추가|
|설정 보안|민감 설정은 ConfigMap 대신 Secret 활용|
|PVC 백업|`hostPath` 또는 `CSI Snapshot` 등 활용|
|클러스터 복원|`nodes.conf`, `appendonly.aof` 파일 포함된 디렉토리 보존 필요|

---

## 📘 6. 참고 정보

|항목|설명|
|---|---|
|Redis 클러스터 명령|`redis-cli --cluster create`|
|클러스터 주소 체계|`pod-name.service.namespace.svc.cluster.local`|
|Slot 수|16384개|
|복제 설정|`--cluster-replicas` 옵션으로 지정 가능|
|StatefulSet 특성|Pod 번호(`redis-0`, `redis-1`)가 고정됨|

---