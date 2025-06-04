## ✅ 1. 브랜치 종류 (Branch Types)

|브랜치|용도|특징|
|---|---|---|
|`main`|제품 릴리즈(배포)|항상 **배포 가능한 상태** 유지|
|`dev`|개발 통합|모든 기능 브랜치를 여기로 통합|
|`feature/기능명`|기능 개발|새로운 기능, 버그 수정 등 세부 작업 공간|

---

## ✅ 2. 브랜치 구조 (Branch Flow)

```
main
└── dev
    └── feature/기능이름
```

- 모든 개발 작업은 `feature/기능명` 브랜치에서 시작
    
- 작업 완료 후 `dev`로 PR(Pull Request)
    
- 통합 및 테스트 후 `main`으로 PR → 배포
    

---

## ✅ 3. 개발 작업 순서

1. `dev` 브랜치 최신 상태로 갱신
    
    ```bash
    git checkout dev
    git pull origin dev
    ```
    
2. 새 기능 브랜치 생성
    
    ```bash
    git checkout -b feature/기능이름
    ```
    
3. 개발 및 커밋 반복
    
4. 기능 개발 완료 후 `dev` 브랜치로 PR
    
5. 코드 리뷰 후 Merge 진행
    

---

## ✅ 4. 브랜치 명명 규칙

|상황|브랜치 이름 예시|
|---|---|
|기능 개발|`feature/login`, `feature/register-form`|
|버그 수정|`feature/fix-login-error`, `feature/fix-null-crash`|

---

## ✅ 5. 커밋 메시지 컨벤션

### 📌 커밋 타입 리스트

|타입|설명|
|---|---|
|`feat`|새로운 기능 추가|
|`fix`|버그 수정|
|`refactor`|리팩토링 (기능 변경 없이 구조 개선)|
|`design`|UI, CSS 수정|
|`comment`|주석 추가 또는 수정|
|`style`|코드 스타일 변경 (포맷, 세미콜론 등)|
|`docs`|문서 수정 (README 등)|
|`test`|테스트 코드 추가/수정/삭제|
|`chore`|기타 변경사항 (빌드 설정, 패키지 등)|
|`init`|초기 세팅|
|`rename`|파일/폴더 이름 변경|
|`remove`|파일 삭제|

---

### 📌 커밋 메시지 형식

```txt
type: Subject line

Body (본문 내용)

Footer (옵션, 이슈 번호 등)
```

#### ✅ Subject (제목)

- 최대 50자 이내
    
- 명령문 형태, 첫 글자 대문자
    
- 마침표(`.`) 금지
    
- 개조식 구문 (간결하고 요점 명확히)
    

#### ✅ Body (본문)

- 한 줄에 72자 이내
    
- 무엇을, 왜 변경했는지 설명
    
- “어떻게”보다 “무엇”과 “이유” 중심
    

#### ✅ Footer (꼬리말 - 선택)

- 이슈 번호 연동: `Fixes: #47`, `Related to: #32, #21`
    
- 유형:
    
    - `Fixes`: 이슈 수정 중
        
    - `Resolves`: 이슈 해결 완료
        
    - `Ref`: 참고용 이슈
        
    - `Related to`: 관련 이슈
        

---

### 📌 커밋 메시지 예시

```txt
Feat: Add signin, signup feature

- 회원가입 기능 구현
- 로그인 기능 API 연동 및 상태 관리 구현

Resolves: #29
```

---

## ✅ 6. 커밋 템플릿 설정 방법

1. `gitmessage.txt` 파일 생성 (원하는 경로에 저장)
    
2. 아래 템플릿 내용을 저장:
    

```txt
# 제목은 최대 50자까지 작성: 예) Feat: Add Key mapping  

# 본문 작성 (무엇을, 왜 변경했는지 설명)  

# 꼬릿말 작성: 예) Resolves: #23  

# --- COMMIT END ---  
# 커밋 타입 리스트  
# feat      : 기능 추가  
# fix       : 버그 수정  
# refactor  : 리팩토링  
# design    : UI/CSS 변경  
# comment   : 주석 추가/변경  
# style     : 코드 포맷 등 스타일 변경  
# docs      : 문서 수정  
# test      : 테스트 코드 변경  
# chore     : 기타 설정 변경  
# init      : 초기 세팅  
# rename    : 파일/폴더명 변경  
# remove    : 파일 삭제  
# ------------------  
# 제목은 대문자로 시작, 명령문 형식  
# 제목 끝에 마침표 금지  
# 제목-본문-꼬릿말 사이 한 줄 띄우기  
```

3. Git에 템플릿 등록
    

```bash
git config --global commit.template /경로/your_gitmessage.txt
```

예:

```bash
git config --global commit.template C:\\Users\\me\\.gitmessage.txt
```

---

