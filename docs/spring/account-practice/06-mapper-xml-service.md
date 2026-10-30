---
layout: default
title: 6단계 - Mapper XML 및 Service 작성
permalink: /spring/account-practice/06-mapper-xml-service/
---

{% include navigation.html %}

# 6단계. Mapper XML에 SQL 작성 및 Service 구현

## 1. AccountMapper.xml 생성

경로: `src/main/resources/mapper/AccountMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.example.accountmanagement.mapper.AccountMapper">

    <select id="findAll"
            resultType="com.example.accountmanagement.dto.AccountDto">
        SELECT ACCOUNT_NO, ACCOUNT_NAME, CUSTOMER_NAME, BALANCE
        FROM ACCOUNT
        ORDER BY ACCOUNT_NO
    </select>

    <select id="findByAccountNo" resultType="com.example.accountmanagement.dto.AccountDto">
        SELECT ACCOUNT_NO, ACCOUNT_NAME, CUSTOMER_NAME, BALANCE
        FROM ACCOUNT
        WHERE ACCOUNT_NO = #{accountNo}
    </select>

    <insert id="insert" parameterType="com.example.accountmanagement.dto.AccountDto">
        INSERT INTO ACCOUNT (ACCOUNT_NO, ACCOUNT_NAME, CUSTOMER_NAME, BALANCE)
        VALUES (#{accountNo}, #{accountName}, #{customerName}, #{balance})
    </insert>

    <update id="update" parameterType="com.example.accountmanagement.dto.AccountDto">
        UPDATE ACCOUNT
        SET ACCOUNT_NAME = #{accountName},
            CUSTOMER_NAME = #{customerName}
        WHERE ACCOUNT_NO = #{accountNo}
    </update>

</mapper>
```

## 2. 인터페이스와 XML 연결 규칙

| XML 항목 | 일치할 내용 |
|---|---|
| `namespace` | Mapper 인터페이스의 전체 경로 |
| SQL `id` | Mapper 인터페이스 메서드명 |
| `#{...}` | DTO 필드명 또는 `@Param` 이름 |

예를 들어 `id="findAll"`은 Java의 `findAll()` 메서드와 연결된다.

`#{accountNo}`는 안전한 파라미터 바인딩 방식이다. 사용자 입력을 `${...}`로 직접 SQL에 삽입하면 SQL Injection 위험이 있으므로 허용 목록 검증 없이 사용하지 않는다.

`map-underscore-to-camel-case: true` 설정으로 `ACCOUNT_NO`와 `accountNo`처럼 컬럼명과 필드명의 표기 차이를 자동 매핑할 수 있다.

## 3. AccountService 작성

경로: `src/main/java/com/example/accountmanagement/service/AccountService.java`

```java
package com.example.accountmanagement.service;

import java.util.List;
import org.springframework.stereotype.Service;
import com.example.accountmanagement.dto.AccountDto;
import com.example.accountmanagement.mapper.AccountMapper;

@Service
public class AccountService {

    private final AccountMapper accountMapper;

    public AccountService(AccountMapper accountMapper) {
        this.accountMapper = accountMapper;
    }

    public List<AccountDto> findAll() {
        return accountMapper.findAll();
    }

    public AccountDto findByAccountNo(String accountNo) {
        return accountMapper.findByAccountNo(accountNo);
    }

    public int create(AccountDto account) {
        return accountMapper.insert(account);
    }

    public int update(AccountDto account) {
        return accountMapper.update(account);
    }
}
```

## 4. Service의 역할

Service는 Controller와 Mapper 사이에서 업무 로직을 처리한다. 실제 업무에서는 계좌 존재 여부, 입력값, 업무 규칙 등을 검증하고 여러 Mapper 호출을 하나의 업무 처리로 구성할 수 있다.

## 완료 체크

- [ ] Mapper XML 생성
- [ ] namespace와 Mapper 인터페이스 경로 일치
- [ ] SQL id와 Java 메서드명 일치
- [ ] Service 작성

## 다음 단계

[7단계 - Controller 작성 및 Postman API 테스트](../07-controller-api-test/)
