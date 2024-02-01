---
layout: post
title: Scala Future, Future[Try]
category: programming
tags: scala
---

Typesafe에서 만든 scala의 database관련 라이브러리 slick을 사용해서 프로그램을 작성해 보고 있습니다. Query 결과가 기본적으로 Scala의 Future를 사용해서 반환되기 때문에 Future에 대해서도 이리저리 다뤄보고 있는데 그 중 한가지 예를 간단히 요약해 보겠습니다.

제가 해야하는 작업은 join된 테이블의 특정 컬럼의 가장 큰 값들을 얻어 그 중에서도 가장 큰 값을 얻는 것인데 slick에서는 join된 열이 하나도 없으면 Failure를 반환하게 됩니다. 물론 테이블 마다 해당하는 열의 숫자를 count 함수로 미리 조사해서 1 이상인 경우에만 쿼리를 실행해도 되겠지만 그러려면 부하가 더 많이 걸리니까 그냥 다뤄보도록 하겠습니다.

다음은 비슷한 결과를 반환하도록 만든 함수입니다. 1초 미만 임의의 시간 동안 멈춘 다음 30%의 확률로 Failure를, 나머지는 임의의 정수를 반환하는 Future를 제공하는 함수입니다.

{% highlight scala %}
def randomFutureWithFailures = {
	val p = Promise[Int]
	Future {
		Thread.sleep(Random.nextInt(1000))
		if (Random.nextDouble < 0.3)
			p.failure(new Throwable("something wrong happened"))
		else
			p.success(Random.nextInt(10))
	}
	p.future
}
{% endhighlight %}

`(1 to 10).map(_ => randomFutureWithFailures)`와 같은 코드로 10개의 Future를 포함하는 Sequence를 얻을 수 있습니다. Future의 companion object는 여러 future의 Sequence를 다룰 수 있는 reduce, fold, sequence등의 함수를 제공하지만 Failure가 포함되는 경우 전체의 결과가 Failure가 되므로 원하는 결과를 얻을 수 없습니다. 이때 성공은 Success로, 실패는 Failure로 싸주는 다음과 같은 함수를 사용하면 반환값에서 볼 수 있듯이 값들의 future에서 값을 가지는 Try의 Future가 됩니다.

{% highlight scala %}
def future2FutureTry[T](f:Future[T]):Future[Try[T]] =
	f.map(Success(_)).recover({ case f => Failure(f) })
{% endhighlight %}

이제는 앞서 언급한 Future.seqence 함수로 Future들의 Sequence에서 Sequence의 Future로 바꾸어준 다음 Future가 끝날 때 Try들의 Sequence를 결과로 처리하면 되겠습니다.

각 Future가 걸린 시간과 결과를 보여주는 전체 소스는 [github](https://gist.github.com/nineclue/8ae82583da87e3be6049)에서 보실 수 있겠습니다.

참고 : [StackOverflow](https://stackoverflow.com/questions/20874186/scala-listfuture-to-futurelist-disregarding-failed-futures)