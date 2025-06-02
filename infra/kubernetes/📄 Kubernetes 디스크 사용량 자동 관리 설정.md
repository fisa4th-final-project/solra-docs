## 📌 목적
Kubernetes 노드에서 **컨테이너 이미지와 파드 로그 등으로 인해 디스크 사용량이 증가**하는 현상을 방지하고, **자동으로 이미지 정리 및 자원 회수**를 통해 디스크를 안정적으로 관리하기 위한 설정을 문서화한다.

---
## 🧱 1. 적용 대상
모든 Kubernetes 노드 (Master 및 Worker 노드 전체)

---
## ⚙️ 2. 설정 항목
### ✅ Kubelet 실행 옵션 수정
Kubelet 실행 시 아래와 같은 플래그를 추가하여 디스크 관리 정책을 구성함:
```bash
--image-gc-high-threshold=80
--image-gc-low-threshold=65
--eviction-hard=imagefs.available<10%
--eviction-soft=imagefs.available<15%
--eviction-soft-grace-period=imagefs.available=1m
```

### 각 옵션 설명:

|옵션|설명|
|---|---|
|`--image-gc-high-threshold=80`|이미지 정리를 시작하는 디스크 사용량 상한선 (%)|
|`--image-gc-low-threshold=65`|이미지 정리를 통해 도달할 목표 디스크 사용량 하한선 (%)|
|`--eviction-hard=imagefs.available<10%`|디스크 사용량이 90% 이상일 경우 강제 Pod 제거|
|`--eviction-soft=imagefs.available<15%`|디스크 사용량이 85% 이상일 경우 소프트 제거 조건 충족|
|`--eviction-soft-grace-period=imagefs.available=1m`|소프트 조건 충족 후 1분 지나면 Pod 제거 시작|

---

## 🗂️ 3. 설정 방법
### (1) `kubelet.service` 확인
```bash
cat /lib/systemd/system/kubelet.service
```
기본적으로 `ExecStart=/usr/bin/kubelet`로 되어 있으며, 세부 인자는 `/etc/default/kubelet` 또는 `/var/lib/kubelet/config.yaml`에서 설정함.

---
### (2) `/etc/default/kubelet` 수정
```bash
sudo nano /etc/default/kubelet
```
```bash
KUBELET_EXTRA_ARGS="--image-gc-high-threshold=80 --image-gc-low-threshold=65 --eviction-hard=imagefs.available<10% --eviction-soft=imagefs.available<15% --eviction-soft-grace-period=imagefs.available=1m"
```
---
### (3) kubelet 재시작
```bash
sudo systemctl daemon-reexec
sudo systemctl restart kubelet
```
---
## 🧪 4. 적용 확인
### (1) 실행 중인 kubelet 옵션 확인
```bash
ps -ef | grep kubelet
```
### (2) 로그 확인
```bash
journalctl -u kubelet -f
```
---
## 🧹 5. 기타 디스크 정리 명령어
### (1) 미사용 이미지 수동 제거
```bash
# containerd 사용 시
sudo crictl rmi --prune

# docker 사용 시
docker image prune -a -f
```
### (2) Evicted/Completed 파드 정리
```bash
kubectl get pods -A | grep -E 'Evicted|Completed' | awk '{print $1, $2}' | while read ns pod; do
  kubectl delete pod -n $ns $pod
done
```
---
## 🧾 6. 디스크 사용량 확인 명령어
```bash
sudo du -h --max-depth=1 /var | sort -hr    # /var 디스크 용량 사용량
df -h                                        # 전체 마운트 디스크 사용량
```
---
## ✅ 결론

이 설정을 통해 Kubernetes 노드에서 디스크 사용량이 80%를 초과하면 자동으로 이미지 정리를 수행하고, 90%를 초과하면 강제로 Pod를 제거하는 안전장치를 마련하였다. 모든 노드에 동일하게 설정하여 일관된 디스크 관리가 가능하다.

---