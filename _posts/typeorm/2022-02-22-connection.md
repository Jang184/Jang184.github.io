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

```ts
import {
    createConnection, Connection
} from "typeorm";

const connection: Connection = await createConnection({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "root",
    password: 1234,
    database: "myservice"
})
```

`mysql`의 커넥션 폼은 위와 같다. 이 포스팅에서 로컬의 MySQL은 `Docker` 환경에 띄워놓았기 때문에 Docker port로 설정하도록 한다. 커넥션은 두 개 이상 생성할 수 있다.

### Connection Options

`createConnection`을 사용해 커넥션을 생성할 때 몇가지 옵션을 설정할 수 있다.

```ts
const connectionOptions = {
    name: "default",
    type: "mysql",
    synchronize: false,
    logging: true,
    entities: [User, Photo],
    namingStrategy: new SnakeNamingStrategy()
}
```
이외에도 subscribers, migrations 등의 추가적인 option을 설정할 수 있다. 새로 생성한 entity를 connection option에 추가하지 않으면 typeorm이 entity를 인식하지 못한다. **꼭 까먹지않고 새로 생성한 entity를 connection에 추가한다!**

### .env 별로 다른 connection option 사용하기

개발 스테이지에 따라 다른 데이터베이스를 이용하게 된다. 그 때마다 connection을 수동으로 바꾸려면 시간이 많이 걸리기 때문에 커넥션에 host나 Port등의 데이터를 하드코딩 하기보다는 환경변수를 둬서 스테이지에 맞는 커넥션 옵션을 사용하도록 한다.

**connection.ts**
```ts
const {
    LOCAL_HOST,
    LOCAL_PORT,
    LOCAL_USER_NAME,
    LOCAL_PASSWORD,
    LOCAL_DATABASE
} = process.env

const connection: Connection = createConnection({
    type: "mysql",
    host: LOCAL_HOST,
    port: LOCAL_PORT,
    username: LOCAL_USER_NAME,
    password: LOCAL_PASSWORD,
    database: LOCAL_DATABASE
})
```
**.env파일**
```.env
LOCAL_HOST = localhost
LOCAL_PORT = 3306
LOCAL_USER_NAME = root
LOCAL_PASSWORD = 1234
LOCAL_DATABASE = myservice
```


### Connection APIs

**getConnection()**
> `creatConnection`으로 만든 커넥션을 가져온다.

```ts
import {
    getConnection
} from "typeorm";

const connection = getConnection();
```

**getManager**
> 커넥션에서 `EntityManager`를 가져온다.

**getRepository**
> 커넥션에 있는 Entity의 `Repository`를 가져온다.

```ts
import {
    getRepository,
    Repository
} from "typeorm";

const userRepository: Repository<User> = getRepository(User)
```

---

**isConnected**
> 데이터베이스와 실제로 커넥션이 되어있는지 확인할 수 있다.

```ts
const isConnected: boolean = connection.isConnected
```

**Entitymanager**
> manager에 있는 메서드로 커넥션에 있는 Entity를 사용할 수 있다.

```ts
const manager: EntityManager = connection.manager

const user : User = await manager.findOne(1)
``` 

**connect**
> 데이터베이스와 연결한다. 커넥션을 만들면 자동으로 connect를 실행한다.
```ts
await connection.connect()
```

**close**
> 데이터베이스와의 연결을 끊는다.

```ts
await connection.close()
```

**transaction**
> 여러 개의 데이터베이스 요청에 대해 하나의 트랜잭션을 제공한다.

트랜잭션은 아래와 같은 순서로 이루어진다. `try-catch`안에서 에러가 발생하면 롤백이 되면서 모든 데이터베이스 요청이 무효화된다.

1. startTransaction 트랜잭션 시작 <br>
2-1. commitTransaction 트랜잭션 커밋<br>
2-2. rollbackTransaction 트랜잭션 롤백<br>
3. release 트랜잭션 종료


```ts
import {getConnection} from "typeorm";

const connection = getConnection();
const queryRunner = connection.createQueryRunner();

await queryRunner.connect()

const user1 = await queryRunner.manager.findOne({
    id: 1
});

await queryRunner.startTransaction();

try {
    user1.name = "juri"
    await queryRunner.manager.save(user1);

    await queryRunner.commitTransaction();
} catch (e) {
    await queryRunner.rollbackTransaction();
} finally {
    await queryRunner.release()
}
```