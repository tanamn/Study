---
layout: default
title: 프론트엔드 프레임워크에 따른 Spring API 설계
permalink: /spring/frontend-integration/
---

{% include navigation.html %}

# 프론트엔드 프레임워크에 따른 Spring API 설계

> WebSquare와 React 연계 방식 비교

## 1. 학습 목표

- WebSquare와 React를 사용할 때 Spring 백엔드에서 달라지는 부분을 이해한다.
- 화면 기술에 종속되지 않는 계층 구조를 설계한다.
- 단건 및 여러 건의 요청·응답 DTO 구성 방법을 익힌다.

## 2. 핵심 정리

Java Spring의 비즈니스 로직과 데이터베이스 처리 방식은 WebSquare와 React에 따라 크게 달라지지 않는다.

다만 화면과 직접 데이터를 주고받는 다음 영역은 달라질 수 있다.

- Controller의 요청·응답 형식
- 요청 및 응답 DTO 구조
- 그리드 변경 데이터 처리 방식
- 오류 응답과 화면 메시지 처리 방식
- 인증 방식과 CORS 설정

따라서 화면별 차이는 Controller와 DTO에서 흡수하고, Service와 Repository는 화면 기술에 의존하지 않도록 설계하는 것이 중요하다.

```text
WebSquare 또는 React
        ↓
Controller · DTO
        ↓
Service
        ↓
Repository · Mapper
        ↓
Database
```

## 3. 영역별 차이

| Spring 영역 | WebSquare | React | 차이 정도 |
| --- | --- | --- | --- |
| Controller | DataMap·DataList 또는 공통 통신 규격 고려 | REST API와 JSON 중심 | 있음 |
| 요청·응답 DTO | 화면 데이터 객체 이름을 포함할 수 있음 | 일반 JSON 객체·배열 중심 | 있음 |
| Service | 업무 로직 처리 | 업무 로직 처리 | 거의 없음 |
| Repository·Mapper | DB 조회 및 저장 | DB 조회 및 저장 | 없음 |
| Entity | 동일 | 동일 | 없음 |
| 트랜잭션 | 동일 | 동일 | 없음 |
| 예외 처리 | 공통 결과 코드·메시지 규격을 사용하는 경우가 많음 | HTTP 상태 코드와 JSON 오류 응답 활용 | 있음 |
| 인증 | 사내 SSO·세션 방식이 흔함 | 세션·JWT·OAuth 2.0 등 다양 | 설계에 따라 다름 |
| CORS | 화면과 서버가 동일 도메인인 경우가 많음 | 개발 서버와 API 서버가 분리되기 쉬움 | 일부 있음 |

## 4. 요청 데이터 형식

### 4.1 React의 단건 요청

React는 일반적으로 JSON 객체를 전송한다.

```json
{
  "customerId": "CUST-001",
  "phone": "010-1234-5678",
  "address": "서울시 중구",
  "job": "회사원"
}
```

Spring에서는 `@RequestBody`를 사용해 DTO로 변환한다.

```java
@PutMapping("/{customerId}")
public ResponseEntity<Void> updateCustomer(
        @PathVariable String customerId,
        @Valid @RequestBody CustomerUpdateDto dto) {

    customerService.updateCustomer(customerId, dto);
    return ResponseEntity.noContent().build();
}
```

### 4.2 WebSquare의 단건 요청

WebSquare는 화면 데이터를 `DataMap`으로 관리하는 경우가 많다. 프로젝트의 공통 통신 모듈에 따라 다음처럼 DataMap 이름이 포함된 JSON을 전송할 수 있다.

```json
{
  "dmCustomer": {
    "customerId": "CUST-001",
    "phone": "010-1234-5678",
    "address": "서울시 중구",
    "job": "회사원"
  }
}
```

이 경우 Wrapper DTO를 사용할 수 있다.

```java
@Getter
@Setter
public class CustomerRequest {

    @Valid
    @NotNull
    private CustomerUpdateDto dmCustomer;
}
```

```java
@PutMapping("/{customerId}")
public ResponseEntity<Void> updateCustomer(
        @PathVariable String customerId,
        @Valid @RequestBody CustomerRequest request) {

    customerService.updateCustomer(
            customerId,
            request.getDmCustomer()
    );

    return ResponseEntity.noContent().build();
}
```

