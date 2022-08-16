---
layout: post
header-style: text
title: Typescript - 타입스크립트 알아보기
subtitle: 이펙티브 타입스크립트 1장
author: Juri
tags:
  - typescript
---

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
