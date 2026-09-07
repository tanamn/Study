---
layout: default
title: WebSquare 입력값 검증
permalink: /websquare/validation/
---

{% include navigation.html %}

# WebSquare 입력값 검증

WebSquare 화면에서 사용자가 입력한 값을 서버로 전송하기 전에  
**필수값, 형식, 길이, 숫자 범위, 중복 여부 등을 확인하는 작업**을 입력값 검증(Validation)이라고 한다.

입력값 검증은 잘못된 데이터가 서버로 전달되는 것을 줄이고,  
사용자에게 입력 오류를 즉시 알려줄 수 있다는 장점이 있다.

---

## 1. 입력값 검증이 필요한 이유

예를 들어 고객 등록 화면에 다음과 같은 입력 항목이 있다고 가정한다.

- 고객명
- 연락처
- 이메일
- 생년월일
- 고객구분

사용자가 다음과 같이 잘못 입력할 수 있다.

```text
고객명   : 입력하지 않음
연락처   : 0101234
이메일   : test@
생년월일 : 20261340
```

이 데이터를 그대로 서버에 전달하면 서버에서 오류가 발생하거나  
잘못된 데이터가 저장될 수 있다.

따라서 일반적으로 다음 순서로 처리한다.

```text
사용자 입력
   ↓
WebSquare 입력값 검증
   ↓
검증 성공
   ↓
Submission 실행
   ↓
Spring Controller
   ↓
서버 Validation
   ↓
DB 저장
```

> 화면 검증만으로는 충분하지 않다.  
> WebSquare에서 검증하더라도 Spring 서버에서도 반드시 다시 검증하는 것이 좋다.

---

# 2. 가장 기본적인 필수값 검증

고객명이 반드시 입력되어야 한다고 가정한다.

```javascript
scwin.validateCustomerName = function() {

    var customerName = ipt_customerName.getValue();

    if (customerName == null || customerName.trim() === "") {
        alert("고객명을 입력해 주세요.");
        ipt_customerName.focus();
        return false;
    }

    return true;
};
```

## 코드 설명

```javascript
var customerName = ipt_customerName.getValue();
```

입력 컴포넌트의 값을 가져온다.

```javascript
customerName == null
```

값 자체가 없는 경우를 확인한다.

```javascript
customerName.trim() === ""
```

공백만 입력된 경우를 확인한다.

예를 들어 다음 입력도 빈 값으로 처리할 수 있다.

```text
"     "
```

`trim()`을 사용하면 앞뒤 공백이 제거되기 때문이다.

---

# 3. 연락처 형식 검증

휴대전화 번호를 다음 형식으로 입력하도록 한다고 가정한다.

```text
010-1234-5678
011-123-4567
```

정규식을 사용할 수 있다.

```javascript
scwin.isValidPhone = function(phone) {

    var phonePattern = /^01[016789]-\d{3,4}-\d{4}$/;

    return phonePattern.test(phone);
};
```

사용 방법은 다음과 같다.

```javascript
var phone = ipt_phone.getValue();

if (!scwin.isValidPhone(phone)) {
    alert("연락처 형식이 올바르지 않습니다.");
    ipt_phone.focus();
    return;
}
```

---

## 정규식 설명

```javascript
/^01[016789]-\d{3,4}-\d{4}$/
```

각 부분의 의미는 다음과 같다.

| 패턴 | 의미 |
|---|---|
| `^` | 문자열 시작 |
| `01` | 반드시 01로 시작 |
| `[016789]` | 0, 1, 6, 7, 8, 9 중 하나 |
| `-` | 하이픈 |
| `\d{3,4}` | 숫자 3자리 또는 4자리 |
| `-` | 하이픈 |
| `\d{4}` | 숫자 4자리 |
| `$` | 문자열 끝 |

---

# 4. 이메일 형식 검증

간단한 이메일 검증은 다음과 같이 작성할 수 있다.

```javascript
scwin.isValidEmail = function(email) {

    var emailPattern =
        /^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$/;

    return emailPattern.test(email);
};
```

사용 예시는 다음과 같다.

```javascript
var email = ipt_email.getValue();

if (email && !scwin.isValidEmail(email)) {
    alert("이메일 형식이 올바르지 않습니다.");
    ipt_email.focus();
    return;
}
```

여기서는 이메일이 선택 입력 항목이라고 가정했기 때문에

```javascript
if (email && ...)
```

처럼 값이 존재할 때만 형식을 검사한다.

---

# 5. 문자열 길이 검증

고객명을 2자 이상 30자 이하로 제한한다고 가정한다.

