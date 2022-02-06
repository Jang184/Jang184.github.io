---
title: SQL GROUP BY로 그룹화하기
category: SQL
tag: TIL
---

## GROUP BY로 그룹화하기

`GROUP BY`는 집계함수로 넘길 집합을 그룹으로 나눠준다.

|no|name|
|:--:|:--:|
|1|A|
|2|A|
|3|B|
|4|C|
|5|NULL|

```sql
SELECT name FROM test_table GROUP BY name;
```

|name|
|-|
|A|
|B|
|C|
|NULL|

위와 같이 지정된 컬럼의 값이 같은 행이 하나의 그룹으로 묶인다. `SELECT`가 name 열을 지정했으므로 name 열이 그룹화되어 반환된다. GROUP BY는 <u>`DISTINCT`와 같이 중복을 제거하는 효과</u>가 있다.

## DISTINCT와의 차이점

집계함수와 사용하지 않는 경우, 둘의 차이점은 없다. <br>
`GROUP BY`는 그룹화된 각 그룹이 하나의 집합으로 집계함수의 인수로 넘겨지게 한다.


|no|name|quantity|
|:--:|:--:|-|
|1|A|1|
|2|A|2|
|3|B|10|
|4|C|3|
|5|NULL|NULL|

```sql
SELECT name, COUNT(name), SUM(quantity)
FROM test_table GROUP BY name;
```

|name|COUNT(name)|SUM(quantity)|
|-|-|-|
|NULL|0|NULL|
|A|2|3|
|B|1|10|
|C|1|3|

A그룹에 속하는 두 개의 열이 한 그룹으로 집계함수의 인수로 넘겨진다.
<br>
예를 들어, 매출관련 데이터를 다룬다고 하자. 점포별, 상품별, 월별 등 특정 단위로 집계할 때 `GROUP BY`를 유용하게 쓸 수 있다.

## HAVING 으로 조건 지정하기

집계함수는 `WHERE`을 사용해 조건을 지정할 수 없다. 아래와 같은 명령은 에러가 발생한다.

```sql
SELECT name, COUNT(name) FROM test_table
WHERE COUNT(name)=1 GROUP BY name;
```
에러의 원인을 알기 위해서 우선 내부처리 순서를 알아야 한다.

> WHERE -> GROUP BY -> SELECT -> ORDER BY

WHERE이 조건에 맞는 데이터를 검색하는 것이 GROUP BY가 그룹화하는 것보다 먼저 처리된다. SELECT 에서 지정한 별명을 WHERE의 조건문에 사용할 수 없는 것과 같은 이유로 그룹화가 필요한 집계함수는 WHERE로 조건을 지정할 수 없다. 그래서 집계한 결과에서 조건에 맞는 값을 찾기 위해 사용하는 것이 `HAVING`이다.

|name|COUNT(name)|
|-|-|
|NULL|0|
|A|2|
|B|1|
|C|1|

```sql
SELECT name, COUNT(name) FROM test_table
GROUP BY name HAVING COUNT(name) = 1;
```

|name|COUNT(name)|
|-|-|
|B|1|
|C|1|

순서는 아래와 같다. 

1. WHERE로 검색
2. GROUP BY로 그룹화
3. HAVING 조건검색

GROUP BY보다 나중에 처리되는 ORDER BY는 집계함수를 사용할 수 있다. 하지만 SELECT보다 먼저 처리되므로 별명을 사용할 수는 없다. 

> WHERE -> GROUP BY -> HAVING -> SELECT -> ORDER BY
```sql
SELECT name, COUNT(name),SUM(quantity)
FROM test_table
GROUP BY name
ORDER BY SUM(quantity) DESC;
```
위와 같이, ORDER BY를 사용할 수 있다. 
```sql
SELECT name AS n, COUNT(name) AS cn
FROM test_table
GROUP BY n
HAVING cn = 1;
```
하지만 위와 같은 코드는 실행할 수 없다. 

## 복수의 열 그룹화하기

`GROUP BY에 지정한 컬럼 이외의 컬럼`은 집계함수를 사용하지 않은 채 SELECT에 사용할 수 없다. 예를 들어, GROUP BY에서 name 컬럼을 그룹화했을 때 SELECT에서 no 열을 지정할 수 없다. name이 A인 두 개의 컬럼이 각각 no열에 1과 2를 가지고 있기 때문에 어떤 값을 반환해야 할지 정할 수 없기때문에 에러가 발생한다.

