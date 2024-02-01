---
layout: post
title: JavaCV를 사용해 웹캠 표시
category: programming
tags: scala javacv javafx
---

[JavaCv](https://github.com/bytedeco/javacv)는 컴퓨터 비전에 관련된 여러 라이브러리들의 java wrapper이다. Scala와 JavaFx를 사용해서 웹캠의 화면을 표시하는 프로그램을 작성해 보았다.

다음과 같이 적당한 디렉토리에 src/main/scala 디렉토리를 만들고 build.sbt 파일을 작성하자.

{% highlight text %}
cd 디렉토리이름
mkdir -p src/main/scala
touch build.sbt
{% endhighlight %}

build.sbt 내용은 다음과 같이 작성했다. 기종에 따라 다른 라이브러리를 받으므로 platform 변수에 맥에서는 `macosx-x86_64`를 사용했는데 윈도우즈라면 `windows-x86` 리눅스라면 `linux-x86_64`를 사용하면 된다고 한다.

{% highlight text %}
val javacppVersion = "0.11"

val platform = "macosx-x86_64"

classpathTypes += "maven-plugin"

libraryDependencies ++= Seq(
  "org.bytedeco" % "javacpp" % javacppVersion,
  "org.bytedeco" % "javacv" % javacppVersion,
  "org.bytedeco.javacpp-presets" % "opencv" % ("2.4.11-" + javacppVersion) classifier "",
  "org.bytedeco.javacpp-presets" % "opencv" % ("2.4.11-" + javacppVersion) classifier platform
)
{% endhighlight %}

src/main/scala 디렉토리에 다음과 같은 내용의 프로그램을 작성하고 sbt run을 실행하면 컴파일 후 프로그램이 실행되면서 웹캠 화면이 보일것이다.

{% highlight scala %}
import javafx.application.{Application, Platform}
import javafx.stage.Stage
import javafx.scene.{Scene, Group}
import javafx.scene.image.{Image, ImageView, WritableImage, PixelFormat}
import concurrent.Future
import org.bytedeco.javacv.{FrameGrabber, Frame, OpenCVFrameConverter, Java2DFrameConverter}
import scala.concurrent.ExecutionContext.Implicits.global
import java.awt.image.{BufferedImage, DataBufferByte}
import org.bytedeco.javacpp.opencv_core.{Mat, CV_8UC3, CV_8UC4}
import org.bytedeco.javacpp.opencv_imgproc.{cvtColor, CV_BGR2RGB, CV_BGRA2RGBA}
import javafx.embed.swing.SwingFXUtils
import util.{Try, Success, Failure}
import java.io.ByteArrayInputStream

object Test1 {
  def main(as:Array[String]) = Application.launch(classOf[Test1], as:_*)
}

class Test1 extends Application {
  val grabber = FrameGrabber.createDefault(0)
  val matConverter = new OpenCVFrameConverter.ToMat()
  val bImgConverter = new Java2DFrameConverter()
  val iv = new ImageView()
  var continueGrab = true

  override def start(ps:Stage) = {
    ps.setTitle("웹캠 시험")
    val group = new Group(iv)
    val scene = new Scene(group, 640, 480)
    ps.setScene(scene)
    ps.show

    startCam
  }

  def startGrabAndDo(job: Frame => Unit) = {
    grabber.start
    def infiniteGrab:Unit =
      Future({ grabber.grab }).map(f => job(f)).onComplete { case _ => if (continueGrab) infiniteGrab }
    infiniteGrab
  }

  def startCam = startGrabAndDo({ f:Frame => runLater( setImageView(f) )})

  def runLater(job: => Unit) =
    Platform.runLater(new Runnable() { def run = job })

  def setImageView(f:Frame) = {
    // iv.setImage(SwingFXUtils.toFXImage(bImgConverter.convert(f), null))
    iv.setImage(mat2Img(matConverter.convert(f)))
  }

  def mat2Img(mat:Mat) = {
    val img = new WritableImage(mat.cols, mat.rows)
    val writer = img.getPixelWriter

    if (mat.channels == 3) {
      cvtColor(mat, mat, CV_BGR2RGB)
      writer.setPixels(0, 0, mat.cols, mat.rows, PixelFormat.getByteRgbInstance,
        mat.getByteBuffer, mat.step.toInt /* (mat.cols + 1) * mat.channels */ )
    } else {
      writer.setPixels(0, 0, mat.cols, mat.rows, PixelFormat.getByteBgraInstance,
        mat.getByteBuffer, mat.step.toInt /* (mat.cols + 1) * mat.channels */ )
    }
    img
  }
}
{% endhighlight %}

* grabber 변수에는 FrameGrabber 인스턴스가 저장되며 start, stop으로 작동시키고 grab으로 화면을 얻게 된다.
* 화면은 JavaCv의 Frame형 인스턴스로 반환되는데 이를 JavaFx용 Image로 바꾸는 방법은 2가지가 있다. Frame -> BufferedImage -> Image 순서로 변환하는 것과 Frame -> Mat -> Image 순서로 변환할 수 있는데 setImageView 함수를 참조하면 되겠다.
* 웹캠에서 계속해서 Frame 변수를 얻어서 이를 화면에 표시하는 것은 scala.concurrent.Future를 사용했는데 startGrabAndDo 함수를 참조하면 되겠다. 화면을 grab해서 변수로 받은 job을 실행시키고 그 일이 끝나면 다시 자기 자신을 호출하는 방식으로 구현했다. 뭔가 허술한 듯 하지만 일을 조각조각 나누어서 합치는 함수형 언어의 특성이 비교적 잘 나와있다고 생각한다.
* runLater 함수는 JavaFx 컨트롤의 변경은 JavaFx 스레드에서 실행되어야 하는데 이를 위해 사용했다.

