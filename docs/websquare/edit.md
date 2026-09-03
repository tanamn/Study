---
layout: default
title: WebSquare 상세정보 수정 및 저장 학습 정리
permalink: /websquare/edit/
---

{% include navigation.html %}

# WebSquare 상세정보 수정 및 저장 학습 정리

> 고객 목록에서 선택한 고객의 상세정보를 수정하고, 저장 Submission을 통해 Spring·MyBatis의 UPDATE 처리까지 연결하는 전체 흐름을 정리한 문서입니다.

이 문서는 [WebSquare 학습 정리]({{ '/websquare/' | relative_url }})의 **상세정보 수정** 단계입니다.

---

## 1. 이번 학습의 목표

앞에서 학습한 고객 상세조회 다음 단계입니다.

```text
고객 상세조회
    ↓
상세정보 화면 표시
    ↓
사용자가 연락처·주소·직업 수정
    ↓
입력값 검증
    ↓
저장 Submission 실행
    ↓
Spring Controller
    ↓
Service의 트랜잭션 처리
    ↓
MyBatis UPDATE
    ↓
저장 결과 반환
    ↓
상세정보와 목록 재조회
```

이번 문서에서 사용하는 주요 객체는 다음과 같습니다.

| 구분 | ID | 역할 |
|---|---|---|
| 상세 DataMap | `dma_customerDetail` | 조회된 고객정보와 사용자가 수정한 값 저장 |
| 저장결과 DataMap | `dma_saveResult` | 저장 성공 여부와 메시지 수신 |
| 저장 Submission | `sbm_saveCustomer` | 수정된 고객정보를 서버에 전달 |
| 상세조회 Submission | `sbm_searchCustomerDetail` | 저장 후 최신 상세정보 재조회 |
| 목록조회 Submission | `sbm_searchCustomer` | 저장 후 고객목록 재조회 |

---

## 2. 상세화면과 DataMap 바인딩

고객 상세정보는 한 명의 데이터이므로 `DataMap`을 사용합니다.

```text
dma_customerDetail

- customerId
- customerName
- birthDate
- phone
- address
- job
```

상세화면의 입력 컴포넌트와 각 항목을 바인딩합니다.

```text
ibx_customerId   ↔ dma_customerDetail.customerId
ibx_customerName ↔ dma_customerDetail.customerName
ibx_birthDate    ↔ dma_customerDetail.birthDate
ibx_phone        ↔ dma_customerDetail.phone
ibx_address      ↔ dma_customerDetail.address
ibx_job          ↔ dma_customerDetail.job
```

상세조회 결과가 다음과 같다고 가정하겠습니다.

```json
{
    "customerId": "C001",
    "customerName": "김철수",
    "birthDate": "1985-01-01",
    "phone": "010-1111-2222",
    "address": "서울특별시 강남구",
    "job": "회사원"
}
```

이 결과가 `dma_customerDetail`에 들어오면 바인딩된 입력 컴포넌트에 자동으로 표시됩니다.

---

## 3. 조회 전용 항목과 수정 가능 항목 구분

모든 값을 수정할 수 있게 만드는 것은 적절하지 않습니다.

| 항목 | 수정 여부 | 이유 |
|---|---:|---|
| 고객번호 | 불가 | 고객 식별값이므로 변경하지 않음 |
| 고객명 | 업무 기준에 따라 결정 | 실명 변경 절차가 별도로 필요할 수 있음 |
| 생년월일 | 일반적으로 불가 | 주요 식별정보이므로 별도 검증 필요 |
| 연락처 | 가능 | 고객 요청에 따라 변경 가능 |
| 주소 | 가능 | 고객 요청에 따라 변경 가능 |
| 직업 | 가능 | 고객정보 갱신 대상으로 사용 가능 |

예를 들어 고객번호 입력 컴포넌트는 읽기 전용으로 설정합니다.

```javascript
ibx_customerId.setReadOnly(true);
```

컴포넌트의 읽기 전용 설정 방식은 WebSquare 버전이나 프로젝트 공통함수에 따라 다를 수 있으므로 기존 화면의 구현 방법을 우선 확인합니다.

---

## 4. 사용자가 값을 수정하면 어떻게 되는가

