---
layout: default
permalink: /spring/spring-account-advanced/05-implementation-guide/
title: Spring 계좌관리 심화 과정 구현 가이드
---

{% include navigation.html %}

# 1. 문서 목적

업무·DB·API 설계를 Spring Boot 애플리케이션으로 구현하기 위한 기술
구성과 작업 순서를 정의한다.

## 2. 프로젝트 기술 구성

  영역           기술                 역할
  -------------- -------------------- --------------------------
  언어           Java                 애플리케이션 구현
  프레임워크     Spring Boot          애플리케이션 기반
  웹             Spring Web           REST API
  검증           Spring Validation    요청값 검증
  데이터 접근    MyBatis              SQL 실행 및 결과 매핑
  데이터베이스   학습용 DB부터 시작   계좌 및 거래 데이터 저장
  테스트         JUnit, Mockito       단위·통합 테스트
  API 문서       Swagger / OpenAPI    API 명세 확인
  실행 환경      Docker(후속)         실행 환경 구성

Java 및 Spring Boot 버전과 DBMS는 프로젝트 생성 시 확정하고 README에
기록한다.

## 3. 권장 패키지 구조

``` text
src/main/java/com/example/account
├── AccountApplication.java
├── controller/
├── service/
├── mapper/
├── domain/
├── dto/
├── exception/
└── config/

src/main/resources
├── mapper/
│   ├── AccountMapper.xml
│   └── TransactionMapper.xml
└── application.yml
```

프로젝트 규모와 기존 기본 과정의 패키지 규칙을 확인한 뒤 최종 구조를
확정한다.

## 4. MyBatis 구성

### 4.1 Mapper Interface

Java 인터페이스에 SQL 호출 메서드를 정의한다.

``` java
@Mapper
public interface AccountMapper {
    Account findByAccountNo(String accountNo);
}
```

### 4.2 Mapper XML

SQL과 결과 매핑을 정의한다.

``` xml
<mapper namespace="com.example.account.mapper.AccountMapper">
    <select id="findByAccountNo"
            resultType="com.example.account.domain.Account">
        SELECT ACCOUNT_NO, ACCOUNT_NAME, BALANCE, STATUS
        FROM ACCOUNT
        WHERE ACCOUNT_NO = #{accountNo}
    </select>
</mapper>
```

`namespace`는 Mapper 인터페이스의 전체 경로와 일치시키고, SQL의 `id`는
호출 메서드와 일치시킨다. 실제 컬럼명과 클래스 경로는 프로젝트에 맞게
조정한다.

### 4.3 파라미터 바인딩

-   `#{}`는 PreparedStatement 기반 파라미터 바인딩에 사용한다.
-   `${}`는 SQL 문자열 치환이므로 사용자 입력값에 직접 사용하지 않는다.
-   동적 SQL은 `<if>`, `<choose>`, `<foreach>` 등의 태그를 활용한다.

## 5. 계층별 책임

  계층         책임
  ------------ -------------------------------
  Controller   HTTP 요청·응답, 입력 DTO 전달
  Service      업무 규칙, 트랜잭션 경계
  Mapper       SQL 호출
  Mapper XML   SQL 및 결과 매핑
  DTO          API 요청·응답 데이터
  Domain       업무 데이터 표현
  Exception    업무·시스템 예외 정의

Controller에 SQL이나 핵심 업무 규칙을 넣지 않고, Mapper가 API 응답
형식을 직접 결정하지 않도록 한다.

## 6. 구현 순서

1.  Spring Boot 프로젝트 생성 및 실행 확인
2.  Maven 의존성 및 DB 연결 설정
3.  MyBatis 설정과 Mapper 조회 테스트
4.  계좌 조회 API 구현
5.  입금 및 출금 구현
6.  거래내역 저장 및 조회 구현
7.  계좌이체와 트랜잭션 적용
8.  동시성 제어 적용
9.  인증·인가 적용
10. 테스트 및 API 문서화

## 7. 트랜잭션 적용

계좌이체 Service 메서드에 `@Transactional`을 적용하여 출금, 입금 및
거래내역 기록을 하나의 트랜잭션으로 처리한다.

``` java
@Transactional
public TransferResponse transfer(TransferRequest request) {
    // 계좌 검증
    // 출금 처리
    // 입금 처리
    // 거래내역 기록
    // 결과 반환
}
```

예외가 발생했을 때 롤백되는 조건을 확인한다. 기본적으로 Spring은
unchecked exception에 대해 롤백하며, checked exception을 사용하는 경우
롤백 정책을 명시적으로 검토한다.

## 8. 구현 시 점검 사항

-   SQL 파라미터 바인딩이 적용되었는가?
-   UPDATE 결과 건수를 확인하는가?
-   Service에서 트랜잭션 경계를 관리하는가?
-   계좌 잔액과 거래내역이 함께 반영되는가?
-   예외가 적절한 계층에서 처리되는가?
-   민감한 정보가 로그와 응답에 노출되지 않는가?
