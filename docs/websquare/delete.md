---
layout: default
title: WebSquare 고객 삭제
permalink: /websquare/delete/
---

{% include navigation.html %}

# 고객 삭제

WebSquare 화면에서 조회된 고객을 선택한 후 삭제하는 기능을 구현해 본다.

이번 학습에서는 다음 흐름을 기준으로 구현한다.

> 고객 선택 → 삭제 확인 → 고객 ID 전송 → Spring Controller → Service → Mapper → DB 삭제 → 목록 재조회

---

## 1. 학습 목표

이번 학습에서는 다음 내용을 익힌다.

- GridView에서 선택된 행 확인
- DataList에서 선택 고객의 ID 조회
- 삭제 전 `confirm()`을 이용한 사용자 확인
- DataMap에 삭제 대상 고객 ID 설정
- Submission을 이용한 서버 요청
- Spring에서 삭제 요청 처리
- Service에서 삭제 결과 검증
- MyBatis Mapper를 이용한 `DELETE` SQL 실행
- 삭제 완료 후 고객 목록 재조회

---

## 2. 삭제 처리 흐름

고객 삭제는 화면에서 행을 바로 제거하는 것만으로 끝나지 않는다.

실제 데이터베이스의 고객정보를 삭제해야 하므로 서버와 통신해야 한다.

```text
[WebSquare]

고객목록에서 행 선택
        ↓
삭제 버튼 클릭
        ↓
선택 여부 확인
        ↓
삭제 확인 메시지
        ↓
customerId를 DataMap에 저장
        ↓
Submission 실행
        ↓
[Spring]

Controller
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

완료 메시지
        ↓
고객목록 재조회
```

---

## 3. 삭제에서 고객 ID가 중요한 이유

고객을 삭제할 때 이름이나 연락처보다 **고객 ID와 같은 고유 식별값**을 사용하는 것이 좋다.

예를 들어 고객 테이블이 다음과 같다고 가정한다.

| customerId | name | phone |
|---|---|---|
| C000001 | 김민수 | 010-1111-2222 |
| C000002 | 이영희 | 010-3333-4444 |

`name`은 동일한 사람이 있을 수 있고 `phone`도 변경될 수 있다.

따라서 삭제 대상은 다음과 같이 고객의 고유 ID로 지정한다.

```sql
DELETE
  FROM CUSTOMER
 WHERE CUSTOMER_ID = 'C000001';
```

실제 프로그램에서는 SQL에 값을 직접 작성하지 않고 Parameter Binding을 사용한다.

---

# 4. WebSquare 화면 구성

고객목록 GridView와 삭제 버튼이 있다고 가정한다.

```text
┌──────────────────────────────────────────────┐
│ 고객목록                                     │
├──────────┬──────────┬────────────────────────┤
│ 고객ID   │ 고객명   │ 연락처                 │
├──────────┼──────────┼────────────────────────┤
│ C000001  │ 김민수   │ 010-1111-2222          │
│ C000002  │ 이영희   │ 010-3333-4444          │
└──────────┴──────────┴────────────────────────┘

                                      [삭제]
```

예제에서 사용하는 객체는 다음과 같다.

| 구분 | ID | 설명 |
|---|---|---|
| GridView | `grd_customer` | 고객목록 화면 |
| DataList | `dlt_customer` | 고객목록 데이터 |
| DataMap | `dma_customerDelete` | 삭제 요청 데이터 |
| DataMap | `dma_customerDeleteResult` | 삭제 결과 데이터 |
| Submission | `sbm_customerDelete` | 삭제 요청 |

---

# 5. 삭제 요청 DataMap

서버에는 삭제할 고객의 `customerId`만 전달한다.

예를 들어 다음과 같은 DataMap을 구성할 수 있다.

```xml
<w2:dataMap id="dma_customerDelete">
    <w2:keyInfo>
        <w2:key id="customerId" name="고객ID" dataType="text"/>
    </w2:keyInfo>
</w2:dataMap>
```

삭제 결과를 받을 DataMap도 준비한다.

