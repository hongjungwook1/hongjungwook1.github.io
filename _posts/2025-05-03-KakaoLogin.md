---
title: Spring Boot OAuth2 카카오 로그인 완벽 가이드 with JWT
description: >-
    Spring Boot 프로젝트에 카카오 로그인 기능을 OAuth2 흐름으로 구현하고 JWT 토큰을 발급하는 전체 과정을 구조적으로 설명합니다.
author: JWHONG
date: 2025-05-03 10:00:00 +0900
categories: [Spring Boot, OAuth2]
tags: [Spring Boot, KakaoLogin, JWT, OAuth2, WebClient]
pin: true
media_subpath: "/posts/202500503"
---

카카오 로그인을 연동하면서 가장 중요한 건 **OAuth2 인증 흐름을 이해하고, 역할에 따라 클래스를 명확하게 분리하는 것**입니다.      
이번 글에서는 Spring Boot 기반 프로젝트에서 카카오 로그인 연동을 어떻게 구성하고 JWT 발급까지의 흐름을 어떻게 설계할 수 있는지 구조적으로 설명드리겠습니다.

---

## 프로젝트 구조 개요

**카카오 로그인을 구현하는 데 필요한 구성은 다음과 같이 나뉩니다**

```yaml
📁 controller
    └── KakaoLoginController.java
📁 service
    └── KakaoOAuthService.java
📁 dto
    ├── KakaoTokenResponse.java
    └── KakaoUserInfo.java
📁 util
    └── JwtTokenProvider.java
📁 config
    └── WebClientConfig.java
```

---

## controller - 사용자 요청 진입 지점

- `KakaoLoginController.java`는 클라이언트가 카카오로부터 전달받은 인가코드를 `GET` 요청으로 보내는 진입점입니다.
- 인가 코드를 받아 `KakaoOAuthService`에 위임하며 결과적으로 JWT 토큰을 반환합니다.

```java
@GetMapping("/auth/login/kakao")
public ResponseEntity<?> kakaoLogin(@RequestParam String code) {
    String jwtToken = kakaoOAuthService.loginWithKakao(code);
    return ResponseEntity.ok(Map.of("token", jwtToken));
}
```

---

## service - OAuth2 흐름 처리 핵심 로직

- `KakaoOAuthService.java`는 다음과 같은 역할을 수행합니다

  1. 인가 코드를 사용해 Access Token 요청
  2. Access Token으로 사용자 정보 요청
  3. 사용자 이메일을 기반으로 JWT 생성

```java
public String loginWithKakao(String code) {
    String accessToken = getAccessToken(code);
    KakaoUserInfo userInfo = getUserInfo(accessToken);
    return jwtTokenProvider.createToken(userInfo.getEmail());
}
```

---

## dto - 외부 응답 데이터 구조 매핑

- `KakaoTokenResponse.java` 카카오에서 받은 Access Token 응답(JSON)을 자바 객체로 변환합니다.

```java
@FieldDefaults(level = AccessLevel.PRIVATE)
@Setter
@Getter
public class KakaoTokenResponse {
    @JsonProperty("access_token")
    String accessToken;

    @JsonProperty("refresh_token")
    private String refreshToken;

}
```

- `KakaoUserInfo.java` 사용자 이메일만 담은 간단한 응답 DTO입니다.

```java
@AllArgsConstructor
@Getter
@FieldDefaults(level = AccessLevel.PRIVATE)
public class KakaoUserInfo {
    String email;
}
```

---

## util - JWT 토큰 생성기

- `JwtTokenProvider.java`는 사용자 이메일을 기반으로 JWT 토큰을 생성합니다.
- **HS256 알고리즘**을 사용할 경우 키는 반드시 **256비트(32바이트 이상)** 길이를 만족해야 합니다.

```java
@Component
public class JwtTokenProvider {

    private static final String SECRET_KEY = "super-secure-secret-key-which-is-at-least-32bytes";
    private static final SecretKey KEY = Keys.hmacShaKeyFor(SECRET_KEY.getBytes(StandardCharsets.UTF_8));

    public String createToken(String email) {
        return Jwts.builder()
                .setSubject(email)
                .setIssuedAt(new Date())
                .setExpiration(new Date(System.currentTimeMillis() + 3600000)) // 1시간
                .signWith(KEY, SignatureAlgorithm.HS256)
                .compact();
    }
}
```
>보안 팁 : **SECRET_KEY**는 외부에 노출되면 안 되며 **application.yml** 또는 **환경변수**로 관리하는 것이 좋습니다.

---

## config - WebClient 설정

- 카카오 API 호출을 위해 `WebClient` Bean을 등록합니다.

```java
@Configuration
public class WebClientConfig {
    @Bean
    public WebClient webClient() {
        return WebClient.builder().build();
    }
}
```

---

## 로그인 흐름 요약

- 프론트엔드에서 카카오 로그인 버튼 클릭
- 사용자가 카카오 동의 &rarr; 리다이렉트 URI로 인가코드 전달
- 백엔드에서는 다음과 같은 흐름으로 처리됩니다

```bash
Controller
   ↓
KakaoOAuthService
   ├── POST /oauth/token 요청 (인가코드 → accessToken)
   ├── GET /v2/user/me 요청 (accessToken → 사용자 이메일)
   ↓
JwtTokenProvider (이메일 → JWT 생성)
```
- 최종적으로 JWT를 클라이언트에 응답으로 반환합니다.

---

## Postman 테스트 전 확인 사항

- `/auth/login/kakao?code=...`에 들어가는 값은 JWT가 아니라 실제 인가코드여야 합니다.
- 인가 코드는 카카오 로그인을 수행한 후 **리다이렉트 URI에 붙어서 전달**됩니다.
```bash
http://localhost:8080/auth/login/kakao?code=k0y8_Wtl... (←  카카오 인가 코드)
```

---

## 결론 

카카오 로그인은 OAuth2 프로토콜을 따르며 아래 요소들을 정확히 설계해야 합니다.

- 인가 코드 &rarr; Access Token &rarr; 사용자 정보 &rarr; JWT 발급 흐름 이해
- 클래스 역할 분리: Controller / Service / DTO / Util / Config
- 시크릿 키 보안: JWT 서명 키는 절대 외부에 노출되면 안 되며 고정 길이를 만족해야 함
- Postman 테스트 시 실제 인가코드를 사용해야만 카카오 API가 정상 동작함