---
title: Spring Boot와 JDBC부터 JPA까지 쉽게 배우기
description: >-
  Spring Boot에서 JDBC와 JPA를 활용한 데이터베이스 연동 방법을 쉽게 배워봅니다.
author: JWHONG
date: 2025-03-18 11:00:00 +0900
categories: [Spring , Spring Boot , JDBC , JPA , Database]
tags: [Spring , Spring Boot , JDBC ,JPA , Database]
pin: true
media_subpath: "/posts/20250318"
---


## JDBC와 JPA란?

### JDBC란? 데이터베이스와 직접 연결하는 방식

**JDBC (Java Database Connectivity)**는 **자바 애플리케이션에서 데이터베이스에 직접 연결하여 SQL을 실행할 수 있도록 하는 표준 API입니다**입니다.  
쉽게 말해 **자바와 MySQL 같은 관계형 데이터베이스(RDBMS) 사이의 다리 역할**을 합니다.

**JDBC의 주요 기능**
  - **Connection** : 데이터베이스 연결
  - **Statement** : SQL 문 실행
  - **ResultSet** : 쿼리 결과 조회
  - **PreparedStatement** : SQL Injection 방지를 위한 안전한 쿼리 실행

**기본적인 JDBC 사용 예제**

```java
Connection connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/example", "root", "1234");
Statement statement = connection.createStatement();
ResultSet resultSet = statement.executeQuery("SELECT * FROM user");

while (resultSet.next()) {
    System.out.println(resultSet.getString("name"));
}
```
이 방식은 SQL을 직접 작성해야 하며 데이터베이스와의 물리적 연결을 매번 생성/해제해야 하는 부담이 있습니다.  
이를 보완하기 위해 **Connection Pool**과 같은 기법이 활용됩니다.

---

### JPA란? 객체 지향 방식의 데이터 관리

**JPA (Java Persistence API)**는 **객체와 데이터베이스 간의 매핑(ORM, Object-Relational Mapping)을 자동으로 처리하는 자바 표준 API**입니다.

JDBC를 사용할 경우 SQL을 직접 작성해야 하지만 JPA는 **객체를 저장하면 자동으로 SQL을 생성하여 실행합니다.**  

**JPA를 사용한 엔티티 예제**

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    private String name;
}
```
이렇게 클래스를 만들면 JPA가 자동으로 user 테이블과 매핑하여 데이터베이스에 데이터를 저장하고 관리합니다.

---

## Spring Boot에서 MySQL 설정하기

먼저 `build.gradle` 파일에서 MySQL 드라이버와 JPA 관련 의존성을 추가합니다.

```xml
dependencies {
    runtimeOnly 'mysql:mysql-connector-java' // MySQL 드라이버
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa' // JPA
}
```

그리고 `application.yml`에서 MySQL 접속 정보를 설정합니다.

```yml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/example?useSSL=false
    username: root
    password: 1234
  jpa:
    generate-ddl: true
    hibernate:
      ddl-auto: update
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQLDialect
    show-sql: true
```
> 위 설정을 통해 MySQL 연결(JDBC Driver), Hibernate 설정(JPA), SQL 로그 출력을 활성화할 수 있습니다.

---

## JDBC와 Connection Pool 이해하기

### JDBC Driver를 통한 MySQL 접속 (물리적 접속)

기본적으로 Java에서는 `DriverManager`를 사용하여 직접 데이터베이스에 연결할 수 있습니다.

```java
Connection connection = DriverManager.getConnection(
    "jdbc:mysql://localhost:3306/example?useSSL=false",
    "root", 
    "1234"
);
Statement statement = connection.createStatement();
ResultSet resultSet = statement.executeQuery("SELECT * FROM user WHERE id = " + userId);

if (resultSet.next()) {
    return new User(
        resultSet.getInt("id"),
        resultSet.getString("name"),
        resultSet.getInt("age"),
        resultSet.getString("job"),
        resultSet.getString("specialty"),
        resultSet.getTimestamp("created_at").toLocalDateTime()
    );
}
```
**이 방식은 매번 새로운 커넥션을 생성하고 닫아야 하기 때문에 성능적으로 비효율적입니다.**

---

### Connection Pool (논리적 접속) 적용

JDBC의 단점을 보완하기 위해 Spring Boot에서는 HikariCP를 기본적으로 사용합니다.  
HikariCP는 빠르고 가벼우며 성능이 뛰어납니다.

```java
HikariConfig config = new HikariConfig();
config.setJdbcUrl("jdbc:mysql://localhost:3306/example?useSSL=false");
config.setUsername("root");
config.setPassword("1234");
config.setDriverClassName("com.mysql.cj.jdbc.Driver");
config.setMaximumPoolSize(20);
config.setMinimumIdle(10);

HikariDataSource hikariDataSource = new HikariDataSource(config);

// Connection Pool에서 커넥션 가져오기
Connection connection = hikariDataSource.getConnection();
```

**Connection Pool 설정**
  - **setConnectionTimeout(30000)** &rarr; 30초 동안 커넥션을 얻지 못하면 예외 발생
  - **setMinimumIdle(10)** &rarr; 최소 10개의 커넥션을 유지
  - **setMaximumPoolSize(20)** &rarr; 최대 20개의 커넥션 사용

**WAS 스레드 수보다 DB 커넥션 개수가 크면 비효율적**이므로 적절한 크기를 설정해야 합니다.

---

## SQL Injection 방지를 위한 PreparedStatement

JDBC에서 쿼리를 직접 문자열로 작성할 경우 **SQL Injection 공격에 취약**합니다.

```java
String name = "' OR 1=1 --";
String query = "SELECT * FROM user WHERE name = '" + name + "'"; // 취약한 코드
```
위 코드에서 `name`에 `"' OR 1=1 --"`을 입력하면 **모든 사용자 정보가 조회되는 보안 취약점**이 발생합니다.

이를 방지하기 위해 **PreparedStatement**를 사용합니다.

```java
String query = "SELECT * FROM user WHERE name = ?";
PreparedStatement preparedStatement = connection.prepareStatement(query);
preparedStatement.setString(1, "홍길동");
```
PreparedStatement는 **내부적으로 값의 이스케이핑을 자동 처리**하여 SQL Injection을 방지합니다.

---

## 결론

Spring Boot에서 MySQL을 사용하려면 **JDBC, JPA, Connection Pool** 등에 대한 개념을 이해하는 것이 중요합니다.

 - **JDBC**는 SQL을 직접 실행할 수 있지만 코드가 길어지고 유지보수가 어렵습니다.
 - **JPA**는 객체를 기반으로 데이터를 관리하며 코드가 간결해지고 유지보수가 편리해집니다.
 - **Connection Pool**(HikariCP 등)을 활용하면 성능을 최적화할 수 있습니다.
 - **SQL Injection 방지**를 위해 PreparedStatement를 사용하는 것이 필수입니다.