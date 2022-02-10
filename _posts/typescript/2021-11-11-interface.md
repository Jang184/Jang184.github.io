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

### 인터페이스 확장하기