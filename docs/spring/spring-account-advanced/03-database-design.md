---
layout: default
permalink: /spring/spring-account-advanced/03-database-design/
title: Spring 계좌관리 심화 과정 - 3단계.데이터베이스 설계
---

{% include navigation.html %}

# Spring 계좌관리 심화 과정 - 3단계.데이터베이스 설계

## 1. 문서 목적

계좌관리 심화 프로젝트의 데이터 구조와 테이블 간 관계를 정의한다.

이번 설계에서는 계좌 기본정보, 현재 잔액, 거래 이력을 서로 다른 업무
데이터로 구분한다.

-   `ACCOUNT`: 계좌의 기본정보와 관리 상태
-   `ACCOUNT_BALANCE`: 계좌의 현재 잔액과 잔액 변경 버전
-   `ACCOUNT_TRANSACTION`: 입금·출금 등 잔액 변동의 거래 기록

이 문서는 학습용 모의 금융 시스템의 논리·물리 설계 기준이다. 실제
금융기관의 계정계 원장 구조나 규제 요건을 그대로 대체하지 않는다.

# 2. 설계 원칙

## 2.1 업무 책임 분리

계좌 기본정보를 변경하는 업무와 잔액을 변경하는 금융 거래 업무를
분리한다.

| 업무 | 관리 대상 |
|---|---|
| 계좌명 변경 | `ACCOUNT` |
| 계좌 상태 변경 | `ACCOUNT` |
| 고객과 계좌 연결정보 변경 | `ACCOUNT` |
| 현재 잔액 조회 | `ACCOUNT_BALANCE` |
| 입금·출금에 따른 잔액 변경 | `ACCOUNT_BALANCE` |
| 입금·출금 거래 기록 | `ACCOUNT_TRANSACTION` |

계좌명이나 계좌 상태를 변경할 때 잔액을 조회하거나 검증하지 않는다. 잔액
변경은 입금·출금·이체 등 거래 업무를 통해서만 수행한다.

## 2.2 현재 잔액과 거래 이력의 구분

-   `ACCOUNT_BALANCE`는 현재 잔액을 빠르게 조회하기 위한 현재 상태
    데이터다.
-   `ACCOUNT_TRANSACTION`은 잔액이 변경된 원인과 결과를 추적하기 위한
    거래 기록이다.
-   거래 이력은 사후 추적을 위해 원칙적으로 임의 수정·삭제하지 않는다.
-   현재 잔액과 거래 이력의 정합성을 검증할 수 있어야 한다.

# 3. 테이블 구성 및 관계

## 3.1 핵심 테이블

| 테이블 | 역할 | 관계 |
|---|---|---|
| `ACCOUNT` | 계좌 기본정보 | 기준 테이블 |
| `ACCOUNT_BALANCE` | 계좌별 현재 잔액 | `ACCOUNT`와 1:1 |
| `ACCOUNT_TRANSACTION` | 계좌별 거래 이력 | `ACCOUNT`와 1:N |

## 3.2 ERD

``` mermaid
erDiagram
    ACCOUNT ||--|| ACCOUNT_BALANCE : has
    ACCOUNT ||--o{ ACCOUNT_TRANSACTION : records

    ACCOUNT {
        varchar ACCOUNT_NO PK
        varchar ACCOUNT_NAME
        varchar CUSTOMER_ID
        varchar STATUS
        timestamp CREATED_AT
        timestamp UPDATED_AT
    }

    ACCOUNT_BALANCE {
        varchar ACCOUNT_NO PK, FK
        decimal BALANCE
        bigint VERSION
        timestamp UPDATED_AT
    }

    ACCOUNT_TRANSACTION {
        bigint TRANSACTION_ID PK
        varchar ACCOUNT_NO FK
        varchar TRANSFER_ID
        varchar TRANSACTION_TYPE
        decimal AMOUNT
        decimal BALANCE_AFTER
        varchar DESCRIPTION
        timestamp CREATED_AT
    }
```

