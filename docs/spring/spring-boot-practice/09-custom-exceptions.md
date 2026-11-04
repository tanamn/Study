---
layout: default
title: 9단계 - 사용자 정의 예외 및 HTTP 상태 코드 구분
permalink: /spring/spring-boot-practice/09-custom-exceptions/
---

{% include navigation.html %}

# 9단계. 사용자 정의 예외 및 HTTP 상태 코드 구분

8단계에서는 `@RestControllerAdvice`와 `@ExceptionHandler`로 예외를 공통 처리했다. 이번 단계에서는 업무 상황에 맞는 사용자 정의 예외를 만들고 HTTP 상태 코드를 400, 404, 409, 500으로 구분한다.

## 9.1 예외 구분 기준

| 상황 | 예외 | HTTP 상태 |
|---|---|---:|
| 잘못된 입력값 | `IllegalArgumentException` | 400 Bad Request |
| 계좌 미존재 | `AccountNotFoundException` | 404 Not Found |
| 중복 계좌 | `DuplicateAccountException` | 409 Conflict |
| 예상하지 못한 오류 | 그 외 `Exception` | 500 Internal Server Error |

Service는 업무 상황에 맞는 예외를 발생시키고, `GlobalExceptionHandler`는 이를 HTTP 응답으로 변환한다.

## 9.2 패키지 및 파일 구성

기존 `exception` 패키지 아래에 두 클래스를 추가한다.

```text
src/main/java/com/example/accountmanagement/
└── exception/
    ├── GlobalExceptionHandler.java
    ├── AccountNotFoundException.java
    └── DuplicateAccountException.java
```

## 9.3 AccountNotFoundException.java

조회 또는 수정할 계좌가 존재하지 않을 때 사용한다.

```java
package com.example.accountmanagement.exception;

public class AccountNotFoundException extends RuntimeException {

    public AccountNotFoundException(String message) {
        super(message);
    }
}
```

`RuntimeException`을 상속한 비검사 예외이다. `super(message)`는 부모 예외 클래스에 메시지를 전달하며, 나중에 `getMessage()`로 읽을 수 있다.

## 9.4 DuplicateAccountException.java

이미 등록된 계좌번호로 신규 등록을 시도할 때 사용한다.

```java
package com.example.accountmanagement.exception;

public class DuplicateAccountException extends RuntimeException {

    public DuplicateAccountException(String message) {
        super(message);
    }
}
```

## 9.5 GlobalExceptionHandler.java 수정

8단계의 `GlobalExceptionHandler.java`를 다음 코드로 교체한다.

```java
package com.example.accountmanagement.exception;

import java.util.Map;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {

    // 잘못된 입력값: 400
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<Map<String, String>> handleIllegalArgument(
            IllegalArgumentException e) {

        return createErrorResponse(
            HttpStatus.BAD_REQUEST, e.getMessage()
        );
    }

    // 계좌 미존재: 404
    @ExceptionHandler(AccountNotFoundException.class)
    public ResponseEntity<Map<String, String>> handleAccountNotFound(
            AccountNotFoundException e) {

        return createErrorResponse(
            HttpStatus.NOT_FOUND, e.getMessage()
        );
    }

    // 중복 계좌: 409
    @ExceptionHandler(DuplicateAccountException.class)
    public ResponseEntity<Map<String, String>> handleDuplicateAccount(
            DuplicateAccountException e) {

        return createErrorResponse(
            HttpStatus.CONFLICT, e.getMessage()
        );
    }

    // 그 외 예상하지 못한 오류: 500
    @ExceptionHandler(Exception.class)
    public ResponseEntity<Map<String, String>> handleException(
            Exception e) {

        // 상세 원인은 서버 로그에서 확인하고 사용자에게 노출하지 않는다.
        return createErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR,
            "서버 내부 오류가 발생했습니다."
        );
    }

    // 공통 오류 응답 생성
    private ResponseEntity<Map<String, String>> createErrorResponse(
            HttpStatus status, String message) {

        Map<String, String> errorResponse = Map.of(
            "status", String.valueOf(status.value()),
            "error", status.getReasonPhrase(),
            "message", message
        );

        return ResponseEntity.status(status).body(errorResponse);
    }
}
```

`createErrorResponse()`는 오류 응답을 공통으로 만드는 메서드다. 각 예외 처리 메서드에서 반복되는 응답 생성 코드를 줄여준다.

## 9.6 AccountService 수정

예외 클래스를 만든 것만으로는 동작이 바뀌지 않는다. Service에서 기존 예외 발생 코드를 새 예외 클래스로 교체해야 한다.

### ① import 추가

`AccountService.java` 상단에 추가한다.

```java
import com.example.accountmanagement.exception.AccountNotFoundException;
import com.example.accountmanagement.exception.DuplicateAccountException;
```

### ② 상세 조회: 계좌 미존재

기존 코드:

```java
if (account == null) {
    throw new IllegalArgumentException("존재하지 않는 계좌입니다.");
}
```

변경 코드:

```java
if (account == null) {
    throw new AccountNotFoundException("존재하지 않는 계좌입니다.");
}
```

전체 메서드 예시:

```java
@Transactional(readOnly = true)
public AccountDto findByAccountNo(String accountNo) {

    validateAccountNo(accountNo);

    AccountDto account =
        accountMapper.findByAccountNo(accountNo);

    if (account == null) {
        throw new AccountNotFoundException(
            "존재하지 않는 계좌입니다."
        );
    }

    return account;
}
```

