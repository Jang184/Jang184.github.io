---
layout: post
header-style: text
title: AWS Cognito - Cognito란 무엇일까?
subtitle: AWS Cognito + API Gateway 를 이용한 인증/인가 
author: Juri
tags:
    - Cloud
    - aws
    - cognito
---

AWS Cognito 
----
![](/img/in-post/cognito-1.png)


AWS Cognito는 인증, 인가 그리고 사용자 관리를 수행하는 서비스로 cognito로 로그인하면 환경 설정, 사용자 프로필과 같이 사용자별 데이터를 저장하고 검색하는 데 사용되는 임시 AWS 토큰을 받는다. `사용자 풀(User Pools)`과 `아이디 풀(Identity Pools)` 두 가지의 서비스를 제공하는 데 사용자 풀은 사용자 자격증명을 만들고 저장하는 위치고, 아이디 풀은 사용자가 AWS 리소스에 접속할 수 있는 권한을 설정하는 곳이다.

어플리케이션에서 Cognito 를 통해 로그인을 하면 Cognito는 사용자의 정보가 사용자 풀에 있는 지 확인한 후 토큰을 준다. 사용자는 이 토큰을 사용해서 아이디 풀에 다른 AWS 서비스에 접근할 권한이 있는지 확인을 한다.

### 사용자 풀 (User Pool)

> 사용자에게 회원가입과 로그인 옵션을 제공하는 일종의 사용자 데이터베이스


### 아이디 풀 (Identity Pool)

> 사용자에게 다른 AWS 리소스에 접근할 권한을 제공.

두 가지 서비스는 별도로 또는 함께 사용할 수 있다.

### 장점

Cognito를 사용하면 복잡한 회원가입, 로그인 기능 구현을 빠르게 할 수 있다. 또한 보안을 강화할 수 있고 접속기기가 달라져도 일관되게 사용할 수 있으며 소셜 미디어를 통한 로그인도 가능하다. 이외에도 마케팅 통계에도 사용하는 등 많은 장점을 갖고 있어 많은 서비스가 Cognito를 사용하고 있다.



Api Gateway Authorizer
---

Api Gateway Authorizer로 `CUSTOM 람다함수`와 `Cognito user pool`를 선택할 수 있다. 이 포스팅에서는 `Cognito user pool`을 Api Gateway Authorizer로 사용해 API에 대한 접근을 제어하는 방법을 알아보도록 한다.

[참고: AWS 공식 홈페이지](https://docs.aws.amazon.com/ko_kr/apigateway/latest/developerguide/apigateway-integrate-with-cognito.html)

### Cognito userpool authorizer 

![](/img/in-post/cognito-2.png)

1. 사용자가 Cognito를 사용해 로그인을 한다.
2. Cognito는 사용자를 검증한다.
3. 사용자 풀에 사용자가 있음을 확인하면 토큰을 반환한다.

이렇게 사용자는 **인증(Authentication)**을 거쳐 받은 토큰을 헤더에 담아 API Gateway에 요청을 보낸다. Api Gateway가 받은 요청은 내부 로직에서 **인가(Authorization)** 단계를 거쳐 작업을 수행한 후 사용자에게 응답한다. 람다함수 내부에서 직접 권한을 확인하거나 미들웨어를 사용할 수 있다.

예를 들어, 요청에 담긴 토큰에 있는 사용자의 아이디와 데이터베이스에 있는 사용자의 아이디를 비교해 해당 API에 접근할 수 있는 권한이 있는 지 확인할 수 있다. 또는, 람다 함수가 실행되기 전에 미들웨어를 실행해 토큰에 있는 정보와 데이터베이스의 정보의 일치 여부를 확인할 수도 있다.

node js의 람다함수 미들웨어 패키지로 `middy`가 있다. 이것은 다음에 포스팅하도록 하겠다! 