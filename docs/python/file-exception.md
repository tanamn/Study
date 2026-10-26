---
layout: default
title: 파일 입출력과 예외 처리
permalink: /python/file-exception/
---

{% include navigation.html %}

# 파일 입출력과 예외 처리

이 문서는 Python에서 파일을 읽고 쓰는 방법과, 프로그램 실행 중 발생할 수 있는 오류를 안전하게 처리하는 방법을 학습합니다.

## 1. 학습 목표

이 단원을 마치면 다음을 할 수 있습니다.

- 텍스트 파일을 읽고 쓸 수 있습니다.
- `with open()`을 사용해 파일을 안전하게 관리할 수 있습니다.
- 파일 경로와 문자 인코딩의 의미를 이해할 수 있습니다.
- `try`, `except`, `else`, `finally`로 예외를 처리할 수 있습니다.
- 파일이 없거나 입력값이 잘못된 경우에도 프로그램이 갑자기 종료되지 않도록 만들 수 있습니다.

## 2. 파일 열기 기본 구조

Python에서는 `open()` 함수로 파일을 엽니다.

```python
file = open("memo.txt", "r", encoding="utf-8")
```

| 항목 | 의미 |
|---|---|
| `"memo.txt"` | 열 파일의 경로 |
| `"r"` | 읽기 모드 |
| `encoding="utf-8"` | 문자 인코딩을 UTF-8로 지정 |

### 자주 사용하는 파일 모드

| 모드 | 의미 | 파일이 없을 때 |
|---|---|---|
| `"r"` | 읽기 | 오류 발생 |
| `"w"` | 새로 쓰기 | 새 파일 생성 |
| `"a"` | 기존 내용 뒤에 추가 | 새 파일 생성 |
| `"x"` | 새 파일 만들기 | 이미 있으면 오류 발생 |

**주의:** `"w"` 모드는 기존 파일이 있으면 내용을 덮어씁니다. 기존 내용을 유지하면서 뒤에 추가하려면 `"a"`를 사용합니다.

## 3. 파일 쓰기

### 3.1 기본 예제

```python
with open("memo.txt", "w", encoding="utf-8") as file:
    file.write("Python 파일 입출력을 학습합니다.\n")
    file.write("두 번째 줄입니다.\n")
```

실행하면 현재 작업 폴더에 `memo.txt` 파일이 만들어집니다.

파일 내용:

```text
Python 파일 입출력을 학습합니다.
두 번째 줄입니다.
```

- `write()`는 문자열을 파일에 씁니다.
- 줄바꿈을 원하면 문자열 끝에 `\n`을 넣습니다.
- `with` 블록이 끝나면 파일이 자동으로 닫힙니다.

### 3.2 여러 줄 한 번에 쓰기

```python
lines = [
    "첫 번째 줄\n",
    "두 번째 줄\n",
    "세 번째 줄\n"
]

with open("memo.txt", "w", encoding="utf-8") as file:
    file.writelines(lines)
```

`writelines()`는 각 문자열 사이에 자동으로 줄바꿈을 추가하지 않습니다. 줄바꿈이 필요하면 각 문자열에 `\n`을 포함해야 합니다.

## 4. 파일 읽기

### 4.1 전체 내용 읽기: `read()`

```python
with open("memo.txt", "r", encoding="utf-8") as file:
    content = file.read()

print(content)
```

`read()`는 파일의 전체 내용을 하나의 문자열로 읽습니다.

### 4.2 한 줄씩 읽기: `readline()`

```python
with open("memo.txt", "r", encoding="utf-8") as file:
    first_line = file.readline()
    second_line = file.readline()

print(first_line, end="")
print(second_line, end="")
```

`readline()`은 한 번 호출할 때 한 줄을 읽습니다.

### 4.3 모든 줄을 리스트로 읽기: `readlines()`

```python
with open("memo.txt", "r", encoding="utf-8") as file:
    lines = file.readlines()

print(lines)
```

각 줄이 리스트의 원소가 됩니다. 파일이 매우 크다면 모든 내용을 한꺼번에 메모리에 올리는 대신 아래처럼 반복문으로 읽는 방법이 더 적절할 수 있습니다.

```python
with open("memo.txt", "r", encoding="utf-8") as file:
    for line in file:
        print(line, end="")
```

## 5. `with open()`을 사용하는 이유

파일을 열면 사용이 끝난 뒤 닫아야 합니다. 파일을 닫지 않으면 자원이 불필요하게 유지되거나, 파일에 기록한 내용이 예상대로 반영되지 않을 수 있습니다.

`with`를 사용하면 블록을 벗어날 때 파일이 자동으로 닫힙니다.

