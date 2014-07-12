---
layout: post
title: 임의 수식 생성하기
category: programming
tags: scala
---

어찌하다보니 벼락치기로 수십개의 수식 문제를 만들 필요가 생겼다. 손으로 일일이 써도 되겠지만 괜히 프로그램을 짜서 한번 만들어보자는 욕심이 들어서... 

처음에 ruby로 짜다가 ruby의 module과 class의 구분을 까먹어서 scala로 변경해서 작성했다. 최근에 공부한 언어라 그런지 type에 익숙해져서 그런지 scala가 더 편하게 느껴졌다. 

`Maker` trait이 기본이다 내부에 난수 생성하는 상수 r을 두고 상속하는 부분에서 제공할 `Map[Double, ()=>T]` 상수 fs를 받으면 `make` 함수에서 난수에 따라 적절한 함수를 고른 다음 (`Option`과 `foldLeft`를 사용했는데 개인적으로 구현방법이 마음에 든다) 함수 호출해서 얻은 값을 반환한다.

`Numbers` 클래스에서는 70%의 확률로 2자리수, 나머지 30%의 확률로 3자리수를 얻으며 (정확히는 3자리수 미만이라 1, 2자리수가 나올수도 있다) `Operators` 클래스에서는 50%의 확률로 +, 30%의 확률로 -, 20%의 확률로 *를 만든다. `Exp` 클래스는 20%의 확률로 2개의 숫자를 사용한 수식을, 80%의 확률로 3개의 숫자를 사용한 수식을 만든다.

소스 코드는 다음과 같다.

{% highlight scala %} 
import java.util.Date

trait Maker[T] {
	val r = util.Random
	val fs:Map[Double, () => T]
	def make():T = {
		val dice = r.nextDouble
		val func:Option[()=>T] = 
			fs.foldLeft(None:Option[()=>T]) { 
				case (prev, (d, f)) => { 
					if (prev.isEmpty) 
						if (dice < d) Some(f)
						else None
					else
						prev
				} 
		} 
		val finalFunc = func.get
		finalFunc()
	}
}

class Numbers extends Maker[Int] {
	val fs:Map[Double, () => Int] = Map((0.7, number(100)), (1.0, number(1000)))
	def number(n:Int) = () => r.nextInt(n)
}

class Operators extends Maker[String] {
	val multi = () => "X"
	val add = () => "+"
	val minus = () => "-"
	val fs = Map((0.5, add), (0.8, minus), (1.0, multi))
}

class Exp extends Maker[String] {
	val n = new Numbers()
	val o = new Operators()
	val fs = Map((0.2, generate(2)), (1.0, generate(3)))
	def generate(num:Int):() => String = { () =>
		val nums = (1 to num).map(_ => n.make().toString)
		val ops = (1 until num).map(_ => o.make())
		nums.zip(ops).map( { case (n, o) => s"$n $o" } ).mkString(" ") + " " + nums.last +  " = "
	}
}

object Exam extends App {
	val e = new Exp
	val es = (1 to 40).map { _ => e.make() }
	val now = new Date
	val f = tools.nsc.io.File("Exam%02d%02d%02d%02d.txt".format(now.getDay, now.getHours, now.getMinutes, now.getSeconds)).writeAll(es.mkString("\n"))
	es.foreach(e => println(e))
}

val args = Array.empty[String]
Exam.main(args)
{% endhighlight %}

수식을 작성하고 파일을 만들고 나서야 답을 계산하지 않았다는 사실을 깨달았다. 곱하기로 'X'를 사용해서 표시하고 있는데다 곱하기를 덧셈, 뺌셈보다 우선해야 하니 이럴때는 수식을 그냥 읽어서 ~무적의~ `eval` 함수를 사용할 수 있는 ruby의 힘을 빌리기로...

{% highlight ruby %}
Dir.glob("Exam*.txt") { |f| 
	puts("FILE : #{f}")
	es = open(f).readlines
	es.each_index { |i| 
		puts("#{i+1} : #{es[i].slice(0..-2)} #{eval(es[i].gsub("X", "*").slice(0..-4))}")
	}  
}
{% endhighlight %}

ruby로 작성한 소스는 황당하리만큼 간단하다. scala 프로그램에서 생성한 파일형식(`"Exam*.txt"`)을 모두 읽어들여 파일의 각 줄을 출력하고 `=` 부분을 제거하고 `X`를 `*`로 치환한 문자열을 `eval` 함수에 넣어주면 끝.
