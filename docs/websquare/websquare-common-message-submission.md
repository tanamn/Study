---
layout: default
title: WebSquare 공통 메시지·Submission 분석
permalink: /websquare/common-message-submission/
---

{% include navigation.html %}

# WebSquare 공통 메시지·Submission 분석

WebSquare 화면을 개발하다 보면 다음과 같은 코드가 반복해서 등장한다.

```javascript
alert("저장되었습니다.");
```

```javascript
if (confirm("삭제하시겠습니까?")) {
    sbm_delete.submit();
}
```

또는 서버와 통신하기 위해 다음과 같은 `Submission`을 사용한다.

```javascript
sbm_customerSearch.submit();
```

처음에는 단순히

- 메시지를 띄우는 기능
- 서버를 호출하는 기능

정도로 이해하기 쉽다.

하지만 실제 프로젝트에서는

```text
공통 메시지 관리
      ↓
사용자 확인
      ↓
Submission 실행
      ↓
서버 요청
      ↓
응답 수신
      ↓
Callback 처리
      ↓
성공 / 실패 메시지 출력
```

처럼 서로 연결되어 사용되는 경우가 많다.

이 문서에서는 WebSquare에서 자주 사용하는  
**공통 메시지 처리 방식과 Submission의 역할 및 동작 흐름**을 함께 정리한다.

---

# 1. 공통 메시지가 필요한 이유

화면마다 다음과 같이 메시지를 직접 작성할 수 있다.

```javascript
alert("저장되었습니다.");
```

```javascript
alert("삭제되었습니다.");
```

```javascript
alert("조회 결과가 없습니다.");
```

작은 프로젝트에서는 문제가 없을 수 있지만  
화면 수가 많아지면 같은 문구가 여러 곳에 반복된다.

예를 들어 저장 완료 메시지를 100개의 화면에서 사용하고 있다고 가정한다.

```text
저장이 완료되었습니다.
저장되었습니다.
정상적으로 저장되었습니다.
저장 처리가 완료되었습니다.
```

같은 의미인데도 화면마다 문구가 달라질 수 있다.

이런 문제를 해결하기 위해 메시지를 공통으로 관리한다.

---

# 2. 공통 메시지의 기본 개념

공통 메시지는 메시지에 **코드**를 부여하고  
화면에서는 코드만 사용하도록 하는 방식이다.

예를 들어 다음과 같이 정의할 수 있다.

| 메시지 코드 | 메시지 |
|---|---|
| `MSG_SAVE_SUCCESS` | 저장되었습니다. |
| `MSG_DELETE_SUCCESS` | 삭제되었습니다. |
| `MSG_DELETE_CONFIRM` | 삭제하시겠습니까? |
| `MSG_REQUIRED` | {0}을(를) 입력해 주세요. |
| `MSG_NO_DATA` | 조회 결과가 없습니다. |

화면에서는 다음과 같이 사용할 수 있다.

```javascript
var message = com.getMessage("MSG_SAVE_SUCCESS");

alert(message);
```

결과:

```text
저장되었습니다.
```

---

# 3. 공통 메시지를 사용하면 좋은 점

공통 메시지를 사용하면 다음과 같은 장점이 있다.

## 1) 문구를 통일할 수 있다

모든 화면에서 동일한 메시지를 사용한다.

```text
저장되었습니다.
```

## 2) 문구 수정이 쉽다

메시지 정의 한 곳만 수정하면 된다.

예를 들어

```text
저장되었습니다.
```

를

```text
정상적으로 저장되었습니다.
```

로 변경할 때 모든 화면 코드를 수정할 필요가 없다.

## 3) 다국어 처리에 유리하다

메시지 코드 하나에 언어별 메시지를 연결할 수 있다.

```text
MSG_SAVE_SUCCESS
```

한국어:

```text
저장되었습니다.
```

영어:

```text
Saved successfully.
```

## 4) 개발자가 임의의 문구를 만드는 것을 줄일 수 있다

프로젝트 표준 메시지를 사용할 수 있다.

---

# 4. 메시지 코드를 이용한 기본 구조

공통 메시지 객체가 다음과 같이 있다고 가정한다.

```javascript
var messageMap = {
    "MSG_SAVE_SUCCESS": "저장되었습니다.",
    "MSG_DELETE_SUCCESS": "삭제되었습니다.",
    "MSG_DELETE_CONFIRM": "삭제하시겠습니까?",
    "MSG_NO_DATA": "조회 결과가 없습니다."
};
```

