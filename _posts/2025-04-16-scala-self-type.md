---
layout: post
title: Scala의 self type
date: 2025-04-16 11:19
category: programming
tags: scala
---

Scala 코드를 읽다보면 가끔 class나 trait 정의에  `{ self =>` 혹은 `{ outer =>` 와 같은 부분이 붙어있는 것을 볼 수 있는데 이를 **self type** 혹은 **self reference**라 부릅니다.

* self 인자에 `=> this: User`와 같이 형이 정해져 있는 경우에는 이 class 혹은 trait을 구현하는 쪽에서는 `User` 같이 지정한 형을 함께 구현해야 함을 지정하는 역할을 합니다. 서로 직접적인 관계는 없지만 지정한 형의 변수나 함수를 사용할 필요가 있을 때 유용하게 쓰일 수 있습니다.
``` scala
trait User:
  def username: String

trait Tweeter:
  this: User =>  // reassign this
  def tweet(tweetText: String) = println(s"$username: $tweetText")

class VerifiedTweeter(val username_ : String) extends Tweeter, User:  // We mixin User because Tweeter required it
  def username = s"real $username_"

val realBeyoncé = VerifiedTweeter("Beyoncé")
realBeyoncé.tweet("Just spilled my glass of lemonade")  // prints "real Beyoncé: Just spilled my glass of lemonade"
```

* self 인자에 `=> outer`와 같이 형을 지정하지 않고 사용하는 경우에는 보통 class나 trait의 내부에 정의된 class나 trait에서 외부의 함수나 변수를 참조할 때 사용할 수 있습니다. 
``` scala
class MyOuter { outer =>
  // this == outer
  class MyInner {
    // this != outer
    def creator = outer
  }
}
```

참조
1. [Scala official document](https://docs.scala-lang.org/tour/self-types.html)
1. [Rock the JVM](https://rockthejvm.com/articles/self-types-in-scala-overview)
1. [Stack Overflow](https://stackoverflow.com/questions/11666374/scala-outer-syntax)