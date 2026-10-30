---
layout: default
title: 1단계 - Java 17 및 VS Code 환경 확인
permalink: /spring/account-practice/01-java-vscode/
---

{% include navigation.html %}

# 1단계. Java 17 및 VS Code 환경 확인

## 1. Java 버전 확인

VS Code 터미널 또는 PowerShell에서 실행한다.

```powershell
java -version
```

현재 확인된 환경은 **OpenJDK 17.0.18 (Eclipse Temurin)**이다. 이번 실습은 Java 17을 기준으로 진행한다.

JAVA_HOME은 다음 명령어로 확인할 수 있다.

```powershell
$env:JAVA_HOME
```

## 2. VS Code Java 확장 기능 설치

1. VS Code 왼쪽 Extensions 아이콘을 선택한다.
2. `Extension Pack for Java`를 검색한다.
3. Microsoft 제공 확장 팩을 설치한다.
4. 설치 후 VS Code를 다시 시작한다.

Java 프로젝트 탐색, 코드 자동 완성, 실행 및 디버깅 기능을 사용할 수 있다.

## 3. Maven 확인에 관한 참고

`mvn -version`은 Maven이 별도 설치되고 PATH에 등록된 경우 실행된다. Maven이 없으면 명령어를 찾지 못할 수 있다.

이번 실습에서는 Spring Initializr가 제공하는 Maven Wrapper를 사용한다. 프로젝트 생성 후 루트 폴더에서 다음을 실행한다.

```powershell
.\mvnw.cmd -version
```

따라서 Maven을 지금 별도로 설치하지 않아도 된다.

## 완료 체크

- [x] Java 17 버전 확인
- [ ] Extension Pack for Java 설치 확인
- [ ] 프로젝트 생성 후 Maven Wrapper 확인

## 다음 단계

[2단계 - Spring Initializr 프로젝트 생성 및 VS Code에서 열기](../02-project-create/)
