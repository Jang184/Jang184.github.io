---
title: SQL 서브쿼리
category: SQL
tag: TIL
---

## 서브쿼리

SQL 명령문의 `SELECT`, `FROM`, `WHERE` 뒤에 SELECT 명령을 넣어 사용할 수 있다. 

```sql
DELETE FROM
test_table WHERE a = (
    SELECT MIN(a) FROM test_table;
)
```
위의 SQL문에서 `WHERE` 뒤에 기술되는 `SELECT`문을 `서브쿼리`라고 한다.

### DELETE에 WHERE을 넣은 서브쿼리 사용하기

<p>예시</p>

|no|a|
|:-:|:-:|
|1|100|
|2|90|
|3|20|
|4|80|

위의 테이블에서 가장 작은 a값을 갖는 행을 삭제하려고 한다.

1. a 열의 값이 가장 작은 행을 찾는다. -> SELECT MIN(a) FROM test_table
2. 1번의 값을 지운다. -> DELETE FROM test_table WHERE a = (1번 SQL문)

```sql
DELETE FROM
test_table WHERE a = (
    SELECT MIN(a) FROM test_table
)
```
<p>결과</p>

|no|a|
|:-:|:-:|
|1|100|
|2|90|
|4|80|

이와 같이 `서브쿼리`를 사용해 `DELETE`와 `SELECT`를 결합시킬 수 있다. 실행 순서는 1번의 서브쿼리 부분을 먼저 수행하고 DELETE 명령을 수행한다고 생각하면 된다.<br>
<p style='font-size:14px'>MySQL에서는 위와 같은 코드를 실행하면 에러가 발생한다. 데이터를 추가하거나 갱신할 경우 동일한 테이블을 서브쿼리에서 사용할 수 없도록 되어있기 때문이다.</p>

## 스칼라 값

> SELECT 명령이 하나의 값만 반환하는 것을 '스칼라값을 반환한다'고 표현한다.

<p>예시</p>

|SUM(a)|
|:-:|
|100|

`스칼라값`을 반환하는 SELECT 명령은 서브쿼리로 사용하기 쉽다. `WHERE`에서 `=`연산자로 서브쿼리를 비교할 수 있기 때문이다. 

## SELECT에서 서브쿼리 사용하기

<p>예시</p>

|no|a|
|:-:|:-:|
|1|100|
|2|90|
|4|80|

```sql
SELECT
    (SELECT COUNT(*) FROM test_table) AS sql1;
```
<p>결과</p>

|sql1|
|:-:|
|3|

상부의 `SELECT`명령에서 `FROM`을 생략할 수 있다.

## SET에서 서브쿼리 사용하기

`SET`은 `UPDATE`와 함께 쓰인다. `SET`에서 서브쿼리를 사용할 때는 스칼라 서브쿼리를 지정해야 한다. 예시로 테이블의 a열을 모두 a열의 최대값으로 갱신하려고 한다.

```sql
UPDATE test_table
SET a = (
    SELECT MAX(a) FROM test_table
);
```
<p>결과</p>

|no|a|
|:-:|:-:|
|1|100|
|2|100|
|4|100|

## FROM 에서 서브쿼리 사용하기

`FROM`뒤에 테이블이 아닌 다른 명령문을 지정할 수 있다.

```sql
SELECT * FROM (SELECT * FROM test_table) test;
```

SELECT 안에 SELECT 명령이 들어있는 이와 같은 구조를 `중첩구조`라고 한다. `test`는 서브쿼리의 별명을 지정한 것이다. 3단계의 중첩구조도 가능하다.

## INSERT 와 서브쿼리

`INSERT`와 서브쿼리를 조합해 사용할 땐 `VALUES` 명령을 사용할 수 있다. 서브쿼리는 스칼라 서브쿼리로 지정하도록 한다.

```sql
INSERT INTO test_table2 VALUES(
    (SELECT COUNT(*) FROM test_table)
);
```
<p>결과</p>

|a|
|:-:|
|3|
