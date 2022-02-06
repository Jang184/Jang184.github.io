---
title: SQL DELETE로 데이터 삭제/UPDATE로 데이터 갱신
category: SQL
tag: TIL
---

## DELETE로 데이터 삭제하기

> DELETE FROM 테이블명 WHERE 조건

```
DELETE FROM test_table;
```
test 테이블의 모든 데이터가 삭제된다. `WHERE`을 사용해 삭제할 데이터를 지정할 수 있다. WHERE 뒤에 오는 조건과 일치하는 모든 행이 삭제된다.<br>
`ORDER BY`는 행을 삭제할 때 데이터를 정렬하는 것이 의미가 없기때문에 DELETE와 함께 사용할 수 없다.

## UPDATE로 데이터 갱신하기

> UPDATE 테이블명 SET 컬럼=값, 컬럼=값,... WHERE 조건

`SET`을 사용해서 갱신할 컬럼과 값을 지정한다. 

```sql
UPDATE test_table 
SET A = 456
WHERE no=2;
```
갱신 전

|no|A|
|:--:|:--:|
|1|ABC|
|2|NULL|

갱신 후

|no|A|
|:--:|:--:|
|1|ABC|
|2|456|

DELETE와 마찬가지로 WHERE 뒤에 오는 조건과 일치하는 모든 행이 갱신된다.

### 예제) UPDATE로 갱신할 때 주의사항

```sql
UPDATE test_table SET no=no+1;
```

`WHERE`이 지정되어 있지 않으므로 갱신 대상은 테이블의 모든 행이 된다. 또한, 위의 코드처럼 SET뒤에 오는 갱신할 값을 컬럼이 포함된 식으로 표기할 수 있다. 

갱신 전

|no|A|
|:--:|:--:|
|1|ABC|
|2|456|

갱신 후
-> `no`에  1이 더해졌다.

|no|A|
|:--:|:--:|
|2|ABC|
|3|456|

### 예제2) 여러 개의 컬럼 갱신하기

예를 들어, 두 개의 컬럼을 갱신한다고 할 때



1) 두 구문으로 나눠서 UPDATE<br>
2) 하나로 묶어서 UPDATE



두 가지의 방법이 있다. UPDATE 명령을 따로 실행하는 전자보다 후자가 더 효율적이다. 

|no|A|B|
|:--:|:--:|:--:|
|2|ABC|DEF|
|3|456|789|

```sql
UPDATE test_table SET no=no+1, A=no;
```

|no|A|B|
|:--:|:--:|:--:|
|3|3|DEF|
|4|4|789|

1. no 의 값에 1을 더하여 no 에 저장한다.
2. 그 값이 A에 대입된다.

### NULL 초기화
```sql
UPDATE test_table SET A = NULL;
```
UPDATE로 셀 값을 `NULL`로 갱신하는 것을 말한다.

|no|A|B|
|:--:|:--:|:--:|
|3|NULL|DEF|
|4|NULL|789|

만약, 컬럼에 `NOT NULL` 제약이 설정되어 있다면 `NULL`이 허용되지 않는다. 