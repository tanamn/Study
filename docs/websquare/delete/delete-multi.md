---
layout: default
title: WebSquare Grid 다건 삭제
permalink: /websquare/delete/multi/
---

{% include navigation.html %}

# WebSquare Grid 다건 삭제

고객목록 Grid에서 여러 고객을 체크한 후 한 번에 삭제하는 기능을 구현해 본다.

단건 삭제에서는 현재 선택된 한 행의 `customerId`를 서버로 전달했다.

다건 삭제에서는 다음과 같이 처리한다.

> 체크박스로 여러 행 선택 → 선택된 고객정보 추출 → 삭제 대상 DataList 구성 → Spring으로 전송 → 일괄 삭제 → 목록 재조회

---

## 1. 학습 목표

이번 학습에서는 다음 내용을 익힌다.

- GridView 체크박스를 이용한 다건 선택
- `getCheckedJSON()`을 이용한 선택 행 조회
- 선택된 고객의 `customerId` 추출
- 삭제 요청용 DataList 구성
- Submission을 이용한 다건 데이터 전송
- Spring에서 `List` 형태의 삭제 요청 처리
- MyBatis `<foreach>`를 이용한 다건 삭제
- 요청 건수와 실제 삭제 건수 확인
- 삭제 완료 후 고객목록 재조회

---

## 2. 단건 삭제와 다건 삭제 비교

### 단건 삭제

```text
고객 한 명 선택
    ↓
customerId 한 건
    ↓
서버 전송
    ↓
DELETE
```

예를 들어 다음과 같은 요청이다.

```json
{
  "customerId": "C000001"
}
```

### 다건 삭제

```text
고객 여러 명 체크
    ↓
customerId 여러 건
    ↓
서버 전송
    ↓
일괄 DELETE
```

예를 들어 다음과 같이 여러 고객 ID를 전달한다.

```json
{
  "customers": [
    {
      "customerId": "C000001"
    },
    {
      "customerId": "C000003"
    },
    {
      "customerId": "C000005"
    }
  ]
}
```

핵심 차이는 서버로 전달되는 삭제 대상이 한 건인지 여러 건인지이다.

---

# 3. 전체 처리 흐름

```text
[WebSquare]

고객목록 조회
        ↓
삭제할 고객 체크
        ↓
[선택삭제] 버튼 클릭
        ↓
선택된 행 조회
        ↓
삭제 대상 customerId 추출
        ↓
삭제 요청용 DataList 구성
        ↓
Submission 실행
        ↓

[Spring]

Controller
        ↓
CustomerBatchDeleteRequest
        ↓
Service
        ↓
Mapper
        ↓
DELETE SQL 실행
        ↓
삭제 결과 반환
        ↓

[WebSquare]

삭제 완료 메시지
        ↓
고객목록 재조회
```

---

# 4. 화면 구성

고객목록의 첫 번째 컬럼을 체크박스로 구성한다고 가정한다.

```text
┌────┬──────────┬──────────┬─────────────────────┐
│ 선택 │ 고객ID   │ 고객명   │ 연락처              │
├────┼──────────┼──────────┼─────────────────────┤
│ ☑  │ C000001  │ 김민수   │ 010-1111-2222       │
│ ☐  │ C000002  │ 이영희   │ 010-3333-4444       │
│ ☑  │ C000003  │ 박철수   │ 010-5555-6666       │
└────┴──────────┴──────────┴─────────────────────┘

                                      [선택삭제]
```

예제에서 사용하는 객체는 다음과 같다.

| 구분 | ID | 설명 |
|---|---|---|
| GridView | `grd_customer` | 고객목록 |
| DataList | `dlt_customer` | 조회된 고객 데이터 |
| DataList | `dlt_customerDelete` | 삭제 요청 데이터 |
| DataMap | `dma_customerDeleteResult` | 삭제 결과 |
| Submission | `sbm_customerDeleteBatch` | 다건 삭제 요청 |

---

# 5. GridView 체크박스 컬럼

GridView의 체크박스 컬럼 ID를 `chk`라고 가정한다.

