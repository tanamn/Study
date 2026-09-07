---
layout: default
title: WebSquare 트랜잭션과 예외 처리
permalink: /websquare/transaction/
---

{% include navigation.html %}

# WebSquare 트랜잭션과 예외 처리

WebSquare에서 등록·수정·삭제 요청을 보내면 실제 데이터 처리는 Spring과 데이터베이스에서 수행된다.

이때 여러 SQL을 하나의 업무로 묶어 **전부 성공하거나 전부 취소**되도록 하는 기능이 트랜잭션(Transaction)이다. 처리 중 발생한 문제를 적절한 응답으로 변환하고 WebSquare 화면에 안내하는 과정이 예외 처리(Exception Handling)이다.

```text
WebSquare 요청
    ↓
Spring Controller
    ↓
Service의 트랜잭션 시작
    ↓
Mapper·DB 처리
    ├─ 모두 성공 → Commit
    └─ 예외 발생 → Rollback
                       ↓
                공통 예외 처리
                       ↓
              WebSquare 오류 안내
```

---

## 1. 트랜잭션이란?

트랜잭션은 여러 데이터베이스 작업을 하나의 작업 단위로 처리하는 것이다.

고객 3명을 일괄 등록한다고 가정한다.

```text
1번 고객 INSERT 성공
2번 고객 INSERT 성공
3번 고객 INSERT 실패
```

트랜잭션을 적용하지 않으면 1번과 2번 고객만 저장되어 데이터가 불완전해질 수 있다. 트랜잭션을 적용하면 3번 고객에서 오류가 발생했을 때 앞서 저장한 내용까지 모두 취소할 수 있다.

```text
모든 처리 성공 → Commit   → DB에 최종 반영
처리 중 오류   → Rollback → 해당 작업의 변경 내용 취소
```

---

## 2. Commit과 Rollback

### 2.1 Commit

트랜잭션 안의 작업이 모두 성공했을 때 변경 내용을 데이터베이스에 확정한다.

```text
INSERT 성공
UPDATE 성공
이력 INSERT 성공
    ↓
Commit
```

### 2.2 Rollback

처리 중 예외가 발생했을 때 해당 트랜잭션에서 수행한 변경 내용을 취소한다.

```text
고객 UPDATE 성공
변경 이력 INSERT 실패
    ↓
Rollback
    ↓
고객 UPDATE도 취소
```

조회인 `SELECT`보다 등록·수정·삭제처럼 데이터를 변경하는 업무에서 트랜잭션이 특히 중요하다.

---

## 3. 트랜잭션의 ACID 특성

| 구분 | 의미 | 예시 |
|---|---|---|
| 원자성(Atomicity) | 작업 전체가 성공하거나 전체가 취소된다. | 고객과 이력이 함께 저장되거나 모두 취소된다. |
| 일관성(Consistency) | 처리 전후에 데이터 규칙이 유지된다. | 필수값·외래키·잔액 규칙을 위반하지 않는다. |
| 격리성(Isolation) | 동시에 실행되는 트랜잭션이 서로 부적절하게 영향을 주지 않는다. | 다른 사용자의 미완료 변경을 함부로 읽지 않는다. |
| 지속성(Durability) | Commit된 결과는 장애가 발생해도 보존되어야 한다. | 정상 저장된 고객정보가 계속 유지된다. |

처음 학습할 때는 다음 문장을 먼저 기억하면 된다.

> 트랜잭션은 하나의 업무를 구성하는 DB 변경 작업들을 한 묶음으로 처리한다.

---

## 4. Spring의 `@Transactional`

Spring에서는 일반적으로 Service 메서드에 `@Transactional`을 선언한다.

```java
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class CustomerService {

    private final CustomerMapper customerMapper;

    public CustomerService(CustomerMapper customerMapper) {
        this.customerMapper = customerMapper;
    }

    @Transactional
    public void updateCustomer(CustomerUpdateDto customer) {
        customerMapper.updateCustomer(customer);
        customerMapper.insertCustomerHistory(customer);
    }
}
```

메서드가 시작될 때 트랜잭션이 시작되고, 정상 종료되면 Commit된다. 처리 중 롤백 대상 예외가 메서드 밖으로 전달되면 Rollback된다.

