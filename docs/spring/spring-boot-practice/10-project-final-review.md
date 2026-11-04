---
layout: default
title: 10단계 - 프로젝트 최종 점검 및 정리
permalink: /spring/spring-boot-practice/10-project-final-review/
---

{% include navigation.html %}

# 10단계. 프로젝트 최종 점검 및 정리

이번 단계는 계좌관리 REST API 기본 프로젝트의 마무리 단계이다. 1~9단계에서 작성한 코드를 점검하고, API 정상 동작 여부와 계층별 역할, 데이터베이스 처리 및 예외 응답을 최종 확인한다.

이번 프로젝트의 목표는 Controller → Service → Mapper → Database로 이어지는 Spring Boot 애플리케이션의 기본 구조를 이해하는 것이다.

## 10.1 최종 프로젝트 구조 확인

```text
account-management
├── pom.xml
└── src/main
    ├── java/com/example/accountmanagement
    │   ├── AccountManagementApplication.java
    │   ├── controller/AccountController.java
    │   ├── dto/AccountDto.java
    │   ├── mapper/AccountMapper.java
    │   ├── service/AccountService.java
    │   └── exception
    │       ├── GlobalExceptionHandler.java
    │       ├── AccountNotFoundException.java
    │       └── DuplicateAccountException.java
    └── resources
        ├── application.yml
        ├── schema.sql
        ├── data.sql
        └── mapper/AccountMapper.xml
```

실제 프로젝트의 파일명과 패키지 구조가 다르면 현재 작성한 구조를 기준으로 확인한다.

| 파일 | 역할 |
|---|---|
| `AccountManagementApplication.java` | Spring Boot 애플리케이션 시작 |
| `AccountController.java` | HTTP 요청 수신 및 응답 반환 |
| `AccountService.java` | 계좌 업무 로직 및 트랜잭션 관리 |
| `AccountMapper.java` | MyBatis SQL 호출 인터페이스 |
| `AccountMapper.xml` | 실제 SQL 정의 |
| `AccountDto.java` | 계좌 데이터 전달 객체 |
| `GlobalExceptionHandler.java` | 공통 예외 처리 |
| `AccountNotFoundException.java` | 계좌 미존재 예외 |
| `DuplicateAccountException.java` | 중복 계좌 예외 |
| `application.yml` | 서버, DB, MyBatis 설정 |
| `schema.sql` | 테이블 생성 |
| `data.sql` | 초기 데이터 등록 |

## 10.2 Controller → Service → Mapper → DB 흐름 복습

전체 계좌 조회를 예로 들어 호출 흐름을 확인한다.

```text
Postman
   |
   | GET /api/accounts
   v
AccountController
   |
   | accountService.findAll()
   v
AccountService
   |
   | accountMapper.findAll()
   v
AccountMapper.java
   |
   | namespace + SQL id로 XML 연결
   v
AccountMapper.xml
   |
   | SELECT 실행
   v
H2 Database
   |
   | 조회 결과 반환
   v
AccountDto 목록
   |
   v
Controller에서 JSON 응답 반환
```

- Controller: HTTP 요청과 응답을 처리한다.
- Service: 업무 규칙을 적용하고 데이터 처리를 조정한다.
- Mapper 인터페이스: XML에 정의된 SQL을 호출한다.
- Mapper XML: 실제 SQL을 관리한다.
- Database: 데이터를 저장하고 조회한다.
- DTO: 계층 간 데이터를 전달한다.
- Exception Handler: 예외를 HTTP 오류 응답으로 변환한다.

## 10.3 API 최종 테스트

Postman에서 다음 API를 순서대로 테스트한다.

| 순서 | Method | URL | 기능 | 정상 응답 |
|---|---|---|---|---:|
| 1 | GET | `/api/accounts` | 전체 계좌 조회 | 200 |
| 2 | GET | `/api/accounts/1000000001` | 계좌 상세 조회 | 200 |
| 3 | POST | `/api/accounts` | 신규 계좌 등록 | 201 |
| 4 | PUT | `/api/accounts/1000000001` | 계좌 정보 수정 | 200 |

Base URL:

```text
http://localhost:8080
```

### ① 전체 계좌 조회

GET 요청:

```text
http://localhost:8080/api/accounts
```

확인 내용:
- 계좌 목록이 JSON 배열로 반환되는가?
- 계좌번호, 계좌명, 예금주, 잔액이 정상 표시되는가?
- 한글이 깨지지 않는가?

