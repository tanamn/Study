---
layout: default
title: 3단계 - Spring Boot 애플리케이션 실행 및 결과 확인
permalink: /spring/spring-boot-practice/03-spring-boot-run/
---

{% include navigation.html %}

# 3단계. Spring Boot 애플리케이션 실행 및 결과 확인

이번 단계에서는 Spring Initializr로 생성한 `account-management` 프로젝트를 빌드하고 실행한다. 실행 로그와 브라우저 접속을 통해 Spring Boot와 내장 Tomcat 웹 서버가 정상적으로 동작하는지 확인한다.

## 실행 전 시작 클래스 확인

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

## 3.1 실행 환경 확인

이번 실습에서 사용하는 개발 환경은 다음과 같다.

| 항목 | 버전 또는 설정 |
|---|---|
| Java | OpenJDK 17.0.18 (Eclipse Adoptium) |
| Maven | 3.9.16 |
| Spring Boot | 4.0.9 |
| 프로젝트 | `account-management` |
| 실행 포트 | 8080 |

프로젝트를 실행하기 전에 VS Code 터미널의 현재 디렉터리가 `account-management` 프로젝트의 최상위 폴더인지 확인한다. 최상위 폴더에는 `pom.xml` 파일이 있어야 한다.

## 3.2 Maven 빌드

먼저 Maven으로 프로젝트를 빌드한다.

```powershell
mvn clean package
```

명령어의 의미는 다음과 같다.

- `clean`: 이전 빌드 과정에서 생성된 결과물을 삭제한다.
- `package`: 소스 코드를 컴파일하고 테스트한 후 실행 가능한 JAR 파일을 패키징한다.

빌드가 정상적으로 끝나면 터미널 마지막 부분에 다음 메시지가 표시된다.

```text
[INFO] BUILD SUCCESS
```

`BUILD SUCCESS`는 Maven 빌드가 성공했다는 의미다. 애플리케이션 서버가 실행 중이라는 뜻은 아니므로, 다음 단계에서 애플리케이션을 별도로 실행한다.

## 3.3 Spring Boot 애플리케이션 실행

프로젝트 최상위 폴더에서 다음 명령어를 실행한다.

```powershell
mvn spring-boot:run
```

이 명령어는 Maven의 Spring Boot 플러그인을 이용해 애플리케이션을 실행한다.

프로젝트에 Maven Wrapper가 포함되어 있다면 Windows에서는 아래 명령어를 사용해도 된다.

```powershell
.\mvnw.cmd spring-boot:run
```

## 3.4 실행 결과 확인

실행이 완료되면 VS Code 터미널에 다음과 유사한 로그가 표시된다. 아래 화면은 이번 실습에서 실제로 실행한 결과다.

![Spring Boot 애플리케이션 실행 성공 로그]({{ '/spring/spring-boot-practice/assets/images/spring-boot-run-success.png' | relative_url }})

로그에서 다음 두 항목을 확인한다.

```text
Tomcat started on port 8080 (http) with context path '/'
Started AccountManagementApplication in 1.27 seconds
```

| 로그 | 의미 |
|---|---|
| `Tomcat started on port 8080 (http)` | 내장 Tomcat 웹 서버가 HTTP 8080 포트에서 시작되었다. |
| `with context path '/'` | 애플리케이션의 기본 Context Path가 `/`로 설정되어 있다. |
| `Started AccountManagementApplication` | Spring Boot 애플리케이션 시작이 완료되었다. |
| `process running` | 애플리케이션 프로세스가 실행 중이며 터미널이 이를 유지하고 있다. |

위 로그가 표시되면 Spring Boot 애플리케이션과 내장 Tomcat이 정상적으로 기동된 것이다.

## 3.5 브라우저에서 접속 확인

Chrome 등 웹 브라우저의 주소창에 다음 URL을 입력한다.

```text
http://localhost:8080
```

현재 단계에서는 Controller와 URL 매핑을 아직 만들지 않았다. 따라서 브라우저에 `Whitelabel Error Page` 또는 HTTP 404 화면이 표시될 수 있다.

이는 서버가 실행되지 않았다는 의미가 아니다. Tomcat이 요청을 받았지만 해당 URL에 연결된 Controller나 화면이 아직 없다는 뜻이다. 현재 단계에서는 터미널의 `Tomcat started` 및 `Started AccountManagementApplication` 로그를 통해 서버 기동 여부를 확인한다.

이후 Controller와 API를 구현하면 해당 URL로 실제 응답을 확인할 수 있다.

## 3.6 애플리케이션 종료

애플리케이션이 실행 중인 터미널에서 `Ctrl + C`를 누르면 서버를 종료할 수 있다.

서버를 종료하지 않은 상태에서 같은 포트로 애플리케이션을 다시 실행하면 8080 포트 충돌이 발생할 수 있으므로, 재실행할 때는 기존 실행 프로세스가 종료되었는지 확인한다.

## 3.7 문제 발생 시 확인 사항

| 증상 | 확인 사항 |
|---|---|
| `mvn` 명령을 찾을 수 없음 | Maven 설치 및 Windows PATH 설정 확인 |
| Java 버전 오류 | `java -version`으로 Java 17 사용 여부 확인 |
| 8080 포트 사용 중 오류 | 다른 프로그램 또는 기존 Spring Boot 프로세스가 8080 포트를 사용 중인지 확인 |
| 의존성 다운로드 실패 | 네트워크 연결 및 Maven 오류 로그 확인 |
| 브라우저에서 404 표시 | 현재 Controller와 URL 매핑을 아직 만들지 않았다면 정상일 수 있음 |

## 3.8 완료 체크

- [ ] 프로젝트 최상위 폴더에서 `mvn clean package` 실행
- [ ] `BUILD SUCCESS` 확인
- [ ] `mvn spring-boot:run` 실행
- [ ] `Tomcat started on port 8080` 로그 확인
- [ ] `Started AccountManagementApplication` 로그 확인
- [ ] 브라우저에서 `http://localhost:8080` 접속 시도
- [ ] `Ctrl + C`로 애플리케이션 종료 방법 확인

## 다음 학습 단계

다음 단계에서는 H2 데이터베이스를 설정하고 계좌 정보를 저장할 테이블을 생성한다.

[4단계 - H2 데이터베이스 연결 및 계좌 테이블 생성]({{ '/spring/spring-boot-practice/04-h2-database/' | relative_url }})