```text
updateCustomer() 호출
    ↓
Transaction 시작
    ↓
고객정보 UPDATE
    ↓
변경이력 INSERT
    ↓
정상 종료
    ↓
Commit
```

---

## 5. Service에 적용하는 이유

Controller는 HTTP 요청과 응답을 담당하고, Mapper는 개별 SQL 실행을 담당한다. 하나의 업무 범위를 가장 잘 알고 있는 계층은 Service이다.

```text
Controller
└─ 요청 수신·응답 반환

Service
└─ 업무 규칙·트랜잭션 범위 결정

Mapper
└─ SQL 실행
```

고객정보 수정과 변경이력 등록을 하나의 업무로 본다면 두 작업을 호출하는 Service 메서드에 트랜잭션을 설정하는 것이 자연스럽다.

```java
@Transactional
public void updateCustomer(CustomerUpdateDto customer) {
    customerMapper.updateCustomer(customer);
    customerMapper.insertCustomerHistory(customer);
}
```

Mapper 메서드마다 별도의 트랜잭션을 만들면 전체 업무를 한 번에 롤백하기 어려워진다.

---

## 6. 다건 등록과 트랜잭션

WebSquare의 입력용 Grid에서 여러 고객을 등록하는 예이다.

```java
@Transactional
public int createCustomers(List<CustomerCreateDto> customers) {

    int insertedCount = 0;

    for (CustomerCreateDto customer : customers) {
        insertedCount += customerMapper.insertCustomer(customer);
    }

    if (insertedCount != customers.size()) {
        throw new CustomerSaveException(
            "요청 건수와 등록 건수가 일치하지 않습니다."
        );
    }

    return insertedCount;
}
```

처리 흐름은 다음과 같다.

```text
고객 목록 3건 수신
    ↓
1번 고객 등록
    ↓
2번 고객 등록
    ↓
3번 고객 등록
    ├─ 성공 → 총 3건 Commit
    └─ 실패 → 예외 발생 → 전체 Rollback
```

`insertedCount`와 요청 건수를 비교하면 SQL은 실행되었지만 일부 행이 반영되지 않은 경우도 오류로 처리할 수 있다.

---

## 7. 다건 수정·삭제에서도 필요한 이유

다건 수정과 다건 삭제도 여러 행을 하나의 업무 요청으로 처리한다.

```java
@Transactional
public int deleteCustomers(List<CustomerDeleteDto> customers) {

    int deletedCount = customerMapper.deleteCustomers(customers);

    if (deletedCount != customers.size()) {
        throw new CustomerDeleteException(
            "일부 고객정보를 삭제하지 못했습니다."
        );
    }

    return deletedCount;
}
```

예를 들어 요청은 5건인데 실제 삭제가 4건이라면 다음 가능성을 확인해야 한다.

- 이미 삭제된 고객이 포함됨
- 존재하지 않는 `customerId`가 전달됨
- 다른 업무 데이터가 고객을 참조하고 있음
- 삭제 조건과 요청 데이터가 일치하지 않음

업무 정책이 **전체 성공**이라면 예외를 발생시켜 전체 작업을 롤백한다. 반대로 일부 성공을 허용하는 업무라면 성공·실패 목록을 별도로 반환하도록 설계해야 한다.

---

## 8. 기본 롤백 기준

Spring의 기본 설정에서는 일반적으로 다음 예외가 롤백 대상이다.

| 예외 구분 | 기본 롤백 여부 |
|---|---|
| `RuntimeException`과 하위 예외 | Rollback |
| `Error`와 하위 오류 | Rollback |
| Checked Exception | 기본적으로 Commit 가능 |

따라서 업무 예외를 다음처럼 `RuntimeException`으로 정의하는 경우가 많다.

```java
public class CustomerSaveException extends RuntimeException {

    public CustomerSaveException(String message) {
        super(message);
    }
}
```

Checked Exception까지 롤백해야 한다면 명시할 수 있다.

```java
@Transactional(rollbackFor = Exception.class)
public void createCustomers(
        List<CustomerCreateDto> customers) throws Exception {

    // 등록 처리
}
```

무조건 `rollbackFor = Exception.class`를 붙이기보다는 프로젝트의 예외 설계와 업무 규칙을 먼저 정하는 것이 좋다.

---

## 9. 예외를 잡아서 없애면 안 되는 이유

다음 코드는 주의해야 한다.

