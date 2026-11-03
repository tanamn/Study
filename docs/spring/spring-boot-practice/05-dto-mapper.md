---
layout: default
title: 5단계 - DTO 및 MyBatis Mapper 인터페이스
permalink: /spring/spring-boot-practice/05-dto-mapper/
---

{% include navigation.html %}

# 5단계. DTO 및 MyBatis Mapper 인터페이스 작성

이번 단계에서는 계좌 정보를 Java 객체로 전달하기 위한 DTO와 MyBatis Mapper 인터페이스를 작성한다.

먼저 Java에서 생성자, Getter, Setter가 어떤 역할을 하는지 직접 작성한 코드로 확인하고, 이어서 Lombok 어노테이션으로 동일한 코드를 간결하게 작성하는 방법도 학습한다.

## 5.1 AccountDto란?

DTO(Data Transfer Object)는 계층 간에 데이터를 전달하기 위해 사용하는 객체이다.

이번 프로젝트에서는 계좌 정보를 다음 필드로 표현한다.

| 필드 | Java 타입 | 설명 |
|---|---|---|
| `accountNo` | `String` | 계좌번호 |
| `accountName` | `String` | 계좌명 |
| `customerName` | `String` | 예금주 |
| `balance` | `BigDecimal` | 계좌 잔액 |

금액은 부동소수점 오차를 피하기 위해 `double` 대신 `BigDecimal`을 사용한다.

파일 위치:

```text
src/main/java/com/example/accountmanagement/dto/AccountDto.java
```

## 5.2 Lombok을 적용하기 전: 직접 작성하는 방식

먼저 Lombok을 사용하지 않고 필드, 생성자, Getter, Setter를 직접 작성하면 다음과 같다.

```java
package com.example.accountmanagement.dto;

import java.math.BigDecimal;

public class AccountDto {

    private String accountNo;
    private String accountName;
    private String customerName;
    private BigDecimal balance;

    // 기본 생성자
    public AccountDto() {
    }

    // 모든 필드를 전달받는 생성자
    public AccountDto(String accountNo, String accountName,
                      String customerName, BigDecimal balance) {
        this.accountNo = accountNo;
        this.accountName = accountName;
        this.customerName = customerName;
        this.balance = balance;
    }

    // accountNo Getter / Setter
    public String getAccountNo() {
        return accountNo;
    }

    public void setAccountNo(String accountNo) {
        this.accountNo = accountNo;
    }

    // accountName Getter / Setter
    public String getAccountName() {
        return accountName;
    }

    public void setAccountName(String accountName) {
        this.accountName = accountName;
    }

    // customerName Getter / Setter
    public String getCustomerName() {
        return customerName;
    }

    public void setCustomerName(String customerName) {
        this.customerName = customerName;
    }

    // balance Getter / Setter
    public BigDecimal getBalance() {
        return balance;
    }

    public void setBalance(BigDecimal balance) {
        this.balance = balance;
    }
}
```

### 생성자 두 가지를 작성한 이유

| 생성자 | 역할 | 사용 예 |
|---|---|---|
| 기본 생성자 | 매개변수 없이 객체 생성 | `new AccountDto()` |
| 전체 필드 생성자 | 생성 시 모든 필드 값 전달 | `new AccountDto("1000000001", ...)` |

기본 생성자를 이용하면 빈 객체를 만든 후 Setter로 값을 지정할 수 있다.

```java
AccountDto account = new AccountDto();

account.setAccountNo("1000000001");
account.setAccountName("급여통장");
account.setCustomerName("홍길동");
account.setBalance(new BigDecimal("1500000"));
```

전체 필드 생성자를 이용하면 객체 생성과 동시에 값을 지정할 수 있다.

```java
AccountDto account = new AccountDto(
    "1000000001",
    "급여통장",
    "홍길동",
    new BigDecimal("1500000")
);
```

`this.accountNo = accountNo;`에서 `this.accountNo`는 현재 객체의 필드이고, 오른쪽 `accountNo`는 생성자 매개변수이다. 전달받은 값을 현재 객체의 필드에 저장한다는 의미이다.

이번 MyBatis 실습은 기본 생성자와 Setter를 사용하는 일반적인 객체 매핑 방식으로 진행한다. 따라서 기본 생성자는 유지한다. 전체 필드 생성자는 Java 코드에서 DTO를 직접 생성할 때 사용할 수 있지만, 이번 Mapper 조회에 반드시 필요한 것은 아니다.