### ② 계좌 상세 조회

GET 요청:

```text
http://localhost:8080/api/accounts/1000000001
```

확인 내용:
- URL의 계좌번호에 해당하는 데이터 한 건이 반환되는가?
- 다른 계좌번호를 입력하면 해당 계좌가 조회되는가?

### ③ 신규 계좌 등록

POST 요청:

```text
http://localhost:8080/api/accounts
```

Headers:

```text
Content-Type: application/json
```

Body → raw → JSON:

```json
{
    "accountNo": "1000000010",
    "accountName": "최종테스트통장",
    "customerName": "테스트사용자",
    "balance": 1000000
}
```

확인 내용:
- 신규 계좌가 등록되는가?
- HTTP 201 응답이 반환되는가?
- GET 상세 조회로 등록 결과를 확인할 수 있는가?

테스트할 때는 DB에 없는 계좌번호를 사용한다. 이미 존재한다면 다른 번호로 변경한다.

### ④ 계좌 정보 수정

PUT 요청:

```text
http://localhost:8080/api/accounts/1000000010
```

Body → raw → JSON:

```json
{
    "accountName": "최종테스트통장_수정",
    "customerName": "테스트사용자",
    "balance": 1500000
}
```

확인 내용:
- URL의 계좌번호를 기준으로 수정되는가?
- 요청 본문에 계좌번호를 포함하지 않아도 되는가?
- HTTP 200 응답이 반환되는가?
- 다시 GET 요청했을 때 변경된 값이 조회되는가?

## 10.4 예외 및 오류 응답 최종 테스트

9단계에서 구현한 예외 처리도 다시 확인한다.

| 테스트 상황 | 요청 | 예상 HTTP 상태 |
|---|---|---:|
| 잘못된 계좌번호 입력 | 빈 계좌번호로 POST | 400 |
| 음수 잔액 입력 | `balance: -1000`으로 POST | 400 |
| 존재하지 않는 계좌 조회 | GET `/api/accounts/9999999999` | 404 |
| 중복 계좌 등록 | 기존 계좌번호로 POST | 409 |
| 존재하지 않는 계좌 수정 | PUT `/api/accounts/9999999999` | 404 |

중복 계좌 응답 예시:

```json
{
    "status": "409",
    "error": "Conflict",
    "message": "이미 등록된 계좌입니다."
}
```

계좌 미존재 응답 예시:

```json
{
    "status": "404",
    "error": "Not Found",
    "message": "존재하지 않는 계좌입니다."
}
```

실제 메시지는 Service에 작성한 예외 메시지와 일치해야 한다.

## 10.5 트랜잭션 적용 확인

Service의 데이터 변경 메서드에 `@Transactional`이 적용되어 있는지 확인한다.

```java
@Transactional
public int createAccount(AccountDto accountDto) {
    // 입력값 검증
    // 중복 계좌 확인
    // INSERT 실행
}
```

```java
@Transactional
public int updateAccount(
        String accountNo,
        AccountDto accountDto) {
    // 수정 대상 확인
    // 입력값 검증
    // UPDATE 실행
}
```

조회 메서드는 다음과 같이 작성할 수 있다.

```java
@Transactional(readOnly = true)
public List<AccountDto> findAll() {
    return accountMapper.findAll();
}
```

트랜잭션은 하나의 업무 처리를 논리적인 작업 단위로 묶는다. Spring의 기본 트랜잭션 설정에서는 일반적으로 RuntimeException 또는 Error가 발생하면 롤백된다. 이번 프로젝트의 `IllegalArgumentException`, `AccountNotFoundException`, `DuplicateAccountException`은 RuntimeException 계열이므로 기본 롤백 대상이다.

## 10.6 DB 기본키 및 중복 제약조건 확인

Service에서 중복 계좌번호를 조회하는 것 외에도 DB 차원에서 중복을 방지해야 한다.

예시:

```sql
CREATE TABLE ACCOUNT (
    ACCOUNT_NO VARCHAR(20) PRIMARY KEY,
    ACCOUNT_NAME VARCHAR(100) NOT NULL,
    CUSTOMER_NAME VARCHAR(100) NOT NULL,
    BALANCE DECIMAL(15, 2) NOT NULL
);
```

이미 `schema.sql`에 ACCOUNT 테이블이 정의되어 있다면 현재 구조를 먼저 확인한다. 테이블을 다시 생성하거나 기존 데이터를 초기화할 필요는 없다.

