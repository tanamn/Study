---
layout: default
title: Spring 기본 구조
permalink: /spring/spring/
---

{% include navigation.html %}

# Spring 기본 구조

## 1. Spring과 Spring Boot

Spring은 Java 애플리케이션 개발을 위한 프레임워크로 객체 관리, 웹 요청 처리, 데이터 접근, 트랜잭션 등의 기능을 제공한다. Spring Boot는 자동 설정과 내장 서버 등을 통해 Spring 애플리케이션을 더 쉽게 구성하고 실행하도록 돕는다.

## 2. 일반적인 계층 구조

```text
Client → Controller → Service → Repository 또는 Mapper → Database
```

- **Controller**: HTTP 요청을 받고 응답을 반환한다.
- **Service**: 업무 규칙과 처리 흐름을 담당한다.
- **Repository**: 저장소 접근을 추상화한다.
- **Mapper**: MyBatis 등에서 SQL과 Java 메서드·객체를 연결한다.
- **DTO**: 계층 간 전달할 데이터를 담는다.
- **Configuration**: Bean 및 애플리케이션 설정을 정의한다.

## 3. IoC와 DI

**IoC(Inversion of Control)**는 객체 생성과 생명주기 관리의 제어권을 Spring 컨테이너가 담당하는 개념이다. **DI(Dependency Injection)**는 객체가 필요로 하는 의존 객체를 외부에서 전달받는 방식이다.

```java
@Service
public class MemberService {
    private final MemberRepository repository;

    public MemberService(MemberRepository repository) {
        this.repository = repository;
    }
}
```

위 코드는 생성자 주입 예시다. `MemberService`가 직접 Repository를 생성하지 않고 외부에서 전달받는다.

## 4. Bean과 Spring 컨테이너

Bean은 Spring 컨테이너가 생성하고 관리하는 객체다. `ApplicationContext`는 Bean을 관리하는 대표적인 컨테이너 인터페이스다.

주요 어노테이션:
- `@Component`: 일반 컴포넌트
- `@Service`: 서비스 계층
- `@Repository`: 데이터 접근 계층
- `@Controller`: MVC 컨트롤러
- `@RestController`: REST API 컨트롤러

## 5. 요청 처리 흐름

1. 프론트엔드가 API 요청을 보낸다.
2. Controller가 요청을 받는다.
3. Service가 업무 처리를 수행한다.
4. Repository 또는 Mapper가 데이터에 접근한다.
5. 결과가 Controller를 통해 응답으로 반환된다.

## 6. 학습 체크

- Controller와 Service의 책임은 어떻게 다른가?
- DI를 사용하면 어떤 장점이 있는가?
- Repository와 Mapper는 어떻게 구분되는가?

## 7. 정리

Spring을 학습할 때는 어노테이션을 외우는 것뿐 아니라 요청이 각 계층을 거쳐 처리되는 흐름과 각 계층의 책임을 이해하는 것이 중요하다.
