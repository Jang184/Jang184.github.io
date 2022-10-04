---
layout: post
title: BeautifulSoup 로 웹 크롤링 하기
subtitle: 블로그 포스팅 제목을 긁어서 가져와봅시다.
header-style: text
author: Juri
tags:
  - python
  - beautifulsoup
---

피쳐 개발 중에 들어온 스몰잡에서 흥미로운 작업을 해 간단하게 정리해본다.
환율을 웹 페이지에서 크롤링해오는 작업인데 크롤링이라는 거 자체가 말만 들어봤지 직접 해본건 처음이라 흥미로웠다!!

참고 : [가비아 라이브러리](https://library.gabia.com/contents/9239/)

```py
import requests
from bs4 import BeautifulSoup as bs

page = requests.get("https://jang184.github.io") # 제 블로그 주소 입니다 ㅎㅎ
soup = bs(page.text, "html.parser")

elements = soup.select("div.post-preview a > h2")

for index, element in enumerate(elements, 1):
    print(f"{index}번째 포스팅 제목 : {element.text}")
```

![](/img/in-post/bs-inspector.png)
관리자 도구에서 html 코드를 보면 제목이 class가 `post-preview`인 div의 하위 a 의 h2가 제목임을 확인할 수 있다. beautifulsoup가 이걸 선택할 수 있게 `soup.select()`안에 전달해준다.

![](/img/in-post/bs-terminal.png)

앗 신기해라.. 내 블로그 포스팅 제목이 모였다 !!