Input과 DataMap이 양방향 바인딩되어 있다면 사용자가 화면 값을 변경했을 때 DataMap 값도 함께 변경됩니다.

수정 전:

```javascript
dma_customerDetail = {
    customerId: "C001",
    phone: "010-1111-2222",
    address: "서울특별시 강남구",
    job: "회사원"
};
```

사용자가 연락처와 주소를 변경한 후:

```javascript
dma_customerDetail = {
    customerId: "C001",
    phone: "010-9999-8888",
    address: "서울특별시 송파구",
    job: "회사원"
};
```

바인딩을 사용하지 않는 화면이라면 저장 전에 컴포넌트 값을 직접 DataMap에 넣어야 합니다.

```javascript
dma_customerDetail.set("phone", ibx_phone.getValue());
dma_customerDetail.set("address", ibx_address.getValue());
dma_customerDetail.set("job", ibx_job.getValue());
```

가능하면 화면 컴포넌트마다 값을 일일이 복사하기보다 DataMap 바인딩을 일관되게 사용하는 것이 관리하기 쉽습니다.

---

## 5. 수정 여부 확인

저장 버튼을 눌렀지만 아무 값도 변경되지 않았다면 서버를 호출하지 않아도 됩니다.

WebSquare DataMap은 원본과 현재 값이 다른 항목을 확인하는 API를 제공합니다.

```javascript
var modifiedIndexes = dma_customerDetail.getModifiedIndex();

if (modifiedIndexes.length === 0) {
    alert("변경된 내용이 없습니다.");
    return;
}
```

변경된 key만 확인할 수도 있습니다.

```javascript
var modifiedKeys = dma_customerDetail.getModifiedKey();
console.log(modifiedKeys);
```

예상 결과:

```javascript
["phone", "address"]
```

변경된 값만 JSON으로 확인하는 방법도 있습니다.

```javascript
var modifiedData = dma_customerDetail.getModifiedJSON();
console.log(modifiedData);
```

예상 결과:

```json
{
    "phone": "010-9999-8888",
    "address": "서울특별시 송파구"
}
```

> 변경 감지 결과는 DataMap의 초기화 방식과 `firstSet` 설정 등의 영향을 받을 수 있습니다. 실제 프로젝트에서는 상세조회 완료 후 DataMap의 원본 상태가 어떻게 관리되는지 확인해야 합니다.

---

## 6. 저장 전 입력값 검증

프론트엔드 검증은 사용자에게 빠르게 오류를 알려주는 역할을 합니다. 서버에서도 동일한 핵심 검증을 다시 수행해야 합니다.

### 6.1 필수값 검증

```javascript
scwin.validateCustomer = function() {
    var phone = dma_customerDetail.get("phone");
    var address = dma_customerDetail.get("address");

    if (!phone) {
        alert("연락처를 입력해주세요.");
        ibx_phone.focus();
        return false;
    }

    if (!address) {
        alert("주소를 입력해주세요.");
        ibx_address.focus();
        return false;
    }

    return true;
};
```

### 6.2 연락처 형식 검증

```javascript
scwin.isValidPhone = function(phone) {
    var phonePattern = /^01[016789]-\d{3,4}-\d{4}$/;
    return phonePattern.test(phone);
};
```

저장 검증 함수에서 사용합니다.

```javascript
if (!scwin.isValidPhone(phone)) {
    alert("연락처 형식을 확인해주세요.");
    ibx_phone.focus();
    return false;
}
```

프로젝트가 숫자만 저장하는지, 하이픈을 포함해 저장하는지에 따라 정규식과 데이터 정규화 규칙을 맞춰야 합니다.

---

## 7. 저장 Submission 만들기

저장 Submission을 다음과 같이 구성합니다.

```text
ID:        sbm_saveCustomer
Action:    /api/customer/update
Method:    POST
Reference: dma_customerDetail
Target:    dma_saveResult
Mode:      asynchronous
```

역할은 다음과 같습니다.

```text
dma_customerDetail
        ↓ Reference
sbm_saveCustomer
        ↓ POST /api/customer/update
Spring Controller
        ↓ 저장 결과
dma_saveResult
```