WebSquare에서도 표준 JSON을 직접 전송하도록 구성하면 React와 같은 DTO와 API를 사용할 수 있다.

## 5. 여러 건의 데이터 처리

### 5.1 React의 배열 요청

```json
[
  {
    "name": "홍길동",
    "phone": "010-1111-2222"
  },
  {
    "name": "김영희",
    "phone": "010-3333-4444"
  }
]
```

```java
@PostMapping("/batch")
public ResponseEntity<Void> createCustomers(
        @Valid @RequestBody List<CustomerCreateDto> customers) {

    customerService.createCustomers(customers);
    return ResponseEntity.ok().build();
}
```

### 5.2 WebSquare의 DataList 요청

```json
{
  "dlCustomer": [
    {
      "name": "홍길동",
      "phone": "010-1111-2222"
    },
    {
      "name": "김영희",
      "phone": "010-3333-4444"
    }
  ]
}
```

```java
@Getter
@Setter
public class CustomerBatchRequest {

    @Valid
    @NotEmpty
    private List<CustomerCreateDto> dlCustomer;
}
```

```java
@PostMapping("/batch")
public ResponseEntity<Void> createCustomers(
        @Valid @RequestBody CustomerBatchRequest request) {

    customerService.createCustomers(request.getDlCustomer());
    return ResponseEntity.ok().build();
}
```

## 6. 그리드의 변경 행 처리

WebSquare의 DataList는 신규·수정·삭제와 같은 행 상태를 관리할 수 있다. 프로젝트에 따라 행 상태를 서버로 함께 전달한다.

```json
{
  "dlCustomer": [
    {
      "rowStatus": "C",
      "name": "홍길동",
      "phone": "010-1111-2222"
    },
    {
      "rowStatus": "U",
      "customerId": "CUST-002",
      "phone": "010-3333-4444"
    },
    {
      "rowStatus": "D",
      "customerId": "CUST-003"
    }
  ]
}
```

```java
@Transactional
public void saveCustomers(List<CustomerSaveDto> customers) {
    for (CustomerSaveDto customer : customers) {
        switch (customer.getRowStatus()) {
            case "C" -> createCustomer(customer);
            case "U" -> updateCustomer(customer);
            case "D" -> deleteCustomer(customer);
            default -> throw new IllegalArgumentException(
                    "유효하지 않은 행 상태입니다."
            );
        }
    }
}
```

React에는 WebSquare DataList와 같은 행 상태 관리 규칙이 기본 제공되지 않는다. 화면에서 변경 상태를 직접 관리하거나 생성·수정·삭제 API를 분리할 수 있다.

```json
{
  "created": [],
  "updated": [],
  "deletedIds": []
}
```

```text
POST   /api/customers
PUT    /api/customers/{id}
DELETE /api/customers/{id}
```

React에서도 대량 편집형 그리드를 사용한다면 WebSquare처럼 행 상태를 전달하도록 설계할 수 있다.

## 7. 응답 구조

### 7.1 React에서 사용하는 일반적인 응답

React는 HTTP 상태 코드와 JSON 응답을 직접 처리하기 쉽다.

```json
{
  "id": "7ff3d547-95fe-473b-9902-fb31519e08dd",
  "customerNo": "CUST-20260904-000001",
  "name": "홍길동"
}
```

목록과 페이징 정보는 다음처럼 전달할 수 있다.

```json
{
  "content": [
    {
      "customerNo": "CUST-001",
      "name": "홍길동"
    }
  ],
  "page": 0,
  "size": 20,
  "totalElements": 125
}
```

### 7.2 WebSquare에서 사용하는 응답

WebSquare에서는 응답 데이터를 화면의 DataMap·DataList와 연결하기 위해 데이터 객체 이름을 포함하기도 한다.

```json
{
  "dmResult": {
    "resultCode": "SUCCESS",
    "resultMessage": "정상 처리되었습니다."
  },
  "dlCustomer": [
    {
      "customerId": "CUST-001",
      "name": "홍길동"
    }
  ]
}
```

```java
@Getter
@AllArgsConstructor
public class CustomerListResponse {

    private ResultDto dmResult;
    private List<CustomerResponseDto> dlCustomer;
}
```

