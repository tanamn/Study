---
layout: default
title: 모듈과 패키지
permalink: /python/modules-packages/
---

{% include navigation.html %}

# 모듈과 패키지

Python 프로그램이 커지면 모든 코드를 하나의 파일에 작성하기보다 기능별로 여러 파일로 나누어 관리하는 것이 좋아. 이번 단원에서는 코드를 재사용하기 위한 **모듈(Module)** 과 **패키지(Package)** 를 학습해.

## 1. 학습 목표

이 단원을 마치면 다음을 할 수 있어.

- 모듈과 패키지의 차이를 설명할 수 있어.
- `import`를 사용해 다른 파일의 기능을 가져올 수 있어.
- `from ... import ...` 문법을 사용할 수 있어.
- `if __name__ == "__main__":`의 기본적인 역할을 이해할 수 있어.
- 간단한 프로그램을 여러 파일로 분리할 수 있어.

## 2. 모듈이란?

**모듈은 Python 코드가 들어 있는 하나의 `.py` 파일**이야. 함수, 변수, 클래스 등을 다른 Python 파일에서 가져와 사용할 수 있어.

예를 들어 `calculator.py` 파일을 만들자.

```python
def add(a, b):
    return a + b


def subtract(a, b):
    return a - b
```

이 파일에는 덧셈과 뺄셈 기능이 정의되어 있어. 다른 파일에서 이 기능을 가져와 사용할 수 있어.

## 3. 모듈 가져오기: `import`

같은 폴더에 `calculator.py`와 `main.py` 파일을 만들자.

`main.py`

```python
import calculator

result = calculator.add(10, 5)
print(result)

result = calculator.subtract(10, 5)
print(result)
```

실행 결과:

```text
15
5
```

- `import calculator`는 `calculator` 모듈을 가져와.
- `calculator.add(10, 5)`처럼 모듈 이름 뒤에 점(`.`)을 붙여 함수에 접근해.
- `return`은 함수가 계산한 값을 호출한 곳으로 돌려줘.

**중요:** 이 예제에서는 `calculator.py`와 `main.py`가 같은 폴더에 있어야 해.

## 4. 필요한 기능만 가져오기: `from ... import ...`

모듈 전체 이름을 매번 쓰지 않고 특정 함수만 가져올 수도 있어.

```python
from calculator import add

result = add(10, 5)
print(result)
```

여러 함수를 가져오려면 쉼표로 구분해.

```python
from calculator import add, subtract

print(add(10, 5))
print(subtract(10, 5))
```

| 문법 | 사용 예 | 호출 방법 |
|---|---|---|
| `import 모듈` | `import calculator` | `calculator.add(1, 2)` |
| `from 모듈 import 함수` | `from calculator import add` | `add(1, 2)` |

학습 초반에는 출처를 알아보기 쉬운 `import calculator` 방식부터 익히는 것을 추천해.

## 5. 모듈에 별칭 지정하기: `as`

```python
import calculator as calc

print(calc.add(10, 5))
```

`calc`는 `calculator`를 대신하는 별칭이야. 의미를 알아보기 어려운 별칭을 남용하지 않는 것이 좋아.

## 6. 모듈의 변수도 가져올 수 있어

`settings.py`

```python
APP_NAME = "메모 프로그램"
MAX_MEMOS = 100
```

`main.py`

```python
import settings

print(settings.APP_NAME)
print(settings.MAX_MEMOS)
```

실행 결과:

```text
메모 프로그램
100
```

여러 파일에서 공통으로 사용하는 설정값을 한 곳에 모아 두면 수정과 관리가 편해져.

## 7. `if __name__ == "__main__":` 이해하기

모듈 파일을 직접 실행할 때만 실행하고 싶은 코드는 다음 조건문 안에 작성할 수 있어.

`calculator.py`

```python
def add(a, b):
    return a + b


if __name__ == "__main__":
    print(add(10, 5))
```

- `calculator.py`를 직접 실행하면 `15`가 출력돼.
- 다른 파일에서 `import calculator`로 가져오면 조건문 안의 출력 코드는 자동으로 실행되지 않아.
- 함수 정의는 가져온 파일에서 사용할 수 있어.

이 문법은 다른 코드에서 재사용할 기능과 직접 실행할 때 수행할 코드를 구분하는 데 도움이 돼.

## 8. 패키지란?

**패키지는 관련된 모듈을 디렉터리(폴더) 단위로 묶어 관리하는 방식**이야.

```text
python_project/
├── main.py
└── tools/
    ├── __init__.py
    ├── calculator.py
    └── text_utils.py
```

- `main.py`: 프로그램의 시작점
- `tools/`: 관련 모듈을 모아 두는 패키지 폴더
- `calculator.py`: 계산 기능
- `text_utils.py`: 문자열 처리 기능
- `__init__.py`: 패키지를 구성할 때 사용하는 파일. 이 예제에서는 비워 두어도 돼.

