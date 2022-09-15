---
layout: post
title: Common JS vs ES Moudles
subtitle: NodeJS에서 모듈 임포트하기
header-style: text
author: Juri
tags:
  - test
---

참고 : [nodejs-modules-imports](https://reflectoring.io/nodejs-modules-imports/)

모듈 시스템을 통해 코드를 파트별로 나누고 다른 개발자가 작성한 코드를 가져와서 사용할 수 있다. 최근, `ES modules`라는 새로운 모듈 시스템이 NodeJS에 추가됐다고 하니 기존의 `Common JS`와 어떻게 다른지 살펴보도록 하자.

## 왜 Node JS에서 모듈 시스템이 필요할까?

방대한 양의 코드를 파일별로 나누면 코드를 재사용하고 구조화할 수 있다. 또한, 어떤 파일의 어떤 부분에 있는 코드가 접근가능한 지 관리할 수 있다.

자바스크립트로 쓰는 모든 코드는 기본으로 전역범위를(global) 갖는다. 복잡한 애플리케이션을 자바스크립트로 작성하면서 이런 점이 큰 문제로 대두됐고 NodeJS 개발자들은 기본 모듈시스템인 **CommonJS**를 도입했다.

## CommonJS : NodeJS의 기본 모듈 시스템

NodeJS는 각각의 `.js`파일들을 분리된 CommonJS 모듈로 다룬다. 이는 변수, 함수, 클래스 등이 기본적으로 다른 파일에서 사용 불가능함을 의미한다. 모듈 시스템에 코드의 어떤 부분이 export 될 지 명시해줘야 한다.

CommonJS 모듈은 `module.exports` 혹은 `exports`를 통해 export 될 부분을 명시한다. import 하고싶은 곳에서 `require()`을 사용해 export된 파일을 사용할 수 있다.

### NodeJS 모듈 import하기

```js
const http = require("http");
```

`http` 모듈은 NodeJS의 내부모듈로 `require()` 함수 안에 `http` 문자열을 넣어 import했다. `require("http")`로 불러온 내용은 `http`라는 지역 변수에 담기며 일반적인 함수의 호출과 동일하다. `http`가 아닌 원하는 이름의 변수로 사용할 수도 있다.

### NPM Dependencies import하기

NodeJS의 내부 모듈을 import하는 것과 동일하게 NPM 패키지 모듈을 import할 수 있다. (NPM 패키지 모듈은 `node_modules` 폴더에 있는 모듈로 npm install 과정이 필요하다.)

```js
const chalk = require("chalk");
```

### 자신이 작성한 코드를 export하고 import하기

자신이 작성한 코드를 import 하려면 먼저 어떤 파일의 어떤 코드가 다른 모듈에서 접근이 가능하게 할 지 CommonJS에 명시해준다.

```js
// logger.js
const chalk = require("chalk");

exports.logInfo = function (message) {
  console.log(chalk.blue(message));
};

exports.logError = function logError(message) {
  console.log(chalk.red(message));
};

exports.defaultMessage = "Hello World";
```

`logInfo()`와 `logError()`를 `exports` 객체에 추가해 다른 모듈에서 접근이 가능하도록 해준다. 함수뿐만 아니라 문자열도 `exports`객체에 추가할 수 있다.

```js
const logger = require("./logger");

logger.logInfo(`${logger.defaultMessage} printed in blue`);
logger.logError("some error message printed in red");
```

`require()`에 import하고자 하는 파일의 상대경로를 넣어 `exports` 객체를 불러오는 것을 볼 수 있다.

### exports 대신 module.exports 사용해 import하기