이 구조는 WebSquare의 필수 규칙이 아니라 프로젝트의 공통 통신 모듈 규칙에 따라 결정된다.

## 8. 예외 처리

Service에서 발생시키는 업무 예외는 화면 기술과 관계없이 동일하게 유지한다.

```java
public void updateCustomer(String customerId, CustomerUpdateDto dto) {
    Customer customer = customerRepository.findById(customerId)
            .orElseThrow(() -> new CustomerNotFoundException(customerId));

    customer.update(dto.getPhone(), dto.getAddress(), dto.getJob());
}
```

React API에서는 HTTP 상태 코드와 표준 오류 응답을 적극적으로 사용할 수 있다.

```json
{
  "code": "CUSTOMER_NOT_FOUND",
  "message": "고객을 찾을 수 없습니다."
}
```

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(CustomerNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleCustomerNotFound(
            CustomerNotFoundException e) {

        ErrorResponse response = new ErrorResponse(
                "CUSTOMER_NOT_FOUND",
                e.getMessage()
        );

        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(response);
    }
}
```

WebSquare 프로젝트에서는 HTTP 응답 상태는 정상으로 유지하고 내부 결과 코드로 성공과 실패를 판단하는 경우도 있다.

```json
{
  "dmResult": {
    "resultCode": "E001",
    "resultMessage": "고객을 찾을 수 없습니다."
  }
}
```

가능하다면 HTTP 상태 코드와 표준 오류 응답을 사용하되, 기존 공통 모듈의 처리 규칙을 먼저 확인해야 한다.

## 9. 인증과 보안

화면 프레임워크가 인증 방식을 직접 결정하는 것은 아니다. 다만 프로젝트 성격에 따라 다음 방식이 자주 사용된다.

| 구분 | 자주 사용하는 방식 |
| --- | --- |
| WebSquare 기반 내부 업무 시스템 | 서버 세션, 사내 SSO |
| React 기반 웹 애플리케이션 | 세션 쿠키, JWT, OAuth 2.0·OIDC |

React라고 해서 반드시 JWT를 사용해야 하는 것은 아니다. 브라우저 기반 업무 시스템에서는 `HttpOnly`, `Secure`, `SameSite`가 설정된 세션 쿠키가 더 적합할 수 있다.

Spring Security 설정은 WebSquare 또는 React가 아니라 프로젝트에서 선택한 인증 방식에 따라 달라진다.

## 10. CORS와 배포 구조

WebSquare 화면과 Spring 서버는 같은 도메인에 배포되는 경우가 많다.

```text
https://company.com/websquare
https://company.com/api
```

React 개발 환경에서는 프런트엔드 개발 서버와 Spring 서버가 분리되기 쉽다.

```text
React  : http://localhost:3000
Spring : http://localhost:8080
```

출처가 서로 다르면 Spring CORS 설정이나 React 개발 프록시 설정이 필요하다. 운영 환경에서 리버스 프록시로 같은 도메인을 사용하면 CORS 구성을 단순화할 수 있다.

## 11. 권장 설계 원칙

1. WebSquare의 `DataMap`, `DataList`를 Service의 매개변수로 사용하지 않는다.
2. 화면 전용 요청 구조는 Controller DTO에서 업무 DTO로 변환한다.
3. Service는 WebSquare와 React 중 어느 화면에서 호출했는지 알지 못하게 한다.
4. Entity를 화면에 직접 반환하지 않고 응답 DTO를 사용한다.
5. 성공 및 오류 응답 형식을 프로젝트 공통 규칙으로 통일한다.
6. 두 화면이 같은 API를 사용해야 한다면 표준 JSON을 기준으로 설계한다.
7. 내부 식별용 UUID와 화면에 표시할 업무번호를 구분한다.

## 12. 최종 정리

WebSquare와 React에 따라 Spring 전체 구조를 다르게 만들 필요는 없다.

차이는 주로 화면과 맞닿는 Controller·DTO·API 응답 영역에서 발생한다. 이 차이를 API 경계에서 처리하면 Service, Repository, Mapper 및 데이터베이스 로직을 공통으로 유지할 수 있다.
