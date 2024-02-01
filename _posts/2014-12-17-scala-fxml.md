---
layout: post
title: Scala에서 FXML 읽어 사용하기
category: programming
tags: scala javafx
---

JavaFx 8과 함께 SceneBuilder 2.0이 발표되었습니다. 프로그램 인터페이스를 구상하는데 요긴하게 사용할 수 있겠는데 scala에서는 java와 거의 동일한 방법으로 사용할 수 있습니다.

화면에 배치한 컨트롤들과 scala 소스내의 변수를 연결해 주는 과정이 필요한데 다음과 같은 방법을 사용하면 됩니다.

1. 컨트롤과 연결하고자 하는 var형 변수를 정의하고 앞에 `@FXML` 디렉티브를 붙여준다
2. FXMLLoader의 load 함수를 호출하기 전에 setController 함수로 콘트롤러 클래스를 지정해준다. 이러면 `fx:id`로 지정한 컨트롤과 이벤트 핸들러가 연결된다.

데모용으로 아래의 스크린 샷과 같은 프로그램을 작성해 보았는데 아주 잘 동작합니다. 단, text 컨트롤이 BorderPane의 center 경계를 벗어나는데 이를 해결하기가 어렵군요. 기본적으로 BorderPane은 clip을 하지 않는듯 합니다.

![Scala & FXML demo]({{ site.url }}/assets/ScalaFXMLDemo.png)

소스코드와 FXML은 다음과 같습니다. Property 끼리의 연결이 아주 많은 일을 해 주는군요.

{% highlight scala %}
import javafx.application.Application
import javafx.scene.image.{Image, ImageView}
import javafx.stage.Stage
import javafx.scene.{Scene, Group, Node}
import javafx.scene.paint.Color
import javafx.scene.layout.{BorderPane, StackPane}
import javafx.scene.control.{TextField, Slider}
import javafx.scene.input.KeyEvent
import javafx.scene.text.Text
import javafx.scene.shape.Rectangle
import javafx.fxml.{FXML, FXMLLoader}

import java.net.URL

object FxmlTest {
  def main(args:Array[String]) = Application.launch(classOf[FxmlTest], args:_*)
}

class FxmlTest extends Application {
  @FXML
  var textInput:TextField = null
  @FXML
  var textOutput:Text = null
  @FXML
  var sizeSlider:Slider = null
  @FXML
  var rotationSlider:Slider = null

  override def start(ps:Stage) = {
    val loader = new FXMLLoader(new URL("file:fxmltest.fxml"))
    loader.setController(this)
    val root = loader.load.asInstanceOf[BorderPane]
    val scene = new Scene(root)

    textOutput.textProperty.bind(textInput.textProperty)
    textOutput.scaleXProperty.bind(sizeSlider.valueProperty)
    textOutput.scaleYProperty.bind(sizeSlider.valueProperty)
    textOutput.rotateProperty.bind(rotationSlider.valueProperty)

    ps.setScene(scene)
    ps.show
  }

  def textInputKeyTyped(e:KeyEvent) = println("key typed : " + textInput.getText)
}
{% endhighlight %}

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.scene.text.*?>
<?import javafx.geometry.*?>
<?import javafx.scene.control.*?>
<?import java.lang.*?>
<?import javafx.scene.layout.*?>


<BorderPane maxHeight="-Infinity" maxWidth="-Infinity" minHeight="-Infinity" minWidth="-Infinity" prefHeight="400.0" prefWidth="600.0" xmlns="http://javafx.com/javafx/8" xmlns:fx="http://javafx.com/fxml/1">
   <left>
      <VBox spacing="5.0" BorderPane.alignment="CENTER">
         <children>
            <Label text="글자">
               <font>
                  <Font name="NanumMyeongjoOTF" size="13.0" />
               </font>
            </Label>
            <TextField fx:id="textInput" onKeyTyped="#textInputKeyTyped">
               <font>
                  <Font name="NanumMyeongjoOTF" size="13.0" />
               </font>
            </TextField>
            <Label text="크기">
               <font>
                  <Font name="NanumMyeongjoOTF" size="13.0" />
               </font>
            </Label>
            <Slider fx:id="sizeSlider" blockIncrement="5.0" max="20.0" min="1.0" value="1.0" />
            <Label text="회전">
               <font>
                  <Font name="NanumMyeongjoOTF" size="13.0" />
               </font>
            </Label>
            <Slider fx:id="rotationSlider" max="360.0" />
         </children>
         <padding>
            <Insets bottom="10.0" left="10.0" right="10.0" top="10.0" />
         </padding>
      </VBox>
   </left>
   <center>
      <Text fx:id="textOutput" strokeType="OUTSIDE" strokeWidth="0.0" text="Text" BorderPane.alignment="CENTER">
         <font>
            <Font name="NanumGothic" size="14.0" />
         </font>
      </Text>
   </center>
</BorderPane>
{% endhighlight %}