저장결과 DataMap은 다음과 같이 구성할 수 있습니다.

```text
dma_saveResult

- success
- message
- updatedCount
```

응답 예시:

```json
{
    "success": true,
    "message": "고객정보가 저장되었습니다.",
    "updatedCount": 1
}
```

프로젝트에 공통 응답 형식이 있다면 `success`, `message` 등을 새로 정의하지 않고 공통 규격을 따라야 합니다.

---

## 8. 저장 버튼 이벤트

저장 버튼 ID를 다음과 같이 가정하겠습니다.

```text
btn_save
```

기본 저장 코드는 다음과 같습니다.

```javascript
scwin.btn_save_onclick = function(e) {
    if (!scwin.validateCustomer()) {
        return;
    }

    var modifiedIndexes = dma_customerDetail.getModifiedIndex();

    if (modifiedIndexes.length === 0) {
        alert("변경된 내용이 없습니다.");
        return;
    }

    if (!confirm("변경한 고객정보를 저장하시겠습니까?")) {
        return;
    }

    $p.executeSubmission(sbm_saveCustomer);
};
```

처리 순서는 다음과 같습니다.

```text
저장 버튼 클릭
    ↓
필수값·형식 검증
    ↓
변경된 값이 있는지 확인
    ↓
저장 확인 메시지
    ↓
저장 Submission 실행
```

금융권 프로젝트에서는 공통함수를 적용해 다음처럼 보일 수도 있습니다.

```javascript
scwin.btn_save_onclick = function(e) {
    if (!scwin.validateCustomer()) {
        return;
    }

    if (dma_customerDetail.getModifiedIndex().length === 0) {
        com.alert("변경된 내용이 없습니다.");
        return;
    }

    com.confirm("저장하시겠습니까?", function(result) {
        if (result) {
            com.sbm.execute(sbm_saveCustomer);
        }
    });
};
```

`com.alert`, `com.confirm`, `com.sbm.execute`는 WebSquare 기본 API가 아니라 프로젝트 공통 라이브러리일 수 있습니다.

---

## 9. 서버에 전달되는 Request

Submission의 Reference가 `dma_customerDetail`이므로 서버에는 다음과 같은 데이터가 전달됩니다.

```json
{
    "customerId": "C001",
    "customerName": "김철수",
    "birthDate": "1985-01-01",
    "phone": "010-9999-8888",
    "address": "서울특별시 송파구",
    "job": "회사원"
}
```

화면에서 고객번호를 읽기 전용으로 설정했더라도 서버는 전달된 값을 그대로 신뢰하면 안 됩니다. 로그인 사용자 권한과 수정 대상 고객을 서버에서 다시 확인해야 합니다.

---

## 10. 수정 요청 DTO

저장 용도에 맞는 DTO를 별도로 만드는 것이 좋습니다.

```java
@Getter
@Setter
public class CustomerUpdateDto {

    @NotBlank(message = "고객번호는 필수입니다.")
    private String customerId;

    @NotBlank(message = "연락처는 필수입니다.")
    private String phone;

    @NotBlank(message = "주소는 필수입니다.")
    private String address;

    private String job;
}
```

조회 DTO 전체를 그대로 UPDATE에 사용하는 것보다 수정 가능한 항목만 명시한 DTO가 안전하고 이해하기 쉽습니다.

```text
상세조회 DTO
  → 화면에 보여줄 모든 정보

수정요청 DTO
  → 실제로 수정할 수 있는 정보만 포함
```

---

## 11. Spring Controller

```java
@RestController
@RequestMapping("/api/customer")
@RequiredArgsConstructor
public class CustomerController {

    private final CustomerService customerService;

    @PostMapping("/update")
    public CustomerUpdateResponse updateCustomer(
            @Valid @RequestBody CustomerUpdateDto updateDto) {

        int updatedCount = customerService.updateCustomer(updateDto);

        return new CustomerUpdateResponse(
            true,
            "고객정보가 저장되었습니다.",
            updatedCount
        );
    }
}
```

응답 DTO 예시:

```java
@Getter
@AllArgsConstructor
public class CustomerUpdateResponse {
    private boolean success;
    private String message;
    private int updatedCount;
}
```

