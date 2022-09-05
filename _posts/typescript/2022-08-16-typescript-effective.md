---
layout: post
header-style: text
title: Typescript - 타입스크립트 알아보기
subtitle: 이펙티브 타입스크립트 1장
author: Juri
tags:
  - typescript
---

## 코드 생성과 타입이 관계없음을 이해하기

타입스크립트 컴파일러는 두 가지 역할을 수행한다.

> 첫 번째, 최신 타입스크립트/자바스크립트를 브라우저에서 동작할 수 있도록 구버전의 자바스크립트로 트랜스파일한다.

> 두 번째, 코드의 타입 오류를 체크한다.

이 두가지는 완벽히 독립적이다. 즉, **타입스크립트가 자바스크립트로 변환될 때 코드 내의 타입에는 영향을 주지 않는다. 또한, 그 자바스크립트의 실행 시점에도 타입은 영향을 미치지 않는다.**

### 🚨 타입 오류가 있는 코드도 컴파일이 가능하다.

컴파일은 타입 체크와 독립적으로 동작하기 때문에 타입 오류가 있는 코드도 컴파일이 가능하다. 타입스크립트 오류는 **경고**와 비슷하다. 문제가 될 만한 부분을 알려 주지만, 빌드를 멈추지는 않는다.

### 🚨 런타임에는 타입 체크가 불가능하다.

[참고 : 타입스크립트의 타입가드(타입 좁히기)](2022-05-26-type-guard.md)

