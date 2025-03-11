---
title: 데이터베이스 확장 전략 Partitioning, Sharding, Replication
description: >-
  대규모 데이터베이스 시스템에서는 부하 분산 및 확장성을 확보하기 위해 다양한 기법을 사용합니다.
author: JWHONG
date: 2025-03-11 15:00:00 +0900
categories: [DataBase, DB, DataBaseExpansionStrategy , Partitioning, Sharding, Replication]
tags: [DataBase, DB, DataBaseExpansionStrategy ,Partitioning, Sharding, Replication]
pin: true
media_subpath: "/posts/20250311"
---

대규모 데이터베이스 시스템에서는 **부하 분산** 및 **확장성**을 확보하기 위해 다양한 기법을 사용합니다.  
대표적으로 **(Vertical) Partitioning, Sharding (Horizontal Partitioning), Replication** 전략이 있습니다.

NoSQL 데이터베이스(Couchbase, Elasticsearch 등)를 사용할 때 **Replica, Shard** 개념을 자주 접하게 됩니다.  
하지만 최근에는 RDBMS(SQL)에서도 이러한 개념이 도입되어 사용되고 있습니다.

## RDBMS와 NoSQL의 확장성

과거에는 Sharding이 주로 NoSQL에서 사용되었지만 최신 RDBMS에서도 지원이 확대되고 있습니다.

  - **CockroachDB (분산형 RDBMS)** : Sharding 지원
  - **MySQL, MSSQL, PostgreSQL** : Sharding 지원

## Partitioning (파티셔닝)

**Partitioning**은 하나의 테이블을 여러 개로 분할하여 저장하는 방식입니다.  
대규모 데이터를 효율적으로 관리할 수 있도록 데이터를 쪼개어 저장합니다.

![Image](https://github.com/user-attachments/assets/16f98041-8090-4a42-8c65-4a6d7e44f798)
> 이미지 출처 : :https://www.digitalocean.com/community/tutorials/understanding-database-sharding

### (Vertical) Partitioning: 컬럼(열) 분할

  - 테이블의 특정 **열(Column)**을 기준으로 분할하여 저장합니다.
  - 주로 데이터 구조가 커질 때 **읽기 성능 최적화**를 위해 사용됩니다.

### Horizontal Partitioning = Sharding: 행(Row) 분할

**Sharding**은 데이터를 **로우(Row) 기준으로 분리하는 방식**으로 대량의 데이터를 효율적으로 관리할 수 있습니다.

 - 주요 방식
  - **Range-based Sharding** : 알파벳, 숫자, 시간 등을 기준으로 데이터를 **범위(range)별로 분리**
  - **Hash-based Sharding** : 해시 함수를 사용해 데이터를 **균등하게 분산**
  - **Composite Sharding** : **Range-based**와 **Hash-based**를 **조합하여 사용**

---

## Replication (복제)

**Replication**은 동일한 데이터를 여러 분산 노드에 저장하는 기법입니다.  
**읽기(Read)와 쓰기(Write) 작업을 분리**하여 성능을 최적화할 수 있습니다.

  - 데이터베이스 부하를 줄이기 위해 **Master-Slave** 구조를 활용
  - **쓰기(Write) 작업은 Master 노드가 담당, 읽기(Read) 작업은 Slave 노드가 분산하여 처리**
  - **쓰기 작업 발생 시, 읽기 담당 노드(Slave) 모두에 동기화**

### CQRS (Command and Query Responsibility Segregation)

  - **CQRS 패턴은 데이터 저장소로부터의 읽기(Read)와 쓰기(Write) 작업을 분리하는 패턴입니다.**
  - **쓰기(Write) 연산은 별도의 커맨드(Command) 핸들러가 처리, 읽기(Read) 연산은 쿼리(Query) 핸들러가 처리**
  - **데이터 일관성을 유지하면서도 성능과 확장성을 극대화 가능**
  - **애플리케이션의 퍼포먼스(Performance), 확장성(Scalability), 보안(Security) 최적화 가능**

![Image](https://github.com/user-attachments/assets/d207f4b2-2710-4e5d-9ab5-50ae5c224b19)
> 출처 : https://docs.aws.amazon.com/ko_kr/prescriptive-guidance/latest/modernization-data-persistence/cqrs-pattern.html