메시지를 조회하는 함수는 다음과 같이 만들 수 있다.

```javascript
com.getMessage = function(messageCode) {

    return messageMap[messageCode];
};
```

사용 방법:

```javascript
alert(
    com.getMessage("MSG_SAVE_SUCCESS")
);
```

---

# 5. `messageMap[messageCode]`의 의미

다음 코드를 살펴보자.

```javascript
messageMap[messageCode]
```

예를 들어

```javascript
var messageCode = "MSG_SAVE_SUCCESS";
```

라고 하면 실제로는 다음과 같이 동작한다.

```javascript
messageMap["MSG_SAVE_SUCCESS"]
```

따라서 다음 값이 반환된다.

```text
저장되었습니다.
```

이 방식은 이전에 학습한 다음 코드와 같은 원리이다.

```javascript
phoneMap[phone]
```

즉 대괄호 `[]` 안에 변수를 넣으면  
**변수의 값을 객체의 Key로 사용**한다.

---

# 6. 치환 변수가 있는 메시지

공통 메시지는 고정된 문구만 사용하지 않는다.

예를 들어 다음과 같은 메시지를 만들 수 있다.

```text
{0}을(를) 입력해 주세요.
```

메시지 코드:

```text
MSG_REQUIRED
```

사용할 때 `{0}`에 실제 항목명을 넣는다.

```javascript
com.getMessage(
    "MSG_REQUIRED",
    "고객명"
);
```

결과:

```text
고객명을 입력해 주세요.
```

---

# 7. 메시지 치환 함수 예제

다음과 같이 구현할 수 있다.

```javascript
com.getMessage = function(
    messageCode,
    param
) {

    var message =
        messageMap[messageCode];

    if (param != null) {
        message =
            message.replace(
                "{0}",
                param
            );
    }

    return message;
};
```

사용 예제:

```javascript
alert(
    com.getMessage(
        "MSG_REQUIRED",
        "연락처"
    )
);
```

결과:

```text
연락처를 입력해 주세요.
```

---

# 8. 여러 개의 치환값 사용하기

메시지가 다음과 같다고 가정한다.

```text
{0}은(는) {1}자 이상 입력해 주세요.
```

예:

```javascript
"MSG_MIN_LENGTH":
    "{0}은(는) {1}자 이상 입력해 주세요."
```

사용:

```javascript
com.getMessage(
    "MSG_MIN_LENGTH",
    ["고객명", "2"]
);
```

결과:

```text
고객명은(는) 2자 이상 입력해 주세요.
```

간단한 함수 예시는 다음과 같다.

```javascript
com.getMessage = function(
    messageCode,
    params
) {

    var message =
        messageMap[messageCode];

    if (!params) {
        return message;
    }

    for (
        var index = 0;
        index < params.length;
        index++
    ) {

        message = message.replace(
            "{" + index + "}",
            params[index]
        );
    }

    return message;
};
```

---

# 9. alert 메시지 공통화

다음 코드를

```javascript
alert("저장되었습니다.");
```

공통 함수로 변경할 수 있다.

```javascript
com.alert = function(messageCode) {

    alert(
        com.getMessage(messageCode)
    );
};
```

사용:

```javascript
com.alert("MSG_SAVE_SUCCESS");
```

---

# 10. confirm 메시지 공통화

삭제 전에 사용자 확인을 받는 경우가 많다.

기존 코드:

```javascript
if (confirm("삭제하시겠습니까?")) {

    sbm_delete.submit();
}
```

공통 메시지를 사용하면 다음과 같이 작성할 수 있다.

```javascript
if (
    confirm(
        com.getMessage(
            "MSG_DELETE_CONFIRM"
        )
    )
) {

    sbm_delete.submit();
}
```

공통 함수로 감싸면 다음과 같이 사용할 수도 있다.

```javascript
com.confirm = function(messageCode) {

    return confirm(
        com.getMessage(messageCode)
    );
};
```

사용:

```javascript
if (
    com.confirm(
        "MSG_DELETE_CONFIRM"
    )
) {

    sbm_delete.submit();
}
```

---

# 11. 입력값 검증과 공통 메시지 연결

이전에 학습한 입력값 검증 코드도 공통 메시지와 연결할 수 있다.

