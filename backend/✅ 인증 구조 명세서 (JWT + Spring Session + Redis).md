# 1. 🔐 인증 구조 개요

SOLRA 서비스는 로그인 시 JWT를 생성하고, 해당 토큰을 Spring의 `HttpSession`에 저장합니다. 세션은 `Spring Session`을 통해 Redis에 저장되며, 사용자의 요청이 들어올 때마다 Redis에서 세션을 조회하여 JWT를 기반으로 인증 객체를 재생성합니다.

---

## 2. ⚙️ 인증 흐름

### 2.1 로그인 요청

```
POST /login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "securePassword"
}

```

### 2.2 로그인 성공 시 처리

- 서버는 다음 정보를 담은 **JWT**를 생성:
    
    ```json
    {
      "userId": 1,
      "orgId": 100,
      "deptId": 200,
      "role": "DEVELOPER"
    }
    
    ```
    
- JWT는 `HttpSession`에 저장됨:
    
    ```java
    session.setAttribute("JWT_TOKEN", jwt);
    
    ```
    
- `HttpSession`은 Redis에 자동 저장됨 (`spring-session-data-redis` 기반)
    

---

## 3. 🧾 Redis 저장 구조

|Key 예시|설명|
|---|---|
|`spring:session:sessions:<sessionId>`|세션 객체 (JWT 포함)|
|`spring:session:sessions:<sessionId>:expiry`|만료 시간|
|`spring:session:index:org.springframework.session.FindByIndexNameSessionRepository.PRINCIPAL_NAME_INDEX_NAME:<username>`|사용자별 세션 인덱스|

---

## 4. 🔄 요청 시 인증 처리

### 요청 흐름

1. 사용자는 세션 쿠키 (`JSESSIONID`) 포함하여 요청
    
2. 서버는 해당 `sessionId`로 Redis에서 세션 조회
    
3. 세션에 저장된 JWT를 꺼내 인증 객체 생성:
    
    ```java
    UsernamePasswordAuthenticationToken auth =
        new UsernamePasswordAuthenticationToken(userId, null, authorities);
    
    ```
    
4. `SecurityContextHolder`에 등록 → Spring Security 필터 통과
    

---

## 5. 📌 토큰 정보 구성

|필드|설명|
|---|---|
|`userId`|사용자 고유 ID|
|`orgId`|조직 ID|
|`deptId`|부서 ID|
|`role`|사용자 역할 (ROOT, ORG_ADMIN, DEPT_ADMIN, DEVELOPER 등)|

---

## 6. 🧪 세션 검증 예시

- 세션은 Redis에 다음과 같은 키로 저장됨:
    
    ```bash
    spring:session:sessions:<sessionId>
    
    ```
    
- 세션 TTL(Time To Live)은 `application.yml`에서 다음처럼 설정 가능:
    
    ```yaml
    spring:
      session:
        timeout: 30m
    
    ```
    

---