```python
with open("memo.txt", "r", encoding="utf-8") as file:
    content = file.read()

# 이 시점에는 파일이 자동으로 닫혀 있습니다.
print(content)
```

가능하면 다음처럼 `with open()`을 사용하는 습관을 들이세요.

## 6. 예외 처리 기본 구조

파일이 없거나 숫자 변환에 실패하면 예외가 발생할 수 있습니다. `try`와 `except`로 처리할 수 있습니다.

```python
try:
    number = int(input("숫자를 입력하세요: "))
    print(f"입력한 숫자: {number}")
except ValueError:
    print("정수로 변환할 수 있는 숫자를 입력해 주세요.")
```

- `try`: 예외가 발생할 수 있는 코드를 실행합니다.
- `except`: 지정한 예외가 발생했을 때 실행합니다.
- `else`: `try`에서 예외가 발생하지 않았을 때 실행합니다.
- `finally`: 예외 발생 여부와 관계없이 실행합니다.

### 6.1 `else`와 `finally` 사용

```python
try:
    number = int(input("숫자를 입력하세요: "))
except ValueError:
    print("올바른 정수가 아닙니다.")
else:
    print(f"입력 성공: {number}")
finally:
    print("입력 처리를 마쳤습니다.")
```

`finally`는 정리 작업 등에 사용할 수 있습니다. `with open()`을 사용하면 파일 닫기 작업은 자동으로 처리되므로 파일을 닫기 위해 `finally`를 직접 작성할 필요가 없습니다.

## 7. 파일 입출력에서 자주 만나는 예외

| 예외 | 발생할 수 있는 상황 | 처리 방법 |
|---|---|---|
| `FileNotFoundError` | 읽으려는 파일이 없음 | 파일 경로 확인 또는 안내 메시지 출력 |
| `PermissionError` | 파일 접근 권한이 없음 | 권한과 파일 사용 상태 확인 |
| `UnicodeDecodeError` | 지정한 인코딩으로 읽기 어려움 | 파일의 실제 인코딩 확인 |
| `ValueError` | 문자열을 정수로 변환할 수 없음 | 입력값 검증 |
| `OSError` | 파일 시스템 작업 중 오류 | 경로, 장치, 파일 상태 확인 |

가능하면 모든 예외를 무조건 숨기기보다 예상되는 예외를 구체적으로 처리합니다.

## 8. 파일이 없을 때 처리하기

```python
try:
    with open("memo.txt", "r", encoding="utf-8") as file:
        content = file.read()
except FileNotFoundError:
    print("memo.txt 파일을 찾을 수 없습니다.")
else:
    print("파일 내용:")
    print(content)
```

읽기 모드 `"r"`로 존재하지 않는 파일을 열면 `FileNotFoundError`가 발생합니다. 이 예제에서는 오류 메시지를 보여 주고 프로그램이 비정상적으로 종료되는 것을 방지합니다.

## 9. 실습: 메모 저장 및 조회 프로그램

사용자가 입력한 메모를 파일에 저장하고, 저장된 메모를 다시 읽어 출력하는 예제입니다.

```python
FILE_NAME = "memo.txt"


def save_memo():
    memo = input("저장할 메모를 입력하세요: ").strip()

    if not memo:
        print("빈 메모는 저장할 수 없습니다.")
        return

    try:
        with open(FILE_NAME, "a", encoding="utf-8") as file:
            file.write(memo + "\n")
    except OSError as error:
        print(f"메모 저장 중 오류가 발생했습니다: {error}")
    else:
        print("메모를 저장했습니다.")


def read_memos():
    try:
        with open(FILE_NAME, "r", encoding="utf-8") as file:
            content = file.read()
    except FileNotFoundError:
        print("저장된 메모가 없습니다. 먼저 메모를 저장해 주세요.")
    except OSError as error:
        print(f"메모 읽기 중 오류가 발생했습니다: {error}")
    else:
        print("\n===== 저장된 메모 =====")
        print(content, end="")
        print("=======================")


while True:
    print("\n1. 메모 저장")
    print("2. 메모 조회")
    print("0. 종료")

    menu = input("메뉴를 선택하세요: ").strip()

    if menu == "1":
        save_memo()
    elif menu == "2":
        read_memos()
    elif menu == "0":
        print("프로그램을 종료합니다.")
        break
    else:
        print("1, 2, 0 중에서 선택해 주세요.")
```

### 실행 방법

1. VS Code에서 `file_exception.py` 파일을 만듭니다.
2. 위 코드를 붙여 넣고 저장합니다.
3. 터미널에서 다음 명령을 실행합니다.

```bash
python file_exception.py
```

4. `1`을 선택해 메모를 저장합니다.
5. `2`를 선택해 저장된 메모를 확인합니다.
6. 같은 폴더에 `memo.txt` 파일이 생성되었는지 확인합니다.