기존:

```javascript
if (scwin.isEmpty(customerName)) {

    alert("고객명을 입력해 주세요.");

    return false;
}
```

공통 메시지 사용:

```javascript
if (scwin.isEmpty(customerName)) {

    alert(
        com.getMessage(
            "MSG_REQUIRED",
            "고객명"
        )
    );

    return false;
}
```

이렇게 하면 화면마다 직접 메시지를 작성하지 않아도 된다.

---

# 12. Submission이란?

WebSquare에서 `Submission`은  
**화면의 데이터를 서버로 보내고 서버의 응답을 받아오는 통신 객체**라고 이해하면 된다.

쉽게 표현하면 다음 역할을 담당한다.

```text
WebSquare 화면
      ↓
Submission
      ↓
HTTP Request
      ↓
Spring Controller
      ↓
Service
      ↓
DB
      ↓
Response
      ↓
Submission
      ↓
WebSquare 화면
```

즉 Submission은 WebSquare와 서버 사이의 통신을 담당한다.

---

# 13. Submission이 필요한 이유

예를 들어 고객을 조회한다고 가정한다.

사용자는 화면에서 다음 조건을 입력한다.

```text
고객명 : 김철수
연락처 : 010-1234-5678
```

이 정보를 서버로 보내야 한다.

그리고 서버에서 조회한 결과를 다시 화면에 받아야 한다.

Submission은 이 과정에서 다음을 담당한다.

```text
요청 데이터 수집
      ↓
서버 URL 호출
      ↓
HTTP Method 적용
      ↓
요청 전송
      ↓
응답 수신
      ↓
DataList / DataMap 반영
      ↓
Callback 호출
```

---

# 14. Submission의 주요 속성

프로젝트와 WebSquare 버전에 따라 설정 방식은 조금 다를 수 있지만  
개념적으로 다음 속성을 많이 사용한다.

| 속성 | 의미 |
|---|---|
| `id` | Submission 식별자 |
| `action` | 호출할 서버 URL |
| `method` | GET, POST 등 HTTP Method |
| `ref` | 서버로 전달할 데이터 |
| `target` | 서버 응답을 받을 데이터 |
| `mediatype` | 데이터 전송 형식 |
| `encoding` | 데이터 인코딩 |
| `submitHandler` | 요청 전 처리 |
| `submitDoneHandler` | 성공 응답 처리 |
| `submitErrorHandler` | 오류 처리 |

핵심은 다음과 같이 이해하면 된다.

```text
ref
= 서버로 보낼 데이터

target
= 서버에서 받아올 데이터
```

---

# 15. Submission 호출

Submission의 ID가

```text
sbm_customerSearch
```

라고 가정한다.

실행은 다음과 같이 한다.

```javascript
sbm_customerSearch.submit();
```

이 한 줄이 실행되면  
Submission에 설정된 URL과 데이터를 기준으로 서버 요청이 발생한다.

---

# 16. 고객 조회 Submission 흐름

고객 조회 버튼을 클릭했을 때 다음 코드가 실행된다고 가정한다.

```javascript
scwin.btn_search_onclick = function() {

    sbm_customerSearch.submit();
};
```

Submission의 개념적인 설정:

```text
ID
sbm_customerSearch

Action
/api/customers/search

Method
POST

Request
dm_searchCondition

Response
dlt_customer
```

전체 흐름:

```text
조회 버튼 클릭
      ↓
sbm_customerSearch.submit()
      ↓
dm_searchCondition 데이터 전송
      ↓
POST /api/customers/search
      ↓
Spring Controller
      ↓
고객 조회
      ↓
조회 결과 반환
      ↓
dlt_customer에 결과 저장
      ↓
Grid에 표시
```

---

# 17. DataMap을 요청 데이터로 사용하기

검색 조건을 DataMap에 저장한다고 가정한다.

```text
dm_searchCondition
```

내용:

```text
customerName : 김철수
phone        : 010-1234-5678
```

Submission에서는 이 DataMap을 서버로 전달한다.

개념적으로:

```text
ref = dm_searchCondition
```

서버에서는 다음과 비슷한 JSON을 받을 수 있다.

```json
{
    "customerName": "김철수",
    "phone": "010-1234-5678"
}
```

---

# 18. DataList를 요청 데이터로 사용하기

