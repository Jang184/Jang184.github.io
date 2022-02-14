---
layout: post
header-style: text
title: Promise.all()을 사용한 비동기 처리
subtitle: 여러 개의 promise를 병렬 실행해서 람다 Timeout 피하기
author: Juri
tags:
    - javascript
    - Dynamodb
---

통계 자료를 만들기 위해서 MySQL에 있는 비디오의 조회수를 일 단위로 Dynamodb에 기록하려고 한다. 매일 자정에 모든 비디오의 조회수를 캡쳐해서 저장하는 것과 비슷하다.

통계 자료를 MySQL이 아닌 Dynamodb에 따로 저장하는 것은 빈번하게 조회되는 통계 데이터를 위한 쿼리를 빠르게 가져오고 기존의 서비스 데이터와 혼동되지 않게 하기 위해서다. 또한, 다른 테이블과 상호관계가 없는 독립적인 데이터 구조상 `NoSQL`이 더 적합하다고 판단했다. 대신 여러가지 조건에 맞게 검색하기 어려워진다.

Process
----

*구체적인 코드는 생략했습니다.*

### 1. MySQL에서 데이터 가져오기

```ts
const connection = conn.connect()

const videoQuery = await connection.execute("SELECT VIEW FROM VIDEO")
```

비디오 테이블의 데이터는 약 1000개 정도가 있다. `mysql2`패키지를 사용해 로컬 MySQL과 연결했으며 `await`을 사용해 videoQuery를 가져오는 작업이 완료되면 그 다음으로 넘어간다.

### 2. 가져온 데이터를 Dynamodb 쿼리에 맞게 가공하기

Dynamodb의 `파티션 키`는 비디오 ID로 새로 들어오는 데이터는 이에 맞게 분류된다. 즉, 데이터는 비디오 ID가 이미 존재하면 `update`, 없으면 `create` 된다. (create or update)
Dynamodb의 쿼리가 의외로 복잡해서 애를 먹었다 (...)

```ts
const currentDate = new Date()

for (const data of VideoQuery[0]) {
      const Item = {
        date: currentDate.toISOString(),
        view_count: data.VIEW
      };
      const params = {
        TableName: "VideoTable",
        Key: { id: data.VIDEO_ID },
        UpdateExpression: "SET history = list_append(if_not_exists(history, :empty_list),:p)",
        ExpressionAttributeValues: {
          ":empty_list": [],
          ":p": [Item],
        },
      };
```

### 3. Dynamodb에 데이터 저장하기

여러 개의 데이터를 `put`할 땐 `BatchWriteItem`을 사용할 수 있지만 `update`의 경우는 여러 개의 데이터를 한번의 쿼리 실행으로 업데이트할 수 없어서 for loop를 사용해 모든 비디오에 대해서 데이터를 따로 update했다.

```ts
for (const data of videoQuery[0]){
    ...

    await documentClient.update(params).promise()  
}

```

문제 발견
---
EventBridge를 람다함수의 Event로 매일 자정마다 자동으로 그 날의 조회수를 저장되도록 했다. 배포를 완료하고 데이터가 잘 쌓이는 것을 확인한 뒤로 몇 주가 지났을까 .. 통계 자료 요청이 들어와 데이터를 조회해보니 절반 가량되는 약 400개의 비디오에만 데이터가 쌓이고 있는 것을 발견했다. cloudwatch로 살펴보니 `Timeout`에러가 발생해 람다함수가 중간에 실행을 멈춘 것이었다. 

![](/img/in-post/promise-1.png)
한 번에 쿼리를 실행시킬 수 없어서 하나씩 for loop 를 돌린 것이 화근이었다. 게다가 `await`를 사용해서 그 많은 쿼리를 동기적으로 처리하고 있었으니 시간이 오래 걸릴 수 밖에!! 

많은 양의 쿼리를 한 번에 병렬적으로 처리할 방법을 찾다가 발견한 것이 `Promise.all()`되신다.

Promise.all()
---

iterable 객체를 매개 변수로 받아 여러 개의 프로미스를 **비동기적으로 이행**한다. 일반적으로 다음 코드를 실행하기 전에 서로 연관된 여러 개의 비동기 작업이 완료되어야할 때 사용한다. 입력된 프로미스 중 하나라도 거부되면 즉시 실행을 중지한다. 

프로미스의 이행 여부에 상관없이 모든 프로미스를 완료하려면 `Promise.allSettled()`를 사용한다.

문제 해결
---

```ts
const updates = []
for(const data of videoQuery[0]){
    ...

    updates.push(documentClient.update(params).promise)
}

Promise.all(updates)
```
promise들을 하나의 배열에 담는다. 그리고 그 배열을 `Promise.all()`안에 담아주면 끝! 

**Promise 적용 전 : 약 9분**
![](/img/in-post/promise-2.png)

**Promise 적용 후 : 약 1초**
![](/img/in-post/promise-3.png)

드라마틱하게 줄어든 `Duration`을 확인할 수 있다.

