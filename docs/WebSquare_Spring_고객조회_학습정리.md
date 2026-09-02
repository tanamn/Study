# WebSquare + Spring 고객조회 학습 정리

> 고객명으로 목록을 조회하고, Grid에서 고객을 선택해 상세정보를 조회하는 전체 흐름을 기준으로 정리한 문서입니다.

## 1. 전체 구조

WebSquare 화면 개발의 기본 구조는 다음과 같습니다.

```text
사용자 입력
  ↓ Binding
DataMap
  ↓ Reference
Submission
  ↓ HTTP 요청
Spring Controller
  ↓
Service → Mapper → DB
  ↓ JSON 응답
Submission Target
  ↓
DataList 또는 DataMap
  ↓ Binding
GridView 또는 상세 입력 영역
```

핵심은 화면 컴포넌트가 서버와 직접 데이터를 주고받는 것이 아니라, `DataMap`·`DataList`와 같은 데이터 객체를 통해 연결된다는 점입니다.

---

## 2. 핵심 구성요소

### 2.1 DataMap

한 건의 데이터를 저장하는 key/value 형태의 객체입니다.

주로 다음 용도로 사용합니다.

- 조회조건
- 상세정보
- 등록·수정 데이터

```javascript
{
    customerName: "김"
}
```

Java의 DTO 한 건과 비슷합니다.

### 2.2 DataList

여러 건의 행 데이터를 저장합니다.

```text
row 0: C001, 김철수
row 1: C002, 김영희
```

Java의 `List<CustomerDto>`와 비슷하며, 일반적으로 GridView와 바인딩합니다.

### 2.3 GridView

DataList의 내용을 표 형태로 보여주는 화면 컴포넌트입니다.

```text
DataList = 실제 데이터
GridView = 데이터를 보여주는 화면
```

### 2.4 Submission

서버와 데이터를 주고받는 통신 객체입니다. React의 `axios` 또는 `fetch()`와 비슷한 역할을 합니다.

```text
Reference = 서버로 보낼 데이터
Target    = 서버에서 받은 데이터를 저장할 곳
```

실행 예시는 다음과 같습니다.

```javascript
$p.executeSubmission(sbm_searchCustomer);
```

프로젝트 공통함수로 감싼 경우 다음처럼 보일 수도 있습니다.

```javascript
com.sbm.execute(sbm_searchCustomer);
```

---

## 3. 고객 목록 조회 예제

### 3.1 화면 구성

```text
고객명 입력: ibx_customerName
조회 버튼:   btn_search
결과 Grid:   grd_customer
```

### 3.2 조회조건 DataMap

```text
ID: dma_search
컬럼: customerName
```

고객명 입력 컴포넌트와 다음처럼 바인딩합니다.

```text
ibx_customerName
        ↕ Binding
dma_search.customerName
```

사용자가 `김`을 입력하면 DataMap은 다음 상태가 됩니다.

```javascript
{
    customerName: "김"
}
```

### 3.3 목록 결과 DataList

```text
ID: dlt_customer

컬럼:
- customerId
- customerName
- birthDate
- phone
```

Grid와 연결합니다.

```text
dlt_customer
     ↓ Binding
grd_customer
```

### 3.4 목록조회 Submission

```text
ID:        sbm_searchCustomer
Action:    /api/customer/search
Method:    POST
Reference: dma_search
Target:    dlt_customer
Mode:      asynchronous
```

요청과 응답의 방향은 다음과 같습니다.

```text
dma_search → Reference → 서버 요청
서버 응답 → Target → dlt_customer
```

### 3.5 조회 버튼 이벤트

```javascript
scwin.btn_search_onclick = function(e) {
    var customerName = dma_search.get("customerName");

    if (!customerName) {
        alert("고객명을 입력해주세요.");
        return;
    }

    $p.executeSubmission(sbm_searchCustomer);
};
```

실무의 공통 라이브러리를 적용한 코드는 다음처럼 보일 수 있습니다.

```javascript
scwin.btn_search_onclick = function(e) {
    if (com.util.isEmpty(dma_search.get("customerName"))) {
        com.alert("MSG001", "고객명을 입력하세요.");
        return;
    }

    com.sbm.execute(sbm_searchCustomer);
};
```

`com.util`, `com.alert`, `com.sbm`은 WebSquare 기본 API가 아니라 프로젝트 공통 라이브러리일 가능성이 높습니다.

---

## 4. 목록조회 백엔드

### 4.1 요청 DTO

