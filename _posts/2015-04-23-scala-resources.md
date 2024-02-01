---
layout: post
title: Scala에서 리소스 파일 읽기
category: programming
tags: scala tip
---

스칼라는 자바와 마찬가지로 /src/main/resources 디렉토리에 리소스 파일들을 저장하는 것으로 간주한다.
`getClass.getResource`는 해당 파일의 URL을, `getClass.getResourceAsStream`은 Stream을 반환하며 `getResourceAsStream`은 리소스가 jar로 패킹되어도 잘 동작하지만 `getResource`는 문제를 일으킬 수 있다.
파일 이름 앞에는 “/“를 붙여준다.