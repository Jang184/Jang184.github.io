---
title: SQL 데이터베이스 객체
category: SQL
tag: TIL
---

## 데이터베이스 객체

>테이블이나 뷰, 인덱스 등 데이터베이스 내에서 정의하는 모든 것을 일컫는다.

객체는 이름을 갖는다. 데이터베이스 내에서 객체를 작성할 땐 이름이 겹치지 않도록 한다. 또한, 의미 없는 번호 등으로 이름을 붙이지 않는다. 

### 명명규칙
1. 기존 이름이나 예약어와 중복하지 않는다.
2. 숫자로 시작할 수 없다.
3. 언더스코어 이외의 기호는 사용할 수 없다.
4. 한글을 사용할 때는 `""`로 둘러싼다. (MySQL의 경우는 `\`\``로)
5. 시스템이 허용하는 길이를 초과하지 않는다.

이름은 객체의 종류와 관계없다는 것에 주의한다. 예를 들어 'test'라는 이름의 `테이블`을 만들면 `테이블`은 물론이고 `뷰`와 같은 `다른 종류의 객체` 역시 같은 이름인 'test'로 작성할 수 없다.

## 스키마

데이터베이스 객체는 `스키마`라는 그릇 안에 만들어진다. 따라서 객체의 이름이 같아도 스키마가 다르면 상관없다. 예를 들어 스키마A에 테이블test가 있을 때 스키마B에 테이블test가 존재할 수 있다.



이와 같은 특징때문에 데이터베이스 객체는 `스키마 객체`라 불리기도 한다. 스키마는 SQL명령의 DDL(Data Definition Language)을 이용하여 정의한다.

## 테이블 작성/삭제/변경

### 1. 테이블 작성
 
 >CREATE TABLE 테이블명 (
     열이름 자료형 기본값 NULL값
     ...
     )

 ```sql
 CREATE TABLE test (
     no INTEGER NOT NULL,
     a VARCHAR(30),
     b DATE);
 ```
 
 <div align=center><img width="453" alt="스크린샷 2021-11-05 오후 5 36 27" src="https://user-images.githubusercontent.com/81026531/140484938-47308613-ed6a-4a52-8db0-2d296b7a86e0.png"></div>

### 2. 테이블 삭제

>DROP TABLE 테이블명

```sql
DROP TABLE test;
```

SQL은 명령을 실행할 때 확인을 요구하지 않기 때문에 실수로 테이블을 삭제하지 않도록 주의한다.

### 3. 테이블 변경

>ALTER TABLE 테이블명 변경명령

`ALTER` 명령을 사용해 테이블에 저장되어 있는 데이터는 건드리지 않고 구성만 변경할 수 있다.

#### 3-1.열 추가

>ALTER TABLE 테이블명 ADD 열정의

```sql
ALTER TABLE test ADD newcol INTEGER;
```

<div align=center><img width="457" alt="스크린샷 2021-11-05 오후 5 41 54" src="https://user-images.githubusercontent.com/81026531/140485004-df4c7833-af08-490f-ad26-71b21f2d4b5f.png"></div>

기존 데이터행이 존재하면 추가한 열의 값이 모두 `NULL`이 된다. 기본값(default)이 지정되어 있으면 기본값으로 데이터가 저장된다. 새롭게 추가하고자 하는 열에 `NOT NULL`제약을 걸고 싶다면 열정의에 기본값을 지정할 필요가 있다.

#### 3-2.열 속성 변경

>ALTER TABLE 테이블명 MODIFY 열정의

```sql
ALTER TABLE test MODIFY newcol VARCHAR(20);
```

<div align=center><img width="451" alt="스크린샷 2021-11-05 오후 5 45 05" src="https://user-images.githubusercontent.com/81026531/140485099-618bdc84-6877-4cb1-9346-e54c2a50324a.png"></div>

기존의 데이터 행이 존재하는 경우엔 속성 변경에 따라 데이터가 변환된다.만약 자료형이 변경되면 테이블에 들어간 데이터의 자료형이 바뀐다. 그 처리과정에서 에러가 발생하면 `ALTER`명령은 실행되지 않는다.

#### 3-3. 열 이름 변경

>ALTER TABLE 테이블명 CHANGE 기존열이름 신규열정의

```sql
ALTER TABLE test CHANGE newcol c VARCHAR(20);
```

<div align=center><img width="439" alt="스크린샷 2021-11-05 오후 5 53 50" src="https://user-images.githubusercontent.com/81026531/140485139-fa17a1ab-f163-4ca7-9627-79c482335222.png"></div>

#### 3-4. 열 삭제

>ALTER TABLE 테이블명 DROP 열명

```sql
ALTER TABLE test DROP c;
```
<div align=center><img width="453" alt="스크린샷 2021-11-05 오후 5 36 27" src="https://user-images.githubusercontent.com/81026531/140485160-6be21a06-54ba-46b7-9a68-86230be4a310.png"></div>