개념적으로 다음과 같은 구조이다.

```text
grd_customer
├─ chk
├─ customerId
├─ name
└─ phone
```

화면 데이터는 다음과 같이 생각할 수 있다.

```text
chk | customerId | name   | phone
-----------------------------------------
1   | C000001    | 김민수 | 010-1111-2222
0   | C000002    | 이영희 | 010-3333-4444
1   | C000003    | 박철수 | 010-5555-6666
```

여기서 체크된 행은

```text
C000001
C000003
```

이다.

---

# 6. 체크된 행 조회

WebSquare GridView에서는 체크박스 컬럼을 기준으로 체크된 행을 JSON 배열로 조회할 수 있다.

```javascript
var checkedRows =
    grd_customer.getCheckedJSON("chk");
```

예를 들어 첫 번째와 세 번째 고객을 체크했다면 다음과 같은 배열을 얻을 수 있다.

```javascript
[
    {
        chk: "1",
        customerId: "C000001",
        name: "김민수",
        phone: "010-1111-2222"
    },
    {
        chk: "1",
        customerId: "C000003",
        name: "박철수",
        phone: "010-5555-6666"
    }
]
```

즉,

```javascript
grd_customer.getCheckedJSON("chk");
```

는

> `chk` 체크박스 컬럼이 선택된 행들의 데이터를 JSON 배열로 가져온다.

라고 이해하면 된다.

---

## 6.1 선택 건수 확인

```javascript
if (checkedRows.length === 0) {
    alert("삭제할 고객을 선택해 주세요.");
    return;
}
```

아무 고객도 선택하지 않았다면 삭제 요청을 보내지 않는다.

---

# 7. 삭제 요청용 DataList

조회용 `dlt_customer` 전체를 서버로 보내는 것보다 삭제에 필요한 값만 별도의 DataList에 담는 것이 이해하기 쉽다.

삭제에 필요한 값은 `customerId`뿐이다.

```xml
<w2:dataList
    id="dlt_customerDelete"
    baseNode="list"
    repeatNode="map">

    <w2:columnInfo>
        <w2:column
            id="customerId"
            name="고객ID"
            dataType="text"/>
    </w2:columnInfo>

</w2:dataList>
```

삭제 대상이 세 명이라면 DataList는 다음과 같은 형태가 된다.

```text
dlt_customerDelete

customerId
-----------
C000001
C000003
C000005
```

---

# 8. 선택 고객의 ID만 추출하기

Grid에서 체크된 행 전체를 가져왔지만 서버에는 삭제에 필요한 `customerId`만 전달한다.

```javascript
var deleteRows = [];

for (var i = 0; i < checkedRows.length; i++) {

    deleteRows.push({
        customerId: checkedRows[i].customerId
    });
}
```

예를 들어 `checkedRows`가 다음과 같다면

```javascript
[
    {
        customerId: "C000001",
        name: "김민수"
    },
    {
        customerId: "C000003",
        name: "박철수"
    }
]
```

`deleteRows`에는 다음 값만 저장된다.

```javascript
[
    {
        customerId: "C000001"
    },
    {
        customerId: "C000003"
    }
]
```

삭제에 필요한 데이터만 서버에 전달하는 것이다.

---

# 9. DataList에 삭제 대상 설정

앞에서 만든 `deleteRows` 배열을 `dlt_customerDelete`에 설정한다.

```javascript
dlt_customerDelete.setJSON(
    deleteRows
);
```

결과는 다음과 같다.

```text
deleteRows
    ↓
dlt_customerDelete

customerId
-----------
C000001
C000003
```

`setJSON()`은 JSON 객체 배열을 DataList에 설정할 때 사용할 수 있다.

---

# 10. 삭제 확인

다건 삭제는 여러 데이터가 한 번에 삭제되므로 선택 건수를 사용자에게 보여 주는 것이 좋다.

```javascript
var result = confirm(
    checkedRows.length +
    "명의 고객정보를 삭제하시겠습니까?"
);

if (!result) {
    return;
}
```

세 명을 선택했다면 다음과 같은 메시지가 표시된다.