실제 프로젝트에 공통 응답 객체가 있다면 다음과 같은 형태가 될 수 있습니다.

```java
return ApiResponse.success("고객정보가 저장되었습니다.");
```

---

## 12. Service와 트랜잭션

```java
@Service
@RequiredArgsConstructor
public class CustomerService {

    private final CustomerMapper customerMapper;

    @Transactional
    public int updateCustomer(CustomerUpdateDto updateDto) {
        int updatedCount = customerMapper.updateCustomer(updateDto);

        if (updatedCount != 1) {
            throw new IllegalStateException(
                "수정할 고객정보를 찾을 수 없습니다."
            );
        }

        return updatedCount;
    }
}
```

`@Transactional`은 저장 도중 오류가 발생했을 때 해당 트랜잭션의 DB 변경을 롤백할 수 있게 합니다.

예를 들어 고객정보 수정과 변경이력 등록을 함께 처리한다면 두 작업을 하나의 트랜잭션으로 묶을 수 있습니다.

```java
@Transactional
public int updateCustomer(CustomerUpdateDto updateDto) {
    int updatedCount = customerMapper.updateCustomer(updateDto);

    if (updatedCount != 1) {
        throw new IllegalStateException("고객정보 수정에 실패했습니다.");
    }

    customerMapper.insertCustomerHistory(updateDto);

    return updatedCount;
}
```

```text
고객정보 UPDATE 성공
    ↓
변경이력 INSERT 실패
    ↓
전체 작업 롤백
```

---

## 13. MyBatis Mapper

Mapper 인터페이스:

```java
@Mapper
public interface CustomerMapper {
    int updateCustomer(CustomerUpdateDto updateDto);
}
```

Mapper XML:

```xml
<update id="updateCustomer">
    UPDATE CUSTOMER
       SET PHONE       = #{phone}
         , ADDRESS     = #{address}
         , JOB         = #{job}
         , UPDATED_AT  = CURRENT_TIMESTAMP
     WHERE CUSTOMER_ID = #{customerId}
</update>
```

MyBatis의 UPDATE 반환값은 일반적으로 영향을 받은 행의 수입니다.

```text
1 → 고객 한 건 수정 성공
0 → 해당 고객이 없거나 수정되지 않음
2 이상 → 고객번호 조건이나 데이터 무결성 확인 필요
```

고객번호처럼 식별에 사용하는 값은 SQL 문자열 연결이 아니라 `#{customerId}` 형태의 Parameter Binding을 사용합니다.

---

## 14. 동적 UPDATE를 사용할 때

수정된 항목만 UPDATE해야 하는 업무라면 MyBatis 동적 SQL을 사용할 수 있습니다.

```xml
<update id="updateCustomer">
    UPDATE CUSTOMER
    <set>
        <if test="phone != null">
            PHONE = #{phone},
        </if>
        <if test="address != null">
            ADDRESS = #{address},
        </if>
        <if test="job != null">
            JOB = #{job},
        </if>
        UPDATED_AT = CURRENT_TIMESTAMP
    </set>
    WHERE CUSTOMER_ID = #{customerId}
</update>
```

다만 빈 문자열을 `null`과 동일하게 볼 것인지, 사용자가 값을 의도적으로 지운 것인지 구분해야 합니다.

```text
null        → 수정 요청에 포함하지 않음
빈 문자열 "" → 값을 비우려는 요청일 수 있음
```

따라서 동적 UPDATE를 적용할 때는 화면·DTO·SQL의 수정 규칙을 먼저 정해야 합니다.

---

## 15. 저장 성공 후 처리

Submission이 정상 완료되면 결과를 확인하고 화면 데이터를 다시 조회합니다.

```javascript
scwin.sbm_saveCustomer_submitdone = function(e) {
    var success = dma_saveResult.get("success");

    if (!success) {
        alert(dma_saveResult.get("message"));
        return;
    }

    alert("고객정보가 저장되었습니다.");

    $p.executeSubmission(sbm_searchCustomerDetail);
    $p.executeSubmission(sbm_searchCustomer);
};
```

저장 후 재조회하는 이유는 다음과 같습니다.

