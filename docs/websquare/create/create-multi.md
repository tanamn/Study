---
layout: default
title: WebSquare 입력 Grid 다건 등록
permalink: /websquare/create/multi/
---

{% include navigation.html %}

# WebSquare 입력 Grid 다건 등록

입력용 Grid에서 여러 고객정보를 작성하고, DataList와 다건 등록
Submission을 이용해 Spring·MyBatis에서 일괄 처리하는 방법을 학습합니다.

### 1. 전체 처리 흐름

```text
행 추가 버튼
    ↓
입력용 DataList에 빈 행 추가
    ↓
Grid에서 여러 고객정보 입력
    ↓
전체 행 검증
    ↓
다건 등록 Submission 실행
    ↓
Spring에서 List<DTO> 수신
    ↓
Service @Transactional
    ↓
고객별 INSERT
    ↓
전체 성공 또는 전체 롤백
    ↓
결과 표시·목록 재조회
```

### 2. 단건 등록과 다건 등록 비교

| 구분 | 단건 등록 | Grid 다건 등록 |
|---|---|---|
| 입력 객체 | DataMap | DataList |
| 요청 JSON | 객체 `{}` | 배열 `[]` |
| Controller 입력 | `CustomerCreateDto` | `List<CustomerCreateDto>` |
| 검증 | 입력값 한 건 검증 | 모든 행을 순회하며 검증 |
| DB 처리 | `INSERT` 1회 | `INSERT` 여러 회 또는 Batch |
| 트랜잭션 | 한 건 성공·실패 | 전체 성공·전체 롤백 정책 권장 |

### 3. 입력용 DataList와 Grid

```text
DataList ID: dlt_customerInput

컬럼:
- customerName
- birthDate
- phone
- address
- job
```

입력용 Grid를 DataList에 바인딩합니다.

```text
dlt_customerInput
        ↕ Binding
grd_customerInput
```

화면 예시는 다음과 같습니다.

| No | 고객명 | 생년월일 | 연락처 | 주소 | 직업 |
|---:|---|---|---|---|---|
| 1 | 김철수 | 1990-04-15 | 010-1111-2222 | 서울시 종로구 | 회사원 |
| 2 | 이영희 | 1988-07-20 | 010-3333-4444 | 서울시 마포구 | 자영업 |
| 3 | 박민수 | 1995-10-02 | 010-5555-6666 | 인천시 남동구 | 개발자 |

Grid의 고객명·생년월일·연락처·주소·직업 컬럼은 사용자가 값을 입력할 수 있도록 편집 가능 상태로 설정합니다.

### 4. 행 추가

```javascript
scwin.btn_addRow_onclick = function(e) {
    var rowIndex = dlt_customerInput.addRow();

    dlt_customerInput.setCellData(rowIndex, "customerName", "");
    dlt_customerInput.setCellData(rowIndex, "birthDate", "");
    dlt_customerInput.setCellData(rowIndex, "phone", "");
    dlt_customerInput.setCellData(rowIndex, "address", "");
    dlt_customerInput.setCellData(rowIndex, "job", "");
};
```

`addRow()`는 DataList에 신규 행을 추가합니다. Grid는 DataList와 바인딩되어 있으므로 추가된 행이 화면에도 표시됩니다.

프로젝트의 WebSquare 버전과 공통 라이브러리에 따라 행 추가 API가 감싸져 있을 수 있으므로 기존 입력 Grid 화면도 함께 확인합니다.

### 5. 선택 행 삭제

```javascript
scwin.btn_deleteRow_onclick = function(e) {
    var rowIndex = grd_customerInput.getFocusedRowIndex();

    if (rowIndex < 0) {
        alert("삭제할 행을 선택해주세요.");
        return;
    }

    dlt_customerInput.deleteRow(rowIndex);
};
```

Grid에서 선택한 행 번호를 가져온 뒤 실제 데이터를 보유한 DataList에서 삭제합니다.

```text
Grid = 화면
DataList = 실제 입력 데이터
```