## 5.3 Lombok이란?

Lombok은 반복적으로 작성하는 Getter, Setter, 생성자 등의 코드를 어노테이션으로 자동 생성해 주는 라이브러리이다.

직접 메서드를 작성하지 않아도 컴파일 과정에서 필요한 메서드가 만들어지므로 DTO나 Service 등의 코드가 간결해진다.

예를 들어 다음 두 코드는 같은 목적을 수행한다.

직접 작성:

```java
public String getAccountNo() {
    return accountNo;
}

public void setAccountNo(String accountNo) {
    this.accountNo = accountNo;
}
```

Lombok 사용:

```java
@Getter
@Setter
private String accountNo;
```

Lombok은 메서드의 동작을 없애는 것이 아니라, 반복 코드를 컴파일 과정에서 생성해 주는 도구이다.

## 5.4 pom.xml에 Lombok 의존성 추가

프로젝트 최상위의 `pom.xml`을 열고 `<dependencies>` 영역에 다음 의존성을 추가한다.

```xml
<!-- Lombok: Getter, Setter, Constructor 등 자동 생성 -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <scope>provided</scope>
</dependency>
```

### 의존성 설정 설명

| 항목 | 의미 |
|---|---|
| `groupId` | Lombok 라이브러리 그룹 |
| `artifactId` | Lombok 라이브러리 이름 |
| `scope: provided` | 컴파일 시 필요하지만 일반적인 실행 배포 시에는 포함하지 않는 범위 |

현재 프로젝트는 Spring Boot 4.0.9의 Maven 의존성 관리를 사용하므로 Lombok 버전을 별도로 적지 않아도 된다. Spring Boot 4.0.9가 관리하는 Lombok 버전은 1.18.48이다.

의존성 추가 후 저장하고 VS Code의 Maven 프로젝트 새로고침을 실행한다. 필요하면 터미널에서 다음 명령으로 의존성을 확인하고 컴파일한다.

```powershell
mvn clean compile
```

## 5.5 Lombok 주요 어노테이션

| 어노테이션 | 생성 기능 | 설명 |
|---|---|---|
| `@Getter` | Getter | 필드 값을 반환하는 메서드 생성 |
| `@Setter` | Setter | 필드 값을 변경하는 메서드 생성 |
| `@NoArgsConstructor` | 기본 생성자 | 매개변수가 없는 생성자 생성 |
| `@AllArgsConstructor` | 전체 필드 생성자 | 모든 인스턴스 필드를 매개변수로 받는 생성자 생성 |
| `@RequiredArgsConstructor` | 필수 필드 생성자 | `final` 필드와 `@NonNull` 필드를 매개변수로 받는 생성자 생성 |
| `@Data` | 여러 메서드 | Getter, Setter, `toString`, `equals`, `hashCode`, required-args 생성자 등을 제공 |

### ① @Getter와 @Setter

클래스에 선언하면 모든 인스턴스 필드에 Getter와 Setter가 생성된다.

```java
@Getter
@Setter
public class AccountDto {

    private String accountNo;
    private String accountName;
    private String customerName;
    private BigDecimal balance;
}
```

필요한 필드에만 개별적으로 선언할 수도 있다.

```java
public class AccountDto {

    @Getter
    @Setter
    private String accountNo;

    @Getter
    private String customerName;
}
```

위 예제에서 `accountNo`에는 Getter와 Setter가 모두 생성되고, `customerName`에는 Getter만 생성된다.

### ② @NoArgsConstructor와 @AllArgsConstructor

```java
@NoArgsConstructor
@AllArgsConstructor
public class AccountDto {
    // fields
}
```

위 어노테이션은 다음 생성자를 대신한다.

```java
public AccountDto() {
}

public AccountDto(String accountNo, String accountName,
                  String customerName, BigDecimal balance) {
    this.accountNo = accountNo;
    this.accountName = accountName;
    this.customerName = customerName;
    this.balance = balance;
}
```

- `@NoArgsConstructor`: 기본 생성자 생성
- `@AllArgsConstructor`: 모든 인스턴스 필드를 매개변수로 받는 생성자 생성

