---
layout: post
header-style: text
title: Typescript - Advanced Type
subtitle:
author: Juri
tags:
    - typescript
---

## type alias vs interface

헷갈리는 `type` 과 `interface`! 언제 무엇을 쓰는 것이 좋을까?

```ts
type PositionType = {
    x: number;
    y: number;
};

interface PositionInterface {
    x: number;
    y: number;
}
```

---

type과 interface를 사용해 object와 class를 특정한 구성으로 규정할 수 있다.

**object**

```ts
const obj1: PositionType = {
    x: 1,
    y: 1
};

const obj2: PositionInterface = {
    x: 1,
    y: 1
};
```

**class**

```ts
class Pos1 implements PositionType {
    x: number;
    y: number;
}
class Pos2 implements PositionInterface {
    x: number;
    y: number;
}
```

type과 interface 모두 **확장**이 가능하다.

```ts
interface ZPositionInterface extends PositionInterface {
    z: number;
}
type ZPositionType = PositionType & { z: number };
```

기존의 type과 interface를 확장해 새로운 type과 interface를 만들었다.

interface는 중복으로 사용되면 합쳐진다. 두번 사용된 PositionInterface를 만족하는 object는 x,y 속성뿐만 아니라 z 속성을 가져야 한다. 반면에, type은 중복으로 사용하면 에러가 발생한다.

```ts
interface PositionInterface {
    x: number;
    y: number;
}

interface PositionInterface {
    z: number;
}

const obj: PositionInterface = {
    x: 1,
    y: 1,
    z: 1
};
```

type은 **computed properties**를 사용할 수 있다. 아래와 같이 type의 key값을 사용해 새로운 type을 선언할 수 있다. 또한, interface에서는 불가능한 **Union Type**을 사용할 수 있다.

```ts
type Person = {
    name: string;
    age: number;
};

type Name = Person["name"]; // string

type Animal = "dog" | "cat";
```

## Utility Type

### index type

```ts

```
