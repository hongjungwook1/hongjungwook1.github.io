---
title: Java 예외 처리 : 컴파일 에러(Checked)와 런타임 에러(Unchecked)
description: >-
  자바는 예외(Exception) 처리를 아주 엄격하게 강제합니다. 자바에서는 Checked Exception과 Unchecked Exception 두 가지로 명확히 구분합니다.
author: JWHONG
date: 2025-02-06 11:00:00 +0900
categories: [Java, Exception, Tutorial]
tags: [Java, Exception]
pin: true
media_subpath: "/posts/20250206"
---

## 자바의 예외 처리 메커니즘

자바는 **try → throw → catch** 구조를 사용하여 예외를 처리합니다.
예를 들어 처리되지 않은 예외가 있다면 IDE에서 빨간색 에러 표시와 함께 컴파일이 되지 않습니다.
특히 아래와 같이 반드시 처리해야 하는 예외(Checked Exception)의 경우 컴파일러가 강제로 예외 처리 코드를 요구합니다.

**Try-Catch로 직접 처리하기**

```java
class FileReader {
    private static void checkedExceptionWithTryCatch() {
        try {
            File file = new File("not_existing_file.txt");
            FileInputStream stream = new FileInputStream(file);
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

**Throws로 예외 전가하기**

```java
class FileReader {
    private static void checkedExceptionWithThrows() throws FileNotFoundException {
        File file = new File("not_existing_file.txt");
        FileInputStream stream = new FileInputStream(file);
    }
}
```

> 위 예제처럼 **throws**를 사용할 때는 메서드 선언에 반드시 예외 타입을 명시해야 합니다.

---

## Checked Exception vs Unchecked Exception

자바에서의 예외는 **발생 원인(Source)**에 따라 두 가지로 구분합니다.

**프로그램 “외부”에서 발생하는 예외: Checked Exception**

- **특징**

  - 충분히 예상 가능한 오류에 대해 컴파일 타임에 예외 처리가 강제됨
  - 파일 입출력, 데이터베이스 연결, 파싱 오류 등

- **주요 예시**

  - **FileNotFoundException** : 파일을 읽을 때 해당 파일이 존재하지 않을 수 있음
  - **IOException** : 입출력 작업 중 발생할 수 있는 다양한 예외
  - **SQLException** : 데이터베이스 연결 및 쿼리 실행 시 발생할 수 있는 예외
  - **ParseException** : 문자열을 특정 형식으로 변환할 때 발생하는 예외

**프로그램 “내부 로직”에서 발생하는 예외: Unchecked Exception**

- **특징**

  - 발생 원인을 예측하기 어렵고 복구 방법이 명확하지 않음
  - 런타임 시 발생하므로 컴파일러가 예외 처리를 강제하지 않음
  - 발생 시 반드시 로그를 남겨서 문제의 원인을 추적할 수 있도록 해야 함

- **주요 예시**

  - **NullPointerException** : 예상치 못하게 참조값이 **null**인 경우
  - **ArithmeticException** : 산술 연산(예: 0으로 나누기)에서 발생
  - **ArrayIndexOutOfBoundsException** : 배열의 잘못된 인덱스 접근

```java
private static void uncheckedExceptionWithTryCatch(String str) {
    try {
        // str에 null이 전달될 가능성이 있음
        str = str.toUpperCase();
        System.out.println(str);
    } catch (NullPointerException e) {
        e.printStackTrace();
    }
}
```

---

## 언제 Checked Exception과 Unchecked Exception을 사용할까?

- **Checked Exception**

  - 외부 환경(파일, 네트워크, DB 등)과 같이 복구 가능한 상황에서 발생하는 예외에 사용합니다.
  - 예외 발생 가능성을 컴파일 타임에 검증할 수 있어 개발자가 대비책을 마련할 수 있습니다.

- **Unchecked Exception**
  - 코드 내부의 논리 오류나 예상치 못한 상황 등 프로그래밍 오류에 사용합니다.
  - 대부분의 경우 이러한 예외는 사전에 완벽히 처리하기 어렵기 때문에 런타임 시 로그를 통해 추적하고 디버깅하는 것이 중요합니다.

---

## 예외 처리를 잘한다는 것은?

예외 처리를 잘 다룬다는 것은 단순히 try-catch 문을 남발하는 것이 아닙니다.
능력 있는 개발자는 아래와 같은 기준으로 예외 처리를 설계합니다.

- **계층을 잘 나누기**

  - 예를 들어 Repository, Service, Controller 등으로 계층을 분리하여 각 계층에서 발생할 수 있는 예외를 구분합니다.

- **예외 종류를 명확히 구분하기**

  - 상황에 맞는 커스텀 예외를 정의하고 필요하다면 Enum 등을 사용하여 다양한 케이스를 체계적으로 관리합니다.

- **적절한 예외 전파와 처리 전략 선택하기**

  - 한 곳에서 모든 예외를 처리하거나 중간에 받아서 다시 전파(예외 타입 변환 포함)하거나 특정 상황에서는 의도적으로 예외를 먹는 등 상황에 맞게 전략을 선택합니다.

---

## 결론

자바의 예외 처리 메커니즘은 프로그램의 안정성과 유지보수성을 높이는 데 큰 역할을 합니다.
**Checked Exception**과 **Unchecked Exception**의 차이 그리고 각각의 적절한 사용 방법을 이해하고 상황에 맞는 예외 처리를 설계한다면 보다 견고한 애플리케이션을 개발할 수 있을 것입니다.
