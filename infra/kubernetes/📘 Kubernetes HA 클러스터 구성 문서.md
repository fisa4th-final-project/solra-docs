## 1. 🧭 클러스터 개요

| 항목            | 값                                   |
| ------------- | ----------------------------------- |
| 클러스터 명        | `onprem-ha-cluster`                 |
| 환경            | 온프레미스, Ubuntu 24.04.2 LTS           |
| Kubernetes 버전 | v1.28.15                            |
| 운영 노드 수       | 총 6대 (Control Plane 3대 + Worker 3대) |
| 커널 버전         | 6.8.0-59-generic                    |
| 설치 도구         | kubeadm + containerd                |
| 컨테이너 런타임      | containerd 1.7.x                    |
| 네트워크 플러그인     | Cilium                              |
| 클러스터 IP 대역    | `10.5.100.0/24`                     |
| 클러스터 로드밸런서    | HAProxy (`10.5.100.10:6443`)        |
| 접근 방식         | VIP 기반 kubectl 및 API 서버 접근          |

---

## 2. 🖥️ 클러스터 노드 구성
| 노드 이름      | 역할            | vCPU       | RAM      | Disk (SSD) | IP 주소     |
| ---------- | ------------- | ---------- | -------- | ---------- | --------- |
| `master-1` | Control Plane | 2 vCPU     | 4 GB     | 100 GB     | 10.5.0.11 |
| `master-2` | Control Plane | 2 vCPU     | 4 GB     | 100 GB     | 10.5.0.12 |
| `master-3` | Control Plane | 2 vCPU     | 4 GB     | 100 GB     | 10.5.0.13 |
| `worker-1` | Worker Node   | **4 vCPU** | **8 GB** | **100 GB** | 10.5.0.21 |
| `worker-2` | Worker Node   | **4 vCPU** | **8 GB** | **100 GB** | 10.5.0.22 |
| `worker-3` | Worker Node   | **4 vCPU** | **8 GB** | **100 GB** | 10.5.0.23 |
| `lb-node`  | HAProxy       | 2vCPU      | 2 GB     | 10 GB      | 10.5.0.10 |

---

## 3. 🔧 클러스터 설치 전체 과정

### 3.1 공통 사전 작업 (모든 노드)

```bash
# hostname 설정
hostnamectl set-hostname <hostname>

# /etc/hosts 설정
10.5.100.11 master
10.5.100.12 master2
10.5.100.13 master3
10.5.100.21 worker1
10.5.100.22 worker2
10.5.100.23 worker3

# swap 비활성화
swapoff -a && sed -i '/swap/d' /etc/fstab

# 커널 파라미터
modprobe overlay && modprobe br_netfilter
cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
sysctl --system

# containerd 설치
apt install -y containerd
containerd config default > /etc/containerd/config.toml
systemctl restart containerd && systemctl enable containerd

# Kubernetes 구성요소 설치
apt install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
apt update
apt install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl
```

### 3.2 Control Plane 초기화 (master)

```bash
kubeadm init \
  --control-plane-endpoint "10.5.100.10:6443" \
  --upload-certs \
  --pod-network-cidr=10.42.0.0/16 \
  --ignore-preflight-errors=Swap
```

```bash
# kubectl 설정
mkdir -p $HOME/.kube
cp /etc/kubernetes/admin.conf $HOME/.kube/config
```

```bash
# 인증서 키 업로드
kubeadm init phase upload-certs --upload-certs
```

### 3.3 Control Plane 조인 (master2, master3)

```bash
kubeadm join 10.5.100.10:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash> \
  --control-plane \
  --certificate-key <cert-key>
```

> ⚠️ `/etc/kubernetes`, `/var/lib/kubelet` 정리 필요 시: 
```bash
> kubeadm reset -f
```

### 3.4 Cilium CNI 설치
```bash
helm repo add cilium https://helm.cilium.io/
helm install cilium cilium/cilium --version 1.14.5 \
  --namespace kube-system \
  --set kubeProxyReplacement=true \
  --set k8sServiceHost=10.5.100.10 \
  --set k8sServicePort=6443
```

### 3.5 Worker 노드 조인
```bash
kubeadm join 10.5.100.10:6443 \
  --token <token> \
  --discovery-token-ca-cert-hash sha256:<hash>
```

---

## 4. ⚙️ 클러스터 상태 및 컴포넌트 확인
```bash
kubectl get nodes
kubectl get pods -n kube-system
kubectl get lease -n kube-system
kubectl cluster-info
curl -k https://10.5.100.10:6443/healthz
ETCDCTL_API=3 etcdctl endpoint status --write-out=table
```

|컴포넌트|상태|개수|비고|
|---|---|---|---|
|etcd|Running|3|static pod|
|kube-apiserver|Running|3|고가용성|
|kube-scheduler|Running|3|1 리더|
|controller-manager|Running|3|1 리더|
|coredns|Running|2||
|kube-proxy|Running|6|Cilium으로 대체 가능|

---

## 5. 🔀 HAProxy 구성

- 설정 위치: `/etc/haproxy/haproxy.cfg`
    
- 서비스 포트: `10.5.100.10:6443`
    
- 설정 예시:
    
```haproxy
backend kubernetes-masters
    balance roundrobin
    server master1 10.5.100.11:6443 check
    server master2 10.5.100.12:6443 check
    server master3 10.5.100.13:6443 check
```

---

## 6. 👑 리더 컴포넌트 상태

|컴포넌트|리더 노드|
|---|---|
|kube-scheduler|`master`|
|controller-manager|`master`|

> Standby 노드는 정상 대기 중이며 리더 전환 가능 상태입니다.

---

## 7. 🧪 운영 검증 및 장애 해결 이력

|이슈|원인 및 조치|
|---|---|
|kubelet 재시작 반복|`/etc/kubernetes/kubelet.conf` 누락 → reset 후 재설정|
|etcd 헬스체크 실패|apiserver down, Cilium 설치 누락 → 재설치|
|kubeadm join 권한 오류|ClusterRole 수동 생성|
|Helm 설치 충돌|release exists → `--replace` 옵션 사용|
|hostname 충돌|`hostnamectl` 및 `/etc/hosts` 정비|

---

## 8. 📂 주요 파일 및 디렉토리 정리

|경로|설명|
|---|---|
|`/etc/kubernetes/admin.conf`|kubeconfig|
|`/etc/kubernetes/manifests/`|static pod 정의|
|`/etc/containerd/config.toml`|containerd 설정|
|`/etc/cni/net.d/`|CNI 설정|

---

## 📌 부록: 클러스터 상태 예시 출력

```bash
kubectl get nodes

NAME      STATUS   ROLES           AGE     VERSION
master    Ready    control-plane   4d      v1.28.15
master2   Ready    control-plane   4d      v1.28.15
master3   Ready    control-plane   4d      v1.28.15
worker1   Ready    <none>          3d      v1.28.15
worker2   Ready    <none>          3d      v1.28.15
worker3   Ready    <none>          3d      v1.28.15
```

---

## 📌 문서 목적

현재 구성된 Kubernetes 고가용성 클러스터의 **구성, 설치 절차, 장애 대응 내역, 컴포넌트 상태**를 체계적으로 문서화하여 운영 안정성과 기술 인수인계 기반을 마련하기 위함입니다.