```java
@Getter
@Setter
public class CustomerSearchDto {
    private String customerName;
}
```

### 4.2 결과 DTO

```java
@Getter
@Setter
public class CustomerDto {
    private String customerId;
    private String customerName;
    private String birthDate;
    private String phone;
    private String address;
    private String job;
}
```

### 4.3 Controller

```java
@RestController
@RequestMapping("/api/customer")
@RequiredArgsConstructor
public class CustomerController {

    private final CustomerService customerService;

    @PostMapping("/search")
    public List<CustomerDto> searchCustomer(
            @RequestBody CustomerSearchDto searchDto) {

        return customerService.searchCustomer(searchDto);
    }
}
```

### 4.4 Service

```java
@Service
@RequiredArgsConstructor
public class CustomerService {

    private final CustomerMapper customerMapper;

    public List<CustomerDto> searchCustomer(
            CustomerSearchDto searchDto) {

        return customerMapper.searchCustomer(searchDto);
    }
}
```

### 4.5 Mapper

```java
@Mapper
public interface CustomerMapper {
    List<CustomerDto> searchCustomer(CustomerSearchDto searchDto);
}
```

```xml
<select id="searchCustomer" resultType="CustomerDto">
    SELECT
          CUSTOMER_ID   AS customerId
        , CUSTOMER_NAME AS customerName
        , BIRTH_DATE    AS birthDate
        , PHONE         AS phone
      FROM CUSTOMER
     WHERE CUSTOMER_NAME LIKE CONCAT('%', #{customerName}, '%')
</select>
```

### 4.6 응답 처리

서버 응답 예시:

```json
[
    {
        "customerId": "C001",
        "customerName": "김철수",
        "birthDate": "1985-01-01",
        "phone": "010-1111-2222"
    },
    {
        "customerId": "C002",
        "customerName": "김영희",
        "birthDate": "1990-04-15",
        "phone": "010-3333-4444"
    }
]
```

응답은 Submission의 Target인 `dlt_customer`에 저장되고, 바인딩된 `grd_customer`에 자동 표시됩니다.

```text
JSON 응답
   ↓
dlt_customer
   ↓ Binding
grd_customer
```

---

## 5. Submission 완료·오류 처리

### 5.1 정상 완료

```javascript
scwin.sbm_searchCustomer_submitdone = function(e) {
    if (dlt_customer.getRowCount() === 0) {
        alert("조회 결과가 없습니다.");
        return;
    }

    console.log("조회 완료");
};
```

### 5.2 오류 발생

```javascript
scwin.sbm_searchCustomer_submiterror = function(e) {
    alert("고객 조회 중 오류가 발생했습니다.");
};
```

프로젝트에 따라 이벤트 함수명이나 공통 처리 방식은 달라질 수 있으므로 기존 화면의 구현 규칙을 먼저 확인해야 합니다.

---

## 6. Grid 선택 후 고객 상세조회

이번 기능의 목표는 다음과 같습니다.

```text
Grid 행 선택
  ↓
선택한 행의 customerId 추출
  ↓
상세조회 조건 DataMap에 저장
  ↓
상세조회 Submission 실행
  ↓
상세정보 DataMap에 응답 저장
  ↓
상세화면에 표시
```

### 6.1 사용하는 데이터 객체

| 객체 | 역할 |
|---|---|
| `dlt_customer` | 고객 목록 |
| `dma_detailSearch` | 상세조회 요청조건 |
| `dma_customerDetail` | 고객 상세조회 결과 |

### 6.2 상세조회 조건 DataMap

```text
ID: dma_detailSearch
컬럼: customerId
```

```javascript
{
    customerId: ""
}
```

### 6.3 상세결과 DataMap

고객 한 명의 정보이므로 DataList가 아닌 DataMap을 사용합니다.

```text
ID: dma_customerDetail

컬럼:
- customerId
- customerName
- birthDate
- phone
- address
- job
```

### 6.4 상세조회 Submission

```text
ID:        sbm_searchCustomerDetail
Action:    /api/customer/detail
Method:    POST
Reference: dma_detailSearch
Target:    dma_customerDetail
```

---

## 7. Grid 클릭 이벤트의 핵심 코드

### 7.1 선택 행의 고객 ID 추출

```javascript
var customerId = dlt_customer.getCellData(
    rowIndex,
    "customerId"
);
```

Java로 생각하면 다음과 비슷합니다.

```java
CustomerDto customer = customerList.get(rowIndex);
String customerId = customer.getCustomerId();
```

