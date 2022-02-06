---
title: SQL ORDER BY로 정렬하기
category: SQL
tag: TIL
---
`ORDER BY`를 이용해 컬럼을 지정하고 정렬할 수 있다.

> SELECT 컬럼명 FROM 테이블명 WHERE 조건 ORDER BY 컬럼명1, 컬럼명2

### 1. 여러 개의 컬럼 정렬하기
   
**test table**

|A|B|
|:---:|:---:|
|1|1|
|2|1|
|2|2|
|1|3|
|1|2|

--------

```sql
SELECT * FROM test_table ORDER BY A,B;
```
A 컬럼으로 정렬한 후, B 컬럼으로 정렬한다.   

|A|B|
|:--:|:--:|
|1|1|
|1|2|
|1|3|
|2|1|
|2|2|

```sql
SELECT * FROM test_table ORDER BY B,A;
```
B 컬럼으로 정렬한 후, A 컬럼으로 정렬한다.

|A|B|
|:--:|:--:|
|1|1|
|2|1|
|1|2|
|2|2|
|1|3|


### 2. NULL 값의 정렬순서
`ORDER BY`로 지정한 컬럼에서 NULL 값을 갖는 로우는 가장 먼저 혹은 가장 나중에 표시된다.
데이터베이스 제품마다 기준이 다르지만 MySQL의 경우엔 NULL 값을 <u>가장 작은 값</u>으로 취급해 오름차순에서는 가장 먼저, 내림차순에서는 가장 나중에 표시한다.