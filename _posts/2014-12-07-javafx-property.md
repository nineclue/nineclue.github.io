---
layout: post
title: javafx의 property
category: programming
tags: javafx
---

Scala만 사용할 때는 필요없지만 javafx를 사용하려면 property에 대해 좀 알아야 편할듯하다. 다음은 구글링 후 나름대로 개념을 잡은 것이니 틀린 내용이 있을 가능성이 굉장히 높다고 미리 변명을 해 놓고 적어보겠다.

Property는 java beans에서 사용하기 시작한 개념인데 기본적으로는 *private으로 정의한 변수에 get/set으로 시작하는 camelcase의 메소드*를 말한다.

{% highlight scala %}
@BeanProperty
var status = ""
{% endhighlight %}

Scala에서는 위와 같이 `@BeanProperty` annotation을 사용하면 컴파일러가 `def setStatus(s:String) { this.status = s }; def getStatus:String = this.status`와 같이 간단히 구현해 준다.

Property가 재밌어지는 것은 `addPropertyChangeListener` 함수를 통해 값이 변화할 때 호출될 핸들러를 등록할 수 있다는 점에서다. 실제 구현하려면 [오라클의 문서](https://docs.oracle.com/javase/tutorial/javabeans/writing/properties.html)에서와 같이 클래스 내에 PropertyChangeSupport 인스턴스를 생성하고 변수 변화, 등록, 제거시 각각 `firePropertyChange`, `addPropertyChangeListener`, `removePropertyChangeListener`함수를 호출하면 핸들러 호출, 등록, 제거를 처리해준다.

변수마다 이렇게 일일히 구현하는 것 보다 Javafx에서는 기본형에 대한 Property를 제공해주니 [오라클의 문서](https://docs.oracle.com/javafx/2/binding/jfxpub-binding.htm)와 같이 사용자의 클래스에서는 변수에 이들 property의 인스턴스를 저장하고 get/set 함수와 property 자체의 접근자를 제공하면 편리하게 사용할 수 있다.

자신의 property와 다른 property를 서로 엮는 것을 bind라 하는데 javafx에서는 scene의 넓이와 node의 넓이를 bind해 놓으면 scene의 크기가 변할 때 node의 크기도 거기에 맞추어 함께 변한다거나 좀 더 복잡하게는 scene의 넓이에서 얼마를 뺀 위치에 node를 위치시킨다거나 하는 동작을 비교적 간단하게 코딩으로 지정할 수 있다.

Property에는 `addListener` 함수에 변화에 따라 호출할 Listener를 붙일 수 있는데 `InvalidationListener`의 인스턴스를 붙이면 값이 변할 때마다 호출되지만 경우에 따라 내부적으로 값을 계산하지 않기도 하며 `ChangeListener`의 인스턴스를 붙이면 이전값과 새로운 값을 인자로 받으므로 항상 값을 계산하게 되다.

{% highlight scala %}
scala> import javafx.beans.binding.NumberBinding
import javafx.beans.binding.NumberBinding

scala> import javafx.beans.property._
import javafx.beans.property._

scala> val n1 = new SimpleIntegerProperty(1)
n1: javafx.beans.property.SimpleIntegerProperty = IntegerProperty [value: 1]

scala> val n2 = new SimpleIntegerProperty(2)
n2: javafx.beans.property.SimpleIntegerProperty = IntegerProperty [value: 2]

scala> val sumN1N2 = n1.add(n2)
sumN1N2: javafx.beans.binding.NumberBinding = IntegerBinding [invalid]

scala> sumN1N2
res0: javafx.beans.binding.NumberBinding = IntegerBinding [invalid]

scala> sumN1N2.getValue
res1: Number = 3

scala> n1.set(100)

scala> sumN1N2.getValue
res3: Number = 102
{% endhighlight %}

위에서 sumN1N2 binding은 n1과 n2의 값이 변화에 따라 합을 구하게 되는데 내부적으로는 항상 값을 구하지 않고 getValue 함수로 값을 요구할 때 계산을 실행한다. `InvalidationListener`의 경우 값이 변할 때 실제 값을 구하지는 않지만 `ChangeListener`의 경우 변화한 값을 구해야 하므로 변화할 때마다 실제 값을 구하게 된다.

##### UPDATE #####

Property와 bind를 처음 접하고 든 해보려고 한 실험은 VBox의 children을 Node의 collection에 bind하고 node collection은 좀 더 단순한 스칼라 (혹은 자바) collection에 bind하는 것이었다. 부연하자면 **스칼라의 collection을 수정하면 거기에 맞춰 Node collection이 update되고 VBox까지 함께 update**되는 뭐 그런걸 생각했다는 말이다.

아무튼 binding을 사용하려면 property를 지원해야 하는데 VBox의 children는 property를 지원하지 않고 대부분에서 숫자값을 property로 지원하고 있다. Children을 자동으로 update하려면 ObservableValue에 ChangeListener를 붙여서 변화에 따라 children을 맞춰주는 code를 따로 작성하도록 해야 할것 같다.

Property와 bind에 대해서는 [다음](http://blog.netopyr.com/2011/05/13/javafx-properties/)에 기본적인 개념 설명이 잘 되어 있는 듯.
