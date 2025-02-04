---
title: Java 확장 문법 Lombok이란
description: >-
  Lombok 메서드 주입으로 편리해진 개발과 객체 생성 방법
author: JWHONG
date: 2025-02-04 11:00:00 +0900
categories: [Java, Lombok, Tutorial]
tags: [Java, Lombok]
pin: true
media_subpath: "/posts/20250204"
---

# Java 확장 문법 Lombok이란

## Lombok의 원리 - AST 조작을 통한 바이트코드 주입

**Lombok**이 동작하는 방식은 단순한 코드 자동 생성이 아닙니다.
Java 컴파일 과정에서 **Annotation Processor**를 활용하여 소스 코드의 **AST(Abstract Syntax Tree)** 를 동적으로 조작하는 방식으로 동작합니다.

#### Lombok의 동작 과정

1️. Javac(컴파일러) 가 소스 코드를 파싱하여 AST(Abstract Syntax Tree)를 생성
2️. Lombok Annotation Processor 가 AST를 분석한 후 필요한 Getter, Setter, 생성자 등의 메서드를 추가
3️. Javac(컴파일러) 는 변환된 AST를 바탕으로 최종적으로 바이트코드(Bytecode)를 생성

우리가 소스 코드에서 직접 Getter, Setter 등을 작성하지 않아도 Lombok이 컴파일 과정에서 자동으로 추가해 주는 것입니다.

---

#### Lombok의 주요 어노테이션 정리

이제 Lombok에서 가장 많이 사용되는 어노테이션을 하나씩 살펴보겠습니다.

##### @AllArgsConstructor - 모든 필드를 포함하는 생성자 자동 생성

```java
@AllArgsConstructor
public class User {
    private String name;
    private int age;
}
```

> **User 객체를 생성할 때 new User("홍길동", 25); 형태로 모든 필드를 초기화할 수 있도록 해줍니다.**

---

##### @Getter / @Setter - Getter, Setter 자동 생성

```java
@Getter
@Setter
public class User {
    private String name;
    private int age;
}
```

> **별도로 getName(), setName() 등의 메서드를 구현할 필요 없이 자동으로 생성됩니다**

> **단 무조건 모든 필드에 적용하는 것은 좋지 않음!
> 불필요한 Setter가 생성되면 객체의 불변성(immutability)이 깨질 위험이 있으므로 신중하게 사용해야 합니다.**

---

##### @ToString - 객체의 toString() 자동 생성

```java
@ToString
public class User {
    private String name;
    private int age;
}
```

> **System.out.println(user); 실행 시
> User(name=홍길동, age=25) 처럼 자동으로 깔끔한 문자열 출력을 지원합니다.**

---

##### @RequiredArgsConstructor - 필수 필드만 포함하는 생성자 생성

```java
@RequiredArgsConstructor
public class User {
    private final String name;
    private int age;
}
```

> \*\*`final` 또는 `@NonNull`이 붙은 필드만 포함하는 생성자를 자동 생성합니다.

---

##### @NoArgsConstructor - 매개변수가 없는 기본 생성자 생성

```java
@NoArgsConstructor
public class User {
    private String name;
    private int age;
}
```

> **기본 생성자가 필요할 때 유용하게 사용할 수 있습니다.
> 특히 JPA 엔티티(Entity)에서는 기본 생성자가 필수이므로 @NoArgsConstructor를 많이 사용합니다.**

---

##### @EqualsAndHashCode - equals(), hashCode() 자동 생성

```java
@EqualsAndHashCode
public class User {
    private String name;
    private int age;
}
```

> **객체 비교 시 equals()와 hashCode()를 자동으로 구현해 줍니다**

- `==` 연산자는 주소값을 비교하기 때문에 `객체 내부의 값`이 같더라도 false가 나올 수 있습니다.

- `equals()`와 `hashCode()`를 올바르게 구현하면 동일한 값이면 같은 객체로 인식할 수 있습니다.

---

##### @FieldDefaults - 필드 접근제어자 일괄 적용

```java
@FieldDefaults(level = AccessLevel.PRIVATE, makeFinal = true)
public class User {
    String name;
    int age;
}
```

> **모든 필드에 private final을 자동으로 적용하여 코드를 간결하게 유지할 수 있습니다.
> 특히 DTO나 Service 클래스에서 필드를 일괄적으로 private으로 설정할 때 유용합니다.**

---

### 결론 : Lombok을 사용하면 생산성이 향상된다!

Lombok은 반복적인 코드를 줄여 개발 효율성을 높이는 강력한 라이브러리입니다.
하지만 무분별한 사용은 오히려 유지보수를 어렵게 만들 수도 있습니다.

- Getter, Setter는 꼭 필요한 경우에만 사용
- 생성자는 상황에 맞게 @AllArgsConstructor / @RequiredArgsConstructor 선택

Lombok을 잘 활용하면 코드를 깔끔하게 유지하면서 유지보수와 확장성이 좋은 구조를 만들 수 있습니다.
