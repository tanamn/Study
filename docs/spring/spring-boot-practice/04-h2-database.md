---
layout: default
title: 4단계 - H2 데이터베이스 연결 및 계좌 테이블 생성
permalink: /spring/spring-boot-practice/04-h2-database/
---

{% include navigation.html %}

# 4단계. H2 데이터베이스 연결 및 계좌 테이블 생성

이번 단계에서는 Spring Boot 애플리케이션에 H2 데이터베이스를 연결하고, SQL 스크립트로 계좌 테이블과 샘플 데이터를 생성한다. 마지막으로 H2 콘솔에 접속하여 데이터가 정상적으로 저장되었는지 확인한다.

## 4.1 H2 데이터베이스란?

H2는 Java 기반의 경량 관계형 데이터베이스이다. 별도의 데이터베이스 서버를 설치하지 않고도 로컬 개발 및 테스트에 사용할 수 있다.

이번 실습에서는 H2의 메모리 모드(`mem`)를 사용한다.

- 데이터베이스 이름: `accountdb`
- JDBC URL: `jdbc:h2:mem:accountdb`
- 사용자 이름: `sa`
- 비밀번호: 공란
- 데이터 저장 방식: 메모리 기반

메모리 데이터베이스는 Spring Boot 애플리케이션이 종료되면 데이터가 사라진다. 애플리케이션을 다시 실행하면 `schema.sql`과 `data.sql`이 다시 실행되어 테이블과 초기 데이터가 생성된다.

## 4.2 프로젝트 파일 위치 확인

다음 파일은 모두 `src/main/resources` 바로 아래에 생성한다.

```text
account-management/
└── src/
    └── main/
        ├── java/
        │   └── com/example/accountmanagement/
        │       └── AccountManagementApplication.java
        └── resources/
            ├── application.yml
            ├── schema.sql
            └── data.sql
```

- `application.yml`: 데이터베이스 연결 및 Spring Boot 설정
- `schema.sql`: 테이블 생성 SQL
- `data.sql`: 초기 데이터 입력 SQL

## 4.3 application.yml 설정

`src/main/resources/application.yml`에 다음 설정을 작성한다.

```yaml
spring:
  application:
    name: account-management

  # H2 데이터베이스 연결 설정
  datasource:
    url: jdbc:h2:mem:accountdb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE
    driver-class-name: org.h2.Driver
    username: sa
    password:

  # H2 웹 콘솔 설정
  h2:
    console:
      enabled: true
      path: /h2-console

  # SQL 초기화 설정
  sql:
    init:
      mode: always
      encoding: UTF-8

# MyBatis 설정
mybatis:
  mapper-locations: classpath:mapper/**/*.xml
  type-aliases-package: com.example.accountmanagement.dto
  configuration:
    map-underscore-to-camel-case: true

# 내장 웹 서버 설정
server:
  port: 8080
```

### 주요 설정 설명

| 설정 | 설명 |
|---|---|
| `jdbc:h2:mem:accountdb` | `accountdb`라는 이름의 메모리 데이터베이스 사용 |
| `DB_CLOSE_DELAY=-1` | 마지막 연결이 닫혀도 JVM이 종료될 때까지 메모리 DB 유지 |
| `DB_CLOSE_ON_EXIT=FALSE` | H2 자체 종료 훅을 사용하지 않도록 설정 |
| `username: sa` | H2 기본 사용자 이름 |
| `password:` | 비밀번호를 사용하지 않으므로 공란 |
| `h2.console.enabled: true` | H2 웹 콘솔 활성화 |
| `h2.console.path: /h2-console` | 콘솔 접속 경로 지정 |
| `sql.init.mode: always` | 애플리케이션 시작 시 SQL 초기화 스크립트 실행 |
| `sql.init.encoding: UTF-8` | SQL 초기화 파일을 UTF-8 인코딩으로 읽음 |
| `server.port: 8080` | 내장 웹 서버 포트 지정 |

