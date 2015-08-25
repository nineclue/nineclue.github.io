---
layout: post
title: iPython에서 한글 입력
category: programming
tags: python
---

![iPython 노트북에서 한글 입력]({{site.url}}/assets/ipython_notebook_input.png)

iPython의 노트북은 브라우저에서 파이썬 코드를 입력하고 브라우저에서 바로 결과를 볼 수 있어 언어를 배우거나 프로토타이핑용으로 사용하기 좋다. 콘솔에서의 `input` 명령어는 한글 사용시에 제대로 동작하지 않고 `raw_input`을 사용해야 하며 한글 문자열은 앞에 `u`를 붙여 unicode임을 명시하고 (윈도우즈 XP에서) 결과값은 `encoding` 함수로 바꿔주어야 제대로 표시된다.
