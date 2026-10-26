---
layout: default
title: 리스트와 딕셔너리
permalink: /python/list-dictionary/
---

{% include navigation.html %}

# 리스트와 딕셔너리

리스트와 딕셔너리는 여러 데이터를 저장하고 관리하는 대표적인 자료구조입니다.

## 1. 리스트(List)

```python
prices = [1000, 2000, 3000]

print(prices[0])   # 1000
print(prices[1])   # 2000
print(prices[-1])  # 3000
```

인덱스는 0부터 시작하며 `-1`은 마지막 항목을 의미합니다.

### 리스트에 값 추가하기: `append()`

```python
prices = []
prices.append(1000)
prices.append(2000)

print(prices)  # [1000, 2000]
```

### 리스트 순회하기

```python
for price in prices:
    print(price)
```

## 2. 딕셔너리(Dictionary)

딕셔너리는 키(`key`)와 값(`value`)을 쌍으로 저장합니다.

```python
product = {
    "name": "노트북",
    "price": 1500000,
    "quantity": 1,
    "type": "고가 상품"
}

print(product["name"])
print(product["price"])
```

### 값 변경과 추가

```python
product["price"] = 1450000  # 기존 값 변경
product["brand"] = "Example" # 새로운 키와 값 추가
```

키가 존재하지 않을 때 `product["key"]`로 접근하면 `KeyError`가 발생할 수 있습니다.

## 3. 리스트와 딕셔너리 조합하기

```python
products = []

product1 = {
    "name": "노트북",
    "price": 1500000,
    "quantity": 1,
    "type": "고가 상품"
}

product2 = {
    "name": "마우스",
    "price": 30000,
    "quantity": 2,
    "type": "중간 상품"
}

products.append(product1)
products.append(product2)
```

- `products`는 리스트입니다.
- `products[0]`은 첫 번째 상품 딕셔너리입니다.
- `products[0]["name"]`은 첫 번째 상품의 이름입니다.

## 4. 딕셔너리 리스트 순회하기

```python
for product in products:
    print(f"상품명: {product['name']}")
    print(f"가격: {product['price']:,}원")
    print(f"수량: {product['quantity']}개")
```

## 5. 전체 구매 금액 계산하기

```python
total_amt = 0

for product in products:
    item_amt = product["price"] * product["quantity"]
    total_amt += item_amt

print(f"총 구매 금액: {total_amt:,}원")
```

## 6. 실습

1. 상품 2개의 딕셔너리를 만듭니다.
2. `append()`로 `products` 리스트에 추가합니다.
3. `for` 반복문으로 상품명, 가격, 수량을 출력합니다.
4. 상품별 금액을 계산해 전체 구매 금액을 출력합니다.
