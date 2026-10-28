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

- `Collections.sort(list)`: 기본 정렬 순서로 목록을 정렬합니다.
- `Comparator`: 별도의 정렬 기준을 정의할 때 사용합니다.
- `TreeSet`, `TreeMap`: 정렬된 순서로 데이터를 관리할 때 활용할 수 있습니다.
- 자료구조를 선택할 때는 데이터 양, 조회 방식, 정렬 필요 여부, 중복 허용 여부를 고려합니다.

## 학습 체크

- [ ] `List`, `Set`, `Map`의 차이를 설명할 수 있다.
- [ ] 중복을 허용하지 않는 자료구조를 선택할 수 있다.
- [ ] `Map`의 키와 값 사용법을 설명할 수 있다.
- [ ] 제네릭이 필요한 이유를 설명할 수 있다.

[이전: 클래스와 객체]({{ '/java/classes-objects/' | relative_url }}) · [다음: 예외 처리]({{ '/java/exceptions/' | relative_url }})
