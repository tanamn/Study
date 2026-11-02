---
layout: default
title: Spring Boot 프로젝트 생성
permalink: /spring/spring-boot-practice/02-project-create/
---

{% include navigation.html %}

# 2. Spring Boot 프로젝트 생성

## 2.1 Spring Initializr란?

Spring Initializr는 Spring Boot 프로젝트의 기본 구조와 의존성(Dependencies)을 자동으로 생성해 주는 도구이다.

- 공식 사이트: https://start.spring.io/
- 프로젝트 빌드 도구: Maven
- 개발 언어: Java

## 2.2 프로젝트 초기값 설정

이번 실습에서는 은행 계좌관리 REST API를 구현하기 위한 프로젝트를 생성한다. 프로젝트명은 `account-management`이며, 계좌 조회·등록·수정 기능을 구현할 예정이다.

### ① 프로젝트 기본 설정

| 항목 | 설정값 | 설명 |
|---|---|---|
| Project | Maven | 프로젝트 빌드 및 의존성 관리 |
| Language | Java | 개발 언어 |
| Spring Boot | 4.0.9 | Initializr에서 선택한 버전 |
| Group | `com.example` | 프로젝트 그룹 식별자 |
| Artifact | `account-management` | 프로젝트 이름 |
| Package name | `com.example.accountmanagement` | Java 패키지 경로 |
| Packaging | Jar | 실행 가능한 JAR 패키징 |
| Configuration | Properties | 설정 파일 형식 |
| Java | 17 | 개발 및 실행 Java 버전 |

### ② 프로젝트 설정 화면

아래는 이번 실습에서 실제로 설정한 Spring Initializr 화면이다.

![Spring Initializr 프로젝트 초기값 설정]({{ '/spring/spring-boot-practice/assets/images/spring-initializr-account-management.png' | relative_url }})

### ③ 주요 설정 항목 이해하기

**Project: Maven**

Maven은 프로젝트 빌드, 라이브러리 의존성 관리, 테스트 및 패키징을 담당한다. 현재 PC에는 Maven 3.9.16이 설치되어 있으므로 별도 설치 없이 사용할 수 있다.

**Spring Boot: 4.0.9**

Spring Boot는 Spring 기반 애플리케이션을 빠르게 구성하고 실행할 수 있도록 지원하는 프레임워크이다. 이번 실습에서는 Initializr에서 MyBatis Framework를 선택할 수 있는 버전인 4.0.9를 선택했다.

**Group: `com.example`**

프로젝트의 그룹 또는 소속을 구분하는 식별자이다. 실제 회사 프로젝트에서는 회사 도메인을 역순으로 표현하는 방식을 주로 사용한다.

예시: `com.company`, `com.bank`, `kr.co.company`

**Artifact: `account-management`**

프로젝트의 이름이다. Maven 프로젝트의 기본 식별 정보로 사용되며 생성되는 프로젝트 폴더명에도 반영된다.

**Package name: `com.example.accountmanagement`**

Java 클래스가 위치할 기본 패키지 경로이다. Artifact에는 하이픈(`-`)을 사용할 수 있지만 Java 패키지명에는 사용할 수 없으므로 별도로 지정한다.

**Packaging: Jar**

Spring Boot 애플리케이션을 실행 가능한 JAR 파일로 패키징한다. 내장 웹 서버를 이용하여 별도의 외부 WAS 설치 없이 실행할 수 있다.

**Configuration: Properties**

Spring Boot 설정 파일의 형식을 지정한다. 이번 실습에서는 `application.properties` 파일을 사용한다.

**Java: 17**

현재 PC에 설치된 Java 버전과 동일하게 설정한다.

- 설치 버전: OpenJDK 17.0.18
- 공급자: Eclipse Adoptium
- 아키텍처: 64-bit

## 2.3 Dependencies 설정

Dependencies는 프로젝트에서 사용할 외부 라이브러리 및 프레임워크를 의미한다. 이번 계좌관리 프로그램에서는 다음 3개 의존성을 추가한다.

| 의존성 | 주요 역할 |
|---|---|
| Spring Web | REST API 및 HTTP 요청 처리 |
| H2 Database | 실습용 관계형 데이터베이스 |
| MyBatis Framework | SQL 실행 및 Java 객체와 데이터 매핑 |

### ① Spring Web

Spring MVC 기반의 웹 애플리케이션을 개발하기 위한 의존성이다.

- REST API 구현
- Controller 작성
- GET, POST, PUT 요청 처리
- JSON 데이터 송수신

### ② H2 Database

별도의 DB 서버 설치 없이 사용할 수 있는 경량 관계형 데이터베이스이다. 이번 실습에서는 계좌 데이터를 저장하고 조회하는 용도로 사용한다.

- 계좌 테이블 생성
- 계좌 데이터 등록
- 계좌 정보 조회 및 수정

### ③ MyBatis Framework

Java 애플리케이션과 데이터베이스 사이에서 SQL 실행 및 데이터 매핑을 담당하는 Persistence Framework이다.

- Mapper 인터페이스 작성
- Mapper XML 작성
- SQL 실행 및 결과 매핑
- 데이터베이스 CRUD 처리

### ④ 의존성 선택 시 확인 사항

Spring Initializr는 Spring Boot 버전에 따라 선택할 수 있는 의존성을 제한한다. 이번 화면에서 MyBatis Framework에는 다음 조건이 표시되었다.

`Requires Spring Boot >= 4.0.0 and < 4.1.0-M1`

따라서 MyBatis Framework를 선택하기 위해 Spring Boot 4.0.9를 사용했다. 프로젝트 생성 후에는 `pom.xml`에 실제로 추가된 MyBatis Spring Boot Starter 버전과 Spring Boot 버전의 호환성을 확인한다.

## 2.4 최종 프로젝트 생성 설정

| 구분 | 설정값 |
|---|---|
| Project | Maven |
| Language | Java |
| Spring Boot | 4.0.9 |
| Group | `com.example` |
| Artifact | `account-management` |
| Package name | `com.example.accountmanagement` |
| Packaging | Jar |
| Configuration | Properties |
| Java | 17 |
| Dependencies | Spring Web, H2 Database, MyBatis Framework |

모든 설정을 확인한 후 화면 하단의 **GENERATE** 버튼을 클릭한다. 프로젝트가 ZIP 파일로 다운로드되면 압축을 해제하고 VS Code에서 프로젝트 폴더를 연다.

## 2.5 생성된 프로젝트에서 확인할 파일

| 파일 | 역할 |
|---|---|
| `pom.xml` | Maven 프로젝트 설정 및 의존성 관리 |
| `mvnw.cmd` | Windows용 Maven Wrapper 실행 파일 |
| `mvnw` | Linux/macOS용 Maven Wrapper 실행 파일 |
| `AccountManagementApplication.java` | Spring Boot 시작 클래스 |
| `application.properties` | Spring Boot 환경설정 파일 |

## 2.6 다음 학습 단계

다음 단계에서는 생성한 프로젝트를 VS Code에서 열고 정상적으로 실행되는지 확인한다.

- Maven 프로젝트 구조 확인
- `pom.xml` 의존성 확인
- Spring Boot 애플리케이션 실행
- 웹 서버 기동 및 실행 로그 확인