고객 여러 건을 한 번에 등록한다고 가정한다.

DataList:

```text
dlt_customerInput
```

데이터:

```text
고객명   연락처
홍길동   010-1111-1111
김철수   010-2222-2222
이영희   010-3333-3333
```

Submission을 통해 서버로 보내면 개념적으로 다음과 같은 형태가 된다.

```json
{
    "customers": [
        {
            "customerName": "홍길동",
            "phone": "010-1111-1111"
        },
        {
            "customerName": "김철수",
            "phone": "010-2222-2222"
        },
        {
            "customerName": "이영희",
            "phone": "010-3333-3333"
        }
    ]
}
```

이 데이터는 Spring에서 다음과 같은 DTO로 받을 수 있다.

```java
public class CustomerBatchCreateRequest {

    private List<CustomerCreateDto> customers;
}
```

---

# 19. Submission과 Spring Controller 연결

WebSquare Submission:

```text
POST /api/customers/create
```

Spring Controller:

```java
@PostMapping("/create")
public CustomerCreateResponse createCustomer(
        @Valid
        @RequestBody
        CustomerCreateRequest request) {

    return customerService.createCustomer(
        request
    );
}
```

WebSquare와 Spring의 관계는 다음과 같다.

```text
WebSquare Submission
        ↓
HTTP POST
        ↓
Spring @PostMapping
        ↓
@RequestBody DTO
```

---

# 20. 저장 처리 기본 구조

저장 버튼을 클릭하면 일반적으로 바로 Submission을 실행하지 않는다.

먼저 입력값을 검증한다.

```javascript
scwin.btn_save_onclick = function() {

    if (!scwin.validateCustomer()) {
        return;
    }

    sbm_customerSave.submit();
};
```

전체 흐름:

```text
저장 버튼 클릭
      ↓
입력값 검증
      ↓
검증 실패
      └── 메시지 출력 후 종료

검증 성공
      ↓
Submission 실행
      ↓
서버 저장
```

---

# 21. 확인 메시지와 Submission 연결

저장 전에 확인 메시지를 사용할 수도 있다.

```javascript
scwin.btn_save_onclick = function() {

    if (!scwin.validateCustomer()) {
        return;
    }

    if (
        !com.confirm(
            "MSG_SAVE_CONFIRM"
        )
    ) {
        return;
    }

    sbm_customerSave.submit();
};
```

흐름:

```text
저장 버튼 클릭
      ↓
입력값 검증
      ↓
"저장하시겠습니까?"
      ↓
취소 → 종료
      ↓
확인
      ↓
Submission 실행
```

---

# 22. Callback이란?

Submission이 서버 요청을 보낸 후  
서버의 응답을 처리하기 위해 사용하는 함수가 Callback이다.

예를 들어 저장이 성공한 후 다음 작업을 할 수 있다.

```text
저장 완료 메시지
목록 재조회
입력 영역 초기화
팝업 닫기
버튼 상태 변경
```

이를 Callback에서 처리한다.

---

# 23. 저장 성공 Callback 예제

예를 들어 저장 성공 후 다음 함수를 호출한다고 가정한다.

```javascript
scwin.sbm_customerSave_submitdone =
    function(e) {

        com.alert(
            "MSG_SAVE_SUCCESS"
        );

        sbm_customerSearch.submit();
    };
```

동작:

```text
고객 저장 성공
      ↓
Callback 실행
      ↓
"저장되었습니다."
      ↓
고객 목록 다시 조회
```

---

# 24. Submit Done과 Submit Error

Submission 처리 결과는 크게 다음과 같이 구분할 수 있다.

```text
정상 응답
→ submitDone

통신 오류
→ submitError
```

예:

```javascript
scwin.sbm_customerSave_submitdone =
    function(e) {

        alert("저장되었습니다.");
    };
```

오류:

```javascript
scwin.sbm_customerSave_submiterror =
    function(e) {

        alert(
            "서버 통신 중 오류가 발생했습니다."
        );
    };
```

---

# 25. HTTP 성공과 업무 성공은 다를 수 있다

중요한 부분이다.

HTTP 통신 자체는 성공했지만  
업무 처리 결과는 실패할 수 있다.

예를 들어 서버 응답:

```json
{
    "success": false,
    "message": "이미 등록된 연락처입니다."
}
```

HTTP 상태는 정상 응답일 수 있지만  
업무적으로는 저장 실패이다.