## 3.3 관계 설명

-   하나의 계좌는 하나의 잔액 레코드를 가진다.
-   하나의 계좌에는 여러 거래 이력이 연결된다.
-   계좌가 생성될 때 잔액 레코드도 함께 생성한다.
-   거래 이력은 계좌번호를 통해 해당 계좌와 연결한다.
-   이체의 출금·입금 기록은 동일한 `TRANSFER_ID`로 연결한다.

# 4. ACCOUNT 테이블 - 계좌 기본정보

## 4.1 테이블 정의

계좌의 식별정보와 관리정보를 저장한다. 잔액 컬럼은 포함하지 않는다.

| 컬럼 | 데이터형(예시) | 필수 | 키 | 설명 |
|---|---|:---:|---|---|
| `ACCOUNT_NO` | `VARCHAR(20)` | Y | PK | 계좌번호 |
| `ACCOUNT_NAME` | `VARCHAR(100)` | Y |  | 계좌명 |
| `CUSTOMER_ID` | `VARCHAR(30)` | Y |  | 고객 식별자(학습용) |
| `STATUS` | `VARCHAR(20)` | Y |  | ACTIVE, SUSPENDED, CLOSED |
| `CREATED_AT` | `TIMESTAMP` | Y |  | 생성 시각 |
| `UPDATED_AT` | `TIMESTAMP` | Y |  | 최종 수정 시각 |

## 4.2 업무 규칙

-   계좌번호는 유일해야 한다.
-   계좌명과 고객 식별자는 필수값으로 검증한다.
-   계좌 상태는 정의된 값만 허용한다.
-   계좌 기본정보 수정은 잔액 데이터에 영향을 주지 않는다.
-   계좌 상태가 거래 가능 여부에 미치는 영향은 Service 업무 규칙에서
    확인한다.

# 5. ACCOUNT_BALANCE 테이블 - 현재 잔액

## 5.1 테이블 정의

계좌별 현재 잔액을 관리한다. `ACCOUNT_NO`를 기본키이자 ACCOUNT를
참조하는 외래키로 사용하여 1:1 관계를 표현한다.

| 컬럼 | 데이터형(예시) | 필수 | 키 | 설명 |
|---|---|:---:|---|---|
| `ACCOUNT_NO` | `VARCHAR(20)` | Y | PK, FK | 계좌번호 |
| `BALANCE` | `DECIMAL(18,2)` | Y |  | 현재 잔액 |
| `VERSION` | `BIGINT` | Y |  | 낙관적 잠금용 버전 |
| `UPDATED_AT` | `TIMESTAMP` | Y |  | 잔액 최종 변경 시각 |

## 5.2 업무 규칙

-   계좌 등록 시 잔액 레코드를 함께 생성한다.
-   최초 잔액은 기본적으로 0으로 시작한다.
-   잔액 변경은 입금·출금·이체 Service를 통해서만 수행한다.
-   출금은 잔액이 충분한 경우에만 성공해야 한다.
-   잔액은 `DECIMAL`과 같은 고정소수점 형식으로 관리한다.
-   `VERSION`은 낙관적 잠금을 적용할 때 사용하며, 실제 적용 방식은 구현
    단계에서 확정한다.

## 5.3 잔액 변경 예시

입금:

``` sql
UPDATE ACCOUNT_BALANCE
   SET BALANCE = BALANCE + #{amount},
       VERSION = VERSION + 1,
       UPDATED_AT = CURRENT_TIMESTAMP
 WHERE ACCOUNT_NO = #{accountNo};
```

출금은 잔액 확인과 차감을 분리하지 않고 조건을 UPDATE에 포함하는 방식을
우선 검토한다.

``` sql
UPDATE ACCOUNT_BALANCE
   SET BALANCE = BALANCE - #{amount},
       VERSION = VERSION + 1,
       UPDATED_AT = CURRENT_TIMESTAMP
 WHERE ACCOUNT_NO = #{accountNo}
   AND BALANCE >= #{amount};
```

