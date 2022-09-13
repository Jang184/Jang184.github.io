---
layout: post
header-style: text
title: Typescript - 타입과 인터페이스의 차이점 알기
subtitle: 이펙티브 타입스크립트 2장
author: Juri
tags:
  - typescript
---

타입스크립트에서 명명된 타입을 정의하는 방법은 두 가지가 있다. 이외에도 명명된 타입을 정의할 때 인터페이스 대신 **클래스**를 사용할 수도 있지만 클래스는 값으로도 쓰일 수 있는 **자바스크립트 런타임의 개념**이다.

1.  타입 사용하기

```ts
type TState = {
  name: string;
  capital: string;
};
```

2. 인터페이스 사용하기

```ts
interface IState {
  name: string;
  capital: string;
}
```

먼저, 타입과 인터페이스의 비슷한 점에 대해서 알아보자.

### 추가 속성 할당

TState와 IState에 `name`과 `capital`을 제외한 추가 속성을 할당하면 오류가 발생한다.

### 인덱스 시그니처

타입과 인터페이스 모두 사용이 가능하다.

```ts
type TDict = { [key: string]: string };
interface IDict {
  [key: string]: string;
}
```

### 함수 타입

타입과 인터페이스 모두 사용이 가능하다.

```ts
type TFn = (x:number) => string;
type IFn {
    (x:number):string;
}

const SomeTFn: TFn = x => '' + x;
const SomeIFn: IFn = x => '' + x;
```

### 제네릭

타입과 인터페이스 모두 가능하다.

```ts
type TPair<T> = {
  first: T;
};
interface IPair<T> {
  first: T;
}
```

### 확장

인터페이스는 타입을 확장할 수 있고 반대로 타입은 인터페이스를 확장할 수 있다.

```ts
interface IStateWithPop extends TState {
  population: number;
}
type TStateWithPop = IState & { population: number };
```

여기에서 `IStateWithPop`과 `TStateWithPop`은 동일하다. 대신 인터페이스는 유니온 타입같은 복잡한 타입을 확장할 수는 없다. 복잡한 타입의 확장은 타입과 `&`을 사용한다.

### 클래스 구현 (implements)

타입과 인터페이스 모두 사용이 가능하다.

```ts
class StateT implements TState {
  name: string = "";
  capital: string = "";
}
class StateI implements IState {
  name: string = "";
  capital: string = "";
}
```