따라서 Callback에서 `success` 값을 확인해야 할 수 있다.

```javascript
scwin.sbm_customerSave_submitdone =
    function(e) {

        var success =
            dm_result.get(
                "success"
            );

        var message =
            dm_result.get(
                "message"
            );

        if (!success) {
            alert(message);
            return;
        }

        alert(message);

        sbm_customerSearch.submit();
    };
```

---

# 26. 공통 응답 객체

서버 응답 형식을 통일하면 화면 개발이 편리해진다.

예:

```json
{
    "success": true,
    "message": "저장되었습니다.",
    "data": null
}
```

조회 결과라면:

```json
{
    "success": true,
    "message": "",
    "data": [
        {
            "customerName": "홍길동",
            "phone": "010-1111-1111"
        }
    ]
}
```

이런 구조를 프로젝트 공통 응답 형식으로 사용할 수 있다.

---

# 27. 서버 메시지와 화면 공통 메시지의 역할

메시지를 어디에서 관리할지 구분하는 것도 중요하다.

## 화면 공통 메시지에 적합한 것

사용자 UI와 관련된 일반 메시지:

```text
저장하시겠습니까?
삭제하시겠습니까?
고객명을 입력해 주세요.
조회 결과가 없습니다.
```

## 서버 메시지에 적합한 것

업무 처리 결과:

```text
이미 등록된 연락처입니다.
삭제할 수 없는 고객입니다.
계약이 진행 중인 고객은 삭제할 수 없습니다.
```

즉 다음과 같이 나눌 수 있다.

```text
UI 메시지
→ WebSquare 공통 메시지

업무 규칙 메시지
→ Spring 서버
```

프로젝트 표준에 따라 서버 메시지도 메시지 코드로 통합 관리할 수 있다.

---

# 28. Submission 실행 전 처리

Submission 실행 전에 공통 처리가 필요할 수 있다.

예:

```text
입력값 검증
로딩바 표시
공통 Header 설정
사용자 정보 설정
요청 시간 기록
```

개념적인 흐름:

```text
Submission 실행 요청
      ↓
Before Submit
      ↓
공통 사전 처리
      ↓
HTTP Request
```

예제:

```javascript
scwin.beforeSubmit = function() {

    com.showLoading();
};
```

---

# 29. Submission 실행 후 처리

서버 응답 후 다음과 같은 공통 처리를 할 수 있다.

```text
로딩바 제거
세션 오류 확인
공통 오류 처리
응답 메시지 처리
```

예:

```javascript
scwin.afterSubmit = function() {

    com.hideLoading();
};
```

---

# 30. 공통 Submission 처리 함수

화면마다 다음 코드를 반복한다고 가정한다.

```javascript
com.showLoading();

sbm_customerSearch.submit();
```

Submission 종료 후:

```javascript
com.hideLoading();
```

프로젝트에서는 이러한 처리를 공통화하는 경우가 많다.

개념적으로:

```javascript
com.submit = function(submission) {

    com.showLoading();

    submission.submit();
};
```

사용:

```javascript
com.submit(
    sbm_customerSearch
);
```

실제 프로젝트에서는 WebSquare 공통 프레임워크에서  
더 많은 기능을 함께 처리할 수 있다.

---

# 31. 공통 Submission 처리에서 다룰 수 있는 내용

공통 Submission 함수에는 다음과 같은 기능이 포함될 수 있다.

```text
로딩 표시
세션 체크
공통 Header 설정
로그 기록
중복 요청 방지
응답 코드 확인
에러 메시지 처리
권한 오류 처리
```

즉 Submission은 단순한 서버 호출 이상의 역할을 할 수 있다.

---

# 32. 중복 Submission 방지

사용자가 저장 버튼을 빠르게 여러 번 클릭하면  
동일한 요청이 여러 번 발생할 수 있다.

예:

```text
저장 클릭
저장 클릭
저장 클릭
```

서버에는 다음과 같이 요청이 발생할 수 있다.

```text
POST /create
POST /create
POST /create
```

이 경우 데이터가 중복 저장될 가능성이 있다.

따라서 저장 중에는 버튼을 비활성화하거나  
중복 요청을 막는 처리가 필요할 수 있다.

---

# 33. 저장 버튼 비활성화 예제

개념적인 예제:

