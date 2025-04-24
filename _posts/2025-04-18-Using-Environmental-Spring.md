---
title: Spring Boot에서 환경변수 사용하기: 윈도우 & 리눅스 서버까지 완벽 정리
description: >-
    Spring Boot에서 이메일 인증과 같은 민감한 설정 정보를 안전하게 관리하기 위한 환경변수 설정 방법을 윈도우, 리눅스, Docker 환경까지 상세히 정리한 글입니다.
author: JWHONG
date: 2025-04-18 21:00:00 +0900
categories: [Spring Boot, 환경 설정]
tags: [spring boot, 환경변수, 윈도우, 리눅스, 서버환경]
pin: true
media_subpath: "/posts/20250418"
---

Spring Boot 프로젝트를 개발하시다 보면 이메일 인증과 같은 기능을 구현할 때 민감한 정보를 **application.yml**에 직접 하드코딩하기보다는 **환경변수로 관리**하시는 것이 보안이나 유지보수 측면에서 훨씬 더 안전하고 효율적입니다.

---

## 왜 하드코딩 말고 환경변수를 써야 할까?

**하드코딩된 민감 정보 예시**

```yaml
spring:
  mail:
    username: youremail@naver.com  # 이런 식으로 직접 쓰면 위험합니다.
    password: yourapppassword
```

- Git 저장소에 올릴 경우 비밀번호가 그대로 노출될 수 있습니다.
- 운영 환경마다 값이 다를 경우 수동으로 수정해주어야 합니다.

**환경변수로 관리하면**

```yaml
spring:
  mail:
    username: ${EMAIL}
    password: ${PASSWORD}
```

- Git에 민감한 정보가 노출되지 않습니다.
- 환경에 따라 값을 따로 관리할 수 있어 편리합니다.

---

## Windows에서 환경변수 설정하기

### CMD 창에서 일시적으로 등록

```bash
set EMAIL=youremail@naver.com
set PASSWORD=yourapppassword
```
> 단점 : CMD 창을 닫으면 사라짐

### 시스템 환경변수로 영구 등록

1. Windows 검색창에 `환경 변수` 검색 &rarr; **시스템 환경 변수 편집** 클릭
2. **환경 변수(N)** 버튼 클릭
3. 사용자 변수에 EMAIL, PASSWORD 추가
4. PC 재시작 또는 새 CMD 창 열기

### 정상 등록 확인 방법

```bash
echo %EMAIL%
```

--- 

## IntelliJ에서 .env 파일로 관리하고 싶다면?

`.env` 파일을 루트 디렉토리에 만들고 아래처럼 작성합니다.

```bash
EMAIL=youremail@naver.com
PASSWORD=yourapppassword
```

### .env 파일 적용을 위한 의존성 추가

```xml
<!-- build.gradle 또는 pom.xml -->
<dependency>
  <groupId>me.snowlight</groupId>
  <artifactId>spring-dotenv</artifactId>
  <version>1.3.0</version>
</dependency>
```

해당 의존성을 등록하시면 `.env` 파일의 값들이 Spring Boot 실행 시 자동으로 환경변수로 인식됩니다.

---

## Ubuntu (리눅스) 서버 환경에서는?

윈도우랑 완전 별개로 **리눅스 환경에서도 따로 환경변수를 등록**해주셔야 합니다.

### 영구적으로 환경변수 등록하기

- 터미널을 열고 `~/.bashrc` 또는 `~/.profile` 파일을 수정합니다.

```bash
nano ~/.bashrc
```

- 하단에 아래 내용을 추가합니다.

```bash
export EMAIL=youremail@naver.com
export PASSWORD=yourapppassword
```

- 변경 사항을 적용합니다.

```bash
source ~/.bashrc
```

- 정상 등록 확인

```bash
echo $EMAIL
```

--- 

## Docker 환경에서 사용하는 방법

### 1. `docker run` 명령어로 직접 전달

```bash
docker run -e EMAIL=youremail@naver.com -e PASSWORD=yourapppassword your-image
```

### 2. `.env` 파일을 활용하여 전달

```bash
# .env 파일 작성
EMAIL=youremail@naver.com
PASSWORD=yourapppassword
```
```bash
docker --env-file .env run your-image
```

---

## `application.yml`에서 환경변수 사용하기

환경변수 설정이 완료되었다면 아래처럼 `application.yml`에 적용할 수 있습니다.

```yaml
spring:
  mail:
    host: smtp.naver.com
    port: 465
    username: ${EMAIL}
    password: ${PASSWORD}
    properties:
      mail:
        smtp:
          auth: true
          ssl:
            enable: true
            trust: smtp.naver.com
    protocol: smtp
    default-encoding: UTF-8
```

---

## 디버깅 팁: Spring에서 환경변수 인식 안 될 때?

### 로그 확인

```java
log.info("환경변수에서 불러온 EMAIL = {}", System.getenv("EMAIL"));
```

### 만약 null로 출력된다면?

- `.env` 적용이 안 된 경우
- IntelliJ에서 Run Configuration에 환경변수를 명시하지 않은 경우
- 윈도우 환경변수 등록 후 재시작 안 한 경우
- 리눅스에서 `source ~/.bashrc` 안 한 경우

---

## 결론

|**상황**|**추천 방식**|
|:---:|:---:|
|**로컬 개발 중**|`.env` + `spring-dotenv`|
|**윈도우 개발 환경**|사용자 환경변수 등록|
|**리눅스 서버 환경**|`~/.bashrc`에 export~|
|**도커 환경**|`--env` or `.env` 파일|

민감한 정보는 절대 깃에 올리지 말고 **환경변수**나 `.env` 파일로 안전하게 관리하자!