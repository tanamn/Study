---
layout: default
title: 8단계 - 공통 예외 처리 클래스 구현
permalink: /spring/spring-boot-practice/08-global-exception-handler/
---

{% include navigation.html %}

# 8단계. 공통 예외 처리 클래스 구현

이번 단계에서는 Service에서 발생한 예외를 Controller에서 일일이 처리하지 않고, 공통 예외 처리 클래스에서 일괄적으로 처리하는 방법을 학습한다.

계좌관리 프로젝트에서는 중복 계좌 등록, 존재하지 않는 계좌 조회, 잘못된 입력값 등 다양한 예외가 발생할 수 있다. 이번 실습의 목표는 API를 호출한 사용자(Postman 등)에게 HTTP 상태 코드와 오류 메시지를 JSON 형태로 전달하는 것이다.

## 8.1 현재 예외 처리 방식과 개선 방향

Service에서 중복 계좌를 발견하면 다음과 같이 예외를 발생시킨다.

```java
if (existingAccount != null) {
    throw new IllegalArgumentException("이미 등록된 계좌입니다.");
}
```

공통 예외 처리기를 만들면 이 예외를 일관된 HTTP 응답으로 변환할 수 있다.

```text
Postman
   |
   v
AccountController
   |
   v
AccountService
   |
   | 업무 규칙 위반
   v
예외 발생
   |
   v
GlobalExceptionHandler
   |
   | HTTP 상태 코드와 메시지 구성
   v
Postman에 JSON 응답 반환
```

| 구성 요소 | 역할 |
|---|---|
| Controller | HTTP 요청 수신 및 Service 호출 |
| Service | 업무 규칙 확인 및 예외 발생 |
| GlobalExceptionHandler | 예외를 공통으로 처리 |
| ResponseEntity | HTTP 상태 코드와 응답 본문 구성 |

## 8.2 exception 패키지 및 클래스 생성

다음 위치에 `exception` 패키지를 생성하고 `GlobalExceptionHandler.java` 파일을 만든다.

```text
src/main/java/com/example/accountmanagement/
├── controller/
├── dto/
├── mapper/
├── service/
├── exception/
│   └── GlobalExceptionHandler.java
└── AccountManagementApplication.java
```

`exception` 패키지는 `com.example.accountmanagement` 하위에 둔다. 그래야 Spring Boot의 기본 컴포넌트 스캔 범위에 포함된다.

## 8.3 GlobalExceptionHandler.java 작성

```java
package com.example.accountmanagement.exception;

import java.util.Map;

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {

    // 잘못된 입력값 및 업무 검증 오류 처리
    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<Map<String, String>> handleIllegalArgument(
            IllegalArgumentException e) {

        Map<String, String> errorResponse = Map.of(
            "status", "400",
            "error", "Bad Request",
            "message", e.getMessage()
        );

        return ResponseEntity
                .status(HttpStatus.BAD_REQUEST)
                .body(errorResponse);
    }

    // 그 외 예상하지 못한 서버 오류 처리
    @ExceptionHandler(Exception.class)
    public ResponseEntity<Map<String, String>> handleException(
            Exception e) {

        Map<String, String> errorResponse = Map.of(
            "status", "500",
            "error", "Internal Server Error",
            "message", "서버 내부 오류가 발생했습니다."
        );

        return ResponseEntity
                .status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(errorResponse);
    }
}
```

### 주요 어노테이션과 클래스

| 항목 | 설명 |
|---|---|
| `@RestControllerAdvice` | 여러 Controller에서 발생하는 예외를 공통 처리하고 결과를 응답 본문으로 반환 |
| `@ExceptionHandler` | 지정한 예외가 발생했을 때 실행할 메서드 지정 |
| `IllegalArgumentException.class` | 잘못된 입력값 또는 업무 검증 예외 처리 |
| `Exception.class` | 별도로 처리하지 않은 일반 예외 처리 |
| `ResponseEntity` | HTTP 상태 코드와 응답 본문을 함께 반환 |
| `Map.of()` | 오류 응답에 포함할 항목과 값 구성 |
| `e.getMessage()` | 예외 발생 시 전달된 메시지 조회 |

예를 들어 Service에서 다음 예외가 발생하면:

```java
throw new IllegalArgumentException("이미 등록된 계좌입니다.");
```

`handleIllegalArgument()`가 실행되고 `e.getMessage()`를 통해 해당 메시지를 응답에 포함한다.

일반 예외 처리에서는 내부 구현 정보나 SQL, 스택 트레이스를 사용자에게 노출하지 않도록 고정된 안내 메시지를 반환한다. 실제 원인 분석을 위해서는 서버 로그에 상세 예외를 기록하는 것이 좋다.

## 8.4 HTTP 상태 코드 이해하기

