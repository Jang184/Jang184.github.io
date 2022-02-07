---
layout: post
header-style: text
title: git rebase로 커밋 합치기
author: Juri
tags:
    - git
---

<i>git workflow 를 알고 있다는 전제하에 기술합니다.</i>

## rebase -i

develop브랜치에서 feature브랜치를 만들어서 기능을 개발중이다. `feature/something`에서 세 개의 커밋을 남겼다고 예를 들자.

```bash
[38814dp] commit3
[39571wq] commit2
[10723ke] commit1
[90275fl] develop
```

합치고자 하는 커밋의 `직전 커밋`을 지정한다.

```bash
git rebase -i 90275fl
```

`squash`를 사용해서 커밋을 하나로 만들고 커밋 메세지를 수정한다. rebase가 완료되면 local과 remote의 내용이 달라지면서 `HEAD`가 분리된다.  

```bash
git push origin feature/something -f
```

push 명령에 `-f`를 붙여 rebase한 내용을 강제로 덮어씌운다.

## 결론

각 push당 한 개의 커밋이 남아 깔끔하고 직관적이나 기존의 내용과 달라진 부분을 표시해주지 않아서 코드리뷰를 해주거나 피드백을 해주는 입장에서는 전체 코드를 다시 봐야하는 번거로움은 있다.