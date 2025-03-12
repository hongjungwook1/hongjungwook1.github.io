---
title: 데이터베이스 동시성 제어 Pessimistic Lock vs Optimistic Lock
description: >-
  데이터베이스 동시성 제어 Pessimistic Lock vs Optimistic Lock
author: JWHONG
date: 2025-03-12 11:00:00 +0900
categories: [DataBase, DB, Concurrency Control , Pessimistic , Optimistic]
tags: [DataBase, DB, Concurrency Control , Pessimistic , Optimistic]
pin: true
media_subpath: "/posts/20250312"
---

## 데이터베이스 동시성 제어란?

멀티유저 환경에서는 여러 개의 애플리케이션이 동시에 데이터베이스에 접근할 수 있습니다.  
이런 환경에서 데이터 일관성을 유지하려면 **동시성 제어(Concurrency Control)**가 필수적입니다.  

- 동시성 제어는 다음과 같은 문제를 해결하는 데 초점을 맞춥니다

  - 여러 개의 CRUD 요청이 동시에 발생할 때 데이터 충돌 방지
  - 데이터 무결성(Data Integrity) 보장
  - 데이터 일관성 유지

동시성 제어 방식은 크게 **Pessimistic Lock(비관적 락)**과 **Optimistic Lock(낙관적 락)** 두 가지로 나뉩니다.

---

## Pessimistic Lock (비관적 락)

> "분명히 충돌이 날 거야! 미리 잠가두자."

Pessimistic Lock은 데이터의 충돌 가능성을 염두에 두고 **데이터를 수정하려는 순간부터 잠금을 걸어** 다른 트랜잭션이 접근하지 못하도록 합니다.  
이 방식은 RDBMS(관계형 데이터베이스)에서 자주 사용됩니다.

**특징**

  - **Lock 기반 제어 방식** : 데이터베이스에서 제공하는 내장 락 기능을 사용
  - **강력한 무결성 보장** : 동시성 문제 방지
  - **성능 저하 가능성** : Lock을 사용하면 처리 속도가 느려질 수 있음
  - **데드락(Deadlock) 발생 가능성** : 여러 트랜잭션이 서로의 락을 기다리면 교착 상태가 발생할 수 있음

---

## Optimistic Lock (낙관적 락)

> "설마 충돌이 나겠어? 그냥 진행하고 나중에 확인하자."

Optimistic Lock은 데이터 충돌이 발생할 가능성이 낮다고 가정하고 **트랜잭션이 데이터를 수정하기 전에 잠금을 걸지 않습니다.**  
대신 데이터 수정 시점에서 충돌 여부를 확인합니다. NoSQL(비관계형 데이터베이스)에서 많이 사용됩니다.

**특징**

  - **소프트웨어 기반 제어 방식** : 개발자가 직접 충돌 검증 로직을 구현해야 함
  - **고성능 처리 가능** : Lock을 사용하지 않으므로 동시성이 뛰어남
  - **데이터 충돌 발생 가능성** : 충돌이 발생하면 예외 처리가 필요함
  - **추가 개발 필요** : 버전 관리 및 충돌 해결 로직을 직접 구현해야 함

---

## Pessimistic vs Optimistic Lock 비교

|**구분**|**Pessimistic Lock**|**Optimistic Lock**|
|:---:|:---:|:---:|
|개념|미리 락을 걸어 충동을 방지|충돌 가능성을 무시하고 진행 후 체크|
|특징|트랜잭션 단위로 강력한 데이터 보호|동시성을 극대화하지만 충돌 시 예외 처리 필요|
|성능|동시성 낮음 (Lock 있음)|동시성 높음 (Lock 없음)|
|적용 사례|RDBMS (MySQL , Oracle , PostgreSQL)|NoSQL (MongoDB , DynamoDB , Redis)|
|사용 시점|쓰기 비중이 높은 경우|읽기 비중이 높은 경우|

---

## 결론

  - **Pessimistic Lock** : 강력한 데이터 무결성 보장, 하지만 성능 저하 가능
  - **Optimistic Lock** : 높은 동시성 제공, 하지만 충돌 발생 가능
  - 시스템의 요구 사항에 맞는 동시성 제어 방식을 선택하는 것이 중요
  - **읽기 중심 시스템**은 Optimistic Lock, **쓰기 중심 시스템**은 Pessimistic Lock을 고려

데이터베이스의 동시성 제어 방식은 애플리케이션의 성능과 안정성에 큰 영향을 미칩니다.  
따라서 트랜잭션의 특성과 시스템의 특성을 고려하여 적절한 방식을 선택하는 것이 중요합니다.