---
title: Java Solid 원칙과 디자인 패턴
description: >-
  좋은 객체지향 프로그래밍 방법론 SOLID 원칙 그리고 디자인 패턴
author: JWHONG
date: 2025-02-03 11:00:00 +0900
categories: [Java, Solid, OOP, Tutorial]
tags: [Java, Solid, OOP, Tutorial]
pin: true
media_subpath: "/posts/20250203"
---

# 좋은 객체지향 프로그래밍을 위한 방법론

## 좋은 코드란 무엇인가?

객체지향뿐만 아니라 좋은 코드란 변경이 용이하고 유지보수와 확장이 쉬운 코드입니다.
이를 위해 다음과 같은 원칙을 고려해야 합니다.

- **중복의 최소화(DRY: Don’t Repeat Yourself)**
  하나의 수정이 다른 여러 부분의 수정으로 이어지지 않도록 중복을 최소화해야 합니다.

- **코드 변경의 용이성**
  코드가 항상 완벽할 수 없으며 요구사항은 지속적으로 변경됩니다. 따라서 변경이 용이해야 합니다.

- **재사용성**
  잘 정리된 코드는 유사한 다른 요구사항에서도 그대로 사용할 수 있어야 합니다.

## 클린 코드의 기준 : 단일 목적과 오용, 남용 방지

모든 변수, 함수, 클래스는 명확한 목적을 가지고 적절하게 사용되어야 합니다.

다음 원칙들을 지키는 것이 중요합니다.

