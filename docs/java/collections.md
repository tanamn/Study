---
layout: default
title: Java 컬렉션
permalink: /java/collections/
---

{% include navigation.html %}

# Java 컬렉션

컬렉션 프레임워크는 여러 객체를 저장하고 관리하기 위한 자료구조와 인터페이스를 제공합니다.

## 1. 주요 컬렉션 비교

| 종류 | 특징 | 대표 구현체 |
|---|---|---|
| `List` | 순서가 있고 중복 허용 | `ArrayList`, `LinkedList` |
| `Set` | 중복 원소를 허용하지 않음 | `HashSet`, `LinkedHashSet`, `TreeSet` |
| `Queue` | 대기열 처리에 사용 | `LinkedList`, `PriorityQueue` |
| `Map` | 키와 값의 쌍으로 저장 | `HashMap`, `LinkedHashMap`, `TreeMap` |

`Map`은 `Collection` 인터페이스를 구현하는 타입은 아니지만 컬렉션 프레임워크의 주요 구성 요소입니다.

## 2. List

```java
List<String> names = new ArrayList<>();

names.add("김민수");
names.add("이서연");
names.add("김민수");

System.out.println(names.size()); // 3
System.out.println(names.get(0)); // 김민수
```

`ArrayList`는 인덱스로 요소를 접근하기 편리하며 일반적인 목록 데이터 처리에 자주 사용됩니다.

## 3. Set

```java
Set<String> codes = new HashSet<>();

codes.add("A01");
codes.add("A02");
codes.add("A01");

System.out.println(codes.size()); // 2
```

`HashSet`은 중복을 허용하지 않지만 요소의 출력 순서를 보장하지 않습니다.

## 4. Map

```java
Map<String, Long> balances = new HashMap<>();

balances.put("1001", 500_000L);
balances.put("1002", 300_000L);

Long balance = balances.get("1001");
System.out.println(balance); // 500000
```

- `put(key, value)`: 키와 값을 저장합니다.
- `get(key)`: 키에 해당하는 값을 조회합니다.
- `containsKey(key)`: 키의 존재 여부를 확인합니다.
- `remove(key)`: 해당 키의 매핑을 제거합니다.

키는 중복될 수 없으며, 동일한 키로 다시 `put()`하면 기존 매핑의 값이 교체됩니다.

## 5. 제네릭과 반복

제네릭은 컬렉션 등에 저장할 데이터 타입을 지정해 타입 안정성을 높입니다.

```java
List<Integer> numbers = new ArrayList<>();
numbers.add(10);
numbers.add(20);

for (Integer number : numbers) {
    System.out.println(number);
}
```

`List<Integer>`는 정수를 다루는 목록임을 명확하게 표현합니다. 기본형 `int` 대신 객체 타입인 `Integer`를 사용합니다.

## 6. 정렬과 선택 기준

컬렉션을 정렬할 때는 **기본 정렬 순서**를 사용할지, 아니면 업무에 맞는 **별도의 정렬 기준**을 지정할지 결정합니다. 정렬이 필요한지와 함께 중복 허용 여부, 데이터 조회 방식도 자료구조 선택에 영향을 줍니다.

### 6.1 `Collections.sort()`로 기본 정렬하기

`Collections.sort(list)`는 `List`의 요소를 기본 정렬 순서로 정렬합니다. 숫자는 오름차순, 문자열은 사전식 순서로 정렬됩니다. 이 메서드는 기존 목록 자체의 순서를 변경합니다.

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

List<Integer> scores = new ArrayList<>();
scores.add(80);
scores.add(95);
scores.add(70);

Collections.sort(scores);
System.out.println(scores); // [70, 80, 95]
```

문자열도 기본 정렬할 수 있습니다.

```java
List<String> names = new ArrayList<>();
names.add("Choi");
names.add("Kim");
names.add("Lee");

Collections.sort(names);
System.out.println(names); // [Choi, Kim, Lee]
```

기본 정렬은 숫자나 문자열처럼 자연스러운 순서가 정의된 데이터에 적합합니다. 객체를 정렬하려면 해당 객체의 기본 정렬 기준(`Comparable`)을 정의하거나 `Comparator`를 전달해야 합니다.

### 6.2 `Comparator`로 정렬 기준 지정하기

`Comparator`는 기본 정렬 순서와 다른 기준을 적용하거나, 클래스 자체를 수정하지 않고 정렬 기준을 지정할 때 사용합니다.

예를 들어 점수를 높은 순서로 정렬하려면 다음과 같이 작성할 수 있습니다.

```java
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;

List<Integer> scores = new ArrayList<>();
scores.add(80);
scores.add(95);
scores.add(70);

scores.sort(Comparator.reverseOrder());
System.out.println(scores); // [95, 80, 70]
```

객체의 특정 필드를 기준으로 정렬할 수도 있습니다.

```java
import java.util.Comparator;
import java.util.List;

class Employee {
    String name;
    int salary;

    Employee(String name, int salary) {
        this.name = name;
        this.salary = salary;
    }
}

List<Employee> employees = List.of(
    new Employee("김민수", 4_000),
    new Employee("이서연", 5_000),
    new Employee("박지훈", 3_500)
);

employees.stream()
    .sorted(Comparator.comparingInt(employee -> employee.salary))
    .forEach(employee ->
        System.out.println(employee.name + ": " + employee.salary));
