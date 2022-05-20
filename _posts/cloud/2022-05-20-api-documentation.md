---
layout: post
header-style: text
title: AWS S3 정적 웹사이트 호스팅하기
subtitle: apidoc로 만든 API 명세서를 S3로 웹 호스팅해봅시다.
author: Juri
tags:
    - Cloud
    - aws
    - S3
    - apidoc
---

[APIDOC 공식 홈페이지](apidocjs.com)
[AWS CLI S3 공식 문서 - sync](docs.aws.amazon.com/cli/latest/reference/s3/sync.html)

API 명세서를 만들어 협업하는 동료들과 공유하기 위해 웹 호스팅을 하고 쉽게 관리하기 위해서 aws-cli를 이용해 로컬 디렉토리와 s3 버킷을 동기화시키는 방법을 알아보자.

## APIDOC

코드에 있는 API 어노테이션을 이용해 API 명세서를 만들어준다.

```
/**
 * @api {get} /user/:id Request User information
 * @apiName GetUser
 * @apiGroup User
 *
 * @apiParam {Number} id Users unique ID.
 *
 * @apiSuccess {String} firstname Firstname of the User.
 * @apiSuccess {String} lastname  Lastname of the User.
 */
```

swagger보다 코드 작성이 직관적이고 공식문서가 자세해 API 명세화하기에 훨씬 편리했다.

터미널에 `apidoc`명령어를 적당히 입력하면 지정한 디렉토리에 정적 파일을 생성한다.

![](/img/in-post/apidoc1.png)

## S3 정적 웹 호스팅

버킷을 생성하고 정적파일을 업로드 한다.

![](/img/in-post/apidoc2.png)

퍼블릭 엑세스가 가능하도록 정책을 추가해준다. 모든 사용자가 해당 버킷에 대해 읽기 권한을 가질 수 있도록 `GetObject`를 선택했다. 웹 호스팅 테스트 후 지정 IP만 접근이 가능하게 정책을 수정할 것이다.

![](/img/in-post/apidoc3.png)

버킷 > 사용하고자 하는 버킷 > 속성 > 정적 웹 사이트 호스팅

속성의 맨 아래에 있는 정적 웹 사이트 호스팅을 활성화 해준다.

![](/img/in-post/apidoc4.png)

버킷 웹 사이트 엔드포인트가 자동으로 생성된다. 주소를 눌러보면 apidoc으로 만든 정적 파일의 index.html 과 동일한 화면의 API 명세서가 뜬다.

![](/img/in-post/apidoc5.png)

## AWS CLI 로 버킷 동기화

npm으로 설치하면 이유는 모르겠지만 명령어가 실행이 안되길래 `pip`으로 설치했다.

```bash
pip install aws-cli
```

많은 명령어 옵션이 있으니 위에 적어놓은 aws-cli 공식 문서에서 잘 찾아 골라 쓴다. 로컬 디렉토리에 없는, 버킷에는 있는 파일을 삭제하기 위해 `--delete` 옵션을 추가해준다.

```bash
aws s3 sync apidoc s3://maggyeo-api-documentation --delete
```

![](/img/in-post/apidoc6.png)

이 때, s3에 업로드할 수 있는 권한이 iam 사용자에 추가돼있어야 한다.

## S3 Access IP Whitelist (특정 IP만 접근 허용하기)

S3 버킷의 정책을 수정해준다.

```py
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::maggyeo-api-documentation/*",
            "Condition": {
                "IpAddress": {
                    "aws:SourceIp": "{my ip}"
                }
            }
        }
    ]
}
```

내 접속 IP를 제외한 모든 IP의 버킷 접근을 제한한다. (반대로, 내 IP만 버킷 내 읽기 권한이 허용된다.)
