---
layout: post
header-style: text
title: AWS S3 presigned URL로 이미지 업로드 하기
author: Juri
tags:
    - Cloud
    - aws
    - S3
---

회원가입할 때 프로필 이미지를 등록하는데 이 이미지를 어떻게 S3에 업로드할 지 생각해본 결과 두가지 방법으로 나눌 수 있었다.

1. 이미지를 서버로 가져와 서버에서 직접 S3로 업로드
2. presigned url을 반환해 브라우저 단에서 이미지를 s3로 업로드

이미지의 크기가 커졌을 때, 서버로 이미지를 가져오는 것이 비효율적이라는 생각이 들어 두번째 방법을 선택했다.

전체적인 과정은 아래의 사진과 같다.

![](/img/in-post/presignedurl.jpg)

## presigned url

미리 서명된 URL이라고 하는 이 **presigned URL**을 사용하면 객체를 업로드할 수 있는 권한을 잠시 부여받아 비공개 버킷에 객체를 업로드할 수 있다. 만료 시간을 지정해 이 URL이 계속 사용되지 않도록 해준다. 만료 시간이 되기 전에 이 URL은 몇번이고 사용할 수 있다!

`aws-sdk`를 이용해 presigned URL을 만들어보자.

```js
import AWS from "aws-sdk";

const s3 = AWS.S3()

const s3Params = {
    Bucket : S3Bucket,
    Key : objectKey,
    Expires : 3000
    ContentType : "image/jpeg",
    ACL: "public-read"
}

const uploadURL = await s3.getSignedUrlPromise("putObject", s3params);
```

s3Params에 이미지가 업로드 될 버킷명과 객체키(경로), 만료시간 등 정보를 입력하고 `putObject`와 함께 presigned URL을 받아온다. 이 URL에 이미지를 보내면 지정한 버킷의 지정 경로에 이미지가 업로드되고 만료시간이 지나면 이 URL은 사용할 수 없다.

![](/img/in-post/presignedurl2.png)

버킷명은 **"test-bucket"**, 객체키(경로)는 **"user/2/unique-string"**, 만료시간은 **300초**로 지정한 presigned URL은 위와 같다.

## 객체키로 쓰기 위한 unique string 만들기

첫번째. **Math** 사용하기

```js
const key = parseInt(Math.random() * 100000000);

console.log(key); //22266871
```

숫자로 이루어진 랜덤 문자열을 얻을 수 있다.

두번째. **Crypto** 사용하기

```js
const key2 = crypto.randomUUID();

console.log(key2); //'0b482a0b-42a1-46b5-943d-17002c8bdb9d'
```

```js
const key3 = crypto.randomBytes(20).toString("hex");

console.log(key3); // '516a7ca4ae103ba74b28d116b966f39e4c72de1a'
```

숫자와 문자가 뒤섞인 문자열을 얻을 수 있다.

## 이미지 업로드

![](/img/in-post/presignedurl3.png)

HTTP method로 **PUT**을 지정한 후 받은 URL로 이미지를 Body에 담아 요청을 보내면 200 OK가 뜨는 것을 확인할 수 있다.

버킷에 들어가서 확인을 하면 올바른 경로에 이미지가 업로드 되어있음을 확인할 수 있다.

![](/img/in-post/presignedurl4.png)

## S3 trigger

참고 : [aws 공식문서](https://docs.aws.amazon.com/ko_kr/lambda/latest/dg/with-s3-example.html)

이미지를 업로드 한 후 S3 이미지 경로를 DB에 저장해야 사용자의 프로필 이미지를 보여줄 수 있다. 처음엔 브라우저단에서 이미지를 S3로 업로드한 후 경로를 다시 서버로 보내서 저장을 해야하나 싶어서 번거롭다고 생각이 들어 더 찾아본 결과 Lambda 는 event-driven 으로 s3에 변경사항이 생기면 이것을 event로 받아 추후처리가 가능하다고 한다.

```yml
function:
    S3Trigger:
        handler: index.s3trigger
        event:
            - s3:
                  bucket: "test-bucket"
                  event: s3:ObjectCreated:*
                  existing: true
```

기존에 존재하는 버킷의 ObjectCreated 이벤트를 받아와 특정 Lambda 함수를 실행할 수 있다.

**S3 event example**

```json
{
    "Records": [
        {
            "eventVersion": "2.0",
            "eventSource": "aws:s3",
            "awsRegion": "us-west-2",
            "eventTime": "1970-01-01T00:00:00.000Z",
            "eventName": "ObjectCreated:Put",
            "userIdentity": {
                "principalId": "EXAMPLE"
            },
            "requestParameters": {
                "sourceIPAddress": "127.0.0.1"
            },
            "responseElements": {
                "x-amz-request-id": "EXAMPLE123456789",
                "x-amz-id-2": "EXAMPLE123/5678abcdefghijklambdaisawesome/mnopqrstuvwxyzABCDEFGH"
            },
            "s3": {
                "s3SchemaVersion": "1.0",
                "configurationId": "testConfigRule",
                "bucket": {
                    "name": "my-s3-bucket",
                    "ownerIdentity": {
                        "principalId": "EXAMPLE"
                    },
                    "arn": "arn:aws:s3:::example-bucket"
                },
                "object": {
                    "key": "HappyFace.jpg",
                    "size": 1024,
                    "eTag": "0123456789abcdef0123456789abcdef",
                    "sequencer": "0A1B2C3D4E5F678901"
                }
            }
        }
    ]
}
```

```js
const s3trigger = async (event) => {
    const bucket = event.Records[0].s3.bucket.name;
    const key = event.Records[0].s3.object.key;
};
```

객체의 URL은 **https://{bucket name}.s3.{region}.amazonaws.com/{key}**의 형식이므로 event에서 버킷명과 객체키를 받아와 생성해준다. 이 URL을 DB에 저장하면 이후 URL을 통해 프로필 이미지에 접근할 수 있다. 이 때, 버킷의 ACL 설정을 허용해 public read가 가능하게 해야 URL을 통해 이미지를 호스팅할 수 있으므로 주의한다.
