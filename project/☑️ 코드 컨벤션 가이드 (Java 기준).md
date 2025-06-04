## 📌 문자열 처리

- 문자열은 **쌍따옴표 `"`*를 사용합니다.

## 📌 세미콜론 사용

- 문장이 종료될 때는 **항상 세미콜론 `;`*을 붙입니다.

## 📌 패키지 네이밍

- **모두 소문자**로 작성하며, `_`(언더스코어) 및 대문자는 사용하지 않습니다.

```java
// Bad
package com.navercorp.apiGateway;
package com.navercorp.api_gateway;

// Good
package com.navercorp.apigateway;

```

## 📌 클래스 & 인터페이스 네이밍

- **파스칼 케이스**를 사용합니다.
- 클래스 이름은 **명사**나 **명사절**로 작성합니다.
- 인터페이스는 **명사, 형용사 또는 형용사절**로 작성 가능합니다.

```java
// Bad
public class reservation;
public class Accesstoken;

// Good
public class Reservation;
public class AccessToken;

public interface RowMapper;
public interface AutoClosable;

```

## 📌 함수명 & 변수명

- **카멜케이스** 사용 (예: `userName`, `getUserInfo`)
- **가독성 있게** 작성하며, 임시변수를 제외하고는 한 글자 사용 금지

```java
// Bad
HtmlParser p = new HtmlParser();

// Good
HtmlParser parser = new HtmlParser();

```

## 📌 클래스 구조

- 클래스 정의 시 **첫 줄과 마지막 줄에 빈 줄**을 둡니다.

```java
class Test {

    private int count;

}

```

## 📌 선언 가독성

- 한 줄에 하나의 문장만 작성합니다.

```java
// Bad
int base, weight;

// Good
int base;
int weight;

```

## 📌 주석 규칙

- 설명하는 코드의 **들여쓰기에 맞춰 작성**합니다.
- `//`, `/* */` 앞뒤에는 **공백**을 추가합니다.

```java
/*
 * This is a comment.
 */

System.out.print(true); // Print statement

```

## 📌 배열 네이밍

- 배열은 **복수형 변수명**을 사용합니다.

```java
// Bad
int[] number = new int[5];

// Good
int[] numbers = new int[5];

```

## 📌 컬렉션 네이밍

- `List`, `Set` 등의 경우 **구체적 타입 명시** 및 **복수형 변수명 사용**

```java
// Bad
ArrayList<Integer> numbers = new ArrayList<>();

// Good
List<Integer> numberList = new ArrayList<>();

```

## 📌 연산자, 콤마 공백 규칙

- **연산자 사이**, **콤마 다음**에는 공백을 둡니다.

```java
// Bad
a+b+c;
int[] arr = {1,2,3};

// Good
a + b + c;
int[] arr = {1, 2, 3};

```

## 📌 메소드 네이밍

- **동사**로 시작하도록 작성합니다.
- 변환 메소드는 **전치사**로 시작할 수 있습니다.

```java
// Good
renderHtml();
toString();

```

## 📌 생성자 함수명

- 함수 이름은 **대문자로 시작**합니다.

```java
function Person() {}

```

## 📌 boolean 함수 네이밍

- `is`, `has`, `can` 등의 접두어 사용

```java
boolean isAvailable = true;
boolean hasPermission = false;

```

## 📌 import 구문

- 대신 **구체적인 클래스명**을 명시합니다.

```java
// Bad
import java.util.*;

// Good
import java.util.List;
import java.util.ArrayList;

```

---

## ✅ 코드 컨벤션의 중요성

- 협업 시 **일관성 있는 코드 스타일**은 가독성과 유지보수성을 높입니다.
- 프로젝트에 코드 컨벤션을 도입했다는 사실은 **팀워크와 실무 능력**을 어필할 수 있는 강점이 됩니다.

---

## 📚 참고 자료

- [Toast UI 코딩 컨벤션](https://ui.toast.com/fe-guide/ko_CODING-CONVENTION)
- [네이버 캠퍼스 핵데이 Java 코딩 컨벤션](https://naver.github.io/hackday-conventions-java/)
    - [Eclipse용 설정 가이드](https://naver.github.io/hackday-conventions-java/#_eclipse)
    - [IntelliJ용 설정 가이드](https://naver.github.io/hackday-conventions-java/#_intellij)

---
