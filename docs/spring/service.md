---
layout: default
title: Spring Service
permalink: /spring/service/
---

{% include navigation.html %}

# Spring Service

## 1. 역할

Service는 업무 규칙과 처리 흐름을 담당한다. Controller에서 전달받은 값을 처리하고, 필요한 Repository 또는 Mapper를 호출한다. 회원 가입처럼 여러 검증과 데이터 변경이 연결된 작업을 하나의 업무 흐름으로 구성할 수 있다.

## 2. 기본 구조

```java
@Service
public class MemberService {
    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    public MemberResponse getMember(Long id) {
        Member member = memberRepository.findById(id)
            .orElseThrow(() ->
                new IllegalArgumentException("회원을 찾을 수 없습니다."));

        return new MemberResponse(member.getId(), member.getName());
    }
}
```

이 코드는 구조를 설명하기 위한 예시다. `Member`, `MemberRepository`, `MemberResponse`는 실제 프로젝트에서 정의해야 한다.

## 3. Service의 주요 책임

- 업무 규칙 및 조건 처리
- 여러 데이터 접근 호출의 순서 조정
- 업무 단위 트랜잭션 설정
- 결과 데이터 구성
- 오류 상황을 적절한 예외로 표현

## 4. 생성자 주입

생성자 주입은 의존성을 명확하게 드러내고 테스트하기 쉽게 한다. 의존 객체를 `new`로 직접 생성하는 대신 Spring이 주입하도록 구성한다. 생성자가 하나인 경우 일반적으로 `@Autowired`를 생략할 수 있다.

## 5. 트랜잭션과 Service

여러 데이터 변경 작업을 하나의 업무 단위로 처리할 때 `@Transactional`을 고려할 수 있다.

```java
@Transactional
public void updateMember(MemberUpdateRequest request) {
    // 회원 정보 변경
    // 관련 데이터 변경
}
```

기본 설정에서 런타임 예외가 밖으로 전파되면 롤백되는 것이 일반적이며, checked exception은 기본적으로 자동 롤백 대상이 아니다. 필요하면 `rollbackFor`를 지정한다.

## 6. Controller와 Service 비교

| 항목 | Controller | Service |
|---|---|---|
| 책임 | HTTP 요청·응답 | 업무 규칙 및 흐름 |
| 주요 관심사 | URL, 요청값, HTTP 상태 | 조건, 처리 순서, 트랜잭션 |
| 호출 대상 | 주로 Service | Repository 또는 Mapper |

## 7. 작성 시 유의점

- 업무 규칙을 Controller에 중복 작성하지 않는다.
- 예외를 무조건 잡아 무시하지 않는다.
- 트랜잭션 범위를 업무 단위에 맞춘다.
- 트랜잭션 안에서 외부 API 호출이나 긴 작업을 수행할 때는 주의한다.

## 8. 학습 체크

1. 업무 규칙은 Controller와 Service 중 어디에 두는가?
2. 여러 데이터 변경을 한 단위로 처리할 때 무엇을 고려하는가?
3. 예외를 잡아 무시하면 어떤 문제가 생길 수 있는가?

## 9. 정리

Service는 애플리케이션의 업무 중심 계층이다. Controller는 요청을 연결하고, Service는 업무를 수행하며, 데이터 접근은 Repository 또는 Mapper에 위임한다.
