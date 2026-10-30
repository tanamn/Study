---
layout: default
title: Spring Boot 프로그램 구현하기 예제
permalink: /spring/spring-boot-account-practice/
---

{% include navigation.html %}

# Spring Boot 프로그램 구현하기 예제 - VS Code로 Spring Boot 계좌 관리 프로그램 구현하기

## 1. 학습 목표

이번 실습에서는 Spring Boot로 간단한 **계좌 관리 프로그램**을 직접 구현한다. Controller, Service, MyBatis Mapper, 데이터베이스가 어떻게 연결되는지 이해하고, REST API를 직접 호출해 보는 것이 목표다.

실습을 통해 다음 내용을 익힌다.

- Spring Boot 프로젝트 생성 및 실행
- Controller, Service, Mapper의 역할
- MyBatis Mapper 인터페이스와 XML을 이용한 SQL 관리
- H2 데이터베이스 연결 및 데이터 조회·등록·수정
- Postman을 이용한 REST API 테스트
- 요청부터 데이터베이스 처리까지의 전체 흐름

## 2. 구현할 프로그램

**계좌 관리(Account Management) 프로그램**을 구현한다.

| 기능 | 설명 | HTTP 메서드 예시 |
|---|---|---|
| 계좌 목록 조회 | 등록된 전체 계좌 조회 | GET |
| 계좌 단건 조회 | 계좌번호로 계좌 조회 | GET |
| 계좌 등록 | 신규 계좌 등록 | POST |
| 계좌 정보 수정 | 계좌 정보 수정 | PUT |

기본 기능 구현 후 잔액 변경, 거래 내역 조회, 입금·출금 기능으로 확장할 수 있다. 실제 금융 거래의 정합성이나 보안 처리를 구현하는 실무 시스템이 아닌 학습용 예제다.

## 3. 개발 환경 및 기술 구성

| 구분 | 기술 | 용도 |
|---|---|---|
| 개발 도구 | Visual Studio Code | 코드 작성 및 프로젝트 실행 |
| 언어 | Java 17 | 애플리케이션 개발 |
| 프레임워크 | Spring Boot | 애플리케이션 구성 및 실행 |
| 빌드 도구 | Maven | 의존성 관리 및 빌드 |
| 데이터 접근 | MyBatis | SQL 실행 및 결과 매핑 |
| 데이터베이스 | H2 Database | 초기 실습용 DB |
| API 테스트 | Postman | REST API 요청 및 응답 확인 |

처음에는 별도의 DB 서버 설치가 필요 없는 H2를 사용한다. 기본 기능을 구현한 뒤 Oracle 등 다른 DB로 변경해 볼 수 있다.

## 4. 전체 프로그램 구조

```text
사용자 / Postman
       |
       v
Controller
       |
       v
Service
       |
       v
Mapper Interface
       |
       v
Mapper XML
       |
       v
DataSource / JDBC
       |
       v
H2 Database
```

| 구성 요소 | 역할 |
|---|---|
| Controller | HTTP 요청을 받고 응답을 반환한다. |
| Service | 업무 로직을 처리하고 데이터 접근 기능을 호출한다. |
| Mapper Interface | Java 코드에서 SQL을 호출할 메서드를 정의한다. |
| Mapper XML | Mapper 메서드와 연결되는 SQL을 작성한다. |
| DataSource | 데이터베이스 연결을 제공한다. |
| Database | 계좌 데이터를 저장하고 조회한다. |

**핵심:** DataSource는 SQL을 관리하는 구성 요소가 아니라 데이터베이스 연결을 제공하는 구성 요소다. SQL은 MyBatis Mapper XML 등에 작성한다.

## 5. Spring Initializr에서 프로젝트 생성

