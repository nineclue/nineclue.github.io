---
layout: post
title: Scala에서 숫자, 문자열 변환
category: programming
tags: scala
---

문자열을 숫자로 변환하고자 할때는 Java의 Integer 클래서에서 제공하는 parseInt 함수를 사용하고 숫자를 문자열로 바꿀 때는 RichInt 클래스에서 제공하는 toBinaryString, toHexString을 사용하면 된다.

{% highlight scala %}
Integer.parseInt("FFFF", 16)
Integer.parseInt("1010", 2)

255.toHexString
255.toBinaryString
{% endhighlight %}