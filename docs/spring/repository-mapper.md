---
layout: default
title: Spring Repository와 Mapper
permalink: /spring/repository-mapper/
---

{% include navigation.html %}

# Spring Repository와 Mapper

## 1. 데이터 접근 계층

데이터 접근 계층은 데이터베이스에서 데이터를 조회·저장·수정·삭제하는 역할을 한다. 프로젝트의 기술 선택에 따라 Repository 또는 Mapper를 사용할 수 있다.

## 2. Repository 예시: Spring Data JPA

```java
public interface MemberRepository
        extends JpaRepository<Member, Long> {
}
```

Spring Data JPA에서는 Repository 인터페이스를 정의하면 기본적인 데이터 접근 구현을 프레임워크가 제공할 수 있다. `Member`는 Entity, `Long`은 식별자 타입을 의미한다.

## 3. Mapper 예시: MyBatis

```java
@Mapper
public interface MemberMapper {
    Member findById(Long memberId);
}
```

Mapper 메서드와 SQL은 XML 또는 어노테이션 등 프로젝트 설정에 따라 연결한다.

```xml
<select id="findById" resultType="Member">
    SELECT member_id, name
    FROM member
    WHERE member_id = #{memberId}
</select>
```

`#{memberId}`는 파라미터 바인딩에 사용된다. 사용자 입력을 SQL 문자열에 직접 이어 붙이는 방식은 SQL Injection 위험을 높일 수 있으므로 피한다.

## 4. Repository와 Mapper 비교

| 항목 | Repository (예: JPA) | Mapper (예: MyBatis) |
|---|---|---|
| 중심 | Entity와 객체 관계 | SQL과 Java 메서드 매핑 |
| SQL | 기본 CRUD는 프레임워크가 생성할 수 있음 | SQL을 직접 작성하는 경우가 많음 |
| 주의점 | 영속성 컨텍스트, 지연 로딩 | SQL과 매핑 설정 관리 |

두 방식을 한 프로젝트에서 함께 쓰는 경우도 있다. 실제 구조는 프로젝트의 의존성과 코드를 확인해 판단한다.

## 5. 계층 간 호출

```text
Controller → Service → Repository 또는 Mapper → Database
```

Controller가 데이터 접근 계층을 직접 호출하기보다 Service를 거치면 업무 규칙과 데이터 접근 책임을 분리하기 쉽다.

## 6. `#{}`와 `${}`

- `#{}`: 일반적으로 PreparedStatement 파라미터 바인딩에 사용한다.
- `${}`: 문자열 치환 방식이므로 사용자 입력을 그대로 사용하면 SQL Injection 위험이 있다.

동적 정렬 컬럼처럼 바인딩할 수 없는 항목은 허용 목록 검증 등 안전한 설계가 필요하다.

## 7. 작성 시 유의점

- SQL과 결과 매핑 타입을 확인한다.
- 조회 결과가 없을 때의 처리 방식을 정한다.
- SQL 파라미터를 안전하게 바인딩한다.
- 데이터 접근 코드를 Controller에 두지 않는다.

## 8. 학습 체크

1. JPA Repository와 MyBatis Mapper의 차이는?
2. MyBatis에서 `#{}`를 사용하는 이유는?
3. 데이터 접근 계층을 Service 뒤에 두는 장점은?

## 9. 정리

Repository와 Mapper는 모두 데이터 접근과 관련되지만 기술과 사용 방식이 다르다. 프로젝트가 JPA 기반인지 MyBatis 기반인지 먼저 확인한다.
