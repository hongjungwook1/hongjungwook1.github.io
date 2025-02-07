---
title: Java 기초 개념 총정리 - 컬렉션 , Optional , Stream
description: >-
  Java의 핵심 개념인 컬렉션, Optional, Stream, 그리고 SOLID 원칙에 대해 정리하였습니다.
author: JWHONG
date: 2025-02-05 11:00:00 +0900
categories: [Java, Collection, Optional, Stream, Tutorial]
tags: [Java, Collection, Optional, Stream]
pin: true
media_subpath: "/posts/20250205"
---

# Java 개념 정리: 컬렉션, Optional, Stream

## Java 컬렉션 프레임워크

### List (ArrayList, LinkedList)

> **가장 많이 사용하는 자료구조.
> 일렬로 나열된 데이터 집합이며, 값을 자유롭게 추가, 수정, 삭제할 수 있습니다.**

- **ArrayList**: 조회 성능이 우수.
- **LinkedList**: 삽입/삭제 성능이 우수.

### Set (HashSet, TreeSet)

> **중복을 방지하는 리스트와 유사한 자료구조.**

- **HashSet**: 중복을 허용하지 않는 기본적인 Set.
- **TreeSet**: 자동 정렬 기능이 추가된 Set.

### Map (HashMap, TreeMap, LinkedHashMap)

> **Key-Value 기반 자료구조.
> 데이터베이스의 Primary Key - Row 구조와 유사함.**

- **HashMap**: 기본적인 Key-Value 저장소.
- **TreeMap**: Key 값에 따라 자동 정렬.
- **LinkedHashMap**: Key 입력 순서를 유지.

### Generic (`<T>`) 개념

> **Generic = `<T>` : 귀에 걸면 "귀"걸이, 코에 걸면 "코"걸이**

- 제네릭은 `<T>`를 활용하여 **우리가 원하는 타입을 넣으면 해당 타입에 맞게 클래스가 변화**하는 기능을 제공합니다.

```java
public class Box<T> {
    private T value;

    public void setValue(T value) {
        this.value = value;
    }

    public T getValue() {
        return value;
    }
}

Box<String> stringBox = new Box<>();
stringBox.setValue("Hello");
System.out.println(stringBox.getValue());
```

---

## Optional의 등장 이유

> **null을 외부에서 처리하지 말고 내부에서 처리하자.**

기존에는 if 문을 사용해 null을 체크했지만 이제는 Optional을 사용할 수 있습니다.

```java
Optional<String> optional = Optional.ofNullable(null);
String result = optional.orElse("Default Value");
```

- `.ofNullable()`: 값이 null인지 확인 후 가져옴.
- `.orElse()`: 기본값 설정.
- `.ifPresent()`: 값이 존재할 때만 실행.

---

## Stream의 등장 이유

> **컬렉션의 요소에 대한 연산을 함수형 프로그래밍 방식으로 처리하자.**

기존의 for/while 문을 사용한 외부 반복을 Stream을 이용한 내부 반복으로 대체할 수 있습니다.

```java
List<String> list = Arrays.asList("A", "B", "C");
list.stream()
    .map(String::toLowerCase)
    .forEach(System.out::println);
```

- **외부 반복**: for/while
- **내부 반복**: Stream API
- `.toList()`를 사용하면 불변 리스트로 반환됩니다.

---

## Enum : 분류된 객체 모음

### Enum 을 통한 메서드 파라미터 제약

메서드 수행 시 파라미터에 너무 많은 경우의 수가 발생하는 것이 싫을 경우 **파라미터를 한정적으로 정의하여 정의**할 수 있습니다.

```java
public enum Status {
    SUCCESS, FAIL, PENDING;
}

public void process(Status status) {
    switch (status) {
        case SUCCESS -> System.out.println("성공");
        case FAIL -> System.out.println("실패");
        case PENDING -> System.out.println("대기 중");
    }
}
```

### Enum 을 통한 메서드 내 로직/변수 제약

- Enum을 활용하면 **메서드의 파라미터 값뿐만 아니라 내부 로직에서 활용할 값도 제한할 수 있습니다.**
- 특정 Enum 값에 따라 고정된 로직을 적용할 수 있습니다.

```java
public enum Operation {
    ADD {
        @Override
        public int apply(int a, int b) {
            return a + b;
        }
    },
    MULTIPLY {
        @Override
        public int apply(int a, int b) {
            return a * b;
        }
    };

    public abstract int apply(int a, int b);
}

int result = Operation.ADD.apply(5, 3); // 8
```
