---
title: SQL INSERT로 데이터 추가하기
category: SQL
tag: TIL
---

## INSERT로 테이블에 행 추가하기
>INSERT INTO 테이블명 VALUES (값1, 값2, ...);

`INSERT`를 사용해 테이블에 행 단위로 데이터를 추가할 수 있다.

```sql
INSERT INTO test_table(no, result) VALUES (1,'ABC');
```

|no|result|
|:--:|:--:|
|1|ABC|



이 때, 데이터를 추가하고자 하는 테이블에 해당 컬럼(no와 result)이 미리 만들어져 있어야 한다.
테이블의 컬럼 구성을 확인할 땐 아래의 `DESC` 명령어를 사용한다.
>DESC test_table;

## 행에 들어갈 데이터의 값을 지정하지 않았을 때

```sql
INSERT INTO test_table(no, result) VALUES 2;
```

|no|result|
|:--:|:--:|
|1|ABC|
|2|NULL|

result 의 행에 들어갈 데이터의 값을 지정하지 않아 자동으로 `NULL`이 저장된다.
테이블 행에 `NOT NULL` 제약을 지정해놓지 않으면 자동으로 default값인 `NULL`이 저장된다.

Default값은 사용자가 지정할 수 있다.
default값이 0으로 지정된 result의 행에 값을 지정하지 않으면
```sql
INSERT INTO test_table(no, result) VALUES 3;
```

|no|result|
|:--:|:--:|
|1|ABC|
|2|NULL|
|3|0|

위의 테이블과 같이 default값인 `0`으로 저장된다.