최신 Python에서는 일부 경우 `__init__.py` 없이도 패키지처럼 사용할 수 있지만, 학습용 프로젝트에서는 파일을 두는 구조부터 익혀도 좋아.

## 9. 패키지 안의 모듈 가져오기

위 구조에서 `tools/calculator.py`를 다음처럼 작성하자.

```python
def add(a, b):
    return a + b
```

`main.py`

```python
from tools.calculator import add

print(add(20, 30))
```

실행 결과:

```text
50
```

`tools.calculator`는 `tools` 패키지 안의 `calculator` 모듈을 뜻해.

## 10. 실습: 메모 프로그램을 여러 파일로 나누기

앞 단원에서 만든 메모 저장·조회 프로그램을 기능별로 분리해 보자.

### 10.1 파일 구조

```text
memo_project/
├── main.py
└── memo_manager.py
```

### 10.2 `memo_manager.py` 작성하기

```python
from datetime import datetime

FILE_NAME = "memo.txt"


def save_memo():
    memo = input("저장할 메모를 입력하세요: ").strip()

    if not memo:
        print("빈 메모는 저장할 수 없습니다.")
        return

    now = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

    try:
        with open(FILE_NAME, "a", encoding="utf-8") as file:
            file.write(f"[{now}] {memo}\n")
    except OSError as error:
        print(f"메모 저장 중 오류가 발생했습니다: {error}")
    else:
        print("메모를 저장했습니다.")


def read_memos():
    try:
        with open(FILE_NAME, "r", encoding="utf-8") as file:
            content = file.read()
    except FileNotFoundError:
        print("저장된 메모가 없습니다.")
    except OSError as error:
        print(f"메모 읽기 중 오류가 발생했습니다: {error}")
    else:
        print("\n===== 저장된 메모 =====")
        print(content, end="")
        print("=======================")
```

### 10.3 `main.py` 작성하기

```python
from memo_manager import save_memo, read_memos


def main():
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


if __name__ == "__main__":
    main()
```

### 10.4 실행 방법

1. VS Code에서 `memo_project` 폴더를 열어.
2. 폴더 안에 `main.py`와 `memo_manager.py`를 만들어.
3. 각각의 코드를 붙여 넣고 저장해.
4. VS Code 터미널에서 `main.py`가 있는 폴더로 이동한 다음 실행해.

```bash
python main.py
```

5. 메모 저장, 메모 조회, 종료 기능이 정상적으로 동작하는지 확인해.

**주의:** `main.py`와 `memo_manager.py`는 이 예제에서 같은 폴더에 있어야 해. 파일 이름을 `datetime.py`, `random.py`처럼 Python 표준 모듈과 같은 이름으로 지정하면 문제가 생길 수 있으니 피하는 것이 좋아.

## 11. 자주 발생하는 오류

| 오류 또는 증상 | 확인할 사항 |
|---|---|
| `ModuleNotFoundError` | 모듈 파일이 있는지, 이름과 폴더 경로가 맞는지 확인 |
| `ImportError` | 가져오려는 함수나 변수 이름이 실제로 정의되어 있는지 확인 |
| `AttributeError` | 모듈 이름과 접근하려는 함수 이름이 맞는지 확인 |
| 수정 내용이 반영되지 않음 | 실행 중인 파일, 저장 여부, 모듈 이름 충돌을 확인 |

오류가 발생하면 메시지의 마지막 부분뿐 아니라 어떤 파일의 몇 번째 줄에서 발생했는지도 확인해 봐.

## 12. 핵심 정리

- **모듈**은 Python 코드가 들어 있는 `.py` 파일이야.
- **패키지**는 관련 모듈을 폴더 단위로 구성해 관리하는 방식이야.
- `import`는 모듈을 가져오고, `from ... import ...`는 필요한 기능을 직접 가져와.
- `as`는 모듈이나 기능에 별칭을 지정할 때 사용해.
- `if __name__ == "__main__":`은 파일을 직접 실행할 때 수행할 코드를 구분하는 데 사용해.
- 기능별로 파일을 분리하면 코드 재사용과 유지보수가 쉬워져.

## 13. 실습 과제

1. `calculator.py`와 `main.py`를 만들어 덧셈과 뺄셈을 모듈로 분리해.
2. 계산 기능에 곱셈 `multiply()` 함수를 추가해.
3. `settings.py`를 만들고 프로그램 이름을 변수로 정의해 가져와.
4. 메모 프로그램을 `main.py`와 `memo_manager.py`로 분리해.
5. `memo_manager.py`의 함수 이름을 일부러 잘못 입력해 보고 오류 메시지를 확인한 다음 수정해.

## 14. 다음 학습 주제

다음 단계에서는 **클래스와 객체**를 학습해. 관련 데이터와 기능을 하나의 객체로 묶는 방법을 익히면, 메모나 상품 구매 정보를 더 체계적으로 관리할 수 있어.
