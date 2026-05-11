---
layout: post
title: Localdate.now without timezone db in scalajs
date: 2026-05-11
category: programming
tags: [scala]
---

scala.js는 javascript로 컴파일 되기 때문에 java의 라이브러리를 바로 사용하지 못합니다. 날짜 관련 함수들을 사용하려면 [scala-java-time](https://cquiroz.github.io/scala-java-time/)을 사용하게 되는데요, 현재 시간을 사용하고자 할때 JVM에서와 같이 단순히 `LocalDate.now()` 함수를 호출하면 시스템의 timezone을 물어보게 되고 모든 timezone을 포함시킨 `scala-java-time-tzdb`를 사용하거나 `sbt`에서 직접 사용하는 timezone을 지정해서 추가해야 합니다.

굳이 여러 TimeZone을 사용할 필요가 없다면 다음과 같이 Zone을 지정해서 사용하면 시스템의 timezone을 물어보지 않고 그냥 현재 zone의 시간을 불러쓸 수 있습니다. Timezone db를 포함시키지 않으니 컴파일 결과 거의 1메가 정도 크기가 줄어드는군요.

```scala
val KST: ZoneOffset = ZoneOffset.of("+09:00")
val clock: Clock = Clock.system(KST)
val today: LocaDate = LocalDate.now(clock)
```