```javascript
scwin.isValidCustomerNameLength = function(customerName) {

    if (customerName.length < 2) {
        return false;
    }

    if (customerName.length > 30) {
        return false;
    }

    return true;
};
```

조금 더 간단하게 작성하면 다음과 같다.

```javascript
scwin.isValidCustomerNameLength = function(customerName) {

    return customerName.length >= 2
        && customerName.length <= 30;
};
```

사용 예시:

```javascript
var customerName = ipt_customerName.getValue().trim();

if (!scwin.isValidCustomerNameLength(customerName)) {
    alert("고객명은 2자 이상 30자 이하로 입력해 주세요.");
    return;
}
```

---

# 6. 숫자 검증

나이 입력값이 숫자인지 확인하는 예제이다.

```javascript
scwin.isValidAge = function(age) {

    if (age === null || age === "") {
        return false;
    }

    if (isNaN(age)) {
        return false;
    }

    return true;
};
```

숫자의 범위까지 검사하려면 다음과 같이 작성할 수 있다.

```javascript
scwin.isValidAge = function(age) {

    var numberAge = Number(age);

    if (isNaN(numberAge)) {
        return false;
    }

    return numberAge >= 0 && numberAge <= 120;
};
```

---

# 7. 날짜 형식 검증

생년월일을 `YYYYMMDD` 형식으로 입력받는다고 가정한다.

```javascript
scwin.isValidDateFormat = function(date) {

    var datePattern = /^\d{8}$/;

    return datePattern.test(date);
};
```

하지만 단순히 숫자 8자리인지 검사하는 것만으로는 부족하다.

예를 들어 다음 값도 정규식 검사를 통과한다.

```text
20261340
```

따라서 실제 날짜인지 추가 검증하는 것이 좋다.

```javascript
scwin.isValidDate = function(value) {

    if (!/^\d{8}$/.test(value)) {
        return false;
    }

    var year = Number(value.substring(0, 4));
    var month = Number(value.substring(4, 6));
    var day = Number(value.substring(6, 8));

    var date = new Date(year, month - 1, day);

    return date.getFullYear() === year
        && date.getMonth() === month - 1
        && date.getDate() === day;
};
```

---

# 8. 공통 검증 함수 만들기

입력 항목마다 같은 코드를 반복하면 유지보수가 어려워진다.

예를 들어 필수값 검사를 공통 함수로 만들 수 있다.

```javascript
scwin.isEmpty = function(value) {

    return value == null
        || String(value).trim() === "";
};
```

사용 방법:

```javascript
var customerName = ipt_customerName.getValue();

if (scwin.isEmpty(customerName)) {
    alert("고객명을 입력해 주세요.");
    return;
}
```

---

# 9. 저장 전에 전체 검증 수행하기

실무에서는 저장 버튼을 클릭했을 때  
여러 입력값을 한 번에 검사하는 함수를 많이 사용한다.

```javascript
scwin.validateCustomer = function() {

    var customerName = ipt_customerName.getValue();
    var phone = ipt_phone.getValue();
    var email = ipt_email.getValue();

    if (scwin.isEmpty(customerName)) {
        alert("고객명을 입력해 주세요.");
        ipt_customerName.focus();
        return false;
    }

    if (scwin.isEmpty(phone)) {
        alert("연락처를 입력해 주세요.");
        ipt_phone.focus();
        return false;
    }

    if (!scwin.isValidPhone(phone)) {
        alert("연락처 형식이 올바르지 않습니다.");
        ipt_phone.focus();
        return false;
    }

    if (email && !scwin.isValidEmail(email)) {
        alert("이메일 형식이 올바르지 않습니다.");
        ipt_email.focus();
        return false;
    }

    return true;
};
```

저장 버튼에서는 다음과 같이 사용한다.

```javascript
scwin.btn_save_onclick = function() {

    if (!scwin.validateCustomer()) {
        return;
    }

    sbm_createCustomer.submit();
};
```

전체 흐름은 다음과 같다.

```text
저장 버튼 클릭
      ↓
validateCustomer()
      ↓
검증 실패 ─────────→ alert 표시 후 종료
      ↓
검증 성공
      ↓
Submission 실행
      ↓
Spring 서버 호출
```

---

# 10. Grid 입력값 검증

여러 명의 고객 정보를 Grid에 입력한 후 한 번에 저장하는 경우  
각 행을 반복하면서 검증해야 한다.

예를 들어 DataList가 다음과 같다고 가정한다.

```text
dlt_customerInput

customerName
phone
email
```

다음과 같이 검증할 수 있다.

