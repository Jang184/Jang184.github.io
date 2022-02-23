---
layout: post
header-style: text
title: TypeORM - QueryBuilder
subtitle: QueryBuilder를 사용해 쿼리를 갖고 놀아보자!
author: Juri
tags:
    - typeorm
--- 

[참고 : TypeORM 공식문서 - query builder ](https://typeorm.io/#/select-query-builder/using-subqueries)

QueryBuilder
---
> 변환한 entity를 자동으로 가져와 SQL쿼리를 편리하게 빌드해준다.


```ts
const firstUser = await connection
    .getRepository(User)
    .createQueryBuilder("user")
    .where("user.id = :id", { id : 1 })
    .getOne();
```
위의 코드는 아래와 같은 SQL 쿼리를 수행한다.

```sql
SELECT 
    user.id as userId,
    user.firstName as userFirstName,
    user.lastName as userLastName
FROM users user
WHERE user.id = 1
```
결과적으로 아래와 같은 User의 인스턴스를 반환한다.

```js
User {
    id: 1,
    firstName: "Timber",
    lastName: "Saw"
}
```

### QueryBuilder 사용법

```ts
import {getRepository} from "typeorm";

const user = await getRepository(User)
    .createQueryBuilder("user")
    .where("user.id = :id", { id: 1 })
    .getOne();
```

위의 코드에서 `createQueryBuilder`가 받는 `user`는 sql의 별칭(alias)이다. 이것이 가리키는 바는 아래의 코드와 동일하다.

```ts
createQieryBuilder()
    .select("user")
    .from(User, "user")
```
```sql
SELECT ... FROM users user
```
그렇다면 맨 위의 코드는 아래의 sql 쿼리와 같다고 할 수 있다.

```ts
SELECT ... FROM users user WHERE user.id = 1
```

### QueryBuilder로 결과 얻기

**getOne**

하나의 결과값을 얻을 수 있다.

```ts
const juri = await GetRepository(User)
    .createQueryBuilder("user")
    .where("user.id = :id OR user.name = :name, { id: 1, name: "Juri" })
    .getOne()
```

**getOneOrFail**

getOne과 같이 하나의 결과값을 얻을 수 있고 만약 결과가 없다면 `EntityNotFoundError`를 반환한다.

```ts
const juri = await GetRepository(User)
    .createQueryBuilder("user")
    .where("user.id = :id OR user.name = :name, { id: 1, name: "Juri" })
    .getOneOrFail()
```

**getMany**

여러 개의 결과값을 얻을 수 있다.

```ts
const users = await getRepository(USer)
    .createQueryBuilder("user")
    .getMany();
```

### paramers를 사용해서 데이터 특정하기

```ts
.where("user.name = :name", { name: "Juri" })
```
위의 코드는 아래의 코드를 짧게 축약해 작성한 것이다.

```ts
.where("user.name= :name")
.setParameter("name", "Juri")
```

배열 형태도 지원한다.

```ts
.where("user.name IN (:...names)", { names: [ "Juri". "Jun", "Mong"]  })
```

### 여러 개의 조건 붙이기

**and**
```ts
createQueryBuilder("user")
    .where("user.firstName = :firstName", { firstName: "Juri" })
    .andWhere("user.lastName = :lastName", { lastName: "Jang" });
```
```sql
SELECT ... FROM users user WHERE user.firstName = "Juri" AND user.lastName = "Jang"
```
**or**

```ts
createQueryBuilder("user")
    .where("user.firstName = :firstName", { firstName: "Juri" })
    .orWhere("user.lastName = :lastName", { lastName: "Jang" });
```

**Brackets를 사용한 복잡한 where문**

```ts
createQueryBuilder("user")
    .where("user.registered = :registered", { registered: true })
    .andWhere(new Brackets(qb => {
        qb.where("user.firstName = :firstName", { firstName: "Juri" })
            .orWhere("user.lastName = :lastName", { lastName: "Jang" })
    }))
```

```ts
SELECT ... FROM users user 
WHERE user.registered = true AND (user.firstName = "Juri" OR user.lastName = "Jang")
```

### HAVING

```ts
createQueryBuilder("user")
    .having("user.name = :name". { name: "Juri" })
```

```sql
SELECT ... FROM users user HAVING user.name = "Juri"
```

### ORDER BY

```ts
createQueryBuilder("user")
    .orderBy("user.id")
```

```sql
SELECT ... FROM users user ORDER BY user.id
```

`DESC`나 `ASC`의 옵션을 줘서 정렬할 수 있다.

### Joining

아래의 두 Entity가 있다. 

```ts
import {Entity, PrimaryGenereatedColumn, Column, OneToMany} from "typeorm";
import {Photo} from "./Photo";

@Entity()
export class User {

    @PrimaryGenereatedColumn()
    id: number;

    @Column()
    name: string;

    @OneToMany(type => Photo, photo => photo.user)
    photos: Photo[];
}
```
```ts
import {Entity, PrimaryGeneratedColumn, Column, ManyToOne} from "typeorm";
import {User} from "./User";

@Entity()
export class photo {
    
    @PrimaryGeneratedColum()
    id: number;

    @Column()
    url: string;

    @ManyToOne(type => User, user => user.photos)
    user: User;
}
```
사용자 주리의 모든 사진을 가져와보자.

```ts
const user = await createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .where "user.name = :name", { name: "Juri" }
```

결과값은 아래와 같다.

```ts
{
    id: 1,
    name: "Juri",
    photos: [{
        id: 1,
        url: "me-with-bom.jpg"
    },{
        id: 2,
        url: "me-with-mom.jpg"
    }]
}
```

`leftJoinAndSelect`가 자동으로 주리의 모든 사진을 가져왔다. 첫번째 매개변수는 가져오고자 하는 관계(relation)이고 두번째 매개변수는 이 관계 테이블에 할당하고자 하는 별칭(alias)이다. 



예를 들어, 주리의 사진 중 지워지지 않은 것들만 가져온다고 하자.

```ts
const user = await createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .where("user.name = :name", { name: "Juri" })
    .andWhere("photo.isRemoved = :isRemoved", {isRemoved: false })
    .getOne();
```

위의 코드는 아래의 sql 쿼리를 실행한다.

```sql
SELECT user.*, photo.* FROM users user
    LEFT JOIN photos photo ON photo.user = user.id
    WHERE user.name = "Juri" AND photo.isRemoved = FALSE
```

WHERE을 사용하지 않고 join 에 조건을 추가하는 방법도 있다.

```ts
SELECT user.*, photo.* FROM users user
    LEFT JOIN photos photo ON photo.user = user.id AND photo.isRemoved = FALSE
    WHERE user.name = "Juri"
```

### pagination

**take**

```ts
const users = await getRepository(User)
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .take(10)
    .getMany();
```

처음 `10명`의 사용자를 사진과 함께 반환한다.

**skip**

```ts
const users = await getRepository(User)
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .skip(10)
    .getMany();
```

처음 10명의 사용자를 제외한 모든 사용자를 사진과 함께 반환한다.

**combination**

```ts
const users = await getRepository(User)
    .createQueryBuilder("user")
    .leftJoinAndSelect("user.photos", "photo")
    .skip(5)
    .take(10)
    .getMany();
```

처음 5명의 사용자를 건너뛰고 그 다음 10명의 사용자에 대한 정보를 반환한다.

### subquery

서브쿼리는 `FROM`, `WHERE`, `JOIN` 에서 함께 사용할 수 있다.

```ts
const qb = await getRepository(Post).createQueryBuilder("Post");
const posts = qb
    .where("post.title IN " + 
        qb.subQuery()
            .select("user.name")
            .from( User, "user")
            .where("user.registered = :registered")
            .getQuery())
    .setParameter("registered", true)
    .getMany();
```

```ts
const posts = await connection.getRepository(Post)
    .createQueryBuilder("post")
    .where(qb => {
        const subQuery = qb.subQuery()
            .select("user.name")
            .from(User, "user")
            .where("user.registered = :registered")
            .getQuery();
        return "post.title IN " + subQuery;
    })
    .setParameter("registered", true)
    .getMany();
```


이외에도 `DISTINC` , `GROUP BY`, `LIMIT`, `OFFSET`등 다양한 SQL 표현을 사용할 수 있다.