[Spring Initializr](https://start.spring.io/)에 접속해 프로젝트를 생성한다.

### 5.1 프로젝트 기본 설정

| 항목 | 설정값 |
|---|---|
| Project | Maven |
| Language | Java |
| Group | `com.example` |
| Artifact | `account-management` |
| Name | `account-management` |
| Packaging | Jar |
| Java | 17 |

Spring Boot 버전은 Initializr에서 Java 17과 호환되는 안정 버전을 선택한다.

### 5.2 Dependencies 추가

다음 의존성을 추가한다.

- **Spring Web**: REST API 및 웹 요청 처리
- **MyBatis Framework**: SQL 실행 및 결과 매핑
- **H2 Database**: 실습용 데이터베이스

처음에는 Lombok을 추가하지 않는다. 생성자, getter, setter 등을 직접 작성하면서 Java 클래스 구조를 익히기 위해서다.

### 5.3 VS Code에서 프로젝트 열기

1. Generate를 눌러 ZIP 파일을 다운로드한다.
2. ZIP 파일의 압축을 해제한다.
3. VS Code에서 `File > Open Folder`를 선택한다.
4. 압축을 해제한 `account-management` 폴더를 연다.

## 6. VS Code 개발 환경 확인

### 6.1 Java 확장 기능

VS Code의 Extensions에서 **Extension Pack for Java**가 설치되어 있는지 확인한다. Java 프로젝트 탐색, 코드 자동 완성, 디버깅 및 실행에 필요한 기능을 제공한다.

### 6.2 Java 버전 확인

VS Code 터미널을 열고 다음 명령어를 실행한다.

```bash
java -version
```

Java 17이 설치되어 있다면 출력 결과에 `17`이 표시된다.

Maven도 확인하려면 다음 명령어를 실행한다.

```bash
mvn -version
```

프로젝트에 Maven Wrapper(`mvnw`, `mvnw.cmd`)가 포함되어 있다면 Wrapper를 이용해 별도 Maven 설치 없이 빌드할 수도 있다.

## 7. 프로젝트 디렉터리 구성

실습을 진행하면서 다음과 같은 구조로 파일을 추가한다.

```text
account-management
├─ pom.xml
├─ mvnw
├─ mvnw.cmd
└─ src
   └─ main
      ├─ java
      │  └─ com
      │     └─ example
      │        └─ account
      │           ├─ AccountManagementApplication.java
      │           ├─ controller
      │           │  └─ AccountController.java
      │           ├─ service
      │           │  └─ AccountService.java
      │           ├─ mapper
      │           │  └─ AccountMapper.java
      │           └─ dto
      │              └─ AccountDto.java
      └─ resources
         ├─ application.yml
         ├─ schema.sql
         ├─ data.sql
         └─ mapper
            └─ AccountMapper.xml
```

| 파일 | 역할 |
|---|---|
| `AccountManagementApplication.java` | Spring Boot 애플리케이션 시작점 |
| `AccountController.java` | 계좌 관련 API 제공 |
| `AccountService.java` | 계좌 업무 로직 처리 |
| `AccountMapper.java` | SQL 호출 메서드 정의 |
| `AccountDto.java` | 계좌 데이터 전달 객체 |
| `AccountMapper.xml` | 계좌 조회·등록·수정 SQL |
| `application.yml` | 데이터베이스 및 MyBatis 설정 |
| `schema.sql` | 계좌 테이블 생성 |
| `data.sql` | 실습용 초기 데이터 등록 |

## 8. 단계별 구현 순서

한 번에 모든 파일을 작성하지 않고 다음 순서로 한 단계씩 구현한다.

### 1단계. 프로젝트 생성 및 실행

- Spring Initializr에서 프로젝트 생성
- 압축 해제 후 VS Code에서 열기
- Java 버전 및 확장 기능 확인
- 기본 Spring Boot 애플리케이션 실행
- 콘솔에서 정상 기동 여부 확인

### 2단계. H2 데이터베이스 설정

- `application.yml`에 H2 연결 정보 작성
- `schema.sql`에 계좌 테이블 생성 SQL 작성
- `data.sql`에 테스트 데이터 작성
- 애플리케이션 실행 후 테이블과 초기 데이터 확인

### 3단계. DTO 작성

- `AccountDto.java` 작성
- 계좌번호, 계좌명, 고객명, 잔액 등 데이터 항목 정의
- 생성자, getter, setter의 역할 이해

### 4단계. Mapper 작성

- `AccountMapper.java`에 조회·등록·수정 메서드 정의
- `AccountMapper.xml`에 SQL 작성
- Java 메서드와 XML의 SQL ID 연결
- 조회 결과가 DTO에 매핑되는 과정 확인

### 5단계. Service 작성

- Mapper 호출
- 업무 로직 처리
- Controller와 데이터 접근 계층 사이의 역할 이해

### 6단계. Controller 작성

- 계좌 목록 조회 API
- 계좌 단건 조회 API
- 계좌 등록 API
- 계좌 수정 API

### 7단계. Postman 테스트

- HTTP 메서드와 URL 설정
- 요청 파라미터 또는 JSON Body 작성
- 응답 상태 코드와 응답 데이터 확인
- 잘못된 계좌번호 등 예외 상황 확인

## 9. Spring Boot 시작 클래스

Spring Initializr가 생성하는 시작 클래스는 보통 다음과 같은 구조다.

```java
package com.example.account;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class AccountManagementApplication {

    public static void main(String[] args) {
        SpringApplication.run(AccountManagementApplication.class, args);
    }
}
```

- `@SpringBootApplication`: Spring Boot 애플리케이션의 시작 클래스임을 나타낸다.
- `main()`: Java 프로그램의 시작 메서드다.
- `SpringApplication.run()`: Spring 애플리케이션을 실행하고 필요한 구성 요소를 초기화한다.

실행 후 콘솔에 애플리케이션이 정상적으로 시작되었다는 로그가 표시되는지 확인한다.

## 10. 실습 중 이해해야 할 핵심 개념

| 개념 | 학습할 내용 |
|---|---|
| REST API | HTTP 메서드와 URL을 이용한 기능 제공 방식 |
| DTO | 계층 간 데이터 전달을 위한 객체 |
| Controller | 요청과 응답 처리 |
| Service | 업무 로직 처리 |
| Mapper | Java 메서드와 SQL 연결 |
| Mapper XML | SQL 작성 및 결과 매핑 |
| DataSource | 데이터베이스 연결 제공 |
| SQL | 데이터 조회·등록·수정 |
| 의존성 주입(DI) | Spring이 객체 간 의존 관계를 관리하는 방식 |

## 11. 학습 진행 체크리스트

- Java 17 설치 및 버전 확인
- VS Code Java 확장 기능 설치 확인
- Spring Initializr 프로젝트 생성
- VS Code에서 프로젝트 열기
- Spring Boot 기본 애플리케이션 실행
- H2 데이터베이스 연결
- 계좌 테이블 및 초기 데이터 생성
- DTO 작성
- MyBatis Mapper 인터페이스 작성
- Mapper XML에 SQL 작성
- Service 작성
- Controller 작성
- Postman으로 API 테스트