```text
3명의 고객정보를 삭제하시겠습니까?
```

사용자가 취소하면 이후 로직은 실행하지 않는다.

---

# 11. 선택삭제 버튼 전체 코드

```javascript
scwin.btn_deleteSelected_onclick = function() {

    var checkedRows =
        grd_customer.getCheckedJSON("chk");

    if (checkedRows.length === 0) {
        alert("삭제할 고객을 선택해 주세요.");
        return;
    }

    var result = confirm(
        checkedRows.length +
        "명의 고객정보를 삭제하시겠습니까?"
    );

    if (!result) {
        return;
    }

    var deleteRows = [];

    for (var i = 0;
         i < checkedRows.length;
         i++) {

        deleteRows.push({
            customerId:
                checkedRows[i].customerId
        });
    }

    dlt_customerDelete.setJSON(
        deleteRows
    );

    $p.executeSubmission(
        "sbm_customerDeleteBatch"
    );
};
```

처리 순서는 다음과 같다.

```text
1. 체크된 행 조회

2. 선택 건수 확인

3. 삭제 여부 확인

4. customerId만 추출

5. 삭제용 DataList에 설정

6. Submission 실행
```

---

# 12. Submission 구성

삭제 요청용 DataList를 서버로 전달한다.

프로젝트의 Submission 구성 방식에 따라 세부 XML은 달라질 수 있지만 개념적으로 다음과 같이 구성한다.

```xml
<w2:submission
    id="sbm_customerDeleteBatch"
    action="/api/customers/delete-batch"
    method="post"
    mediatype="application/json"
    ev:submitdone="scwin.sbm_customerDeleteBatch_submitdone"
    ev:submiterror="scwin.sbm_customerDeleteBatch_submiterror">
</w2:submission>
```

서버에는 최종적으로 다음과 같은 구조가 전달되도록 구성한다.

```json
{
  "customers": [
    {
      "customerId": "C000001"
    },
    {
      "customerId": "C000003"
    }
  ]
}
```

> 실제 프로젝트에서는 공통 Submission 모듈의 요청 구조와 DataCollection 매핑 규칙을 우선 적용한다.

---

# 13. Spring 요청 DTO

다건 삭제 요청을 받을 DTO를 작성한다.

```java
@Getter
@Setter
public class CustomerBatchDeleteRequest {

    @NotEmpty
    @Valid
    private List<CustomerDeleteDto> customers;
}
```

고객 한 건의 삭제 정보를 나타내는 DTO를 작성한다.

```java
@Getter
@Setter
public class CustomerDeleteDto {

    @NotBlank
    private String customerId;
}
```

요청 데이터는 다음과 같이 매핑된다.

```json
{
  "customers": [
    {
      "customerId": "C000001"
    },
    {
      "customerId": "C000003"
    }
  ]
}
```

```text
CustomerBatchDeleteRequest
        │
        └─ customers
              │
              ├─ CustomerDeleteDto
              │      └─ C000001
              │
              └─ CustomerDeleteDto
                     └─ C000003
```

---

# 14. Controller

```java
@RestController
@RequestMapping("/api/customers")
@RequiredArgsConstructor
public class CustomerController {

    private final CustomerService customerService;

    @PostMapping("/delete-batch")
    public CustomerBatchDeleteResponse deleteCustomers(
            @Valid
            @RequestBody
            CustomerBatchDeleteRequest request) {

        int requestedCount =
                request.getCustomers().size();

        int deletedCount =
                customerService.deleteCustomers(
                    request.getCustomers()
                );

        return new CustomerBatchDeleteResponse(
                true,
                "고객정보가 일괄 삭제되었습니다.",
                requestedCount,
                deletedCount
        );
    }
}
```

Controller에서는 크게 세 가지 작업을 한다.

```text
1. WebSquare 요청 수신

2. Service에 삭제 대상 전달

3. 요청 건수와 삭제 건수를 응답
```

---

# 15. 응답 DTO

