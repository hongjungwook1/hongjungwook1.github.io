---
title: 웹의 기본 구조와 아키텍처
description: >-
  웹 애플리케이션의 핵심 개념인 요청과 반환, 그리고 Monolithic 아키텍처와 MSA, API Gateway 구조의 차이와 장단점을 설명합니다.
author: HongJW
date: 2024-12-10 11:00:00 +0900
categories: [Web Architecture, Backend, Tutorial]
tags: [Web Structure, Monolithic, MSA, API Gateway, Architecture]
pin: true
media_subpath: "/posts/20241210"
---

# 웹의 기본 구조 = 요청과 반환

웹은 **요청(Request)**과 **반환(Response)**이라는 두 가지 주체로 구성된다.   
이 간단한 구조가 현대 웹 애플리케이션의 근본을 이룬다.


## Monolithic Architecture: 단일 서버 기반의 아키텍처

**Monolithic Architecture**이란 하나의 서버가 모든 서비스를 처리하는 구조를 말한다.  

예를 들어 **[장바구니, 구매, 상품 보기 : 서버 1]** 와 같은 기능이 하나의 서버에서 실행된다면

![Image](https://github.com/user-attachments/assets/f286c847-3a9a-4677-90ba-d3ca131149e7)
> 출처 : ASAC 07 강의자료

- **장점**
    - **빠르고 간단한 개발** : 작은 프로젝트에서는 단일 서버로 모든 서비스를 구현하므로 개발 속도가 빠르다.

- **단점**
  - **서비스 간 의존성**: 한쪽 서비스에 문제가 발생하면 다른 서비스도 영향을 받아 전체 시스템이 중단될 위험이 있다.
  - **SPOF(Single Point of Failure)**: 단일 실패 지점이 전체 시스템의 문제로 이어질 가능성이 크다.

---

## MSA(Microservice Architecture): 마이크로서비스 기반의 아키텍처

**MSA(Microservice Architecture)**는 각각의 서비스가 독립적인 서버에서 실행되는 구조이다.   

예를 들어: **[장바구니: 서버 1] , [구매: 서버 2] , [상품 보기: 서버 3]**

![Image](https://github.com/user-attachments/assets/6d01019f-6b9b-40bc-8f0b-e3e09a82c170)
> 출처 : ASAC 07 강의자료

- **장점**
  - **독립적인 서비스 운영**: 특정 서비스에 문제가 생겨도 다른 서비스에는 영향이 없다.
  - **유연한 확장성**: 필요한 서비스만 확장하거나 유지보수할 수 있다.

- **단점**
  - **복잡한 관리**: 수많은 서버 속 어떤 서버의 어떤 API 를 사용해야할지 정리정돈, 버전관리 힘들다.

---

## API Gateway로 MSA 단점 해결

MSA의 단점을 보완하기 위해 API Gateway가 등장했다.   
API Gateway는 **모든 서버에 대한 API 호출을 중앙화**하여 관리하는 리버스 프록시 역할을 한다.


**API Gateway의 특징**
 - **중앙화된 API 관리**: 각 서버의 API를 중앙에서 관리하여 어떤 서버에서 어떤 API를 제공하는지 쉽게 확인 가능하다.
 - **버전 관리 및 문서화**: Swagger와 같은 도구를 통해 API 버전과 설명을 체계적으로 관리하다.