### 7.2 상세조회 DataMap에 저장

```javascript
dma_detailSearch.set("customerId", customerId);
```

### 7.3 상세조회 실행

```javascript
$p.executeSubmission(sbm_searchCustomerDetail);
```

### 7.4 전체 코드

```javascript
scwin.grd_customer_oncellclick = function(rowIndex, colIndex) {
    var customerId = dlt_customer.getCellData(
        rowIndex,
        "customerId"
    );

    dma_detailSearch.set("customerId", customerId);

    $p.executeSubmission(sbm_searchCustomerDetail);
};
```

이번 예제의 핵심은 다음 세 단계입니다.

```text
getCellData()
    ↓
DataMap.set()
    ↓
executeSubmission()
```

---

## 8. 실무형 코드 구조

Grid 이벤트와 상세조회 함수를 분리하면 다른 이벤트에서도 상세조회 기능을 재사용할 수 있습니다.

```javascript
scwin.grd_customer_oncellclick = function(rowIndex, colIndex) {
    if (rowIndex < 0) {
        return;
    }

    var customerId = dlt_customer.getCellData(
        rowIndex,
        "customerId"
    );

    if (!customerId) {
        return;
    }

    scwin.searchCustomerDetail(customerId);
};

scwin.searchCustomerDetail = function(customerId) {
    dma_detailSearch.set("customerId", customerId);
    $p.executeSubmission(sbm_searchCustomerDetail);
};
```

다음과 같은 여러 진입점에서 재사용할 수 있습니다.

```text
Grid 행 클릭 ──────┐
Grid 더블클릭 ─────┼→ searchCustomerDetail()
팝업 고객 선택 ───┘
```

---

## 9. 상세조회 백엔드

### 9.1 요청 DTO

```java
@Getter
@Setter
public class CustomerDetailSearchDto {
    private String customerId;
}
```

### 9.2 Controller

```java
@PostMapping("/detail")
public CustomerDto getCustomerDetail(
        @RequestBody CustomerDetailSearchDto searchDto) {

    return customerService.getCustomerDetail(
        searchDto.getCustomerId()
    );
}
```

### 9.3 Service

```java
public CustomerDto getCustomerDetail(String customerId) {
    return customerMapper.selectCustomerDetail(customerId);
}
```

### 9.4 Mapper

```java
CustomerDto selectCustomerDetail(String customerId);
```

```xml
<select id="selectCustomerDetail" resultType="CustomerDto">
    SELECT
          CUSTOMER_ID   AS customerId
        , CUSTOMER_NAME AS customerName
        , BIRTH_DATE    AS birthDate
        , PHONE         AS phone
        , ADDRESS       AS address
        , JOB           AS job
      FROM CUSTOMER
     WHERE CUSTOMER_ID = #{customerId}
</select>
```

목록의 전체 데이터를 보내지 않고 `customerId`만 전달해 DB에서 최신 상세정보를 다시 조회하는 것이 일반적입니다.

```text
목록 화면의 값 ≠ 반드시 최신 상세 값
```

---

## 10. 상세정보 화면 바인딩

상세영역의 입력 컴포넌트를 `dma_customerDetail`의 각 컬럼에 바인딩합니다.

```text
ibx_customerId   ↔ dma_customerDetail.customerId
ibx_customerName ↔ dma_customerDetail.customerName
ibx_birthDate    ↔ dma_customerDetail.birthDate
ibx_phone        ↔ dma_customerDetail.phone
ibx_address      ↔ dma_customerDetail.address
ibx_job          ↔ dma_customerDetail.job
```

Submission 결과가 DataMap에 저장되면 화면에도 자동으로 반영되므로 다음과 같은 개별 값 설정 코드는 필요하지 않습니다.

```javascript
// 바인딩했다면 일반적으로 불필요
ibx_customerName.setValue(...);
ibx_phone.setValue(...);
```

---

## 11. 목록조회부터 상세조회까지 한 번에 보기

```text
[고객명 입력]
ibx_customerName
    ↓ Binding
dma_search
    ↓ Reference
sbm_searchCustomer
    ↓ POST /api/customer/search
Controller → Service → Mapper → DB
    ↓ List<CustomerDto>
dlt_customer
    ↓ Binding
grd_customer

[Grid 행 선택]
    ↓
dlt_customer.getCellData(rowIndex, "customerId")
    ↓
dma_detailSearch.set("customerId", customerId)
    ↓ Reference
sbm_searchCustomerDetail
    ↓ POST /api/customer/detail
Controller → Service → Mapper → DB
    ↓ CustomerDto
dma_customerDetail
    ↓ Binding
고객 상세정보 화면
```

