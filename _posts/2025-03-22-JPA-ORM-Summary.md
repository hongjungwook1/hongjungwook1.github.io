---
title: JPA와 ORM 개념 정리
description: >-
    JDBC의 한계를 극복하는 ORM 기술, JPA의 동작 방식과 영속성 컨텍스트, JPQL, Hibernate까지 정리
author: JWHONG
date: 2025-03-22 11:00:00 +0900
categories: [Java, Spring, JPA, ORM]
tags: [Java, Spring, JPA, ORM , Hibernate]
pin: true
media_subpath: "/posts/20250322"
---

# JPA와 ORM 개념 정리

## JDBC 사용의 한계

기본적인 JDBC API 또는 JDBC Template을 사용할 경우 개발자가 직접 관리해야 하는 요소들이 많습니다.

- **Connection 객체** : 트랜잭션을 직접 관리해야 함
- **Statement 객체** : SQL 쿼리를 직접 작성해야 함
- **ResultSet 객체** : 조회 결과를 객체로 변환해야 함 (RowMapper 필요)

이러한 불편함을 해결하기 위해 **ORM(Object-Relational Mapping)** 기술이 등장했습니다.

---

## ORM(Object-Relational Mapping)이란?

ORM은 **객체(Object)와 관계형 데이터베이스(Relational Database)를 자동으로 매핑하는 기술입니다.**  
Java의 엔티티 객체(Entity)를 통해 DB 데이터를 조작할 수 있도록 도와줍니다.

### ORM의 기본 원리
- **객체(Entity) 조작 &rarr; DB에 반영**
    - SQL을 직접 작성하는 것이 아니라 **엔티티 객체를 다루는 것만으로도 DB가 변경됨**
- **트랜잭션 관리 자동화**
    - 트랜잭션이 commit() 되는 시점에 SQL이 실행됨
- **JDBC API를 직접 사용하지 않고 추상화된 방식으로 데이터 접근 가능**

이러한 ORM 기술의 대표적인 표준 명세가 **JPA(Java Persistence API)** 입니다.

---

## JPA (Java Persistence API)

JPA는 Java의 ORM 기술을 정의한 **표준 명세**입니다.  
JPA자체는 구현체가 아니며 이를 구현한 대표적인 라이브러리가 **Hibernate**입니다.

### JPA의 핵심 개념

### 영속성 컨텍스트 (Persistence Context)

JPA에서 가장 중요한 개념으로 **엔티티 객체를 관리하는 일종의 캐시(버퍼) 역할을 수행**합니다.

- 엔티티 객체를 영속성 컨텍스트에 보관하면 **트랜잭션이 종료될 때까지 관리됨**
- **1차 캐시 역할**을 수행하여 같은 데이터를 여러 번 조회해도 DB 쿼리를 재실행하지 않음
- 트랜잭션이 commit될 때 **변경된 엔티티를 자동으로 반영 (변경 감지, Dirty Checking)**


```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
}
```
```java
// JPA의 EntityManager 사용 예시
User user = em.find(User.class, 1L); // DB 조회 후 1차 캐시에 저장됨
User user2 = em.find(User.class, 1L); // 1차 캐시에서 조회, 쿼리 실행 X

user.setName("Updated Name"); // 변경 감지
em.getTransaction().commit(); // 변경 사항이 자동으로 DB 반영됨
```

### JPQL(Java Persistence Query Language)

JPQL은 **JPA에서 제공하는 객체지향 쿼리 언어**로 엔티티 객체를 대상으로 데이터 조회 및 조작을 수행합니다.

- **SQL이 데이터베이스의 테이블을 대상으로 쿼리하는 반면, JPQL은 엔티티 객체를 대상으로 실행됨**
- 특정 **DBMS에 의존하지 않으며** 실행 시점에 각 DBMS에 맞는 **SQL로 변환**되어 적용됨
- DBMS별로 다른 SQL 문법을 해결하기 위해 **Dialect(방언) 클래스**를 활용하여 SQL 변환을 처리
- ANSI SQL을 기반으로 하고 있으며, DBMS마다 제공하는 고유 SQL 기능(PL/SQL, T-SQL 등)은 Dialect를 통해 지원

![Image](https://github.com/user-attachments/assets/11f6358b-63aa-4ac7-95df-5d0db7fb6acd)
이미지 출처 : ASAC 07 강의자료

### JPA의 주요 기능

|**기능**|**설명**|
|:---:|:---:|
|**persist()**|엔티티 저장 (INSERT)|
|**find()**|엔티티 조회 (SELECT)|
|**remove()**|엔티티 삭제 (DELETE)|
|**merge()**|준영속 상태 엔티티를 영속 상태로 변경 (UPDATE)|
|**flush()**|변경 사항을 DB에 반영|
|**clear()**|영속성 컨텍스트 초기화|
|**close()**|영속성 컨텍스트 종료|

---

## JPA의 트랜잭션과 OSIV (Open Session In View) 

### 트랜잭션 관리
JPA에서 데이터를 변경하는 모든 작업은 반드시 **트랜잭션 내에서 수행**되어야 합니다.

```java
EntityTransaction tx = em.getTransaction();
tx.begin(); // 트랜잭션 시작

User user = new User();
user.setName("John Doe");
em.persist(user); // INSERT 실행 준비

tx.commit(); // 트랜잭션 종료 (INSERT 실행)
```

### OSIV(Open Session In View)란?

- OSIV가 **활성화(ON)** 되어 있으면 **View에서 데이터 조회 시에도 영속성 컨텍스트가 유지됨**
- OSIV가 **비활성화(OFF)** 되어 있으면 **트랜잭션 종료 시 영속성 컨텍스트가 닫힘**
- OSIV를 활성화하면 편리하지만 **DB 커넥션이 오래 유지될 위험이 있음** (실시간 트래픽이 많은 경우 비권장)

---

## JPA의 구현체 Hibernate

JPA는 인터페이스(표준 명세)이므로 실제로는 구현체가 필요합니다.  
Hibernate는 가장 널리 사용되는 JPA 구현체입니다.

### Hibernate의 특징
- **Lazy Loading (지연 로딩)** : 필요한 시점에만 쿼리를 실행하여 성능 최적화
- **Dirty Checking (변경 감지)** : 엔티티 객체의 변경 사항을 자동으로 감지하여 반영
- **HQL (Hibernate Query Language)** : JPQL과 유사하지만 Hibernate에 특화된 기능 포함
- **2차 캐시 지원** : 애플리케이션 종료 전까지 데이터를 캐싱하여 DB 부하 감소

---

## 정리: JPA의 장점

1. **생산성 증가** : SQL 작성 없이 엔티티 객체만으로 DB 조작 가능
2. **유지보수 용이** : 객체 지향적인 코드로 가독성 향상
3. **DBMS 독립성** : JPQL 사용으로 특정 DBMS에 종속되지 않음
4. **성능 최적화** : 1차 캐시, 지연 로딩, 변경 감지, 트랜잭션 자동 관리 등 제공

JPA를 잘 활용하면 객체 중심으로 개발하면서도 SQL을 능숙하게 다루는 것과 같은 효과를 얻을 수 있습니다.