```java
@Transactional
public void updateCustomer(CustomerUpdateDto customer) {

    try {
        customerMapper.updateCustomer(customer);
        customerMapper.insertCustomerHistory(customer);
    } catch (Exception e) {
        log.error("고객 수정 오류", e);
    }
}
```

예외를 `catch`한 뒤 다시 던지지 않으면 메서드는 정상 종료된 것으로 판단될 수 있다. 그러면 앞에서 성공한 SQL이 Commit될 위험이 있다.

다음처럼 예외를 다시 던진다.

```java
@Transactional
public void updateCustomer(CustomerUpdateDto customer) {

    try {
        customerMapper.updateCustomer(customer);
        customerMapper.insertCustomerHistory(customer);
    } catch (Exception e) {
        log.error("고객 수정 오류", e);
        throw new CustomerSaveException(
            "고객정보 수정 중 오류가 발생했습니다."
        );
    }
}
```

또는 Service에서는 불필요하게 잡지 않고 공통 예외 처리기로 전달할 수 있다.

```java
@Transactional
public void updateCustomer(CustomerUpdateDto customer) {
    customerMapper.updateCustomer(customer);
    customerMapper.insertCustomerHistory(customer);
}
```

---

## 10. 예외의 종류

실무에서는 예외를 성격에 따라 구분하면 처리하기 쉽다.

| 구분 | 예시 | 권장 응답 |
|---|---|---|
| 입력값 검증 오류 | 고객명 누락, 형식 오류 | `400 Bad Request` |
| 업무 규칙 오류 | 이미 삭제된 고객, 중복 연락처 | `409 Conflict` 또는 프로젝트 정책에 맞는 상태 |
| 데이터 없음 | 고객번호에 해당하는 고객 없음 | `404 Not Found` |
| 인증·권한 오류 | 로그인 만료, 수정 권한 없음 | `401 Unauthorized`, `403 Forbidden` |
| 시스템 오류 | DB 장애, 예상하지 못한 오류 | `500 Internal Server Error` |

HTTP 상태 코드는 프로젝트의 공통 API 규칙에 맞춰 통일해야 한다.

---

## 11. 업무 예외 클래스 작성

고객정보가 존재하지 않을 때 사용할 예외이다.

```java
public class CustomerNotFoundException extends RuntimeException {

    public CustomerNotFoundException(String customerId) {
        super("고객정보를 찾을 수 없습니다. customerId=" + customerId);
    }
}
```

Service에서 조회 결과를 확인한 뒤 예외를 발생시킨다.

```java
@Transactional
public void deleteCustomer(String customerId) {

    int deletedCount = customerMapper.deleteCustomer(customerId);

    if (deletedCount == 0) {
        throw new CustomerNotFoundException(customerId);
    }
}
```

이처럼 예외 이름에 오류의 의미를 담으면 어느 업무에서 왜 발생했는지 이해하기 쉽다.

---

## 12. 공통 오류 응답 DTO

화면이 모든 오류를 같은 방식으로 처리할 수 있도록 응답 구조를 통일한다.

```java
import lombok.AllArgsConstructor;
import lombok.Getter;

@Getter
@AllArgsConstructor
public class ErrorResponse {

    private boolean success;
    private String code;
    private String message;
}
```

응답 예시는 다음과 같다.

```json
{
  "success": false,
  "code": "CUSTOMER_NOT_FOUND",
  "message": "고객정보를 찾을 수 없습니다."
}
```

`code`는 화면의 분기 처리나 다국어 메시지 연결에 사용할 수 있고, `message`는 사용자 안내에 사용할 수 있다.

---

## 13. `@RestControllerAdvice` 공통 예외 처리

