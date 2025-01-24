---
title: 객체 생성 방법 생성자, 빌더, 정적 팩토리 메서드 비교
description: >-
  객체를 생성하는 방법 중 주요한 세 가지 방법(생성자, 빌더, 정적 팩토리 메서드)의 장단점과 활용법.
author: JWHONG
date: 2025-01-23 15:00:00 +0900
categories: [Java, OOP, Tutorial]
tags: [Java, OOP, Constrictor, Builder, Static Factory Method]
pin: true
media_subpath: "/posts/20250123"
---

# 객체 생성 방법: 생성자, 빌더, 정적 팩토리 메서드 비교

객체를 생성하는 방법에는 여러 가지가 있으며 이 중 주요한 3가지 방법은 **생성자**, **빌더**, 그리고 **정적 팩토리 메서드**입니다.

---

## 생성자(Constructor)

생성자는 객체를 초기화하는 역할을 하며 초기화할 필드에 따라 여러 형태로 정의할 수 있습니다.

### 기본 생성자 + Setter 활용

필드 값을 나중에 설정할 필요가 있을 때 사용하는 방식입니다.

```java
Member aaron = new Member();
aaron.setName("Hong");
aaron.setEmail("Hong@example.com");
```

```java
@NoArgsConstructor
public class Member {
    private String name;
    private String email;

    //public Member() {}

    public void setName(String name) { this.name = name; }
    public void setEmail(String email) { this.email = email; }
}
```

### 필드 값을 주입받는 커스텀 생성자

1. 특정 필드만 초기화

```java
Member aaron = new Member("Hong");
aaron.setEmail("Hong@example.com");
```

```java
public class Member {
    private String name;
    private String email;

    public Member(String name) { this.name = name; }

    public void setEmail(String email) { this.email = email; }
}
```

2. DTO를 활용한 생성자

```java
public class Member {
    private String name;
    private String email;

    public Member(MemberRequestDto dto) {
        this.name = dto.getName();
        this.email = dto.getEmail();
    }
}
```

```java
MemberRequestDto dto = new MemberRequestDto("Hong", "Hong@example.com");
Member aaron = new Member(dto);
```

3. @AllArgsConstructor 사용

```java
Member aaron = new Member(1, "Hong", 10, "Hong@example.com");
```

```java
public class Member {
    private Integer id;
    private String name;
    private int age;
    private String email;

    public Member(Integer id, String name, int age, String email) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.email = email;
    }
}
```

---

## 빌더 패턴(Builder)

빌더는 객체 생성 시 모든 경우의 수를 지원하며 더 유연한 방식으로 객체를 생성할 수 있게 합니다.

### 2.1. 기본 빌더 패턴 사용

빌더 패턴은 3단계로 구성됩니다

- 빌더 정의
- 필드 설정
- 객체 생성

```java
Member.MemberBuilder builder = Member.builder();
builder
    .name("Hong")
    .email("Hong@example.com");

Member member = builder.build();
```

### 2.2. 필드 주입 순서 무관

필드 설정 순서에 구애받지 않고 값을 주입할 수 있습니다.

```java
Member member = Member.builder()
    .name("Hong")
    .email("Hong@example.com")
    .build();
```

### @Builder와 다양한 활용

1. @Builder.Default로 기본값 설정

```java
@Builder
public class Member {
    private Integer id;
    @Builder.Default
    private String name = "Unnamed";
    private int age;
    @Builder.Default
    private String email = "Undefined";
}
```

2. @Singular로 컬렉션 값 주입

```java
@Builder
public class Member {
    private Integer id;
    private String name;
    private int age;
    private String email;
    @Singular
    private List<String> favorites;
}
```

```java
Member member = Member.builder()
    .name("Hong")
    .email("Hong@example.com")
    .favorite("Book")
    .favorite("Cook")
    .build();
```

---

## 정적 팩토리 메서드(Static Factory Method)

정적 팩토리 메서드는 객체 생성 방식에 제약을 두고 캡슐화된 메서드를 통해 객체를 생성합니다.

```java
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class Member {
    private Integer id;
    private String name;
    private int age;
    private String email;

    public static Member from(MemberCreateRequestDto requestDto) {
        return new Member(
            null,
            requestDto.getName(),
            0,
            requestDto.getEmail()
        );
    }
}
```

```java
MemberCreateRequestDto requestDto = new MemberCreateRequestDto("Hong", "Hong@example.com");
Member member = Member.from(requestDto);
```

#### 결론

객체 생성 방식은 프로그램의 설계와 유지보수에 큰 영향을 미칩니다.

- 생성자는 간단한 객체 초기화에 적합합니다.
- 빌더는 복잡한 객체 생성과 필드 값 설정이 자유로워야 할 때 사용합니다.
- 정적 팩토리 메서드는 객체 생성 과정을 명확히 표현하고 캡슐화해야 할 때 유용합니다.
