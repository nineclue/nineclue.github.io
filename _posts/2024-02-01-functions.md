---
layout: post
title: Functions
date: 2024-02-01 09:00
category: programming
tags: scala
---

## 함수 정의
* def로 정의한 것은 type이 없으며 object나 class내부여야 한다. 
* val로 정의한 것은 type을 가지며 `andThen`과 같은 추가 함수를 사용할 수 있다
* **eta expansion** : def 함수를 val 형식으로 사용하려면 `f1 _`과 같이 underbar 사용하면 변환된다. 필요하면 `f1 _:String` 같이 type 붙여주자

## curried function
* 인자들을 사용 순서에 따라 () 그룹으로 분리
* implicit 인자를 전달할때에도 흔히 사용된다
* generic 사용하는 경우 **함수 인자들의 type inference는 그룹별로 한번에 일어나므로** 아래와 같은 경우 컴파일 되지 않는다. currying을 사용하면 T가 Int라는 것을 컴파일러가 알고 에러 없이 컴파일된다.
{% highlight scala %}
def withListItems[T](list : List[T],f : T => Unit) : Unit = {
   list match { 
     case Nil => () 
     case h :: t => 
       f(h)
       withListItems(t,f) 
   } 
 } 
withListItems(List(1,2,3,4), number => println(number + 2)) // does not compile
{% endhighlight %}

## partial function
* `PartialFunction` trait을 사용하여 정의하며 `isDefinedAt`, `apply` 함수를 지정해야 한다
* case 문으로 정의할 수도 있고 `orElse` 등으로 compose 가능
{% highlight scala %}
val isWorkingAge : PartialFunction[Int,String] = {
  case age if age >= 18 && age <= 60 => s"You are $age years old within working age"
}

val isYoung : PartialFunction[Int,String] = {
  case age if age < 18 => s"You are less than 18, and not within the working age"
}

val isOld : PartialFunction[Int,String] = {
  case age if age > 60 => s"You are greater than 60 and not within the working age"
}

val verdict = isWorkingAge orElse isYoung orElse isOld

verdict(12) // You are less than 18, and not within the working age
verdict(22) // You are 22 years old within working age
verdict(70) // You are greater than 60 and not within the working age
{% endhighlight %}

## lift
* partial function에 사용하면 whole function이 되고 Option[결과값 type]의 값을 가진다
{% highlight scala %}
def getSqrtRootMessageTotalFunction(x: Double) = {
  squareRoot.lift(x).map(result => s"Square root of ${x} is ${result}")
    .getOrElse(s"Cannot calculate square root for $x")
}

Seq("one", "two", "three").lift(1) // Some("two")
Seq("one", "two", "three").lift(7) // None
{% endhighlight %}
* cats의 Functor에도 lift 사용할 수 있다. Functor의 내용에 적용할 함수를 미리 지정하는 셈
{% highlight scala %}
// ammonite의 경우
import $ivy.`org.typelevel:cats:2.10.0`

val doubleList = Functor[List].lift((x:Int) => x * 2)
doubleList(Range(0, 5).toList)  // List(0, 2, 4, 6, 8)

Functor[List].compose[Option].map(l)(_.length)   // baeldung의 예제... (@.@)???
{% endhighlight %}

#### Functor
* fproduct  
  map과 동일하나 인자와 결과값을 tuple로 반환한다. 바로 Map에 인자로 줄 수 있다.
* compose  
  같은 Functor끼리 결합
{% highlight scala %}
Functor[List].fproduct(List(1,2,3))(_.toString) // List((1, "1"), (2, "2"), (3, "3"))

val listOptOpt = Functor[List].compose(Functor[Option]).compose(Functor[Option])
listOptOpt.fproduct(List(Some(Some(2)), None, Some(None), Some(Some(4))))(_ + 1) 
// List(
  Some(value = Some(value = (2, 3))),
  None,
  Some(value = None),
  Some(value = Some(value = (4, 5)))
)
{% endhighlight %}

## monad transformer
* monad 속에 monad가 있는 경우, 내부의 monad가 아닌 값을 liftF를 사용하여 monad를 적용시키고 있다
{% highlight scala %}
val sayHello: Future[Option[String]] = Future.successful(Some("Say hello to"))
val firstname: Future[String] = Future.successful("Fabio")

def getGreetingsMonadTranformer() = {
  val maybeHello: OptionT[Future, String] = for {
    hello <- OptionT(sayHello)
    name  <- OptionT.liftF(firstname)
  } yield s"$hello $name"

  val result: Future[Option[String]] = maybeHello.value

  Await.result(result, 1.second)
}
{% endhighlight %}