행 선택 API와 삭제 방식은 프로젝트별 Grid 설정에 따라 다를 수 있습니다.

### 6. 전체 행 검증

```javascript
scwin.validateCustomerRows = function() {
    var rowCount = dlt_customerInput.getRowCount();

    if (rowCount === 0) {
        alert("등록할 고객을 한 건 이상 입력해주세요.");
        return false;
    }

    for (var rowIndex = 0; rowIndex < rowCount; rowIndex++) {
        var customerName = dlt_customerInput.getCellData(
            rowIndex,
            "customerName"
        );
        var phone = dlt_customerInput.getCellData(
            rowIndex,
            "phone"
        );
        var address = dlt_customerInput.getCellData(
            rowIndex,
            "address"
        );

        if (!customerName || customerName.trim() === "") {
            alert((rowIndex + 1) + "행의 고객명을 입력해주세요.");
            grd_customerInput.setFocusedCell(rowIndex, "customerName");
            return false;
        }

        if (!phone || phone.trim() === "") {
            alert((rowIndex + 1) + "행의 연락처를 입력해주세요.");
            grd_customerInput.setFocusedCell(rowIndex, "phone");
            return false;
        }

        if (!scwin.isValidPhone(phone)) {
            alert((rowIndex + 1) + "행의 연락처 형식을 확인해주세요.");
            grd_customerInput.setFocusedCell(rowIndex, "phone");
            return false;
        }

        if (!address || address.trim() === "") {
            alert((rowIndex + 1) + "행의 주소를 입력해주세요.");
            grd_customerInput.setFocusedCell(rowIndex, "address");
            return false;
        }
    }

    return true;
};
```

검증 오류 메시지에는 몇 번째 행에서 오류가 발생했는지 포함하는 것이 좋습니다.

```text
3행의 연락처 형식을 확인해주세요.
```

`setFocusedCell()`의 인자 형식은 프로젝트에서 사용하는 WebSquare 버전에 따라 다를 수 있으므로 실제 Grid API를 확인해야 합니다.

### 7. Grid 내부 중복 검사

동일한 연락처가 여러 행에 입력됐는지 등록 전에 확인할 수 있습니다.

```javascript
scwin.hasDuplicatePhone = function() {
    var phoneMap = {};
    var rowCount = dlt_customerInput.getRowCount();

    for (var rowIndex = 0; rowIndex < rowCount; rowIndex++) {
        var phone = dlt_customerInput.getCellData(
            rowIndex,
            "phone"
        );

        if (phoneMap[phone]) {
            alert((rowIndex + 1) + "행에 중복된 연락처가 있습니다.");
            return true;
        }

        phoneMap[phone] = true;
    }

    return false;
};
```

화면의 중복 검사는 사용자 실수를 빠르게 알려주기 위한 것입니다. 최종 중복 여부는 서버와 DB에서도 검사해야 합니다.

### 8. 일괄등록 버튼 이벤트

```javascript
scwin.btn_createBatch_onclick = function(e) {
    if (!scwin.validateCustomerRows()) {
        return;
    }

    if (scwin.hasDuplicatePhone()) {
        return;
    }

    var rowCount = dlt_customerInput.getRowCount();

    if (!confirm(rowCount + "명의 고객을 등록하시겠습니까?")) {
        return;
    }

    btn_createBatch.setDisabled(true);
    $p.executeSubmission(sbm_createCustomers);
};
```

### 9. 다건 등록 Submission

```text
ID:        sbm_createCustomers
Action:    /api/customer/create-batch
Method:    POST
Reference: dlt_customerInput
Target:    dma_batchCreateResult
Mode:      asynchronous
```

요청 JSON은 다음과 같은 배열 형태라고 가정합니다.

```json
[
    {
        "customerName": "김철수",
        "birthDate": "1990-04-15",
        "phone": "010-1111-2222",
        "address": "서울시 종로구",
        "job": "회사원"
    },
    {
        "customerName": "이영희",
        "birthDate": "1988-07-20",
        "phone": "010-3333-4444",
        "address": "서울시 마포구",
        "job": "자영업"
    }
]
```

