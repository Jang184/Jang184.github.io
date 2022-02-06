---
title: SQL 문자열 자르기
category: SQL
tag: TIL
---

## 문자열을 자르는 내장함수

### LEFT

>LEFT(컬럼명 또는 문자열, 문자열의 길이)

```sql
SELECT LEFT("test_text", 4)

// "test"
```

### RIGHT

>RIGHT(컬럼명 또는 문자열, 문자열의 길이)

```sql
SELECT RIGHT("test_text", 4)

// "text"
```

### SUBSTR

>SUBSTR(컬럼명 또는 문자열, 시작위치, 길이)

```sql
SELECT SUBSTR("20211229", 1, 4)

// "2021"
```
