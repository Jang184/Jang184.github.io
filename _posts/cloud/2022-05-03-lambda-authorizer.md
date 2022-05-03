---
layout: post
header-style: text
title: AWS API Gateway 의 권한 부여자 - Lambda authorizer 사용하기
author: Juri
tags:
    - Cloud
    - aws
    - API Gateway
---

## Lambda authorizer

API Gateway 의 Lambda authorizer는 Lambda 함수를 사용하여 API 메서드에 대한 액세스를 제어하는 API Gateway 기능이다. 클라이언트가 API를 호출하면 API Gateway는 이 API와 연결된 Lambda authorizer를 호출한다. 이것은 호출자의 자격 증명을 입력으로 받아 IAM 정책을 출력으로 반환한다.

**token authorizer**와 **request authorizer**가 있다. token authorizer는 JWT 또는 OAuth 토큰에 있는 호출자의 자격 증명을 받고, request authorizer는 헤더, 쿼리 스트링, stageVariables 및 $context 변수의 조합으로 호출자의 자격 증명을 수신한다. API Gateway가 지원하는 `WebSocket`의 경우 request authorizer만 사용가능하다.

### Lambda authorizer workflow

![](/img/in-post/custom-auth-workflow.png)

1. 클라이언트가 API를 호출하고 토큰이나 파라미터를 전달한다.
2. API Gateway는 Lambda authorizer구성여부를 확인하고 있으면 호출한다.
3. Lambda authorizer는 호출자를 인증한다.
4. 호출이 성공하면 Lambda authorizer는 IAM 정책과 보안주체 식별자를 포함하는 출력 객체를 반환해 엑세스를 부여한다.
5. API Gateway가 정책을 평가한다.

쉽게 생각하면 내가 원하는 Lambda 함수를 호출하기 전에 연결된 Lambda 함수 (authorizer)를 실행시켜 인증단계를 거치는 것이다. 순서를 도식화해보면 이렇다.

![](/img/in-post/lambda-authorizer.jpg)

(엄밀히 말하면 위의 사진은 틀렸다. 정확한 워크플로우는 더 위의 공식 홈페이지에서 가져온 사진을 보면 된다.)
클라이언트가 보낸 JWT를 확인한 후 backend API를 호출한다.

## 실습

도식화한대로 실습을 해보겠다. 내 정보를 가져오기 위해서 API를 호출하고자 한다. 내 개인정보는 소중하기 때문에 나만 접근할 수 있어야한다. Lambda authorizer는 내가 보낸 JWT를 받아 확인한 후 claim에 있는 나의 user 정보를 API로 전달할 것이다. 그러면 API에서는 그 user 정보를 이용해 내 정보를 가져올 것이다.

Lambda authorizer가 반환하는 response에는 `policyDocument`가 포함되어야 한다.

```js
import jwt from "jsonwebtoken";

const generatePolicy = (principalId, methodArn) => {
    return {
        principalId,
        policyDocument: {
            Version: "2012-10-17",
            Statement: [
                {
                    Action: "execute-api:Invoke",
                    Effect: "Allow",
                    Resource: methodArn
                }
            ]
        }
    };
};
```

```js
export const handler = async (event, context) => {
    const token = event.authorizationToken;
    if (!token) throw "Unauthorized";

    try {
        const user = validateToken(token);
        const policy = generatePolicy(user.userId, event.methodArn);

        return {
            ...policy,
            context: user
        };
    } catch (err) {
        console.log(err);
        throw "Unauthorized";
    }
};

const validateToken = (token) => jwt.verify(token, process.env.AUTH_TOKEN_SALT);
```