주의: 직접 생성자를 작성한 상태에서 같은 생성자를 Lombok으로 다시 생성하면 중복 생성자 오류가 발생할 수 있다. Lombok으로 전환할 때는 기존에 직접 작성한 Getter, Setter, 생성자를 제거하고 어노테이션으로 대체한다.

### ③ @Data

`@Data`는 다음 기능을 한 번에 제공하는 어노테이션이다.

- 모든 필드의 Getter
- `final`이 아닌 필드의 Setter
- `toString()`
- `equals()`
- `hashCode()`
- 필수 필드를 위한 생성자

간단한 DTO에서는 편리하지만, `@Data`가 생성하는 `toString()`이 계좌번호나 고객명 같은 정보를 로그에 노출할 수 있다는 점에 주의한다. 민감한 정보를 다루는 DTO에서는 필요한 어노테이션만 선택해서 사용하는 편이 더 명확하다.

## 5.6 AccountDto에 Lombok 적용하기

직접 작성한 코드와 Lombok 코드를 비교한 후, 실제 프로젝트에는 다음과 같이 적용할 수 있다.

```java
package com.example.accountmanagement.dto;

import java.math.BigDecimal;

import lombok.Getter;
import lombok.Setter;
import lombok.NoArgsConstructor;
import lombok.AllArgsConstructor;

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class AccountDto {

    private String accountNo;
    private String accountName;
    private String customerName;
    private BigDecimal balance;
}
```

이 코드에는 Getter, Setter, 기본 생성자, 전체 필드 생성자가 직접 보이지 않는다. 하지만 Lombok이 컴파일 과정에서 해당 메서드와 생성자를 생성한다.

### 실제 호출 방법은 이전과 동일하다

```java
AccountDto account = new AccountDto();

account.setAccountNo("1000000001");
account.setAccountName("급여통장");
account.setCustomerName("홍길동");
account.setBalance(new BigDecimal("1500000"));

String accountNo = account.getAccountNo();
String customerName = account.getCustomerName();
```

Lombok을 사용하더라도 Java 코드에서 Getter와 Setter를 호출하는 방법은 달라지지 않는다.

## 5.7 VS Code에서 Lombok 지원 확인

VS Code에서 Lombok 어노테이션을 인식하고 자동 완성 및 코드 분석을 사용할 수 있도록 Java 확장 지원을 확인한다.

1. `Ctrl + Shift + X`를 눌러 Extensions 화면을 연다.
2. `Extension Pack for Java`가 설치되어 있는지 확인한다.
3. 설치되어 있지 않으면 설치한다.
4. VS Code를 다시 로드한다.
5. `pom.xml`을 저장하고 Maven 프로젝트를 새로고침한다.

Lombok 지원은 VS Code의 Java 확장 환경과 연동된다. 편집기에서 Getter나 Setter가 인식되지 않더라도 먼저 Maven 컴파일 결과를 확인한다.

## 5.8 Lombok 적용 여부 확인

### ① AccountDto.java 작성

5.6절의 Lombok 적용 코드를 저장한다.

### ② Maven 컴파일

프로젝트 최상위 폴더에서 실행한다.

```powershell
mvn clean compile
```

`BUILD SUCCESS`가 표시되면 Lombok이 포함된 코드의 컴파일이 정상적으로 완료된 것이다.

### ③ Getter와 Setter 호출 확인

실제 Java 코드에서 다음 메서드를 호출했을 때 오류가 없어야 한다.

```java
account.setAccountNo("1000000001");

String accountNo = account.getAccountNo();
```

Getter와 Setter를 직접 작성하지 않았더라도 Lombok이 생성한 메서드를 호출할 수 있다.

## 5.9 Service에서 사용하는 Lombok: @RequiredArgsConstructor

DTO의 Getter와 Setter 외에 Spring 개발에서 자주 접하는 Lombok 어노테이션이 `@RequiredArgsConstructor`이다.

이 어노테이션은 `final` 필드 등을 매개변수로 받는 생성자를 생성한다.

예를 들어 다음 Service 코드에서:

```java
@Service
@RequiredArgsConstructor
public class AccountService {

    private final AccountMapper accountMapper;

}
```

Lombok은 개념적으로 다음 생성자를 생성한다.

```java
public AccountService(AccountMapper accountMapper) {
    this.accountMapper = accountMapper;
}
```