| HTTP 상태 코드 | 의미 | 예시 |
|---|---|---|
| 200 OK | 요청 성공 | 계좌 조회 및 수정 성공 |
| 201 Created | 신규 리소스 생성 성공 | 계좌 등록 성공 |
| 400 Bad Request | 잘못된 요청 | 필수 입력값 누락, 음수 잔액 |
| 404 Not Found | 요청한 리소스를 찾을 수 없음 | 존재하지 않는 계좌 |
| 409 Conflict | 기존 데이터와 충돌 | 중복 계좌 등록 |
| 500 Internal Server Error | 예상하지 못한 서버 오류 | 내부 처리 오류 |

현재 구현한 처리기는 `IllegalArgumentException`을 400으로, 그 외 `Exception`을 500으로 반환한다. 따라서 아직 404와 409가 자동으로 구분되는 것은 아니다. 다음 단계에서 사용자 정의 예외를 추가하면 세분화할 수 있다.

## 8.5 Postman 테스트 - 중복 계좌 등록

### ① Spring Boot 실행

프로젝트 최상위 폴더에서 실행한다.

```powershell
mvn spring-boot:run
```

### ② 이미 등록된 계좌번호로 POST 요청

Postman에서 다음과 같이 설정한다.

| 항목 | 값 |
|---|---|
| Method | POST |
| URL | `http://localhost:8080/api/accounts` |
| Header | `Content-Type: application/json` |
| Body | raw / JSON |

요청 JSON:

```json
{
    "accountNo": "1000000004",
    "accountName": "비상금통장",
    "customerName": "박민수",
    "balance": 500000
}
```

이 계좌번호가 이미 등록되어 있어야 중복 오류를 확인할 수 있다.

### ③ 응답 확인

예상 HTTP 상태:

```text
400 Bad Request
```

예상 JSON:

```json
{
    "status": "400",
    "error": "Bad Request",
    "message": "이미 등록된 계좌입니다."
}
```

## 8.6 잘못된 잔액 입력 테스트

Service의 잔액 검증 로직도 테스트한다.

```json
{
    "accountNo": "1000000005",
    "accountName": "테스트통장",
    "customerName": "테스트사용자",
    "balance": -1000
}
```

Service에서 다음 검증 코드가 실행된다.

```java
if (accountDto.getBalance() == null ||
    accountDto.getBalance().compareTo(BigDecimal.ZERO) < 0) {

    throw new IllegalArgumentException(
        "계좌 잔액이 올바르지 않습니다."
    );
}
```

예상 응답:

```json
{
    "status": "400",
    "error": "Bad Request",
    "message": "계좌 잔액이 올바르지 않습니다."
}
```

## 8.7 주의할 점

### ① 예외 처리기는 예외를 발생시키는 클래스가 아니다

Service는 업무 규칙을 확인하고 예외를 발생시킨다.

```java
throw new IllegalArgumentException("이미 등록된 계좌입니다.");
```

GlobalExceptionHandler는 발생한 예외를 받아 HTTP 응답으로 변환한다.

### ② 모든 예외를 400으로 처리하지 않는다

입력값 오류는 400이 적절하지만, 서버 내부 오류는 500으로 구분해야 한다. 실제 업무에서는 중복 데이터 충돌을 409, 조회 대상 미존재를 404 등으로 세분화할 수 있다.

### ③ 내부 오류 상세 내용을 사용자에게 노출하지 않는다

DB 접속 정보, SQL 문장, 스택 트레이스 등은 사용자 응답에 포함하지 않는다. 사용자에게는 이해할 수 있는 안내 메시지를 반환하고, 상세 오류는 서버 로그에서 확인한다.

### ④ 현재 구현은 학습용 기본 형태다

현재 예제는 `Map`으로 간단히 오류 응답을 구성한다. 실무에서는 `ErrorResponse`와 같은 별도 응답 DTO를 만들어 오류 응답 형식을 일관되게 관리할 수 있다.

## 8.8 이번 단계 완료 체크

- `exception` 패키지 생성
- `GlobalExceptionHandler.java` 생성
- `@RestControllerAdvice` 역할 이해
- `@ExceptionHandler` 역할 이해
- `IllegalArgumentException` 처리 확인
- 일반 `Exception` 처리 확인
- 중복 계좌 등록 시 HTTP 400 응답 확인
- 오류 메시지가 JSON으로 반환되는지 확인
- 음수 잔액 입력 시 오류 메시지 확인

## 다음 단계

다음 단계에서는 예외를 더 세분화한다.

- 잘못된 입력값: 400 Bad Request
- 계좌 미존재: 404 Not Found
- 중복 계좌: 409 Conflict

이를 위해 사용자 정의 예외 클래스를 만들고, GlobalExceptionHandler에서 예외별로 서로 다른 HTTP 상태 코드를 반환하도록 개선할 수 있다.

이번 단계의 핵심은 **Service는 업무 예외를 발생시키고, GlobalExceptionHandler는 이를 사용자에게 전달할 HTTP 응답으로 변환한다**는 역할 구분을 이해하는 것이다.
