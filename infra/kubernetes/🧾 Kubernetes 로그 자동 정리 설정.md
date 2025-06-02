Kubernetes 노드에서 로그 파일이 디스크를 과도하게 점유하지 않도록 **자동 정리하는 설정 과정 전체를 정리한 운영 문서**입니다.

## ✅ 목적
- 컨테이너 런타임(containerd) 환경에서 지속적으로 생성되는 `/var/log/containers` 디렉토리의 로그 파일 중, **7일 이상 지난 파일을 자동 삭제**하여 디스크 사용량을 관리합니다.
---
## 📁 대상 경로
- `/var/log/containers/`
    - 각 Pod의 STDOUT/STDERR 로그가 저장됨
    - 예시: `myapp-123456_default_mycontainer-abc123.log`

---
## 🛠️ 1. 로그 정리 스크립트 작성
### 경로
`/usr/local/bin/cleanup_logs.sh`
### 내용
```bash
#!/bin/bash

# 7일 이상 지난 로그 파일 삭제
find /var/log/containers -type f -mtime +7 -exec rm -f {} \\;
```
### 실행 권한 부여
```bash
sudo chmod +x /usr/local/bin/cleanup_logs.sh
```
---
## ⏱️ 2. 크론 작업 등록
### 명령어
```bash
sudo crontab -e
```
### 추가 내용
```
0 3 * * * /usr/local/bin/cleanup_logs.sh >> /var/log/cleanup_logs_cron.log 2>&1
```
- 매일 새벽 3시에 실행
- 결과 및 에러 로그를 `/var/log/cleanup_logs_cron.log`에 기록
---
## 🔍 3. 크론 서비스 상태 확인
```bash
sudo systemctl status cron
```
- `active (running)` 상태여야 정상 동작
---
## 📑 4. 로그 확인
```bash
sudo tail -n 50 /var/log/cleanup_logs_cron.log
```
- 삭제 결과, 에러 여부 확인 가능
---
## 🧪 5. 수동 테스트 (옵션)
```bash
sudo /usr/local/bin/cleanup_logs.sh
```
- 실제로 7일이 지난 로그가 있다면 삭제됨
---