다음은 지금까지 SOLRA 시스템에 **권한을 적용한 모든 API**를 도메인별로 정리한 **권한 적용 명세서**입니다.

각 API에 대해 다음 항목을 포함합니다:

- 요구되는 권한 명칭 (`PERMISSION_NAME`)
- 권한 체크 위치 (Controller 단 / Service 단)
- 조직/부서 일치 검사 여부
- `ROOT` 우회 여부
- 설명 및 특이사항

---

# ✅ SOLRA 시스템 권한 적용 명세서

> 최신 구조 기준
> 
> `RBAC + 조직 일치 검증 혼합 구조`
> 
> `@PreAuthorize → Controller`, `조직 일치 확인 → Service`

---

## 📘 구조 개요

|항목|적용 위치|
|---|---|
|**권한명(PERMISSION_NAME)** 검사|✅ `@PreAuthorize(...)` (Controller 단)|
|**조직 ID(orgId)** 일치 여부 검사|✅ Service 단에서 `validateClusterAccess()`|
|**ROOT 권한 우회 허용 여부**|✅ 모든 도메인에서 허용 (`SecurityUtil.hasRole("ROOT")`)|
|**권한 체크 도구**|✅ `@permissionService.checkPermission("...")` 사용|

---

## 📂 도메인별 권한 정의 및 적용 방식

### 🔹 1. User 도메인

|기능|권한명|권한 위치|조직 검사|ROOT 우회|
|---|---|---|---|---|
|전체 목록 조회|`USER_READ`|✅ Controller|❌|✅|
|단일 조회|`USER_READ`|✅ Controller|✅ Service|✅|
|생성|`USER_CREATE`|✅ Controller|❌|✅|
|수정|`USER_UPDATE`|✅ Controller|✅ Service|✅|
|삭제|`USER_DELETE`|✅ Controller|✅ Service|✅|

---

### 🔹 2. Organization 도메인

|기능|권한명|권한 위치|조직 검사|ROOT 우회|
|---|---|---|---|---|
|생성|`ORGANIZATION_CREATE`|✅ Controller|❌|✅|
|전체 조회|`ORGANIZATION_READ`|✅ Controller|❌|✅|
|단일 조회|`ORGANIZATION_READ`|❌|✅ Service|✅|
|수정|`ORGANIZATION_UPDATE`|❌|✅ Service|✅|
|삭제|`ORGANIZATION_DELETE`|❌|✅ Service|✅|

---

### 🔹 3. Department 도메인

|기능|권한명|권한 위치|조직 검사|ROOT 우회|
|---|---|---|---|---|
|전체 조회|`DEPARTMENT_READ`|✅ Controller|❌|✅|
|단일 조회|`DEPARTMENT_READ`|❌|✅ Service|✅|
|생성|`DEPARTMENT_CREATE`|❌|✅ Service|✅|
|수정|`DEPARTMENT_UPDATE`|❌|✅ Service|✅|
|삭제|`DEPARTMENT_DELETE`|❌|✅ Service|✅|

---

### 🔹 4. Role 도메인

|기능|권한명|권한 위치|조직 검사|ROOT 우회|
|---|---|---|---|---|
|생성|`ROLE_CREATE`|✅ Controller|❌|✅|
|전체 조회|`ROLE_READ`|✅ Controller|❌|✅|
|단일 조회|`ROLE_READ`|✅ Controller|❌|✅|
|수정|`ROLE_UPDATE`|✅ Controller|❌|✅|
|삭제|`ROLE_DELETE`|✅ Controller|❌|✅|

---

### 🔹 5. Permission 도메인

|기능|권한명|권한 위치|조직 검사|ROOT 우회|
|---|---|---|---|---|
|생성|`PERMISSION_CREATE`|✅ Controller|❌|✅|
|전체 조회|`PERMISSION_READ`|✅ Controller|❌|✅|
|수정|`PERMISSION_UPDATE`|✅ Controller|❌|✅|
|삭제|`PERMISSION_DELETE`|✅ Controller|❌|✅|

---

### 🔹 6. Role-Permission 도메인

|기능|권한명|권한 위치|조직 검사|ROOT 우회|
|---|---|---|---|---|
|권한 부여|`ROLE_PERMISSION_ASSIGN`|✅ Controller|❌|✅|
|권한 조회|`ROLE_PERMISSION_READ`|✅ Controller|❌|✅|
|권한 제거|`ROLE_PERMISSION_REVOKE`|✅ Controller|❌|✅|

