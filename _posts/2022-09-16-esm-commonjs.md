---
layout: post
title: Common JS vs ES Moudles
subtitle: NodeJS에서 모듈 임포트하기
header-style: text
author: Juri
tags:
  - CJS
  - ESM
---

참고 : [nodejs-modules-imports](https://reflectoring.io/nodejs-modules-imports/)

참고의 글을 번역, 정리한 글입니다.

---

모듈 시스템을 통해 코드를 파트별로 나누고 다른 개발자가 작성한 코드를 가져와서 사용할 수 있다. 최근, `ES modules`라는 새로운 모듈 시스템이 NodeJS에 추가됐다고 하니 기존의 `Common JS`와 어떻게 다른지 살펴보도록 하자.

## 왜 Node JS에서 모듈 시스템이 필요할까?

방대한 양의 코드를 파일별로 나누면 코드를 재사용하고 구조화할 수 있다. 또한, 어떤 파일의 어떤 부분에 있는 코드가 접근가능한 지 관리할 수 있다.

자바스크립트로 쓰는 모든 코드는 기본으로 전역범위를(global) 갖는다. 복잡한 애플리케이션을 자바스크립트로 작성하면서 이런 점이 큰 문제로 대두됐고 NodeJS 개발자들은 기본 모듈시스템인 **CommonJS**를 도입했다.

## CommonJS : NodeJS의 기본 모듈 시스템

`package.json`에 `type`이 정확히 명시되어있지 않거나 `commonjs`로 되어있다면 `.js`파일은 NodeJS의 기본 모듈시스템인 CommonJS 모듈로 다뤄진다.

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

`exports` 객체는 read-only로 덮어쓸 수 없다. 단순하게 `module` 객체의 `exports` 프로퍼티에 접근하는 shortcut으로 아래와 같이 다시 쓸 수 있다.

```js
const chalk = require("chalk");

function info(message) {
  console.log(chalk.blue(message));
}

function error(message) {
  console.log(chalk.red(message));
}

const defaultMessage = "Hello World";

module.exports = {
  logInfo: info,
  logError: error,
  defaultMessage,
};
```

`exports` 객체에 직접 값을 할당하는 것 대신에 객체 전체를 한번에 선언해 `module.exports`에 할당하는 것이다.

이 방법을 통해 함수 `info()`와 `error()`의 이름을 각각 `logInfo`와 `logError`로 다시 쓸 수 있다. 즉, 외부 API와 내부 API를 분리할 수 있다. (하지만 내외부 API의 이름을 동일하게 유지하는 편이 코드를 쉽게 다룰 수 있을 것이다.)

### 특정한 값만 import하기

```js
const { logError } = require("./logger");
```

자바스크립트의 구조분해할당을 사용해 import하고자 하는 코드의 특정한 부분만 가져올 수 있다. logger 객체에서 `logError`라는 속성을 지역변수 `logError`에 저장해 사용하는 것이다.

## ES 모듈 : ECMAScript

ECMAScript standard(ES2015) 이후로 언어자체에서 표준화된 모듈 시스템을 탑재했으며 이를 ES 모듈이라고 한다. 노드 14버전에서 처음 안정화되었다.

`package.json`의 `type` 필드에 **module**이라고 명시되어 있다면 `.js`파일은 ES 모듈로 다뤄진다.

```json
{
  ...
  "type":"module",
  ...
}
```

`package.json`과 별개로 `.mjs` 확장자 파일은 항상 ES 모듈로, `.cjs` 확장자 파일은 항상 CommonJS 모듈로 다룬다.

### ES 모듈로 export하기

위에서 작성한 로깅 예제를 클래스로 다시 적어보자.

```js
// logger.mjs
import chalk from "chalk";

export class Logger {
  static defaultMessage = "Hello World";

  static info(message) {
    console.log(chalk.blue(message));
  }

  static error(message) {
    console.log(chalk.red(message));
  }
}
```

모듈을 import할 때 `require()` 대신 `import`를 사용한다. 또한, `module` 객체대신 클래스 앞에 붙이는 `export` 키워드를 사용한다. 이 키워드가 컴파일러에 어떤 파일의 어떤 부분이 다른 파일에서 접근이 가능하다고 명시해준다.

### ES 모듈로 import하기

```js
import { Logger } from "./logger.mjs";
```

CommonJS와 비슷하게 구조분해할당을 이용해 사용하고자 하는 값만 import 해올 수 있다.

### Exports vs Default Exports

자바스크립트는 일반 exports와 기본(default) exports를 구분한다. `default`를 사용하면 특정값을 모듈의 기본 export로 지정할 수 있다.

```js
export const defaultMessage = "Hello World";

export default class Logger {
  static info(message) {
    console.log(chalk.blue(message));
  }

  static error(message) {
    console.log(chalk.red(message));
  }
}
```

import하고자 하는 속성을 지정하지 않으면 `default` 키워드가 붙은 값을 기본으로 import 하도록 한다.

```js
import Logger from "./logger.mjs";
```

`logger.mjs`에서 어떤 값을 import할 지 지정하지 않았으므로 `default` 키워드가 붙은 `Logger` 클래스를 불러온다.

### import에 이름붙이기

```js
import * as LoggerModule from "./logger.mjs";
```

`logger.mjs`에 있는 모든 것을 `LoggerModule`이라는 네임스페이스에 담아 불러온다.

default import에 이름을 붙이려면 아래와 같이 코드를 작성한다.

```js
import { default as Logger } from "./logger.mjs";
```

## ES 모듈에서 CommonJS 모듈 import 하기

NodeJS는 ES 모듈에서 CommonJS 모듈을 import할 수 있다.

```js
import Logger from "./logger.js";
```

`module.exports`는 default export로 취급한다.

## CommonJS 모듈 vs ES 모듈

두 NodeJS 모듈 시스템의 차이점에 대해 알아보자.

### 파일 확장자

ES 모듈은 CommonJs와 Webpack, Typescript와 반대로 import할 때 파일 확장자를 명시해야 한다.

NodeJS가 파일 확장자를 통해 CommonJS 모듈과 ES 모듈을 구분한다는 점에서 이러한 차이점은 중요하다. 기본적으로 `.js` 확장자를 갖는 파일은 CommonJS 모듈, `.mjs` 확장자를 갖는 파일은 ES 모듈로 취급한다.

NodeJS 프로젝트의 기본 모듈 시스템을 ES 모듈로 하고싶다면 [여기](https://nodejs.org/api/packages.html#packagejson-and-file-extensions)에서 프로젝트를 설정하는 법을 볼 수 있다.

위에서 언급했듯이 ES 모듈은 CommonJS 모듈을 import할 수 있지만 반대로 CommonJS 모듈에서 ES 모듈을 import하는 것은 불가능하다. 즉, `.mjs` 파일에서 `.js`파일을 import 할 수 없다.

### 동적 vs 정적

두 모듈은 import와 export를 다루는 방식에서 차이가 있다.

CommonJS import는 런타임에서 동적으로 실행된다. `require()` 함수는 단순하게 코드가 실행될 때 동작하며 결과적으로 코드의 어느 곳이든 부를 수 있으며 promise를 리턴하거나 callback을 호출하지 않는다.

ES 모듈의 import는 정적이고 구문 분석 시간에 실행된다. import한 스크립트를 파싱해 export하는 구문이 있는지 확인한 후 스크립트가 실행된다. 이는 import가 **호이스팅**된다고 볼 수 있다. import가 암시적으로 파일의 맨 위로 옮겨지기 때문에 코드의 중간에서 import를 할 수가 없는 것이다. 이는 에러를 사전에 확인할 수 있고 개발자 도구를 사용해 더 유효한 코드를 작성할 수 있는 장점이 있다.

동적으로 모듈을 import해야만 하는 상황도 있을 것이다. 이때는 동적 `import()` 함수를 사용하면 된다.

이외에도 ES 모듈 시스템은 기본적으로 strict mode를 사용하고 `this`는 전역 객체를 참조하지 않으며 스코프가 다르게 동작한다.

## 언제 무엇을 사용해야할까?

만약 새로운 프로젝트를 시작할 참이라면 **ES 모듈을 권장**한다. ES 모듈은 몇년동안 표준화되어왔고 NodeJS 14부터 안정적으로 지원하고 있다.

이미 CommonJS 모듈을 사용하고 있는 기존의 프로젝트라면 **굳이 ES 모듈로 마이그레이션할 필요는 없다**. CommonJS 모듈은 앞으로도 NodeJS의 기본 모듈 시스템일 것이기 때문이다. 그러나, Babel이나 Typescript 같은 툴을 사용해 더 쉽게 CommonJS 를 유지하면서 ES 모듈을 사용할 수도 있다.

CommonJS에서 ES 모듈로 마이그레이션 시 이전 버전과의 호환성에서 문제가 생길 수 있는 것을 주의하자.
