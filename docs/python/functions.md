---
layout: default
title: 함수와 입력값 검증
permalink: /python/functions/
---

# 함수와 입력값 검증

함수는 특정 작업을 수행하는 코드 블록입니다. 반복되는 기능을 함수로 분리하면 재사용성과 가독성을 높일 수 있습니다.

## 1. 함수 정의와 호출

```python
def hello():
    print("안녕하세요!")

hello()
```

함수는 정의만 해서는 실행되지 않습니다. `hello()`처럼 호출해야 실행됩니다.

## 2. 매개변수와 인자

```python
def print_product(name, price):
    print(f"상품명: {name}")
    print(f"상품 가격: {price:,}원")

print_product("노트북", 1500000)
```

- `name`, `price`: 매개변수(parameter)
- `"노트북"`, `1500000`: 인자(argument)

## 3. `return`으로 결과 반환하기

```python
def calculate_total(price, quantity):
    total = price * quantity
    return total

result = calculate_total(30000, 2)
print(result)  # 60000
```

`return`은 함수의 결과를 호출한 위치로 돌려줍니다. `print()`가 화면에 표시하는 것과는 다릅니다.

## 4. 양의 정수 입력값 검증

```python
def get_positive_integer(prompt):
    while True:
        try:
            value = int(input(prompt))

            if value <= 0:
                print("0보다 큰 수를 입력해 주세요.")
                continue

            return value

        except ValueError:
            print("유효한 정수를 입력해 주세요.")
```

사용 예:

```python
range_cnt = get_positive_integer(
    "몇 개의 상품을 구매하시겠습니까? : "
)
```

문자가 입력되어 정수 변환에 실패하면 `ValueError`를 처리하고, 0 이하의 값이면 다시 입력받습니다.

## 5. 공백이 아닌 문자열 입력받기

```python
def get_non_blank_string(prompt):
    while True:
        value = input(prompt)

        if value.strip():
            return value

        print("공백이 아닌 값을 입력해 주세요.")
```

`strip()`은 문자열 앞뒤의 공백을 제거한 문자열을 반환합니다. `if value.strip():`는 공백만 입력했는지 확인합니다. 앞뒤 공백을 제거해 저장하려면 `return value.strip()`으로 작성할 수 있습니다.

## 6. 상품 유형 분류 함수

```python
def get_product_type(price):
    if price >= 50000:
        return "고가 상품"
    elif price >= 30000:
        return "중간 상품"
    else:
        return "일반 상품"
```

## 7. 여러 값 반환하기

```python
def calculate_discount(total_amt):
    if total_amt >= 100000:
        discount = int(total_amt * 0.1)
        message = "10% 할인 적용!"
    else:
        discount = 0
        message = "할인 적용되지 않습니다."

    return discount, message

discount, discount_msg = calculate_discount(150000)
```

`return discount, message`는 두 값을 튜플로 반환하며, 호출한 쪽에서 각각의 변수로 받을 수 있습니다.

## 8. 변수의 유효 범위

함수 안에서 만든 변수는 일반적으로 지역 변수입니다.

```python
def calculate_total(price, quantity):
    total = price * quantity
    return total

result = calculate_total(10000, 3)
```

함수 밖의 변수를 직접 수정하기보다는 매개변수로 전달하고 반환값을 받는 방식을 권장합니다.

## 9. 실습

다음 기능을 함수로 분리해 봅니다.

- 양의 정수를 입력받는 `get_positive_integer()`
- 공백이 아닌 상품명을 입력받는 `get_non_blank_string()`
- 가격에 따른 유형을 반환하는 `get_product_type()`
- 총액에 따른 할인을 계산하는 `calculate_discount()`

상품 입력, 딕셔너리 생성, 리스트 저장, 총액 누적은 메인 프로그램에서 처리할 수 있습니다.
