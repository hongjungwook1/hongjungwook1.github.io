---
title: Java 기본 문법 final 과 static 이란
description: >-
  final , static 키워드 완벽 정리
author: JWHONG
date: 2025-02-04 15:00:00 +0900
categories: [Java, final, static, Tutorial]
tags: [Java, final, static]
pin: true
media_subpath: "/posts/20250204"
---

# Java 기본 문법 final, static 키워드

Java에서 자체적으로 제공하는 클래스나 라이브러리를 사용하다 보면 `final`, `static` 키워드를 자주 접하게 됩니다.
단순히 문법적인 요소로만 생각하고 지나치기 쉽지만 이를 정확히 이해하고 적절히 활용하는 것이 중요합니다.

클래스, 필드, 메서드의 목적에 맞게 `final`과 `static`을 설정하면 코드의 가독성을 높이고 실수를 방지할 수 있으며 동료 개발자에게 해당 요소의 용도를 명확히 전달할 수 있습니다.

이 글에서는 `final`과 `static` 키워드가 각각 어떤 의미를 가지며 어떻게 사용되는지에 대해 자세히 알아보겠습니다.

#### Final 키워드 : "고정" - 변경 불가능

##### Final 필드 : 값이 변하지 않는다.

> `final` 키워드는 **변수를 변경할 수 없도록** 만드는 역할을 합니다.
> 즉 `final`이 붙은 필드는 **한 번 값이 할당되면 변경할 수 없습니다.**

```java
class Example {
    final int number = 10; // 값 변경 불가
}
```

> 이와 같은 특성 때문에 `final` 필드는 상수처럼 활용됩니다.
> 만약 선언과 동시에 값을 지정하지 않았다면 **생성자를 통해 단 한 번만 초기화할 수 있습니다.**

```java
class Example {
    final int number;

    Example(int num) {
        this.number = num; // 생성자를 통해 한 번만 초기화 가능
    }
}
```

---

##### Final 메서드 : 상속(Override)되지 않는다.

> 클래스 내의 특정 메서드가 `final`로 선언되면 **해당 메서드는 하위 클래스에서 오버라이딩(재정의)할 수 없습니다.**

```java
class Parent {
    final void show() {
        System.out.println("This is a final method.");
    }
}

class Child extends Parent {
    // 컴파일 오류 발생! (final 메서드는 오버라이딩 불가)
    // void show() { System.out.println("Cannot override!"); }
}
```

> `final` 메서드는 **기능이 변경되지 않아야 하는 중요한 메서드**에 사용됩니다.

---

##### Final 클래스 : 상속(Extends)되지 않는다.

> 클래스 자체가 `final`로 선언되면 더 이상 **상속이 불가능**합니다.

```java
final class FinalClass {
    void display() {
        System.out.println("This is a final class.");
    }
}

// 컴파일 오류 발생! (final 클래스는 상속 불가)
// class SubClass extends FinalClass { }
```

> 이처럼 `final` 클래스를 선언하면 **외부에서 상속을 통한 변경을 막을 수 있으며 해당 클래스를 그대로 사용하도록 강제할 수 있습니다.**

---

#### Static 키워드 : "정적" - 인스턴스 없이 사용 가능

##### static 필드 : 인스턴스 없이 사용 가능한 필드

> 일반적인 인스턴스 변수는 객체를 생성해야 사용할 수 있지만 `static` 필드는 **클래스 자체에 속하며 객체 없이도 접근이 가능합니다.**

```java
class Example {
    static int count = 0; // 정적 필드

    Example() {
        count++;
    }
}

public class Main {
    public static void main(String[] args) {
        Example e1 = new Example();
        Example e2 = new Example();

        System.out.println(Example.count); // 2
    }
}
```

> 위 코드에서 `count`는 `static` 필드이므로 여러 개의 객체가 생성되더라도 **클래스 단위로 하나의 값만 유지합니다.**

---

##### static 메서드 : 인스턴스 없이 사용 가능한 메서드

> `static` 메서드는 **인스턴스를 생성하지 않아도 호출할 수 있는 메서드**입니다.

```java
class Example {
    static void showMessage() {
        System.out.println("Hello, static method!");
    }
}

public class Main {
    public static void main(String[] args) {
        Example.showMessage(); // 객체 생성 없이 직접 호출 가능
    }
}
```

---

##### static 클래스 : 인스턴스 없이 사용 가능한 (Nested) 클래스

> 클래스 내부에 선언되는 **중첩(Nested) 클래스**가 `static`으로 선언되면 **외부 클래스의 인스턴스 없이도 사용할 수 있습니다.**

```java
class Outer {
    static class Inner {
        void show() {
            System.out.println("Static nested class.");
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Outer.Inner obj = new Outer.Inner(); // 외부 클래스 객체 없이 직접 사용 가능
        obj.show();
    }
}
```