```javascript
scwin.btn_save_onclick = function() {

    if (!scwin.validateCustomer()) {
        return;
    }

    btn_save.setDisabled(true);

    sbm_customerSave.submit();
};
```

Submission 완료 후 다시 활성화한다.

```javascript
scwin.sbm_customerSave_submitdone =
    function(e) {

        btn_save.setDisabled(false);

        com.alert(
            "MSG_SAVE_SUCCESS"
        );
    };
```

오류 시에도 다시 활성화해야 한다.

```javascript
scwin.sbm_customerSave_submiterror =
    function(e) {

        btn_save.setDisabled(false);

        com.alert(
            "MSG_SYSTEM_ERROR"
        );
    };
```

---

# 34. 조회 후 결과가 없을 때 메시지 처리

고객 조회 결과가 0건인 경우:

```javascript
scwin.sbm_customerSearch_submitdone =
    function(e) {

        var rowCount =
            dlt_customer.getRowCount();

        if (rowCount === 0) {

            com.alert(
                "MSG_NO_DATA"
            );

            return;
        }
    };
```

이 코드에서는

```javascript
dlt_customer.getRowCount()
```

를 통해 조회 결과 행 수를 확인한다.

---

# 35. 삭제 Submission 예제

삭제는 다음 단계로 구성하는 것이 일반적이다.

```text
선택 여부 확인
      ↓
삭제 확인 메시지
      ↓
Submission 실행
      ↓
삭제 결과 확인
      ↓
완료 메시지
      ↓
목록 재조회
```

예제:

```javascript
scwin.btn_delete_onclick = function() {

    var checkedCount =
        scwin.getCheckedCount();

    if (checkedCount === 0) {

        com.alert(
            "MSG_SELECT_DELETE_DATA"
        );

        return;
    }

    if (
        !com.confirm(
            "MSG_DELETE_CONFIRM"
        )
    ) {
        return;
    }

    sbm_customerDelete.submit();
};
```

---

# 36. 삭제 완료 Callback

```javascript
scwin.sbm_customerDelete_submitdone =
    function(e) {

        var success =
            dm_result.get("success");

        if (!success) {

            alert(
                dm_result.get(
                    "message"
                )
            );

            return;
        }

        com.alert(
            "MSG_DELETE_SUCCESS"
        );

        sbm_customerSearch.submit();
    };
```

---

# 37. 조회·등록·수정·삭제 Submission 구분

CRUD 기준으로 Submission을 나눌 수 있다.

| 기능 | Submission 예 |
|---|---|
| 조회 | `sbm_customerSearch` |
| 등록 | `sbm_customerCreate` |
| 수정 | `sbm_customerUpdate` |
| 삭제 | `sbm_customerDelete` |

이렇게 이름을 명확하게 지으면  
코드만 보고도 Submission의 목적을 알 수 있다.

---

# 38. Submission ID 명명 규칙 예시

프로젝트에서 다음과 같은 규칙을 사용할 수 있다.

```text
sbm_<업무명><처리명>
```

예:

```text
sbm_customerSearch
sbm_customerCreate
sbm_customerUpdate
sbm_customerDelete
```

다건 처리라면:

```text
sbm_customerBatchCreate
sbm_customerBatchDelete
```

중요한 것은 특정 규칙 자체보다  
**프로젝트 전체에서 같은 규칙을 사용하는 것**이다.

---

# 39. DataMap과 DataList 역할 구분

Submission을 이해하려면  
DataMap과 DataList의 역할도 함께 이해하는 것이 좋다.

## DataMap

한 건의 데이터 또는 검색 조건을 표현할 때 적합하다.

```text
dm_searchCondition

customerName
phone
status
```

## DataList

여러 건의 목록 데이터를 표현할 때 적합하다.

```text
dlt_customer

customerId
customerName
phone
status
```

정리하면:

```text
DataMap
= 한 건 또는 조건 데이터

DataList
= 여러 행의 목록 데이터
```

---

# 40. 조회 Submission에서 자주 사용하는 구조

```text
입력 컴포넌트
      ↓
DataMap
      ↓
Submission
      ↓
Spring
      ↓
조회 결과
      ↓
DataList
      ↓
Grid
```

예:

```text
ipt_customerName
      ↓
dm_searchCondition
      ↓
sbm_customerSearch
      ↓
CustomerController
      ↓
dlt_customer
      ↓
grd_customer
```