UPDATE 결과 건수가 1이면 차감 성공, 0이면 계좌 없음 또는 잔액 부족
등으로 판단할 수 있다. 두 원인을 구분해야 한다면 별도 조회와 업무 오류
처리를 추가한다. 실제 동시성 보장은 DBMS와 트랜잭션 격리 수준을 포함해
테스트한다.

# 6. ACCOUNT_TRANSACTION 테이블 - 거래 이력

## 6.1 테이블 정의

입금·출금 등 잔액 변동의 근거를 기록한다. 이체는 출금 계좌와 입금 계좌에
각각 거래 기록을 남기고, 동일한 이체 식별자로 연결한다.

| 컬럼 | 데이터형(예시) | 필수 | 키 | 설명 |
|---|---|:---:|---|---|
| `TRANSACTION_ID` | `BIGINT` | Y | PK | 거래 이력 식별자 |
| `ACCOUNT_NO` | `VARCHAR(20)` | Y | FK | 거래 계좌번호 |
| `TRANSFER_ID` | `VARCHAR(36)` | N |  | 이체 연결 식별자 |
| `TRANSACTION_TYPE` | `VARCHAR(20)` | Y |  | DEPOSIT 또는 WITHDRAWAL |
| `AMOUNT` | `DECIMAL(18,2)` | Y |  | 거래 금액(양수) |
| `BALANCE_AFTER` | `DECIMAL(18,2)` | Y |  | 거래 처리 후 잔액 |
| `DESCRIPTION` | `VARCHAR(200)` | N |  | 거래 설명 |
| `CREATED_AT` | `TIMESTAMP` | Y |  | 거래 기록 시각 |

## 6.2 거래 유형

| 거래 유형 | 의미 | 잔액 영향 |
|---|---|---|
| `DEPOSIT` | 입금 | 증가 |
| `WITHDRAWAL` | 출금 | 감소 |

이체는 하나의 거래 이력 행으로만 표현하지 않는다. 출금 계좌에는
`WITHDRAWAL`, 입금 계좌에는 `DEPOSIT` 기록을 각각 남기고 두 행에 같은
`TRANSFER_ID`를 저장한다.

## 6.3 거래 이력 관리 원칙

-   거래 금액은 양수로 저장하고 증감 방향은 거래 유형으로 구분한다.
-   거래 후 잔액을 기록하여 거래 시점의 결과를 추적한다.
-   이체의 두 거래 기록은 동일한 트랜잭션에서 저장한다.
-   완료된 거래 기록은 직접 수정·삭제하지 않는 것을 원칙으로 한다.
-   거래 취소나 정정은 기존 거래를 지우는 대신 반대 방향의 별도 거래를
    기록하는 방식으로 확장할 수 있다.

# 7. 계좌 등록 및 거래 처리

## 7.1 계좌 신규 등록

계좌 기본정보와 잔액 레코드는 함께 생성한다.

``` text
계좌 등록 요청
    |
ACCOUNT INSERT
    |
ACCOUNT_BALANCE INSERT (초기 잔액 0)
    |
모두 성공?
  | 예              | 아니오
 COMMIT           ROLLBACK
```

계좌와 잔액 중 한쪽만 생성되는 상태가 발생하지 않도록 하나의
트랜잭션으로 처리한다.

## 7.2 입금 처리

``` text
입금 요청
  |
계좌 및 거래 가능 상태 확인
  |
입금 금액 검증
  |
ACCOUNT_BALANCE 잔액 증가
  |
ACCOUNT_TRANSACTION 입금 기록
  |
COMMIT
```

## 7.3 출금 처리

``` text
출금 요청
  |
계좌 및 거래 가능 상태 확인
  |
출금 금액 검증
  |
잔액 조건을 포함한 UPDATE
  |
ACCOUNT_TRANSACTION 출금 기록
  |
COMMIT
```

