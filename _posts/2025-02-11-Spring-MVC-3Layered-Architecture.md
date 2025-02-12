---
title: Spring Boot 특장점 및 MVC , 3LayeredArchitecture
description: >-
  Spring Boot 특장점 및 MVC , 3LayeredArchitecture에 대해 정리하였습니다.
author: JWHONG
date: 2025-02-11 11:00:00 +0900
categories: [Spring, Spring Boot, MVC, 3LayeredArchitecture, Backend]
tags: [Spring, Spring Boot, MVC, 3LayeredArchitecture, Backend]
pin: true
media_subpath: "/posts/20250211"
---

# Spring Boot 의의 : 의존성 관리 + 자동 설정

Spring Boot는 기존 Spring Framework의 불편함을 해소하고 보다 쉽게 개발할 수 있도록 돕는 프레임워크입니다.

## Spring Boot의 주요 특징

### 의존성 관리

`Spring Boot`는 여러 라이브러리의 버전을 신경 쓰지 않고 최적의 조합으로 구성된 최상위 패키지를 제공하여 버전 충돌을 방지합니다.

**대표적인 Starter 패키지**

- `spring-boot-starter-web` : Tomcat 내장 및 웹 애플리케이션 개발을 위한 모든 의존성 포함.
- `spring-boot-starter-security` : Spring Security 및 인증, 인가 관련 라이브러리 포함.
- `spring-boot-starter-jdbc` : HikariCP 커넥션 풀을 활용한 JDBC 기능 제공.
- `spring-boot-starter-data-jpa` : Spring Data JPA 및 Hibernate 포함.

### 자동 설정 (@EnableAutoConfiguration)

Spring Boot는 다양한 라이브러리를 포함할 뿐만 아니라 이를 자동으로 설정하여 개발자가 설정 파일을 직접 작성할 필요가 없습니다.

- `@SpringBootApplication` : Spring Boot 애플리케이션의 기본 설정을 포함하는 어노테이션.
- `@SpringBootConfiguration` : `@Configuration` 을 포함하며 추가적인 `@Bean` 등록 가능.
- `@EnableAutoConfiguration` : 의존성에 따라 사전 정의된 설정을 자동으로 적용.
- `@ComponentScan` : `@Controller` 등의 어노테이션이 붙은 클래스를 자동으로 스캔하여 Bean으로 등록.

---

## Spring과 Spring Boot의 차이

### Spring

- `WAR(Web Application Archive) 방식` : Servlet Container(WAS)에 배포되는 웹 애플리케이션 포맷.
- `외장 톰캣 필요` : 별도로 Tomcat 같은 웹 컨테이너를 실행한 후 WAR 파일을 배포해야 함.
- `Tomcat을 미리 띄워서 WAR 파일을 배포 가능`
  - 하나의 Tomcat 서버에서 여러 개의 WAR 파일을 실행하는 것도 가능함.

### Spring Boot

- `JAR(Java Archive) 방식` : JRE에서 바로 실행 가능한 Java 애플리케이션 포맷.
- `내장 톰캣 포함` : 별도로 톰캣을 실행할 필요 없이 애플리케이션을 실행하면 자동으로 내장 톰캣이 실행됨.
- `JAR 실행 방식` : `java -jar myapp.jar` 명령어만으로 실행 가능.
  - WAR 방식과 달리 하나의 Tomcat에서 여러 애플리케이션을 실행할 수 없음 (각 JAR은 독립적).

---

## Spring MVC 아키텍처 패턴

Spring은 크게 두 가지 아키텍처 패턴을 기반으로 동작합니다

**1. MVC 아키텍처 패턴**

**2. 3-Layered 아키텍처 패턴**

### MVC 아키텍처 패턴 개요

MVC(Model-View-Controller) 패턴은 클라이언트 요청을 처리하고 적절한 응답을 반환하기 위한 아키텍처입니다.

- Front Controller (DispatcherServlet)
  - 모든 요청을 중앙에서 관리하며 적절한 Controller를 찾아서 실행하는 역할.
  - Controller는 View 이름과 Model 데이터를 반환.
  - 반환하는 View가 HTML인 경우 → `ViewResolver`가 적절한 템플릿을 찾아 렌더링.
  - 반환하는 데이터가 JSON인 경우 → `HttpMessageConverter`가 JSON으로 변환하여 응답.

### Spring MVC 아키텍처 흐름

1. 클라이언트가 요청을 보냄.
2. Tomcat이 요청을 받고 정적 페이지가 존재하는지 확인.
3. 정적 페이지가 없다면 Front Controller(DispatcherServlet)이 요청을 처리.
4. Front Controller(DispatcherServlet)이 적절한 Controller를 찾아서 실행.
5. Controller는 데이터를 가공한 후 Model과 View를 반환.
6. ViewResolver가 적절한 View 템플릿을 찾아서 Model 데이터를 적용 후 최종 View를 생성.

---

### 3-Layered 아키텍처 패턴

Spring 애플리케이션은 `관심사의 분리(Separation of Concern)` 원칙을 따르기 위해 3-Layered 아키텍처 패턴을 적용합니다.

- `Presentation Layer (@Controller)` : 클라이언트 요청을 받고 응답을 반환하는 계층.
- `Business Layer (@Service)` : 비즈니스 로직을 처리하는 계층.
- `Data Access Layer (@Repository)` : 데이터베이스 CRUD 작업을 담당하는 계층.

---

### 결론 : Spring Boot는 개발을 더욱 간편하게 만든다!

Spring Boot는 개발자가 복잡한 설정을 신경 쓰지 않고 비즈니스 로직에만 집중할 수 있도록 돕는 강력한 프레임워크입니다.
기존 Spring과 비교했을 때 의존성 관리 및 자동 설정 기능을 제공하여 더욱 빠르게 개발할 수 있습니다. 또한 MVC 및 3 계층 아키텍처를 활용하여 유지보수성과 확장성을 높일 수 있습니다.