---

# 41. 등록 Submission에서 자주 사용하는 구조

단건 등록:

```text
Input 컴포넌트
      ↓
DataMap
      ↓
Submission
      ↓
Spring Controller
      ↓
Service
      ↓
DB INSERT
```

다건 등록:

```text
Grid 입력
      ↓
DataList
      ↓
Submission
      ↓
Spring Controller
      ↓
List<DTO>
      ↓
Service
      ↓
Batch INSERT
```

---

# 42. 공통 메시지와 Submission을 함께 보는 이유

실제 화면 코드에서는 두 기능이 따로 존재하지 않는다.

예를 들어 고객 저장은 다음과 같이 연결된다.

```javascript
scwin.btn_save_onclick = function() {

    if (!scwin.validateCustomer()) {
        return;
    }

    if (
        !com.confirm(
            "MSG_SAVE_CONFIRM"
        )
    ) {
        return;
    }

    sbm_customerSave.submit();
};
```

그리고 성공 Callback:

```javascript
scwin.sbm_customerSave_submitdone =
    function(e) {

        com.alert(
            "MSG_SAVE_SUCCESS"
        );

        sbm_customerSearch.submit();
    };
```

전체 흐름은 다음과 같다.

```text
입력값 검증
      ↓
공통 확인 메시지
      ↓
Submission
      ↓
Spring
      ↓
DB 저장
      ↓
응답
      ↓
Callback
      ↓
공통 완료 메시지
      ↓
목록 재조회
```

---

# 43. 실무형 전체 예제

다음은 고객 등록 화면을 단순화한 예제이다.

```javascript
scwin.btn_save_onclick = function() {

    if (!scwin.validateCustomer()) {
        return;
    }

    if (
        !com.confirm(
            "MSG_SAVE_CONFIRM"
        )
    ) {
        return;
    }

    btn_save.setDisabled(true);

    sbm_customerCreate.submit();
};
```

성공 Callback:

```javascript
scwin.sbm_customerCreate_submitdone =
    function(e) {

        btn_save.setDisabled(false);

        var success =
            dm_result.get(
                "success"
            );

        var message =
            dm_result.get(
                "message"
            );

        if (!success) {

            alert(message);

            return;
        }

        com.alert(
            "MSG_SAVE_SUCCESS"
        );

        sbm_customerSearch.submit();
    };
```

오류 Callback:

```javascript
scwin.sbm_customerCreate_submiterror =
    function(e) {

        btn_save.setDisabled(false);

        com.alert(
            "MSG_SYSTEM_ERROR"
        );
    };
```

---

# 44. 전체 처리 흐름 분석

위 코드를 순서대로 분석하면 다음과 같다.

## 1단계: 저장 버튼 클릭

```javascript
scwin.btn_save_onclick()
```

## 2단계: 입력값 검증

```javascript
scwin.validateCustomer()
```

검증 실패:

```text
메시지 출력
→ 처리 종료
```

## 3단계: 저장 확인

```javascript
com.confirm(
    "MSG_SAVE_CONFIRM"
)
```

취소:

```text
처리 종료
```

확인:

```text
다음 단계 진행
```

## 4단계: 저장 버튼 비활성화

```javascript
btn_save.setDisabled(true);
```

중복 요청을 방지한다.

## 5단계: Submission 실행

```javascript
sbm_customerCreate.submit();
```

## 6단계: Spring Controller 호출

```text
POST /api/customers/create
```

## 7단계: 서버 처리

```text
Controller
→ Service
→ Repository
→ DB
```

## 8단계: 응답 수신

```json
{
    "success": true,
    "message": "저장되었습니다."
}
```

## 9단계: Callback 실행

```javascript
scwin.sbm_customerCreate_submitdone()
```

## 10단계: 성공 메시지 표시

```javascript
com.alert(
    "MSG_SAVE_SUCCESS"
);
```

## 11단계: 목록 재조회

```javascript
sbm_customerSearch.submit();
```

---

# 45. 공통 메시지 관리 시 주의할 점

## 메시지 코드 이름을 의미 있게 작성한다

좋은 예:

```text
MSG_SAVE_SUCCESS
MSG_DELETE_CONFIRM
MSG_REQUIRED
MSG_SYSTEM_ERROR
```

의미를 알기 어려운 예:

```text
MSG001
MSG002
MSG003
```

