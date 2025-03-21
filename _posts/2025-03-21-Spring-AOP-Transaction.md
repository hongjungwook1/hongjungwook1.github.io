---
title: Spring AOP와 Transaction 동작 원리
description: >-
    Spring AOP와 @Transactional의 동작 원리를 이해하고 JDK Dynamic Proxy와 CGLIB을 활용한 트랜잭션 관리 방식과 주의해야 할 사항
author: JWHONG
date: 2025-03-21 11:00:00 +0900
categories: [Spring , Spring Boot , Transaction, Database , AOP]
tags: [Spring , Spring Boot , Transaction, Database , AOP]
pin: true
media_subpath: "/posts/20250321"
---

## @Transactional과 Spring AOP 원리 (JDK Dynamic Proxy / CGLIB)

Spring에서 `@Transactional`을 사용하려면 **트랜잭션 관리 기능을 활성화하는 설정이 필요합니다.**

- 일반적으로 `@EnableTransactionManagement`를 선언해야 하지만
- **Spring Boot**에서는 **자동 설정(AutoConfiguration)** 덕분에 기본적으로 활성화됩니다.

## AOP란? (Aspect-Oriented Programming)

AOP는 **Aspect-Oriented Programming(관점 지향 프로그래밍)**의 약자로 핵심 로직과 부가적인 기능을 분리하여 개발하는 방식입니다.

### AOP의 필요성

애플리케이션을 개발하다 보면 **공통적으로 반복되는 부가적인 기능**이 많아질 수 있습니다.
    - **비즈니스 로직(핵심 기능)** : 실제 애플리케이션의 주요 동작을 담당
    - **부가 기능(인프라 로직)** : 로깅, 보안, 트랜잭션 관리, 예외 처리, 캐싱 등

> 이러한 부가 기능을 AOP로 분리하면 **코드의 중복을 줄이고 가독성을 높이며 유지보수를 쉽게 할 수 있습니다.**

### AOP의 주요 개념

- **Target** : 부가 기능을 적용할 대상 (클래스, 메서드 등)
- **Advice** : 실제 부가 기능을 구현하는 코드
    - **@Before** : 메서드 실행 전에 실행
    - **@After** : 메서드 실행 후 항상 실행
    - **@AfterReturning** : 정상 실행 후 실행
    - **@AfterThrowing** : 예외 발생 시 실행
    - **@Around** : 메서드 실행 전/후에 실행
- **Join Point** : Advice가 적용될 수 있는 지점 (메서드 실행, 객체 생성 등)
- **Pointcut** : 실제 Advice가 적용될 대상을 정의하는 표현식

---

## AOP 적용 방식 (Compile Weaving vs Runtime Weaving)

AOP를 적용하는 방식에는 크게 **컴파일 시점(Compile-Time Weaving)**과 **런타임 시점(Runtime Weaving)**이 있습니다.

### Compile-Time Weaving (CTW)

- **AspectJ**를 활용해 **바이트코드를 조작**하여 AOP를 적용하는 방식
- 높은 성능을 제공하지만 별도의 **컴파일러(AJC)** 사용 및 JVM 실행 옵션 변경이 필요

### Runtime Weaving (RTW)

- **Spring AOP**는 런타임에 **프록시 객체(Proxy Object)**를 생성하여 AOP를 적용합니다.
- 바이트코드를 직접 조작하지 않기 때문에 원본 코드를 변경하지 않아도 됨
- **Spring AOP는 기본적으로 Runtime Weaving 방식을 사용**

---

## Spring AOP 동작 원리 : 프록시 패턴

Spring AOP는 **프록시 패턴(Proxy Pattern)**을 사용하여 부가 기능을 적용합니다.

### 프록시 패턴이란?

프록시(Proxy)란 실제 객체(Target) 대신 대리 객체를 만들어 메서드를 실행하는 패턴

### Spring AOP의 동작 방식

1. 클라이언트가 메서드를 호출하면 직접 타겟 객체를 호출하는 것이 아니라 **프록시 객체**가 이를 가로챕니다.
2. 프록시 객체는 부가 기능(트랜잭션, 로깅 등)을 수행한 후 타겟 객체의 메서드를 호출합니다.
3. 타겟 메서드가 실행된 후 부가 기능(예외 처리, 트랜잭션 종료 등)이 다시 실행됩니다. 

이때 프록시 객체를 생성하는 방식에 따라 **JDK Dynamic Proxy**와 **CGLIB**가 사용됩니다.

|**방식**|**동작 원리**|**특징**|
|:---:|:---:|:---:|
|**JDK Dynamic Proxy**|인터페이스를 구현한 프록시 객체 생성|인터페이스가 필수|
|**CGLIB**|기존 클래스(Concrete Class)를 상속하여 프록시 생성|인터페이스 없이 클래스 기반으로 동작|

> Spring은 **타겟 객체가 인터페이스를 구현하고 있으면 JDK Dynamic Proxy를 그렇지 않으면 CGLIB**을 사용합니다.

--- 

## @Transactional 사용 시 주의사항

### private 메서드에는 적용되지 않음

Spring AOP는 **프록시 객체를 생성하여 동작**하기 때문에 **private** 메서드에는 트랜잭션이 적용되지 않습니다.

```java
@Transactional
private void someMethod() { // 적용 안됨
    // 코드
}
```

### final 키워드 사용 주의

Spring AOP는 **클래스를 상속하여 프록시 객체를 생성**하는 방식(CGLIB)도 사용하기 때문에 **final** 키워드가 붙은 메서드에는 적용할 수 없습니다.

```java
@Transactional
public final void someMethod() { // 적용 안됨
    // 코드
}
```

### 프록시 내부 호출 문제

Spring AOP는 **클라이언트가 프록시 객체를 호출할 때만** 동작합니다.

```java
@Service
public class MyService {
    @Transactional
    public void outerMethod() {
        innerMethod(); // 내부 호출 (AOP 적용 안됨)
    }

    @Transactional
    public void innerMethod() {
        // 실제로는 적용 안됨
    }
}
```

위 코드에서 `outerMethod()`는 트랜잭션이 적용된 상태지만 내부에서 직접 `innerMethod()`를 호출하면 트랜잭션이 적용되지 않습니다.

**해결 방법**
- **Self-invocation 방지** : 같은 객체에서 내부 호출하지 않도록 설계 변경
- **ApplicationContext 주입** : `@Autowired`로 자기 자신을 주입받아 프록시를 통해 호출
- **별도 서비스 분리** : `innerMethod()`를 다른 서비스 클래스로 분리

---

## 결론 

- Spring AOP는 **프록시 패턴**을 사용하여 런타임에 트랜잭션을 적용합니다.
- @Transactional은 **JDK Dynamic Proxy 또는 CGLIB 프록시 객체**를 통해 트랜잭션을 관리합니다.
- **private, final 메서드 및 내부 호출**에 주의해야 합니다.