---
title: Java & Spring에서 자주 사용하는 어노테이션 정리
description: >-
    Java 기본 어노테이션부터 Lombok, Spring, AOP 어노테이션까지 정리하고 각각의 동작 방식과 사용법
author: JWHONG
date: 2025-03-21 16:00:00 +0900
categories: [Java, Spring, Lombok, AOP, Annotation]
tags: [Java, Spring, Lombok, AOP, Annotation]
pin: true
media_subpath: "/posts/20250321"
---

## 어노테이션(Annotation)이란?

어노테이션(Annotation)은 **소스 코드에 추가적인 정보를 제공하는 메타데이터**입니다.  
실행 중 특정 코드가 어떻게 동작해야 하는지를 **컴파일러 또는 프레임워크에 전달하는 역할을 합니다.**

> 하지만 어노테이션 자체만으로 동작을 수행하지 않습니다. 어노테이션을 해석하는 **컴파일러, 프레임워크, AOP 프록시**가 있어야 합니다.

---

## Java 기본 어노테이션 (@Override, @Deprecated 등)

**Java에서 기본 제공하는 어노테이션**은 컴파일러에게 특정한 정보를 전달하는 역할을 합니다.

- **@Override** : 부모 클래스의 메서드를 재정의 할때 사용
- **@Deprecated** : 해당 요소(클래스, 메서드 등)가 더 이상 사용되지 않음을 나타냄
- **@SuppressWarnings("unchecked")** : 특정 경고를 무시하도록 설정

```java
class Parent {
    void print() {
        System.out.println("Parent method");
    }
}

class Child extends Parent {
    @Override  // 부모 메서드를 재정의했음을 명시
    void print() {
        System.out.println("Child method");
    }
}
```

---

## Lombok 어노테이션 (@RequiredArgsConstructor 등)

Lombok은 **반복적인 코드(boilerplate)를 자동 생성해주는 라이브러리**입니다.  
대표적인 어노테이션은 다음과 같습니다.

- `@Getter`, `@Setter` : 자동으로 getter, setter 메서드 생성
- `@RequiredArgsConstructor` : final 필드만을 초기화하는 생성자 자동 생성
- `@NoArgsConstructor` : 기본 생성자 자동 생성
- `@AllArgsConstructor` : 모든 필드를 초기화하는 생성자 자동 생성

```java
import lombok.RequiredArgsConstructor;

@RequiredArgsConstructor
public class ExampleClass {
    private final String name;
    private final int age;
}
```

Lombok이 컴파일 시 코드를 변환하여 아래처럼 생성자를 자동 추가합니다.

```java
public class ExampleClass {
    private final String name;
    private final int age;

    public ExampleClass(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

---

## Spring AOP & 트랜잭션 어노테이션 (@Transactional)

**Spring AOP(Aspect-Oriented Programming)**을 활용하면 트랜잭션, 로깅, 보안 등 부가적인 기능을 핵심 로직과 분리할 수 있습니다.

- `@Transactional` : 메서드 실행을 트랜잭션 단위로 관리
- `@Aspect` : AOP에서 사용되는 클래스임을 명시
- `@Before`, `@After`, `@Around` : 특정 메서드 실행 전후에 부가 기능을 삽입

---

## Spring 주요 어노테이션 (@Controller, @Service, @Repository 등)

Spring에서는 특정 역할을 하는 클래스를 정의할 때 다양한 어노테이션을 사용합니다.

- `@Controller` : 웹 컨트롤러 역할 (MVC 패턴에서 사용)
- `@Service` : 비즈니스 로직을 담당하는 서비스 계층
- `@Repository`: 데이터 액세스 계층 (DAO)
- `@Configuration` : 설정 클래스를 나타냄

---

## 어노테이션 정의 방법

### @Target : 어디에 적용할 것인가?

어노테이션이 적용될 수 있는 위치를 지정합니다.

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
public @interface CustomAnnotation {}
```

|**값**|**설명**|
|:---:|:---:|
|**ElementType.TYPE**|클래스 , 인터페이스 , ENUM에 적용|
|**ElementType.METHOD**|메소드에 적용|
|**ElementType.FIELD**|멤버 변수에 적용|


### @Retention : 언제까지 유지할 것인가?

어노테이션의 **생명 주기**를 지정합니다.

```java
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME)
public @interface CustomAnnotation {}
```

|**값**|**설명**|
|:---:|:---:|
|**RetentionPolicy.SOURCE**|컴파일 후 사라짐 (예: @Override)|
|**RetentionPolicy.CLASS**|바이트코드에 남아 있지만 리플렉션을 사용하지 않는 경우 실행 시 사용되지 않음|
|**RetentionPolicy.RUNTIME**|실행 시점까지 유지됨 (Spring AOP에서 사용)|

---

## 결론 

어노테이션은 코드에 부가적인 정보를 추가하는 메타데이터이지만 Spring, Lombok, AOP 등의 동작 방식에서 핵심적인 역할을 합니다.