---

### 🔹 7. User-Role 도메인

|기능|권한명|권한 위치|조직 검사|ROOT 우회|
|---|---|---|---|---|
|역할 부여|`USER_ROLE_ASSIGN`|✅ Controller|✅ Service|✅|
|역할 조회|`USER_ROLE_READ`|✅ Controller|✅ Service|✅|
|역할 제거|`USER_ROLE_REVOKE`|✅ Controller|✅ Service|✅|

---

### 🔹 8. Cluster 도메인

|기능|권한명|권한 위치|조직 검사|ROOT 우회|
|---|---|---|---|---|
|생성|`CLUSTER_CREATE`|✅ Controller|❌|✅|
|전체 조회|`CLUSTER_READ`|✅ Controller|✅ (Service, orgId 필터)|✅|
|단일 조회|`CLUSTER_READ`|✅ Controller|✅ Service|✅|
|수정|`CLUSTER_UPDATE`|✅ Controller|✅ Service|✅|
|삭제|`CLUSTER_DELETE`|✅ Controller|✅ Service|✅|
|연결 테스트|`CLUSTER_READ`|✅ Controller|✅ Service|✅|

---

### 🔹 9. Deployment 도메인

|기능|권한명|권한 위치|조직 검사|ROOT 우회|
|---|---|---|---|---|
|조회 (리스트/단일)|`DEPLOYMENT_READ`|✅ Controller|✅ Service|✅|
|생성|`DEPLOYMENT_CREATE`|✅ Controller|✅ Service|✅|
|수정|`DEPLOYMENT_UPDATE`|✅ Controller|✅ Service|✅|
|삭제|`DEPLOYMENT_DELETE`|✅ Controller|✅ Service|✅|

---

### 🔹 10. Service 도메인

|기능|권한명|권한 위치|조직 검사|ROOT 우회|
|---|---|---|---|---|
|조회|`SERVICE_READ`|✅ Controller|✅ Service|✅|
|생성|`SERVICE_CREATE`|✅ Controller|✅ Service|✅|
|수정|`SERVICE_UPDATE`|✅ Controller|✅ Service|✅|
|삭제|`SERVICE_DELETE`|✅ Controller|✅ Service|✅|

---

### 🔹 11. Pod 도메인

|기능|권한명|권한 위치|조직 검사|ROOT 우회|
|---|---|---|---|---|
|조회|`POD_READ`|✅ Controller|✅ Service|✅|
|삭제|`POD_DELETE`|✅ Controller|✅ Service|✅|

---

### 🔹 12. Namespace 도메인

|기능|권한명|권한 위치|조직 검사|ROOT 우회|
|---|---|---|---|---|
|조회|`NAMESPACE_READ`|✅ Controller|✅ Service|✅|
|생성|`NAMESPACE_CREATE`|✅ Controller|✅ Service|✅|
|수정|`NAMESPACE_UPDATE`|✅ Controller|✅ Service|✅|
|삭제|`NAMESPACE_DELETE`|✅ Controller|✅ Service|✅|

---

### 🔹 13. Node 도메인

|기능|권한명|권한 위치|조직 검사|ROOT 우회|
|---|---|---|---|---|
|조회|`NODE_READ`|✅ Controller|✅ Service|✅|

---

## ✅ 설계 특징 요약

|특징|설명|
|---|---|
|🔐 권한 명칭은 기능 기반으로 직관적 네이밍 (`RESOURCE_ACTION`)||
|🧩 컨트롤러는 권한 선언만 담당 (`@PreAuthorize`)||
|🧠 서비스는 리소스 실제 소속 조직 검사 + ROOT 우회||
|📦 도메인별로 명확한 권한 범위 구분||
|📜 모든 예외는 `BusinessException + ErrorCode` 방식 통일||

---

## ✅ 활용 방안

- 🔍 Swagger 문서에 권한별 주석 추가 시 기반 문서로 활용
- ✅ 테스트 계정별 권한 부여/제한 기준 설계 시 체크리스트
- 🧪 E2E 테스트 시 `@WithMockUser + Permission` 세트 적용 기준
- 📄 운영 매뉴얼 또는 관리자 콘솔 권한 관리 UI 기획 시 기초 자료

---