---
layout: default
title: Python 기본 구조
permalink: /python/basics/
---

{% include navigation.html %}

# Python 기본 구조

## 1. Python이란?

Python은 문법이 비교적 간결하고 자동화, 데이터 처리, 웹 개발, AI 등 다양한 분야에서 사용하는 프로그래밍 언어입니다.

Python은 변수를 사용하기 전에 자료형을 별도로 선언하지 않습니다. 값을 대입하면 변수가 만들어지고, 값에 따라 자료형이 결정됩니다.

## 2. 출력: `print()`

```python
print("안녕하세요!")
print(100)
print(10 + 20)
```

실행 결과:

```text
안녕하세요!
100
30
```

## 3. 변수와 자료형

```python
product_name = "노트북"  # str: 문자열
product_price = 1500000  # int: 정수
tax_rate = 0.1           # float: 실수
is_available = True      # bool: 참 또는 거짓

print(product_name)
print(product_price)
print(type(product_name))
print(type(product_price))
```

| 자료형 | 의미 | 예시 |
|---|---|---|
| `str` | 문자열 | `"노트북"` |
| `int` | 정수 | `10000` |
| `float` | 실수 | `0.1` |
| `bool` | 참 또는 거짓 | `True`, `False` |
| `list` | 여러 값을 순서대로 저장 | `[1000, 2000]` |
| `dict` | 키와 값의 쌍으로 저장 | `{"price": 1000}` |

## 4. 사용자 입력: `input()`

`input()`은 사용자 입력을 문자열로 반환합니다.

```python
name = input("이름을 입력하세요: ")
print(f"입력한 이름: {name}")
```

숫자 계산을 하려면 정수로 변환해야 합니다.

```python
price = int(input("상품 가격을 입력하세요: "))
qty = int(input("상품 수량을 입력하세요: "))

total = price * qty
print(f"구매 금액: {total:,}원")
```

## 5. 문자열과 f-string

```python
product_name = "마우스"
price = 30000
qty = 2

print(f"상품명: {product_name}")
print(f"상품 가격: {price:,}원")
print(f"구매 금액: {price * qty:,}원")
```

f-string은 문자열 안에 변수나 계산 결과를 넣을 때 사용합니다. `:,`는 천 단위 구분 쉼표를 표시합니다.

## 6. 주석과 변수 이름

```python
# 상품 가격과 수량을 곱해 총액을 계산합니다.
total = price * qty
```

변수 이름은 의미가 분명하게 작성하고, 여러 단어는 `product_price`처럼 밑줄로 연결하는 방식이 흔합니다.

## 7. 실습

1. 상품명, 가격, 수량을 입력받습니다.
2. 가격과 수량을 곱해 총액을 계산합니다.
3. f-string으로 상품명과 총액을 출력합니다.
4. 총액에 천 단위 쉼표가 표시되는지 확인합니다.