```java
@Getter
@AllArgsConstructor
public class CustomerBatchDeleteResponse {

    private boolean success;

    private String message;

    private int requestedCount;

    private int deletedCount;
}
```

예를 들어 3명을 요청했고 3명이 정상적으로 삭제되었다면 다음과 같이 응답한다.

```json
{
  "success": true,
  "message": "고객정보가 일괄 삭제되었습니다.",
  "requestedCount": 3,
  "deletedCount": 3
}
```

---

# 16. Service

```java
@Service
@RequiredArgsConstructor
public class CustomerService {

    private final CustomerMapper customerMapper;

    @Transactional
    public int deleteCustomers(
            List<CustomerDeleteDto> customers) {

        int deletedCount =
                customerMapper.deleteCustomers(
                    customers
                );

        if (deletedCount != customers.size()) {
            throw new IllegalStateException(
                "일부 고객정보가 삭제되지 않았습니다."
            );
        }

        return deletedCount;
    }
}
```

여기서 중요한 부분은 다음 코드이다.

```java
if (deletedCount != customers.size()) {
```

예를 들어 요청한 고객이 3명이라면

```text
요청 건수 = 3
```

실제로 삭제된 데이터도

```text
삭제 건수 = 3
```

이어야 한다.

---

## 16.1 일부 데이터만 삭제된 경우

예를 들어 다음 고객을 요청했다고 가정한다.

```text
C000001
C000003
C999999
```

그런데 `C999999`가 DB에 존재하지 않는다면

```text
요청 건수 = 3
삭제 건수 = 2
```

가 될 수 있다.

이 경우 Service에서 오류로 처리하면 `@Transactional`에 의해 전체 작업을 롤백하도록 구성할 수 있다.

```text
C000001 삭제
C000003 삭제
C999999 없음
        ↓
오류 발생
        ↓
Transaction Rollback
```

즉, 다건 작업에서는 **전체 성공 또는 전체 실패** 방식으로 처리하는 것이 이해하기 쉽다.

업무 요구사항에 따라 일부 성공을 허용하는 방식으로 설계할 수도 있다.

---

# 17. Mapper

```java
@Mapper
public interface CustomerMapper {

    int deleteCustomers(
        List<CustomerDeleteDto> customers
    );
}
```

Mapper에서는 삭제된 Row의 개수를 `int`로 반환한다.

---

# 18. MyBatis Mapper XML

여러 고객 ID를 한 번에 삭제하기 위해 `IN` 조건과 MyBatis `<foreach>`를 사용할 수 있다.

```xml
<delete id="deleteCustomers">

    DELETE
      FROM CUSTOMER
     WHERE CUSTOMER_ID IN
     (
        <foreach
            collection="list"
            item="customer"
            separator=",">

            #{customer.customerId}

        </foreach>
     )

</delete>
```

예를 들어 다음 데이터가 전달되었다고 가정한다.

```text
C000001
C000003
C000005
```

MyBatis는 개념적으로 다음과 같은 SQL을 실행한다.

```sql
DELETE
  FROM CUSTOMER
 WHERE CUSTOMER_ID IN (
       'C000001',
       'C000003',
       'C000005'
 );
```

실제 값은 MyBatis Parameter Binding을 통해 전달된다.

---

# 19. foreach 이해하기

다음 코드를 자세히 살펴보자.

```xml
<foreach
    collection="list"
    item="customer"
    separator=",">

    #{customer.customerId}

</foreach>
```

### collection

```xml
collection="list"
```

Mapper로 전달된 `List`를 반복한다.

### item

```xml
item="customer"
```

List에서 현재 반복 중인 한 건의 데이터를 `customer`라는 이름으로 사용한다.

JavaScript의 반복문으로 비유하면 다음과 비슷하다.

```javascript
for (var customer of customers) {
    ...
}
```

### customerId

```xml
#{customer.customerId}
```

현재 고객 객체의 `customerId` 값을 가져온다.

### separator

```xml
separator=","
```

각 값을 쉼표로 구분한다.

결과적으로 다음과 같은 형태가 만들어진다.

```text
?, ?, ?
```

각 `?`에는 실제 `customerId`가 Parameter Binding 된다.