```javascript
scwin.validateCustomerGrid = function() {

    var rowCount = dlt_customerInput.getRowCount();

    for (var rowIndex = 0; rowIndex < rowCount; rowIndex++) {

        var customerName =
            dlt_customerInput.getCellData(
                rowIndex,
                "customerName"
            );

        var phone =
            dlt_customerInput.getCellData(
                rowIndex,
                "phone"
            );

        if (scwin.isEmpty(customerName)) {
            alert(
                (rowIndex + 1)
                + "행의 고객명을 입력해 주세요."
            );

            return false;
        }

        if (!scwin.isValidPhone(phone)) {
            alert(
                (rowIndex + 1)
                + "행의 연락처 형식이 올바르지 않습니다."
            );

            return false;
        }
    }

    return true;
};
```

---

# 11. Grid 중복값 검증

여러 고객을 동시에 등록할 때  
동일한 연락처가 중복 입력되었는지 확인할 수 있다.

```javascript
scwin.hasDuplicatePhone = function() {

    var phoneMap = {};
    var rowCount = dlt_customerInput.getRowCount();

    for (var rowIndex = 0; rowIndex < rowCount; rowIndex++) {

        var phone =
            dlt_customerInput.getCellData(
                rowIndex,
                "phone"
            );

        if (phoneMap[phone]) {

            alert(
                (rowIndex + 1)
                + "행에 중복된 연락처가 있습니다."
            );

            return true;
        }

        phoneMap[phone] = true;
    }

    return false;
};
```

---

## `phoneMap[phone] = true`의 의미

예를 들어 첫 번째 행의 연락처가 다음과 같다고 가정한다.

```text
010-1234-5678
```

처음에는 객체가 비어 있다.

```javascript
var phoneMap = {};
```

첫 번째 연락처를 처리하면 다음과 같은 데이터가 만들어진다.

```javascript
phoneMap["010-1234-5678"] = true;
```

결과적으로 객체는 개념적으로 다음과 같이 된다.

```javascript
{
    "010-1234-5678": true
}
```

다음 행에서 동일한 전화번호가 나오면

```javascript
if (phoneMap[phone])
```

의 결과가 `true`가 되므로 중복이라고 판단할 수 있다.

> `phoneMap`은 배열이 아니라 JavaScript 객체이다.  
> 전화번호 값을 객체의 **동적인 Key**로 사용하고 있다.

---

# 12. `phoneMap.phone`이 아니라 `phoneMap[phone]`을 사용하는 이유

다음 두 문장은 의미가 다르다.

```javascript
phoneMap.phone
```

```javascript
phoneMap[phone]
```

`phoneMap.phone`은 이름이 정확히 `"phone"`인 속성을 조회한다.

즉 다음과 같다.

```javascript
phoneMap["phone"]
```

반면

```javascript
phoneMap[phone]
```

은 변수 `phone` 안에 들어 있는 값을 Key로 사용한다.

예를 들어

```javascript
var phone = "010-1234-5678";
```

이라면

```javascript
phoneMap[phone]
```

은 실제로 다음과 같이 동작한다.

```javascript
phoneMap["010-1234-5678"]
```

따라서 값이 실행 시점에 결정되는 경우에는 대괄호 표기법을 사용한다.

---

# 13. Grid 전체 검증과 중복 검증 함께 사용하기

저장 전에 다음과 같이 처리할 수 있다.

```javascript
scwin.btn_save_onclick = function() {

    if (!scwin.validateCustomerGrid()) {
        return;
    }

    if (scwin.hasDuplicatePhone()) {
        return;
    }

    sbm_createCustomers.submit();
};
```

흐름은 다음과 같다.

```text
저장 클릭
   ↓
행별 필수값/형식 검증
   ↓
중복 연락처 검증
   ↓
검증 성공
   ↓
Submission 실행
   ↓
Spring Batch 등록 API 호출
```

---

# 14. 검증 함수의 반환값 패턴

검증 함수는 일반적으로 `boolean` 값을 반환하도록 작성하면 편리하다.

## 정상인 경우 `true`

```javascript
return true;
```

## 오류가 있는 경우 `false`

```javascript
return false;
```

그러면 호출부에서 다음과 같이 사용할 수 있다.

```javascript
if (!scwin.validateCustomer()) {
    return;
}
```

의미는 다음과 같다.

```text
validateCustomer() 결과가 false이면
현재 함수 실행을 종료한다.
```

---

# 15. 하나의 함수에 모든 검증을 넣지 않는 이유

다음처럼 모든 검증 코드를 하나의 함수에 몰아넣을 수도 있다.

```javascript
scwin.validate = function() {

    // 필수값 검사
    // 전화번호 검사
    // 이메일 검사
    // 날짜 검사
    // Grid 검사
    // 중복 검사
};
```

하지만 코드가 길어지면 관리하기 어려워진다.

