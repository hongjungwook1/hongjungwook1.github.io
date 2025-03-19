---
title: Spring 트랜잭션 관리 Connection을 직접 사용하지 말자
description: >-
  JDBC API에서 Connection을 직접 관리하는 방식과 Spring이 제공하는 트랜잭션 관리 방법을 비교하고 선언적 트랜잭션과 프로그래밍 방식의 트랜잭션을 효과적으로 활용하는 방법
author: JWHONG
date: 2025-03-19 11:00:00 +0900
categories: [Spring , Spring Boot , Transaction, Database]
tags: [Spring , Spring Boot , Transaction, Database]
pin: true
media_subpath: "/posts/20250319"
---

## Spring 트랜잭션 관리 - Connection을 직접 사용하지 말자

### 트랜잭션의 기본 개념 

트랜잭션을 간단히 정리하면 다음 세 단계로 나눌 수 있습니다.

1. **트랜잭션 생성** &rarr; `Connection` 생성
2. **트랜잭션 참여** &rarr; 기존 `Connection`을 공유하여 여러 SQL 실행
3. **트랜잭션 처리** &rarr; 성공 시 `Commit` , 실패 시 `Rollback`

JDBC API에서는 다음과 같은 두 가지 방식으로 트랜잭션을 처리할 수 있습니다.

- **(A) 1 쿼리 : 1 트랜잭션**

    - **쿼리 한 개를 하나의 트랜잭션으로 수행하는 경우** (기본 설정)
    - `Auto-Commit`이 활성화된 상태에서 개별 SQL 문이 실행될 때마다 자동으로 커밋됩니다.

- **(B) N 쿼리 : 1 트랜잭션**

    - **여러 개의 쿼리를 하나의 트랜잭션으로 묶어 수행하는 경우** (개발자가 직접 설정)
    - `Auto-Commit`을 비활성화하고 여러 개의 SQL 문을 하나의 트랜잭션으로 묶어 실행한 후 명시적으로 `Commit` 또는 `Rollback`을 호출합니다.

(B) 방식은 같은 `Connection`을 유지해야 하므로 관리가 어렵고 개발자가 직접 `try-catch-finally` 블록을 사용해 트랜잭션을 처리해야 합니다.  
이를 해결하기 위해 Spring에서는 `JdbcTemplate`과 `PlatformTransactionManager`를 제공합니다.

---

## Spring이 제공하는 트랜잭션 관리 방식

Spring은 트랜잭션 관리를 자동화하여 개발자가 `Connection`을 직접 제어하지 않고도 안전하게 트랜잭션을 관리할 수 있도록 지원합니다.

1. **트랜잭션 생성** &rarr; `PlatformTransactionManager`가 `Connection`을 생성하고 트랜잭션을 시작합니다.
2. **트랜잭션 참여** &rarr; `JdbcTemplate` 또는 `@Transactional`을 통해 기존 트랜잭션에 참여합니다.
3. **트랜잭션 처리** &rarr; 성공 시 `Commit`, 실패 시 `Rollback`이 자동으로 실행됩니다.

이 과정에서 Spring은 **트랜잭션 동기화 (TransactionSynchronizationManager)** 와 **트랜잭션 추상화 (PlatformTransactionManager)** 를 통해 트랜잭션을 효율적으로 관리합니다.

---

## Spring 트랜잭션의 핵심 개념

### 트랜잭션 동기화 (TransactionSynchronizationManager)

> **트랜잭션 동기화란?**  
같은 트랜잭션을 유지하려면 같은 `Connection`을 사용해야 합니다.  
Spring은 `TransactionSynchronizationManager`를 사용하여 `Connection`을 자동으로 저장하고 재사용할 수 있도록 합니다.

### 트랜잭션 추상화 (PlatformTransactionManager)

> **트랜잭션 추상화란?**
다양한 트랜잭션 기술(JDBC, JPA, R2DBC 등)을 통일된 방식으로 사용할 수 있도록 제공하는 인터페이스입니다.

Spring의 `PlatformTransactionManager`는 **트랜잭션의 시작과 종료를 담당하는 인터페이스입니다.**  
로컬 트랜잭션이든 글로벌 트랜잭션이든 **모든 트랜잭션을 동일한 방식으로 관리**할 수 있도록 설계되었습니다.

---

## Spring 트랜잭션 관리 방법

Spring에서 트랜잭션을 관리하는 방법은 크게 **두 가지**로 나뉩니다.

- **프로그래밍 방식 (TransactionTemplate 사용)**

    - `TransactionTemplate`을 사용하여 **개발자가 직접 트랜잭션을 관리**하는 방식입니다.
    - 특정 메서드의 일부분만 트랜잭션을 적용할 수 있습니다.
    - `@Transactional`과 다르게 **Self Invocation 문제 없이 내부 호출 시에도 트랜잭션을 적용할 수 있습니다.**

**사용 예시**

```java
@Service
@RequiredArgsConstructor
public class SimpleService {
    private final TransactionTemplate transactionTemplate;

    public void method() {
        String result = transactionTemplate.execute(new TransactionCallback<String>() {
                public String doInTransaction(TransactionStatus status) {
                    jdbcTemplate.update(...);
                    jdbcTemplate.update(...);
                    return "";
                }
        });
    }
}
```


- **선언적 방식 (@Transactional 사용)**

    - `@Transactional` 애너테이션을 사용하여 **트랜잭션을 자동으로 관리**하는 방식입니다.
    - Spring AOP를 기반으로 동작하며 보일러플레이트 코드 없이 간편하게 트랜잭션을 적용할 수 있습니다.
    - 가장 많이 사용되는 방식이지만 **메서드 내 일부 로직만 트랜잭션을 적용할 수 없으며 Self Invocation 문제가 발생할 수 있습니다.**

**사용 예시**

```java
@Service
public class SimpleService {
	
    @Transactional
    public void method() {
        jdbcTemplate.update(...);
        jdbcTemplate.update(...);
    }
}
```

---

## 결론

**Spring 트랜잭션을 사용하는 이유**

- `JdbcTemplate`은 **JDBC API의 반복적인 작업을 줄이고** 트랜잭션을 자동으로 동기화해 줍니다.
- `PlatformTransactionManager`를 사용하면 **트랜잭션을 추상화하여** 여러 데이터베이스 기술에서도 동일한 방식으로 트랜잭션을 사용할 수 있습니다.
- 선언적 트랜잭션(`@Transactional`)을 사용하면 **트랜잭션을 보다 간편하게 적용할 수 있습니다.**