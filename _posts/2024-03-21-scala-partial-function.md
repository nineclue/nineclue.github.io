---
layout: post
title: Scala의 Partial Function 
date: 2024-03-21 15:04
category: programming
tags: scala, cats
---

다음은 Int형 인자를 받아 Option[Int]를 반환하는 세가지 함수입니다. 
```scala
val f1 = (i: Int) => if i >= 0 && i < 100 then 100.some else Option.empty
val f2 = (i: Int) => Option.when(i >= 100 && i < 1000)(200)
val f3_: PartialFunction[Int, Int] = { case i if i >= 1000 && i <= 10000 }
val f3 = f3_.lift
val f3a = ({ case i if i >= 1000 && i <= 10000 => 300 }: PartialFunction[Int, Int]).lift
```
정해진 범위의 인자를 받으면 함수마다 특정한 값을 반환하는데 범위에 해당하면 Option으로 감싸고 해당하지 않으면 None을 반환하여 범위에 해당하지 않는 것을 나타냅니다. 마지막 함수는 f3를 정의하기 위해 먼저 `PartialFunction` f3_를 정의하고 lift 함수를 사용하였고 한줄로는 f3a와 같은 형태로 정의할 수도 있습니다.

`PartialFunction`은 인자의 범위중 일부에 대해서 반환값이 정해지는 함수의 type입니다. 다시말해 **인자에 따라 값이 있을수도 있고 없을수도 있다**는 것을 표현하는 방법입니다. 이 trait은 isDefinedAt 함수를 가지며 따라서 `f3_.isDefinedAt(999)`를 호출하면 `false`를 반환받습니다.

`PartialFunction`은 `lift` 함수를 사용해서 `PartialFunction[A, B]`를 `Function1[A, Option[B]]`로 만들수 있습니다. 반대로 `Function1[A, Option[B]]` 함수는 unlift를 사용해서 `PartialFunction[A, B]`를 얻을 수 있습니다.

> ##### TIP
> 
> `unlift`는 pattern matching에도 사용할 수 있다고 합니다.
> `val f1.unlift(r1) = 0 : @unchecked`
> 위의 코드를 실행하면 r1은 100의 값을 가집니다.
> 범위를 벗어나는 값을 주면 실행시 MatchError가 발생할 수 있다고 컴파일러가 워닝으로 알려주어 위와 같이 `@unchecked` annotation을 사용하였습니다.
> 다음과 같이 `case` 문에서 해당하지 않는 경우를 함께 사용하면 되겠지요.
> ``` scala
> 0 match
>   case f1.unlift(r1) => s"answer is $r1"
>   case _ => "answer is not defined"
> ```
{: .block-tip}

세가지 함수를 하나로 합치고 싶다면 어떻게해야 할까요? 각각의 범위에 따라 다른 값을 반환하려면 새로 함수를 작성해야 할까요? 

`PartialFunction`은 orElse를 사용하여 합칠 수 있습니다. 첫번째 함수의 범위에 해당하지 않으면 두번째 함수를 호출하는 방식으로 서로 합치는 것이죠. 
```scala
val f = (f1.unlift.orElse(f2.unlift).orElse(f3.unlift)).lift
val ff = (f1.unlift orElse f2.unlift orElse f3.unlift).lift // .을 생략한 문법
```
f1, f2, f3는 Option을 반환하는 함수이므로 먼저 unlift를 사용해서 `PartialFunction`으로 바꾼 다음 합쳐서 다시 lift 함수를 사용해서 Option 반환하는 함수로 바꾸었습니다.

`Kleisli`를 사용하면 `Function1[A, Option[B]]` 함수들을 합칠 수 있습니다. 
```scala
val kf = Kleisli(f1) <+> Kleisli(f2) <+> Kleisli(f3)
```
위에서 정의한 ff와 kf는 같은 동작을 하는 함수들입니다. 실제로 사용할 것 같지 않은 예를 들었지만 실제로 [http4s](https://www.http4s.org) 라이브러리에서 HttpRoutes를 합칠때 Kleisli를 사용하고 있습니다. 