Controller마다 `try-catch`를 반복하지 않고 공통 예외 처리 클래스를 사용할 수 있다.

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(CustomerNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleCustomerNotFound(
            CustomerNotFoundException e) {

        ErrorResponse response = new ErrorResponse(
            false,
            "CUSTOMER_NOT_FOUND",
            "고객정보를 찾을 수 없습니다."
        );

        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(response);
    }

    @ExceptionHandler(CustomerSaveException.class)
    public ResponseEntity<ErrorResponse> handleCustomerSave(
            CustomerSaveException e) {

        ErrorResponse response = new ErrorResponse(
            false,
            "CUSTOMER_SAVE_FAILED",
            e.getMessage()
        );

        return ResponseEntity
            .status(HttpStatus.CONFLICT)
            .body(response);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleException(
            Exception e) {

        ErrorResponse response = new ErrorResponse(
            false,
            "INTERNAL_SERVER_ERROR",
            "처리 중 오류가 발생했습니다."
        );

        return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(response);
    }
}
```

`@RestControllerAdvice`는 여러 Controller에서 발생한 예외를 공통으로 처리하고 응답 본문을 반환할 때 사용한다. 구체적인 예외 처리 메서드를 먼저 정의하고 마지막에 예상하지 못한 `Exception`을 처리한다.


### 13.1 참고: HTTP 상태 코드의 5가지 분류

HTTP 상태 코드는 응답의 성격에 따라 크게 5가지로 구분된다.

| 구분 | 의미 | 대표 상태 코드 |
|---|---|---|
| 1xx | 정보 응답 | 100 Continue |
| 2xx | 요청 성공 | 200 OK, 201 Created, 204 No Content |
| 3xx | 리다이렉션 | 301 Moved Permanently, 302 Found |
| 4xx | 클라이언트 요청 오류 | 400 Bad Request, 404 Not Found, 409 Conflict |
| 5xx | 서버 오류 | 500 Internal Server Error, 503 Service Unavailable |

Spring에서는 `HttpStatus` enum을 통해 이러한 상태 코드를 사용할 수 있다.

```java
HttpStatus.OK
HttpStatus.BAD_REQUEST
HttpStatus.NOT_FOUND
HttpStatus.CONFLICT
HttpStatus.INTERNAL_SERVER_ERROR


---

## 14. Validation 예외 처리

`@Valid` 검증에 실패하면 Controller 메서드가 실행되기 전에 `MethodArgumentNotValidException`이 발생할 수 있다.

```java
import org.springframework.web.bind.MethodArgumentNotValidException;

@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<ErrorResponse> handleValidation(
        MethodArgumentNotValidException e) {

    String message = e.getBindingResult()
        .getFieldErrors()
        .stream()
        .findFirst()
        .map(fieldError -> fieldError.getDefaultMessage())
        .orElse("입력값을 확인해 주세요.");

    ErrorResponse response = new ErrorResponse(
        false,
        "VALIDATION_ERROR",
        message
    );

    return ResponseEntity
        .badRequest()
        .body(response);
}
```

요청 처리 순서는 다음과 같다.

```text
WebSquare 요청
    ↓
JSON을 DTO로 변환
    ↓
@Valid 검증
    ├─ 성공 → Controller → Service
    └─ 실패 → GlobalExceptionHandler → 400 응답
```

---

## 15. Controller 작성 예시

Controller는 예외를 직접 처리하기보다 Service를 호출하고 정상 응답을 반환하는 역할에 집중한다.

```java
@PostMapping("/create-batch")
public CustomerBatchCreateResponse createCustomers(
        @Valid
        @RequestBody
        CustomerBatchCreateRequest request) {

    int requestedCount = request.getCustomers().size();

    int insertedCount = customerService.createCustomers(
        request.getCustomers()
    );

    return new CustomerBatchCreateResponse(
        true,
        "고객정보가 일괄 등록되었습니다.",
        requestedCount,
        insertedCount
    );
}
```

```text
정상 처리
Controller → 성공 응답

예외 발생
Service → 예외 전달 → GlobalExceptionHandler → 오류 응답
```

---

## 16. WebSquare 성공 처리

서버가 정상 응답을 반환하면 `submitdone` 콜백에서 결과를 처리한다.

```javascript
scwin.sbm_createCustomers_submitdone = function(e) {

    var success = dma_createResult.get("success");
    var message = dma_createResult.get("message");

    if (success) {
        alert(message);
        scwin.search();
    }
};
```

성공 후에는 서버 데이터를 다시 조회해 화면과 DB 상태를 일치시키는 것이 좋다.

---

## 17. WebSquare 오류 처리

HTTP 오류 응답은 `submiterror` 콜백에서 처리할 수 있다. 실제 이벤트 객체와 응답 본문 접근 방법은 프로젝트의 WebSquare 버전 및 공통 Submission 모듈에 따라 다를 수 있다.

```javascript
scwin.sbm_createCustomers_submiterror = function(e) {

    var message = "처리 중 오류가 발생했습니다.";

    if (e && e.responseJSON && e.responseJSON.message) {
        message = e.responseJSON.message;
    }

    alert(message);
};
```

프로젝트에서 공통 콜백을 제공한다면 개별 화면에서 직접 파싱하기보다 공통 함수를 사용한다.

```javascript
scwin.sbm_createCustomers_submiterror = function(e) {
    com.handleSubmissionError(e);
};
```

공통 처리에서는 보통 다음 항목을 담당한다.

- HTTP 상태 코드 확인
- 서버 오류 응답 파싱
- 로그인 만료 처리
- 공통 메시지 출력
- 오류 로그 또는 추적 ID 처리

---

## 18. HTTP 오류와 업무 실패의 차이

두 가지 응답 설계가 혼용되지 않도록 프로젝트 규칙을 정해야 한다.

### 방식 1: HTTP 상태 코드로 오류 표현

```http
HTTP/1.1 409 Conflict
Content-Type: application/json
```

```json
{
  "success": false,
  "code": "DUPLICATE_PHONE",
  "message": "이미 등록된 연락처입니다."
}
```

이 경우 일반적으로 `submiterror`에서 처리한다.

### 방식 2: HTTP 200 응답 안에 업무 성공 여부 표현

```json
{
  "success": false,
  "code": "DUPLICATE_PHONE",
  "message": "이미 등록된 연락처입니다."
}
```

이 경우 `submitdone`에서 `success` 값을 확인해야 한다.

가능하면 HTTP 상태 코드와 표준 오류 응답을 활용하되, 기존 공통 Submission 모듈의 규칙이 있다면 그 규칙을 일관되게 적용한다.

---

## 19. 전체 처리 예시

### 19.1 Service

```java
@Transactional
public int createCustomers(List<CustomerCreateDto> customers) {

    int insertedCount = 0;

    for (CustomerCreateDto customer : customers) {

        int duplicateCount = customerMapper.countByPhone(
            customer.getPhone()
        );

        if (duplicateCount > 0) {
            throw new DuplicatePhoneException(
                "이미 등록된 연락처입니다."
            );
        }

        insertedCount += customerMapper.insertCustomer(customer);
    }

    if (insertedCount != customers.size()) {
        throw new CustomerSaveException(
            "일부 고객정보가 등록되지 않았습니다."
        );
    }

    return insertedCount;
}
```

### 19.2 공통 예외 처리

```java
@ExceptionHandler(DuplicatePhoneException.class)
public ResponseEntity<ErrorResponse> handleDuplicatePhone(
        DuplicatePhoneException e) {

    return ResponseEntity
        .status(HttpStatus.CONFLICT)
        .body(new ErrorResponse(
            false,
            "DUPLICATE_PHONE",
            e.getMessage()
        ));
}
```

### 19.3 WebSquare 오류 콜백

```javascript
scwin.sbm_createCustomers_submiterror = function(e) {
    com.handleSubmissionError(e);
};
```

### 19.4 전체 흐름

```text
WebSquare Grid에서 고객 3건 입력
    ↓
입력값 검증
    ↓
Submission 실행
    ↓
Controller 요청 수신
    ↓
Service @Transactional 시작
    ↓
1번 고객 INSERT 성공
    ↓
2번 고객 중복 연락처 발견
    ↓
DuplicatePhoneException 발생
    ↓
1번 고객 INSERT Rollback
    ↓
GlobalExceptionHandler
    ↓
409 + ErrorResponse 반환
    ↓
WebSquare submiterror
    ↓
"이미 등록된 연락처입니다." 안내
```

---

## 20. `@Transactional` 사용 시 주의사항

### 20.1 같은 클래스 내부 호출

Spring의 일반적인 프록시 방식에서는 같은 클래스 안에서 한 메서드가 다른 `@Transactional` 메서드를 직접 호출하면 기대한 트랜잭션 설정이 적용되지 않을 수 있다.

```java
public void execute() {
    this.saveCustomers();
}

@Transactional
public void saveCustomers() {
    // 저장 처리
}
```

트랜잭션 경계를 외부에서 호출되는 Service의 공개 메서드에 두는 방식이 이해하기 쉽다.

### 20.2 `private` 메서드

프록시 기반 트랜잭션은 일반적으로 외부에서 호출되는 메서드를 기준으로 동작한다. `private` 메서드에만 `@Transactional`을 붙이는 방식은 피한다.

### 20.3 예외를 반환값으로만 처리

다음처럼 실패 값을 반환하고 메서드를 정상 종료하면 자동 롤백되지 않는다.

```java
return false;
```

전체 롤백이 필요한 실패라면 롤백 대상 예외를 발생시키는 것이 명확하다.

### 20.4 너무 긴 트랜잭션

트랜잭션 안에서 외부 API 호출이나 오래 걸리는 파일 처리를 수행하면 DB 연결과 잠금을 오래 유지할 수 있다. 트랜잭션에는 필요한 DB 작업만 포함하도록 범위를 조정한다.

### 20.5 로그에 민감정보 출력

예외 로그에 비밀번호, 주민등록번호, 전체 연락처, 인증 토큰 등을 남기지 않는다. 사용자에게는 이해하기 쉬운 메시지를 제공하고, 서버 로그에는 원인 분석에 필요한 정보만 안전하게 기록한다.

---

## 21. 잘못된 처리와 개선 방법

| 잘못된 처리 | 문제 | 개선 방법 |
|---|---|---|
| Controller마다 `try-catch` 작성 | 중복 코드 증가 | `@RestControllerAdvice` 사용 |
| 예외를 잡고 로그만 출력 | 부분 Commit 가능 | 롤백 대상 예외를 다시 발생 |
| DB 오류 문구를 화면에 그대로 표시 | 보안·사용성 문제 | 사용자용 메시지로 변환 |
| Service 없이 Mapper를 여러 번 직접 호출 | 업무 단위 불명확 | Service에서 트랜잭션 범위 지정 |
| 오류인데 항상 HTTP 200 반환 | 성공·실패 판별 혼란 | 프로젝트의 응답 규칙 통일 |
| 오류 후 화면 데이터를 임의로 유지 | DB와 화면 불일치 | 실패 상태 안내 및 필요 시 재조회 |

---

## 22. 실무 점검 항목

- 하나의 업무에 포함되는 DB 변경 작업을 정의했는가?
- `@Transactional`을 적절한 Service 메서드에 적용했는가?
- 업무 예외가 기본 롤백 대상인지 확인했는가?
- `catch`한 예외를 무시하고 있지 않은가?
- 요청 건수와 처리 건수를 검증하는가?
- 오류 응답의 `success`, `code`, `message` 구조가 통일되어 있는가?
- Validation·업무·시스템 오류를 구분하는가?
- WebSquare의 `submitdone`과 `submiterror` 역할이 구분되어 있는가?
- 사용자 메시지에 DB 상세 오류나 민감정보가 노출되지 않는가?
- 서버 로그에는 원인 예외와 업무 식별 정보가 남는가?

---

## 23. 핵심 정리

```text
@Transactional
    = 여러 DB 작업을 하나의 업무 단위로 처리

정상 종료
    = Commit

롤백 대상 예외 발생
    = Rollback

RuntimeException
    = 기본적으로 Rollback 대상

Checked Exception
    = 기본 설정에서는 자동 Rollback 대상이 아님

@RestControllerAdvice
    = 여러 Controller의 예외를 공통 처리

submitdone
    = 정상 응답 처리

submiterror
    = HTTP 오류 응답 처리
```

트랜잭션과 예외 처리를 함께 설계하면 다음과 같은 구조가 완성된다.

```text
WebSquare 요청
    ↓
Controller
    ↓
Service @Transactional
    ↓
Mapper·DB
    ├─ 성공 → Commit → 성공 응답 → submitdone
    └─ 실패 → Rollback → 오류 응답 → submiterror
```

---

## 함께 보면 좋은 내용

- [입력값 검증]({{ '/websquare/validation/' | relative_url }})
- [공통 메시지·Submission 분석]({{ '/websquare/common/' | relative_url }})
- [WebSquare 학습 목차로 이동]({{ '/websquare/' | relative_url }})

## 참고 자료

- [Spring Framework - Using `@Transactional`](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/annotations.html)
- [Spring Framework - Rolling Back a Declarative Transaction](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/rolling-back.html)
- [Spring Framework - Controller Advice](https://docs.spring.io/spring-framework/reference/web/webmvc/mvc-controller/ann-advice.html)
