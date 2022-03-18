---
layout: post
header-style: text
title: 데이터 규격화 포맷에 어떤 것이 있을까? - JSON, YAML, XML
author: Juri
tags:
    - json
    - yaml
    - xml
---

<i>도서 <학교에서 알려주지 않는 17가지 실무 개발 기술, 이기곤 저> 을 참고했습니다.</i>

## 1. JSON

JSON(`JavaScript Object Notation`)은 "키-값 쌍"으로 이루어진 데이터를 오브젝트(객체)에 담아 처리하는 포맷이다. 가공된 JSON 데이터는 텍스트 기반이므로 `사람이 읽거나 수정할 수 있다`. 그만큼 `디버깅`이 편리하므로 생산성을 높일 수 있다. 서버-클라이언트 사이에 주고 받는 메세지 포맷 규격중 하나로 널리 사용되고 있다.

### 특징

-   **UTF-8** 문자열 인코딩만 허용
    UTF-16 혹은 EUC-KR 을 사용하는 환경에서 JSON 파일을 사용할 땐 주의해야 한다.
-   **주석**을 지원하지 않음
    서버리스 로컬환경에서 사용할 S3 목데이터를 만드는 과정에서 json파일에 주석을 적어서 에러를 봤던 게 기억난다.

### 구조

Amazon S3에서 전송하는 알림 메세지 형식이 json이라서 예시로 가져와보았다.

객체는 `{}`로 , 배열은 `[]`로 표현한다. (자바스크립트의 객체, 배열의 형식과 같다.) 키값과 문자열은 `""`(큰 따옴표)로 감싼다.

```json
{
    "Records": [
        {
            "eventVersion": "2.2",
            "eventSource": "aws:s3",
            "awsRegion": "us-west-2",
            "eventTime": "The time, in ISO-8601 format, for example, 1970-01-01T00:00:00.000Z, when Amazon S3 finished processing the request",
            "eventName": "event-type",
            "userIdentity": {
                "principalId": "Amazon-customer-ID-of-the-user-who-caused-the-event"
            },
            "s3": {
                "s3SchemaVersion": "1.0",
                "configurationId": "ID found in the bucket notification configuration",
                "bucket": {
                    "name": "bucket-name",
                    "ownerIdentity": {
                        "principalId": "Amazon-customer-ID-of-the-bucket-owner"
                    },
                    "arn": "bucket-ARN"
                },
                "object": {
                    "key": "object-key",
                    "size": "object-size in bytes",
                    "eTag": "object eTag",
                    "versionId": "object version if bucket is versioning-enabled, otherwise null",
                    "sequencer": "a string representation of a hexadecimal value used to determine event sequence, only used with PUTs and DELETEs"
                }
            }
        }
    ]
}
```

보통 실무에선 파일보다 HTTP 요청 메세지에 있는 문자열을 JSON 으로 읽어 사용(파싱)하는 경우가 많다.

```ts
const userName: string = JSON.parse(event.body);

// 혹은

const response = {
    statusCode: 500,
    body: JSON.stringify("Internal Error"),
};
```

postman과 같은 API 테스트 툴에서도 HTTP 요청 메세지의 body에 json형식으로 데이터를 담는다.

![](/img/in-post/json1.png)

### 한계

사람이 읽기 쉬운만큼 컴퓨터 입장에선 비효율적인 규격이다.

-   불필요한 트래픽 오버헤드
    텍스트 기반이기 때문에 실질적인 데이터를 표현하는 데 비용이 크다.
-   메시지 호환성 유지의 어려움
    서버에서 JSON 파일을 업데이트하면 같은 규격의 JSON 키와 값을 사용하여 통신하는 모든 클라이언트 프로그램도 모두 동일하게 반영해야 한다.

> 특별히 사용해야하는 데이터의 규격이 정해지지 않았다면, 가장 널리 사용되는 JSON으로 시작하길 추천!

## 2. YAML

YAML(`Yet Another Markup Language`)는 이메일 형식에서 힌트를 얻어 만든 사람이 쉽게 읽을 수 있는 데이터 직렬화 포맷이다. JSON만큼 범용적인 데이터 포맷은 아니지만 YAML만이 갖는 고유한 특징덕에 조금씩 영역을 넓혀가고 있다.

### 특징

-   주석 지원
    데이터를 주고받을 땐 주석이 필요없지만 설정 파일처럼 구조화된 데이터에 설명을 추가할 땐 유용하다. 실제로 Docker 설정파일에서 주석을 보고 도움을 많이 받았다
