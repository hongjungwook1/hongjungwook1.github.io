---
title: Spring 트랜잭션 전파(Propagation)
description: >-
  @Transactional을 사용할 때 Propagation 옵션을 올바르게 이해하고 적용하는 방법
author: JWHONG
date: 2025-03-19 15:00:00 +0900
categories: [Spring , Spring Boot , Transaction, Database , Propagation]
tags: [Spring , Spring Boot , Transaction, Database , Propagation]
pin: true
media_subpath: "/posts/20250319"
---

## 선언적 트랜잭션 @Transactional 사용 시 옵션

Spring에서 `@Transactional`을 사용할 때 `PlatformTransactionManager`의 다양한 옵션을 설정할 수 있습니다.  
이번 글에서는 그중에서도 **Propagation(전파 옵션)**을 중심으로 상세히 알아보겠습니다.

트랜잭션을 선언하는 기본적인 예제는 다음과 같습니다.

```java
@Transactional(
        propagation   = Propagation.REQUIRED,      // 트랜잭션 전파 방식
        isolation     = Isolation.REPEATABLE_READ, // 트랜잭션 격리 수준
        timeout       = -1,                        // 트랜잭션 타임아웃 설정
        readOnly      = false,                     // 읽기 전용 여부
        rollbackFor   = Exception.class,           // 롤백 대상 예외
        noRollbackFor = RuntimeException.class     // 롤백 제외 예외
)
public void method() {
    jdbcTemplate.update(...);
    jdbcTemplate.update(...);
}
```

이번 글에서는 위 옵션 중 **Propagation(전파 옵션)**을 집중적으로 살펴보겠습니다.

---

## Propagation 전파: @Transactional 안에서 또 @Transactional 선언

### 트랜잭션 전파란?

일반적으로 트랜잭션은 하나의 작업 단위로 여러 개의 쿼리를 묶어 실행하는데 하나라도 실패하면 전체를 롤백(rollback)합니다. 하지만 모든 작업을 하나의 트랜잭션으로 묶는 것이 항상 옳은 것은 아닙니다.

예를 들어 **로깅(Log)이나 감사(Audit) 테이블을 기록하는 작업**은 핵심 비즈니스 로직과 독립적으로 실행되는 것이 좋습니다.  
만약 핵심 로직에서 예외가 발생했다고 로깅까지 롤백되어서는 안 되기 때문입니다.

이처럼 일부 작업을 독립적인 트랜잭션으로 실행하고 싶을 때 **Propagation(전파 옵션)**을 활용합니다.

---

## 물리 트랜잭션과 논리 트랜잭션

트랜잭션을 이야기할 때 헷갈릴 수 있는 개념이 **물리 트랜잭션(Physical Transaction)**과 **논리 트랜잭션(Logical Transaction)**입니다.

|**구분**|**설명**|
|:---:|:---:|
|**물리 트랜잭션**|실제 데에터베이스 트랜잭션 (Connection을 통해 관리됨)|
|**논리 트랜잭션**|Spring의 `PaltformTransactionManger`가 관리하는 트랜잭션|

Spring에서는 **하나의 물리 트랜잭션 안에 여러 개의 논리 트랜잭션이 존재할 수 있습니다.**  
`@Transactional`을 여러 번 선언해도 내부적으로는 하나의 물리 트랜잭션에 종속될 수도 있고 독립적으로 실행될 수도 있습니다.

---

## Propagation 전파 옵션 정리

### REQUIRED (기본값) - 기존 트랜잭션이 있으면 참여, 없으면 새로 생성
> Single Shared Transaction (양방향) : 내부 롤백 시 외부 롤백

```java
@Transactional(propagation = Propagation.REQUIRED)
public void method() {
    jdbcTemplate.update(...);
    jdbcTemplate.update(...);
}
```
- **기존 트랜잭션이 존재하면 그 트랜잭션을 공유**합니다.
- **기존 트랜잭션이 없으면 새로운 트랜잭션을 생성**합니다.
- **내부 트랜잭션에서 예외가 발생하면 전체가 롤백됩니다.**

### REQUIRES_NEW - 항상 새로운 트랜잭션을 생성
> Inner Transaction (단절) : 내부와 외부는 각자 별도의 롤백/커밋

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void method() {
    jdbcTemplate.update(...);
    jdbcTemplate.update(...);
}
```

- 기존 트랜잭션이 있더라도 **새로운 트랜잭션**을 생성합니다.
- **독립적인 트랜잭션이므로 내부 트랜잭션이 롤백되더라도 외부 트랜잭션에 영향을 주지 않습니다.**
- 예를 들어 핵심 비즈니스 로직과 로깅 작업을 독립적으로 수행할 때 사용됩니다.

### NESTED - 부모 트랜잭션 내에서 중첩 트랜잭션 생성
> Nested Transaction (단방향) : 내부 롤백 시 외부 롤백하지 않음

```java
@Transactional(propagation = Propagation.NESTED)
public void method() {
    jdbcTemplate.update(...);
    jdbcTemplate.update(...);
}
```

- 기존 트랜잭션이 존재하면 **그 안에서 중첩된 트랜잭션**을 생성합니다.
- 부모 트랜잭션이 커밋되면 내부 트랜잭션도 커밋되지만 내부 트랜잭션이 롤백되더라도 부모 트랜잭션에는 영향을 주지 않습니다.


### 기타 Propagation 옵션 요약 

|**구분**|**설명**|
|:---:|:---:|
|**SUPPORTS**|트랜잭션이 있으면 참여, 없으면 트랜잭션 없이 실행|
|**NOT_SUPPORTED**|트랜잭션을 사용하지 않고 실행 (기존 트랜잭션이 있어도 무시)|
|**MANDATORY**|반드시 기존 트랜잭션이 있어야 하며, 없으면 예외 발생|
|**NEVER**|기존 트랜잭션이 있으면 예외 발생 (트랜잭션 없이 실행해야 함)|

---

## 결론

Spring의 트랜잭션 전파 옵션을 이해하고 적절히 활용하면 **비즈니스 로직에 따라 유연한 트랜잭션 관리가 가능**합니다.

- **REQUIRED** : 대부분의 경우 기본값으로 충분
- **REQUIRES_NEW** : 독립적인 트랜잭션이 필요할 때
- **NESTED** : 부모-자식 관계의 트랜잭션을 만들고 싶을 때