---
title: 데이터베이스 트랜잭션의 격리 레벨 (Isolation Level) 4가지
description: >-
  DB에 여러 개의 CRUD 접근이 발생할 때 충돌을 방지하고 동시성 제어를 위해 개발자가 설정할 수 있는 격리 수준을 이해해야 합니다.
author: JWHONG
date: 2025-03-12 15:00:00 +0900
categories: [DataBase, DB, Transaction Isolation Level]
tags: [DataBase, DB, Transaction Isolation Level]
pin: true
media_subpath: "/posts/20250312"
---

## 트랜잭션과 격리 레벨

데이터베이스에서 **트랜잭션(Transaction)**은 ACID 원칙을 준수해야 하며 이 중 **Consistency(일관성)과 Data Integrity(데이터 무결성)**을 보장하는 것이 중요하다.  
하지만 트랜잭션 간의 완전한 독립성 보장은 성능 저하를 초래할 수 있다.

### 왜 트랜잭션의 격리 정도를 조절할까?

  - **동시성 향상** : 여러 트랜잭션이 동시에 실행될 수 있도록 하기 위해
  - **읽기 성능 최적화** : 트랜잭션이 읽을 수 있는 데이터의 범위를 조절해 처리 속도를 높이기 위해
  - **격리 수준(L0 ~ L3)** : 격리 정도에 따라 SQL 표준에서 정의된 네 가지 단계가 존재

### 격리 수준과 해결해야 할 문제들

|**격리 레벨**|**문제 발생 유형**|
|:---:|:---:|
|**Read Uncommitted**|Dirty Read|
|**Read Committed**|Non-Repeatable Read|
|**Repeatable Read**|Phantom Read|
|**Serializable**|완전한 직렬화 보장|

---

## L0 : Read Uncommitted (Dirty Read 허용)

  - **커밋되지 않은 데이터도 읽을 수 있음**
  - 성능은 가장 뛰어나지만 데이터 일관성(Consistency)이 보장되지 않음
  - **Dirty Read 문제 발생** : 한 트랜잭션에서 수정한 값이 롤백되더라도 다른 트랜잭션에서 읽을 수 있음
  - **사용 사례** : 데이터 일관성이 크게 중요하지 않고 빠른 성능이 필요한 경우 (예: 로그 데이터 수집)

## L1 : Read Committed (Non-Repeatable Read 허용)

  - **커밋된 데이터만 읽을 수 있음 (MVCC 활용)**
  - **Dirty Read 방지**
  - **Non-Repeatable Read 문제 발생** : 같은 트랜잭션 내에서 같은 데이터를 두 번 읽었을 때 값이 다를 수 있음
  - **사용 사례** : 대부분의 애플리케이션에서 기본적으로 사용하는 격리 수준

### MVCC (Multiversion Concurrency Control)

  - **MySQL (InnoDB)** : Undo 영역 활용
  - **PostgreSQL** : Snapshot 기반 MVCC 활용
  - **MySQL의 Consistent Read** : 실제 테이블이 아니라 Undo 영역의 백업된 데이터를 읽음 

## L2 : Repeatable Read (Phantom Read 허용)

- **트랜잭션 내에서 같은 데이터를 여러 번 읽어도 같은 값이 유지됨**
- **Non-Repeatable Read 방지**
- **Phantom Read 문제 발생 가능** : 한 트랜잭션이 실행 중일 때 다른 트랜잭션이 새로운 행을 추가하면 기존 트랜잭션이 인지하지 못할 수 있음
- **MySQL에서는 Gap Lock 사용으로 Phantom Read 문제를 방지함**
- **사용 사례** : 데이터 일관성이 중요한 애플리케이션 (예: 금융, 결제 시스템)


## L3 : Serializable (완전한 직렬화 보장)

  - **가장 엄격한 격리 수준 완전한 ACID 보장**
  - **모든 트랜잭션을 직렬화된 순서로 실행** &rarr; 동시성이 가장 낮음
  - **성능이 가장 비효율적** (테이블 레벨 Lock 발생 가능)
  - **사용 사례** : 극단적으로 높은 데이터 일관성이 필요한 경우 (예: 회계 시스템)

---

## Spring Data JPA에서 @Transactional의 Isolation Level

Spring Boot에서 **Spring Data JPA**를 사용하면 @Transactional 애노테이션을 활용해 트랜잭션의 격리 수준을 설정할 수 있다.

### @Transactional에서 사용할 수 있는 Isolation 옵션

|**옵션**|**설명**|
|:---:|:---:|
|**Default**|DB 기본 설정을 따름 (일반적으로 Read Committed)|
|**Read Uncommitted**|가장 낮은 수준 Dirty Read 허용|
|**Read Committed**|커밋된 데이터만 읽음, Non-Repeatable Read 가능|
|**Repeatable Read**|동일한 데이터를 여러 번 읽어도 동일한 값 유지|
|**Serializable**|가장 높은 격리 수준 모든 트랜잭션 직렬화|

```java
@Service
public class ExampleService {

    @Transactional(isolation = Isolation.REPEATABLE_READ)
    public void processTransaction() {
        // 트랜잭션 내에서 동일한 데이터를 읽어도 동일한 결과 유지
    }
}
```

--- 

## 결론

  - **트랜잭션 격리 수준은 성능과 데이터 정합성의 균형을 맞추는 것이 핵심**
  - 일반적으로 **Read Committed(L1)** 또는 **Repeatable Read(L2)**를 많이 사용함
  - 데이터 정합성이 중요한 경우 **Repeatable Read** 이상을 고려해야 함
  - Spring Data JPA에서는 @Transactional을 활용해 격리 수준을 조정 가능