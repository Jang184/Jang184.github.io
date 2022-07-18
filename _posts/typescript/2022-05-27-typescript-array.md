---
layout: post
header-style: text
title: Typescript - Array 인터페이스
subtitle: typescript 소스코드 뜯어보기
author: Juri
tags:
  - typescript
---

microsoft의 [typescript](github.com/microsoft/TypeScript) 오픈소스 코드를 뜯어보고 array 인터페이스를 분석해보자.

## interface Array

먼저, Array는 다양한 타입을 다루기 때문에 **제네릭**을 이용해 유동적으로 타입을 정해줄 것이다.

```ts
// lib.es5.d.ts

interface Array<T> {
  length: number;

  // ...
}
```

`length`는 array 안의 요소의 개수를 나타내는 프로퍼티로 number 타입을 갖는다.

### method of Array

인터페이스는 일종의 규정으로 Array 타입이 필수로 가져야 할 요소들에 대해 정해놓은 것이라고 생각하면 된다.

```ts
{
    pop(): T | undefined;
}
```

`pop`은 array의 마지막 요소를 제거함과 동시에 반환한다. array가 비어있으면 아무것도 반환하지 않으므로 반환값은 T 아니면 undefined 타입을 갖는다.

```ts
{
    push(...items: T[]): number;
}
```

`push`는 array의 끝에 새 요소를 추가하고 만든 새로운 array의 `length`를 반환하므로 number 타입을 갖는다. spread operator로 array를 전개해 다수의 parameter를 받을 수 있다.

```ts
{
    concat(...items: ConcatArray<T>[]): T[];
}

interface ConcatArray<T> {
    readonly length: number;
    readonly [n:number]: T;
    join(separator?: string): string;
    slice(start?: number, end?: number): T[];
}
```

`concat`은 ConcatArray 타입 요소들의 array를 spread operator를 사용해 parameter로 받아 array로 반환하는 것으로 예상된다.

ConcatArray 인터페이스를 찾아본다. `length`라는 프로퍼티와 인덱스 시그니처를 갖는다. 인덱스 시그니처는 ConcatArray가 number로 색인화되면 T 타입을 반환할 것이다.

```ts
type Animal<T> = {
  [key: string]: T;
};

let animal: Animal<string> = {
  dog: "bom",
  cat: "gaeul",
};
```

인덱스 시그니처는 위의 코드와 같이 object의 Key, Value 의 타입을 정한다고 생각하면 된다. 인덱스 시그니처를 number 타입으로 선언해 obj[0], obj[1] 과 같은 방식으로 할당이 가능해진다.

이어서, `join` 메서드와 `slice` 메서드가 보인다. `join`은 separator라는 string 타입의 option parameter를 받아 string 타입의 값을 반환한다. `slice`는 start와 end라는 number 타입의 optional parameter를 받아 T 타입 요소들의 배열을 반환한다.

`concat`은 2개 이상의 array를 합치며 기존의 array의 변화없이 새로운 array를 반환한다.

만약, string으로 이루어진 array에 concat 메소드를 사용한다면

```ts
{
    concat(...items: (string | ConcatArray<string>)[]): string[];
}
```

parameter로 string 혹은 string 의 array를 넣을 수 있으며 결과값은 string의 array가 되는 것이다.

```ts
const arr1 = [1, 2, 3];
const arr2 = ["a", "b", "c"];

const result = arr1.concat(arr2);

console.log(result); // [ 1, 2, 3, 'a', 'b', 'c' ]

const arr3 = "123";
const arr4 = "abc";

const result2 = arr3.concat(arr4);

console.log(result2); // '123abc'
```

```ts
{
    shift(): T | undefined;
}
```

`shift`는 array의 첫번째 요소를 제거함과 동시에 반환한다. array가 비어있으면 `undefined`를 반환한다.

```ts
{
    sort(compareFn?: (a: T, b: T) => number): this;
}
```

`sort`는 T 타입의 인자를 받아 number 를 반환하는 compareFn이라는 함수를 받아 array를 정렬한다.

```ts
{
    splice(start: number, deleteCount?:number): T[];
}
```

`splice`는 start번째부터 deleteCount만큼의 개수에 해당하는 요소를 삭제하고 남는 T타입의 요소들의 배열을 반환한다.

```ts
{
    unshift(...items: T[]): number;
}
```

`unshift`는 T타입 요소들의 배열을 펼쳐 배열의 첫부분에 삽입하고 최종 길이를 반환한다.

```ts
{
    indexOf(searchElement: T, fromIndex?: number): number;
}
```

`indexOf`는 searchElement의 값과 일치하는 첫번째 인덱스를 반환한다. 만약 일치하는 값이 없으면 -1을 반환한다. fromIndex 값을 입력하면 검색을 시작하는 인덱스를 지정할 수 있다.

```ts
{
    every<S extends T>(predicate: (value: T, index: number, array: T[]) => value is S, thisArg?: any): this is S[];
}
```

`every`는 배열 내 모든 요소들이 주어진 조건을 만족하는지 검사하는 메소드이다. predicate는 value, index, array를 파라미터로 받는 함수로 배열의 모든 요소에 대해 어떤 요소가 false를 반환할 때까지 혹은 배열의 마지막요소까지 실행된다.

S타입은 T를 제약조건으로 가지며 value is S는 스코프내에서 타입을 보장하는 런타임 검사를 수행한다는 type predicate 표현식으로 value를 S 타입으로 제한할 것이다.

```ts
{
    some(predicate: (value: T, index: number, array: T[]) => unknown, thisArg?: any): boolean;
}
```

`some`은 배열의 모든 요소가 predicate를 만족해야 하는 것과 달리, 요소 중 하나라도 만족하면 true를 반환한다.

```ts
{
    forEach(callbackfn: (value: T, index: number, array: T[]) => void, thisArg?: any): void;
}
```

`forEach`는 배열의 모든 요소에 callbackfn에 들어온 액션을 수행한다. 아무것도 반환하지 않는다.

```ts
{
    map<U>(callbackfn: (value: T, index: number, aaray: T[]) => U, thisArg?: any): U[];
}
```

`map`은 forEach와 동일하게 배열의 모든 요소에 callbackfn에 들어온 액션을 수행하고 그 결과(U)를 배열 형태로 반환(U[])한다.

```ts
{
    filter<S extends T>(predicate: (value: T, index: number, array: T[]) => value is S, thisArg?: any): S[];
}
```

`filter`는 predicate에 들어온 함수를 만족하는 배열의 요소만 반환한다.
