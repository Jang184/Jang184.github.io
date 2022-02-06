---
title: SQL SELF JOIN 
category: SQL
tag: TIL
---

## Self JOIN

### 예제1

[Leetcode-Employees Earning More Than Their Managers](https://leetcode.com/problems/employees-earning-more-than-their-managers/)

한 테이블에 같은 테이블을 JOIN할 수 있다. 동일한 테이블을 구분하기 위해서 `alias`를 사용해야한다.

```sql
SELECT Employee.name AS Employee
FROM Employee
	 INNER JOIN Employee AS Manager ON Employee.managerId = Manager.id
WHERE Employee.salary > Manager.salary
```

`Left JOIN`을 사용하면 managerId가 없는 Sam과 Max의 데이터도 출력된다.

### 예제2

[Leetcode-Rising Temperature](https://leetcode.com/problems/rising-temperature/)


```sql
SELECT Today.id
From Weather Today
     INNER JOIN Weather Yesterday ON Today.id -1 = Yesterday.id
WHERE Today.temperature > Yesterday.temperature
```

위의 풀이법은 Date가 1일 늘어날때마다 id가 1씩 늘어나는 것을 전제로 한다. 만약 Date가 역순으로 정렬되어 있는 데이터라면 다른 방법을 사용해야한다.


## 시간 다루기

### DATE_ADD(기준날짜, INTERVAL) 시간더하기
```
SELECT DATE_ADD(NOW(), INTERVAL 1 SECOND)
SELECT DATE_ADD(NOW(), INTERVAL 1 HOUR)
SELECT DATE_ADD(NOW(), INTERVAL 1 DAY)
SELECT DATE_ADD(NOW(), INTERVAL 1 MONTH)
SELECT DATE_ADD(NOW(), INTERVAL 1 YEAR)
SELECT DATE_ADD(NOW(), INTERVAL -1 YEAR)
```
### DATE_SUB(기준날짜, INTERVAL) 시간빼기
```
SELECT DATE_SUB(NOW(), INTERVAL 1 SECOND)
```
id가 아닌 Date를 기준으로 JOIN 한다.

```sql
SELECT Today.id
From Weather Today
     INNER JOIN Weather Yesterday 
     ON DATE_SUB(Today.recordDate, INTERVAL 1 DAY) = Yesterday.recordDate
WHERE Today.temperature > Yesterday.temperature
```