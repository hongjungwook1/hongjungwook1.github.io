---
title: ORM (Hibernate) vs SQL Mapper (MyBatis) 비교
description: >-
    ORM과 SQL Mapper의 개념과 차이점을 비교하고 각각의 장점과 단점을 분석하여 어떤 상황에서 적절하게 사용할 수 있는지 설명합니다.
author: JWHONG
date: 2025-03-24 11:00:00 +0900
categories: [Spring, JPA, MyBatis]
tags: [ORM, Hibernate, MyBatis, SQL, Spring, DataBase]
pin: true
media_subpath: "/posts/20250324"
---

# ORM (Hibernate) vs SQL Mapper (MyBatis) 비교

**ORM(Object-Relational Mapping)**과 **SQL Mapper**는 데이터베이스와 애플리케이션 간의 매핑을 수행하는 두 가지 대표적인 방식입니다.  
이 두 접근 방식의 개념과 차이점을 상세히 살펴보겠습니다.

## ORM (Object-Relatinal Mapping)

ORM은 데이터베이스의 테이블을 객체(Entity)로 매핑하고 이를 조작하는 방식으로 SQL을 직접 작성하지 않고도 데이터 조작이 가능합니다.  
대표적인 ORM 프레임워크로는 **JPA(Java Persistence API), Hibernate** 등이 있습니다.

- **ORM의 핵심 개념**
    - **Persistence Context(영속성 컨텍스트)** 를 통해 **Entity** 객체를 관리
    - 데이터베이스의 테이블과 객체(Entity)를 자동으로 매핑
    - SQL이 아닌 **메서드 기반으로 데이터를 조작**
    - 객체 간 관계를 기반으로 **SQL을 자동 생성**

- **특징**
    - **객체 중심 개발** &rarr; 객체 모델을 기반으로 데이터베이스 작업 수행
    - **SQL을 직접 작성하지 않아도 자동으로 SQL 생성** &rarr; 코드 유지보수가 용이
    - **트랜잭션 관리 및 캐싱 지원** &rarr; 성능 최적화 가능
    - **Lazy Loading 및 Eager Loading 기능 제공** &rarr; 필요에 따라 데이터 로딩 방식 조절 가능

### JPA 사용 예제

```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Setter
    private String name;
}

// Repository를 이용한 데이터 조작
User user = new User();
user.setName("홍길동");
userRepository.save(user); // SQL 없이 저장 가능
```

### 장점

- SQL을 직접 작성할 필요가 없어 생산성이 향상됨
- 엔티티 간의 연관 관계를 쉽게 관리할 수 있음
- JPQL (Java Persistence Query Language) 제공으로 객체 중심의 질의가 가능
- 데이터 변경 이력(Dirty Checking) 관리 및 자동 반영 지원

### 단점

- 복잡한 SQL 튜닝이 어려움 (대용량 데이터 처리 시 성능 저하 가능)
- 학습 곡선이 높고 설정이 복잡할 수 있음
- 자동 생성된 SQL이 최적화되지 않을 수 있음

---

## SQL Mapper (MyBatis)

SQL Mapper는 SQL 쿼리를 직접 작성하고 이를 Java 객체와 매핑하는 방식입니다.  
**MyBatis가 대표적인 SQL Mapper 프레임워크입니다.**

- **SQL Mapper의 핵심 개념**
    - **Persistence Context(영속성 컨텍스트)가 존재하지 않음** &rarr; 상태 관리 기능 없음
    - **Entity 개념이 없음** &rarr; 일반 객체와 데이터베이스의 매핑만 수행
    - SQL을 직접 작성하여 **DBMS에 최적화된 쿼리 실행 가능**

- **특징**
    - **SQL 중심 개발** &rarr; SQL을 직접 작성하고 XML 또는 애노테이션을 통해 매핑
    - **Persistence Context 없음** &rarr; **JPA의 1차 캐시, 변경 감지(Dirty Checking) 등의 기능 없음**
    - **최신 MyBatis에서는 SqlSessionTemplate 대신 Mapper Interface 사용**

### MyBatis 사용 예제

#### Mapper XML 방식
```xml
<mapper namespace="com.example.UserMapper">
    <select id="findUserById" resultType="User">
        SELECT * FROM users WHERE id = #{id}
    </select>
</mapper>
```

#### Mapper Interface 방식
```java
@Mapper
public interface UserMapper {
    @Select("SELECT * FROM users WHERE id = #{id}")
    User findUserById(@Param("id") Long id);
}
```

### 장점

- SQL을 직접 작성하므로 **성능 최적화 가능**
- 복잡한 쿼리를 명확하게 제어할 수 있음 
- 다양한 DBMS에 쉽게 적용 가능
- 간단한 설정으로 빠르게 적용 가능

### 단점

- SQL을 직접 작성해야 하므로 **개발 생산성이 떨어짐**
- 데이터 변경 시 SQL 수정이 필요함 &rarr; **유지보수가 어려움**
- **JPA의 캐싱, 자동 트랜잭션 관리 등의 기능이 없음**

--- 

## ORM vs SQL Mapper 비교

|**항목**|**ORM (JPA/Hibernate)**|**SQL Mapper (MyBatis)**|
|:---:|:---:|:---:|
|**개발 방식**|객체 (Entity) 중심 개발|SQL 중심 개발|
|**SQL 작성**|자동 생성|직접 작성|
|**쿼리 최적화**|자동 최적화 (하지만 튜닝 어려움)|직접 최적화 가능|
|**연관 관계 관리**|객체 간 관계를 자동 매핑|SQL을 통한 수동 매핑|
|**성능**|캐싱 및 변경 감지 지원|SQL 최적화 가능하시만 캐싱 없음|

---

## 언제 ORM과 SQL Mapper를 사용할까?

|**상황**|**ORM (JPA/Hibernate)**|**SQL Mapper (MyBatis)**|
|:---:|:---:|:---:|
|**CURD 위주의 단순한 시스템**|✅적합|❌비효율적|
|**복잡한 조회 및 대량 데이터 처리**|❌성능 저하 가능|✅최적화 가능|
|**트랜잭션 및 연관관계 관리**|✅강력한 기능 지원|❌수동 관리 필요|
|**레거시 시스템과의 호환**|❌마이그레이션 어려움|✅기존 SQL 활용 가능|

---

## 결론 

ORM과 SQL Mapper는 각각의 장점과 단점이 있기 때문에 **프로젝트의 요구 사항에 맞게 선택하는 것이 중요합니다.**

- **ORM (JPA, Hibernate) 사용 추천**

    - 도메인 중심 설계가 필요할 때
    - 객체 지향적인 개발이 필요한 경우
    - 데이터 변경이 잦고 유지보수가 중요한 경우

- **SQL Mapper (MyBatis) 사용 추천**

    - SQL 최적화가 중요한 경우 (복잡한 쿼리 작성 필요)
    - 레거시 시스템과의 호환이 필요한 경우
    - 대량 데이터 처리 및 성능이 중요한 경우

ORM과 SQL Mapper를 혼합하여 사용하는 것도 좋은 선택입니다.  
JPA를 기본으로 사용하되 **복잡한 SQL이 필요한 경우 MyBatis를 병행하여 성능을 최적화**할 수 있습니다.