
> **문서 목적:** Kubernetes 클러스터 내 핵심 구성 요소들의 역할과 동작 방식을 명확하게 이해하고, 각 요소의 기능과 실행 위치를 비교하여 운영 및 장애 대응에 활용하기 위함입니다.
---
## ✅ [1] kube-apiserver – _클러스터 제어의 중심 게이트웨이_

- 모든 요청(리소스 생성/조회/수정 등)을 수신
- 사용자, `kubectl`, 내부 컨트롤러 등이 접근하는 **유일한 진입점**
- `etcd`와 통신하여 클러스터 상태의 읽기/쓰기 담당
- **다중 Master 구성** 시 모든 Control Plane 노드에 독립 실행

📌 흐름:
```
사용자/관리자/컨트롤러 → kube-apiserver → etcd
```
---
## ✅ [2] etcd – _Kubernetes의 Key-Value 저장소_

- 클러스터의 전체 상태 정보를 저장
- 모든 리소스(파드, 시크릿, 서비스 등)의 영속적 저장소
- **3개 노드 이상으로 구성된 quorum 기반 클러스터** 권장
- 백업 필수: 손상 시 클러스터 상태 상실

📌 흐름:
```
파드 생성 요청 → kube-apiserver → etcd에 저장 → 스케줄러 감지
```
---
## ✅ [3] kube-controller-manager – _Desired State 유지 관리자_

- 다양한 컨트롤 루프를 통해 클러스터 상태를 원하는 상태로 유지
- 주요 컨트롤러:
    - ReplicaSet Controller: 파드 수 유지
    - Node Controller: 노드 장애 감지
    - Namespace Controller: 네임스페이스 관리
    - ServiceAccount Controller: SA 자동 생성 등

📌 흐름:
```
etcd 정보 감시 → 누락된 리소스 감지 → 자동 생성/복구
```
---
## ✅ [4] kube-scheduler – _Pod의 노드 배치 결정자_

- `Pending` 상태의 파드를 감지하여 적절한 워커 노드에 배치
- 고려 조건: 노드 자원, affinity, taint/toleration 등
- 스케줄링 알고리즘은 기본 제공되며, 확장 가능

📌 흐름:
```
New Pod → kube-scheduler → 적절한 노드에 배치
```
---
## ✅ [5] kube-proxy – _Service 트래픽 라우팅 담당_

- 각 노드에서 실행되며 `iptables` 또는 `eBPF`로 서비스 트래픽 처리
- `Service` → Pod 라우팅 수행
- CNI 플러그인(Cilium 등)과 연동

📌 흐름:
```
ClusterIP:Port → kube-proxy → PodIP:Port
```
---
## ✅ [6] CoreDNS – _클러스터 내 DNS 서비스 제공자_

- `kube-dns`의 현대적 대체
- DNS 이름(`svc.cluster.local` 등)을 IP로 변환
- 외부 DNS 포워딩 지원

📌 흐름:
```
Pod → DNS 요청 → CoreDNS → IP 반환
```
---
## ✅ [7] Cilium – _eBPF 기반 고성능 CNI_

- CNI(Network Plugin) + 보안 + 네트워크 제어 기능 제공
- eBPF 기반 네트워크 처리로 높은 성능
- 네트워크 정책(CNP) 설정 및 Hubble 통한 시각화 지원
- `cilium-operator` + `cilium` DaemonSet 구성

📌 흐름:
```
Pod 생성 시 → Cilium이 네트워크 라우팅 및 인터페이스 구성
```
---
## 🧩 핵심 비교 요약표

|구성 요소|주요 역할|실행 위치|
|---|---|---|
|**kube-apiserver**|클러스터 요청 처리|Control Plane|
|**etcd**|상태 정보 저장|Control Plane|
|**kube-controller-manager**|리소스 상태 유지|Control Plane|
|**kube-scheduler**|Pod 스케줄링|Control Plane|
|**kube-proxy**|네트워크 라우팅 처리|모든 노드 (DaemonSet)|
|**CoreDNS**|DNS 요청 응답|kube-system 네임스페이스|
|**Cilium**|네트워크/보안/CNI|모든 노드 (CNI DaemonSet)|
