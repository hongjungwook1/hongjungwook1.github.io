---
title: Spring Boot @Value Injection Error 해결
description: >-
    Spring Boot에서 @Value와 @ConfigurationProperties를 함께 사용할 때 발생하는 주입 오류 문제와 해결 방법을 사례와 함께 자세히 설명합니다.
author: JWHONG
date: 2025-05-04 10:00:00 +0900
categories: [Spring Boot, Configuration, Troubleshooting]
tags: [Spring Boot, Value, ConfigurationProperties, YAML, 설정관리]
pin: true
media_subpath: "/posts/202500504"
---

# Spring Boot @ConfigurationProperties와 @Value 트러블슈팅

개발을 하다 보면 실수하기 쉬운 부분 중 하나가 바로 외부 설정값을 불러오는 방법인 것 같습니다.   
Spring Boot에서는 주로 `@Value`와 `@ConfigurationProperties`를 사용하는데 이 두 어노테이션을 함께 사용할 때 발생할 수 있는 문제점과 해결 방법에 대해 공유하고자 합니다.

## 문제 상황

카카오 API를 연동하기 위해 클라이언트 ID와 시크릿 키를 `application.yml`에 설정하고 이를 불러와 사용하려고 했습니다.   
처음에는 다음과 같이 코드를 작성했습니다

```java
@Setter
@Getter
@Component
@ConfigurationProperties(prefix = "kakao")
public class KakaoProperties {
    
    @Value("${kakao.client-id}")
    private String clientId;
    
    @Value("${kakao.client-secret}")
    private String clientSecret;
}
```

그리고 `application.yml`은 다음과 같이 설정했습니다

```yaml
kakao:
  client-id: abcdefg12345
  client-secret: xyz789secret
```

그런데 이 설정으로는 값이 제대로 주입되지 않는 문제가 발생했습니다.

## 원인 분석

트러블슈팅을 해본 결과 다음과 같은 문제점들을 발견했습니다.

1. **`@ConfigurationProperties`와 `@Value`를 함께 사용함**

   - `@ConfigurationProperties(prefix = "kakao")`를 사용하면 이미 "kakao" 접두사 아래의 모든 속성을 해당 클래스의 필드에 바인딩합니다.
   - 이 상태에서 다시 `@Value("${kakao.client-id}")`로 같은 값을 주입하려고 하면 충돌이 발생합니다.

2. **`@Component` 어노테이션과 `@EnableConfigurationProperties`의 충돌**
   - `@ConfigurationProperties`를 사용할 때는 해당 클래스를 빈으로 등록하는 방법이 여러 가지인데 이 중 하나만 선택해야 합니다.

3. **YAML 파일의 들여쓰기 문제**
   - YAML 파일은 들여쓰기에 민감하기 때문에 들여쓰기가 잘못되면 속성이 제대로 인식되지 않을 수 있습니다.

## 해결 방법

### 방법 1: `@ConfigurationProperties`만 사용하기

```java
@Setter
@Getter
@ConfigurationProperties(prefix = "kakao")
public class KakaoProperties {
    
    private String clientId;
    private String clientSecret;
}
```

### 방법 2: `@EnableConfigurationProperties` 사용하기

메인 애플리케이션 클래스에 다음과 같이 추가합니다

```java
@SpringBootApplication
@EnableConfigurationProperties(KakaoProperties.class)
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

이때는 `KakaoProperties` 클래스에서 `@Component` 어노테이션을 제거해야 합니다

```java
@Setter
@Getter
@ConfigurationProperties(prefix = "kakao")
public class KakaoProperties {

    private String clientId;
    private String clientSecret;
}
```

### 방법 3: YAML 파일 들여쓰기 확인

```yaml
# 올바른 형식
kakao:
  client-id: abcdefg12345
  client-secret: xyz789secret

# 잘못된 형식 (들여쓰기 문제)
kakao:
client-id: abcdefg12345  # 들여쓰기 누락
  client-secret: xyz789secret
```

## 실제 사용 예시

올바르게 설정한 후의 사용 예시입니다

```java
@Service
public class KakaoService {
    
    private final KakaoProperties kakaoProperties;
    
    public KakaoService(KakaoProperties kakaoProperties) {
        this.kakaoProperties = kakaoProperties;
    }
    
    public void connectToKakaoApi() {
        String clientId = kakaoProperties.getClientId();
        String clientSecret = kakaoProperties.getClientSecret();
    
        // API 연동 로직
    }
}
```

## 결론

Spring Boot에서 외부 설정을 불러올 때는 `@Value`와 `@ConfigurationProperties` 중 하나를 선택해서 사용하는 것이 좋습니다.   
특히 관련된 여러 설정값을 묶어서 관리해야 할 때는 `@ConfigurationProperties`가 더 적합합니다. 

또한 YAML 파일의 들여쓰기에 주의하고 빈 등록 방식(`@Component` vs `@EnableConfigurationProperties`)을 명확히 하는 것이 중요합니다.