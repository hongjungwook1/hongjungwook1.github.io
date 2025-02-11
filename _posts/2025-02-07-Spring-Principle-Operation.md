---
title: Spring Boot 동작 원리 - 요청부터 응답까지
description: >-
  Spring Boot의 요청 처리 흐름과 DispatcherServlet의 동작 방식에 대해 정리하였습니다.
author: JWHONG
date: 2025-02-07 20:00:00 +0900
categories: [Spring, Spring Boot, MVC, Backend]
tags: [Spring, Spring Boot, MVC, Backend]
pin: true
media_subpath: "/posts/20250207"
---

# Spring Boot의 동작 원리

## Spring Boot의 요청 처리 흐름

`Spring Boot`는 기본적으로 `내장된 Tomcat` 서버 위에서 동작하는 Java 애플리케이션입니다.

클라이언트가 요청을 보내면 `Tomcat의 서블릿 컨테이너`가 이를 받아 `DispatcherServlet`에게 전달하고 Spring이 이를 적절한 컨트롤러로 매핑하여 처리하는 구조입니다.

요청 처리 흐름은 다음과 같이 요약할 수 있습니다

**1. 요청(Request) → DispatcherServlet이 처리**

1.  클라이언트가 HTTP 요청을 보냅니다.
2.  Tomcat의 **Servlet Container**가 `DispatcherServlet`을 호출합니다.
3.  `DispatcherServlet`은 적절한 컨트롤러를 찾기 위해 `HandlerMapping`을 사용합니다.

**2. 처리(Processing) → 적절한 컨트롤러 실행**

1.  `HandlerMapping`이 요청과 일치하는 컨트롤러와 메서드를 찾습니다.
2.  `HandlerAdapter`가 해당 컨트롤러를 실행하여 요청을 처리합니다.
3.  컨트롤러는 요청을 처리한 후 **응답 데이터를 반환**합니다.

**3. 응답(Response) → DispatcherServlet이 최종 응답 반환**

1.  컨트롤러에서 반환된 응답을 `DispatcherServlet`이 받습니다.
2.  반환된 타입에 따라 `ViewResolver` 또는 `MessageConverter`를 사용해 응답을 생성합니다.
3.  최종적으로 클라이언트에게 HTTP 응답을 반환합니다.

> 참고: `DispatcherServlet`은 기본적으로 싱글톤으로 동작하며 다수의 요청이 동시에 들어오면 각각의 요청을 개별 스레드에서 처리합니다.

---

## **Controller에서의 반환 방식**

컨트롤러는 클라이언트 요청을 처리한 후 응답을 웹 페이지(HTML) 또는 JSON 데이터 형식으로 반환할 수 있습니다.

### **HTML 웹 페이지 반환 (View 반환) → ViewResolver 사용**

Spring MVC에서는 컨트롤러가 뷰 이름(String) 또는 `ModelAndView`를 반환하면 `ViewResolver`가 이를 처리하여 웹 페이지를 생성합니다.

#### **String 반환 → 정적 페이지(ViewTemplate) 렌더링**

```java
@Controller
public class HomeController {
    @GetMapping("/home")
    public String home() {
        return "home.html"; // View 이름 반환
    }
}
```

- `home.html` 템플릿 파일을 찾아 렌더링합니다.
- `ViewResolver`가 템플릿 엔진(Thymeleaf, JSP 등)을 사용해 HTML 페이지를 생성합니다.

#### **ModelAndView 반환 → 동적 페이지(ViewTemplate + Model) 렌더링**

```java
@Controller
public class UserController {
    @GetMapping("/user")
    public ModelAndView user() {
        ModelAndView mv = new ModelAndView();
        mv.addObject("name", "홍길동");
        mv.setViewName("user");
        return mv;
    }
}
```

- `user.html` 템플릿이 렌더링되며 `name` 데이터가 포함됩니다.
- `ModelAndView`를 사용하면 뷰 템플릿뿐만 아니라 데이터를 함께 반환할 수 있습니다.

---

### **JSON 데이터 반환 → MessageConverter 사용**

`@ResponseBody`를 적용한 컨트롤러 메서드는 데이터를 JSON 형태로 변환하여 반환합니다. Spring은 내부적으로 `HttpMessageConverter`를 사용해 객체를 JSON으로 변환합니다.

#### **객체 반환 → JSON 변환 (@ResponseBody 사용)**

```java
@RestController
public class ApiController {
    @GetMapping("/api/user")
    public User user() {
        return new User("홍길동", 25);
    }
}
```

- User 객체가 JSON으로 변환되어 반환됩니다.
- 내부적으로 `MappingJackson2HttpMessageConverter`가 동작하여 Java 객체를 JSON으로 변환합니다.

#### **@RestController 사용 → 모든 메서드가 JSON 반환**

```java
@RestController
@RequestMapping("/api")
public class UserRestController {
    @GetMapping("/user")
    public User getUser() {
        return new User("홍길동", 25);
    }
}
```

`@RestController`는 `@Controller` + `@ResponseBody`가 결합된 형태이며 모든 메서드가 JSON 데이터를 반환합니다.

> 참고: JSON 데이터 변환을 위해 Spring Boot에서는 기본적으로 Jackson 라이브러리를 사용하지만 필요하면 Gson 또는 XML 변환 라이브러리를 사용할 수도 있습니다.
