---
layout: post
title: Scala Future와 JavaFx 합치기
category: programming
tags: scala
---

JavaFX를 이리저리 만져보다 scala의 Future와 합치면 콘솔에서 글을 입력받고 JavaFX에서 표시할 수 있겠다는 생각이 들었다. StackOverflow를 좀 검색해 본 다음 이리저리 실험해 보니 가능하다. Thread를 따로 설정해야 하는데 Console input 부분은 scala에서 제공하는 기본 ExecutionContex를 사용하고 JavaFX의 화면에 관계된 부분을 업데이트 하는 부분은 JavaFX의 thread에서 실행되도록 하는게 중요한듯 하다.

![콘솔 입력을 JavaFX 화면에 출력하기]({{ site.url}}/assets/ConsoleInputJavaFx.png)

* `pipeLoop` 함수에서 2개의 ExecutionContex를 받아서 두개의 code가 서로 다른 context에서 실행될 수 있도록 하는 것이 중요한 포인트.
* texts에 입력받은 문자열을 추가하는 부분에서 addListener의 코드를 사용해도 되지만 ObservableList와 ListChangeListener가 어떻게 동작하는지 알아볼 겸 ArrayList에 문자열만 추가하고 ListChangeListner에서 추가된 문자열을 다시 Text Node로 만들어 VBox에 붙이도록 했다.
* 원래는 ArrayBuffer와 같은 scala의 집합에 문자열을 추가하면 거기에 맞추어 java의 Text Node List가 업데이트되고 이것을 VBox의 children property에 bind 하면 자동으로 화면 업데이트까지 될 것을 기대했는데 지난번 property에서 언급한 것과 같이 VBox의 children은 property가 아니라 bind 함수를 사용할 수 없고 따라서 ListChangeListener에서 맞춰서 업데이트 해야 한다.

{% highlight scala %}
import javafx.application.Application
import javafx.scene.image.{Image, ImageView}
import javafx.stage.Stage
import javafx.scene.{Scene, Group}
import javafx.scene.layout.{VBox, Pane}
import javafx.scene.control.ScrollPane
import javafx.scene.paint.Color
import javafx.scene.text.Text
import javafx.beans.property.ReadOnlyListWrapper
import javafx.beans.value.ChangeListener
import javafx.collections.{FXCollections, ListChangeListener}
import javafx.application.Platform
import javafx.geometry.Insets

import java.util.ArrayList
import java.util.concurrent.Executor
import concurrent.{Future, ExecutionContext}
import scala.util.{Success, Failure}
import scala.collection.JavaConversions._
import scala.concurrent.ExecutionContext.Implicits.global

object PTest {
  def main(args:Array[String]) = Application.launch(classOf[PTest], args:_*)

  implicit val javaFxExecutionContext: ExecutionContext =
    ExecutionContext.fromExecutor(new Executor {
      def execute(command: Runnable): Unit = Platform.runLater(command)
    })
}

class PTest extends Application {
  val texts = FXCollections.observableList(new ArrayList[String]())
  val textROList = new ReadOnlyListWrapper[String](texts)
  val fxec = PTest.javaFxExecutionContext

  def pipeLoop[T, U](code: => T, handler: T => U)(implicit xc1:ExecutionContext, xc2:ExecutionContext):Unit = {
    val f = Future(code)(xc1)
    f.onComplete({
      case Success(v) =>
        handler(v)
        pipeLoop(code, handler)(xc1, xc2)
      case Failure(e) => throw new Exception(e)
    })(xc2)
  }

  override def start(ps:Stage) = {
    ps.setTitle("Propagation Test")
    val root = new ScrollPane()
    root.setFitToWidth(true)
    root.setFitToHeight(true)
    val scene = new Scene(root, 600, 400, Color.BLACK)
    val vbox = new VBox()
    vbox.setPadding(new Insets(10))
    vbox.setStyle("-fx-background-color: black;")
    root.setContent(vbox)

    ps.setScene(scene)
    ps.show

    textROList.addListener(new ListChangeListener[String]() {
      override def onChanged(c:ListChangeListener.Change[_ <: String]) = {
        while (c.next) {
          if (c.wasPermutated) println("permutated!")
          if (c.wasAdded) println("added!")
          if (c.wasRemoved) println("removed!")
          if (c.wasReplaced) println("replaced!")
          if (c.wasUpdated) println("updated!")
          for (s <- c.getAddedSubList) {
            println(s"added item : $s")
            s.split("\\s").filter(!_.isEmpty).foreach(w => {
              val t = new Text(w)
              t.setStyle("-fx-font-family: \"Nanum Brush Script OTF\"; -fx-font-size: 40; -fx-fill:yellow;")
              vbox.getChildren.add(t)
              root.setVvalue(1)
          })}
        }}
    })


    // 콘솔 입력은 Fx와 다른 thread에서 일어나고 입력에 대한 handler는 Fx thread에서 일어나야
    pipeLoop({
      print("글자를 입력하세요 : ")
      val l = Console.readLine
      l
    }, { (l:String) =>
      texts.add(l)
    })(concurrent.ExecutionContext.Implicits.global, fxec)
  }
}
{% endhighlight %}
