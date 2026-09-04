---
layout: default
title: WebSquare 신규 고객 등록 학습 정리
permalink: /websquare/create/
---

{% include navigation.html %}

# WebSquare 신규 고객 등록 학습 정리

> 신규 버튼으로 입력 화면을 초기화하고, 고객정보를 입력한 뒤 등록 Submission을 실행하여 Spring·MyBatis의 `INSERT` 처리까지 연결하는 흐름을 정리한 문서입니다.

이 문서는 [WebSquare 학습 정리]({{ '/websquare/' | relative_url }})의 **신규 고객 등록** 단계입니다.

---

## 1. 이번 학습의 목표

```text
신규 버튼 클릭
    ↓
등록용 DataMap 초기화
    ↓
고객정보 입력
    ↓
화면 입력값 검증
    ↓
등록 Submission 실행
    ↓
Spring Controller
    ↓
Service
    ↓
MyBatis INSERT
    ↓
등록 결과 반환
    ↓
목록 재조회
```

이번 학습에서 다루는 핵심은 다음과 같습니다.

- 신규 입력 화면 초기화
- 등록용 DataMap 구성
- 필수값과 연락처 형식 검증
- 등록 Submission 실행
- Spring의 JSON 요청 DTO 변환
- Service의 트랜잭션 처리
- MyBatis `INSERT`
- 등록 완료 후 목록 재조회

---

## 2. 신규 등록과 수정의 차이

| 구분 | 신규 등록 | 상세정보 수정 |
|---|---|---|
| 목적 | 새로운 고객 생성 | 기존 고객정보 변경 |
| SQL | `INSERT` | `UPDATE` |
| 고객번호 | 서버 또는 DB에서 생성 | 기존 고객번호 사용 |
| 화면 초기값 | 대부분 빈 값 | 상세조회 결과 |
| API 예시 | `/api/customer/create` | `/api/customer/update` |
| 성공 후 처리 | 신규 고객 목록 반영 | 변경된 상세·목록 반영 |

신규 등록에서는 고객번호를 화면에서 임의로 만드는 것보다 서버나 DB에서 생성하는 방식이 일반적입니다.

---

## 3. 사용할 WebSquare 객체

| 종류 | ID | 역할 |
|---|---|---|
| DataMap | `dma_customerForm` | 신규 고객 입력값 |
| DataMap | `dma_createResult` | 등록 처리 결과 |
| Submission | `sbm_createCustomer` | 신규 고객 등록 요청 |
| Button | `btn_new` | 입력 화면 초기화 |
| Button | `btn_create` | 등록 요청 실행 |
| GridView | `grd_customer` | 등록 후 고객목록 표시 |

등록용 DataMap은 다음 필드를 가진다고 가정합니다.

```text
dma_customerForm
├─ customerName
├─ birthDate
├─ phone
├─ address
└─ job
```

고객번호 `customerId`는 화면에서 보내지 않고 서버 또는 DB에서 생성합니다.

---

## 4. 입력 컴포넌트와 DataMap 바인딩

```text
ibx_customerName ↔ dma_customerForm.customerName
cal_birthDate    ↔ dma_customerForm.birthDate
ibx_phone        ↔ dma_customerForm.phone
ibx_address      ↔ dma_customerForm.address
ibx_job          ↔ dma_customerForm.job
```

사용자가 값을 입력하면 바인딩된 DataMap에도 값이 반영됩니다.

```javascript
{
    customerName: "김철수",
    birthDate: "1990-04-15",
    phone: "010-1234-5678",
    address: "서울특별시 종로구",
    job: "회사원"
}
```

---

## 5. 신규 버튼 처리

신규 버튼을 클릭하면 기존 상세조회 값이 남지 않도록 등록 폼을 초기화합니다.

```javascript
scwin.btn_new_onclick = function(e) {
    scwin.clearCustomerForm();
    ibx_customerName.focus();
};
```

초기화 함수는 다음처럼 구성할 수 있습니다.

```javascript
scwin.clearCustomerForm = function() {
    dma_customerForm.set("customerName", "");
    dma_customerForm.set("birthDate", "");
    dma_customerForm.set("phone", "");
    dma_customerForm.set("address", "");
    dma_customerForm.set("job", "");
};
```

프로젝트에서 DataMap 전체 초기화 공통함수를 제공한다면 해당 공통함수를 우선 사용합니다.

```javascript
// 프로젝트별 예시이며 실제 함수명은 다를 수 있습니다.
com.data.clear(dma_customerForm);
```

---

## 6. 화면 입력값 검증

### 필수값 검증

