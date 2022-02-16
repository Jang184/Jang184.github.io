---
layout: post
header-style: text
title: serverless framework - 플러그인과 로컬테스트
subtitle: 서버리스 왕초보 탈출기 (2) 서버유지보수로부터 자유로워지자! 
author: Juri
tags:
    - serverless
    - aws
---

> 이 글을 읽기 위해서는 javascript와 node js에 대한 기본 지식이 필요합니다.

서버리스에서 지원하는 여러 가지 `플러그인`에 대해 알아보고 배포 전 `로컬환경에서 테스트`를 하는 방법을 알아보자.

플러그인 (Plugins)
----

플러그인 : 서버리스 프레임워크의 새로운 기능을 지원하는 커스텀 자바스크립트 코드

### 플러그인 설치

먼저, 플러그인을 사용하기 위해서 플러그인을 설치한다.

```bash
$ serverless plugin install -n plugin1
```

설치한 후 `serverless.yml` 파일에 플러그인을 등록한다. 이 때 플러그인을 작성한 순서에 따라 플러그인이 실행된다.

```ts
plugins:
  - plugin1
  - plugin2
```

### 플러그인 종류

**● Serverless Offline**
>Emulate AWS λ and API Gateway locally when developing your Serverless project

로컬환경에서 `AWS Lambda`와 `API Gateway`를 테스트할 수 있게 도와주는 플러그인이다.
```bash
$ serverless offline
혹은
$ sls offline
```
`serverless.yml`에서 포트를 따로 지정하지 않으면 기본 3000번 포트로 서버가 열린다. `http://localhost:3000`으로 요청 메세지를 보내 람다 핸들러를 실행시킬 수 있다.


**● Serverless Webpack**
> Serverless plugin to bundle your lambdas with Webpack

람다함수들을 웹팩으로 묶어주는 플러그인이다. `웹팩`은 여러 개의 모듈을 하나의 자바스크립트 코드로 압축하는 라이브러리다.

webpack configuration file<br>
아래와 같이 webpack 경로를 따로 지정하지 않으면 `webpack.config.js`를 찾는다.
```yml
custom:
  webpack:
    webpackConfig: ./src/my-webpack.config.js
    includeModules: true
      forceExclude:
        - aws-sdk

```
```ts
// my-webpack.config.js
const slsw = require("serverless-webpack")
const nodeExternals = require("webpack-node-externals")
const path = require("path")

module.exports = {
  mode: "development",
  entry: slsw.lib.entries,
  outputs: {
    path: path.join(__dirname, ".webpack")
  },
  externals: [nodeExternals()]
}
```
- mode: 모드 (`production`, `development`)에 따라 최적화 혹은 빠르게 빌드된다.
- entry: webpack이 빌드할 파일의 시작위치를 나타낸다.
- output: 생성된 번들을 내보낼 위치와 파일이름을 지정한다.
- externals: 번들에서 제외할 모듈을 지정한다. 위의 코드는 외부모듈을 지정하고 있다.

**● Serverless Prune Plugin**
>Deletes old versions of functions from AWS, preserving recent and aliased versions

새로 배포했을 때 기존의 배포된 버전은 지워지지 않기 때문에 버전이 계속 쌓이게 된다. 최신 배포 버전을 제외한 모든 버전을 제거하는 플러그인이다.

```bash
$ sls prune -n <number of version to keep>
```
루트 디렉토리에서 남길 버전을 입력과 함께 명령어를 입력한다.

```yml
custom:
  prune:
    automatic: true
    number: 1
```

배포를 할 때마다 자동으로 실행되게 설정할 수 있다. 또한, 지우지 않고 남길 버전의 갯수도 지정해야 한다.

**● Serverless Dotenv Plugin**
>Preload environment variables from `.env` into serverless.

`.env`파일에 있는 데이터를 `yml`파일에서 사용할 수 있게 해주는 플러그인이다.

```yml
...
provider:
  name: aws
  runtime: nodejs14.x
  stage: ${env:STAGE}
  region: ${env:AWS_REGION}
...
```
스테이지나 리젼을 여러개 사용하는 서비스라면 각각의 `.env`파일을 생성해 데이터를 관리하는 것이 편하다.

**● Serverless Plugin Warmup**
>Keep your lambdas warm during Winter.

일정한 시간간격을 두고 람다를 실행시켜 `cold start`를 방지하는 플러그인이다. 

```yml
custom:
  warmup:
    enabled: true
    events:
        - schedule: cron(0/5 8-17 ? * MON-FRI *)
```

시간간격을 조정할 수 있고 람다 함수별로 warmup을 실행할 수도 있다.

**● Serverless Plugin Scripts**
>Add scripting capabilities to the Serverless Framework

npm script와 비슷하게 명령어를 만들어서 실행할 수 있다.
```yml
custom:
  scripts:
    commands:
      hello: echo Hello from ${self:service} service!
```

```bash
$ sls hello
```
CLI에 미리 지정해 둔 명령어를 입력해서 빠르게 스크립트를 실행할 수 있다. 혹은 배포할 때 스크립트가 자동으로 실행되게 할 수 있다.

로컬 테스트
---

`Serverless Offline` 플러그인을 사용해서 쉽게 로컬테스트를 할 수 있다.

1. PostMan과 같은 툴
2. CLI 명령어

```bash
$ serverless invoke local --function myFunction

$ serverless invoke local --function myFunction --path ./lib/data.json

$ serverless invoke local --function myFunction --data '{"a":"bar"}'
```
함수를 지정하는 것뿐만 아니라 함수의 경로를 지정하거나 데이터를 전달할 수도 있다. context도 환경변수도


마치며
----

다음 포스팅에서는 