1. **DRY (Don't Repeat Yourself) - 중복을 피하라**
   > 같은 기능을 하는 로직이 여러 곳에 존재해서는 안 된다

- 동일한 로직이 여러 곳에 존재하면 수정 시 모든 곳을 변경해야 하므로 유지보수가 어렵습니다.
- 공통 로직은 하나의 함수 또는 클래스로 묶어 재사용성을 높여야 합니다.

2. **KISS (Keep It Simple and Stupid) - 단순하게 유지하라**
   > 함수는 하나의 역할만 수행해야 한다.

- 함수나 클래스는 하나의 역할만 수행해야 하며 여러 개의 기능을 묶지 않아야 합니다.

3. **YAGNI (You Ain't Gonna Need It) - 미래의 불필요한 기능을 만들지 마라**
   > 현재 필요하지 않은 기능을 미리 구현하지 않는다.

- 미래에 필요할 것 같다고 기능을 추가하는 것은 코드의 복잡성을 증가시키고 유지보수 비용을 높입니다.

---

## 객체지향 설계 원칙 (SOLID)

SOLID 원칙은 객체지향 프로그래밍에서 유지보수성과 확장성을 높이는 핵심 개념입니다.

### 단일 책임 원칙 (Single Responsibility Principle, SRP)

> 하나의 클래스(또는 함수)는 하나의 책임(기능)만 가져야 한다.

**잘못된 예시**

```java
public class Ramen {
    public void make() {
        System.out.println("물 끓이기");
        System.out.println("스프 넣기");
        System.out.println("면 넣기");
    }
}
```

- Ramen 클래스가 모든 조리 과정을 직접 처리하고 있어 책임이 분리되지 않음.

**개선된 예시**

```java
@RequiredArgsConstructor
public class Ramen {
    private final Water water;
    private final Soup soup;
    private final Noodle noodle;

    public void make() {
        water.input();
        soup.input();
        noodle.input();
    }
}

public class Soup {
    public void input() { System.out.println("스프 넣기"); }
}
```

- `Water`, `Soup`, `Noodle` 클래스를 별도로 분리하여 책임을 각각의 클래스에 부여.

---

### 개방-폐쇄 원칙 (Open-Closed Principle, OCP)

> 확장에는 열려 있어야 하고, 수정에는 닫혀 있어야 한다.

**잘못된 예시 (수정이 필요한 코드)**

```java
public class Ramen {
    private final Water water;
    private final String soupType;

    public void make() {
        water.input();
        if (soupType.equals("신라면")) {
            System.out.println("신라면 스프 넣기");
        } else if (soupType.equals("진라면")) {
            System.out.println("진라면 스프 넣기");
        }
    }
}
```

- 새로운 라면 종류를 추가하려면 기존 코드를 수정해야 하므로 OCP를 위반.

**개선된 예시**

```java
public interface Soup {
    void input();
}

public class SinSoup implements Soup {
    public void input() { System.out.println("신라면 스프 넣기"); }
}

public class JinSoup implements Soup {
    public void input() { System.out.println("진라면 스프 넣기"); }
}

@RequiredArgsConstructor
public class Ramen {
    private final Water water;
    private final Soup soup;
    private final Noodle noodle;

    public void make() {
        water.input();
        soup.input();
        noodle.input();
    }
}
```

- `Soup` 인터페이스를 통해 새로운 스프를 추가할 때 기존 클래스를 수정할 필요 없음.

---

### 리스코프 치환 원칙 (Liskov Substitution Principle, LSP)

> 하위 클래스는 상위 클래스를 대체할 수 있어야 한다.

리스코프 치환 원칙이란 상위 클래스가 사용되는 모든 곳에서 하위 클래스를 대체하여 사용할 수 있어야 한다는 원칙입니다.
즉, 상속 관계에서는 부모 클래스가 수행할 수 있는 모든 작업을 자식 클래스도 동일하게 수행할 수 있어야 합니다.

**잘못된 예시 (LSP 위반)**

```java
class Rectangle {
    protected int width;
    protected int height;

    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int getArea() {
        return width * height;
    }
}

class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width; // 정사각형은 너비와 높이가 같아야 하므로 변경
    }
}
```

- 위 코드에서 `Square`(정사각형) 클래스는 `Rectangle`(직사각형)을 상속받았지만 `setWidth()`를 오버라이드하면서 `height` 값도 같이 변경하고 있습니다. 이로 인해 `Rectangle`이 예상하는 동작이 깨질 수 있습니다.

**개선된 예시 (LSP 준수)**

```java
interface Shape {
    int getArea();
}

class Rectangle implements Shape {
    protected int width;
    protected int height;

    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public int getArea() {
        return width * height;
    }
}

class Square implements Shape {
    private int side;

    public Square(int side) {
        this.side = side;
    }

    @Override
    public int getArea() {
        return side * side;
    }
}
```

- 위처럼 `Rectangle`과 `Square`를 각각 `Shape` 인터페이스를 통해 별도로 정의하면 두 클래스가 상속 관계에서 발생하는 문제 없이 독립적으로 동작할 수 있습니다.

---

### 인터페이스 분리 원칙 (Interface Segregation Principle, ISP)

> 클라이언트가 필요하지 않는 인터페이스에 의존하지 않도록 인터페이스를 최소한으로 유지해야 한다.

**잘못된 예시 (불필요한 메서드 포함)**

```java
public interface Soup {
    void input();
    void onion();
    void egg();
}
```

- `onion()`과 `egg()` 메서드가 필요 없는 경우에도 구현해야 하므로 ISP를 위반.

**개선된 예시 (인터페이스 분리)**

```java
public interface Soup {
    void input();
}

public interface Onion {
    void input();
}

public interface Egg {
    void input();
}
```

- 필요한 기능만 가진 인터페이스를 분리하여 각 클래스가 필요 없는 메서드를 구현하지 않도록 개선.

---

### 의존성 역전 원칙 (Dependency Inversion Principle, DIP)

> 고수준 모듈이 저수준 모듈에 의존하지 않고, 인터페이스(추상화)에 의존해야 한다.

**잘못된 예시 (불필요한 메서드 포함)**

```java
public interface Soup {
    void input();
    void onion();
    void egg();
}
```

- `onion()`과 `egg()` 메서드가 필요 없는 경우에도 구현해야 하므로 ISP를 위반.

**개선된 예시 (인터페이스 분리)**

```java
public interface Soup {
    void input();
}

public interface Onion {
    void input();
}

public interface Egg {
    void input();
}
```

- 필요한 기능만 가진 인터페이스를 분리하여 각 클래스가 필요 없는 메서드를 구현하지 않도록 개선.

---

### 디자인 패턴 (Design Patterns)

디자인 패턴은 반복적으로 발생하는 소프트웨어 설계 문제를 해결하기 위한 재사용 가능한 해결책입니다.
이는 객체지향 설계 원칙을 기반으로 한 일반적인 설계 템플릿이라고 볼 수 있습니다.

**디자인 패턴의 목표**

- `코드 재사용성 향상` : 이미 검증된 패턴을 사용하여 중복을 줄이고 유지보수를 쉽게 만듦
- `유지보수성 향상` : 설계가 구조적으로 정리되어 변경이 용이함
- `확장성 증가` : 기존 코드에 영향을 최소화하면서 기능을 확장할 수 있음

![Image](https://github.com/user-attachments/assets/5ac869e4-b587-4825-95f2-d19a3b215008)