`DB_CLOSE_DELAY=-1`은 애플리케이션이 실행 중인 동안 메모리 DB를 유지하기 위한 설정이다. Spring Boot 프로세스 자체를 종료하면 메모리 DB와 데이터는 사라진다.

`mybatis.mapper-locations`와 `mybatis.type-aliases-package`는 다음 단계에서 Mapper XML과 DTO를 사용할 때 적용할 설정이다. 실제 DTO 패키지 경로와 일치해야 한다.

## 4.4 H2 콘솔 의존성 확인

Spring Boot 4.x에서는 H2 웹 콘솔 기능이 별도 모듈로 제공된다. `pom.xml`의 `<dependencies>` 영역에 다음 의존성이 없을 경우 추가한다.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-h2console</artifactId>
</dependency>
```

버전은 Spring Boot의 의존성 관리에 맡기므로 별도로 지정하지 않는다.

프로젝트에는 H2 데이터베이스 의존성도 있어야 한다. Spring Initializr에서 추가한 `H2 Database` 의존성을 유지한다.

## 4.5 계좌 테이블 생성: schema.sql

`src/main/resources/schema.sql` 파일에 다음 SQL을 작성한다.

```sql
CREATE TABLE IF NOT EXISTS ACCOUNT (
    ACCOUNT_NO VARCHAR(20) PRIMARY KEY,
    ACCOUNT_NAME VARCHAR(100) NOT NULL,
    CUSTOMER_NAME VARCHAR(100) NOT NULL,
    BALANCE DECIMAL(15, 2) DEFAULT 0 NOT NULL
);
```

### 컬럼 설명

| 컬럼명 | 데이터 타입 | 설명 |
|---|---|---|
| `ACCOUNT_NO` | `VARCHAR(20)` | 계좌번호, 기본키 |
| `ACCOUNT_NAME` | `VARCHAR(100)` | 계좌 별칭 또는 통장명 |
| `CUSTOMER_NAME` | `VARCHAR(100)` | 예금주 |
| `BALANCE` | `DECIMAL(15, 2)` | 계좌 잔액 |

`CREATE TABLE IF NOT EXISTS`는 동일한 테이블이 이미 존재하는 경우 테이블 생성 오류를 방지한다.

## 4.6 초기 데이터 입력: data.sql

`src/main/resources/data.sql` 파일에 다음 SQL을 작성한다.

```sql
INSERT INTO ACCOUNT (ACCOUNT_NO, ACCOUNT_NAME, CUSTOMER_NAME, BALANCE)
VALUES ('1000000001', '급여통장', '홍길동', 1500000.00);

INSERT INTO ACCOUNT (ACCOUNT_NO, ACCOUNT_NAME, CUSTOMER_NAME, BALANCE)
VALUES ('1000000002', '생활비통장', '김영희', 850000.00);