따라서 다음과 같이 역할별로 나누는 것이 좋다.

```javascript
scwin.isEmpty
scwin.isValidPhone
scwin.isValidEmail
scwin.isValidDate
scwin.validateCustomer
scwin.validateCustomerGrid
scwin.hasDuplicatePhone
```

이렇게 나누면 각각의 함수가 하나의 역할만 담당하게 된다.

---

# 16. 화면 검증과 서버 검증의 차이

WebSquare 검증과 Spring 검증은 목적이 조금 다르다.

| 구분 | WebSquare | Spring |
|---|---|---|
| 실행 위치 | 브라우저 | 서버 |
| 목적 | 사용자 편의, 빠른 오류 안내 | 데이터 안전성 보장 |
| 필수값 검사 | 가능 | 반드시 권장 |
| 형식 검사 | 가능 | 반드시 권장 |
| DB 중복 검사 | 제한적 | 가능 |
| 보안 검증 | 불충분 | 필수 |
| 우회 가능성 | 있음 | 상대적으로 낮음 |

예를 들어 WebSquare에서 고객명을 필수로 검사하더라도  
Spring DTO에서도 다시 검사하는 것이 좋다.

```java
@NotBlank
private String customerName;
```

연락처도 다음과 같이 검사할 수 있다.

```java
@Pattern(
    regexp = "^01[016789]-\\d{3,4}-\\d{4}$"
)
private String phone;
```

즉 다음과 같이 이해하면 된다.

```text
WebSquare Validation
= 사용자에게 빠르게 오류를 알려주는 1차 검증

Spring Validation
= 잘못된 데이터가 서버에 들어오는 것을 막는 최종 검증
```

---

# 17. 권장 검증 순서

고객 등록 화면에서는 일반적으로 다음 순서가 이해하기 쉽다.

```text
1. 필수값 검증
      ↓
2. 형식 검증
      ↓
3. 길이 / 범위 검증
      ↓
4. 화면 내 중복 검증
      ↓
5. Submission 실행
      ↓
6. 서버 Validation
      ↓
7. DB 중복 / 업무 규칙 검증
      ↓
8. 저장
```

---

# 18. 실무형 예제

단건 고객 등록 화면을 기준으로 검증 코드를 정리하면 다음과 같다.

```javascript
scwin.isEmpty = function(value) {

    return value == null
        || String(value).trim() === "";
};


scwin.isValidPhone = function(phone) {

    var phonePattern =
        /^01[016789]-\d{3,4}-\d{4}$/;

    return phonePattern.test(phone);
};


scwin.isValidEmail = function(email) {

    var emailPattern =
        /^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$/;

    return emailPattern.test(email);
};


scwin.validateCustomer = function() {

    var customerName =
        ipt_customerName.getValue();

    var phone =
        ipt_phone.getValue();

    var email =
        ipt_email.getValue();

    if (scwin.isEmpty(customerName)) {

        alert("고객명을 입력해 주세요.");

        ipt_customerName.focus();

        return false;
    }

    if (scwin.isEmpty(phone)) {

        alert("연락처를 입력해 주세요.");

        ipt_phone.focus();

        return false;
    }

    if (!scwin.isValidPhone(phone)) {

        alert(
            "연락처 형식이 올바르지 않습니다."
        );

        ipt_phone.focus();

        return false;
    }

    if (
        !scwin.isEmpty(email)
        && !scwin.isValidEmail(email)
    ) {

        alert(
            "이메일 형식이 올바르지 않습니다."
        );

        ipt_email.focus();

        return false;
    }

    return true;
};


scwin.btn_save_onclick = function() {

    if (!scwin.validateCustomer()) {
        return;
    }

    sbm_createCustomer.submit();
};
```

---

# 19. 핵심 정리

WebSquare 입력값 검증에서 가장 중요한 흐름은 다음과 같다.

```text
입력값 조회
   ↓
필수값 검사
   ↓
형식 검사
   ↓
길이 / 범위 검사
   ↓
중복 검사
   ↓
검증 성공 여부 반환
   ↓
Submission 실행
```

검증 함수는 가능하면 다음처럼 작은 단위로 분리하는 것이 좋다.

```javascript
scwin.isEmpty()
scwin.isValidPhone()
scwin.isValidEmail()
scwin.isValidDate()
scwin.validateCustomer()
scwin.validateCustomerGrid()
scwin.hasDuplicatePhone()
```

그리고 가장 중요한 원칙은 다음과 같다.

> **WebSquare의 입력값 검증은 사용자 편의를 위한 1차 검증이며,  
> 실제 데이터의 안전성을 위해 Spring 서버에서도 반드시 다시 검증해야 한다.**
