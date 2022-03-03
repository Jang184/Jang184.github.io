---
layout: post
header-style: text
title: Python으로 csv 파일 읽고 쓰기
subtitle: MySQL 데이터를 csv로 변환해서 엑셀에 통계자료를 정리해보자.
author: Juri
tags:
    - Python
    - csv
---

MySQL에 있는 데이터를 가공해서 통계 자료로 사용해야 할 때가 있다. 보통은 엑셀이나 구글 스프레드 시트로 통계 자료를 정리하기 때문에 MySQL 데이터를 export할 때 `csv`를 이용한다.

### csv 파일

`csv`파일은 `,`로 구분한 텍스트 데이터를 가리키는 말로

```csv
id,user_id,name
1,dslkaj,juri
2,asdlgk,jun
```

와 같은 형식이다. 엑셀 프로그램에서 csv 파일을 import하면 엑셀 칸에 맞게 자동으로 정리되어 매우 편리하다. python을 이용해서 csv파일을 불러와 가공할 수 있어 통계자료를 정리할 때 노가다를 하지않아도 단순반복작업을 빠르게 수행할 수 있다.

### csv파일 읽기

`csv` 라이브러리를 사용해서 csv 파일을 읽고 쓸 수 있다.

```py
import csv

with open("./userList.csv") as file:
    reader = csv.reader(file)
    for row in reader:
        print(row)
```

csv파일의 첫번째 줄부터 차례대로 print를 찍어보면

```bash
['id', 'user_id', 'name']
['1', 'dslkaj', 'juri']
['2', 'asdlgk', 'jun']
```

와 같이 list의 형태로 나온다.

### csv파일 쓰기

```py
import os

with open("./userList.csv", "a") as file:
    writer = csv.writer(file)
    writer.writerow([3,onsdak,hoto])
```

csv파일을 읽을 때와 비슷하게 csv 파일을 쓸 수 있다. `a`는 csv파일을 추가한다는 옵션으로 읽기는 `r`, 쓰기는 `w`라고 쓴다.
이 때 추가하는 데이터는 리스트 형태로 지정한 csv파일의 '행'에 추가된다.

```bash
['id', 'user_id', 'name']
['1', 'dslkaj', 'juri']
['2', 'asdlgk', 'jun']
['3', 'onsdak', 'hoto']
```

### 응용

왜 갑자기 주언어도 아닌 python으로 csv를 다루게 됐느냐하면.. sparky DB에 쌓인 비디오 조회수를 쉽게 기록하기 위해서 python까지 도달하게 됐다.

달마다 조회수를 기록해야하는 비디오 목록이 따로 있고 그 목록에 있는 비디오의 조회수만 필터링한 후 데이터를 엑셀(스프레드 시트)로 저장해야 하기 때문에

1. MySQL에 있는 조회수 데이터 csv파일로 export하기
2. python으로 csv파일 만들기
3. csv파일을 구글 스프레드 시트에 업로드하기

의 순서로 진행했다.

먼저, MySQL에서 csv파일을 추츨하는 것은 쉽다. **MySQL 워크벤치**를 사용하고 있다면.... CLI로도 가능하지만 굳이 그런 수고를 하진 않았다.

![](/img/in-post/csv-file.png)
디스크 버튼을 누르면 결과창에 있는 데이터를 export할 수 있고 테이블 전체 혹은 데이터베이스 전체를 export 할 수도 있다.

```py
import csv

CSV_PATH_VIEWCOUNT = "./viewCount.csv"
CSV_PATH_VIDEOLIST = "./videoList.csv"
CSV_PATH_RESULT= "./result.csv"

dataList = []
with open(CSV_PATH_VIEWCOUNT, "r") as file:
    reader = csv.reader(file)
    next(file, None)
    for row in reader:
            dataList.append([row[2],row[3]])
```

next는 맨 윗줄의 컬럼 이름은 가공할 필요가 없어서 skip하기 위해서 썼다.

먼저, MySQL에서 가져온 csv파일에 있는 데이터 중 필요한 데이터만 남도록 가공한다. `id`와 `제목`, `video_id`, `조회수` 3개의 컬럼 데이터를 가져왔기 때문에 이 중에서 video_id와 조회수만 남기기 위해서 2, 3번 인덱스의 데이터만 가져온다. 이제 생각해보면 MySQL에서 데이터를 가져올 때 애초에 필요한 데이터만 가져오면 이 단계는 생략할 수 있을 것 같다.

```py
newData = []
with open(CSV_PATH_VIDEOLIST) as file:
    reader = csv.reader(file)
    next(file, None)
    for row in reader:
        for data in dataList:
            if row[1] in data:
                with open(CSV_PATH_RESULT, "a) as w:
                    writer = csv.writer(w)
                    writer.writerow(data)


```

다음으로, videoList에 있는 비디오의 조회수만 가져오기 위해서 비디오 목록을 읽어온다. 인덱스 1번에 video_id가 있기때문에 row[1]을 지정했다.

만약 video_id가 비디오 목록에 있으면 해당 데이터를 새로운 csv파일에 적도록 했다.

마지막으로, csv파일은 엑셀에서 열거나 구글 스프레드 시트에 업로드하면 자동으로 정렬되어 입력되므로 간편하다. 후에 컬럼명을 적어주거나 오름차순으로 정렬을 해서 보기 좋게 정리한다.

## 마치며

파이썬으로 개발에 입문했지만 자바스크립트와 타입스크립트에 맘을 뺏긴 지 오래라서 오랜만에 만난 친구마냥 옛 추억을 더듬으면서 코드 몇자 적어보았다. for문이 왜이렇게 낯선지 ..

자바스크립트로 csv파일을 읽고 쓰는 방법도 찾아보았지만 파이썬이 훨씬 더 간단해보여서 이번엔 파이썬으로 빠르게 코드를 적어보았다. 확실히 통계나 복잡한 계산을 하는데에는 파이썬의 라이브러리가 훨씬 많고 잘 준비돼있는 느낌이다.

언어는 개발에 있어서 장벽이 될 수 없다는 것을 깨달았다. 중요한 것은 논리적인 사고 ..! 그 자체가 아니겠는가
