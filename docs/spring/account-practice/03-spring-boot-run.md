---
layout: default
title: 3단계 - Spring Boot 기본 애플리케이션 실행
permalink: /spring/account-practice/03-spring-boot-run/
---

{% include navigation.html %}

# 3단계. Spring Boot 기본 애플리케이션 실행

## 1. 시작 클래스 확인

Initializr가 생성한 `src/main/java` 아래의 시작 클래스를 확인한다. Group과 Artifact 설정에 따라 실제 패키지명은 달라질 수 있다.

예시:

```java
package com.example.accountmanagement;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class AccountManagementApplication {
    public static void main(String[] args) {
        SpringApplication.run(AccountManagementApplication.class, args);
    }
}
```

- `@SpringBootApplication`: Spring Boot 시작 클래스임을 나타낸다.
- `main()`: Java 프로그램의 시작 메서드다.
- `SpringApplication.run()`: Spring 애플리케이션을 실행한다.

## 2. 실행

시작 클래스의 `Run` 버튼을 누르거나 프로젝트 루트에서 실행한다.

```powershell
.\mvnw.cmd spring-boot:run
```

## 3. 정상 실행 확인

콘솔에서 다음과 유사한 로그를 확인한다.

```text
Tomcat started on port 8080
Started AccountManagementApplication
```

기본 포트는 일반적으로 8080이다.

## 문제 발생 시

| 증상 | 확인 사항 |
|---|---|
| Java 버전 오류 | 프로젝트 JDK가 Java 17인지 확인 |
| 포트 충돌 | 8080 포트를 다른 프로그램이 사용 중인지 확인 |
| 의존성 다운로드 실패 | 네트워크와 Maven 로그 확인 |
| 시작 클래스 오류 | 패키지명 및 애너테이션 확인 |

## 완료 체크

- [ ] 시작 클래스 확인
- [ ] Spring Boot 실행
- [ ] Started 로그 확인

## 다음 단계

[4단계 - H2 데이터베이스 연결 및 계좌 테이블 생성](../04-h2-database/)
