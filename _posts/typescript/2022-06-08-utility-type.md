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

위에서 말한 것과 같이 type의 key값을 사용해 새로운 type을 선언할 수 있으며 key값 자체를 type으로 선언할 수도 있다. 아래의 **Keys**는 type Person의 key값인 `name`, `age` (문자열 그대로)만 값으로 갖도록 허용한다.

```ts
type Person = {
    name: string;
    age: number;
    gender: "male" | "female";
};

type Name = Person["name"];
type Keys = keyof Person; // "name" | "age"

const key: Keys = "name";
```

다른 type을 만들 때 다른 type의 key값의 타입을 가져올 수도 있다.

```ts
type Animal = {
    name: string;
    gender: Person["gender"];
};
```

### map type

```ts
type Video = {
    title: string;
    author: string;
};

type VideoOptional = {
    title?: string;
    author?: string;
};

type VideoReadOnly = {
    readonly itle: string;
    readonly author: string;
};
```

`Video`라는 타입이 필요해서 생성했다. 이후 `Video`의 프로퍼티가 없어도 되는 **optional**한 타입과 **readonly** 타입이 필요해졌다. `VideoOptional`과 `VideoReadOnly`와 같이 새로 타입을 생성했으나 `Video` 타입이 수정될 때마다 관련된 모든 타입들을 수정해야만 한다.

이 때, 이런 번거로움을 피하기 위해 아래와 같은 방법을 사용할 수 있다.

**Optional**

```ts
type Optional<T> = {
    [P in keyof T]?: T[P];
};
```

`[]`가 T의 모든 key값을 순회한다고 생각하면 된다. 즉, T의 key값인 P는 `?`가 붙어 optional property가 되고 그 P가 갖는 타입(`T[P]`)을 그대로 갖는다.

**Readonly**

```ts
type ReadOnly<T> = {
    readonly [P in keyof T]: T[P];
};
```

optional과 마찬가지로 T의 key값인 P 에 `readonly`를 붙여 읽기전용 속성으로 만들고 그 P가 갖는 타입을 그대로 갖는다.

```ts
const videoOptional: Optional<Video> = {};

const videoReadOnly: Readonly<Video> = {
    title: "tomboy",
    author: "idle:
}
videoReadOnly.title = "my bag" // error
```

`Optional<Video>` 타입은 모든 프로퍼티가 optional이므로 빈 객체일 수 있고 `Readonly<Video>` 타입은 모든 프로퍼티가 readonly이므로 프로퍼티 값을 변경할 수 없다.

```ts
type Proxy<T> = {
    get(): T;
    set(value: T): void;
};
type Proxify<T> = {
    [P in keyof T]: Proxy<T[P]>;
};
```

위의 코드는 typescript 공식 홈페이지에 있는 예제 코드로 key값의 타입을 한번더 다른 타입으로 감싸고 있다.