잔액 변경 또는 거래 이력 저장 중 하나라도 실패하면 전체 작업을 롤백한다.

## 7.4 계좌이체 처리

``` text
이체 요청
  |
출금·입금 계좌 검증
  |
출금 계좌 잔액 차감
  |
입금 계좌 잔액 증가
  |
출금·입금 거래 이력 기록
  |
전체 성공?
  | 예                  | 아니오
 COMMIT               ROLLBACK
```

이체는 `TRANSFER_ID`를 공통으로 사용하여 두 계좌의 거래 기록을 연결한다.
동시 이체에서 교착상태 위험을 줄이기 위해 계좌 잠금 순서 등도 일관되게
설계한다.

# 8. 인덱스 및 제약조건

| 대상 | 제약조건 또는 인덱스 | 목적 |
|---|---|---|
| `ACCOUNT.ACCOUNT_NO` | PK | 계좌 식별 |
| `ACCOUNT_BALANCE.ACCOUNT_NO` | PK, FK | 계좌별 잔액 1건 보장 |
| `ACCOUNT_TRANSACTION.TRANSACTION_ID` | PK | 거래 이력 식별 |
| `ACCOUNT_TRANSACTION.ACCOUNT_NO` | FK 및 조회 인덱스 | 계좌별 거래 조회 |
| `ACCOUNT_TRANSACTION(ACCOUNT_NO, CREATED_AT)` | 복합 인덱스 검토 | 기간별 거래 조회 |
| `ACCOUNT_TRANSACTION.TRANSFER_ID` | 인덱스 검토 | 이체 거래 추적 |

금액 양수, 잔액 0 이상, 상태 허용값 등은 애플리케이션 검증과 DB
제약조건을 함께 적용한다. 제약조건 문법과 인덱스 전략은 실제 DBMS에 맞춰
확정한다.

# 9. 데이터 정합성 및 동시성 설계

## 9.1 정합성 원칙

-   잔액 변경과 거래 이력 저장은 동일한 트랜잭션으로 처리한다.
-   거래 이력의 `BALANCE_AFTER`와 처리 시점의 잔액이 일치해야 한다.
-   이체의 출금·입금 및 두 거래 기록은 모두 성공하거나 모두 롤백되어야
    한다.
-   계좌 기본정보 수정은 잔액 변경과 독립적으로 수행한다.

## 9.2 동시성 제어 방식

| 방식 | 개념 | 검토 사항 |
|---|---|---|
| 조건부 UPDATE | 잔액 조건을 SQL에 포함하여 차감 | UPDATE 결과 건수 확인 |
| 비관적 잠금 | 대상 행을 잠근 뒤 처리 | 잠금 순서와 대기·교착 관리 |
| 낙관적 잠금 | VERSION 값으로 갱신 충돌 확인 | 충돌 시 재시도 또는 오류 처리 |

초기 구현에서는 조건부 UPDATE를 적용해 보고, 이후 비관적 잠금 또는
낙관적 잠금과 비교 실습한다.

# 10. DDL 예시

아래 DDL은 H2 학습 환경을 기준으로 작성한 예시다. 사용하는 DBMS에 따라
자동 증가 컬럼, 제약조건 및 시간 데이터형 문법을 조정한다.

