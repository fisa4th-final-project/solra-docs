
## 📌 설치 환경

- OS: Ubuntu 24.04 LTS
- GitLab: Omnibus 설치 (Community Edition or Enterprise Edition)
- 접속 주소: `http://10.5.100.30`

---

## ✅ 1. GitLab Omnibus 설치

```bash
# 필수 패키지 설치
sudo apt update
sudo apt install -y curl ca-certificates gnupg

# 저장소 등록
curl <https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh> | sudo bash

# GitLab 설치 (external_url 포함)
sudo EXTERNAL_URL="<http://10.5.100.30>" apt install gitlab-ee

# 초기 구성
sudo gitlab-ctl reconfigure

```

---

## ⚠️ 2. 설치 중 발생한 주요 문제 및 트러블슈팅

### 🔸 문제 A: 설치가 `ruby_block[wait for logrotate service socket]`에서 멈춤

**원인:** `logrotate` 서비스가 `runit`에 의해 supervise되지 않음

**해결:**

```bash
# logrotate 서비스 디렉토리 재연결
sudo rm -rf /opt/gitlab/service/logrotate
sudo ln -s /opt/gitlab/sv/logrotate /opt/gitlab/service/logrotate

# runsvdir이 /opt/gitlab/service 를 감시하도록 실행
sudo /opt/gitlab/embedded/bin/runsvdir -P /opt/gitlab/service &

```

---

### 🔸 문제 B: `sv` 명령어 없음 오류

**원인:** `runit` 명령어가 누락됨

**해결:**

```bash
sudo apt install runit
```

---

### 🔸 문제 C: logrotate `supervise/status` 파일이 없음

**원인:** runit가 logrotate를 아직 감시하지 않음

**해결:** 위의 runsvdir 수동 실행으로 해결됨

---

## ✅ 3. 사용자 계정 수동 생성 (`solra`)

### ▶ 콘솔에서 생성

```ruby
user = User.new(
  username: 'solra',
  name: 'Solra Dev',
  email: 'solra@example.com',
  password: 'S0lr@_Secure_123!',
  password_confirmation: 'S0lr@_Secure_123!',
  admin: true,
  confirmed_at: Time.now
)

```

### ▶ GitLab 16.9+ 이상에서 필수: 조직 및 네임스페이스 생성

```ruby
org = Organizations::Organization.find_or_create_by!(name: 'SolraOrg', path: 'solra-org')

ns = Namespace.find_or_create_by!(
  name: user.name,
  path: user.username,
  type: 'UserNamespace',
  owner: user,
  organization: org
)

user.namespace = ns
user.save!

```

---

## ⚠️ 4. UI 로그인 시 `422 The change you requested was rejected`

**원인:**

- 세션 토큰 꼬임
- 사용자에 namespace 누락
- 브라우저 쿠키 캐시 문제

**해결 순서:**

1. 브라우저 시크릿 모드 또는 쿠키/캐시 삭제
    
2. `user.namespace` 누락 시 수동 생성 후 연결
    
3. 세션 토큰 초기화:
    
    ```ruby
    user.update_columns(session_token: nil)
    
    ```
    
4. GitLab 재시작:
    
    ```bash
    sudo gitlab-ctl restart
    
    ```
    

---

## ✅ 5. `root` 계정 비밀번호 재설정

```bash
sudo gitlab-rails console

user = User.find_by(username: 'root')
user.password = '<password>'
user.password_confirmation = '<password>'
user.save!

```

---

## ✅ 6. 최종 확인 체크리스트

|항목|결과|
|---|---|
|`gitlab-ctl status` 전체 서비스 `run:`|✅|
|`logrotate` `sv status` 정상 동작|✅|
|`solra` 계정 생성 및 로그인 가능|✅|
|`solra` 계정에 namespace 및 admin 권한|✅|
|root 계정 비밀번호 재설정 및 관리자 UI 진입|✅|
|GitLab 접속 정상 ([](http://10.5.100.30/)[http://10.5.100.30](http://10.5.100.30))|✅|

---

## 📦 보완 작업 권장

- ✅ SSH Key 등록 및 Git push 테스트
- ✅ 프로젝트 및 그룹 생성 테스트
- ✅ CI/CD `.gitlab-ci.yml` 구성
- ✅ 백업 스케줄링 설정 (`gitlab:backup:create`)
- ✅ HTTPS 인증서 적용 (Let's Encrypt 또는 자체 서명)

---

## 🧾 참고 명령 요약

|명령|설명|
|---|---|
|`sudo gitlab-ctl reconfigure`|GitLab 구성 반영|
|`sudo gitlab-ctl status`|서비스 상태 확인|
|`sudo gitlab-ctl restart`|전체 서비스 재시작|
|`sudo gitlab-rails console`|Rails 콘솔 접속|
|`sudo gitlab-rake gitlab:backup:create`|전체 백업 생성|

---