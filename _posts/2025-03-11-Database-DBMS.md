---
title: DB와 DBMS
description: >-
  DB 와 DBMS 관계형 데이터베이스 & 비관계형 데이터베이스
author: JWHONG
date: 2025-03-11 11:00:00 +0900
categories: [DataBase, DB, DataBaseManagerService]
tags: [DataBase, DB, DataBaseManagerService]
pin: true
media_subpath: "/posts/20250311"
---

## 데이터베이스(DB)

  - **데이터 저장소**로 정보를 체계적으로 관리하는 공간
  - **CSV , Excel 같은 단순한 데이터 저장 형식도 포함**
  - 구조화된 데이터를 저장하고 검색할 수 있도록 설계됨

## 데이터베이스 관리 시스템(DBMS)

  - **데이터 + 엔진 + 관리 기능을 제공하는 소프트웨어**
  - 데이터를 효율적으로 저장 , 수정 , 삭제 , 검색할 수 있도록 함
  - SQL 같은 질의(Query) 언어를 사용하여 데이터 조작 가능
  - 대표적인 DBMS: **Oracle, MySQL, PostgreSQL, MongoDB**

---

## 관계형 데이터베이스 (RDBMS)

  - **행렬(Row-Column) 기반의 정형 데이터 저장**
  - **고정된 스키마(Fixed Schema) 사용**
  - **데이터 간 관계(Relation)를 이용해 일관성 유지 (PK-FK 관계)**
  - **SQL(Structured Query Language) 사용**
  - **트랜잭션과 무결성 보장 (ACID 특성 지원)**

### 관계형 데이터베이스의 특징

  - **높은 데이터 무결성(Data Integrity)과 신뢰성**
  - **강력한 트랜잭션(Transaction) 지원 (ACID 준수)**
  - **JOIN을 통한 복잡한 데이터 조회 가능**
  - **SQL 최적화 기능을 통한 성능 개선**

## 비관계형 데이터베이스 (NoSQL)

  - **비정형 데이터 저장 (JSON, Key-Value, Graph 등)**
  - **스키마 없이(Schemaless) 자유롭게 데이터 저장 가능**
  - **수평적 확장(Scalability)에 유리**
  - **트랜잭션 및 데이터 무결성 보장이 약함 (BASE 모델 적용)**

### NoSQL의 특징

  - **대용량 데이터 저장 및 빠른 처리 속도**
  - **복잡한 데이터 구조를 유연하게 저장 가능**
  - **수평적 확장 용이 (클러스터링 및 샤딩 지원)**
  - **일관성(Consistency) 보다는 가용성(Availability) 중시**

---

## 관계형 데이터베이스의 트랜잭션과 ACID

### 트랜잭션(Transaction)이란 ? 

  - 데이터베이스에서 한 번에 실행되어야 하는 연산의 집합
  - 트랜잭션 중 일부 연산이 실패하면 전체 작업이 취소(Rollback)됨

### ACID 특성

  - **원자성(Atomicity)** : 트랜잭션 내 모든 연산이 성공 또는 실패해야 함
  - **일관성(Consistency)** : 트랜잭션 실행 전후 데이터 일관성이 유지됨
  - **격리성(Isolation)** : 여러 트랜잭션이 동시에 실행될 때 서로 간섭하지 않음
  - **지속성(Durability)** : 트랜잭션이 성공적으로 완료되면 데이터가 영구적으로 저장됨

> 예시 : 은행 계좌 이체 시 A 계좌에서 돈이 빠져나가고 B 계좌에 입금되는 과정이 원자성을 보장해야 함

## NoSQL 데이터베이스의 BASE 모델

### BASE 모델이란?

  - NoSQL에서는 ACID 대신 BASE 모델을 따름
  - **Basically Available (기본적인 가용성)**
  - **Soft state (일시적인 비일관성 허용)**
  - **Eventual consistency (최종적 일관성 보장)**

> 예시 : SNS 게시물 조회 시 최신 댓글이 즉시 반영되지 않을 수 있으며 일정 시간이 지나야 동기화됨

---

## 관계형 vs 비관계형 데이터베이스 비교

|**비교 항목**|**관계형 데이터베이스(RDBMS)**|**비관계형 데이터베이스(NoSql)**|
|:---:|:---:|:---:|
|데이터 구조|정형 (Row-Column)|비정형 (JSON , Key-Value 등)| 
|스키마|고정 (Fixed Schema)|유동적 (Schema-less)| 
|관계|PK-FK 기반의 관계 설정|데이터 간 명확한 관계 없음| 
|트랜잭션|ACID 보장|BASE 모델| 
|확장성|수직적 확장 (Scale-up)|수평적 확장 (Scale-out)| 
|사용 예시|은행 , ERP , 쇼핑몰|SNS , 빅데이터 , 로그 분석|

---

## 대표적인 DBMS 종류

### 관계형 데이터베이스 (RDBMS)

  - **Oracle** : 기업용 데이터베이스, 강력한 보안 및 성능 제공
  - **MySQL** : 오픈소스 RDBMS, 웹 서비스에서 많이 사용
  - **PostgreSQL** : 확장성이 뛰어난 오픈소스 DB
  - **SQL Server** : Microsoft에서 제공하는 RDBMS

### 비관계형 데이터베이스 (NoSQL)

  - **MongoDB** : JSON(Document) 기반 데이터베이스
  - **Redis** : Key-Value 기반의 초고속 데이터 저장소
  - **Cassandra** : 대규모 분산 환경에서 사용되는 NoSQL DB
  - **Neo4j** : 그래프(Graph) 기반 데이터베이스

--- 

## 데이터베이스 선택 시 고려 사항

  - **데이터 구조** : 정형 데이터는 RDBMS, 비정형 데이터는 NoSQL이 적합
  - **확장성** : 수평적 확장이 필요하면 NoSQL이 유리
  - **트랜잭션 요구사항** : 강력한 트랜잭션이 필요하면 RDBMS 선택
  - **처리 속도** : 대량의 데이터를 빠르게 처리하려면 NoSQL이 효과적

---

## 결론

데이터베이스를 선택할 때는 **데이터 구조, 확장성, 트랜잭션 지원 여부**를 고려해야 합니다.

 - **빠르고 안정적인 트랜잭션이 중요하다면?** &rarr; 관계형 데이터베이스 (RDBMS)
 - **대용량 데이터를 빠르게 처리하고 싶다면?** &rarr; 비관계형 데이터베이스 (NoSQL)