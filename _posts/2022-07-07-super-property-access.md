---
layout: post
title: faster super property access
subtitle: 자바스크립트 V8 엔진 버전 9.0 업데이트 된 것은 무엇일까?
header-style: text
author: Juri
tags:
  - V8 engine
---

이전 포스팅에서 소개했던 Node.js 16의 새로운 피쳐 중 하나가 V8 엔진 버전 9.0의 업데이트였습니다. V8 엔진이 버전업 되면서 바뀐 점 중 하나인 **super property**로의 접근이 어떻게 빨라졌는지에 대해 정리해보았습니다.

![](/img/in-post/v8tweet.png)

참고 :

- [v8 블로그 - 버전9 업데이트 내용](https://v8.dev/blog/v8-release-90)
- [faster super property access 번역](https://ui.toast.com/weekly-pick/ko_20210224)
- [V8 엔진의 Shape와 Inline cache system](https://shlrur.github.io/javascripts/javascript-engine-fundamentals-shapes-and-Inline-caches/)

## 인트로

super property (`super.x`)로의 접근은 런타임 실행으로 구현됩니다. V8 버전 9.0부터 인라인 캐시 시스템 (IC system)을 재사용해 런타임 실행없이 최적화된 코드를 생성해 super property에 접근할 수 있게 되었습니다.

![](/img/in-post/super-property.png)

## super keyword

| 💡 `super` 를 사용해 부모 객체의 프로퍼티와 함수에 접근할 수 있다.

```js
class A {}
A.prototype.x = 100;

class B extends A {
  m() {
    return super.x;
  }
}
const b = new B();
b.m(); // 100
```

클래스 B의 인스턴스 b는 왜 클래스 A의 프로퍼티 x 값을 가져올 수 있을까?

### 자바스크립트 prototype의 상속

참고 : [코어자바스크립트 - 프로토타입](./javascript/2022-05-11-core5.md)

![](/img/in-post/prototype2.jpg)

- 어떤 생성자 함수를 new 연산자와 함께 호출하면
- Constructor에서 정의된 내용을 바탕으로 새로운 인스턴스가 생성된다.
- 이 때 instance에는 \_\_proto\_\_라는 프로퍼티가 자동으로 부여되는데
- 이 프로퍼티는 Constructor의 prototype이라는 프로퍼티를 참조한다.

prototype 객체 내부에는 인스턴스가 사용할 메소드를 저장합니다. 그러면 인스턴스에서도 \_\_proto\_\_ 를 통해 이 메소드에 접근할 수 있게 됩니다.

![](/img/in-post/prototype3.png)

**클래스 A**와 클래스 A를 확장(extends)해 만든 **클래스 B**의 관계를 자세하게 들여다 봅니다. A의 prototype과 B의 prototype의 \_\_proto\_\_가 같습니다. 콘솔창에 확인해보면 실제로 값이 동일함을 알 수 있습니다.

![](/img/in-post/prototype4.png)

클래스 B는 클래스 A를 확장하면서 자신의 \_\_proto\_\_에 클래스 A의 prototype을 저장함으로써 인스턴스 b가 클래스 A의 프로퍼티와 메소드에 접근할 수 있게 됩니다. 결과적으로 인스턴스 b의 \_\_proto\_\_ 내부의 \_\_proto\_\_에 클래스 A의 프로퍼티와 메소드가 복사됩니다.

### 프로토타입 체이닝

어떤 데이터의 \_\_proto\_\_ 내부에 다시 \_\_proto\_\_ 가 연쇄적으로 이어진 것을 **프로토타입 체인**이라고 합니다. 그리고 이 체인을 따라가며 검색하는 것을 **프로토타입 체이닝**이라고 합니다.

![](/img/in-post/prototype-chain.png)

이 체인을 통해 인스턴스 b는 클래스 B에 정의된 모든 속성에 접근합니다.

super property access는 홈 객체 (메소드가 정의된 객체, b의 경우 B.prototype)의 \_\_proto\_\_ 부터 조회가 시작되어 프로토타입 체인을 거슬러 올라갑니다.

## How to access object property in Javascript

| 💡 자바스크립트 엔진은 ICs를 사용해 객체에서 프로퍼티를 찾을 수 있는 위치에 대한 정보를 암기하여 높은 cost를 갖는 조회 횟수를 줄입니다.

![](/img/in-post/shape1.png)

`object.y` 에 접근한다고 할 때, JS 엔진은 Object에서 key `y` 를 찾아 해당 property attribute들을 로드한 후 `[[Value]]` 를 반환합니다. 이 property attribute는 메모리의 어느 곳에 저장될까요?

나중에 이 shape의 object를 더 사용하게 되면, property 이름이 동일한 object가 반복되므로 object 자체에 저장하는 것은 낭비가 됩니다. 그러므로 JS 엔진은 object의 shape를 별도로 저장합니다.

### Shape

자바스크립트 엔진은 shape를 사용해 객체의 프로퍼티에 접근하는 것을 최적화할 수 있습니다.

```js
const object1 = { x: 1, y: 2 };
const object2 = { x: 3, y: 4 };
```

객체의 구조와 key값이 일치하는 object1과 object2는 구조가 같다고 말할 수 있습니다.

![](/img/in-post/shape2.png)

Shape는 `[[Value]]` 를 제외한 모든 property 이름과 attribute를 포함합니다. 대신 shape는 JSObject에 있는 value의 offset값을 갖고 있어 JS엔진이 value의 위치를 알 수 있습니다.

같은 shape를 갖는 모든 JSObject는 같은 shape instance를 가리키므로 모든 JSObject는 고유한 value만 저장하면 됩니다.

### Inline cache

![](/img/in-post/shape3.png)
![](/img/in-post/shape4.png)

다음 명령문을 실행할 때 IC는 shape만 비교하면 되며, 이전과 같다면 저장되어 있는 offset을 보고 value를 가져오면 됩니다.

JS엔진이 IC가 이전에 기록한 shape의 object를 볼 경우, 더이상 property 정보에 접근할 필요가 없습니다.

## super faster super

빠른 property access를 위해 **인라인 캐시 시스템(IC system)**이 개발되었습니다. V8은 이미 빠른 property access가 가능하게 개량되었지만 super property 와 normal property 의 접근에 다른 몇가지(조회시작객체와 수신자가 다른 점)가 있어 생기는 버그를 수정하였습니다.

또한, 새로운 Ignition 바이트 코드 `LdaNamedPropertFromSuper` 을 추가해 인터프리터 모드에서도 IC system을 사용할 수 있게 되어 최적화된 코드를 생성할 수 있게 되었습니다. 이 새로운 바이트코드로 추가한 새로운 IC인 **LoadSuperIC**는 자신이 본 조회시작 객체의 모습과 그와 비슷한 모습의 객체에서 속성을 어떻게 불러왔는 지 기억해둡니다.