WebSquare Submission이 만드는 실제 JSON 구조는 프로젝트의 전송 설정과 공통 통신 모듈에 따라 달라질 수 있습니다. 서버 개발 전에 브라우저 개발자 도구의 Network 탭에서 요청 본문을 확인하고 Controller의 DTO 구조를 맞춰야 합니다.

### 10. Spring Controller에서 `List<DTO>` 수신

요청 본문이 JSON 배열이라면 Controller에서 리스트로 받을 수 있습니다.

```java
@PostMapping("/create-batch")
public CustomerBatchCreateResponse createCustomers(
        @RequestBody
        @NotEmpty(message = "등록할 고객이 없습니다.")
        List<@Valid CustomerCreateDto> customers) {

    int insertedCount =
        customerService.createCustomers(customers);

    return new CustomerBatchCreateResponse(
        true,
        "고객정보가 일괄 등록되었습니다.",
        customers.size(),
        insertedCount
    );
}
```

각 요소의 `@NotBlank`, `@Pattern` 등을 검사하려면 리스트의 요소 타입에 `@Valid`를 적용합니다.

```java
List<@Valid CustomerCreateDto> customers
```

프로젝트의 Spring 및 Bean Validation 버전에 따라 메서드 파라미터 검증을 위한 추가 설정이나 `@Validated`가 필요할 수 있습니다.

### 11. 요청 Wrapper를 사용하는 방식

프로젝트에서 요청 본문을 객체 형태로 통일한다면 Wrapper DTO를 사용할 수 있습니다.

```json
{
    "customers": [
        {
            "customerName": "김철수",
            "phone": "010-1111-2222",
            "address": "서울시 종로구"
        }
    ]
}
```

```java
@Getter
@Setter
public class CustomerBatchCreateRequest {

    @NotEmpty(message = "등록할 고객이 없습니다.")
    private List<@Valid CustomerCreateDto> customers;
}
```

```java
@PostMapping("/create-batch")
public CustomerBatchCreateResponse createCustomers(
        @Valid @RequestBody CustomerBatchCreateRequest request) {

    List<CustomerCreateDto> customers = request.getCustomers();
    int insertedCount = customerService.createCustomers(customers);

    return new CustomerBatchCreateResponse(
        true,
        "고객정보가 일괄 등록되었습니다.",
        customers.size(),
        insertedCount
    );
}
```

JSON 배열 방식과 Wrapper 방식 중 무엇을 사용할지는 프로젝트의 API 표준에 맞춰 결정합니다.

### 12. 일괄등록 결과 DTO

```java
@Getter
@AllArgsConstructor
public class CustomerBatchCreateResponse {

    private boolean success;
    private String message;
    private int requestedCount;
    private int insertedCount;
}
```

응답 예시는 다음과 같습니다.

```json
{
    "success": true,
    "message": "고객정보가 일괄 등록되었습니다.",
    "requestedCount": 3,
    "insertedCount": 3
}
```

### 13. Service의 전체 롤백 처리

소량의 데이터를 등록하는 기본 예시입니다.

```java
@Transactional
public int createCustomers(List<CustomerCreateDto> customers) {
    int insertedCount = 0;

    for (CustomerCreateDto customer : customers) {
        insertedCount += customerMapper.insertCustomer(customer);
    }

    if (insertedCount != customers.size()) {
        throw new IllegalStateException(
            "일부 고객정보가 등록되지 않았습니다."
        );
    }

    return insertedCount;
}
```

`@Transactional`이 적용되어 있으므로 반복 처리 중 한 건이라도 예외가 발생하면 앞서 등록한 행도 함께 롤백됩니다.

```text
1행 성공
2행 성공
3행 실패
    ↓
1·2·3행 모두 롤백
```

