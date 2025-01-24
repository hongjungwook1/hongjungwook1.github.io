---
title: Java 개발 환경 설정 및 Spring Boot 기본 가이드
description: >-
  Java 개발의 핵심 개념(JDK, JRE, JVM)부터 Gradle과 Spring Boot 환경 설정
author: JWHONG
date: 2025-01-24 15:00:00 +0900
categories: [Java, Spring Boot, Setting, Tutorial]
tags: [Java, Spring Boot, Gradle]
pin: true
media_subpath: "/posts/20250124"
---

# Java 개발 환경 설정과 동작 원리

## JDK, JRE, 그리고 JVM의 이해

### JRE (Java Runtime Environment)

- Java 프로그램을 **"실행"**하기 위한 환경.

- JVM (Java Virtual Machine)을 포함하며 Java 바이트코드를 실행하는 역할을 수행.

- Java 프로그램을 "실행"하려면 JRE만 있으면 충분.

### JDK (Java Development Kit)

- Java 프로그램을 **"개발"**하기 위한 환경.

- JDK = JRE + 개발 도구 (예: 컴파일러 javac, 디버거).

- JDK 설치가 필요한 경우:
  - IntelliJ IDEA, Eclipse 등 IDE를 사용하여 Java 애플리케이션을 개발하려는 경우.

#### Java 프로그램 실행 절차

1. 소스 코드 작성 : .java 파일 생성.

2. 컴파일 : javac 명령어를 사용하여 바이트코드(.class) 생성.

3. 실행 : java 명령어를 사용하여 JVM이 바이트코드를 실행.

---

### Gradle 의존성 관리와 빌드 도구

#### Gradle의 개요

Java 프로젝트의 의존성 및 빌드 프로세스를 관리하는 도구.
빌드 스크립트 언어로 Groovy 또는 Kotlin 사용.

#### 주요 의존성 종류

- **implementation**: 런타임 및 빌드에 필요한 의존성.
- **compileOnly**: 컴파일 시에만 필요한 의존성.
- **testImplementation**: 테스트 시에만 필요한 의존성.

#### 예시: `build.gradle`

```java
groovy
repositories {
  mavenCentral()
}
```

```java
dependencies {
  implementation 'org.springframework.boot:spring-boot-starter-web'
  testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

---

### Spring과 Spring Boot의 차이

#### Spring Framework

- Java 기반 웹 애플리케이션 프레임워크.

- 전통적인 방식으로 WAR 패키징 지원.

- 설정이 많아 초기 진입 장벽이 높은 편.

#### Spring Boot

- Spring Framework의 확장판으로 설정을 자동화하여 개발을 간소화.

- 주요 특징
  - 내장 톰캣 서버 제공 : 별도의 WAS 설치 불필요.
  - JAR 패키징 지원으로 간편 배포 가능.
  - Auto Configuration : 복잡한 XML 설정 없이 간단한 애플리케이션 개발 가능.

---

### Spring Reactive Web과 Spring Web

#### Spring Reactive Web

- 비동기 지원 (Reactor 기반 Mono/Flux).

#### Spring Web

- Tomcat 기반으로 동기 처리.

---

### 객체 초기화와 빌더 패턴

#### 생성자 관련 어노테이션

- @NoArgsConstructor: 빈 객체를 생성하고 이후 값을 설정 (setter 사용).
- @AllArgsConstructor: 모든 필드를 한 번에 초기화.
- @RequiredArgsConstructor: 필수 필드만 초기화.

#### 빌더 패턴 (@Builder)

- 특정 필드만 선택적으로 초기화 가능.
- 생성자와 달리 설정 순서에 구애받지 않음.

```java
Default 값 설정 가능:
@Builder
public class Example {
@Builder.Default
private String field = "default value";
}
```

#### 빌더 패턴의 장점

- 필드 선택적 초기화.
- 생성 시점과 설정 시점 분리.