Spring은 이 생성자를 이용해 `AccountMapper` 의존성을 주입한다. 이러한 방식을 생성자 주입(Constructor Injection)이라고 한다.

이때 생성자를 자동 생성하는 주체는 Lombok이고, 생성자를 호출해 객체를 주입하는 주체는 Spring이라는 점을 구분한다.

`@RequiredArgsConstructor`는 다음 단계에서 `AccountService`를 작성할 때 사용한다.

## 5.10 실무에서 자주 보는 Lombok 어노테이션 정리

| 클래스 | 자주 사용하는 어노테이션 | 목적 |
|---|---|---|
| DTO | `@Getter`, `@Setter`, `@NoArgsConstructor` | 데이터 조회·설정 및 기본 생성 |
| Service | `@RequiredArgsConstructor` | `final` 의존성 생성자 주입 |
| Controller | `@RequiredArgsConstructor` | Service 의존성 생성자 주입 |
| Entity(JPA) | `@Getter`, `@NoArgsConstructor` 등 | 엔티티 필드 접근 및 생성 규칙 관리 |

프로젝트마다 코딩 표준과 프레임워크 구성은 다를 수 있으므로, Lombok 어노테이션을 무조건 많이 사용하는 것보다 클래스의 목적에 맞는 어노테이션을 선택하는 것이 중요하다.

## 5.11 AccountMapper 인터페이스 작성

파일 위치:

```text
src/main/java/com/example/accountmanagement/mapper/AccountMapper.java
```

```java
package com.example.accountmanagement.mapper;

import java.util.List;

import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

import com.example.accountmanagement.dto.AccountDto;

@Mapper
public interface AccountMapper {

    List<AccountDto> findAll();

    AccountDto findByAccountNo(@Param("accountNo") String accountNo);

    int insert(AccountDto account);

    int update(AccountDto account);
}
```

### 메서드 역할

| 메서드 | 기능 | 반환값 |
|---|---|---|
| `findAll()` | 전체 계좌 조회 | 계좌 목록 |
| `findByAccountNo()` | 계좌번호로 조회 | 계좌 1건 |
| `insert()` | 계좌 등록 | 처리 건수 |
| `update()` | 계좌 정보 수정 | 처리 건수 |

- `@Mapper`: MyBatis Mapper 인터페이스임을 표시한다.
- `@Param("accountNo")`: XML에서 `#{accountNo}`로 매개변수를 참조할 수 있게 한다.
- `List<AccountDto>`: 여러 계좌 데이터를 DTO 목록으로 반환한다.
- `AccountDto`: 단일 계좌 데이터를 DTO 객체로 반환한다.
- `int`: INSERT 또는 UPDATE로 처리된 행의 수를 반환한다.

## 5.12 이번 단계의 완료 체크

- `AccountDto.java`의 필드와 자료형 확인
- 기본 생성자와 전체 필드 생성자의 역할 이해
- 직접 작성한 Getter와 Setter의 역할 이해
- `pom.xml`에 Lombok 의존성 추가
- `@Getter`, `@Setter` 사용 방법 확인
- `@NoArgsConstructor`, `@AllArgsConstructor` 사용 방법 확인
- Lombok을 적용한 `AccountDto.java` 작성
- `mvn clean compile` 실행 및 `BUILD SUCCESS` 확인
- `AccountMapper.java` 작성
- `@Mapper`, `@Param`의 역할 이해

## 다음 단계

[6단계 - Mapper XML 및 Service 작성](../06-mapper-xml-service/)

6단계에서는 `AccountMapper.xml`에 실제 SQL을 작성하고, Service 계층에서 Mapper를 호출하는 방법을 학습한다. 이때 Service 클래스에서 `@RequiredArgsConstructor`를 사용해 Spring의 생성자 주입도 함께 확인한다.

## 참고 자료

- [Project Lombok - Maven 설정](https://projectlombok.org/setup/maven)
- [Project Lombok - VS Code 설정](https://projectlombok.org/setup/vscode)
- [Project Lombok - 어노테이션 소개](https://projectlombok.org/features/)
- [Spring Boot 4.0.9 - Managed Dependency Versions](https://docs.spring.io/spring-boot/4.0/appendix/dependency-versions/coordinates.html)
