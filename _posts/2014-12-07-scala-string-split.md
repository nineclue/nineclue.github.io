---
layout: post
title: readLine 입력을 단어별로 분리하기
category: programming
tags: scala
---

scala에서 입력받은 문자열을 단어별로 분리하려면 String의 split 함수가 regular expression을 지원하므로 다음과 같이 사용할 수 있다.

{% highlight scala %}
val input = """ abc 123
   can you split ? """
input.split("\\s")
// Array[String] = Array("", abc, 123, "", "", "", can, you, split, ?)
{% endhighlight %}

공백이나 개행문자에 의해 나눠지므로 빈 String들이 결과값에 나온다. 따라서 결과에 공백을 제거하는 filter를 더해주면 되겠다.

{% highlight scala %}
def mysplit(s:String) = s.split("\\s").filter(!_.isEmpty)
mysplit(input)
// Array[String] = Array(abc, 123, can, you, split, ?)
{% endhighlight %}
