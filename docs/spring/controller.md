---
layout: default
title: Spring Controller
permalink: /spring/controller/
---

{% include navigation.html %}

# Spring Controller

## 1. 역할

Controller는 클라이언트의 HTTP 요청을 받아 Service를 호출하고, 결과를 HTTP 응답으로 반환한다. 복잡한 업무 규칙이나 SQL을 Controller에 직접 작성하기보다 해당 역할을 Service와 데이터 접근 계층에 위임한다.

## 2. 주요 어노테이션

| 어노테이션 | 역할 |
|---|---|
| `@Controller` | Spring MVC 컨트롤러 선언. 주로 뷰 이름을 반환한다. |
| `@RestController` | 반환값을 응답 본문으로 처리한다. REST API에서 자주 사용한다. |
| `@RequestMapping` | 클래스 또는 메서드의 기본 경로를 지정한다. |
| `@GetMapping` | GET 요청 처리 |
| `@PostMapping` | POST 요청 처리 |
| `@PutMapping` | PUT 요청 처리 |
| `@PatchMapping` | PATCH 요청 처리 |
| `@DeleteMapping` | DELETE 요청 처리 |
| `@PathVariable` | URL 경로 변수 전달 |
| `@RequestParam` | 쿼리 파라미터 등 요청 파라미터 전달 |
| `@RequestBody` | 요청 본문(JSON 등)을 객체로 변환해 전달 |

## 3. REST API 예시

```java
@RestController
@RequestMapping("/api/members")
public class MemberController {
    private final MemberService memberService;

    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }

    @GetMapping("/{memberId}")
    public MemberResponse getMember(@PathVariable Long memberId) {
        return memberService.getMember(memberId);
    }
}
```

`GET /api/members/10` 요청에서 `memberId`는 `10`이 된다. `MemberService`와 `MemberResponse`는 프로젝트에 맞게 별도로 정의해야 하는 예시 타입이다.

## 4. 요청값 받기

경로 변수:
```java
@GetMapping("/{id}")
public MemberResponse get(@PathVariable Long id) { ... }
```
예시: `GET /api/members/10`

쿼리 파라미터:
```java
@GetMapping
public List<MemberResponse> search(
        @RequestParam(required = false) String name) { ... }
```
예시: `GET /api/members?name=Kim`

요청 본문:
```java
@PostMapping
public MemberResponse create(@RequestBody MemberCreateRequest request) { ... }
```

예시 JSON:
```json
{ "name": "Kim" }
```

실제 서비스에서는 DTO와 `@Valid` 등을 활용해 입력값을 검증할 수 있다.

## 5. 작성 시 유의점

- URL과 HTTP 메서드를 목적에 맞게 설계한다.
- 업무 규칙은 Service에 위임한다.
- 입력값 검증과 오류 응답 형식을 일관되게 관리한다.
- 민감한 내부 오류 정보를 응답에 노출하지 않는다.

## 6. 학습 체크

1. `@Controller`와 `@RestController`의 차이는?
2. URL의 `/members/10`에서 `10`을 받는 어노테이션은?
3. JSON 요청 본문을 받는 어노테이션은?

## 7. 정리

Controller는 HTTP 요청과 애플리케이션 내부 로직 사이의 경계다. 요청을 해석하고 Service를 호출한 뒤 적절한 응답을 반환하는 역할에 집중한다.
