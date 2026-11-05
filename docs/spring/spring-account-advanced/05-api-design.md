---
layout: default
permalink: /spring/spring-account-advanced/05-api-design/
title: Spring 계좌관리 심화 과정 API 설계
---

{% include navigation.html %}

# 1. 문서 목적

계좌관리 업무를 외부에서 호출할 수 있도록 REST API의 URI, HTTP 메서드,
요청·응답 및 오류 규칙을 정의한다. 실제 구현 시 세부 필드와 상태 코드를
확정한다.

## 2. API 설계 원칙

-   리소스 중심의 URI를 사용한다.
-   조회는 GET, 생성성 거래 요청은 POST를 기본으로 한다.
-   요청 DTO와 응답 DTO를 분리한다.
-   Bean Validation으로 필수값과 금액 범위를 검증한다.
-   오류 응답 형식을 일관되게 유지한다.
-   인증 도입 후 사용자 소유 계좌인지 서버에서 확인한다.

## 3. API 목록(초안)

  ----------------------------------------------------------------------------------------------
  기능              Method            URI                                      설명
  ----------------- ----------------- ---------------------------------------- -----------------
  계좌 조회         GET               /api/accounts/{accountNo}                계좌 기본정보 및
                                                                               잔액

  입금              POST              /api/accounts/{accountNo}/deposits       입금 요청

  출금              POST              /api/accounts/{accountNo}/withdrawals    출금 요청

  계좌이체          POST              /api/transfers                           계좌 간 이체

  거래내역 조회     GET               /api/accounts/{accountNo}/transactions   거래내역 목록
  ----------------------------------------------------------------------------------------------

## 4. 입금 요청 예시

``` http
POST /api/accounts/10001/deposits
Content-Type: application/json
```

``` json
{
  "amount": 50000,
  "description": "현금 입금"
}
```

응답 예시:

``` json
{
  "transactionId": 100001,
  "accountNo": "10001",
  "transactionType": "DEPOSIT",
  "amount": 50000,
  "balanceAfter": 150000,
  "status": "COMPLETED"
}
```

## 5. 계좌이체 요청 예시

``` http
POST /api/transfers
Content-Type: application/json
```

``` json
{
  "fromAccountNo": "10001",
  "toAccountNo": "20002",
  "amount": 30000,
  "description": "생활비 이체"
}
```

이체 API는 요청 식별자 또는 멱등성 키를 활용한 중복 요청 방지 정책을
후속 설계에서 검토한다.

## 6. 오류 응답 형식 예시

``` json
{
  "code": "INSUFFICIENT_BALANCE",
  "message": "출금 가능 잔액이 부족합니다.",
  "timestamp": "2026-01-01T10:00:00"
}
```

오류 코드는 업무 오류와 시스템 오류를 구분하고, 내부 SQL이나 스택
트레이스 등 민감한 정보를 외부에 노출하지 않는다.

## 7. HTTP 상태 코드 기준

  상태 코드                   사용 예
  --------------------------- --------------------------------
  200 OK                      조회 또는 정상 처리
  201 Created                 거래 리소스 생성이 적절한 경우
  400 Bad Request             요청 형식 또는 입력값 오류
  401 Unauthorized            인증되지 않은 요청
  403 Forbidden               권한이 없는 요청
  404 Not Found               계좌 등 대상 리소스 없음
  409 Conflict                잔액·상태·중복 요청 충돌
  500 Internal Server Error   예기치 않은 시스템 오류

실제 상태 코드는 API의 의미와 오류 정책을 확정한 뒤 일관되게 적용한다.

## 8. API 명세 산출물

-   API 목록
-   요청·응답 DTO 정의
-   필드별 타입 및 필수 여부
-   검증 규칙
-   성공·실패 응답 예시
-   인증 및 권한 요구사항
-   Swagger / OpenAPI 문서
