---
layout: post
title: scala에서 콘솔입력
category: programming
tags: scala
---

터미널에서 실행시 사용자에게 입력값을 받아야 할때는 `scala.io.StdIn` 객체의 메소드들을 호출하면 된다. 문자열을 받을 경우 `readLine`, 정수는 `readInt`와 같이 type에 따른 메소드들이 마련되어 있다.

예전에는 Console에 정의되어 있었는데 2.11 부터는 io.StdIn으로 옮겨졌다.

다음은 stackoverflow에서 따온 예

{% highlight scala %}
Iterator.continually(io.StdIn.readLine).
	takeWhile(_.nonEmpty).
	foreach(line => println("read :" + line))
{% endhighlight %}
