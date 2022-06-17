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

### conditional type

```ts
type Check<T> = T extends string ? boolean : number;
type Type = Check<string>; //'boolean'
```

타입 Check는 T가 `string`을 상속하면 boolean 타입을, 그렇지않으면 number 타입을 갖는다. 따라서 T가 string인 타입 Type은 boolean 타입을 갖는 것을 확인할 수 있다.

```ts
type TypeName<T> = T extends string
    ? "string"
    : T extends number
    ? "number"
    : T extends boolean
    ? "boolean"
    : T extends undefined
    ? "undefined"
    : T extends Function
    ? "function"
    : "object";
```

자바스크립트의 삼항 조건 연산자를 중첩해서 사용한 type 선언이다. T가 어떤 것을 상속하느냐에 따라 타입을 결정한다. `string`을 상속하면 'string'타입을, `number`을 상속하면 'number'타입을 갖는다.

```ts
type T0 = TypeName<string>; //'string'
type T1 = TypeName<"abc">; //'string'
type T2 = TypeName<() => void>; //'function'
```

T에 `string`이 아닌 진짜 문자열이 들어와도 이 문자열은 string 을 상속하므로 'string' 타입을 갖는다. 함수가 들어오면 'function' 타입을 갖는다.

### Readonly type

위의 map type에서 살펴본 응용 타입들은 직접 구현할 필요없이 타입스크립트 개발자들이 미리 만들어놓았기 때문에 잘 가져다가 쓰기만 하면 된다.

```ts
type ToDo = {
    title: string;
    description: string;
};

const display = (todo: Readonly<ToDo>) => {
    todo.title = "something"; //error
};
```

함수 display의 parameter로 들어오는 todo는 읽기 전용 속성이기 때문에 수정할 수 없다.

### partial type

`Partial<T>` 를 사용해 optional한 타입을 만들 수 있다.

```ts
type ToDo = {
    title: string;
    description: string;
    label: string;
    priority: "high" | "low";
};

const updateTodo(todo: Todo, fieldsToUpdate: Partial<ToDo>) : ToDo => {
    return {
        ...todo,
        ...fieldsToUpdate
    }
}
```

함수 updateTodo의 매개변수 fieldsToUpdate는 ToDo의 Partial type으로 타입 ToDo의 프로퍼티를 optional하게 갖는다.

```ts
const todo: ToDo = {
    title: "let's study",
    description: "typescript",
    label: "study",
    priority: "high"
};

const updated = updateTodo(todo, { priority: "low" });
```

updated를 확인하면 priority가 low로 바뀌는 것을 확인할 수 있다.

### Pick, Omit type

video 타입에서 특정 프로퍼티만 취급하고 싶을 때 pick과 omit을 사용할 수 있다. video 타입에서 'id'와 'title' 프로퍼티만 갖는 타입을 선언하고 싶다고 하자.

```ts
type Video = {
    id: string;
    title: string;
    url: string;
    data: string;
};

type VideoMetadata1 = Pick<Video, "id" | "title">;
type VideoMetadata2 = Omit<Video, "url" | "data">;
```

Pick은 취급할 프로퍼티를 지정하고 이와 반대로 Omit은 제외할 프로퍼티를 지정한다.

### Record type

```ts
type PageInfo = {
    title: string;
};
type Page = "home" | "about" | "contact";

const nav: Record<Page, PageInfo> = {
    home: { title: "home" },
    about: { title: "about" },
    contact: { title: "contact" }
};
```

간단하게 생각하면 nav는 Page 타입을 key 값으로, PageInfo를 value 값으로 갖는 타입이라고 할 수 있다.

```ts
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```

여기에서 keyof any는 ' string | number | symbol '을 의미한다.
