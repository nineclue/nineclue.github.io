---
layout: post
title: Scala에서 함수형의 variance
category: programming
tags: scala
---

Scala는 다른 객체지향 언어와 같이 어떤 형의 인자를 원하는 함수에 그 형의 하위형을 인자로 사용할 수 있다. 예를 들어 ~~그럴리는 없겠지만~~ `AnyVal`형의 인자를 원하는 함수에 대신 `Double`형의 인자를 줄 수 있다는 말이다. 하위형은 상위형의 모든 값들과 함수들을 가지기 때문이다.

또한 Scala는 함수형 언어로 함수를 인자로 줄 수 있다. 인자를 하나 가지는 함수는 다음과 같은 정의를 가진다.

{% highlight scala %}
trait Function1 [-T1, +R] extends AnyRef
{% endhighlight %}

위와 같이 정의되어 있고 형은 `(T1) => R`이라고 되어 있다. T1형 인자 하나를 받아서 R형 반환값을 가지는 함수인 셈이다. 이때 T1 앞의 -와 R 앞의 +는 함수값에서 상하위를 말하는 것으로 -T1은 contravariance로 인자는 지정한 것보다 상위의 것을 받는 것이 하위이고 +R은 covariance로 반환값은 지정한 것보다 상위의 것을 받는 것이 하위란 의미이다.

다음의 예제는 Coursera의 Programming Language에서 나왔던 ML 소스 코드를 Scala로 나타낸 것이다. 

{% highlight scala %}
object FunctionType extends App {

    class Point1D(val x:Double)
    class Point2D(x:Double, val y:Double) extends Point1D(x)
    class ColorPoint2D(x:Double, y:Double, val color:String) extends Point2D(x, y)


    def distMoved(f:Point2D => Point2D, p:Point2D) = {
        val p2 = f(p)
        val (dx, dy) = (p2.x - p.x, p2.y - p.y)
        math.sqrt(dx*dx + dy*dy)
    }

    // Point2D => Point2D
    def flip(p:Point2D) = new Point2D(-p.x, -p.y)
    // Point2D => ColorPoint2D, covariant return
    def flipGreen(p:Point2D) = new ColorPoint2D(-p.x, -p.y, "Green")

    val d1 = distMoved(flipGreen, new Point2D(3.0, 4.0))
    val d2 = distMoved(flip, new ColorPoint2D(3.0, 4.0, "Green"))
    println(d1, d2)

    // ColorPoint2D => Point2D, covariant argument
    def flipIfGreen(p:ColorPoint2D) = 
        if (p.color == "Green") new Point2D(-p.x, -p.y)
        else new Point2D(p.x, p.y)

    // 에러 발생
    // val d3 = distMoved(flipIfGreen, new Point2D(3.0, 4.0))

    // Point1D => Point2D, contravariant argument
    def flip1D(p:Point1D) = new Point2D(-p.x, 0.0)

    // OK
    val d4 = distMoved(flip1D, new Point2D(3.0, 4.0))

    // Point1D => ColorPoint2D, contravariant argument / covariant return
    def flip1DToGreen2D(p:Point1D) = new ColorPoint2D(-p.x, 0.0, "Green")

    // OK
    val d5 = distMoved(flip1DToGreen2D, new Point2D(3.0, 4.0))
    println(d4, d5)
}
{% endhighlight %}

* `distMoved` 함수는 위치 변환 함수 f와 점 p를 인자로 가지며 p를 f에 적용하여 변환된 점과 원래 점의 거리를 반환한다. 
* Point1D <:< Point2D <:< ColorPoint2D의 관계가 성립한다. 
* `Point2D => Point2D`가 distMoved 함수가 받는 위치 변환함수의 기본 인자형이라면 `Point2D => ColorPoint2D`, `Point1D => Point2D`, `Point1D => ColorPoint2D` 형의 함수는 인자로 사용할 수 있으나 `Point2D => Point1D`나 `ColorPoint2D => Point2D`형의 함수는 인자로 사용할 수 없다. 
* 함수형은 인자(T, Point2D)와 반환값(R, Point2D)으로 구성(T => R, Point2D => Point2D)되며 인자로 전달된 함수(f)를 호출하면 호출된 함수(f)는 인자(T)를 가지고 반환값(R)을 얻으므로 원래 지정한 인자형에서 얻을 수 있는 정보와 같거나 적은 정보를 원하는 (인자가 상위형) 함수여야 사용할 수 있으며 하위형 인자를 가지는 함수는 원래형 보다 더 많은 정보가 필요할 수 있으므로 줄 수 없다. 반대로 함수의 반환값 R은 원래 필요한 것보다 더 많은 정보를 얻을 수 있는 하위형을 주는 것은 관계없으나 상위형은 곤란하다.

결론적으로 함수형에서는 인자와 반환값 두가지를 고려해야 하며 *원래보다 적은 정보를 제공하는 인자를 받거나 더 많은 정보를 제공하는 반환값을 제공하는 경우는 상관없으나* 반대의 경우는 함수 호출 전후로 문제가 생길 수 있다.

