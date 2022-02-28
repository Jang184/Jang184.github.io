---
layout: post
header-style: text
title: TypeORM - EntityManager 와 Repository
author: Juri
tags:
    - typeorm
---

## EntityManager 와 Repository

### EntityManager

> EntityManager를 사용해서 Entity의 `insert`, `update`, `delete` 등 처리가 가능하다.

```ts
import { getConnection, EntityManager } from "typeorm";
import User from "./User";

const entityManager: EntityManager = getConnection().manager;
const user: User = await entityManager.findOne(User, 1);
user.name = "Juri";
await entityManager.save(user);
```

## Repository

> EntityManager와 같이 동작하지만 `concrete entity`에 한정한다.

```ts
import { getRepository, Repository } from "typeorm";
import User from "./User";

const userRepository: Repository<User> = getConnection().getRepository(User);
const user: User = await userRepository.findOne(1);
user.name = "Jun";
await userRepository.save(user);
```

-   **Repository** : Regular repository for any entity.
-   **TreeRepository** : Repository, extensions of Repository used for tree-entities (`@Tree`)
-   **MongoRepository** : Repository with special functions used only with MongoDB

## Custom Repository

커스텀 repository를 만들어서 사용할 수도 있다.

```ts
@EntityRepository(User)
export class UserRepository extends Repository<User> {
    findByName(Firstname: string, lastName: string) {
        return this.findOne({ firstName, lastName });
    }
}
```

User Repository를 확장해 만든 커스텀 Repository 내부에 메소드(`findByName`)를 만들어 사용한다.

```ts
const userRepository = getCustomRepository(UserRepository);
const user = new User();
user.firstName = "Juri";
user.lastName = "Jang";
await userRepository.save(user);

const juri = await userRepository.findByName("Juri", "Jang");
```

## EntityManager API

**connection**

```ts
const connection = manager.connection;
```

**queryRunner**
Entitymanager의 트랜잭션 안에서만 사용할 수 있다.

```ts
const queryRunner = manager.queryRunner;
```

**query**
로우쿼리를 실행할 수 있다.

```ts
const rawData = await manager.query("SELECT * FROM USER");
```

**save**
entity 나 entity의 배열을 저장할 수 있다. 데이터베이스에 entity가 이미 존재하면 update를 하고 없으면 insert를 한다. (`upsert`) entity의 배열이 저장될 때 하나의 트랜잭션에서 실행된다.

```ts
await manager.save([category1, category2, categoryy3]);
```

**remove**
위의 save 와 반대로 entity 나 entity의 배열을 삭제할 수 있다. 마찬가지로 하나의 트랜잭션에서 실행된다.

**increment**
지정한 컬럼의 값을 지정한 값만큼 증가시킨다.

```ts
await manager.increment(User, { firstName: "Juri" }, "count", 1);
```

**find**
주어진 조건에 맞는 모든 entity를 찾는다. `array`를 반환한다.

```ts
const users = await manager.find(User, { firstName: "Jang" });
```

**findOne**
주어진 조건에 맞는 entity중 맨 첫번째 entity를 찾는다.

```ts
const user = await manager.findOne(User, { firstName: "Jang" });
```

**findOneOrFail**
주어진 조건에 맞는 entity를 찾고 없으면 reject 프로미스를 반환한다.

**getRepository**
특정 entity의 `Repository`를 가져온다.

이외에도 **insert**, **update**, **delete** 등의 많은 EntityManager API가 있으니 목적에 맞게 잘 사용하면 된다. Entity Manager API와 Repository API는 매우 유사하고 사용법도 비슷하므로 잘 검색해보고 따라 사용해보면 금방 적용할 수 있을 것이다.
