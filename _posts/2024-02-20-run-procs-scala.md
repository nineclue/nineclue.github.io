---
layout: post
title: 시작시 프로그램 실행시키기
date: 2024-02-20 16:45
category: programming
tags: [scala]
---

백업이나 스케쥴 보여주는 프로그램들을 몇개 짜다보니 컴퓨터 부팅할 때마다 실행시키려고 몇개의 폴더를 왔다갔다 하면서 일일이 실행시키는 게 부담이 되었습니다.

간단히 쉘 스크립트를 짜서 실행시키면 되겠지만 [os-lib](https://github.com/com-lihaoyi/os-lib)나 [scala-cli](https://scala-cli.virtuslab.org) 한번 써 볼까 싶어 간단히 프로그램을 짜 보았습니다. 

### os.proc
os.proc으로 실행시킬 프로그램, 인자, 실행 위치 등을 지정할 수 있습니다. 실행을 마치는 프로그램이면 call을 호출하고 아니면 spawn을 사용하면 되겠습니다.

### Resource
프로그램 종료시 spawn으로 실행시킨 프로세스들을 어떻게 종료시켜야 하나 고민하다 cats의 Resource를 사용하기로 했습니다. `Resource.make`에서 리소스를 생성하는 함수와 종료시 필요한 함수를 지정하면 프로그램 종료시 자동으로 리소스를 해제해 줍니다.

### scala-cli
scala-cli를 사용하면 build.sbt를 사용하지 않고도 `>// using dep` 형식으로 사용할 라이브러리들을 지정할 수 있고 **package** 명령을 사용하면 직접 컴파일해서 실행 파일까지 만들어 줍니다. 현재로는 graal-native image는 잘 동작하나 더 작은 크기의 scala-native를 사용하는 native 명령은 컴파일 에러가 발생합니다. 아직 scala-native는 좀 더 기다려야 하는 걸까요. 

`>// using dep` 명령 사용시 native 형식을 사용하기 위해서는 `//> using dep com.lihaoyi::os-lib::0.9.3`와 같이 두번째 버전 앞에 콜론을 2개 사용해야 합니다. `os-lib:0.9.3`과 같이 colon을 하나만 사용하면 JVM에서는 잘 동작하지만 native 형식에서는 컴파일이 되지 않습니다.

```scala
//> using scala 3.3.1
//> using dep org.typelevel::cats-effect::3.5.3
//> using dep com.lihaoyi::os-lib::0.9.3

import cats.effect.*
import cats.syntax.all.*

object ResourceTest extends IOApp:
    def r(label: String, cmd: String, wd: Option[String] = None)(args: String*) =
        Resource.make(
            IO.pure(os.proc(cmd, args)
                .spawn(cwd = wd.map(os.Path(_)).getOrElse(os.pwd),
                    stdout = os.Inherit, stderr = os.Inherit
                ))
        )((p: os.SubProcess) =>
            if p.isAlive() then
                IO.println(s"Destroying $label") *>
                IO(p.destroy())
            else
                IO.println(s"$label already finished.")
        )

    case class Command(label: String, cmd: String, wd: String, args: List[String])
    val commands = Seq(
            Command("mcal", "mix", "mcal", List("phx.server")),
            Command("cpraddb", "java", "cpraddb", List("-Dconfig.file=application.conf", "-jar", "target/scala-3.3.1/craddb.jar")),
            Command("radreport", "java", "radreportserver2", List("-Dconfig.file=rrs.conf", "-jar", "rrs-server.jar"))
        ).map(c => c.copy(wd = "/Users/nineclue/lab/" ++ c.wd))

    def run(as: List[String]): IO[ExitCode] =
        commands.map(c => r(c.label, c.cmd, Some(c.wd))(c.args:_*))
            .sequence
            .use(_ => IO.never)
        *> IO(ExitCode.Success)
```