---

## 12. React와 비교

| React | WebSquare |
|---|---|
| `useState({ customerName: "" })` | DataMap |
| 고객 목록 배열 | DataList |
| `axios.post()` / `fetch()` | Submission |
| 요청 객체 | Submission Reference |
| `response.data` | Submission Target |
| DataGrid | GridView |
| `row.customerId` | `dlt_customer.getCellData()` |
| `setCustomerDetail()` | 상세 DataMap에 Target 저장 |
| state와 input 연결 | DataMap Binding |

React의 목록조회 코드가 다음과 같다면:

```javascript
const response = await axios.post(
    "/api/customer/search",
    searchCondition
);

setCustomers(response.data);
```

WebSquare에서는 다음 구조로 대응됩니다.

```text
searchCondition → dma_search
axios.post()     → Submission
response.data   → dlt_customer
setCustomers()  → Submission Target 처리
```

---

## 13. Java/Spring과 WebSquare 대응표

| Java/Spring | WebSquare |
|---|---|
| DTO 한 건 | DataMap |
| `List<DTO>` | DataList |
| REST API 호출 | Submission |
| Controller URL | Submission Action |
| `@RequestBody` 데이터 | Reference |
| Response 데이터 | Target |
| HTML Table/DataGrid | GridView |
| 버튼 클릭 핸들러 | `scwin.xxx_onclick` |
| `axios`/`fetch` | `$p.executeSubmission()` |

---

## 14. 처음 보는 프로젝트 소스 추적 순서

조회 버튼의 동작을 확인할 때는 XML 전체를 처음부터 읽기보다 다음 순서로 따라가는 것이 효율적입니다.

1. 화면에서 버튼 ID를 확인합니다.

   ```text
   btn_search
   ```

2. 버튼 이벤트를 검색합니다.

   ```javascript
   scwin.btn_search_onclick
   ```

3. 이벤트 안에서 실행하는 Submission을 확인합니다.

   ```javascript
   $p.executeSubmission(sbm_searchCustomer);
   ```

4. Submission의 Action(URL)을 확인합니다.

   ```text
   /api/customer/search
   ```

5. Reference와 Target을 확인합니다.

   ```text
   Reference: dma_search
   Target:    dlt_customer
   ```

6. 백엔드에서 Controller URL을 검색합니다.

   ```java
   @PostMapping("/search")
   ```

7. Service → Mapper → SQL 순서로 따라갑니다.

실전 공식으로 줄이면 다음과 같습니다.

```text
Button
  ↓
onclick
  ↓
Submission
  ↓
Reference / Target
  ↓
DataMap / DataList
  ↓
Controller
  ↓
Service
  ↓
Mapper
  ↓
SQL
```

---

## 15. 꼭 기억할 핵심 다섯 가지

1. **DataMap**: 조회조건이나 상세정보 같은 한 건의 데이터
2. **DataList**: 조회목록 같은 여러 건의 데이터
3. **GridView**: DataList를 화면에 표시
4. **Submission**: 서버와 통신
5. **핵심 흐름**

   ```text
   DataMap
      ↓
   Submission
      ↓
   Spring Controller
      ↓
   DataList 또는 DataMap
      ↓
   GridView 또는 상세화면
   ```

Grid 선택 상세조회의 핵심 세 줄도 함께 기억하면 좋습니다.

```javascript
var customerId = dlt_customer.getCellData(rowIndex, "customerId");
dma_detailSearch.set("customerId", customerId);
$p.executeSubmission(sbm_searchCustomerDetail);
```

---

## 16. 다음 학습 단계

다음 단계로 아래 흐름을 연결하면 WebSquare의 기본 CRUD 구조를 완성할 수 있습니다.

```text
상세정보 수정
  ↓
DataMap 값 변경
  ↓
저장 Submission 실행
  ↓
Spring @PostMapping
  ↓
INSERT 또는 UPDATE
  ↓
저장 성공 처리
  ↓
목록 재조회
```

추천 학습 순서는 다음과 같습니다.

1. 상세정보 수정 및 저장
2. 신규 고객 등록
3. 고객 삭제
4. 입력값 검증
5. 공통 메시지·공통 Submission 함수 분석
6. 트랜잭션과 예외 처리

```

