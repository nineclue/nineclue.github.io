---
layout: post
title: HTML에 SVG 겹쳐 표시하기
date: 2024-11-08 08:41
category: programming
tags: scala
---

개인적인 프로젝트를 하다 HTML image 위에 점을 표시할 일이 생겼습니다. 작성중이던 코드에 덧붙여 시험해 보려니 머리도 복잡해지고 포스팅 올린지도 좀 된것 같아 간단히 새로운 프로젝트를 만들어 시험해 보았습니다. Scala.js를 사용해서 javascript로 컴파일하고 index.html에서 불러 실행하는 방식입니다. 

관리는 sbt를 사용하겠습니다. 적당히 이름을 붙여 폴더를 생성하고 들어가 project 폴더를 만들고 거기에 다음과 같은 내용의 plugins.sbt 파일을 만들어 줍니다.

```
mkdir html_with_svg
cd html_with_svg
mkdir project
touch project/plugins.sbt
touch build.sbt
mkdir -p src/main/scala
touch src/main/scala/JSMain.scala
touch index.html
```

#### plugins.sbt
```scala
addSbtPlugin("org.scala-js" % "sbt-scalajs" % "1.17.0")
```

#### build.sbt
```scala
ThisBuild / scalaVersion := "3.5.2"

enablePlugins(ScalaJSPlugin)    // scala.js plugin 사용 => 컴파일 결과 javascript 파일 생성
libraryDependencies ++= Seq(
    "com.lihaoyi" %%% "scalatags" % "0.13.1"    // scalatags 라이브러리 사용
)
scalaJSUseMainModuleInitializer := true     // main 함수에서 시작하는 방식 사용
```

#### JSMain.scala
```scala
object JSMain:
    def main(as: Array[String]): Unit =
        println("안녕? 여러분!")
```

#### index.html
```html
<html>
    <head> </head>
    <body>
        <script
            type="text/javascript"
            src="./target/scala-3.5.2/html_with_svg-fastopt/main.js"
        ></script>
    </body>
</html>
```

`sbt fastLinkJS` 명령을 내리면 필요한 파일을 다운 받고 컴파일해서 `target/scala-3.5.2/html_with_svg-fastopt` 폴더에 main.js 파일이 만들어집니다. 브라우저에서 index.html 파일을 열고 개발자 창의 console에 메시지가 출력되는 것을 확인해 봅시다.

> #####  INFO
> 
> 특정한 에러메시지가 표시되지 않는데 제대로 동작하지 않는 경우 브라우저 옵션에서 cache를 disable 시켜봅시다. javascript 파일을 한번 읽고 계속 사용하는 경우가 있습니다. 
{: .block- info}

제대로 동작한다면 이제 브라우저 내에 html element 들을 표시해 봅시다. 

#### JSMain.scala
```scala
import org.scalajs.dom
import scalatags.JsDom.all.*

object JSMain:
    def words =
        div(
            Seq("안녕?", "여러분", "만나서", "반가워요").map(div(_))
        )
    def main(as: Array[String]): Unit =
        // println("안녕? 여러분!")    
        dom.document.body.appendChild(words.render)
```

> #####  INFO
> 
> 소스 코드를 수정할 때마다 `sbt fastLink` 명령을 실행해 줘야 새로운 javascript 파일이 컴파일되며 이것이 귀찮은 분들을 위해 `sbt` 실행후 프롬프트에서 `~fastLinkJS` 명령을 실행하면 소스코드가 변결될 때마다 새로 컴파일합니다. 
{: .block- info}

브라우저에 여러 div element들이 표시되는 것을 확인하였으면 이제 image와 svg element를 겹쳐서 표시해 봅시다. image는 scala.js 로고 svg 파일을 800 X 800 픽셀 크기의 png 파일로 변환한 것이며 둘의 **position**을 **absolute**로 지정하였고 이들의 containing block이 body가 되지 않기 위해 상부에 "container" 이름의 div로 감싸고 이의 **position**을 **relative**로 하였습니다. 그리고 div의 크기 역시 image, svg와 동일하게 지정하였습니다. Element들의 positioning에 대해서는 [문서]:(https://developer.mozilla.org/en-US/docs/Web/CSS/position)를 참조하시면 되겠습니다.

#### JSMain.scala
```scala
val eSize = modifier(
        width := "800px",
        height := "800px"
    )

def logo =
    img(
        position := "absolute",
        top := "0",
        left := "0",
        eSize,
        src := "./scala-js-logo.png"
    )

def svgs =
    import scalatags.JsDom.svgTags.*
    import scalatags.JsDom.svgAttrs.*
    svg(
        id := "svg",
        position := "absolute",
        top := "0",
        left := "0",
        eSize,
        circle(cx := "100", cy := "100", r := "50", fill := "orange")
    )

def combo =
    div(
        id := "container",
        position := "relative",
        eSize,
        border := "1px solid black",
        logo,
        svgs
)

def main(as: Array[String]): Unit =
    // println("안녕? 여러분!")
    // dom.document.body.appendChild(words.render)
    dom.document.body.appendChild(combo.render)
```

브라우저에서 로고 위에 오렌지색 원이 표시되는 것을 확인하면 성공입니다. 마지막으로 마우스로 클릭하였을 때 svg에 circle을 추가하여 점을 찍어보도록 합시다.

#### JSMain.scala
```scala
def main(as: Array[String]): Unit =
    // println("안녕? 여러분!")
    // dom.document.body.appendChild(words.render)
    dom.document.body.appendChild(combo.render)
    val con = dom.document.getElementById("container")
    val svg = dom.document.getElementById("svg")
    con.addEventListener("mousedown", e => mouseHandler(con, svg)(e))

def mouseHandler(con: Element, s: Element)(e: MouseEvent) =
    import scalatags.JsDom.svgTags.*
    import scalatags.JsDom.svgAttrs.*
    val rect = con.getBoundingClientRect()
    // println(s"Mouse Clicked! ${e.clientX - rect.left}, ${e.clientY - rect.top}")
    s.appendChild(
        circle(
            cx := e.clientX - rect.left,
            cy := e.clientY - rect.top,
            r := "10",
            fill := "tomato"
        ).render
    )
```

전체 소스 코드는 [github](https://github.com/nineclue/html_with_svg)에서 받을 수 있습니다.
[실행화면](./assets/html_with_svg.png)