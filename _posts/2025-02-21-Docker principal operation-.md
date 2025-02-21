---
title: Docker를 사용하는 이유 - 격리(Isolation), 일관성(Consistency), 다중 컨테이너(Multiple Containers)
description: >-
  Docker는 애플리케이션을 격리된 환경에서 실행할 수 있도록 도와주는 도구로 일관된 환경을 유지하고 다중 컨테이너를 효과적으로 관리할 수 있도록 합니다.
author: JWHONG
date: 2025-02-21 11:00:00 +0900
categories: [Docker, DevOps, Containerization]
tags: [Docker, 컨테이너, DevOps, CI/CD, 배포 자동화]
pin: true
media_subpath: "/posts/20250221"
---

## Docker란 ?

Docker는 격리된 공간에서 프로세스를 실행할 수 있도록 해주는 가상화 기술입니다. <br>
이를 통해 하나의 서버(Host OS)에서 다양한 환경의 애플리케이션을 효율적으로 실행할 수 있습니다.

### Docker의 주요 개념

- **Docker Image (도커 이미지)**

  - 애플리케이션 실행에 필요한 **Bins/Libs 및 App**이 포함된 패키지
  - 예: Spring Boot 앱, MySQL 데이터베이스 등

- **Dockerfile (도커파일)**

  - **Docker Image**를 만들기 위한 설정 파일
  - 어떤 애플리케이션과 라이브러리를 포함할지 명시

- **Docker Container (도커 컨테이너)**

  - **실행 중인 Docker Image**
  - 메모리와 CPU를 할당받아 독립적인 프로세스로 동작

---

## Docker Workflow (사용 절차)

### 일반적인 Docker 사용 절차

1. **Develop** → Java 애플리케이션 개발
2. **Test** → 코드 테스트
3. **Build Java** → Java 빌드 (JAR, WAR 생성)
4. **Build Docker Image** → Dockerfile을 기반으로 이미지 생성
5. **Push Docker Image** → 도커 저장소(Docker Hub, AWS ECR 등)에 업로드
6. **Pull Docker Image** → 서버에서 도커 이미지 다운로드
7. **Run Docker Image** → 도커 컨테이너 실행

> **CI/CD 전략** : 위 과정에서 빌드 및 배포 단계의 자동화를 설정할 수 있음.

### Docker Image 배포 방법

1. **SCP 방식** : Docker Image를 tar 압축 후 서버로 전송하여 실행
2. **ECR 방식** : AWS ECR에 Docker Image 업로드 후 서버에서 다운로드 및 실행
   - **ECS 또는 Terraform과 연계하여 자동 배포 가능**

---

## Consistency (일관성 보장)

애플리케이션이 어느 환경(개발 PC, 서버, 클라우드 인스턴스)에서든 정상적으로 동작하려면 실행 환경이 동일해야 합니다. <br>
이를 **일관성(Consistency)**이라고 하며 Docker를 사용하면 이를 보장할 수 있습니다.

- **일관성을 위해 필요한 요소**

  - **애플리케이션 파일**
  - **환경 변수(Environment Variables)**
  - **런타임 환경(Runtime Environment, 예: Java, Python 버전)**
  - **서드파티 라이브러리**
  - **경량 운영체제(Cut-down OS)**

Docker를 사용하면 애플리케이션과 필요한 환경을 패키징하여 어디서든 동일한 상태로 실행할 수 있습니다.

---

## Isolation (격리) & Consistency (일관성 보장)

Docker는 애플리케이션 실행 환경을 **격리(Isolation)**하여 로컬 환경과 충돌하지 않게 하고 일관된 환경에서 실행되도록 보장합니다.

- **Docker가 없다면** : 로컬 환경과 애플리케이션 환경이 충돌하여 실행 문제가 발생할 가능성이 높음.
- **Docker가 있다면** : 컨테이너 내부에서 애플리케이션이 실행되므로 로컬 환경과 독립적으로 운영됨.
- **컨테이너는 일회용품처럼 사용 가능** : 필요할 때 실행하고 필요 없으면 삭제하면 됨.

![Image](https://github.com/user-attachments/assets/71462699-c169-456c-9496-dbcd76c822c0)

---

## 결론

Docker를 사용하면 애플리케이션의 **일관성(Consistency)과 격리(Isolation)**을 보장하고 **다중 컨테이너(Multiple Containers)**를 효율적으로 운영할 수 있습니다. <br>
또한 CI/CD와 연계하여 자동화된 배포 환경을 구축할 수 있어 개발과 운영이 더욱 편리해집니다.