계좌번호가 null 또는 공백이면 `validateAccountNo()`에서 `IllegalArgumentException`이 발생하므로 400이다. 형식상 입력은 되었지만 DB에 계좌가 없으면 404다.

### ③ 계좌 등록: 중복 계좌

기존 코드:

```java
if (existing != null) {
    throw new IllegalArgumentException("이미 등록된 계좌입니다.");
}
```

변경 코드:

```java
if (existing != null) {
    throw new DuplicateAccountException(
        "이미 등록된 계좌입니다."
    );
}
```

나머지 입력값 검증과 INSERT 코드는 그대로 유지한다.

### ④ 계좌 수정: 수정 대상 미존재

기존의 `IllegalArgumentException`을 다음과 같이 변경한다.

```java
AccountDto existing =
    accountMapper.findByAccountNo(accountNo);

if (existing == null) {
    throw new AccountNotFoundException(
        "수정할 계좌가 존재하지 않습니다."
    );
}
```

## 9.7 Postman 테스트

수정 후 서버를 재실행한다.

```powershell
mvn clean compile
mvn spring-boot:run
```

### 테스트 ① 잘못된 입력값: 400

POST `http://localhost:8080/api/accounts`

Body → raw → JSON:

```json
{
    "accountNo": "",
    "accountName": "테스트통장",
    "customerName": "테스트사용자",
    "balance": 100000
}
```

예상 응답:

```text
400 Bad Request
```

```json
{
    "status": "400",
    "error": "Bad Request",
    "message": "계좌번호를 입력해 주세요."
}
```

메시지는 Service의 실제 검증 문구에 따라 달라질 수 있다.

### 테스트 ② 존재하지 않는 계좌 조회: 404

GET `http://localhost:8080/api/accounts/9999999999`

DB에 존재하지 않는 계좌번호를 사용한다.

예상 응답:

```text
404 Not Found
```

```json
{
    "status": "404",
    "error": "Not Found",
    "message": "존재하지 않는 계좌입니다."
}
```

### 테스트 ③ 중복 계좌 등록: 409

먼저 DB에 등록되어 있는 계좌번호를 사용해 POST 요청을 보낸다.

POST `http://localhost:8080/api/accounts`

```json
{
    "accountNo": "1000000004",
    "accountName": "비상금통장",
    "customerName": "박민수",
    "balance": 500000
}
```

`1000000004`가 실제로 등록되어 있어야 한다. 등록되어 있지 않으면 먼저 한 번 등록한 뒤 같은 요청을 다시 보낸다.

예상 응답:

```text
409 Conflict
```

```json
{
    "status": "409",
    "error": "Conflict",
    "message": "이미 등록된 계좌입니다."
}
```

### 테스트 ④ 존재하지 않는 계좌 수정: 404

PUT `http://localhost:8080/api/accounts/9999999999`

```json
{
    "accountName": "변경통장",
    "customerName": "테스트사용자",
    "balance": 300000
}
```

예상 응답은 404이며 메시지는 `수정할 계좌가 존재하지 않습니다.`이다.

## 9.8 예외 처리 흐름

```text
AccountController
       |
       v
AccountService
       |
       +-- 잘못된 입력값
       |      └── IllegalArgumentException → HTTP 400
       |
       +-- 계좌 미존재
       |      └── AccountNotFoundException → HTTP 404
       |
       +-- 계좌 중복
       |      └── DuplicateAccountException → HTTP 409
       |
       +-- 기타 예기치 않은 오류
              └── Exception → HTTP 500
```

## 9.9 이번 단계 완료 체크

- `AccountNotFoundException.java` 생성
- `DuplicateAccountException.java` 생성
- `GlobalExceptionHandler.java` 수정
- Service의 계좌 미존재 예외 변경
- Service의 중복 계좌 예외 변경
- 잘못된 입력값 테스트: HTTP 400
- 존재하지 않는 계좌 조회 테스트: HTTP 404
- 중복 계좌 등록 테스트: HTTP 409
- 존재하지 않는 계좌 수정 테스트: HTTP 404
- 응답 JSON의 상태 코드와 메시지 확인

## 9.10 실무 관점에서 기억할 점

1. 사용자 정의 예외는 업무 상황을 코드에서 명확하게 표현한다.
2. Service는 업무 상황에 맞는 예외를 발생시킨다. HTTP 상태 코드를 직접 결정하지 않는다.
3. GlobalExceptionHandler는 예외를 HTTP 응답으로 변환한다.
4. DB 접속 정보, SQL, 스택 트레이스 등 내부 상세 정보는 사용자 응답에 노출하지 않는다.
5. 중복 확인용 SELECT만으로 동시 등록을 완전히 막을 수 없다. 실제 DB에도 계좌번호 PRIMARY KEY 또는 UNIQUE 제약조건을 설정해야 한다.

이번 단계의 핵심은 **업무 예외와 HTTP 응답을 분리하고, 예외 유형에 따라 일관된 API 오류 응답을 제공하는 것**이다.

## 다음 단계

다음 단계에서는 계좌관리 API의 정상·오류 테스트를 정리하고 Controller부터 DB까지 이어지는 전체 호출 흐름을 점검한다. 계좌 등록·수정의 트랜잭션과 DB 제약조건도 함께 확인할 수 있다.