```

출력은 급여 오름차순으로 `박지훈`, `김민수`, `이서연` 순서가 됩니다. 위 예시는 `stream().sorted()`로 정렬 결과를 처리하므로 원래 `employees` 목록 자체를 정렬하지는 않습니다. 목록 자체를 정렬하려면 변경 가능한 목록에서 `employees.sort(Comparator.comparingInt(employee -> employee.salary));`를 사용할 수 있습니다.

> 참고: `List.of()`는 변경할 수 없는 목록을 만듭니다. 따라서 이 목록에 `sort()`를 직접 호출해 순서를 변경할 수는 없습니다. 예시에서는 정렬된 결과를 스트림으로 처리했습니다.

### 6.3 `TreeSet`과 `TreeMap`

`TreeSet`과 `TreeMap`은 요소 또는 키를 정렬된 순서로 관리합니다. 기본적으로 자연 정렬 순서를 사용하며, 생성할 때 `Comparator`를 전달해 정렬 기준을 지정할 수도 있습니다.

**`TreeSet`: 중복 없는 정렬 집합**

```java
import java.util.Set;
import java.util.TreeSet;

Set<Integer> numbers = new TreeSet<>();
numbers.add(30);
numbers.add(10);
numbers.add(20);
numbers.add(10);

System.out.println(numbers); // [10, 20, 30]
```

- 중복 요소는 저장하지 않습니다.
- 요소가 정렬된 순서로 순회됩니다.
- 요소의 순서를 유지할 필요가 없고 정렬도 필요 없다면 `HashSet`이 더 적합할 수 있습니다.

**`TreeMap`: 키 기준으로 정렬되는 맵**

```java
import java.util.Map;
import java.util.TreeMap;

Map<String, Long> balances = new TreeMap<>();
balances.put("2002", 300_000L);
balances.put("1001", 500_000L);
balances.put("1500", 200_000L);

System.out.println(balances);
// {1001=500000, 1500=200000, 2002=300000}
```

`TreeMap`은 **값이 아니라 키를 기준으로 정렬**합니다. 위 예시는 계좌번호 문자열의 오름차순으로 정렬된 결과를 보여 줍니다. 숫자로 된 문자열 키는 숫자 크기가 아니라 문자열 순서로 비교될 수 있으므로, 숫자 크기 순서가 필요하면 숫자 타입 키를 사용하는 것이 적절합니다.

### 6.4 자료구조 선택 기준

| 필요한 기능 | 고려할 자료구조 | 선택 이유 |
|---|---|---|
| 입력한 순서를 유지하고 인덱스로 조회 | `ArrayList` | 순서와 인덱스 접근이 중요할 때 적합 |
| 중복을 허용하지 않고 빠르게 존재 여부 확인 | `HashSet` | 중복 제거가 필요하고 정렬은 불필요할 때 적합 |
| 중복 없이 정렬된 요소 관리 | `TreeSet` | 중복 제거와 정렬이 모두 필요할 때 적합 |
| 키로 값을 조회하고 입력 순서를 유지 | `LinkedHashMap` | 키-값 조회와 입력 순서 유지가 필요할 때 적합 |
| 키로 값을 조회하고 키 순서대로 정렬 | `TreeMap` | 키 기준 정렬이 필요할 때 적합 |

자료구조를 선택할 때는 다음 질문을 순서대로 생각하면 좋습니다.

1. **중복이 필요한가?** 중복이 필요하면 `List`, 필요 없으면 `Set`을 우선 고려합니다.
2. **입력 순서를 유지해야 하는가?** 필요하면 `ArrayList` 또는 `LinkedHashSet`·`LinkedHashMap` 등을 고려합니다.
3. **정렬된 순서가 필요한가?** 필요하면 `TreeSet`, `TreeMap` 또는 정렬 가능한 `List`를 고려합니다.
4. **어떤 방식으로 조회하는가?** 인덱스 기반이면 `List`, 키 기반이면 `Map`, 포함 여부 확인이 중심이면 `Set`이 적합할 수 있습니다.
5. **데이터 크기와 성능 요구는 어떠한가?** 자료구조별 시간 복잡도와 메모리 사용량도 함께 검토합니다. 정렬된 자료구조는 정렬을 유지하는 비용이 있으므로 항상 가장 빠른 선택은 아닙니다.

금융 업무 예시로 보면, 거래 내역을 발생 순서대로 화면에 표시할 때는 `List`를, 중복된 코드 목록을 제거할 때는 `Set`을, 계좌번호로 잔액을 찾을 때는 `Map`을 고려할 수 있습니다. 다만 실제 업무에서는 거래 발생 시각, 계좌번호의 자료형, 동시성 및 데이터베이스 조회 여부 등 추가 조건도 함께 살펴야 합니다.

## 학습 체크

- `List`, `Set`, `Map`의 차이를 설명할 수 있다.
- 중복을 허용하지 않는 자료구조를 선택할 수 있다.
- `Map`의 키와 값 사용법을 설명할 수 있다.
- 제네릭이 필요한 이유를 설명할 수 있다.

[이전: 클래스와 객체]({{ '/java/classes-objects/' | relative_url }}) · [다음: 예외 처리]({{ '/java/exceptions/' | relative_url }})
