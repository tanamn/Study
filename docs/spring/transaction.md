---
layout: default
title: Spring 트랜잭션
permalink: /spring/transaction/
---

{% include navigation.html %}

# Spring 트랜잭션

## 1. 트랜잭션이란?

트랜잭션(Transaction)은 하나의 논리적인 작업 단위다. 여러 데이터베이스 작업을 묶어 모두 반영하거나 문제가 생겼을 때 변경을 취소하도록 설계할 수 있다. 계좌 이체처럼 출금과 입금이 함께 성공해야 하는 작업이 대표적인 예다.

## 2. ACID

| 속성 | 의미 |
|---|---|
| Atomicity (원자성) | 작업이 모두 수행되거나 모두 취소된다. |
| Consistency (일관성) | 데이터 규칙이 트랜잭션 전후로 유지된다. |
| Isolation (격리성) | 동시에 수행되는 트랜잭션 간 간섭을 제어한다. |
| Durability (지속성) | 커밋된 결과가 보존된다. |

실제 보장 범위는 데이터베이스와 격리 수준 설정에 따라 달라질 수 있다.

## 3. `@Transactional`

Spring에서는 보통 Service 메서드 또는 클래스에 `@Transactional`을 적용한다.

```java
@Service
public class TransferService {
    @Transactional
    public void transfer(Long fromId, Long toId, BigDecimal amount) {
        // 출금 처리
        // 입금 처리
    }
}
```

실제 계좌 조회, 잔액 검증, 출금·입금 구현은 별도로 필요하다.

## 4. Commit과 Rollback

- **Commit**: 변경 사항을 확정한다.
- **Rollback**: 트랜잭션의 변경 사항을 취소한다.

기본 설정에서 `RuntimeException` 또는 `Error`가 밖으로 전파되면 일반적으로 롤백된다. checked exception은 기본적으로 자동 롤백 대상이 아니므로 필요하면 다음과 같이 지정한다.

```java
@Transactional(rollbackFor = Exception.class)
public void process() throws Exception {
    // 데이터 변경 작업
}
```

예외를 메서드 내부에서 잡아 무시하면 Spring이 롤백해야 할 상황을 감지하지 못할 수 있다.

## 5. 전파 옵션

전파(propagation)는 기존 트랜잭션이 있을 때 새 메서드의 트랜잭션을 어떻게 처리할지 정한다.

| 옵션 | 의미 |
|---|---|
| `REQUIRED` | 기존 트랜잭션이 있으면 참여하고, 없으면 새로 시작한다. 기본값이다. |
| `REQUIRES_NEW` | 기존 트랜잭션을 중단하고 별도 트랜잭션을 시작한다. |
| `SUPPORTS` | 기존 트랜잭션이 있으면 참여하고, 없으면 트랜잭션 없이 실행할 수 있다. |

## 6. 주의할 점

- `@Transactional`은 일반적으로 Spring이 관리하는 Bean의 프록시를 통해 적용된다.
- 같은 객체 내부에서 `this.someMethod()`로 자기 메서드를 직접 호출하면 프록시를 거치지 않아 트랜잭션 설정이 적용되지 않을 수 있다.
- 예외를 잡아 무시하면 기대한 롤백이 발생하지 않을 수 있다.
- 외부 API 호출이나 긴 작업은 트랜잭션을 오래 유지할 수 있으므로 주의한다.
- `readOnly = true`는 읽기 전용 의도를 나타내는 설정이지만 모든 환경에서 쓰기를 강제로 차단하는 것은 아니다.

## 7. 학습 체크

1. Commit과 Rollback의 차이는?
2. `@Transactional`의 기본 롤백 대상은?
3. `REQUIRED`와 `REQUIRES_NEW`의 차이는?
4. 예외를 잡아 무시하면 롤백에 어떤 영향이 있을 수 있는가?

## 8. 정리

트랜잭션은 데이터 일관성을 유지하는 핵심 장치다. `@Transactional`을 사용할 때는 롤백 규칙, 전파 방식, 프록시 적용 조건을 함께 이해해야 한다.