```xml
<w2:dataMap id="dma_customerDeleteResult">
    <w2:keyInfo>
        <w2:key id="success" name="처리결과" dataType="boolean"/>
        <w2:key id="message" name="처리메시지" dataType="text"/>
    </w2:keyInfo>
</w2:dataMap>
```

삭제 요청 데이터는 서버에 다음과 같은 JSON 형태로 전달된다.

```json
{
  "customerId": "C000001"
}
```

---

# 6. Submission 구성

학습 예제에서는 WebSquare에서 `POST` 방식으로 삭제 요청을 보낸다.

```xml
<w2:submission
    id="sbm_customerDelete"
    action="/api/customers/delete"
    method="post"
    mediatype="application/json"
    ref="data:json,dma_customerDelete"
    target="data:json,dma_customerDeleteResult"
    ev:submitdone="scwin.sbm_customerDelete_submitdone"
    ev:submiterror="scwin.sbm_customerDelete_submiterror">
</w2:submission>
```

각 속성의 의미는 다음과 같다.

| 속성 | 설명 |
|---|---|
| `id` | Submission ID |
| `action` | 요청할 서버 URL |
| `method` | HTTP 요청 방식 |
| `mediatype` | 요청 데이터 형식 |
| `ref` | 서버로 전달할 데이터 |
| `target` | 서버 응답을 저장할 데이터 |
| `submitdone` | 정상 응답 시 실행할 함수 |
| `submiterror` | 통신 오류 시 실행할 함수 |

> 프로젝트의 공통 Submission 모듈이나 API 규칙이 있다면 해당 규칙을 우선 적용한다.

---

# 7. 삭제 버튼 처리

GridView에서 현재 선택된 행을 확인한다.

```javascript
scwin.btn_delete_onclick = function() {

    var rowIndex = grd_customer.getFocusedRowIndex();

    if (rowIndex < 0) {
        alert("삭제할 고객을 선택해 주세요.");
        return;
    }

    var customerId = dlt_customer.getCellData(
        rowIndex,
        "customerId"
    );

    var customerName = dlt_customer.getCellData(
        rowIndex,
        "name"
    );

    var result = confirm(
        customerName + " 고객을 삭제하시겠습니까?"
    );

    if (!result) {
        return;
    }

    dma_customerDelete.set(
        "customerId",
        customerId
    );

    $p.executeSubmission(
        "sbm_customerDelete"
    );
};
```

전체 코드를 단계별로 살펴보자.

---

## 7.1 선택된 행 확인

```javascript
var rowIndex = grd_customer.getFocusedRowIndex();
```

`getFocusedRowIndex()`는 현재 GridView에서 포커스가 있는 행의 인덱스를 반환한다.

첫 번째 행은 `0`부터 시작한다.

```text
첫 번째 행 → 0
두 번째 행 → 1
세 번째 행 → 2
```

선택된 행이 없으면 `-1`이 반환될 수 있으므로 먼저 확인한다.

```javascript
if (rowIndex < 0) {
    alert("삭제할 고객을 선택해 주세요.");
    return;
}
```

---

## 7.2 선택 고객의 정보 조회

선택된 행 번호를 이용해 DataList에서 데이터를 가져온다.

```javascript
var customerId = dlt_customer.getCellData(
    rowIndex,
    "customerId"
);
```

예를 들어 첫 번째 고객을 선택했다면 다음과 같다.

```text
rowIndex = 0

dlt_customer

0 → C000001 / 김민수
1 → C000002 / 이영희
```

따라서 다음 코드의 결과는

```javascript
dlt_customer.getCellData(
    0,
    "customerId"
);
```

다음 값이 된다.

```text
C000001
```

---

## 7.3 삭제 확인

삭제는 데이터를 실제로 제거하는 작업이므로 사용자가 실수로 버튼을 누르는 상황을 방지하는 것이 좋다.

```javascript
var result = confirm(
    customerName + " 고객을 삭제하시겠습니까?"
);
```

사용자가 **확인**을 누르면

```javascript
true
```