---

# 20. WebSquare 삭제 완료 처리

서버에서 정상 응답을 받으면 완료 이벤트를 실행한다.

```javascript
scwin.sbm_customerDeleteBatch_submitdone =
function(e) {

    var success =
        dma_customerDeleteResult.get(
            "success"
        );

    var deletedCount =
        dma_customerDeleteResult.get(
            "deletedCount"
        );

    if (success) {

        alert(
            deletedCount +
            "명의 고객정보가 삭제되었습니다."
        );

        scwin.search();
    }
};
```

예를 들어 세 건이 삭제되었다면 다음 메시지가 표시된다.

```text
3명의 고객정보가 삭제되었습니다.
```

그리고

```javascript
scwin.search();
```

를 호출해 고객목록을 다시 조회한다.

---

# 21. 오류 처리

```javascript
scwin.sbm_customerDeleteBatch_submiterror =
function(e) {

    alert(
        "고객정보 일괄 삭제 중 오류가 발생했습니다."
    );
};
```

다건 삭제에서 발생할 수 있는 오류는 다음과 같다.

```text
삭제할 고객 미선택

삭제 대상 고객이 DB에 존재하지 않음

요청 건수와 삭제 건수가 다름

DB 오류

서버 오류

네트워크 오류
```

---

# 22. 전체 JavaScript

```javascript
scwin.btn_deleteSelected_onclick = function() {

    var checkedRows =
        grd_customer.getCheckedJSON("chk");

    if (checkedRows.length === 0) {
        alert("삭제할 고객을 선택해 주세요.");
        return;
    }

    var result = confirm(
        checkedRows.length +
        "명의 고객정보를 삭제하시겠습니까?"
    );

    if (!result) {
        return;
    }

    var deleteRows = [];

    for (var i = 0;
         i < checkedRows.length;
         i++) {

        deleteRows.push({
            customerId:
                checkedRows[i].customerId
        });
    }

    dlt_customerDelete.setJSON(
        deleteRows
    );

    $p.executeSubmission(
        "sbm_customerDeleteBatch"
    );
};


scwin.sbm_customerDeleteBatch_submitdone =
function(e) {

    var success =
        dma_customerDeleteResult.get(
            "success"
        );

    var deletedCount =
        dma_customerDeleteResult.get(
            "deletedCount"
        );

    if (success) {

        alert(
            deletedCount +
            "명의 고객정보가 삭제되었습니다."
        );

        scwin.search();
    }
};


scwin.sbm_customerDeleteBatch_submiterror =
function(e) {

    alert(
        "고객정보 일괄 삭제 중 오류가 발생했습니다."
    );
};
```

---

# 23. 전체 Spring 코드 흐름

```text
CustomerController
        │
        │ List<CustomerDeleteDto>
        ▼
CustomerService
        │
        │ @Transactional
        ▼
CustomerMapper
        │
        ▼
MyBatis foreach
        │
        ▼
DELETE
FROM CUSTOMER
WHERE CUSTOMER_ID IN (...)
```

각 계층의 역할은 다음과 같다.

| 계층 | 역할 |
|---|---|
| WebSquare | 삭제 대상 고객 선택 |
| DataList | 삭제 요청 데이터 구성 |
| Submission | 서버로 다건 데이터 전송 |
| Controller | 삭제 요청 수신 |
| Service | 삭제 건수 검증 및 Transaction 처리 |
| Mapper | 다건 DELETE SQL 실행 |
| DB | 실제 고객정보 삭제 |

---

# 24. 물리 삭제와 논리 삭제

앞의 예제는 DB의 데이터를 실제로 제거하는 **물리 삭제** 방식이다.

```sql
DELETE
  FROM CUSTOMER
 WHERE CUSTOMER_ID IN (...);
```

업무 시스템에서는 실제 Row를 삭제하지 않고 삭제 여부만 변경하는 **논리 삭제**를 사용할 수도 있다.

예를 들어 다음과 같다.

```sql
UPDATE CUSTOMER
   SET DELETE_YN = 'Y'
 WHERE CUSTOMER_ID IN (...);
```

