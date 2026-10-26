# 조건문과 반복문

조건문은 조건에 따라 실행할 코드를 선택하고, 반복문은 코드를 여러 번 실행합니다.

## 1. 조건문: `if`, `elif`, `else`

```python
price = 40000

if price >= 50000:
    product_type = "고가 상품"
elif price >= 30000:
    product_type = "중간 상품"
else:
    product_type = "일반 상품"

print(product_type)
```

조건은 위에서부터 순서대로 확인하며, 처음 참이 되는 조건의 블록이 실행됩니다.

## 2. `for` 반복문

```python
for i in range(3):
    print(i)
```

실행 결과는 `0`, `1`, `2`입니다. `range(3)`은 끝 숫자 3을 포함하지 않습니다.

```python
for i in range(1, 4):
    print(f"{i}번째 상품")
```

## 3. 리스트 순회하기

```python
prices = [1000, 2000, 3000]

for price in prices:
    print(f"{price:,}원")
```

## 4. `while` 반복문

```python
count = 1

while count <= 3:
    print(count)
    count += 1
```

반복 조건을 바꾸지 않으면 무한 반복에 빠질 수 있으므로 주의합니다.

## 5. `break`와 `continue`

- `break`: 반복문을 종료합니다.
- `continue`: 현재 반복의 남은 부분을 건너뛰고 다음 반복으로 이동합니다.

```python
while True:
    value = int(input("양의 정수를 입력하세요: "))

    if value > 0:
        break

    print("0보다 큰 값을 입력해 주세요.")

print(f"입력값: {value}")
```

이 예제는 양의 정수가 입력될 때까지 반복합니다. 문자 입력의 예외 처리는 `try/except`로 추가할 수 있습니다.

## 6. 누적 계산

```python
prices = [1000, 2000, 3000]
total = 0

for price in prices:
    total += price

print(f"총액: {total:,}원")
```

## 7. 실습

1. 빈 리스트 `prices = []`를 만듭니다.
2. `for` 반복문으로 가격 3개를 입력받습니다.
3. 입력한 가격을 리스트에 추가합니다.
4. 다른 `for` 반복문으로 총액을 계산합니다.
5. 총액을 천 단위 쉼표와 함께 출력합니다.
