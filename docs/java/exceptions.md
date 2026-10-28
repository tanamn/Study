---
layout: default
title: Java 예외 처리
permalink: /java/exceptions/
---

{% include navigation.html %}

# Java 예외 처리

예외 처리는 프로그램 실행 중 발생한 오류 상황을 감지하고 적절히 대응하는 방법입니다.

## 1. `try-catch-finally`

```java
try {
    int result = 10 / 0;
    System.out.println(result);
} catch (ArithmeticException e) {
    System.err.println("산술 연산 오류: " + e.getMessage());
} finally {
    System.out.println("마무리 작업");
}
```

- `try`: 예외가 발생할 수 있는 코드를 실행합니다.
- `catch`: 지정한 예외를 처리합니다.
- `finally`: 일반적으로 예외 발생 여부와 관계없이 마무리 작업을 수행합니다. 다만 JVM 종료 등 특수한 상황에서는 실행이 보장되지 않습니다.

## 2. Checked Exception과 Unchecked Exception

| 구분 | 특징 | 예시 |
|---|---|---|
| Checked Exception | 컴파일 시 예외 처리 또는 선언이 요구됨 | `IOException`, `SQLException` |
| Unchecked Exception | `RuntimeException` 계열이며 필수 선언 대상이 아님 | `NullPointerException`, `IllegalArgumentException` |

## 3. `throw`와 `throws`

- `throw`: 예외 객체를 실제로 발생시킵니다.
- `throws`: 메서드가 전달할 수 있는 예외를 선언합니다.

```java
public void validateAmount(long amount) {
    if (amount <= 0) {
        throw new IllegalArgumentException("금액은 0보다 커야 합니다.");
    }
}
```

```java
public String readFile() throws IOException {
    // 파일 읽기 작업
    return "내용";
}
```

## 4. 사용자 정의 예외

업무 규칙을 명확히 표현하기 위해 사용자 정의 예외를 만들 수 있습니다.

```java
public class InsufficientBalanceException extends RuntimeException {
    public InsufficientBalanceException(String message) {
        super(message);
    }
}
```

```java
if (balance < amount) {
    throw new InsufficientBalanceException("잔액이 부족합니다.");
}
```

## 5. 예외 처리 시 주의점

- 예외를 무시하거나 원인 없이 빈 `catch` 블록으로 삼키지 않습니다.
- 로그에는 원인과 업무 식별 정보를 적절히 남기되 비밀번호, 인증정보 등 민감한 데이터를 기록하지 않습니다.
- 호출자에게 전달할 예외와 현재 계층에서 처리할 예외를 구분합니다.
- 예외를 다시 던질 때는 원래 원인을 보존하는 것이 좋습니다.
- Spring의 `@Transactional`에서는 예외 종류와 전파 여부가 롤백에 영향을 줄 수 있습니다. 기본 정책은 일반적으로 `RuntimeException`과 `Error`에 롤백하고, 체크 예외에는 자동 롤백하지 않습니다. 필요하면 `rollbackFor`로 정책을 지정합니다.

## 학습 체크

- [ ] Checked Exception과 Unchecked Exception을 구분할 수 있다.
- [ ] `throw`와 `throws`의 차이를 설명할 수 있다.
- [ ] 예외를 로그로 남기고 다시 전달하는 이유를 설명할 수 있다.
- [ ] Spring 트랜잭션의 롤백 정책과 예외의 관계를 설명할 수 있다.

[이전: 컬렉션]({{ '/java/collections/' | relative_url }}) · [다음: JDBC]({{ '/java/jdbc/' | relative_url }})