- DB에 최종 저장된 값을 화면에 다시 반영
- 서버가 변환하거나 보정한 값 반영
- 수정 상태를 조회 완료 상태로 초기화
- 고객목록에도 변경된 정보 반영

목록과 상세조회 순서가 중요한 경우에는 두 Submission을 동시에 실행하지 않고, 상세조회 완료 후 목록조회를 실행하는 등 프로젝트 규칙에 맞게 순서를 제어합니다.

---

## 16. 저장 오류 처리

```javascript
scwin.sbm_saveCustomer_submiterror = function(e) {
    alert("고객정보 저장 중 오류가 발생했습니다.");
};
```

실무에서는 서버 오류 메시지를 그대로 사용자에게 노출하지 않고 공통 오류처리를 사용하는 경우가 많습니다.

```text
사용자 메시지
→ "고객정보 저장 중 오류가 발생했습니다."

서버 로그
→ 예외 종류, SQL 오류, 요청 ID 등 상세내용 기록
```

저장 버튼의 중복 클릭을 막기 위해 처리 중에는 버튼을 비활성화하고, 완료 또는 오류 시 다시 활성화할 수도 있습니다.

```javascript
scwin.executeSave = function() {
    btn_save.setDisabled(true);
    $p.executeSubmission(sbm_saveCustomer);
};

scwin.sbm_saveCustomer_submitdone = function(e) {
    btn_save.setDisabled(false);
    alert("고객정보가 저장되었습니다.");
};

scwin.sbm_saveCustomer_submiterror = function(e) {
    btn_save.setDisabled(false);
    alert("고객정보 저장 중 오류가 발생했습니다.");
};
```

컴포넌트 활성화·비활성화 API 이름은 WebSquare 버전과 프로젝트 공통 컴포넌트에 따라 확인해야 합니다.

---

## 17. 동시 수정 문제

사용자 A와 사용자 B가 같은 고객정보를 동시에 조회한 뒤 각각 수정할 수 있습니다.

```text
10:00 사용자 A 상세조회
10:01 사용자 B 상세조회
10:02 사용자 A 주소 수정 및 저장
10:03 사용자 B 연락처 수정 및 저장
```

단순 UPDATE에서는 사용자 B의 저장이 사용자 A의 변경을 덮어쓸 수 있습니다. 중요한 업무라면 버전값이나 최종수정일시를 이용한 낙관적 잠금을 고려합니다.

조회 응답에 버전을 포함합니다.

```json
{
    "customerId": "C001",
    "phone": "010-1111-2222",
    "version": 3
}
```

UPDATE 조건에 버전을 추가합니다.

```xml
<update id="updateCustomer">
    UPDATE CUSTOMER
       SET PHONE   = #{phone}
         , ADDRESS = #{address}
         , VERSION = VERSION + 1
     WHERE CUSTOMER_ID = #{customerId}
       AND VERSION = #{version}
</update>
```

UPDATE 결과가 0건이면 다른 사용자가 먼저 수정했을 가능성이 있으므로 다음과 같이 안내할 수 있습니다.

```text
다른 사용자가 먼저 고객정보를 변경했습니다.
최신 정보를 다시 조회한 후 수정해주세요.
```

단순 학습 예제에서는 생략할 수 있지만 금융권과 관리자 화면에서는 중요한 개념입니다.

---

## 18. 전체 프론트엔드 코드

```javascript
scwin.validateCustomer = function() {
    var phone = dma_customerDetail.get("phone");
    var address = dma_customerDetail.get("address");
    var phonePattern = /^01[016789]-\d{3,4}-\d{4}$/;

    if (!phone) {
        alert("연락처를 입력해주세요.");
        ibx_phone.focus();
        return false;
    }

    if (!phonePattern.test(phone)) {
        alert("연락처 형식을 확인해주세요.");
        ibx_phone.focus();
        return false;
    }

    if (!address) {
        alert("주소를 입력해주세요.");
        ibx_address.focus();
        return false;
    }

    return true;
};

scwin.btn_save_onclick = function(e) {
    if (!scwin.validateCustomer()) {
        return;
    }

    if (dma_customerDetail.getModifiedIndex().length === 0) {
        alert("변경된 내용이 없습니다.");
        return;
    }

    if (!confirm("변경한 고객정보를 저장하시겠습니까?")) {
        return;
    }

    btn_save.setDisabled(true);
    $p.executeSubmission(sbm_saveCustomer);
};

scwin.sbm_saveCustomer_submitdone = function(e) {
    btn_save.setDisabled(false);

    if (!dma_saveResult.get("success")) {
        alert(dma_saveResult.get("message"));
        return;
    }

    alert("고객정보가 저장되었습니다.");

    $p.executeSubmission(sbm_searchCustomerDetail);
    $p.executeSubmission(sbm_searchCustomer);
};

scwin.sbm_saveCustomer_submiterror = function(e) {
    btn_save.setDisabled(false);
    alert("고객정보 저장 중 오류가 발생했습니다.");
};
```