```javascript
scwin.validateCustomerForm = function() {
    var customerName = dma_customerForm.get("customerName");
    var phone = dma_customerForm.get("phone");
    var address = dma_customerForm.get("address");

    if (!customerName || customerName.trim() === "") {
        alert("고객명을 입력해주세요.");
        ibx_customerName.focus();
        return false;
    }

    if (!phone || phone.trim() === "") {
        alert("연락처를 입력해주세요.");
        ibx_phone.focus();
        return false;
    }

    if (!address || address.trim() === "") {
        alert("주소를 입력해주세요.");
        ibx_address.focus();
        return false;
    }

    if (!scwin.isValidPhone(phone)) {
        alert("연락처 형식을 확인해주세요.");
        ibx_phone.focus();
        return false;
    }

    return true;
};
```

### 연락처 형식 검증

```javascript
scwin.isValidPhone = function(phone) {
    var phonePattern = /^01[016789]-\d{3,4}-\d{4}$/;
    return phonePattern.test(phone);
};
```

예시 결과:

| 연락처 | 결과 |
|---|---:|
| `010-1234-5678` | 정상 |
| `011-123-4567` | 정상 |
| `01012345678` | 오류 |
| `02-1234-5678` | 오류 |

화면 검증은 사용자 편의를 위한 1차 검증입니다. 요청 데이터는 조작될 수 있으므로 서버에서도 반드시 다시 검증해야 합니다.

---

## 7. 등록 버튼 이벤트

```javascript
scwin.btn_create_onclick = function(e) {
    if (!scwin.validateCustomerForm()) {
        return;
    }

    if (!confirm("신규 고객을 등록하시겠습니까?")) {
        return;
    }

    btn_create.setDisabled(true);
    $p.executeSubmission(sbm_createCustomer);
};
```

처리 순서는 다음과 같습니다.

```text
등록 버튼 클릭
    ↓
입력값 정상?
    ├─ 아니요 → 메시지 표시 후 종료
    └─ 예
        ↓
등록 확인
    ├─ 취소 → 종료
    └─ 확인
        ↓
중복 클릭 방지
        ↓
Submission 실행
```

---

## 8. 등록 Submission 설정

```text
ID:        sbm_createCustomer
Action:    /api/customer/create
Method:    POST
Reference: dma_customerForm
Target:    dma_createResult
Mode:      asynchronous
```

방향을 정리하면 다음과 같습니다.

```text
dma_customerForm
    ↓ Reference
sbm_createCustomer
    ↓ POST /api/customer/create
Spring Controller
    ↓ JSON 응답
dma_createResult
```

서버로 전달되는 JSON 예시는 다음과 같습니다.

```json
{
    "customerName": "김철수",
    "birthDate": "1990-04-15",
    "phone": "010-1234-5678",
    "address": "서울특별시 종로구",
    "job": "회사원"
}
```

---

## 9. 등록 요청 DTO

이 코드는 WebSquare Script가 아니라 Spring 백엔드의 Java 코드입니다.

```java
@Getter
@Setter
public class CustomerCreateDto {

    @NotBlank(message = "고객명은 필수입니다.")
    private String customerName;

    @NotNull(message = "생년월일은 필수입니다.")
    private LocalDate birthDate;

    @NotBlank(message = "연락처는 필수입니다.")
    @Pattern(
        regexp = "^01[016789]-\\d{3,4}-\\d{4}$",
        message = "연락처 형식이 올바르지 않습니다."
    )
    private String phone;

    @NotBlank(message = "주소는 필수입니다.")
    private String address;

    private String job;
}
```

Java 문자열에서는 정규식의 역슬래시를 한 번 더 이스케이프해야 합니다.

```text
JavaScript 정규식: \d
Java 문자열:        \\d
```

---

## 10. Jackson의 DTO 변환

Spring의 `@RequestBody`는 요청 JSON을 Java DTO로 변환합니다. 이때 일반적으로 Jackson이 사용됩니다.

```text
WebSquare DataMap
    ↓ Submission
JSON 요청
    ↓ Jackson 역직렬화
CustomerCreateDto
```

JSON 속성명과 DTO 필드명이 일치해야 자연스럽게 연결됩니다.

| JSON | DTO |
|---|---|
| `customerName` | `customerName` |
| `birthDate` | `birthDate` |
| `phone` | `phone` |
| `address` | `address` |
| `job` | `job` |

---

## 11. Spring Controller

```java
@RestController
@RequestMapping("/api/customer")
@RequiredArgsConstructor
public class CustomerController {

    private final CustomerService customerService;

    @PostMapping("/create")
    public CustomerCreateResponse createCustomer(
            @Valid @RequestBody CustomerCreateDto createDto) {

        int insertedCount =
            customerService.createCustomer(createDto);

        return new CustomerCreateResponse(
            true,
            "신규 고객이 등록되었습니다.",
            insertedCount
        );
    }
}
```