취소를 누르면

```javascript
false
```

가 반환된다.

따라서 다음과 같이 처리할 수 있다.

```javascript
if (!result) {
    return;
}
```

취소를 선택했다면 이후 삭제 로직을 실행하지 않는다.

---

## 7.4 DataMap에 customerId 저장

```javascript
dma_customerDelete.set(
    "customerId",
    customerId
);
```

예를 들어

```javascript
customerId = "C000001";
```

이라면 DataMap은 다음과 같은 상태가 된다.

```text
dma_customerDelete
└─ customerId : C000001
```

즉,

```javascript
dma_customerDelete.set(
    "customerId",
    customerId
);
```

는

> `dma_customerDelete`의 `customerId`라는 Key에 `customerId` 변수의 값을 저장한다.

라고 이해하면 된다.

---

## 7.5 Submission 실행

```javascript
$p.executeSubmission(
    "sbm_customerDelete"
);
```

이 코드가 실행되면 WebSquare가 `sbm_customerDelete`에 정의된 설정에 따라 서버로 요청을 보낸다.

```text
dma_customerDelete
        ↓
{
    "customerId": "C000001"
}
        ↓
POST /api/customers/delete
        ↓
Spring Controller
```

---

# 8. Spring Controller

서버에서는 WebSquare에서 전달한 삭제 요청을 받는다.

```java
@RestController
@RequestMapping("/api/customers")
@RequiredArgsConstructor
public class CustomerController {

    private final CustomerService customerService;

    @PostMapping("/delete")
    public CustomerDeleteResponse deleteCustomer(
            @Valid @RequestBody CustomerDeleteRequest request) {

        customerService.deleteCustomer(
                request.getCustomerId()
        );

        return new CustomerDeleteResponse(
                true,
                "고객정보가 삭제되었습니다."
        );
    }
}
```

처리 흐름은 다음과 같다.

```text
WebSquare
   ↓
POST /api/customers/delete
   ↓
@RequestBody
   ↓
CustomerDeleteRequest
   ↓
customerService.deleteCustomer()
```

---

# 9. 삭제 요청 DTO

삭제 요청으로 전달받을 객체를 작성한다.

```java
@Getter
@Setter
public class CustomerDeleteRequest {

    @NotBlank
    private String customerId;
}
```

`@NotBlank`를 사용하면 고객 ID가 없는 요청을 Validation 단계에서 차단할 수 있다.

잘못된 요청

```json
{
  "customerId": ""
}
```

정상 요청

```json
{
  "customerId": "C000001"
}
```

---

# 10. 삭제 응답 DTO

```java
@Getter
@AllArgsConstructor
public class CustomerDeleteResponse {

    private boolean success;
    private String message;
}
```

정상적으로 삭제되면 다음과 같은 응답을 전달할 수 있다.

```json
{
  "success": true,
  "message": "고객정보가 삭제되었습니다."
}
```

---

# 11. Service

Controller가 직접 Mapper를 호출하지 않고 Service를 거쳐 삭제하도록 구성한다.

```java
@Service
@RequiredArgsConstructor
public class CustomerService {

    private final CustomerMapper customerMapper;

    @Transactional
    public void deleteCustomer(String customerId) {

        int deletedCount =
                customerMapper.deleteCustomer(customerId);

        if (deletedCount == 0) {
            throw new IllegalArgumentException(
                    "삭제할 고객정보가 존재하지 않습니다."
            );
        }
    }
}
```

여기서 중요한 부분은 다음 코드다.

```java
int deletedCount =
        customerMapper.deleteCustomer(customerId);
```

SQL의 `DELETE` 실행 결과로 삭제된 행의 개수를 받는다.

예를 들어

```text
삭제 성공
deletedCount = 1
```

삭제 대상이 없다면

```text
deletedCount = 0
```

이 될 수 있다.

따라서 다음과 같이 삭제 여부를 확인한다.

```java
if (deletedCount == 0) {
    throw new IllegalArgumentException(
            "삭제할 고객정보가 존재하지 않습니다."
    );
}
```

