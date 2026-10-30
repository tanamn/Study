---
layout: default
title: 7단계 - Controller 및 Postman API 테스트
permalink: /spring/account-practice/07-controller-api-test/
---

{% include navigation.html %}

# 7단계. Controller 작성 및 Postman API 테스트

## 1. AccountController 작성

경로: `src/main/java/com/example/accountmanagement/controller/AccountController.java`

```java
package com.example.accountmanagement.controller;

import java.util.List;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import com.example.accountmanagement.dto.AccountDto;
import com.example.accountmanagement.service.AccountService;

@RestController
@RequestMapping("/api/accounts")
public class AccountController {

    private final AccountService accountService;

    public AccountController(AccountService accountService) {
        this.accountService = accountService;
    }

    @GetMapping
    public List<AccountDto> findAll() {
        return accountService.findAll();
    }

    @GetMapping("/{accountNo}")
    public ResponseEntity<AccountDto> findByAccountNo(
            @PathVariable String accountNo) {
        AccountDto account = accountService.findByAccountNo(accountNo);
        if (account == null) {
            return ResponseEntity.notFound().build();
        }
        return ResponseEntity.ok(account);
    }

    @PostMapping
    public ResponseEntity<String> create(@RequestBody AccountDto account) {
        int result = accountService.create(account);
        if (result == 1) {
            return ResponseEntity.status(HttpStatus.CREATED)
                    .body("계좌가 등록되었습니다.");
        }
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body("계좌 등록에 실패했습니다.");
    }

    @PutMapping("/{accountNo}")
    public ResponseEntity<String> update(
            @PathVariable String accountNo,
            @RequestBody AccountDto account) {
        account.setAccountNo(accountNo);
        int result = accountService.update(account);
        if (result == 1) {
            return ResponseEntity.ok("계좌 정보가 수정되었습니다.");
        }
        return ResponseEntity.notFound().build();
    }
}
```

## 2. 애플리케이션 실행

프로젝트 루트에서 실행한다.

```powershell
.\mvnw.cmd spring-boot:run
```

## 3. Postman 테스트

기본 URL은 `http://localhost:8080/api/accounts`다.

### 전체 계좌 조회

- Method: `GET`
- URL: `http://localhost:8080/api/accounts`
- 예상 결과: HTTP 200, 초기 계좌 3건

### 계좌 단건 조회

- Method: `GET`
- URL: `http://localhost:8080/api/accounts/1000000001`
- 계좌가 없으면 HTTP 404

### 계좌 등록

- Method: `POST`
- URL: `http://localhost:8080/api/accounts`
- Body에서 `raw`와 `JSON` 선택

```json
{
  "accountNo": "1000000004",
  "accountName": "비상금통장",
  "customerName": "박민수",
  "balance": 500000
}
```

정상 등록 시 HTTP 201을 반환한다.

### 계좌 정보 수정

- Method: `PUT`
- URL: `http://localhost:8080/api/accounts/1000000004`
- Body에서 `raw`와 `JSON` 선택

```json
{
  "accountName": "비상금통장(수정)",
  "customerName": "박민수"
}
```

정상 수정 시 HTTP 200을 반환한다. 이 SQL은 계좌명과 고객명만 수정하고 잔액은 변경하지 않는다.

## 4. 테스트 요약

| 기능 | 요청 | 정상 결과 |
|---|---|---|
| 목록 조회 | GET `/api/accounts` | 200 |
| 단건 조회 | GET `/api/accounts/1000000001` | 200 |
| 없는 계좌 조회 | GET `/api/accounts/9999999999` | 404 |
| 계좌 등록 | POST `/api/accounts` | 201 |
| 계좌 수정 | PUT `/api/accounts/1000000004` | 200 |

## 5. 유의사항 및 확장

- 계좌번호 중복 등록 시 기본키 제약조건 오류가 발생할 수 있다.
- 금액은 `BigDecimal`로 처리한다.
- 현재 코드는 입력값 검증과 공통 예외 처리를 최소화한 학습용이다.
- 실제 서비스에서는 Bean Validation, 예외 처리, 인증·인가, 로깅, 트랜잭션 및 보안 설정을 보완해야 한다.
- H2 메모리 DB는 애플리케이션 재시작 시 데이터가 초기화된다.

## 최종 완료 체크

- [ ] 계좌 목록 조회
- [ ] 계좌 단건 조회
- [ ] 없는 계좌 404 확인
- [ ] 계좌 등록
- [ ] 계좌 정보 수정

## 다음 확장 학습

1. 입력값 검증 및 계좌번호 중복 처리
2. Service 트랜잭션 적용
3. 입금·출금 기능
4. 거래 내역 테이블 및 API
5. H2에서 Oracle로 DB 변경
