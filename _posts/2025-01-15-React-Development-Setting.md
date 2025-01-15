---
title: React 개발 환경 설정
description: >-
  React 개발 환경 설정 및 설정 내용
author: HongJW
date: 2025-01-15 11:00:00 +0900
categories: [React, Setting, Tutorial]
tags: [React Development Setting]
pin: true
media_subpath: "/posts/20250115"
---

# Vite, Next.js, 그리고 Node.js 더 나은 React 개발 환경을 위한 선택

React 개발을 시작하려면 더 이상 `create-react-app`을 사용하지 않습니다. 대신 개발 목표에 따라 다음 도구를 활용할 수 있습니다.

- **Vite**: 빠르고 경량화된 환경에서 React 애플리케이션을 개발하고 싶다면 Vite를 사용하세요.
- **Next.js**: React 기반 애플리케이션의 확장성과 서버 사이드 렌더링(SSR)을 활용하려면 Next.js가 적합합니다.
- **Node.js**: React 프로젝트의 필수 기반입니다. Node.js는 JavaScript 런타임 환경으로, 패키지 관리 도구(NPM, Yarn 등)와 함께 라이브러리 설치, 스크립트 실행, 빌드 작업 등을 지원합니다.

  - Node.js 버전 관리를 간편하게 하려면 `nvm(Node Version Manager)`을 사용하는 것을 추천합니다.

---

## Node.js와 패키지 관리

### nvm (Node Version Manager)

Node.js 버전을 간단히 변경하거나 관리하는 도구입니다.

---

### nvm 설치 방법

Bash 명령어가 필요하기에 윈도우의 경우 **Git Bash** 사용 필수

1. > `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash`

2. > `nano ~/.baschrc` 혹은 `vim ~/.bashrc` 입력 후

3. > `export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm` 위 내용 추가 후 저장

4. > `nvm -v` 명령어를 통해 nvm 설치가 제대로 되었는지 확인 가능합니다.

### 자주 사용하는 nvm 명령어 모음

- `nvm list` : 현재 내 로컬에 설치되어있는 node 버전은 무엇들이 있는지 + 현재 쓰고있는지 **확인**
- `nvm install 18` : **18** 버전 node **설치**
- `nvm install --lts 18` : **18** 버전 중 가장 최신의 LTS 버전 node **설치**
- `nvm install` : 현재 위치한 프로젝트의 `.nvmrc` 파일 내 명시되어있는 node 버전 자동 **설치**
- `nvm use 18` : **18** 버전 node **선택**
- `nvm use` : 현재 위치한 프로젝트의 `.nvmrc` 파일 내 명시되어있는 node 버전 자동 **선택**

---

### .nvmrc

특정 Node.js 버전을 이 프로젝트에서 사용해야 함을 명시합니다.

### .npmrc

프로젝트 내에서 강제적으로 특정 Node.js 버전을 사용하도록 설정합니다.

### Dependencies vs DevDependencies

- **Dependencies**: 배포용 라이브러리
- **DevDependencies**: 개발 중에만 필요한 라이브러리

### package.json vs package-lock.json

- `package.json`: 라이브러리의 집합과 스크립트를 정의합니다.
- `package-lock.json`: 라이브러리 설치 순서와 의존성 트리를 관리합니다.

---

## React 프로젝트의 주요 파일 설명

- **vite.config.js**: 개발용 설정과 운영용 번들링 설정을 관리합니다.
- **index.html**: React가 렌더링될 시작점이 되는 HTML 파일입니다.
  - `root`는 React 렌더링이 이루어지는 기준입니다.
- **main.js** / **main.jsx**: React 렌더링을 위한 JS 파일입니다.

---

## ESLint와 Prettier: 협업을 위한 필수 설정

### ESLint

컴파일 오류를 사전에 탐지하는 도구입니다.

### Prettier

협업 시 코드 스타일을 통일하기 위한 포매터입니다.

- **충돌 방지**: Prettier와 ESLint 간의 충돌을 방지하기 위해 관련 플러그인을 함께 설치하세요.
