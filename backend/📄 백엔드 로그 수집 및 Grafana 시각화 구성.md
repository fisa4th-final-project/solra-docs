## ✅ 1. 로그 출력 방식 (Spring Boot)

### 🔹 출력 포맷

- AOP 기반 성공 요청 로그:
    
    ```plaintext
    ✅ 사용자 요청 | ID: {id} | ROLE: {roles} | IP: {ip} | URI: {uri} | 메서드: {method} | 결과: 성공 | 처리시간: {ms}ms
    ```
    
- AOP 기반 실패 요청 로그:
    
    ```plaintext
    ❌ 사용자 요청 실패 | ID: {id} | ROLE: {roles} | IP: {ip} | URI: {uri} | 메서드: {method} | 에러: {message}
    ```
    
- GlobalExceptionHandler:
    
    ```json
    {
      "@timestamp": "2025-05-30T11:30:07.1905059+09:00",
      "message": "❌ 권한 실패 | ID: 5 | ROLE: [] | IP: 0:0:0:0:0:0:0:1 | URI: /api/users | 메서드: GET | 에러: 이 작업을 수행할 권한이 없습니다.",
      "level": "WARN"
    }
    ```
    

### 🔹 출력 대상

- `stdout` (컨테이너 표준 출력)
    
- Promtail이 `/var/log/containers/*.log` 경로의 파일을 수집
    

---

## ✅ 2. 로그 수집 대상 지정

- Spring Boot 애플리케이션의 Deployment 또는 Pod 메타데이터에 `job` label을 명시
    

```yaml
metadata:
  labels:
    job: solra-backend
```

- Grafana Explore 탭에서 쿼리 예시:
    

```logql
{job="solra-backend"}
```

---

## ✅ 3. 로그 전달 경로

```
Spring Boot (stdout)
   ↓
container runtime log (예: /var/log/containers/solra-backend*.log)
   ↓
Promtail DaemonSet
   ↓
Loki (Push API)
   ↓
Grafana Explore 뷰에서 시각화 및 분석
```

---

## ✅ 4. 결과 확인

- Grafana 접속 후:
    
    - **Explore** → **Datasource: Loki**
        
    - `{job="solra-backend"}` 입력 후 검색
        
    - 로그 메시지 검색 가능 (예: `"권한"`, `"에러"`, `"성공"` 등)
        
    - 로그 레벨 필터도 가능 (예: `level="WARN"`)
        

---

이 구성을 통해 **Spring Boot 백엔드에서 출력하는 모든 표준 로그**는 Kubernetes 로그 파일을 통해 Promtail이 자동 수집하고, Loki에 저장되어 Grafana에서 실시간 확인 가능합니다.