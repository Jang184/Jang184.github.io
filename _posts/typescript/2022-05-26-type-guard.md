---
layout: post
header-style: text
title: Typescript - Type Guards 타입 가드
author: Juri
tags:
    - typescript
---

참고 : [typescript 핸드북](https://typescript-kr.github.io/pages/advanced-types.html#%EC%82%AC%EC%9A%A9%EC%9E%90-%EC%A0%95%EC%9D%98-%ED%83%80%EC%9E%85-%EA%B0%80%EB%93%9C-user-defined-type-guards)

```ts
interface Bird {
    fly();
    layEggs();
}
interface Fish {
    swim();
    layEggs();
}
function getSmallPet(): Fish | Bird {
    // ...
}

let pet = getSmallPet();
```

유니온 타입은 값의 타입이 겹쳐질 수 있는 상황에 유용하지만 아래와 같이 각 프로퍼티들에 접근하는 것은 불가능하다. 유니언 타입의 모든 구성 성분을 가지고 있다고 보장되는 멤버에만 접근할 수 있다. 아래와 같이 각 인터페이스에 있는 `swim`과 `fly`에 접근할 수 없는 반면 공통으로 갖고 있는 프로퍼티인 `layEggs`에는 접근이 가능하다.

```ts
pet.layEggs(); // 성공
pet.swim(); // 오류

if (pet.swim) {
    pet.swim();
} else if (pet.fly) {
    pet.fly();
}
```

코드를 동작하게 하기 위해서 타입 단언을 사용할 수 있다.

```ts
if ((pet as Fish).swim) {
    (pet as Fish).swim();
} else if ((pet as Bird).fly) {
    (pet as Bird).fly();
}
```

## User-Defined 타입가드

타입 가드는 스코프 안에서의 타입을 보장하는 런타임 검사를 수행한다는 표현식이다. `{parameter} is (Type)` 형식으로 어떠한 인자명은 어떠한 타입이다라고 명시해주는 역할을 한다.

### type predicates 사용하기

타입 가드를 정의하기 위해 반환 타입이 type predicate인 함수를 정의하면 된다. 아래 함수는 인자로 Fish 나 Bird 타입 두가지를 가질 수 있는 값을 전달하면 함수에서 Fish 타입일 경우에만 true로 전달한다.

```ts
function isFish(pet: Fish | Bird): pet is Fish {
    return (pet as Fish).swim !== undefined;
}
```

`isFish`가 변수와 함께 호출될 때마다, typescript는 기존 타입과 호환된다면 그 변수를 특정 타입으로 제한할 것이다.

```ts
if (isFish(pet)) {
    pet.swim();
} else {
    pet.fly();
}
```

### in operator 사용하기

`n in x` 에서 `n`은 문자열 리터럴 혹은 문자열 리터럴 타입이고 `x`는 유니언 타입이다. true 분기에서 프로퍼티 `n`을 갖는 타입으로 타입을 좁힐 수 있다.

```ts
function move(pet: Fish | Bird) {
    if ("swim" in pet) {
        return pet.swim();
    }
    return pet.fly();
}
```

## type of 타입가드

```ts
function isNumber(x: any): x is number {
    return typeof x === "number";
}

function isString(x: any): x is string {
    return typeof x === "string";
}

function padLeft(value: string, padding: string | number) {
    if (isNumber(padding)) {
        return Array(padding + 1).join(" ") + value;
    }
    if (isString(padding)) {
        return padding + values;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}
```

typescript는 `typeof`를 type guard로 인식해 **typeof x === "number"**로 함수를 추상할 필요가 없다.

```ts
function padLeft(value: string, padding: string | number) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}
```

`typeof`와 함께 사용할 수 있는 타입으로 number, string, boolean, symbol이 있다.

### instanceof 타입가드

`instanceof` 타입가드는 생성자 함수를 사용해 타입을 좁히는 방법이다. `instanceof`의 오른쪽은 생성자 함수여야 한다.

```ts
interface Padder {
    getPaddingString(): string;
}

class SpaceRepeatingPadder implements Padder {
    constructor(private numSpaces: number) {}
    getPaddingString() {
        return Array(this.numSpaces + 1).join(" ");
    }
}

class StringPadder implements Padder {
    constructor(private value: string) {}
    getPaddingString() {
        return this.value;
    }
}

function getRandomPadder() {
    return Math.random() < 0.5
        ? new SpaceRepeatingPadder(4)
        : new StringPadder("  ");
}

let padder: Padder = getRandomPadder();

if (padder instanceof SpaceRepeatingPadder) {
    padder;
}
if (padder instanceof StringPadder) {
    padder;
}
```

`padder`의 타입은 SpaceRepeatingPadder \| StringPadder 이며 `instanceof`의 오른쪽에 있는 생성자 함수를 타입으로 좁혀진다.
