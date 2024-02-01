---
layout: post
title: Partial Function
date: 2024-01-31 14:30
category: programming
tags: scala
---

Baeldung.com에서 Scala에 관한 문서를 읽다 다음과 같은 내용을 발견했다. 

{% highlight scala %}
def calculateSellingPrice(discount: Double, productPrice: Double): Double = {
  (1 - discount/100) * productPrice
}

val discountApplied = calculateSellingPrice(25, _)
{% endhighlight %}

함수에 partial application을 위해서는 currying을 사용하고 있었는데 위와 같이 밑줄을 사용해서 쓸 수도 있다는 말.

시험해 보니 다음과 같은 함수 만들기와 호출도 가능해진다. 인자들을 굳이 정해지는 순서에 따라 나열하지 않아도 필요한 경우 _를 사용해서 위치를 지정할 수 있다.

{% highlight scala %}
def f[A, B, C](a: A, b: B, c: C) = s"$a, $b, $c"

val g = f("hello", "world", _)
g("!")   // "hello, world, !"
val h = f(123, _, 789)
h(456)   // "123, 456, 789"
val i = f(_, "second", "third")
i("first") // "first, second, third"
val j = f(_, "second", _)
j("1st", "3rd")  // "1st, second, 3rd"
val k = j(_, "3rd")
k("1st") // "1st, second, 3rd"
{% endhighlight %}

가끔씩 partially applied function를 사용하면 표현이 간단해지는 경우가 있는데 예를 들어 문자열 하나를 받아 문자열을 반환하는 함수를 인자로 받는 경우 위에서 k 함수를 만들어 전달하는 경우랄까. 