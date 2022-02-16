---
layout: post
header-style: text
title: serverless framework - 환경 셋팅과 배포
subtitle: 서버리스 왕초보 탈출기 (1) 서버유지보수로부터 자유로워지자! 
author: Juri
tags:
    - serverless
    - aws
---

현재 일하고 있는 회사 ([데브언리밋](www.devunlimit.com))에서 사용하고 있는 `서버리스 프레임워크`를 소개하는 글이다. 처음 접해보는 기술스택이라서 한참 헤맸지만 지금은 어느 정도 안정기(?)에 접어들었다. 지금까지 공부해온 것들을 정리해보는 시간을 가지려고 한다.

> 이 글을 읽기 위해서는 javascript와 node js에 대한 기본 지식이 필요합니다.

서버리스는 AWS와 같은 클라우드 서비스를 사용하는 서비스에게 가뭄에 단비와도 같은 프레임워크다. AWS 콘솔을 사용해서 인프라를 구축해본 사람이라면 얼마나 복잡하고 눈에 안들어오는 지 알 것이다. 서버리스는 코드 몇 줄로 복잡한 AWS 인프라를 뚝-딱 구축해준다. **천리길도 한걸음부터!** 천천히 알아보자.

환경 셋팅하기
----

### 설치

```bash
npm init

npm install -g serverless
```

node가 설치되었다면 `npm`을 이용해 쉽게 설치할 수 있다. 프로젝트를 생성한 후 `serverless`를 전역에 설치한다. 

### AWS 계정 연결

```bash
serverless config credentials --provider aws --key {엑세스키} --secret {시크릿엑세스키}
```

연결하고자 하는 AWS 계정 정보를 저장한다. `엑세스키`와 `시크릿엑세스키`는 AWS iam 에서 발급가능하다. 이로써 저장한 AWS 계정에 앞으로 만들 프로젝트를 배포할 수 있다. 

### 템플릿 생성

```bash
serverless create --template aws-nodejs

serverless create --template aws-nodejs --path myService
```

프로젝트에 `nodejs`를 사용할 것이므로 해당하는 템플릿을 생성한다. 


루트 디렉토리에 `handler.js`와 `serverless.yml`이 생긴 것을 볼 수 있다. 

`handler.js`엔 배포하고자 하는 lambda 함수를 작성한다.

`serverless.yml`에 람다함수와 각종 이벤트를 설정한다. `ApiGateWay`나 `iam` 등 AWS 인프라 구축에 필요한 모든 리소스의 설정을 이 파일에서 할 수 있다. 간단하게 살펴보도록 하겠다.

```yml
service: myService

frameworkVersion: '3'

provider:
  name: aws
  runtime: nodejs14.x
 stage: dev
 region: ap-northeast-2
```

프로젝트의 기본 설정. nodejs의 버전, 배포 스테이지, 리젼 등을 작성한다.

```yml
  iam:
    role:
      statements:
        - Effect: "Allow"
          Action:
            - "s3:PutObject"
          Resource:
            Fn::Join:
              - ""
              - - "arn:aws:s3:::"
               - "Ref" : "ServerlessDeploymentBucket"
               - "/*"
```
람다 함수의 iam 역할을 설정한다. 위의 코드는 람다 함수에 S3 버킷에 객체를 업로드할 수있는 권한을 주고 있다.

```yml
functions:
  hello:
    handler: handler.hello
    events:
     - httpApi:
         path: /users/create
         method: get
     - websocket: $connect
     - s3: ${env:BUCKET}
     - schedule: rate(10 minutes)
```

람다 함수의 이벤트를 설정할 수 있다. ApiGateWay로 HTTP 엔드포인트를 지정할 수 있고 그 외에도 크론잡(EventBridge)을 설정하거나 다양한 AWS 서비스와 연결할 수 있다.

```yml
resources:
 Resources:
   NewResource:
     Type: AWS::S3::Bucket
     Properties:
       BucketName: my-new-bucket
```
`cloudformation`으로 리소스를 자동 생성할 수 있다. 배포할 때 클라우드포메이션을 사용해 S3 버킷이나 DynamoDB 테이블, ApiGateWay RestApi, Cognito UserPool 등을 생성할 수 있다. 정책도 생성할 수 있으니 얼마나 간편한가! 콘솔로 들락날락 하지 않아도 `serverless.yml`파일 안에서 모두 해결이 가능하다.

[참고:서버리스 공식페이지](serverless.com/framework/docs/provider/aws/guide/serverless.yml)

배포
-----

```bash
serverless deploy
```

설정에 비해 배포는 간단하다. deploy를 입력하면 serverless가 yml파일을 읽어서 AWS에서 인프라를 뚝-딱 뚝-딱 구축한다. ApiGateWay에서 API 엔드포인트를 설정했으면 주소창에 입력해 배포가 됐는지 바로 확인 가능하다.


마치며..
-----

처음에 헤매던 때의 추억(?)이 새록새록 난다. 지금은 이렇게나 간편하게 배포를 도와주는 프레임워크가 있다는 사실에 감사의 눈물을 줄줄 흘리고 있다.. 

다음 포스팅에서는 서버리스 플러그인과 로컬테스트 하는 법에 대해 설명하도록 하겠다! 