최종 호출 주소는 두 애너테이션이 합쳐져 만들어집니다.

```text
@RequestMapping("/api/customer")
@PostMapping("/create")
              ↓
/api/customer/create
```

이 주소는 WebSquare Submission의 Action과 일치해야 합니다.

---

## 12. 등록 결과 DTO

```java
@Getter
@AllArgsConstructor
public class CustomerCreateResponse {

    private boolean success;
    private String message;
    private int insertedCount;
}
```

Controller가 객체를 반환하면 Jackson이 JSON으로 직렬화합니다.

```json
{
    "success": true,
    "message": "신규 고객이 등록되었습니다.",
    "insertedCount": 1
}
```

이 결과는 `dma_createResult`에 저장됩니다.

---

## 13. Service와 트랜잭션

```java
@Service
@RequiredArgsConstructor
public class CustomerService {

    private final CustomerMapper customerMapper;

    @Transactional
    public int createCustomer(CustomerCreateDto createDto) {
        return customerMapper.insertCustomer(createDto);
    }
}
```

`@Transactional`은 등록 도중 예외가 발생하면 DB 작업을 롤백하는 역할을 합니다.

```text
INSERT 성공 → Commit
INSERT 실패 → Rollback
```

등록 작업이 한 건뿐이어도 Service 계층을 트랜잭션 경계로 두면 이후 연관 데이터 등록이 추가될 때 관리하기 좋습니다.

---

## 14. MyBatis Mapper 인터페이스

```java
@Mapper
public interface CustomerMapper {

    int insertCustomer(CustomerCreateDto createDto);
}
```

반환값은 INSERT로 영향을 받은 행의 수입니다.

```text
1 → 한 건 등록 성공
0 → 등록되지 않음
```

---

## 15. MyBatis INSERT

Oracle 또는 Tibero에서 Sequence로 고객번호를 생성하는 예시입니다.

```xml
<insert id="insertCustomer"
        parameterType="CustomerCreateDto">

    INSERT INTO CUSTOMER (
          CUSTOMER_ID
        , CUSTOMER_NAME
        , BIRTH_DATE
        , PHONE
        , ADDRESS
        , JOB
        , CREATED_AT
    ) VALUES (
          'C' || LPAD(SEQ_CUSTOMER.NEXTVAL, 6, '0')
        , #{customerName}
        , #{birthDate}
        , #{phone}
        , #{address}
        , #{job}
        , SYSDATE
    )

</insert>
```

예를 들어 Sequence 값이 `15`라면 고객번호는 다음처럼 만들어집니다.

```text
C000015
```

고객번호 생성 규칙은 프로젝트 표준에 따라 UUID, Sequence 또는 별도의 채번 서비스를 사용할 수 있습니다.

---

## 16. 등록 완료 이벤트

```javascript
scwin.sbm_createCustomer_submitdone = function(e) {
    btn_create.setDisabled(false);

    var success = dma_createResult.get("success");
    var message = dma_createResult.get("message");

    if (!success) {
        alert(message || "고객을 등록하지 못했습니다.");
        return;
    }

    alert(message);
    scwin.clearCustomerForm();
    $p.executeSubmission(sbm_searchCustomer);
};
```

등록에 성공하면 다음 작업을 수행합니다.

1. 등록 버튼을 다시 활성화합니다.
2. 성공 메시지를 표시합니다.
3. 입력 폼을 초기화합니다.
4. 고객목록을 다시 조회합니다.

프로젝트에서 등록된 고객의 상세화면으로 이동해야 한다면 서버가 생성한 `customerId`를 응답에 포함하도록 응답 DTO를 확장할 수 있습니다.

---

## 17. 등록 오류 이벤트

```javascript
scwin.sbm_createCustomer_submiterror = function(e) {
    btn_create.setDisabled(false);
    alert("신규 고객 등록 중 오류가 발생했습니다.");
};
```

성공과 오류 처리 모두에서 버튼을 다시 활성화해야 오류 발생 후 등록 버튼이 계속 비활성화되는 문제를 방지할 수 있습니다.

---

## 18. 중복 고객 확인

고객명만으로는 동명이인을 구분할 수 없으므로 이름 하나만으로 중복을 판단하면 안 됩니다.

프로젝트의 업무 규칙에 따라 다음 항목을 조합할 수 있습니다.

- 고객 식별번호
- 휴대전화 번호
- 생년월일
- 외부 고객번호

예를 들어 연락처 중복 확인이 필요하다면 Service에서 등록 전에 조회합니다.

