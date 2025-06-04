아래는 Spring Boot 프로젝트에서 사용되는 **디렉토리 구조, 네이밍 규칙, URL 설계, DTO 가이드, 공통 응답 및 예외 처리 방식**을 문서화한 것입니다.

---

## 1. 📂 파일/폴더 구조 (Directory Structure)

```
src/main/java/com/yourproject
├── global/                    # 공통 모듈
│   ├── common/                # 유틸, 상수 등
│   ├── config/                # 설정 클래스 (Security, WebMvc 등)
│   ├── exception/             # 공통 예외 처리
│   └── response/              # ApiResponse 등 응답 포맷
│
├── domain/                   # 비즈니스 도메인
│   ├── user/
│   │   ├── controller/
│   │   ├── service/
│   │   ├── repository/
│   │   ├── dto/
│   │   ├── entity/
│   │   └── UserApplicationService.java (optional)
│   │
│   ├── product/
│   │   └── ... (동일 구조)
│
│   └── order/
│       └── ... (동일 구조)
│
└── Application.java          # 메인 실행 파일

```

---

## 2. 📌 네이밍 규칙

### 공통 규칙

|항목|규칙|
|---|---|
|클래스명|`UpperCamelCase`|
|메서드/변수|`lowerCamelCase`|
|패키지명|모두 소문자, 연결어 없이|

---

### Entity

|항목|규칙|
|---|---|
|이름|도메인 모델 명 (ex. User, Product)|

```java
@Entity
public class User { ... }

```

---

### Repository

|항목|규칙|
|---|---|
|이름|Entity명 + Repository|
|예시|UserRepository|

```java
public interface UserRepository extends JpaRepository<User, Long> { }

```

---

### Service

|항목|규칙|
|---|---|
|이름|Entity명 + Service|
|예시|UserService|

```java
@Service
public class UserService { ... }

```

---

### Controller

|항목|규칙|
|---|---|
|이름|Entity명 + Controller|
|예시|UserController|

```java
@RestController
@RequestMapping("/users")
public class UserController { ... }

```

---

### DTO

|구분|형식|
|---|---|
|Request|Entity명 + RequestDto (또는 Create/UpdateRequestDto)|
|Response|Entity명 + ResponseDto|

```java
public class UserRequestDto { ... }
public class UserResponseDto { ... }

```

---

### Exception

|항목|규칙|
|---|---|
|이름|도메인명 + Exception|
|예시|UserNotFoundException|

```java
public class UserNotFoundException extends RuntimeException { ... }

```

---

## 3. ✅ 개발 시 권장 사항

- Controller는 RESTful URL 형태 유지: `/users`, `/products`
- DTO → Entity 변환은 Service 또는 Mapper에서 처리
- 서비스 메서드 네이밍: `동사 + 대상` (예: `createUser()`, `deleteProduct()`)

---

## 4. 📚 Controller URL & HTTP Method 설계

|Method|URL|파라미터|설명|
|---|---|---|---|
|GET|`/users`|`id` (Query)|단일 조회|
|GET|`/users/all`|없음|전체 조회|
|POST|`/users`|JSON Body|생성|
|PUT|`/users`|`id`, JSON Body|수정|
|DELETE|`/users`|`id`|삭제|

> ✅ 모든 ID, 조건은 @RequestParam 혹은 SearchCondition DTO로 전달

---

## 5. 🔍 복수 조건 검색 예시

```java
@Getter @Setter
public class UserSearchCondition {
    private String name;
    private Integer age;
    private String city;
}

```

```java
@GetMapping
public ApiResponse<List<UserResponseDto>> searchUsers(UserSearchCondition condition) {
    return ApiResponse.success(userService.searchUsers(condition));
}

```

페이징 추가 시:

```java
@GetMapping
public ApiResponse<Page<UserResponseDto>> searchUsers(UserSearchCondition condition, Pageable pageable) {
    return ApiResponse.success(userService.searchUsers(condition, pageable));
}

```

---

## 6. 📦 DTO 사용 가이드

|타입|역할|
|---|---|
|RequestDto|Controller → Service로 데이터 전달|
|ResponseDto|Service → Controller로 응답 구조 생성|

---

## 7. 📤 공통 응답 포맷

```json
{
  "success": true,
  "code": 200,
  "message": "성공",
  "data": {
    ...
  }
}

```

- `success`: 성공 여부
- `code`: HTTP 상태 코드
- `message`: 응답 메시지
- `data`: 응답 데이터

---

## 8. 🚨 예외 처리 전략

- `@RestControllerAdvice` 사용
- 공통 예외 핸들러 클래스: `GlobalExceptionHandler`

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ApiResponse<Object>> handleUserNotFound(UserNotFoundException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(ApiResponse.fail(404, e.getMessage()));
    }
}

```

---