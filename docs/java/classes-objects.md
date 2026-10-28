---
layout: default
title: Java 클래스와 객체
permalink: /java/classes-objects/
---

{% include navigation.html %}

# Java 클래스와 객체

## 1. 클래스와 객체의 차이

- **클래스(Class):** 객체를 만들기 위한 설계도입니다.
- **객체(Object):** 클래스를 바탕으로 생성된 실체입니다.
- **인스턴스(Instance):** 특정 클래스에 따라 생성된 객체를 가리킬 때 사용하는 표현입니다.
- **필드(Field):** 객체의 상태를 저장하는 변수입니다.
- **메서드(Method):** 객체가 수행하는 동작입니다.

```java
public class Account {
    private String accountNumber;
    private long balance;

    public Account(String accountNumber, long balance) {
        this.accountNumber = accountNumber;
        this.balance = balance;
    }

    public long getBalance() {
        return balance;
    }
}
```

```java
Account account = new Account("123-456", 100_000);
System.out.println(account.getBalance());
```

## 2. 생성자

생성자는 객체를 생성할 때 초기 상태를 설정합니다.

```java
public class Customer {
    private String name;

    public Customer(String name) {
        this.name = name;
    }
}
```

- 생성자 이름은 클래스 이름과 같습니다.
- 생성자는 반환형을 선언하지 않습니다.
- 생성자를 명시하지 않으면 컴파일러가 기본 생성자를 제공할 수 있습니다.
- 다른 생성자를 선언하면 기본 생성자가 자동으로 제공되지 않습니다.

## 3. 캡슐화와 접근 제어자

캡슐화는 객체의 내부 상태를 직접 변경하지 못하게 하고, 정해진 메서드를 통해 접근하도록 설계하는 방식입니다.

| 접근 제어자 | 접근 범위 |
|---|---|
| `public` | 어디서나 접근 가능 |
| `protected` | 같은 패키지 및 다른 패키지의 하위 클래스 등 |
| 기본 접근(생략) | 같은 패키지 |
| `private` | 같은 클래스 내부 |

```java
public class Account {
    private long balance;

    public long getBalance() {
        return balance;
    }

    public void deposit(long amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("입금액은 0보다 커야 합니다.");
        }
        balance += amount;
    }
}
```

## 4. 상속과 다형성

상속은 기존 클래스의 특성을 확장하는 방법입니다.

```java
class Payment {
    public void pay() {
        System.out.println("결제 처리");
    }
}

class CardPayment extends Payment {
    @Override
    public void pay() {
        System.out.println("카드 결제 처리");
    }
}
```

다형성은 같은 상위 타입으로 서로 다른 하위 객체를 다룰 수 있는 특성입니다.

```java
Payment payment = new CardPayment();
payment.pay(); // 카드 결제 처리
```

- `extends`: 클래스를 상속할 때 사용합니다.
- `implements`: 인터페이스를 구현할 때 사용합니다.
- `@Override`: 상위 타입의 메서드를 재정의했음을 표시합니다.
- 인터페이스는 구현체가 제공해야 할 동작의 계약을 정의하는 데 유용합니다.

## 5. `static`, `final`, `this`

- `static`: 특정 객체가 아니라 클래스에 속하는 필드나 메서드를 선언합니다.
- `final` 변수: 한 번 할당한 값을 다시 대입할 수 없습니다.
- `final` 메서드: 하위 클래스에서 재정의할 수 없습니다.
- `final` 클래스: 상속할 수 없습니다.
- `this`: 현재 객체를 가리킵니다.

```java
public class Calculator {
    public static final double PI = 3.14159;

    public int add(int a, int b) {
        return a + b;
    }
}
```

## 6. `equals()`와 `hashCode()`

객체의 논리적 동등성을 비교하려면 `equals()`를 적절히 재정의해야 합니다. `HashMap`이나 `HashSet`에서 객체를 키 또는 원소로 사용할 때는 `equals()`와 `hashCode()`의 규약도 함께 지켜야 합니다.

## 학습 체크

- [ ] 클래스와 객체의 차이를 설명할 수 있다.
- [ ] 생성자와 `this`의 역할을 설명할 수 있다.
- [ ] 캡슐화와 접근 제어자를 설명할 수 있다.
- [ ] 상속과 다형성의 차이를 설명할 수 있다.
- [ ] `equals()`와 `hashCode()`가 함께 중요한 이유를 설명할 수 있다.

[이전: Java 기본 문법]({{ '/java/basics/' | relative_url }}) · [다음: 컬렉션]({{ '/java/collections/' | relative_url }})