이 방식은 이해하기 쉽지만 고객 수만큼 SQL을 실행합니다. 수백·수천 건을 처리한다면 MyBatis Batch Executor나 별도의 대량등록 방식을 검토해야 합니다.

### 14. 전체 성공과 부분 성공 정책

| 정책 | 동작 | 적합한 경우 |
|---|---|---|
| 전체 성공·전체 실패 | 한 건이라도 실패하면 전체 롤백 | 데이터 정합성이 중요한 업무 |
| 부분 성공 | 정상 건은 등록하고 오류 건만 반환 | 개별 건의 독립성이 높은 업무 |

고객 마스터처럼 정합성이 중요한 데이터는 일반적으로 전체 롤백 방식이 이해하기 쉽고 안전합니다.

부분 성공을 적용한다면 각 행을 독립 트랜잭션으로 처리하고 결과에 행별 상태와 오류 사유를 포함해야 합니다.

```json
{
    "successCount": 2,
    "failureCount": 1,
    "results": [
        { "rowNumber": 1, "success": true },
        { "rowNumber": 2, "success": false, "message": "연락처 중복" },
        { "rowNumber": 3, "success": true }
    ]
}
```

부분 성공 방식은 구현과 재처리 로직이 복잡해지므로 업무 요구사항을 먼저 확정해야 합니다.

### 15. 완료·오류 이벤트

```javascript
scwin.sbm_createCustomers_submitdone = function(e) {
    btn_createBatch.setDisabled(false);

    var success = dma_batchCreateResult.get("success");
    var message = dma_batchCreateResult.get("message");

    if (!success) {
        alert(message || "고객 일괄등록에 실패했습니다.");
        return;
    }

    alert(message);
    dlt_customerInput.removeAll();
    $p.executeSubmission(sbm_searchCustomer);
};
```

```javascript
scwin.sbm_createCustomers_submiterror = function(e) {
    btn_createBatch.setDisabled(false);
    alert("고객 일괄등록 중 오류가 발생했습니다.");
};
```

등록 성공 후 입력 DataList를 비울지는 화면 정책에 따라 결정합니다. 사용자가 등록 결과를 확인해야 한다면 즉시 삭제하지 않고 성공 상태를 표시할 수도 있습니다.

### 16. 대량 데이터 처리 시 주의사항

- 화면에서 한 번에 입력 가능한 최대 행 수를 제한합니다.
- 중복 클릭을 막기 위해 요청 중 등록 버튼을 비활성화합니다.
- 수백 건 이상은 일정 크기로 나누어 처리하는 방식을 검토합니다.
- 서버에서 요청 건수 제한을 다시 검사합니다.
- DB Unique 제약조건으로 최종 중복을 방지합니다.
- 실패한 행과 오류 사유를 사용자가 확인할 수 있게 합니다.
- 개인정보가 서버 로그에 그대로 출력되지 않도록 주의합니다.

서버에서도 최대 건수를 검사할 수 있습니다.

```java
if (customers.size() > 100) {
    throw new IllegalArgumentException(
        "한 번에 최대 100건까지 등록할 수 있습니다."
    );
}
```

### 17. 다건 등록 핵심 정리

```text
입력 Grid
    ↕ Binding
DataList
    ↓ Submission
JSON 배열
    ↓ Jackson
List<CustomerCreateDto>
    ↓ @Transactional
반복 INSERT 또는 Batch
    ↓
전체 성공·전체 롤백
```

핵심 코드는 다음과 같습니다.

```javascript
var rowIndex = dlt_customerInput.addRow();
```

```javascript
if (!scwin.validateCustomerRows()) {
    return;
}
```

```javascript
$p.executeSubmission(sbm_createCustomers);
```

```java
public int createCustomers(List<CustomerCreateDto> customers)
```

```java
@Transactional
```

---

[← 신규 고객 단건 등록]({{ '/websquare/create/' | relative_url }})
· [WebSquare 목차]({{ '/websquare/' | relative_url }})
· [다음 학습: 고객 삭제 →]({{ '/websquare/delete/' | relative_url }})