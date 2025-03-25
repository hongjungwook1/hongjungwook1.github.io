---
title: JPA CascadeType과 orphanRemoval
description: >-
    JPA에서 CascadeType과 orphanRemoval을 활용하여 연관관계에서 부모-자식 객체의 생명주기를 관리하는 방법 정리
author: JWHONG
date: 2025-03-23 15:00:00 +0900
categories: [JPA, Spring]
tags: [JPA, Cascade, orphanRemoval, Entity, Spring]
pin: true
media_subpath: "/posts/20250323"
---

# JPA에서 CascadeType과 orphanRemoval 

JPA에서는 부모-자식 관계의 엔티티를 매핑할 때 **CascadeType**과 **orphanRemoval**을 활용하여 생명주기를 관리할 수 있습니다.  
이를 통해 **부모 엔티티가 변경될 때 자식 엔티티에 대한 자동 처리 여부를 결정**할 수 있습니다.

---

## CascadeType이란?

**CascadeType**은 부모 엔티티의 작업이 자식 엔티티에도 영향을 미칠지를 결정하는 옵션입니다.  
부모 엔티티를 `persist()`하면 연관된 자식 엔티티도 자동으로 `persist()`됩니다.

**주요 CascadeType 옵션**

|**옵션**|**설명**|
|:---:|:---:|
|**CascadeType.PERSIST**|부모 엔티티가 저장될 때 자식 엔티티도 함께 저장됨|
|**CascadeType.MERGE**|부모 엔티티가 병합(merge)될 때 자식 엔티티도 병합됨|
|**CascadeType.REMOVE**|부모 엔티티가 삭제될 때 자식 엔티티도 함께 삭제됨|
|**CascadeType.REFRESH**|부모 엔티티가 새로고침(refresh)될 때 자식 엔티티도 새로고침됨|
|**CascadeType.ALL**|위의 모든 Cascade 옵션을 포함|

**CascadeType 적용**
```java
@Entity
public class User {
    @Id @GeneratedValue
    private Long id;
    private String name;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL)
    private List<Message> messages = new ArrayList<>();
}

@Entity
public class Message {
    @Id @GeneratedValue
    private Long id;
    private String content;
    
    @ManyToOne
    @JoinColumn(name = "user_id")
    private User user;
}
```

위 코드에서 `CascadeType.ALL`을 설정했기 때문에 `User`를 저장하면 연관된 `Message` 객체도 자동으로 저장됩니다.

---

## orphanRemoval이란?

`orphanRemoval = true`는 부모와의 연관관계가 끊어진 **고아 객체(orphan)를 자동으로 삭제하는 옵션입니다.**  
이는 `CascadeType.REMOVE`와는 다른 동작을 수행합니다.

### CascadeType.REMOVE vs orphanRemoval


|**옵션**|**설명**|
|:---:|:---:|
|**CascadeType.REMOVE**|부모 엔티티가 삭제될 때 자식 엔티티도 함께 삭제됨|
|**orphanRemoval = true**|부모와의 연관관계가 제거되면 자식 엔티티도 삭제됨|

**orphanRmoval 적용**
```java
@Entity
public class User {
    @Id @GeneratedValue
    private Long id;
    private String name;
    
    @OneToMany(mappedBy = "user", orphanRemoval = true)
    private List<Message> messages = new ArrayList<>();
}
```

위 코드에서 `orphanRemoval = true`를 설정하면 `user.getMessages().remove(0);`을 실행하는 순간 해당 메시지는 자동으로 삭제됩니다.

---

## CascadeType.REMOVE vs orphanRemoval 주의점

- **@OneToMany 관계에서의 문제**
    - `CascadeType.REMOVE`는 부모 엔티티가 삭제될 때만 자식 엔티티가 삭제됩니다.
    - `orphanRemoval = true`를 설정하면 **부모 엔티티가 삭제되지 않더라도 연관관계가 제거되면 자식 엔티티가 삭제**됩니다.

- **다수의 부모 엔티티를 가진 자식 엔티티**

    - 예를 들어 `Member`가 `Team`, `Club`, `School`에 속하는 경우 `Team`이 `Member`를 삭제하면 `Club`, `School`에서도 `Member`가 삭제될 수 있음
    - **이러한 경우 orphanRemoval = true를 사용하면 안 됨**

---

## CascadeType과 orphanRemoval 함께 사용하기

**CascadeType.REMOVE + orphanRemoval 적용**
```java
@Entity
public class User {
    @Id @GeneratedValue
    private Long id;
    private String name;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.REMOVE, orphanRemoval = true)
    private List<Message> messages = new ArrayList<>();
}
```

### 주의할 점 

- **부모가 삭제되면 자식도 삭제됨 (CascadeType.REMOVE)**
- **부모와 연관관계가 끊긴 자식도 삭제됨 (orphanRemoval = true)**
- **ManyToOne 관계에서는 orphanRemoval = true를 단독으로 사용하면 안 됨 (고아 객체 문제 발생 가능)**

---

## 결론

JPA에서 **CascadeType과 orphanRemoval을 올바르게 설정하면 부모-자식 객체의 생명주기를 효과적으로 관리**할 수 있습니다.  
하지만 다수의 부모가 연관된 경우 불필요한 삭제가 발생할 수 있으므로 신중하게 설정해야 합니다.  
기본적으로는 **CascadeType.PERSIST**나 **MERGE를 사용하고 필요할 때만 REMOVE나 orphanRemoval을 추가**하는 것이 좋습니다.