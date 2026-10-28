---
layout: default
title: Java JDBC
permalink: /java/jdbc/
---

{% include navigation.html %}

# Java JDBC

JDBC(Java Database Connectivity)는 Java 프로그램에서 관계형 데이터베이스에 접속하고 SQL을 실행하기 위한 API입니다.

## 1. JDBC 처리 순서

1. 데이터베이스 연결 획득
2. SQL 실행 객체 생성
3. SQL 실행
4. 조회 결과 처리
5. 사용한 자원 정리

## 2. 주요 객체

| 객체 | 역할 |
|---|---|
| `Connection` | 데이터베이스 연결 및 트랜잭션 관리 |
| `PreparedStatement` | 매개변수가 있는 SQL 실행 |
| `ResultSet` | SELECT 조회 결과 탐색 |
| `SQLException` | JDBC 처리 중 발생한 예외 |

## 3. PreparedStatement를 이용한 조회

```java
String sql = """
    SELECT ACCOUNT_ID, BALANCE
      FROM ACCOUNT
     WHERE ACCOUNT_ID = ?
    """;

try (Connection conn = dataSource.getConnection();
     PreparedStatement pstmt = conn.prepareStatement(sql)) {

    pstmt.setLong(1, accountId);

    try (ResultSet rs = pstmt.executeQuery()) {
        if (rs.next()) {
            long balance = rs.getLong("BALANCE");
            System.out.println(balance);
        }
    }
}
```

위 예제의 `dataSource`와 `accountId`는 미리 준비되어 있다고 가정합니다.

- `?`는 SQL 매개변수 자리 표시자입니다.
- `setLong(1, accountId)`는 첫 번째 매개변수에 값을 설정합니다.
- `executeQuery()`는 일반적으로 SELECT 실행에 사용됩니다.
- `executeUpdate()`는 INSERT, UPDATE, DELETE 등의 실행 건수를 반환합니다.
- `try-with-resources`는 사용이 끝난 자원을 자동으로 닫도록 도와줍니다.

## 4. SQL 파라미터 바인딩과 보안

사용자 입력값을 SQL 문자열에 직접 이어 붙이지 말고 `PreparedStatement`의 파라미터 바인딩을 사용합니다.

```java
// 권장하지 않는 방식
String sql = "SELECT * FROM ACCOUNT WHERE ACCOUNT_ID = '" + accountId + "'";
```

문자열 결합 방식은 SQL Injection 위험을 높일 수 있습니다. SQL 구조와 사용자 입력값을 분리하는 파라미터 바인딩을 사용하세요.

## 5. JDBC 트랜잭션

JDBC에서는 `Connection`의 자동 커밋 설정을 제어하여 여러 SQL을 하나의 트랜잭션으로 처리할 수 있습니다.

```java
try (Connection conn = dataSource.getConnection()) {
    conn.setAutoCommit(false);

    try {
        // 첫 번째 SQL 실행
        // 두 번째 SQL 실행
        conn.commit();
    } catch (SQLException e) {
        conn.rollback();
        throw e;
    }
}
```

- `setAutoCommit(false)`: 자동 커밋을 끕니다.
- `commit()`: 트랜잭션을 확정합니다.
- `rollback()`: 트랜잭션의 변경 사항을 취소합니다.

Spring에서는 일반적으로 `@Transactional`을 사용해 트랜잭션 경계를 선언적으로 관리합니다. Spring 트랜잭션과 JDBC를 함께 사용할 때는 동일한 트랜잭션 관리 체계와 연결을 사용하도록 구성해야 합니다.

## 학습 체크

- [ ] `Connection`, `PreparedStatement`, `ResultSet`의 역할을 설명할 수 있다.
- [ ] `executeQuery()`와 `executeUpdate()`의 차이를 설명할 수 있다.
- [ ] SQL 파라미터 바인딩이 필요한 이유를 설명할 수 있다.
- [ ] JDBC의 커밋·롤백과 Spring의 `@Transactional`을 연결해 설명할 수 있다.

[이전: 예외 처리]({{ '/java/exceptions/' | relative_url }}) · [Java 학습 목차]({{ '/java/' | relative_url }})