조회할 때는 삭제되지 않은 고객만 조회한다.

```sql
SELECT *
  FROM CUSTOMER
 WHERE DELETE_YN = 'N';
```

다건 처리 방식 자체는 동일하다.

```text
선택된 customerId 목록
        ↓
WHERE CUSTOMER_ID IN (...)
```

차이는 실행하는 SQL이 `DELETE`인지 `UPDATE`인지이다.

---

# 25. 다건 삭제에서 주의할 점

## 25.1 화면에서 행만 제거하면 안 된다

화면의 DataList Row만 제거하면 화면에서는 사라져 보일 수 있지만 서버 DB의 데이터가 삭제된 것은 아니다.

다건 삭제에서도 반드시 서버를 통해 DB 삭제 처리를 해야 한다.

## 25.2 고객명 대신 customerId를 사용한다

다음과 같이 고객명으로 삭제하면 안 된다.

```sql
WHERE NAME = ...
```

동명이인이 있을 수 있기 때문이다.

삭제 조건은 고유 식별값을 사용한다.

```sql
WHERE CUSTOMER_ID IN (...)
```

## 25.3 요청 건수와 삭제 건수를 확인한다

```text
요청 5건
삭제 5건
```

이면 정상이다.

하지만

```text
요청 5건
삭제 4건
```

이라면 한 건이 삭제되지 않은 것이다.

따라서 Service에서 두 값을 비교하는 것이 좋다.

## 25.4 Transaction을 사용한다

다건 삭제는 여러 데이터를 하나의 업무로 처리한다.

```java
@Transactional
public int deleteCustomers(...) {
    ...
}
```

중간에 오류가 발생하면 전체 삭제 작업을 롤백할 수 있다.

---

# 26. 단건 삭제와 다건 삭제 정리

| 구분 | 단건 삭제 | 다건 삭제 |
|---|---|---|
| 선택 방법 | 한 행 선택 | 체크박스 여러 행 선택 |
| Grid API | 선택 행 확인 | `getCheckedJSON()` |
| 요청 데이터 | `customerId` 한 건 | `customerId` 여러 건 |
| Java DTO | 단일 DTO | `List<DTO>` |
| SQL | `WHERE CUSTOMER_ID = ...` | `WHERE CUSTOMER_ID IN (...)` |
| MyBatis | 단일 Parameter | `<foreach>` |
| Transaction | 선택적 | 중요 |
| 결과 확인 | 삭제 여부 | 요청 건수와 삭제 건수 |

---

# 27. 핵심 정리

WebSquare 다건 삭제의 핵심 흐름은 다음과 같다.

```text
1. Grid 체크박스로 여러 고객 선택

2. grd_customer.getCheckedJSON("chk")
   → 체크된 행 조회

3. checkedRows.length
   → 선택 건수 확인

4. customerId만 추출
   → deleteRows 배열 생성

5. dlt_customerDelete.setJSON()
   → 삭제 요청용 DataList 구성

6. $p.executeSubmission()
   → Spring으로 요청

7. Controller
   → List 형태의 삭제 요청 수신

8. Service
   → Transaction 처리
   → 요청 건수와 삭제 건수 비교

9. Mapper
   → MyBatis foreach 실행

10. DELETE ... WHERE CUSTOMER_ID IN (...)
    → DB 다건 삭제

11. submitdone
    → 삭제 결과 확인

12. scwin.search()
    → 고객목록 재조회
```

가장 중요한 부분은 다음 네 가지이다.

```text
체크된 행을 정확히 추출한다.

삭제에는 고유한 customerId만 전달한다.

다건 삭제는 Transaction 단위로 처리한다.

삭제 후 서버 데이터를 다시 조회한다.
```

---

## 함께 보면 좋은 내용

```text
/websquare/create/
    └─ 고객 단건 등록

/websquare/create/multi/
    └─ 고객 다건 등록

/websquare/delete/
    └─ 고객 단건 삭제

/websquare/delete/multi/
    └─ 고객 다건 삭제
```