---

## 19. 전체 백엔드 코드

### Controller

```java
@RestController
@RequestMapping("/api/customer")
@RequiredArgsConstructor
public class CustomerController {

    private final CustomerService customerService;

    @PostMapping("/update")
    public CustomerUpdateResponse updateCustomer(
            @Valid @RequestBody CustomerUpdateDto updateDto) {

        int updatedCount = customerService.updateCustomer(updateDto);

        return new CustomerUpdateResponse(
            true,
            "고객정보가 저장되었습니다.",
            updatedCount
        );
    }
}
```

### Service

```java
@Service
@RequiredArgsConstructor
public class CustomerService {

    private final CustomerMapper customerMapper;

    @Transactional
    public int updateCustomer(CustomerUpdateDto updateDto) {
        int updatedCount = customerMapper.updateCustomer(updateDto);

        if (updatedCount != 1) {
            throw new IllegalStateException(
                "수정할 고객정보를 찾을 수 없습니다."
            );
        }

        return updatedCount;
    }
}
```

### Mapper

```java
@Mapper
public interface CustomerMapper {
    int updateCustomer(CustomerUpdateDto updateDto);
}
```

### Mapper XML

```xml
<update id="updateCustomer">
    UPDATE CUSTOMER
       SET PHONE       = #{phone}
         , ADDRESS     = #{address}
         , JOB         = #{job}
         , UPDATED_AT  = CURRENT_TIMESTAMP
     WHERE CUSTOMER_ID = #{customerId}
</update>
```

---

## 20. React와 비교

React에서는 다음과 같은 형태가 될 수 있습니다.

```javascript
const saveCustomer = async () => {
    const response = await axios.post(
        "/api/customer/update",
        customerDetail
    );

    if (response.data.success) {
        await searchCustomerDetail(customerDetail.customerId);
        await searchCustomers();
    }
};
```

WebSquare와 대응하면 다음과 같습니다.

| React | WebSquare |
|---|---|
| `customerDetail` state | `dma_customerDetail` |
| Input과 state 연결 | Input과 DataMap Binding |
| `axios.post()` | 저장 Submission |
| `response.data` | Submission Target |
| 저장 성공 후 재조회 함수 | `submitdone`에서 조회 Submission 실행 |

---

## 21. 실무 소스 추적 순서

처음 보는 저장 기능은 다음 순서로 확인합니다.

```text
저장 버튼 ID
    ↓
onclick 이벤트
    ↓
입력값 검증 함수
    ↓
저장 Submission
    ↓
Reference DataMap
    ↓
Submission Action URL
    ↓
Spring Controller
    ↓
Service의 @Transactional
    ↓
Mapper UPDATE SQL
    ↓
submitdone / submiterror
    ↓
저장 후 재조회
```

확인해야 할 질문은 다음과 같습니다.

1. 어느 항목까지 수정할 수 있는가?
2. 저장 전에 어떤 검증을 하는가?
3. 서버로 전체 DataMap을 보내는가, 수정값만 보내는가?
4. Controller URL은 어디인가?
5. Service에 트랜잭션이 적용되어 있는가?
6. UPDATE 결과가 0건일 때 어떻게 처리하는가?
7. 저장 후 상세와 목록을 다시 조회하는가?
8. 중복 클릭과 동시 수정은 어떻게 막는가?

---

## 22. 자주 발생하는 문제