## 10. 이전 상품 구매 계산 프로그램과 연결하기

앞서 작성한 상품 구매 계산 프로그램은 실행 중에 상품 정보를 입력받고 결과를 출력합니다. 다음 단계에서는 구매 내역을 파일에 저장해 프로그램을 종료한 뒤에도 확인할 수 있도록 확장할 수 있습니다.

예를 들어 구매 내역을 텍스트 파일로 저장할 수 있습니다.

```python
with open("purchase_history.txt", "a", encoding="utf-8") as file:
    file.write(f"구매 금액: {total_amt:,}원\n")
    file.write(f"할인 금액: {discount:,}원\n")
    file.write(f"최종 금액: {final_amt:,}원\n")
    file.write("--------------------\n")
```

이 코드는 `total_amt`, `discount`, `final_amt`가 계산된 뒤에 실행해야 합니다. `"a"` 모드이므로 실행할 때마다 기존 구매 내역 뒤에 기록됩니다.

## 11. 핵심 정리

- `open()`으로 파일을 열고, 모드에 따라 읽거나 씁니다.
- `encoding="utf-8"`을 지정하면 한글 텍스트 파일을 다룰 때 인코딩을 명시할 수 있습니다.
- `with open()`을 사용하면 파일이 자동으로 닫힙니다.
- `try/except`로 예상되는 오류를 처리합니다.
- `FileNotFoundError`는 파일이 없을 때, `ValueError`는 값 변환에 실패할 때 대표적으로 발생합니다.
- `"w"`는 덮어쓰기, `"a"`는 이어 쓰기입니다.

## 12. 실습 과제

1. 메모 저장 프로그램에 메모 작성 날짜와 시간을 함께 저장하도록 수정합니다.
2. 저장된 메모의 줄 수를 계산해 출력합니다.
3. 파일이 없을 때 빈 파일을 자동 생성하는 기능과, 파일이 없다는 안내만 표시하는 기능의 차이를 확인합니다.
4. 상품 구매 계산 프로그램의 구매 내역을 `purchase_history.txt`에 저장합니다.
5. 숫자 입력에서 발생하는 `ValueError`와 파일 읽기에서 발생하는 `FileNotFoundError`를 각각 직접 재현하고 처리 결과를 확인합니다.

## 13. 메모 작성 날짜와 시간을 함께 저장

메모를 저장할 때 작성 날짜와 시간을 함께 기록하면 언제 작성한 메모인지 확인할 수 있어. Python의 `datetime` 모듈을 사용하면 현재 날짜와 시간을 가져올 수 있어.

### 13.1 현재 날짜와 시간 가져오기

```python
from datetime import datetime

now = datetime.now()

print(now)
```

실행 결과 예시:

```text
2026-10-26 16:25:30.123456
```

`datetime.now()`는 현재 날짜와 시간을 가져와. 출력 형식을 보기 좋게 바꾸려면 `strftime()`을 사용하면 돼.

```python
from datetime import datetime

now = datetime.now()
formatted_time = now.strftime("%Y-%m-%d %H:%M:%S")

print(formatted_time)
```

실행 결과 예시:

```text
2026-10-26 16:25:30
```

### 13.2 날짜와 시간 형식 알아보기

날짜와 시간 형식은 다음과 같아.

| 코드 | 의미 | 예시 |
|---|---|---|
| `%Y` | 연도 4자리 | `2026` |
| `%m` | 월 | `10` |
| `%d` | 일 | `26` |
| `%H` | 24시간제 시 | `16` |
| `%M` | 분 | `25` |
| `%S` | 초 | `30` |

### 13.3 메모 저장 프로그램에 적용하기

기존 `save_memo()` 함수에서 현재 날짜와 시간을 가져온 다음, 메모 내용과 함께 파일에 저장하면 돼.

```python
from datetime import datetime

now = datetime.now()
formatted_time = now.strftime("%Y-%m-%d %H:%M:%S")

with open("memo.txt", "a", encoding="utf-8") as file:
    file.write(f"[{formatted_time}] 오늘 Python을 공부했다.\n")
```

저장 결과 예시:

```text
[2026-10-26 16:25:30] 오늘 Python을 공부했다.
```

**핵심 정리**

- `datetime.now()` : 현재 날짜와 시간을 가져와.
- `strftime()` : 날짜와 시간을 원하는 문자열 형식으로 변환해.
- `\n` : 파일에 내용을 쓸 때 줄을 바꿔 줘.
- `"a"` : 기존 파일의 내용을 유지하면서 새로운 메모를 뒤에 추가해.



## 14. 다음 학습 주제

다음 단계에서는 코드를 여러 파일로 나누고 재사용하는 **모듈과 패키지**를 학습합니다.
