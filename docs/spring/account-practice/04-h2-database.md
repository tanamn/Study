---
layout: default
title: 4단계 - H2 연결 및 계좌 테이블 생성
permalink: /spring/account-practice/04-h2-database/
---

{% include navigation.html %}

# 4단계. H2 데이터베이스 연결 및 초기 데이터 생성

## 1. H2 소개

H2는 Java 기반 경량 관계형 데이터베이스다. 별도 DB 서버 없이 실습할 수 있다. 이번 예제는 메모리 모드이므로 애플리케이션 종료 시 데이터가 사라진다.

## 2. application.yml

`src/main/resources/application.yml`에 작성한다.

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:accountdb
    driver-class-name: org.h2.Driver
    username: sa
    password:
  h2:
    console:
      enabled: true
      path: /h2-console
  sql:
    init:
      mode: always

mybatis:
  mapper-locations: classpath:mapper/**/*.xml
  type-aliases-package: com.example.accountmanagement.dto
  configuration:
    map-underscore-to-camel-case: true

server:
  port: 8080
```

`type-aliases-package`는 실제 DTO 패키지명과 일치시킨다.

## 3. 테이블 생성: schema.sql

`src/main/resources/schema.sql` 파일:

```sql
CREATE TABLE IF NOT EXISTS ACCOUNT (
    ACCOUNT_NO VARCHAR(20) PRIMARY KEY,
    ACCOUNT_NAME VARCHAR(100) NOT NULL,
    CUSTOMER_NAME VARCHAR(100) NOT NULL,
    BALANCE DECIMAL(15, 2) DEFAULT 0 NOT NULL
);
```

## 4. 초기 데이터: data.sql

`src/main/resources/data.sql` 파일:

```sql
INSERT INTO ACCOUNT (ACCOUNT_NO, ACCOUNT_NAME, CUSTOMER_NAME, BALANCE)
VALUES ('1000000001', '급여통장', '홍길동', 1500000.00);

INSERT INTO ACCOUNT (ACCOUNT_NO, ACCOUNT_NAME, CUSTOMER_NAME, BALANCE)
VALUES ('1000000002', '생활비통장', '김영희', 850000.00);

INSERT INTO ACCOUNT (ACCOUNT_NO, ACCOUNT_NAME, CUSTOMER_NAME, BALANCE)
VALUES ('1000000003', '저축통장', '이철수', 3200000.00);
```

## 5. H2 콘솔 접속

애플리케이션 실행 후 `http://localhost:8080/h2-console`에 접속한다.

| 항목 | 값 |
|---|---|
| JDBC URL | `jdbc:h2:mem:accountdb` |
| User Name | `sa` |
| Password | 공란 |

다음 SQL로 초기 데이터를 확인한다.

```sql
SELECT * FROM ACCOUNT;
```

H2 콘솔 활성화는 학습용 설정이다. 운영 환경에서는 비활성화하고 접근을 제한해야 한다.

## 완료 체크

- [ ] application.yml 작성
- [ ] schema.sql 작성
- [ ] data.sql 작성
- [ ] 애플리케이션 실행
- [ ] H2 콘솔에서 ACCOUNT 및 초기 데이터 확인

## 다음 단계

[5단계 - DTO 및 MyBatis Mapper 인터페이스 작성](../05-dto-mapper/)
