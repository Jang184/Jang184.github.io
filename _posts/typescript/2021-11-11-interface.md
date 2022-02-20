---
layout: post
header-style: text
title: Typescript - 인터페이스(interface)를 알아보자
author: Juri
tags:
    - typescript
---

[참고: 코딩앙마](https://youtube.be/OIMPLNICzoc)

타입스크립트 인터페이스 활용하기
----

### 인터페이스

```ts
type extension = 'jpeg'|'png'

Interface Image {
    url: string;
    size?: number;
    extension: extension;
}
```

object의 타입을 만든다고 생각하면 쉽다. 위의 Image 인터페이스를 타입으로 갖는 객체는 `url`, `size`, `extension` 세 가지의 파라미터를 가질 수 있다. `?`가 붙은 `size`는 옵션 파라미터로 값을 갖지 않아도 에러가 발생하지 않는다. `extension`은 위에 정의한 extension 타입이므로 값으로 jpeg 혹은 png를 입력할 수 있다.

```ts
let image: Image = {
    url: '/myService/img',
    size: 350,
    extension: 'jpeg'
}
```

타입을 자세하게 지정하면 나중에 코드양이 방대해졌을 때 타입 오류를 쉽게 찾을 수 있다.

### 인터페이스로 함수 정의하기

```ts
interface Add {
    (num1:number, num2:number): number;
}

const add: Add = (x, y) => {
    return x + y;
```
인터페이스를 사용해 함수의 `매개변수` 타입과 `결과값` 타입을 지정할 수 있다.

### 인터페이스 확장하기 - implements, extends

**implemetns**

```ts
interface Car {
    color: string;
    wheels: number;
    start(): void;
}

class Bmw implements Car {
    color;
    wheels = 4;
    constructor(c: string){
        this.color = c;
    }
    start(){
        console.log("go");
    }
}
```
클래스 BMW는 인터페이스 Car에서 지정한 타입값을 가져와 사용한다. 만약 인터페이스 Car의 확장 없이 클래스 BWM의 타입값을 지정한다면 아래와 같이 작성한다.

```ts
class BMW {
    color: string;
    wheels: number;
    constructor(c: string){
        this.color = c;
    }
    start(){
        console.log("go")
    }
}
```
**extends**

```ts
intercace Benz extends Car {
    door: number;
    stop(): void;
}

const benz : Benz = {
    color: "black",
    wheels : 4,
    start(){}
    door: 5,
    stop(){
        console.log("stop")
    }
}
```
다른 인터페이스의 타입값을 가져와서 합칠 수 있다. 위의 코드에서 인터페이스 Benz에 인터페이스 Car를 가져와서 확장해 사용히는 것을 확인할 수 있다.

```ts
interface Car {
    color: string;
    wheels: number;
    star(): void;
}

interface Toy {
    name: string;
}

interface CarToy extends Car, Toy {
    price : number;
}
```
`extends`를 사용하면 여러 개의 인터페이스를 합칠 수도 있다.