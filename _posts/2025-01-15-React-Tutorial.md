---
title: React 데이터 바인딩과 Hook 가이드
description: >-
  React 데이터 바인딩과 Hook 가이드
author: JWHONG
date: 2025-01-15 15:40:00 +0900
categories: [React, Tutorial]
tags: [React Tutorial]
pin: true
media_subpath: "/posts/20250115"
---

# React 단방향 바인딩, JSX, 그리고 Hook의 이해

React는 UI를 효율적으로 구성할 수 있는 JavaScript 라이브러리입니다.
React의 주요 개념인 데이터 바인딩, JSX의 특징, 그리고 Hook을 활용하는 방법에 대해 정리해 보겠습니다.

---

### 바인딩이란?

바인딩은 화면(View)과 데이터(Model)를 동기화하는 것을 의미합니다. React는 단방향 데이터 흐름을 기본으로 합니다.

---

### 단방향 바인딩

React의 단방향 바인딩은 데이터가 부모 컴포넌트 → 자식 컴포넌트로 흐릅니다.

**장점**

- **책임 분리** : Model과 View 간의 역할이 명확히 구분됩니다.
- **일관성** : 데이터 흐름이 단순하고 명확합니다.
- **성능 최적화 가능** : 불필요한 DOM 렌더링을 방지할 수 있습니다.

**단점**

- 사용자의 이벤트 처리를 위해 상태 업데이트 코드(setState)를 매번 작성해야 합니다.

---

### JSX의 특징

JSX는 React에서 UI를 정의하기 위해 사용하는 문법입니다. JavaScript와 XML의 특성을 모두 포함하고 있으며 다음과 같은 특징과 제약사항이 있습니다.

#### 특징

**컴포넌트 중심 설계** : XML과 JavaScript가 함께 작성되어 응집도가 높습니다.
**트랜스파일링** : JSX는 Babel과 같은 도구를 통해 JavaScript로 변환됩니다.

---

### JSX의 제약사항

하나의 루트 요소로 반환
JSX는 항상 하나의 요소로 감싸야 합니다.

```jsx
<div>
  <h1>오늘의 할 일</h1>
  <ul>
    <li>빨래하기</li>
    <li>숙제하기</li>
    <li>1시 약속</li>
  </ul>
</div>
```

`<div>` 태그를 사용하지 않으려면 `<React.Fragment>` 또는 빈 태그 <>를 사용할 수 있습니다.

```jsx
<>
  <h1>오늘의 할 일</h1>
  <ul>
    <li>빨래하기</li>
    <li>숙제하기</li>
    <li>1시 약속</li>
  </ul>
</>
```

**JavaScript 로직 포함
JSX 내부에서 JavaScript 로직을 사용하려면 중괄호 {}를 이용합니다.**

```jsx
function Container({ children }) {
  const title = "오늘의 할 일";
  return (
    <div>
      <h1>{title}</h1>
      {children}
    </div>
  );
}
```

#### 주석 처리

HTML 주석 `<!-- -->` 대신 `{/\* \*/}`를 사용해야 합니다.

#### 닫는 태그 필수

모든 태그는 명시적으로 닫아야 합니다.

```jsx
<input type="text" />
```

---

### React Hook

React Hook은 함수형 컴포넌트에서 상태 관리와 생명주기 기능을 사용할 수 있도록 도와줍니다.

#### 규칙

함수 이름은 use로 시작해야 합니다.
반드시 함수형 컴포넌트 내부에서 호출해야 합니다.

##### 주요 Hook

**useState** : 간단한 상태 관리

```jsx
const [count, setCount] = useState(0);
```

**useReducer** : 복잡한 상태 관리

- `useState`보다 제약이 강하며 Context와 함께 사용해 전역 상태를 관리할 수 있습니다.

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

**useEffect** : 렌더링 후 동작을 정의

```jsx
useEffect(() => {
  console.log("Component rendered");
}, [dependency]);
```

**useMemo** : 값 캐싱

```jsx
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

**useCallback**: 함수 재사용

```jsx
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

**React.memo**: 컴포넌트 재사용

```jsx
export default React.memo(MyComponent);
```

**useLayoutEffect** : 화면에 렌더링되기 직전 실행

```jsx
useLayoutEffect(() => {
  console.log("Before paint");
});
```

---

### 성능 최적화 팁

> **불필요한 렌더링 방지** : 상태 관리와 데이터 흐름을 효율적으로 설계합니다.

> **구조 설계**: 상태를 최상단에 두지 말고 필요한 컴포넌트에만 전달합니다.
