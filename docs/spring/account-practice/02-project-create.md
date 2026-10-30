---
layout: default
title: 2단계 - Spring Initializr 프로젝트 생성 및 열기
permalink: /spring/account-practice/02-project-create/
---

{% include navigation.html %}

# 2단계. Spring Initializr 프로젝트 생성 및 VS Code에서 열기

## 1. Spring Initializr 접속

[Spring Initializr](https://start.spring.io/)에서 프로젝트를 생성한다.

## 2. 기본 설정

| 항목 | 설정값 |
|---|---|
| Project | Maven |
| Language | Java |
| Spring Boot | Java 17 호환 안정 버전 |
| Group | `com.example` |
| Artifact | `account-management` |
| Name | `account-management` |
| Packaging | Jar |
| Java | 17 |

## 3. Dependencies

다음 항목을 추가한다.

- **Spring Web**: REST API 구현
- **MyBatis Framework**: SQL 실행 및 결과 매핑
- **H2 Database**: 실습용 데이터베이스

처음에는 Lombok을 추가하지 않고 생성자, getter, setter를 직접 작성한다.

## 4. 다운로드 및 VS Code에서 열기

1. `Generate`를 눌러 ZIP을 다운로드한다.
2. 압축을 해제한다. 예: `C:\study\workspace\account-management`
3. VS Code에서 `File > Open Folder`를 선택한다.
4. `pom.xml`이 들어 있는 프로젝트 최상위 폴더를 연다. `src` 폴더만 열지 않는다.

## 5. Maven Wrapper 확인

VS Code 터미널에서 프로젝트 루트 폴더인지 확인한 후 실행한다.

```powershell
.\mvnw.cmd -version
```

최초 실행 시 Maven 배포 파일을 내려받을 수 있어 인터넷 연결이 필요할 수 있다.

## 완료 체크

- [ ] 프로젝트 생성 및 압축 해제
- [ ] Spring Web, MyBatis Framework, H2 Database 선택
- [ ] VS Code에서 프로젝트 루트 열기
- [ ] `pom.xml`, `mvnw.cmd` 확인
- [ ] Maven Wrapper 실행 확인

## 다음 단계

[3단계 - Spring Boot 기본 애플리케이션 실행](../03-spring-boot-run/)