```java
@Transactional
public int createCustomer(CustomerCreateDto createDto) {
    if (customerMapper.existsByPhone(createDto.getPhone()) > 0) {
        throw new DuplicateCustomerException(
            "이미 등록된 연락처입니다."
        );
    }

    return customerMapper.insertCustomer(createDto);
}
```

동시 요청까지 안전하게 막으려면 애플리케이션의 사전 조회뿐 아니라 DB의 Unique 제약조건도 함께 검토해야 합니다.

---

## 19. 서버 검증 오류 처리

`@Valid` 검증에 실패하면 Controller 메서드에 진입하기 전에 예외가 발생합니다.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(
            MethodArgumentNotValidException e) {

        String message = e.getBindingResult()
            .getFieldErrors()
            .get(0)
            .getDefaultMessage();

        return ResponseEntity.badRequest()
            .body(new ErrorResponse(false, message));
    }
}
```

처리 흐름은 다음과 같습니다.

```text
요청 JSON
    ↓
Jackson DTO 변환
    ↓
@Valid 검증
    ├─ 정상 → Controller 실행
    └─ 오류 → 예외 처리기에서 오류 응답
```

---

## 20. 전체 소스 흐름

```text
[WebSquare]

btn_new
    ↓
dma_customerForm 초기화
    ↓
고객정보 입력
    ↓ Binding
dma_customerForm
    ↓
btn_create
    ↓
validateCustomerForm()
    ↓
sbm_createCustomer
    ↓ POST /api/customer/create

[Spring]

CustomerController
    ↓
CustomerCreateDto
    ↓
CustomerService @Transactional
    ↓
CustomerMapper
    ↓
INSERT INTO CUSTOMER
    ↓
CustomerCreateResponse

[WebSquare]

dma_createResult
    ↓
submitdone
    ↓
메시지 표시·폼 초기화·목록 재조회
```

---

## 21. 파일별 작성 위치

```text
WebSquare 화면
├─ DataMap
│  ├─ dma_customerForm
│  └─ dma_createResult
├─ Submission
│  └─ sbm_createCustomer
└─ Script
   ├─ btn_new_onclick
   ├─ btn_create_onclick
   ├─ validateCustomerForm
   ├─ sbm_createCustomer_submitdone
   └─ sbm_createCustomer_submiterror

Spring 프로젝트
└─ src/main/java
   └─ customer
      ├─ controller/CustomerController.java
      ├─ dto/CustomerCreateDto.java
      ├─ dto/CustomerCreateResponse.java
      ├─ service/CustomerService.java
      └─ mapper/CustomerMapper.java

MyBatis
└─ src/main/resources
   └─ mapper/customer/CustomerMapper.xml
```

---

## 22. 실무 확인 항목

| 확인 항목 | 점검 내용 |
|---|---|
| 초기화 | 이전 고객의 상세정보가 남아 있지 않은가? |
| 필수값 | 고객명·연락처·주소가 비어 있을 때 등록되지 않는가? |
| 형식 | 연락처와 날짜 형식이 올바른가? |
| 중복 실행 | 등록 버튼을 연속 클릭해도 중복 등록되지 않는가? |
| 서버 검증 | 화면 검증을 우회한 요청도 서버에서 차단되는가? |
| 고객번호 | 중복되지 않는 번호가 생성되는가? |
| 트랜잭션 | 오류가 발생하면 등록 내용이 롤백되는가? |
| 결과 처리 | 등록 성공·실패 메시지가 적절한가? |
| 재조회 | 등록된 고객이 목록에 표시되는가? |
| 개인정보 | 로그에 연락처 등 개인정보가 불필요하게 남지 않는가? |

---

## 23. 핵심 코드 요약

### 신규 폼 초기화

```javascript
scwin.clearCustomerForm();
```

### 등록 실행

```javascript
$p.executeSubmission(sbm_createCustomer);
```

### Controller 요청 수신

```java
@PostMapping("/create")
public CustomerCreateResponse createCustomer(
        @Valid @RequestBody CustomerCreateDto createDto) {
    // 등록 처리
}
```

### Service 트랜잭션

```java
@Transactional
public int createCustomer(CustomerCreateDto createDto) {
    return customerMapper.insertCustomer(createDto);
}
```

### DB 등록

```xml
INSERT INTO CUSTOMER (...) VALUES (...)
```

한 줄로 정리하면 다음과 같습니다.

```text
폼 초기화 → 입력·검증 → 등록 Submission → @Valid → @Transactional → INSERT → 목록 재조회
```

---

[← 이전 학습: 상세정보 수정]({{ '/websquare/edit/' | relative_url }}) · [WebSquare 목차]({{ '/websquare/' | relative_url }}) · [다음 학습: 고객 삭제 →]({{ '/websquare/delete/' | relative_url }})
