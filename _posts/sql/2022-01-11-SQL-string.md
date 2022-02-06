---
title: SQL 문자열을 다루는 함수
category: SQL
tag: TIL
---

## 1. LOWER


> LOWER(컬럼명/문자열)

모든 문자를 소문자로 반환한다.

```sql
SELECT LOWER(UserID)
FROM Users
```

## 2. UPPER

> UPPER(컬럼명/문자열)

모든 문자를 대문자로 반환한다.

```sql
SELECT UPPER(UserID)
FROM Users
```

## 3. REPLACE

> REPLACE(컬럼명/문자열, 패턴1, 패턴2)

문자열에 포함된 패턴1을 패턴2로 대체해 반환한다.

```sql
SELECT REPLACE(UserID, 'A', 'B')
FROM Users
```

input: Users테이블

|ID|UserID|
|-|-|
|1|A1|
|2|A2|

output

|ID|UserID|
|-|-|
|1|B1|
|2|B2|

## 4. CONCAT

> CONCAT(컬럼명/문자열, 컬럼명/문자열2, ...)

여러 개의 문자열을 차례대로 연결해 하나의 문자열로 반환한다.

```sql
SELECT CONCAT(ID, ':', UserID) AS User
FROM Users
```

output

|User|
|-|
|1:A1|
|2:A2|