```ts
interface Square {
  width: number;
}
interface Rectangle extends Square {
  height: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

`instanceof` 체크는 런타임에 일어나지만 Rectangle은 타입이기 때문에 런타임 시점에 아무런 역할을 할 수 없다. 실제로 자바스크립트로 컴파일되는 과정에서 모든 인터페이스, 타입, 타입 구문은 제거된다.
코드에서 다루는 shape 타입을 명확하게 하려면, **런타임에 타입 정보를 유지하는 방법**이 필요하다.

```ts
function calculateArea(shape: Shape) {
  if ("height" in shape) {
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

속성 체크는 런타임에 접근 가능한 값에만 관련되지만 타입 체커 역시 shape의 타입을 Rectangle로 보정해 주기 때문에 오류가 사라진다.

```ts
interface Square {
  kind: "square";
  width: number;
}
interface Rectangle {
  kind: "rectangle";
  height: number;
  width: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape.kind === "rectangle") {
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

런타임에 접근 가능한 타입 정보를 명시적으로 저장할 수도 있다.

또는, Square와 Rectangle을 클래스로 만들 수도 있다.

```ts
class Square {
  constructor(public width: number) {}
}
class Rectangle extends Square {
  constructor(public width: number, public height: number) {
    super(width);
  }
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```

인터페이스는 타입으로만 사용 가능하지만 Rectangle을 클래스로 선언하면 타입과 값으로 모두 사용할 수 있어 오류가 발생하지 않는다.

### 🚨 타입 연산은 런타임에 영향을 주지 않는다.

아래는 string 또는 number 타입인 값을 number로 정제하는 함수다.

```ts
function asNumber(val: number | string): number {
  return val as number;
}
```

이 코드는 아래와 같이 자바스크립트로 변환된다.

```js
function asNumber(val) {
  return val;
}
```

parameter로 들어온 val가 그대로 반환되는 코드로 아무런 정제 과정이 없다. `as number`는 타입 연산이고 런타임 동작에는 아무런 영향을 미치지 않는다.
값을 정제하기 위해서는 런타임의 타입을 체크해야 하고 자바스크립트 연산을 통해 변환을 수행해야 한다.

```ts
function asNumber(val: number | string): number {
  return typeof val === "string" ? Number(val) : val;
}
```

### 🚨 런타임 타입은 선언된 타입과 다를 수 있다.

다음 함수가 `console.log`까지 실행될 수 있을까?

```ts
fuction setLightSwitch(value: boolean) {
  switch(value){
    case true:
      turnLightOn();
      break;
    case false:
      turnLightOff();
      break;
    default:
      console.log('Everything is Alright!')
  }
}
```

`:boolean`은 타입이기 때문에 런타임에서 제거된다. value에 문자열이 들어와도 런타임에서는 함수가 `turnLightOn`이 호출될 수도 있다.

### 🚨 타입스크립트 타입으로는 함수를 오버로드할 수 없다.

동일한 이름에 parameter만 다른 여러 버전의 함수를 허용하는 것을 **함수 오버로딩**이라고 한다. 타입스크립트에서는 타입과 런타임 동작이 무관해 함수 오버로딩이 불가능하다.

다음과 같은 중복된 함수 구현은 불가능하다.

```ts
function add(a: number, b: number) {
  return a + b;
}
function add(a: string, b: string) {
  return a + b;
}
```

하나의 함수에 대해 여러 개의 선언문을 작성할 수 있지만 타입 수준에서만 동작하며 구현체는 하나뿐이다.

```ts
// typescript
function add(a: number, b: number): number;
function add(a: string, b: string): string;

// javascript
function add(a, b) {
  return a + b;
}
```

위의 두 선언문은 자바스크립트로 변환되면서 제거되며 구현체만 남는다.

### 🚨 타입스크립트 타입은 런타임 성능에 영향을 주지 않는다.

타입과 타입 연산자는 자바스크립트 변환 시점에 제거되어 런타임의 성능에 영향을 주지 않는다.

## 구조적 타이핑에 익숙해지기

자바스크립트는 **덕 타이핑(duck typing)** 기반이다. (덕 타이핑이란, 동적 타이핑의 한 종류로 객체의 변수 및 메소드의 집합이 객체의 타입을 결정하는 것이다. "만약 어떤 새가 오리처럼 걷고 헤엄치고 꽥꽥거리는 소리를 낸다면 나는 그 새를 오리라고 부를것이다.")

만약 어떤 함수의 매개변수 값이 모두 주어진다면 그 값이 어떻게 만들어졌는지와 관계없이 사용한다. 타입스크립트는 이것을 그대로 모델링한다.

**예제 코드**

아래는 2D 벡터 타입과 벡터의 길이를 계산하는 함수이다.

```ts
interface Vector2D {
  x: number;
  y: number;
}

function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x * v.x + v.y * v.y);
}

interface NamedVector {
  name: string;
  x: number;
  y: number;
}
```

`NamedVector`는 number 타입의 x와 y 프로퍼티가 있기 때문에 `calculateLength` 함수로 호출이 가능하다.

```ts
const v: NamedVector = { name: "juri", x: 3, y: 3 };
calculateLength(v); // 5
```

각각 다른 인터페이스인 Vector2D와 NamedVector의 관계가 전혀 선언되지 않았음에도 불구하고 NamedVector를 위한 별도의 함수를 구현할 필요가 없는 것이다. 여기에서 문제가 발생하기도 한다.

아래는 3D 벡터 타입과 벡터의 길이를 1로 만드는 정규화 함수이다.

```ts
interface Vector3D {
  x: number;
  y: number;
  z: number;
}

function normalize(v: Vector3D) {
  const length = calculateLength(v);

  return {
    x: v.x / length,
    y: v.y / length,
    z: v.z / length,
  };
}
```

```ts
normalize({ x: 3, y: 4, z: 5 }); // { x: 0.6, y: 0.8, z: 1}
```

`calculateLength`는 2D 벡터로 연산을 하도록 선언되었는데도 왜 3D 벡터를 받는 데 문제가 없었을까?

타입스크립트 타입 시스템에서 타입은 타입의 확장에 열려(open)있다. 즉, 타입에 선언된 속성 외에 임의의 속성을 추가해도 오류가 발생하지 않는다는 것이다.

```ts
function calculateLengthL1(v: Vector3D) {
  let length = 0;

  for (const axis of Object.keys(v)) {
    const coord = v[axis];
    length += Math.abs(coord);
  }

  return length;
}
```

```bash
'string'은 'Vector3D'의 인덱스로 사용할 수 없기에 엘리먼트는 일시적으로 'any'타입입니다.
```

겉보기엔 문제가 없어보이지만 타입스크립트는 위와 같은 오류를 뱉는다. axis는 Vector3D의 key 중 하나이기 때문에 x, y, z 중 하나이며 이것들 모두 number인 값을 갖기 때문에 coord의 타입이 number일 것으로 예상된다.
하지만, 이전에 본 것처럼 타입스크립트는 열려있기 때문에 매개변수의 타입에 선언된 것과 다른 타입의 속성이 들어올 수 있고 이 때문에 `v[axis]`는 number라고 확정할 수 없는 것이다.

```ts
const vec3D = { x: 3, y: 4, z: 1, address: "123 Seoul" };
```

이것처럼 선언되지 않은 string 타입의 속성인 address가 매개변수에 포함되면 calculateLengthL1 함수는 정상적으로 NaN을 반환할 것이다.
따라서 상황에 맞게 다시 구현하자면 루프보다는 아래와 같이 모든 속성을 각각 더하는 것이 나을 수 있다.

```ts
function calculateLengthL1(v: Vector3D) {
  return Math.abs(v.x) + Math.abs(v.y) + Math.abs(v.z);
}
```

테스트를 작성할 땐 구조적 타이핑이 유리하다. 아래는 데이터베이스에 쿼리하고 결과를 처리하는 함수이다.

```ts
interface Author {
  first: string;
  last: string;
}
function getAuthors(database: PostgresDB): Author[] {
  const authorRows = database.runQuery(`SELECT FIRST, LAST FROM AUTHORS`);
  return authorRows.map((row) => ({ first: row[0], last: row[1] }));
}
```

`getAuthors` 함수를 테스트하기 위해서는 모킹한 PostgresDB를 생성해야 하지만 구조적 타이핑을 활용해 더 구체적인 인터페이스를 정의할 수 있다.

```ts
interface DB {
  runQuery: (sql: string) => any[];
}
function getAuthors(database: DB): Author[] {
  const authorRows = database.runQuery(`SELECT FIRST, LAST FROM AUTHORS`);
  return authorRows.map((row) => ({ first: row[0], last: row[1] }));
}
```

구조적 타이핑덕분에 PostgresDB가 DB 인터페이스를 어떻게 구현하는지 명확히 선언할 필요가 없다.

타입스크립트는 테스트 DB가 해당 인터페이스를 충족하는지 확인한다. 그리고 테스트 코드에는 실제 환경의 DB에 대한 정보가 불필요하다.
