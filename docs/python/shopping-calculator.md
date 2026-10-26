---
layout: default
title: 상품 구매 계산 프로그램
permalink: /python/shopping-calculator/
---

# 상품 구매 계산 프로그램

이 문서는 Python 기본 문법, 리스트와 딕셔너리, 함수, 입력값 검증을 종합한 실습 프로그램을 정리합니다.

## 1. 프로그램 목표

사용자가 구매할 상품 개수를 입력하고, 각 상품의 이름·가격·수량을 입력하면 다음 기능을 수행합니다.

- 입력값 검증: 상품 개수, 가격, 수량은 0보다 큰 정수만 허용합니다.
- 상품명 검증: 빈 문자열이나 공백만 입력한 경우 다시 입력받습니다.
- 상품 유형 분류: 가격에 따라 고가 상품, 중간 상품, 일반 상품으로 구분합니다.
- 총 구매 금액 계산: 상품별 가격과 수량을 곱해 누적합니다.
- 할인 계산: 총 구매 금액이 100,000원 이상이면 10% 할인합니다.
- 구매 내역 출력: 상품별 정보, 구매 금액, 할인 금액, 최종 금액을 표시합니다.

## 2. 프로그램 구조

| 함수 또는 변수 | 역할 |
|---|---|
| `get_positive_integer(prompt)` | 0보다 큰 정수를 입력받고 잘못된 입력을 처리합니다. |
| `get_non_blank_string(prompt)` | 공백이 아닌 문자열을 입력받습니다. |
| `get_product_type(price)` | 상품 가격에 따른 유형을 반환합니다. |
| `calculate_discount(total_amt)` | 할인 금액과 할인 안내 문구를 반환합니다. |
| `products` | 상품별 딕셔너리를 저장하는 리스트입니다. |
| `total_amt` | 전체 상품 금액을 누적합니다. |
| `discount`, `discount_msg` | 할인 금액과 안내 문구를 저장합니다. |
| `final_amt` | 할인 적용 후 최종 금액입니다. |

## 3. 전체 소스 코드

아래 코드는 현재까지 학습한 최종 구매 계산 프로그램입니다.

```python
# 변수 초기화
products = []
total_amt = 0
discount = 0
discount_rate = 0.1
discount_msg = ""
final_amt = 0


# 사용자로부터 입력을 받는 정수값에 대한 검증 함수
def get_positive_integer(prompt):
    while True:
        try:
            value = int(input(prompt))
            if value <= 0:
                print("0보다 큰 수를 입력해 주세요.")
                continue
            return value
        except ValueError:
            print("유효한 숫자를 입력해 주세요.")


# 상품명이 공백인지 확인하는 함수
def get_non_blank_string(prompt):
    while True:
        string = input(prompt)
        if not string.strip():
            print("공백이 아닌 값을 입력해 주세요.")
        else:
            break
    return string


# 상품 가격에 따른 상품 유형을 반환하는 함수
def get_product_type(price):
    if price >= 50000:
        return "고가 상품"
    elif price >= 30000:
        return "중간 상품"
    else:
        return "일반 상품"


# 총 구매 금액에 따른 할인 금액과 메시지를 반환하는 함수
def calculate_discount(total_amt):
    if total_amt >= 100000:
        discount = int(total_amt * 0.1)
        discount_msg = "구매 금액이 10만원 이상이므로 10% 할인 적용!"
    else:
        discount = 0
        discount_msg = "구매 금액이 10만원 미만이므로 할인 적용되지 않습니다."

    return discount, discount_msg


# 상품 개수 입력
range_cnt = get_positive_integer(
    "몇 개의 상품을 구매하시겠습니까? : "
)

# 상품 정보 입력
for i in range(1, range_cnt + 1):
    print(f"{i}번째 상품 구매 내역을 입력하세요.")

    name = get_non_blank_string("상품명을 입력하세요: ")
    price = get_positive_integer("상품 가격을 입력하세요: ")
    quantity = get_positive_integer("상품 수량을 입력하세요: ")

    # 상품 하나의 정보를 딕셔너리로 구성
    product = {
        "name": name,
        "price": price,
        "quantity": quantity,
        "type": get_product_type(price)
    }
    products.append(product)

    # 총 구매 금액 계산
    total_amt += price * quantity


# 총 구매 금액에 따른 할인 적용
discount, discount_msg = calculate_discount(total_amt)

# 최종 금액 계산
final_amt = total_amt - discount


# 구매 내역 출력
print("\n=========== 구매 내역 ===========")

for product in products:
    print(f"\n상품명: {product['name']}")
    print(f"상품 유형: {product['type']}")
    print(f"상품 가격: {product['price']:,}원")
    print(f"상품 수량: {product['quantity']:,}개")

print(f"\n구매 금액: {total_amt:,}원")
print(discount_msg)
print(f"할인 금액: {discount:,}원")
print("---------------------------------")
print(f"최종 금액: {final_amt:,}원")
print("=================================")
```

## 4. 실행 예시

상품 2개를 다음과 같이 입력한다고 가정합니다.

- 노트북: 가격 1,000,000원, 수량 1개
- 마우스: 가격 30,000원, 수량 2개

예상 계산 결과:

```text
구매 금액: 1,060,000원
구매 금액이 10만원 이상이므로 10% 할인 적용!
할인 금액: 106,000원
---------------------------------
최종 금액: 954,000원
```

## 5. 핵심 개념

### 리스트와 딕셔너리

상품 하나의 속성은 딕셔너리로 묶고, 여러 상품은 리스트에 저장합니다.

```python
product = {
    "name": name,
    "price": price,
    "quantity": quantity,
    "type": get_product_type(price)
}
products.append(product)
```

- `product`는 상품 하나의 정보를 저장하는 딕셔너리입니다.
- `products`는 여러 상품 딕셔너리를 저장하는 리스트입니다.
- `product["price"]`처럼 키로 상품 정보를 조회합니다.

### 함수와 반환값

`get_product_type()`은 상품 유형을 반환하고, `calculate_discount()`는 할인 금액과 안내 문구 두 값을 반환합니다.

```python
discount, discount_msg = calculate_discount(total_amt)
```

### 예외 처리

`get_positive_integer()`의 `try/except ValueError`는 `abc`처럼 정수로 변환할 수 없는 입력을 처리합니다. `while True`와 `continue`를 사용해 올바른 값이 입력될 때까지 다시 입력받습니다.

## 6. 개선할 수 있는 점

현재 프로그램은 학습용으로 동작하지만, 다음과 같은 개선을 생각해 볼 수 있습니다.

1. `discount_rate = 0.1`을 실제 할인 계산에 사용해 할인율을 한 곳에서 관리합니다.
2. `get_non_blank_string()`에서 `return string.strip()`을 사용하면 상품명 앞뒤 공백을 제거할 수 있습니다.
3. 할인 계산 함수에서 할인 금액과 메시지를 반환하는 현재 방식을 유지하되, 출력 메시지 생성과 계산 로직을 분리할 수 있습니다.
4. 상품 정보 입력·할인 계산·출력 기능을 더 작은 함수로 나누어 테스트하기 쉽게 만들 수 있습니다.
5. 상품 목록을 파일에 저장하고 다시 불러오는 기능을 추가할 수 있습니다.

## 7. 실습 과제

- 할인율을 `discount_rate` 변수로 관리하도록 수정합니다.
- 상품명 앞뒤의 공백을 제거해 저장합니다.
- 구매 내역에 상품별 소계(`가격 × 수량`)를 출력합니다.
- 상품 목록을 JSON 파일로 저장하고 다시 읽어오는 기능을 추가합니다.