``` sql
CREATE TABLE ACCOUNT (
    ACCOUNT_NO   VARCHAR(20)  NOT NULL,
    ACCOUNT_NAME VARCHAR(100) NOT NULL,
    CUSTOMER_ID  VARCHAR(30)  NOT NULL,
    STATUS       VARCHAR(20)  NOT NULL,
    CREATED_AT   TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP,
    UPDATED_AT   TIMESTAMP    NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT PK_ACCOUNT PRIMARY KEY (ACCOUNT_NO),
    CONSTRAINT CK_ACCOUNT_STATUS
        CHECK (STATUS IN ('ACTIVE', 'SUSPENDED', 'CLOSED'))
);

CREATE TABLE ACCOUNT_BALANCE (
    ACCOUNT_NO VARCHAR(20) NOT NULL,
    BALANCE    DECIMAL(18,2) NOT NULL DEFAULT 0,
    VERSION    BIGINT NOT NULL DEFAULT 0,
    UPDATED_AT TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT PK_ACCOUNT_BALANCE PRIMARY KEY (ACCOUNT_NO),
    CONSTRAINT FK_BALANCE_ACCOUNT
        FOREIGN KEY (ACCOUNT_NO) REFERENCES ACCOUNT (ACCOUNT_NO),
    CONSTRAINT CK_ACCOUNT_BALANCE_NONNEGATIVE
        CHECK (BALANCE >= 0)
);

CREATE TABLE ACCOUNT_TRANSACTION (
    TRANSACTION_ID   BIGINT GENERATED BY DEFAULT AS IDENTITY,
    ACCOUNT_NO       VARCHAR(20) NOT NULL,
    TRANSFER_ID      VARCHAR(36),
    TRANSACTION_TYPE VARCHAR(20) NOT NULL,
    AMOUNT           DECIMAL(18,2) NOT NULL,
    BALANCE_AFTER    DECIMAL(18,2) NOT NULL,
    DESCRIPTION      VARCHAR(200),
    CREATED_AT       TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT PK_ACCOUNT_TRANSACTION PRIMARY KEY (TRANSACTION_ID),
    CONSTRAINT FK_TRANSACTION_ACCOUNT
        FOREIGN KEY (ACCOUNT_NO) REFERENCES ACCOUNT (ACCOUNT_NO),
    CONSTRAINT CK_TRANSACTION_TYPE
        CHECK (TRANSACTION_TYPE IN ('DEPOSIT', 'WITHDRAWAL')),
    CONSTRAINT CK_TRANSACTION_AMOUNT
        CHECK (AMOUNT > 0),
    CONSTRAINT CK_TRANSACTION_BALANCE
        CHECK (BALANCE_AFTER >= 0)
);

CREATE INDEX IDX_TRANSACTION_ACCOUNT_DATE
    ON ACCOUNT_TRANSACTION (ACCOUNT_NO, CREATED_AT);
```

## 10.1 계좌 등록 데이터 예시

``` sql
INSERT INTO ACCOUNT (
    ACCOUNT_NO, ACCOUNT_NAME, CUSTOMER_ID, STATUS
) VALUES (
    '10001', '급여 계좌', 'CUST001', 'ACTIVE'
);

INSERT INTO ACCOUNT_BALANCE (
    ACCOUNT_NO, BALANCE
) VALUES (
    '10001', 0
);
```

실제 애플리케이션에서는 두 INSERT를 하나의 트랜잭션으로 실행한다.

# 11. 설계 검증 체크리스트

-   ACCOUNT에 잔액 컬럼이 없는가?
-   ACCOUNT와 ACCOUNT_BALANCE가 1:1 관계인가?
-   계좌 등록 시 잔액 레코드도 생성되는가?
-   계좌 기본정보 변경이 잔액 변경과 분리되는가?
-   입금·출금 시 잔액과 거래 이력이 함께 반영되는가?
-   이체의 출금·입금 기록이 동일한 TRANSFER_ID로 연결되는가?
-   거래 금액과 잔액에 적절한 데이터형 및 제약조건을 적용했는가?
-   동시 출금 상황에 대한 제어 방식이 정의되어 있는가?
-   거래 이력을 통한 잔액 검증이 가능한가?

# 12. 후속 산출물

-   ERD
-   테이블 정의서
-   DDL 및 초기 데이터 스크립트
-   PK / FK / CHECK 제약조건 목록
-   인덱스 정의서
-   MyBatis Mapper SQL 목록
-   동시성 및 트랜잭션 테스트 케이스

실제 구현을 시작하기 전에 DBMS와 동시성 제어 방식을 확정하고, DDL 및
테스트 시나리오를 해당 환경에 맞게 조정한다.