단, 기존 프로젝트에서 숫자 기반 메시지 코드 체계를 사용한다면  
프로젝트 표준을 따르는 것이 우선이다.

---

# 46. 너무 많은 화면 전용 메시지는 공통화하지 않아도 된다

모든 메시지를 반드시 공통화할 필요는 없다.

예를 들어 특정 화면에서 한 번만 사용하는 복잡한 안내 문구까지  
모두 메시지 코드로 만들면 관리가 오히려 어려워질 수 있다.

다음과 같은 기준이 실용적이다.

```text
여러 화면에서 반복 사용
→ 공통 메시지

특정 업무 규칙
→ 서버 또는 업무 메시지

한 화면에서만 사용하는 상세 안내
→ 화면 로컬 메시지 검토
```

---

# 47. Submission 분석 시 확인해야 할 항목

기존 WebSquare 소스를 분석할 때 Submission을 발견하면  
다음 순서로 확인하면 이해하기 쉽다.

```text
1. Submission ID는 무엇인가?
2. 어느 이벤트에서 submit()을 호출하는가?
3. action URL은 무엇인가?
4. HTTP Method는 무엇인가?
5. 서버로 보내는 ref 데이터는 무엇인가?
6. 응답을 받는 target 데이터는 무엇인가?
7. 연결되는 Spring Controller는 무엇인가?
8. 성공 Callback은 어디인가?
9. 오류 Callback은 어디인가?
10. Callback 이후 어떤 화면 처리를 하는가?
```

---

# 48. Submission 분석 예제

다음 코드가 있다고 가정한다.

```javascript
scwin.btn_search_onclick = function() {

    sbm_customerSearch.submit();
};
```

분석 순서:

```text
sbm_customerSearch
      ↓
Submission 설정 찾기
      ↓
action 확인
      ↓
/api/customers/search
      ↓
Spring Controller 검색
      ↓
@PostMapping("/search")
      ↓
Request DTO 확인
      ↓
Response DTO 확인
      ↓
target DataList 확인
      ↓
Grid와 연결 여부 확인
```

이런 식으로 화면부터 서버까지 추적하면 된다.

---

# 49. Callback 분석 시 확인할 내용

예를 들어 다음 Callback이 있다고 하자.

```javascript
scwin.sbm_customerSearch_submitdone =
    function(e) {

        if (
            dlt_customer.getRowCount()
            === 0
        ) {

            com.alert(
                "MSG_NO_DATA"
            );
        }
    };
```

이 코드는 다음을 의미한다.

```text
Submission 정상 완료
      ↓
dlt_customer 행 수 확인
      ↓
0건이면
      ↓
"조회 결과가 없습니다." 출력
```

즉 Callback을 보면  
Submission 이후 화면이 무엇을 하는지 알 수 있다.

---

# 50. Submission을 분석할 때 중요한 관점

Submission 자체만 보면 안 된다.

다음 요소를 연결해서 봐야 한다.

```text
버튼 / 이벤트
      ↓
Validation
      ↓
공통 메시지
      ↓
Submission
      ↓
Request DataMap / DataList
      ↓
Spring Controller
      ↓
Response
      ↓
Target DataMap / DataList
      ↓
Callback
      ↓
Grid / Input / Popup
```

이 전체 흐름을 하나의 기능으로 보는 것이 중요하다.

---

# 51. 핵심 정리

공통 메시지는

```text
화면마다 반복되는 메시지를
코드로 통합 관리하는 방식
```

이라고 이해하면 된다.

Submission은

```text
WebSquare 화면과 Spring 서버 사이에서
요청과 응답을 처리하는 통신 객체
```

라고 이해하면 된다.

두 기능을 함께 보면 일반적인 화면 처리 흐름은 다음과 같다.

```text
사용자 입력
      ↓
입력값 검증
      ↓
공통 확인 메시지
      ↓
Submission
      ↓
Spring Controller
      ↓
Service / DB
      ↓
Response
      ↓
Submission Callback
      ↓
공통 결과 메시지
      ↓
화면 갱신
```

WebSquare 소스를 분석할 때는  
단순히 `submit()` 한 줄만 보는 것이 아니라

```text
누가 호출하는지
무엇을 보내는지
어디로 보내는지
무엇을 받는지
받은 후 무엇을 하는지
```

를 순서대로 따라가는 것이 가장 중요하다.