Service의 중복 SELECT만으로는 동시에 같은 계좌번호를 등록하는 상황을 완전히 막을 수 없다. DB의 PRIMARY KEY 또는 UNIQUE 제약조건이 최종적인 중복 방지 역할을 한다.

## 10.7 프로젝트 실행 방법 정리

### ① VS Code에서 프로젝트 열기

`account-management` 프로젝트의 최상위 폴더를 연다.

### ② Spring Boot 실행

Maven이 설치된 환경:

```powershell
mvn spring-boot:run
```

Maven Wrapper를 사용하는 환경:

```powershell
.\mvnw.cmd spring-boot:run
```

### ③ 서버 실행 확인

터미널에서 다음과 유사한 로그를 확인한다.

```text
Tomcat started on port 8080 (http)
Started AccountManagementApplication
```

### ④ API 호출

Postman에서 다음 주소로 요청한다.

```text
http://localhost:8080/api/accounts
```

### ⑤ 서버 종료

터미널에서 `Ctrl + C`를 눌러 서버를 종료한다.

## 10.8 README.md 작성 예시

프로젝트 최상위 폴더에 README.md를 작성하면 프로젝트의 목적과 실행 방법을 정리할 수 있다.

```markdown
# Account Management REST API

## 1. 프로젝트 개요

Spring Boot와 MyBatis를 이용한 계좌관리 REST API 학습 프로젝트

## 2. 개발 환경

- Java 17
- Spring Boot
- Maven
- MyBatis
- H2 Database
- Postman
- VS Code

## 3. 주요 기능

- 전체 계좌 조회
- 계좌 상세 조회
- 계좌 등록
- 계좌 정보 수정
- 공통 예외 처리

## 4. 프로젝트 실행

mvn spring-boot:run

## 5. API 기본 주소

http://localhost:8080/api/accounts

## 6. 주요 기술 구조

Controller → Service → Mapper → Database
```

실제 적용한 Spring Boot 및 라이브러리 버전은 `pom.xml`을 확인하여 README에 추가한다.

## 10.9 최종 학습 내용 정리

| 기술 | 학습 내용 |
|---|---|
| Java | 클래스, 객체, 생성자, Getter/Setter |
| Lombok | 반복 코드 자동 생성 |
| Spring Boot | 애플리케이션 구성 및 실행 |
| REST API | GET, POST, PUT |
| Controller | HTTP 요청 및 응답 |
| Service | 업무 로직 및 트랜잭션 |
| MyBatis | Mapper 인터페이스 및 XML SQL |
| H2 | 데이터베이스 연결 및 SQL 실행 |
| DTO | 데이터 전달 객체 |
| Exception | 공통 예외 및 사용자 정의 예외 |
| Postman | API 정상·오류 테스트 |
| Maven | 프로젝트 빌드 및 실행 |

## 10.10 프로젝트 마무리

이번 프로젝트에서 구현한 기능:

- 계좌 전체 조회
- 계좌 상세 조회
- 계좌 신규 등록
- 계좌 정보 수정
- 입력값 및 업무 규칙 검증
- 중복 계좌 등록 방지
- 공통 예외 처리
- HTTP 상태 코드별 오류 응답
- DB 연결 및 SQL 실행
- Postman API 테스트

이 프로젝트는 금융 업무 시스템을 이해하기 위한 기본 CRUD 실습이다. 실제 금융 서비스의 계좌 개설, 잔액 변경, 입출금, 이체, 권한 및 보안 기능까지 구현한 것은 아니다.

기본 프로젝트를 마무리한 뒤에는 별도 심화 과정으로 진행하는 것이 좋다.

### 다음 학습 주제 제안

| 심화 과정 | 학습 주제 |
|---|---|
| 1 | 금융 거래: 입금, 출금, 계좌 이체 |
| 2 | 트랜잭션 심화: 동시성, 롤백, 데이터 정합성 |
| 3 | 테스트 자동화: JUnit, Mockito |
| 4 | API 문서화: Swagger / OpenAPI |
| 5 | 인증 및 권한: Spring Security |
| 6 | 운영 환경: Oracle, Docker, 배포 |

**기본 계좌관리 REST API 프로젝트는 10단계에서 마무리한다.** 이후 심화 주제는 기본 프로젝트에 무리하게 추가하지 않고 별도 실습으로 확장한다.
