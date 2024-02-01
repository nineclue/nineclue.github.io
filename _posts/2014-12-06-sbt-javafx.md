---
layout: post
title: sbt에서 JavaFx실행시 css 파일 문제
category: setup
tags: scala javafx
---

Java 8이 발표되면서 javafx가 기본적으로 포함되어 설치되기 때문에 특별히 build.sbt를 만들지 않고도 scala에서 javafx를 불러 쓸 수 있게 되었다. 프로그램을 실행시키면 `WARNING: Resource "com/sun/javafx/scene/control/skin/modena/modena.css" not found.`와 같은 메시지가 나오는데 이는 `modena.css`가 들어있는 *jfxrt.jar* 파일을 찾지 못해서라고 한다. 다음과 같은 줄을 build.sbt에 추가시켜주면 문제없이 실행된다.

`unmanagedJars in Compile += Attributed.blank(file(System.getenv("JAVA_HOME") + "/jre/lib/ext/jfxrt.jar"))`

![그냥 올려보는 짤방]({{ site.url }}/assets/javafx_modena.png)