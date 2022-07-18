---
layout: post
header-style: text
title: Typescript - 타입선언과 @types
subtitle: 이펙티브 타입스크립트 6장
author: Juri
tags:
  - typescript
---

외부 API를 사용하다보면 타입 선언이 없는 js 기반의 API를 쓸 일이 있다. 자체적으로 타입을 선언해보기도 하고 typescript 커뮤니티에서 제공하는 @types 라이브러리를 사용해보기도 하면서 삽질도 많이 하고 구글링도 많이 한 결과, 타입스크립트의 근본을 더 공부할 필요성을 느껴 책을 뒤져보았다.

이번 포스팅에서는 타입스크립트에서 의존성이 어떻게 동작하는지 알아보고 의존성 관리를 하다가 맞닦뜨릴 수도 있는 문제의 해결방법을 찾아보도록 한다.

### devDependencies에 typescript와 @types 추가하기

npm은 3가지 종류의 의존성을 구분해서 관리하며 각각의 의존성은 `package.json`파일 내에서 확인할 수 있다.

- dependencies
  현재 프로젝트를 실행하는 데 필수적인 라이브러리를 포함한다. 프로젝트를 npm에 공개해 다른 사용자가 해당 프로젝트를 설치한다면 dependencies에 포함된 라이브러리가 설치된다.
- devDependencies
  현재 프로젝트를 개발하고 테스트하는 데 사용되지만 런타임에는 필요없는 라이브러리를 포함한다. 프로젝트를 npm에 공개해 다른 사용자가 해당 프로젝트를 설치한다면 devDependencies에 포함된 라이브러리들은 제외된다.
- peerDependencies
  런타임에 필요하지만 의존성을 직접 관리하지 않는 라이브러리를 포함한다.

타입정보는 런타임에 존재하지 않기 때문에 타입스크립트와 관련된 라이브러리는 일반적으로 `devDependencies`에 속한다.

타입스크립트 프로젝트에서 공통적으로 고려해야 하는 의존성으로 두가지가 있다.

1. 타입스크립트 자체 의존성
2. 타입 의존성

### 타입스크립트 자체 의존성

프로젝트 팀원 모두가 항상 동일한 버전의 타입스크립트를 설치하리라는 보장이 없고 프로젝트 셋업시 별도의 단계가 필요해 타입스크립트를 시스템 레벨로 설치하는 것 보다 `devDependencies`에 넣는 것이 좋다. 커멘드 라인에서 `npx`을 사용해 `devDependencies`에 설치된 타입스크립트 컴파일러를 실행할 수 있다.

### 타입 의존성

사용하려는 라이브버리에 타입 선언이 포함되어 있지 않아도 [DefinitelyTyped](https://github.com/DefinitelyTyped)에서 타입 정보를 얻을 수 있다. DefinitelyTyped의 타입 정의들은 npm 레지스트리의 `@types` 스코프에 공개된다. @types 라이브러리는 타입 정보만 포함하므로 원본 라이브러리 자체가 dependencies에 있더라도 @types 의존성은 devDependencies에 있어야 한다.