---

# 12. Mapper

MyBatis Mapper를 다음과 같이 작성할 수 있다.

```java
@Mapper
public interface CustomerMapper {

    int deleteCustomer(String customerId);
}
```

삭제 건수를 확인하기 위해 반환형을 `int`로 사용한다.

---

# 13. Mapper XML

```xml
<delete id="deleteCustomer"
        parameterType="string">

    DELETE
      FROM CUSTOMER
     WHERE CUSTOMER_ID = #{customerId}

</delete>
```

`#{customerId}`는 MyBatis의 Parameter Binding이다.

Java에서

```java
customerMapper.deleteCustomer(
    "C000001"
);
```

을 호출하면 최종적으로 다음 조건으로 삭제된다.

```text
CUSTOMER_ID = C000001
```

개념적으로는 다음 SQL과 같다.

```sql
DELETE
  FROM CUSTOMER
 WHERE CUSTOMER_ID = 'C000001';
```

---

# 14. 삭제 완료 처리

서버에서 정상 응답을 받으면 WebSquare의 `submitdone` 이벤트가 실행된다.

```javascript
scwin.sbm_customerDelete_submitdone = function(e) {

    var success =
        dma_customerDeleteResult.get("success");

    var message =
        dma_customerDeleteResult.get("message");

    if (success) {
        alert(message);

        scwin.search();
    }
};
```

여기서

```javascript
scwin.search();
```

는 기존 고객조회 함수를 다시 호출한다는 의미다.

예를 들어 삭제 전 고객목록이

```text
C000001 김민수
C000002 이영희
C000003 박철수
```

이고 `C000002`를 삭제했다면 서버에서 삭제한 후 목록을 다시 조회한다.

```text
C000001 김민수
C000003 박철수
```

화면의 DataList만 임의로 삭제하는 것보다 서버의 최신 데이터를 다시 조회하는 방식이 안전하다.

---

# 15. 통신 오류 처리

Submission 자체가 실패한 경우 오류 메시지를 처리할 수 있다.

```javascript
scwin.sbm_customerDelete_submiterror = function(e) {

    alert(
        "고객정보 삭제 중 오류가 발생했습니다."
    );
};
```

삭제 요청은 다음 두 종류의 실패를 구분해서 생각하는 것이 좋다.

```text
1. 업무 오류
   └─ 삭제 대상 고객이 존재하지 않음

2. 통신/시스템 오류
   └─ 서버 오류
   └─ 네트워크 오류
```

---

# 16. 전체 JavaScript

최종적으로 WebSquare 코드를 모으면 다음과 같다.

```javascript
scwin.btn_delete_onclick = function() {

    var rowIndex =
        grd_customer.getFocusedRowIndex();

    if (rowIndex < 0) {
        alert("삭제할 고객을 선택해 주세요.");
        return;
    }

    var customerId =
        dlt_customer.getCellData(
            rowIndex,
            "customerId"
        );

    var customerName =
        dlt_customer.getCellData(
            rowIndex,
            "name"
        );

    var result = confirm(
        customerName +
        " 고객을 삭제하시겠습니까?"
    );

    if (!result) {
        return;
    }

    dma_customerDelete.set(
        "customerId",
        customerId
    );

    $p.executeSubmission(
        "sbm_customerDelete"
    );
};


scwin.sbm_customerDelete_submitdone = function(e) {

    var success =
        dma_customerDeleteResult.get(
            "success"
        );

    var message =
        dma_customerDeleteResult.get(
            "message"
        );

    if (success) {
        alert(message);

        scwin.search();
    }
};


scwin.sbm_customerDelete_submiterror = function(e) {

    alert(
        "고객정보 삭제 중 오류가 발생했습니다."
    );
};
```

---

# 17. 전체 Spring 처리 구조

```text
CustomerController
        │
        │ customerId
        ▼
CustomerService
        │
        │ deleteCustomer(customerId)
        ▼
CustomerMapper
        │
        ▼
DELETE
  FROM CUSTOMER
 WHERE CUSTOMER_ID = #{customerId}
```