| 증상 | 확인할 내용 |
|---|---|
| 화면을 수정했는데 DataMap 값이 안 바뀜 | Input과 DataMap의 바인딩 설정 확인 |
| 항상 변경된 내용이 없다고 나옴 | DataMap 원본 상태와 `firstSet` 설정 확인 |
| 저장 요청값이 비어 있음 | Submission Reference와 DataMap ID 확인 |
| UPDATE 결과가 0건 | `customerId`, WHERE 조건, 동시 수정 여부 확인 |
| 저장은 됐지만 화면은 이전 값 | 저장 성공 후 상세조회 실행 여부 확인 |
| 저장 버튼을 여러 번 눌러 중복 처리 | 처리 중 버튼 비활성화 또는 서버 중복 방지 확인 |
| 일부 저장 후 오류 발생 | Service의 `@Transactional` 적용 여부 확인 |
| 다른 사용자의 변경값이 사라짐 | 버전값을 이용한 동시 수정 제어 검토 |

---

## 23. 이번 학습의 핵심 코드

### 수정값은 DataMap에 반영

```javascript
dma_customerDetail.set("phone", "010-9999-8888");
```

### 변경 여부 확인

```javascript
dma_customerDetail.getModifiedIndex().length
```

### 저장 실행

```javascript
$p.executeSubmission(sbm_saveCustomer);
```

### 서버 UPDATE

```xml
UPDATE CUSTOMER
   SET PHONE = #{phone}
 WHERE CUSTOMER_ID = #{customerId}
```

### 저장 후 재조회

```javascript
$p.executeSubmission(sbm_searchCustomerDetail);
$p.executeSubmission(sbm_searchCustomer);
```

한 줄로 정리하면 다음과 같습니다.

```text
DataMap 수정 → 검증 → 저장 Submission → @Transactional → UPDATE → 재조회
```

---

## 24. GitHub Pages에 현재 문서 추가하기

이 파일을 Repository의 `docs/websquare` 폴더에 `edit.md`라는 이름으로 복사합니다.

```text
Study
└─ docs
   ├─ _includes
   │  └─ navigation.html
   ├─ index.md
   ├─ websquare.md
   └─ websquare
      ├─ edit.md
      ├─ create.md
      ├─ delete.md
      ├─ validation.md
      ├─ common.md
      └─ transaction.md
```

현재 문서를 복사하는 PowerShell 명령은 다음과 같습니다.

```powershell
New-Item -ItemType Directory -Force ".\docs\websquare"

Copy-Item `
  ".\WebSquare_상세정보_수정_저장_학습정리.md" `
  ".\docs\websquare\edit.md"
```

그다음 GitHub에 반영합니다.

```powershell
git add docs
git commit -m "WebSquare 하위 메뉴 및 상세정보 수정 문서 추가"
git push origin main
```

Pages 배포가 끝나면 다음 경로로 접근할 수 있습니다.

```text
https://사용자이름.github.io/Study/websquare/edit/
```

문서 상단의 다음 코드가 공통 메뉴를 표시합니다.

```liquid
{% raw %}
{% include navigation.html %}
{% endraw %}
```

새 문서를 만들 때도 YAML 머리말 아래에 같은 코드를 넣어야 합니다.

---

## 25. 다음 학습 단계

이번 수정·저장 흐름 다음에는 신규 고객 등록을 학습하면 좋습니다.

```text
신규 버튼
    ↓
DataMap 초기화
    ↓
신규 고객정보 입력
    ↓
입력값 검증
    ↓
등록 Submission
    ↓
Spring INSERT
    ↓
등록 결과 처리
    ↓
목록 재조회
```

그다음 삭제까지 연결하면 고객관리 CRUD가 완성됩니다.

```text
Create → 신규 등록
Read   → 목록·상세 조회
Update → 상세 수정·저장
Delete → 고객 삭제
```

---

[← WebSquare 목차로 이동]({{ '/websquare/' | relative_url }}) · [다음 학습: 신규 고객 등록 →]({{ '/websquare/create/' | relative_url }})

## 참고 자료

- [WebSquare DataMap API](https://docs.inswave.com/support/api/ws5_sp4/5.0_4.4971A.20230726.110743/WebSquare.uiplugin.dataMap/WebSquare.uiplugin.dataMap.html)
