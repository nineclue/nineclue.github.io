---
layout: post
title: git eclipse sbt 사용 설정
category: programming
tags: scala
---

자주 쓰이므로 [스택 오버플로우](http://stackoverflow.com/questions/15381294/how-to-initialize-a-new-scala-project-in-sbt-eclipse-and-github)에서 발췌

1. github이나 bitbucket에 repository 생성
2. sbt에 [sbt eclipse plugin 설치](https://github.com/typesafehub/sbteclipse/)
3. eclipse에서 Git Repositories View 사용하거나 커맨드 라인에서 git 사용해서 repository를 가져옴
4. sbt eclipse 명령으로 프로젝트 생성
5. eclipse에서 File > Import > General > Existing Projects 선택해서 프로젝트 읽어들이기
6. .gitignore를 적당히 수정


{% highlight text %}
*.class
*.log

# sbt specific
dist/*
target/
lib_managed/
src_managed/
project/boot/
project/plugins/project/

# Scala-IDE specific
.scala_dependencies

# Eclipse specific
.project
.classpath
.cache
{% endhighlight %}