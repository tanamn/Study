---
layout: default
title: 5단계 - DTO 및 MyBatis Mapper 인터페이스
permalink: /spring/account-practice/05-dto-mapper/
---

{% include navigation.html %}

# 5단계. DTO 및 MyBatis Mapper 인터페이스 작성

아래 예제는 기본 패키지가 `com.example.accountmanagement`인 경우다. 프로젝트의 실제 패키지명에 맞춰 수정한다.

## 1. AccountDto 작성

`src/main/java/com/example/accountmanagement/dto/AccountDto.java`

```java
package com.example.accountmanagement.dto;

import java.math.BigDecimal;

public class AccountDto {
    private String accountNo;
    private String accountName;
    private String customerName;
    private BigDecimal balance;

    public AccountDto() {}

    public AccountDto(String accountNo, String accountName,
                      String customerName, BigDecimal balance) {
        this.accountNo = accountNo;
        this.accountName = accountName;
        this.customerName = customerName;
        this.balance = balance;
    }

    public String getAccountNo() { return accountNo; }
    public void setAccountNo(String accountNo) { this.accountNo = accountNo; }

    public String getAccountName() { return accountName; }
    public void setAccountName(String accountName) { this.accountName = accountName; }

    public String getCustomerName() { return customerName; }
    public void setCustomerName(String customerName) { this.customerName = customerName; }

    public BigDecimal getBalance() { return balance; }
    public void setBalance(BigDecimal balance) { this.balance = balance; }
}
```

금액은 부동소수점 오차를 피하기 위해 `double` 대신 `BigDecimal`을 사용한다.

## 2. AccountMapper 인터페이스 작성

`src/main/java/com/example/accountmanagement/mapper/AccountMapper.java`

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

## 3. 메서드 역할

| 메서드 | 기능 | 반환값 |
|---|---|---|
| `findAll()` | 전체 계좌 조회 | 계좌 목록 |
| `findByAccountNo()` | 계좌번호 조회 | 계좌 1건 |
| `insert()` | 계좌 등록 | 처리 건수 |
| `update()` | 계좌 정보 수정 | 처리 건수 |

`@Mapper`는 MyBatis Mapper임을 표시한다. `@Param("accountNo")`는 XML에서 `#{accountNo}`로 매개변수를 참조할 수 있게 한다.

## 완료 체크

- [ ] dto 패키지 및 AccountDto 작성
- [ ] mapper 패키지 및 AccountMapper 작성
- [ ] 실제 프로젝트 패키지명과 package 선언 일치 확인

## 다음 단계

[6단계 - Mapper XML에 SQL 작성 및 Service 작성](../06-mapper-xml-service/)
