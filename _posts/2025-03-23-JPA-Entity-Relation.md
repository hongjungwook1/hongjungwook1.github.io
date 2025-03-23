---
title: JPA에서 Entity 객체와 연관관계 매핑
description: >-
    JPA에서 Entity 객체 간의 연관관계를 매핑하는 방법과 주의할 점 정리
author: JWHONG
date: 2025-03-23 11:00:00 +0900
categories: [JPA, Spring]
tags: [JPA, Entity, 연관관계, 매핑, Spring]
pin: true
media_subpath: "/posts/20250323"
---

# JPA에서 Entity 객체와 연관관계 매핑

JPA를 사용하면 데이터베이스 테이블 간의 관계를 객체 모델로 매핑하여 **JOIN 없이 객체 탐색을 수행**할 수 있습니다.  
이를 위해 **Entity 객체의 연관관계를 명확하게 정의하고 어노테이션을 활용하여 설정해야 합니다.**

## 연관관계 매핑 시 고려해야 할 사항

Entity 객체의 연관관계를 정의할 때 고려해야 할 요소는 크게 다음 세 가지입니다.

- **다중성** : N:M 관계 정의 → `@OneToMany` , `@ManyToOne` 등
- **방향** : 단방향 또는 양방향 관계 설정 → 어느 객체에서 참조를 가질 것인가?
- **연관관계의 주인** : 외래 키(FK) 관리 주체 → 연관관계 주인은 `@JoinColumn`을 사용하여 FK를 관리

---

## 다중성 : 연관관계 N:M 정의

객체 간 연관관계는 다음과 같은 어노테이션으로 정의됩니다.

|**관계 유형**|**어노테이션**|
|:---:|:---:|
|**1:N**|@OneToMany, @ManyToOne|
|**1:1**|@OneToOne |
|**N:M**|@ManyToMany @ManyToMany (보통 중간 테이블을 이용하여 1:N + N:1로 변환)|

### 방향 : 단반향 vs 양방향

**데이터베이스 테이블은 FK 하나로 양방향 조인이 가능하지만 객체는 참조 필드가 필요하므로 명시적으로 방향을 설정해야 합니다.**

- **양방향 관계** : 두 객체가 서로 참조를 가지는 경우 (`@OneToMany`, `@ManyToOne`을 각각 설정)
- **단방향 관계** : 한쪽 객체에서만 참조를 가지는 경우 (`@ManyToOne` 또는 `@OneToOne`만 설정)

> **주의** : 양방향 매핑 시 toString(), Lombok, JSON 생성 라이브러리에서 무한 루프가 발생할 수 있으므로 DTO 사용을 권장 합니다.


### 예제 : 양방향 관계 매핑

**Team (연관관계 주인이 아님)**
```java
@Entity
public class Team {
    @Id
    @GeneratedValue
    private Long id;
    private String name;
    
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
}
```

**Member (연관관계 주인)**
```java
@Entity
public class Member {
    @Id
    @GeneratedValue
    private Long id;
    private String name;
    
    @ManyToOne
    @JoinColumn(name = "team_id")
    private Team team;
}
```

---

## 연관관계의 주인과 @JoinColumn

### 연관관계 주인

- **외래 키(FK)를 갖는 테이블이 연관관계의 주인이 됩니다.**
- **연관관계 주인 객체에서만 @JoinColumn을 설정하며 연관된 객체의 변경이 가능합니다.**

> 연관관계 주인 결정 규칙
- `1:N` 또는 `N:1` : **N 쪽이 연관관계 주인**
- `1:1` : **주 테이블이 연관관계 주인**
- `M:N` : 중간 테이블을 추가하여 `1:N` 및 `N:1`로 변환 &rarr; M과 N이 각각 연관관계 주인


### 예제 : ID 기반 Entity 저장

```java
@Entity
public class Message {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;
    
    @Column(name = "user_id")
    private Integer userId;

    @ManyToOne
    @JoinColumn(name = "user_id", insertable = false, updatable = false)
    private User user;
}
```

**ID만을 기반으로 Entity를 저장할 경우** `insertable = false, updatable = false` 속성을 사용하여 JPA가 userId 컬럼을 직접 조작하지 않도록 설정합니다.

---

## 단방향 관계의 장점

- **불필요한 양방향 관계를 최소화**하여 코드 가독성을 높이고 성능을 최적화할 수 있습니다.
- `User` 엔티티가 여러 테이블과 연관될 경우 **모든 관계를 양방향으로 설정하면 복잡성이 증가**하고 불필요한 참조가 많아집니다.
- **따라서 기본적으로 단방향으로 설정하고 필요할 때만 양방향을 추가하는 것이 권장됩니다.**

### 예제 : 단반향향 관계 매핑

**Team (연관관계 주인이 아님)**
```java
@Entity
public class Team {
    @Id
    @GeneratedValue
    private Long id;
    private String name;
}
```

**Member (연관관계 주인)**
```java
@Entity
public class Member {
    @Id
    @GeneratedValue
    private Long id;
    private String name;
    
    @ManyToOne
    @JoinColumn(name = "team_id")
    private Team team;
}
```

---

## mappedBy 속성: 연관관계 주인이 아닌 객체

**양방향 관계에서 연관관계 주인이 아닌 객체는 데이터베이스에 영향을 주지 않고 읽기 전용**입니다.

- **연관관계 주인** : `@JoinColumn`을 설정한 객체 &rarr; FK 값을 직접 조작 가능
- **연관관계 주인이 아닌 객체** : `mappedBy`를 설정한 객체 &rarr; 단순 조회만 가능

> mappedBy 값은 연관관계 주인 객체의 필드명과 일치해야 합니다.

### 예제 : mappedBy 설정정

```java
@Entity
public class Team {
    @Id
    @GeneratedValue
    private Long id;
    private String name;
    
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
}
```

```java
@Entity
public class Member {
    @Id
    @GeneratedValue
    private Long id;
    private String name;
    
    @ManyToOne
    @JoinColumn(name = "team_id")
    private Team team;
}
```

`mappedBy = "team"`은 **Member 엔티티의 team 필드를 통해 연관 관계가 설정됨을 의미**합니다.  
**Team 객체에서 Member 객체를 수정할 수 없고 조회만 가능합니다.**

---

## 결론

JPA에서 **연관관계를 설정할 때는 다중성, 방향, 연관관계의 주인을 명확하게 설정하는 것이 중요**합니다.  
또한 **기본적으로 단방향 매핑을 사용하고 필요한 경우에만 양방향을 추가하는 것이 좋습니다.**