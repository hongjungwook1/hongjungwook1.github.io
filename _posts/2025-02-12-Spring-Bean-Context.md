---
title: Spring Bean 원리 - ApplicationContext와 Bean 등록 및 주입 방법
description: >-
  Spring Bean 원리 - ApplicationContext와 Bean 등록 및 주입 방법
author: JWHONG
date: 2025-02-12 11:00:00 +0900
categories: [Spring, Spring Boot, Bean, Backend]
tags: [Spring, Spring Boot, Bean, Backend]
pin: true
media_subpath: "/posts/20250212"
---

## ApplicationContext : Spring Bean 관리

### ApplicationContext 란?

**ApplicationContext**란 개발자가 직접 객체를 생성하는 것이 아니라 Spring이 대신 객체를 생성하여 관리하는 컨테이너를 의미합니다.

### Spring Bean 이란?

- **Spring Bean**이란 개발자가 직접 **new** 키워드를 사용하여 객체를 생성하는 것이 아니라 Spring이 대신 객체를 생성하고 주입하는 단위를 의미합니다.
- **Singleton(싱글톤) 패턴**은 Spring의 **ApplicationContext**가 객체를 싱글톤으로 생성하고 관리하는 방식입니다.
- **주의** : Spring Bean은 JavaBeans와 다릅니다. (JavaBeans는 getter/setter를 갖춘 일반적인 Java 클래스입니다.)

### Spring Bean은 두 가지만 기억하시면 됩니다.

1. **Bean 등록** : Spring Container 내에서 싱글톤 객체로 등록됩니다. (new 키워드로 미리 인스턴스화되어 있습니다.)
2. **Bean 사용** : Spring Container에 등록된 싱글톤 객체를 가져와 주입하여 사용합니다.

### Spring Container = ApplicationContext

- **Spring Container**는 **ApplicationContext**라고도 불리며 다음과 같이 두 가지로 나뉩니다.
  - **Servlet WebApplicationContext** : `@Controller` 등과 관련된 웹 계층
  - **Root WebApplicationContext** : `@Service`, `@Repository` 등 비즈니스 로직 및 데이터 계층

---

## Spring Bean은 왜 싱글톤으로 관리 되는 이유

### 싱글톤으로 관리하는 이유

- **대규모 엔터프라이즈 서버 환경**

  - 하나의 서버에서 **초당 수십~수백 번**의 요청을 처리해야 합니다.
  - 요청마다 객체를 새로 생성하면 **서버 부하가 증가**하여 성능이 저하될 수 있습니다.

- **Spring MVC 구조**

  - 하나의 요청이 데이터 액세스, 서비스 로직 등 **여러 객체를 거치며 처리됩니다.**
  - 각 요청마다 새로운 객체를 생성하면 **메모리 낭비 및 성능 저하**가 발생할 수 있습니다.

### Java 싱글톤 패턴의 한계

- private 생성자로 인해 **상속이 불가능하고** , **테스트가 불편합니다.**
- 클래스 로더에 따라 싱글톤 유지가 어려울 수 있습니다. (예: **1개의 애플리케이션이 여러 개의 클래스 로더를 가질 경우 싱글톤 유지가 불가능할 수 있습니다.**)

### Spring의 해결책 : 싱글톤 레지스트리

- Spring은 자체적으로 **싱글톤 레지스트리**를 제공하여 객체 생성, 할당 및 관리를 수행합니다.
- **IoC(Inversion of Control) 기반 DI(Dependency Injection) 컨테이너**를 사용하여 객체 관리의 책임을 Spring에게 위임합니다.

---

## 제어 역전(IoC, Inversion of Control) : Spring이 객체 생성 및 주입

### IoC란 ?

객체 생성과 의존 관계 주입을 **개발자가 직접 하는 것이 아니라** Spring과 같은 **컨테이너가 대신 수행하는 개념**을 의미합니다.

---

## Service Locator vs. DI (의존성 주입)

### Service Locator 방식 (개발자가 직접 주입)

- Locator가 직접 Bean을 가져와 사용합니다.
- 의존성을 파악하기 어렵습니다. (런타임에서 동적으로 Bean을 찾습니다.)

![Image](https://github.com/user-attachments/assets/3d1d91b6-d4c6-4e92-953f-bae5a37df0a3)
이미지 출처 : ASAC 07 강의자료

### DI (의존성 주입)

- Spring Container가 대신 Bean을 주입합니다.
- 생성자 기반 주입이 가능하여 의존성을 명확하게 파악할 수 있습니다.

![Image](https://github.com/user-attachments/assets/444dbb17-dbdb-4b76-83bd-9b3e0358026f)
이미지 출처 : ASAC 07 강의자료

---

## Spring Bean 등록 방법

### Bean 등록 방법

- **@ComponentScan + @Component**

  - 특정 패키지를 지정하여 **해당 패키지 내의 @Component가 붙은 클래스를 Bean으로 자동 등록합니다.**
  - **@Controller**, **@Service**, **@Repository**는 **@Component**를 포함하는 어노테이션입니다.

- **@Configuration + @Bean**
  - **직접 Bean을 등록하는 방법** (Java Config 방식)
  - XML 기반 설정 방식에서 발전된 형태입니다.

---

## Bean 주입 방법 (DI 방식)

Spring에서 빈(Bean)을 주입하는 방법에는 여러 가지가 있습니다.  
이 글에서는 대표적인 **생성자 주입, 필드 주입, 수정자 주입** 방법을 살펴보겠습니다.

### 생성자 주입 (Constructor Injection)

> **Spring 공식 추천 방식!**

#### 생성자 주입의 장점

- **순환 참조 방지**
  - 순환 참조가 발생할 경우 런타임이 아닌 **컴파일 시점**에 감지할 수 있음.
- **불변성 보장**
  - **final** 키워드를 적용할 수 있어 객체가 한 번 주입되면 변경되지 않음을 보장할 수 있음.

### 코드 예제

```java
@Controller
@RequestMapping("/users")
@RequiredArgsConstructor
@FieldDefaults(makeFinal = true, level = AccessLevel.PRIVATE)
public class UserController {
  UserServiceInterface userService;
}
```

> **@RequiredArgsConstructor**를 사용하면 **final** 필드에 대한 생성자 자동 생성이 가능합니다.

---

### 필드 주입 (Property Injection)

#### 필드 주입의 단점

- **private** 접근자로 선언된 필드에 **바로 주입**되므로 **테스트하기 어려움**
- **순환 참조 문제를 조기에 발견하기 어려움**
- **불변성이 보장되지 않음**

### 코드 예제

```java
@Controller
@RequestMapping("/users")
public class UserController {
    @Autowired
    private UserServiceInterface userService;
}
```

> **@Autowired**를 필드에 직접 적용하는 방식은 **지양하는 것이 좋습니다**. 생성자 주입을 사용하는 것이 더 안전하고 유지보수하기 쉽습니다.

---

### 수정자 주입 (Setter Injection)

> 필드 주입과 유사하지만 **Setter 메서드를 통해 주입하는 방식**입니다.

#### 수정자 주입의 특징

- **선택적 주입이 가능** (필수가 아닌 경우 사용할 수 있음)
- **객체가 변경될 가능성이 높아짐** (불변성이 보장되지 않음)

### 코드 예제

```java
@Controller
@RequestMapping("/users")
public class UserController {
    private UserServiceInterface userService;

    @Autowired
    public void setUserService(UserServiceInterface userService) {
        this.userService = userService;
    }
}
```
