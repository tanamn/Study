---
layout: default
title: Java 기본 문법
permalink: /java/basics/
---

{% include navigation.html %}

# Java 기본 문법

## 1. Java 프로그램의 기본 구조

```java
public class HelloJava {
    public static void main(String[] args) {
        System.out.println("Hello, Java!");
    }
}
```

- `class`: 클래스를 선언합니다.
- `main()`: 일반적인 Java 애플리케이션의 시작 메서드입니다.
- `System.out.println()`: 콘솔에 값을 출력하고 줄을 바꿉니다.
- Java는 대소문자를 구분합니다. `String`과 `string`은 서로 다릅니다.

## 2. 변수와 자료형

변수는 데이터를 저장하기 위한 이름이 있는 공간입니다.

| 자료형 | 설명 | 예시 |
|---|---|---|
| `int` | 정수 | `int age = 30;` |
| `long` | 큰 범위의 정수 | `long count = 1000L;` |
| `double` | 실수 | `double rate = 3.14;` |
| `boolean` | 참 또는 거짓 | `boolean active = true;` |
| `char` | 문자 하나 | `char grade = 'A';` |
| `String` | 문자열(참조형) | `String name = "Java";` |

```java
int quantity = 3;
double price = 1500.0;
double total = quantity * price;

System.out.println(total); // 4500.0
```

금융 등 금액 계산에서는 부동소수점 오차를 피하기 위해 업무 요건에 따라 `BigDecimal`을 사용하는 경우가 많습니다.

### BigDecimal로 정밀한 숫자 계산하기

`double`은 일부 소수를 정확히 표현하지 못하므로 금액·이자·수수료 계산에는 `BigDecimal`이 적합합니다.

```java
import java.math.BigDecimal;
import java.math.RoundingMode;

BigDecimal a = new BigDecimal("0.1");
BigDecimal b = new BigDecimal("0.2");

System.out.println(a.add(b)); // 0.3

BigDecimal result = new BigDecimal("100")
        .divide(new BigDecimal("3"), 2, RoundingMode.HALF_UP);

System.out.println(result); // 33.33
```

- **생성:** `new BigDecimal("0.1")`처럼 문자열로 생성합니다. `new BigDecimal(0.1)`은 피하는 것이 좋습니다.
- **연산:** `add()`, `subtract()`, `multiply()`, `divide()`를 사용합니다.
- **반올림:** 나눗셈은 자릿수와 `RoundingMode`를 업무 규칙에 맞게 지정합니다.
- **비교:** 숫자 값 비교에는 `compareTo()`를 사용합니다. `equals()`는 소수점 자릿수(scale)도 비교합니다.
- **불변 객체:** 연산 결과는 새 객체이므로 결과를 변수에 저장해야 합니다.

```java
BigDecimal balance = new BigDecimal("100.0");
BigDecimal minimum = new BigDecimal("100.00");

System.out.println(balance.compareTo(minimum) == 0); // true
```

## 3. 연산자

- 산술: `+`, `-`, `*`, `/`, `%`
- 비교: `==`, `!=`, `>`, `<`, `>=`, `<=`
- 논리: `&&`, `||`, `!`
- 대입: `=`, `+=`, `-=`
- 증감: `++`, `--`

```java
int balance = 100_000;
int withdrawal = 30_000;

if (balance >= withdrawal) {
    balance -= withdrawal;
}
System.out.println(balance); // 70000
```

## 4. 조건문

```java
int score = 85;

if (score >= 90) {
    System.out.println("A");
} else if (score >= 80) {
    System.out.println("B");
} else {
    System.out.println("C 이하");
}
```

여러 선택지 중 하나를 분기할 때는 `switch`도 사용할 수 있습니다.

```java
String status = "APPROVED";

switch (status) {
    case "APPROVED":
        System.out.println("승인");
        break;
    case "REJECTED":
        System.out.println("거절");
        break;
    default:
        System.out.println("처리 중");
}
```

## 5. 반복문

```java
for (int i = 1; i <= 5; i++) {
    System.out.println(i);
}
```

```java
int count = 0;
while (count < 3) {
    System.out.println(count);
    count++;
}
```

- `for`: 반복 범위나 횟수가 명확할 때 유용합니다.
- `while`: 조건이 참인 동안 반복합니다.
- `break`: 반복문을 종료합니다.
- `continue`: 현재 반복을 건너뛰고 다음 반복으로 이동합니다.

## 6. 배열과 메서드

배열은 같은 자료형의 값을 여러 개 저장합니다.

```java
int[] scores = {80, 90, 75};

for (int score : scores) {
    System.out.println(score);
}
```

메서드는 특정 기능을 수행하도록 코드를 묶은 단위입니다.

```java
public static int add(int a, int b) {
    return a + b;
}

public static void main(String[] args) {
    int result = add(10, 20);
    System.out.println(result); // 30
}
```

- 매개변수: 메서드에 전달되는 입력값
- 반환형: 메서드가 돌려주는 값의 자료형
- `void`: 반환값이 없음을 의미
- 오버로딩: 이름은 같고 매개변수 목록이 다른 메서드를 정의하는 것

## 7. 문자열 비교

문자열의 내용 비교에는 일반적으로 `equals()`를 사용합니다.

```java
String status = "SUCCESS";

if ("SUCCESS".equals(status)) {
    System.out.println("성공");
}
```

참조형 변수의 `==`는 객체의 동일성을 비교하므로 문자열의 내용 비교에는 `equals()`를 사용하는 것이 좋습니다.

## 학습 체크

- 기본 자료형과 참조형의 차이를 설명할 수 있다.
- 조건문과 반복문을 작성할 수 있다.
- 메서드의 매개변수와 반환형을 설명할 수 있다.
- 문자열 내용 비교에 `equals()`를 사용하는 이유를 설명할 수 있다.

[다음: 클래스와 객체]({{ '/java/classes-objects/' | relative_url }})
