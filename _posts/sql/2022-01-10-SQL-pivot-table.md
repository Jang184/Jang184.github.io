---
title: SQL CASE를 활용한 테이블 피봇
category: SQL
tag: TIL
---

> 피봇 테이블 : 데이터 처리의 한 기법으로 유용한 정보에 집중할 수 있도록 통계를 재정렬하는 것을 말한다.

## 예제 

먼저, 아래의 코드는 카테고리번호가 1인 상품들의 가격을 반환한다.
```sql
SELECT CASE 
			WHEN categoryid = 1 THEN price
			ELSE NULL 
		END AS category1_price, ProductId, ProductName
FROM Products
```

|category1_price|productID|ProductName|
|-|-|-|-|
|18|1|Chais|
|19|2|Chang|
|29|3|Cho|
10|4|Cheong|

CASE문에 통째로 AVG를 씌운다.

```sql
SELECT AVG(CASE 
			WHEN categoryid = 1 THEN price
			ELSE NULL 
		END) AS category1_price
FROM Products
```

|category1_price|
|-|
|19|

카테고리 번호가 1인 상품들의 가격의 `평균값`을 반환한다.

```sql
SELECT AVG(CASE WHEN categoryid = 1 THEN price	ELSE NULL 
		END) AS category1_price,
		AVG(CASE WHEN categoryid = 2 THEN price	ELSE NULL 
		END) AS category2_price,
		AVG(CASE WHEN categoryid = 3 THEN price	ELSE NULL 
		END) AS category3_price
FROM Products
```

|category1_price|category2_price|category3_price|
|-|-|-|
|19|23.9|16.9|

이와 같이 데이터를 가로로 펼쳐서 표현할 수 있다.
