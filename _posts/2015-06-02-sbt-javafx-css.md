---
layout: post
title: Scala sbt에서 JavaFx css 파일 지정
category: programming
tags: scala javafx
---

Scala로 프로그램을 작성하려면 eclipse/intelliJ 등의 IDE나 sbt를 사용하게 된다. 개인적으로는 sublime에 sbt 조합이나 eclipse 사이를 왔다갔다 하고 있는데 sublime으로 컨트롤이 들어간 프로그램을 작성하고 sbt에서 run 실행시 `WARNING: Resource "com/sun/javafx/scene/control/skin/modena/modena.css" not found.` 워닝과 함께 컨트롤이 제대로 표시되지 않는 경우가 생긴다.

Sbt에서 JavaFx의 jar를 포함시키지 않기 때문이라고 하는데 다음과 같은 행을 build.sbt에 추가하면 정상적으로 표시된다.

{% highlight text %}
unmanagedJars in Compile += Attributed.blank(file(System.getenv("JAVA_HOME")+"/jre/lib/ext/jfxrt.jar"))
{% endhighlight %}

혹시 JAVA_HOME이 제대로 설정되지 않았다면 OSX의 경우 다음을 `~/.bash_profile`에 추가하자. 최신 버전은 Java 8이므로 인자로 1.8을 주었는데 설치된 java에 맞추면 되겠다.

{% highlight text %}
export JAVA_HOME=`/usr/libexec/java_home -v 1.8`
{% endhighlight %}