INSERT INTO ACCOUNT (ACCOUNT_NO, ACCOUNT_NAME, CUSTOMER_NAME, BALANCE)
VALUES ('1000000003', '저축통장', '이철수', 3200000.00);
```

### 초기 데이터

| 계좌번호 | 계좌명 | 예금주 | 잔액 |
|---|---|---|---:|
| 1000000001 | 급여통장 | 홍길동 | 1,500,000 |
| 1000000002 | 생활비통장 | 김영희 | 850,000 |
| 1000000003 | 저축통장 | 이철수 | 3,200,000 |

## 4.7 SQL 파일 인코딩 확인

한글이 포함된 `data.sql`은 UTF-8로 저장해야 한다.

VS Code에서 `data.sql` 파일을 연 다음 오른쪽 아래 상태 표시줄의 인코딩 표시를 확인한다.

1. 인코딩 표시를 클릭한다.
2. `Save with Encoding`을 선택한다.
3. `UTF-8`을 선택한다.
4. 파일을 저장한다.

`application.yml`의 `spring.sql.init.encoding: UTF-8` 설정도 함께 적용한다. 이 설정은 Spring Boot가 SQL 초기화 파일을 UTF-8로 읽도록 지정한다.

### 한글 깨짐이 발생했던 사례

이번 실습에서는 처음 H2 콘솔에서 한글이 깨져 보이는 현상이 발생했다. SQL 파일을 UTF-8로 저장하고 `application.yml`에 SQL 초기화 인코딩을 지정한 후 애플리케이션을 재실행하자 한글이 정상적으로 조회되었다.

문제가 발생하면 다음 두 항목을 함께 확인한다.

- `data.sql` 파일 자체가 UTF-8로 저장되어 있는지
- `spring.sql.init.encoding: UTF-8` 설정이 적용되어 있는지

주의: 인코딩을 수정하기 전에 잘못된 문자로 이미 저장된 데이터는 설정 변경만으로 자동 복구되지 않는다. 메모리 DB를 사용하는 이번 실습에서는 애플리케이션을 완전히 재시작하여 SQL 초기화 스크립트를 다시 실행하면 된다.

## 4.8 Spring Boot 애플리케이션 실행

VS Code 터미널에서 프로젝트 최상위 폴더(`account-management`)로 이동한 상태인지 확인한다.

### ① 기존 실행 종료

이미 Spring Boot가 실행 중이라면 터미널에서 `Ctrl + C`를 눌러 종료한다.

### ② 애플리케이션 실행

```powershell
mvn spring-boot:run
```

빌드와 애플리케이션 시작이 정상적으로 완료되면 다음과 유사한 로그가 표시된다.

```text
Tomcat started on port 8080 (http) with context path '/'
Started AccountManagementApplication
```

- `Tomcat started on port 8080`: 내장 Tomcat 웹 서버가 8080 포트에서 시작되었다는 의미
- `Started AccountManagementApplication`: Spring Boot 애플리케이션 시작이 완료되었다는 의미

애플리케이션 실행 중에는 이 터미널을 닫거나 `Ctrl + C`로 종료하지 않는다.

## 4.9 H2 콘솔 접속 및 로그인

H2 콘솔은 브라우저에서 데이터베이스에 접속해 테이블과 데이터를 확인하고 SQL을 직접 실행하는 웹 관리 화면이다. Oracle SQL Developer나 DBeaver에서 데이터베이스에 접속해 SQL을 실행하는 것과 유사한 역할을 한다.

H2 Database는 실제 데이터를 저장하는 데이터베이스이고, H2 Console은 그 데이터베이스를 확인하고 조작하는 관리 화면이다.

### ① 브라우저 접속

Spring Boot가 실행 중인 상태에서 Chrome 주소창에 다음 주소를 입력한다.

```text
http://localhost:8080/h2-console
```

### ② 로그인 화면 확인

아래는 이번 실습에서 실제로 표시된 H2 콘솔 로그인 화면이다.

![H2 콘솔 로그인 화면]({{ '/spring/spring-boot-practice/assets/images/h2-console-login.png' | relative_url }})

### ③ 접속 정보 입력

| 항목 | 입력값 |
|---|---|
| Driver Class | `org.h2.Driver` |
| JDBC URL | `jdbc:h2:mem:accountdb;DB_CLOSE_DELAY=-1;DB_CLOSE_ON_EXIT=FALSE` |
| User Name | `sa` |
| Password | 빈칸 |

JDBC URL은 `application.yml`의 `spring.datasource.url` 값과 동일하게 입력해야 한다. URL이 서로 다르면 애플리케이션이 사용하는 DB가 아닌 다른 데이터베이스에 접속할 수 있다.

입력 후 **Connect** 버튼을 클릭한다.

## 4.10 ACCOUNT 테이블 및 데이터 조회

접속에 성공하면 H2 콘솔의 SQL 편집 화면이 나타난다. 왼쪽에는 테이블 목록이 표시되고, 오른쪽 SQL 입력 영역에서 쿼리를 실행할 수 있다.

### ① ACCOUNT 테이블 확인

왼쪽 테이블 목록에 `ACCOUNT`가 표시되는지 확인한다.

### ② 계좌 데이터 조회

SQL 입력창에 다음 쿼리를 입력하고 `Run` 버튼을 클릭한다.

```sql
SELECT * FROM ACCOUNT;
```

### ③ 실제 조회 결과

이번 실습에서 계좌 데이터 3건이 정상적으로 조회된 화면이다.

![H2 콘솔에서 ACCOUNT 데이터 조회 성공]({{ '/spring/spring-boot-practice/assets/images/h2-console-account-query-success.png' | relative_url }})

조회 결과는 다음과 같다.

| ACCOUNT_NO | ACCOUNT_NAME | CUSTOMER_NAME | BALANCE |
|---|---|---|---:|
| 1000000001 | 급여통장 | 홍길동 | 1500000.00 |
| 1000000002 | 생활비통장 | 김영희 | 850000.00 |
| 1000000003 | 저축통장 | 이철수 | 3200000.00 |

### ④ 추가 SQL 조회 실습

특정 계좌만 조회할 수도 있다.

```sql
SELECT *
FROM ACCOUNT
WHERE ACCOUNT_NO = '1000000001';
```

계좌 잔액 합계를 조회할 수도 있다.

```sql
SELECT SUM(BALANCE) AS TOTAL_BALANCE
FROM ACCOUNT;
```

## 4.11 자주 발생하는 문제

| 증상 | 확인 사항 |
|---|---|
| `/h2-console` 접속 시 404 | `spring-boot-h2console` 의존성과 `spring.h2.console.enabled: true` 확인 |
| H2 콘솔 로그인 실패 | JDBC URL, 사용자 이름, 비밀번호 확인 |
| `ACCOUNT` 테이블이 표시되지 않음 | `schema.sql` 위치와 애플리케이션 시작 로그 확인 |
| 테이블은 있지만 데이터가 없음 | `data.sql` SQL 문법 및 초기화 로그 확인 |
| 한글이 깨져 보임 | `data.sql`의 UTF-8 저장 여부와 `spring.sql.init.encoding: UTF-8` 확인 |
| 재실행 후 데이터가 초기화됨 | 메모리 DB는 애플리케이션 재시작 시 데이터가 사라지는 것이 정상 |

### 메모리 DB의 특징

이번 실습은 `jdbc:h2:mem:accountdb` 형태의 메모리 DB를 사용한다.

- 애플리케이션이 실행 중인 동안 데이터가 유지된다.
- 애플리케이션을 종료하면 데이터가 사라진다.
- 다시 실행하면 `schema.sql`과 `data.sql`이 다시 실행된다.

H2 콘솔은 개발 및 테스트 용도이다. 실제 운영 환경에서는 콘솔을 비활성화하고 외부 접근을 제한해야 한다.

## 4.12 4단계 완료 체크

- [ ] `application.yml`에 H2 데이터베이스 연결 정보 작성
- [ ] `application.yml`에 H2 콘솔 경로 설정
- [ ] `application.yml`에 SQL 초기화 및 UTF-8 인코딩 설정
- [ ] `pom.xml`에 `spring-boot-h2console` 의존성 추가
- [ ] `schema.sql`에 ACCOUNT 테이블 생성 SQL 작성
- [ ] `data.sql`에 계좌 초기 데이터 3건 입력
- [ ] `schema.sql`, `data.sql`을 UTF-8로 저장
- [ ] `mvn spring-boot:run`으로 애플리케이션 실행
- [ ] H2 콘솔 로그인 성공
- [ ] H2 콘솔에서 ACCOUNT 테이블 확인
- [ ] 계좌 데이터 3건 및 한글이 정상 조회되는지 확인

## 다음 단계

5단계에서는 Java DTO와 MyBatis Mapper 인터페이스를 작성한다. H2 콘솔에서 직접 실행했던 SQL을 Java 애플리케이션에서 호출할 수 있도록 데이터 전달 객체와 Mapper의 역할을 학습한다.
