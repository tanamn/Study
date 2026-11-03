---
layout: default
title: 6단계 - Mapper XML 및 Service 작성
permalink: /spring/spring-boot-practice/06-mapper-xml-service/
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

import java.math.BigDecimal;
import java.util.List;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import com.example.accountmanagement.dto.AccountDto;
import com.example.accountmanagement.mapper.AccountMapper;

import lombok.RequiredArgsConstructor;

@Service
@RequiredArgsConstructor
public class AccountService {

    private final AccountMapper accountMapper;

    // public AccountService(AccountMapper accountMapper) {
    //     this.accountMapper = accountMapper;
    // }

    // 1. 계좌 리스트 조회
    @Transactional(readOnly = true) 
    public List<AccountDto> findAll() {
        return accountMapper.findAll();
    }

    // 2. 게좌 번호로 계좌 조회
    public AccountDto findByAccountNo(String accountNo) {

        validateAccountNo(accountNo);
        AccountDto accountDto = accountMapper.findByAccountNo(accountNo);

        if (accountDto == null) {
            throw new IllegalArgumentException("존재하지 않는 계좌입니다");
        }

        return accountDto;
    }

    // 3. 계좌 생성
    @Transactional
    public int createAccount(AccountDto accountDto) {

        // 계좌 정보 유효성 검사
        validateAccount(accountDto);

        // 계좌 번호 null check
        if (accountDto.getAccountNo() == null || accountDto.getAccountNo().isEmpty()) {
            throw new IllegalArgumentException("계좌 번호는 필수 입력 항목입니다");
        }

        // 계좌 번호 중복 체크
        AccountDto existingAccount = accountMapper.findByAccountNo(accountDto.getAccountNo());
        if (existingAccount != null) {
            throw new IllegalArgumentException("이미 등록된 계좌입니다");
        }

        // 계좌 생성
        int result = accountMapper.insertAccount(accountDto);
        if (result != 1) {
            throw new RuntimeException("계좌 생성에 실패했습니다");
        }

        return result;
    }

    // 4. 계좌 정보 수정
    @Transactional
    public int updateAccount(String accountNo, AccountDto accountDto) {

        // 계좌번호 확인
        validateAccountNo(accountNo);

        // 계좌 정보 null check
        if (accountDto == null) {
            throw new IllegalArgumentException(
                "수정할 계좌 정보가 필요합니다."
            );
        }        

        // 수정 대상 계좌가 존재하는지 확인
        AccountDto existingAccount = accountMapper.findByAccountNo(accountNo);
        if (existingAccount == null) {
            throw new IllegalArgumentException("수정할 계좌가 존재하지 않습니다");
        }

        // 계좌 번호는 수정 불가, 기존 계좌 번호를 유지
        accountDto.setAccountNo(existingAccount.getAccountNo()); 

        // 계좌 정보 유효성 검사
        validateAccount(accountDto);

        // 계좌 정보 수정
        int result = accountMapper.updateAccount(accountDto);
        if (result != 1) {
            throw new RuntimeException("계좌 정보 수정에 실패했습니다");
        }

        return result;
    }

    // 계좌번호 공통 검증
    private void validateAccountNo(String accountNo) {

        if (accountNo == null || accountNo.isBlank()) {
            throw new IllegalArgumentException(
                "계좌번호를 입력해 주세요."
            );
        }
    }

    // 계좌 정보 공통 검증
    private void validateAccount(AccountDto accountDto) {

        if (accountDto == null) {
            throw new IllegalArgumentException(
                "계좌 정보가 필요합니다."
            );
        }

        validateAccountNo(accountDto.getAccountNo());

        if (accountDto.getAccountName() == null ||
            accountDto.getAccountName().isBlank()) {
            throw new IllegalArgumentException(
                "계좌명을 입력해 주세요."
            );
        }

        if (accountDto.getCustomerName() == null ||
            accountDto.getCustomerName().isBlank()) {
            throw new IllegalArgumentException(
                "예금주를 입력해 주세요."
            );
        }

        if (accountDto.getBalance() == null ||
            accountDto.getBalance().compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException(
                "계좌 잔액이 올바르지 않습니다."
            );
        }
    }    
}
```

## 4. Service의 역할

Service는 Controller와 Mapper 사이에서 업무 로직을 처리한다. 실제 업무에서는 계좌 존재 여부, 입력값, 업무 규칙 등을 검증하고 여러 Mapper 호출을 하나의 업무 처리로 구성할 수 있다.

## 완료 체크

- Mapper XML 생성
- namespace와 Mapper 인터페이스 경로 일치
- SQL id와 Java 메서드명 일치
- Service 작성

## 다음 단계

[7단계 - Controller 작성 및 Postman API 테스트](../07-controller-api-test/)
