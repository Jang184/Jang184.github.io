---
title: MySQL Character Set과 Collation
category: SQL
tag:
    - TIL
    - database
---
[참고](https://mysqldba.tistory.com/154)

## 1. Character Set 과 Collation

> Character Set : 문자와 encoding의 집합. 

> Collation : character set안에서 character들을 비교하기 위한 규칙들의 정의.

`MySQL`은 언어에 따른 여러가지 타입의 character set과 데이터를 비교하는 여러가지 collation을 지원한다. 또한, MySQL은 다른 DBMS와 다르게 서버단위, 데이터베이스 단위, 테이블단위, 컬럼단위로 character set 및 collation을 설정할 수 있다.


character set은 InnoDB 스토리지 엔진에서 설정하여 사용이 가능하다.

> InnoDB(이노DB)는 MySQL을 위한 데이터베이스 엔진으로 따로 설정하지 않으면 default 설정되는 스토리지 엔진이다.

### 1.1 Character Set

대표적으로 사용하는 character set으로 `utf8`이 있다. UTF-8 unicode를 지원하는 문자셋으로 한 단어당 1byte에서 3byte까지 사용한다.

### 1.2 Collation

현재 개발중인 서비스는 한글, 영어가 아닌 언어를 사용하는 유저가 많기 때문에 `utf8mb4_unicode_ci`로 collation을 설정하게 됐다. 또한, Emoji를 지원하기 때문에 유저들이 남기는 댓글이나 설명에 사용하기 좋다. 