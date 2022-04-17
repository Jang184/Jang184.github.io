---
layout: post
header-style: text
title: TypeORM - 데이터베이스 연결
subtitle: TypeORM 사용해서 MySQL과 애플리케이션 연결하기
author: Juri
tags:
    - typeorm
    - MySQL
---

TypeORM을 사용해서 로컬의 데이터베이스(MySQL)와 서버리스 서비스간의 통신이 가능하다. 이를 위해서 데이터베이스와의 커넥션을 설정해야 한다.

## Connection Pool (CP)

데이터베이스와 통신할 때, 커넥션 풀에서 커넥션 객체를 가져와 작업을 수행하고 완료하면 다시 커넥션 풀로 돌려준다. 매 요청마다 커넥션 객체를 생성하여 연결하면 시간이 오래걸리고 메모리를 많이 차지해 비효율적이다. 반면, 커넥션 풀을 사용하면 커넥션 풀 속에 미리 커넥션 객체를 생성해놓아 시간이 소비되지 않는다. 또한, 사용하고 있지 않은 커넥션 객체를 재사용하기 때문에 계속해서 커넥션을 생성하지 않는다. 커넥션 풀을 사용하면 프로그램의 효율과 성능이 증가한다.

TypeORM의 `createConnection`을 사용하면 **커넥션 풀**을 생성한다. 커넥션 풀에서 커넥션 객체를 가져오기 위해서 `getConnection`을 사용하면 된다.

```ts
import { createConnection, Connection } from "typeorm";

const connection: Connection = await createConnection({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "root",
    password: 1234,
    database: "myservice",
});
```

`mysql`의 커넥션 폼은 위와 같다. 이 포스팅에서 로컬의 MySQL은 `Docker` 환경에 띄워놓았기 때문에 Docker port로 설정하도록 한다. 커넥션은 두 개 이상 생성할 수 있다.

## Connection Options

`createConnection`을 사용해 커넥션을 생성할 때 몇가지 옵션을 설정할 수 있다.

```ts
const connectionOptions = {
    name: "default",
    type: "mysql",
    synchronize: false,
    logging: true,
    entities: [User, Photo],
    namingStrategy: new SnakeNamingStrategy(),
};
```

이외에도 subscribers, migrations 등의 추가적인 option을 설정할 수 있다. 새로 생성한 entity를 connection option에 추가하지 않으면 typeorm이 entity를 인식하지 못한다. **꼭 까먹지않고 새로 생성한 entity를 connection에 추가한다!**

## .env 별로 다른 connection option 사용하기

개발 스테이지에 따라 다른 데이터베이스를 이용하게 된다. 그 때마다 connection을 수동으로 바꾸려면 시간이 많이 걸리기 때문에 커넥션에 host나 Port등의 데이터를 하드코딩 하기보다는 환경변수를 둬서 스테이지에 맞는 커넥션 옵션을 사용하도록 한다.

**connection.ts**

```ts
const {
    LOCAL_HOST,
    LOCAL_PORT,
    LOCAL_USER_NAME,
    LOCAL_PASSWORD,
    LOCAL_DATABASE,
} = process.env;

const connection: Connection = createConnection({
    type: "mysql",
    host: LOCAL_HOST,
    port: LOCAL_PORT,
    username: LOCAL_USER_NAME,
    password: LOCAL_PASSWORD,
    database: LOCAL_DATABASE,
});
```

**.env파일**

```.env
LOCAL_HOST = localhost
LOCAL_PORT = 3306
LOCAL_USER_NAME = root
LOCAL_PASSWORD = 1234
LOCAL_DATABASE = myservice
```

## 다양한 Connection APIs

**getConnection()**

`creatConnection`으로 만든 커넥션 풀에서 커넥션 객체를 가져온다.

```ts
const connection = getConnection();
```

**getRepository**

커넥션에 있는 Entity의 `Repository`를 가져온다.

```ts
const userRepository: Repository<User> = getRepository(User);
```

**transaction**

여러 개의 데이터베이스 요청에 대해 하나의 트랜잭션을 제공한다. `QueryRunner`라고 하는 독립된 데이터베이스 커넥션을 사용한다. 이 query runner는 getConnection으로 가져온 커넥션 객체로부터 만든다.

트랜잭션은 아래와 같은 순서로 이루어진다. `try-catch`안에서 에러가 발생하면 롤백이 되면서 모든 데이터베이스 요청이 무효화된다.

1. startTransaction 트랜잭션 시작 <br>
   1-1. commitTransaction 트랜잭션 커밋<br>
   1-2. rollbackTransaction 트랜잭션 롤백<br>
2. release 트랜잭션 종료

```ts
import { getConnection } from "typeorm";

const connection = getConnection();
const queryRunner = connection.createQueryRunner();

await queryRunner.connect();

const user1 = await queryRunner.manager.findOne({
    id: 1,
});

await queryRunner.startTransaction();

try {
    user1.name = "juri";
    await queryRunner.manager.save(user1);

    await queryRunner.commitTransaction();
} catch (e) {
    await queryRunner.rollbackTransaction();
} finally {
    await queryRunner.release();
}
```

## 서버리스와 커넥션 풀

lambda 는 다른 lambda 와 독립된 컨테이너로 각각의 lambda가 같은 커넥션 풀을 사용할 수 없을것이라고 생각했다. 그러나 이전에 실행된 lambda가 warm state를 유지하는 동안 다음 lambda가 실행되면 같은 커넥션 풀을 사용할 수 있다고 한다.

![](/img/in-post/connectionpool.jpeg)

그래서 서버리스에서 커넥션 풀을 사용하려면 어떤 단계를 지날지 도식화해보았다. 전제로 현재 실행될 lambda 이전에 사용할 데이터베이스의 커넥션 풀이 생성된 적이 있어야 한다.

1. connection pool이 있는지 확인
2. 있으면 커넥션 객체를 가져옴, 없으면 connection pool을 만들어 커넥션을 사용

위의 간단한 흐름으로 진행될 것이라고 예상된다. lambda의 cold start를 방지하기 위한 warm-up 방법은 여러가지가 있으니 warm state만 유지된다면 무리없이 커넥션 풀을 사용할 수 있지 않을까?
대신 RESTapi를 사용하면 각 엔드포인트마다 함수가 존재할텐데 그 많은 함수가 전부 connection 관련된 코드를 포함해야하니 모듈화가 관건일 것 같다. 혹은 미들웨어를 사용해 함수가 실행되기 전, 실행된 후에 connection 로직이 실행되도록 코드를 포함해도 좋을 것 같다.

### 확인

실험 과정

`create user`, `get user` 두 개의 lambda 함수를 만들었습니다. 아래의 순서대로 lambda 함수를 실행시켜 connection이 어떻게 유지되는지 확인해보았습니다.
![](/img/in-post/connection-lambda.jpg)

먼저, create user는 get user의 connection pool에 영향을 주지 않았습니다. 반면에 get user는 최초로 실행된 이후 다시 실행했을 때 connection pool이 있음을 확인할 수 있습니다.
![](/img/in-post/connection-cloudwatch.png)
즉, 같은 lambda 함수간에 warm state를 유지하는 동안 connection pool을 공유할 수 있습니다.