-   UTF-8과 UTF-16지원
    UTF-16을 기본으로 사용하는 윈도우에서 설정 파일로 사용할 수 있다.

-   앵커와 별칭
    공통으로 사용하는 값을 한 곳에서 관리할 수 있다. 특히 애플리케이션 설정 파일은 환경 변수 등 다른 설정에 영향을 주는 경우가 많아 **설정의존성**을 관리하기 편하다

### 구조

docker compose 설정파일을 예시로 가져와보았다.

JSON과 달리 `""`을 사용하지 않고 `{}`대신 들여쓰기를 사용한다. 이 때 **탭을 사용한 들여쓰기**는 허용되지 않으므로 주의한다. 대괄호 대신 하이픈(-)을 사용해 배열을 작성한다. 마지막으로 주석은 `#` 뒤에 적는다.

```yaml
# 주석은 이렇게 사용합니다.

version: "3.4"

services:
    webmvc:
        image: eshop/webmvc
        environment:
            - CatalogUrl=http://catalog-api
            - OrderingUrl=http://ordering-api
            - BasketUrl=http://basket-api
        ports:
            - "5100:80"
        depends_on:
            - catalog-api
            - ordering-api
            - basket-api
#######################################################
# #을 이용해서 코드를 구분하기도 한다.
```

YAML의 고유한 기능인 앵커, 별칭, 주석을 제외하면 JSON 포맷과 완벽하게 호환이 가능하다.

### 앵커, 별칭

`default`설정을 `&default`을 이용해 앵커로 지정했다. 이 앵커를 configurations의 `dev`, `staging`, `production`에서 참조하고 있다.
`<<:`을 사용해 앵커를 참조하는 값에 더 많은 값을 추가하거나 기존값들을 덮어씌울 수 있다. 중복되는 값을 환경마다 쉽게 설정할 수 있다.

```yaml
definitions:
    default: &default
        app_name: myService
        secure_mode: false
configurations:
    dev:
        <<: *default
        address: dev_host
    staging:
        <<: *default
        address: staging_host
    production:
        <<: *default
        address: prod_host
```

JSON과 마찬가지로 텍스트 기반 데이터 포맷이기 때문에 사람이 읽기 쉬우면서 중괄호나 큰따옴표를 사용하지 않아 비교적 적은 용량으로 데이터를 주고 받을 수 있다. 하지만 JSON만큼 범용적으로 사용되지 않기 때문에 함께 사용할 안정적인 라이브러리가 있는지 확인해볼 필요가 있다.

## 3. XML

XML은 웹에서 규격화된 데이터를 효율적으로 주고받기 위해 다목적 마크업 언어이다. 자바나 C# 애플리케이션 설정 파일, 서버-클라이언트 간 메시지를 주고받는 용도로 사용한다. JSON과 YAML의 등장에 사용 빈도가 점점 줄어들고는 있다.

### 특징

-   텍스트 기반 데이터
    JSON과 YAML과 마찬가지로 텍스트를 기반으로 규격화된 데이터를 표현한다. 그러나 데이터를 표현하는 데 불필요한 데이터를 많이 사용해 JSON과 YAML보다 많은 데이터를 사용한다.

-   문자열 인코딩 지원
    문자 인코딩을 직접 지정할 수 있다.

### 구조

XML의 모든 구성 요소들은 `<>`로 이루어진다. 배열이라는 개념은 없지만 같은 이름을 가진 요소가 여러 개 있을 때 이것을 배열로 취급한다.

```xml
<?xml version="1.0"?>
<CAT>
  <NAME>Izzy</NAME>
  <BREED>Siamese</BREED>
  <AGE>6</AGE>
</CAT>

<DOG>
  <NAME>Happy</NAME>
  <NAME>Gamja</NAME>
  <NAME>BOMI</NAME>
</DOG>
```

> 반드시 데이터 직렬화를 위해 XML을 사용해야 하는 것이 아니면 JSON을 사용할 것을 추천!
> 반드시 설정파일 규격으로 XML을 사용해야 하는 것이 아니면 YAML을 사용할 것을 추천!

## 마치며

JSON은 실무에서 사용해본 적 있어 정체(?)를 알고있엇지만 YAML과 XML은 어디선가 봤는데 정확히 정체가 뭔지는 몰랐다. 이번 기회에 뭐하는 친구들인지 알아보았고 어떤 구조로 작성할 수 있는지 까지 알아보았다. 이젠 헷갈리지 말자!