각 계층의 역할은 다음과 같다.

| 계층 | 역할 |
|---|---|
| Controller | WebSquare 요청 수신 |
| DTO | 요청 데이터 전달 |
| Service | 삭제 업무 로직 처리 |
| Mapper | SQL 실행 요청 |
| DB | 실제 고객정보 삭제 |

---

# 18. POST와 DELETE 방식

REST API에서는 다음과 같은 형태도 많이 사용한다.

```http
DELETE /api/customers/C000001
```

Spring에서는 다음과 같이 작성할 수 있다.

```java
@DeleteMapping("/{customerId}")
public CustomerDeleteResponse deleteCustomer(
        @PathVariable String customerId) {

    customerService.deleteCustomer(
            customerId
    );

    return new CustomerDeleteResponse(
            true,
            "고객정보가 삭제되었습니다."
    );
}
```

다만 WebSquare 프로젝트에서는 사용하는 WebSquare 버전, Submission 공통 모듈, 사내 API 표준에 따라 요청 방식이 달라질 수 있다.

따라서 학습 예제에서는 이해하기 쉬운 다음 구조를 사용했다.

```text
POST /api/customers/delete

Body
{
    "customerId": "C000001"
}
```

실제 프로젝트에서는 프로젝트의 API 설계 규칙을 우선 적용한다.

---

# 19. 물리 삭제와 논리 삭제

지금까지 작성한 SQL은 실제 DB Row를 제거하는 **물리 삭제** 방식이다.

```sql
DELETE
  FROM CUSTOMER
 WHERE CUSTOMER_ID = #{customerId};
```

하지만 업무 시스템에서는 데이터를 실제로 제거하지 않고 삭제 여부만 변경하는 **논리 삭제**를 사용하기도 한다.

예를 들어 고객 테이블에 다음 컬럼이 있다고 가정한다.

```text
DELETE_YN
```

삭제 시

```sql
UPDATE CUSTOMER
   SET DELETE_YN = 'Y'
 WHERE CUSTOMER_ID = #{customerId};
```

조회 시에는

```sql
SELECT *
  FROM CUSTOMER
 WHERE DELETE_YN = 'N';
```

와 같이 처리한다.

### 물리 삭제

```text
DB에서 데이터 자체를 제거
```

### 논리 삭제

```text
데이터는 유지
삭제 여부만 Y로 변경
```

금융, 공공, 업무 이력 관리 시스템처럼 데이터 이력이 중요한 시스템에서는 논리 삭제가 사용되는 경우가 많다.

---

# 20. 핵심 정리

고객 삭제의 핵심 흐름은 다음과 같다.

```text
1. GridView에서 고객 선택

2. getFocusedRowIndex()
   → 선택된 행 번호 확인

3. dlt_customer.getCellData()
   → customerId 조회

4. confirm()
   → 삭제 여부 확인

5. dma_customerDelete.set()
   → customerId 저장

6. $p.executeSubmission()
   → Spring으로 삭제 요청

7. Controller
   → 요청 수신

8. Service
   → 삭제 로직 실행 및 삭제 건수 확인

9. Mapper
   → DELETE SQL 실행

10. submitdone
    → 완료 메시지 표시

11. scwin.search()
    → 고객목록 재조회
```

가장 중요한 점은 다음 세 가지다.

```text
삭제 대상은 고유 ID로 식별한다.

삭제 전에 사용자에게 확인을 받는다.

삭제 후에는 서버 데이터를 다시 조회해 화면을 최신 상태로 맞춘다.
```

---

## 다음 학습

고객 삭제 다음에는 다음 내용을 학습하면 CRUD의 전체 흐름을 이해하기 좋다.

```text
고객 조회
   ↓
고객 등록
   ↓
고객 수정
   ↓
고객 삭제
   ↓
다건 처리
```

다건 삭제가 필요한 경우에는 체크박스로 여러 고객을 선택한 후 고객 ID 목록을 서버에 전달하는 방식으로 확장할 수 있다.
