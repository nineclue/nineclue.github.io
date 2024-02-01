---
layout: post
title: Scala에서 다양한 갯수의 인자 받기
category: programming
tags: scala
---

scala에서 다양한 갯수의 인자를 하나의 이름으로 받으려면 인자형 다음에 `*`를 붙여준다. 다음과 같이 실제로는 Array형으로 전달된다.

{% highlight scala %}
def t[T](xs:T*) = xs.getClass

t(0)               // class scala.collection.mutable.WrappedArray$ofInt
{% endhighlight %}

마찬가지로 List, Array 등을 가변인자에 전달하려고 할때는 변수 이름 다음에 `_*`를 붙여준다.

{% highlight scala %}
def sum(xs:Int*):Int = if (xs.isEmpty) 0 else xs.head + sum(xs.tail:_*)

sum()              // 0
sum(1)             // 1
sum(1, 2, 3)       // 6

val ns = List(1, 3, 5)
sum(ns:_*)         // 9
{% endhighlight %}
