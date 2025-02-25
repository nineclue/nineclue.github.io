---
layout: post
title: Let's make a App with Scala.js and Tauri part 2
date: 2025-02-24 14:16
category: programming
tags: scala tauri
---

Previously we rendered app's screen using scala.js. Now it's time to call Tauri's API from scala.

### Facade
Javascript is untyped and scala is typed language. To call javascript code from scala.js, we need to [facade for the library](https://www.scala-js.org/doc/interoperability/facade-types.html). 

Hand coding facade is very time consuming and hard to update. But there is [scalablytyped](https://scalablytyped.org) which makes facade automatically. 

There's also [mill plugin for scalablytyped](https://github.com/lolgab/mill-scalablytyped) but I've encounterd problems which I couldn't solve yet. Let's use [command line tool](https://scalablytyped.org/docs/cli). Install `stc` command line tool. You may need install [`coursier`](https://get-coursier.io/) first.

First add `typescript` as developmental dependency and `@tauri-apps/api` as dependency. `package.json` will be changed. Then run stc to make a facade of `@tauri-apps/api`. Facade library file will be located under `~/.ivy2/local/org.scalablytyped/tauri-apps__api_sjs1_3/` and is avaliable for our project.
```
pnpm add typescript -D
pnpm add @tauri-apps/api
stc
```
Modify scala file to display tauri version number. Many of tauri's methods return Promise[T], and we can use it's result chaining `then` method. Please notify that content has been changed as function to get Tauri version as an argument.
##### Front.scala
``` scala
import typings.tauriAppsApi.appMod

object Front:
    def main(args: Array[String]): Unit =
        appMod.getTauriVersion().`then`: v =>
            dom.document.body.appendChild(content(v).render)

    def content(taurVersion: String) =
        tag("main")(cls := "container",
            h1(s"Welcome to Tauri ($taurVersion) with Scala.js"),
        ...
```
Finally, we should change `build.sc` to output our code to module type and add tauri facade as dependency.
##### build.sc
``` scala
import mill._, scalalib._, scalajslib._
import mill.scalajslib.api._

object front extends ScalaJSModule {
    ...
    def ivyDeps = Agg(
      ivy"com.lihaoyi::scalatags::0.13.1",
      // add tauri facade in dependencies
      ivy"org.scalablytyped::tauri-apps__api::2.2.0-631484",
    )

    def moduleKind = ModuleKind.ESModule
    ...
}
```
Now you can compile and link to make `main.js`, but nothing will appear in app's screen. If you examine javascript console you can find `TypeError: Module name, '@tauri-apps/api/app' does not resolve to a valid URL.` error message. That's because facade has been made and saved at local ivy repository but not bundled to main.js. We can bundle neccessary files using [mill-bundler](https://github.com/nafg/mill-bundler).
### bundler
##### build.sc
``` scala
import mill._, scalalib._, scalajslib._
import mill.scalajslib.api._
import $ivy.`io.github.nafg.millbundler::millbundler::0.2.0`
import io.github.nafg.millbundler._
import io.github.nafg.millbundler.jsdeps._

// base class has been changed
object front extends ScalaJSRollupModule {
    ...
    def moduleKind = ModuleKind.ESModule
    // add dependet modules 
    def jsDeps =
      super.jsDeps() ++
        JsDeps(
          dependencies = Map(
            "@tauri-apps/api" -> "^2.2.0"
          )
        )

    // command to make bundle also changed 
    def fscala = Task {
      val bundles = front.devBundle()
      val jsPath = bundles.head.path / os.up
      val targetPath = jsPath / os.up / os.up / os.up / "src"
      val target = targetPath / "main.js"
      os.copy.over(jsPath / "out-bundle.js", target)
      val sourceMap = jsPath / "out-bundle.js.map"
      if (os.exists(sourceMap)) {
        os.copy.over(sourceMap, targetPath / "main.js.map")
      }
      PathRef(target)
    }

    // also added command to make bundle for production
    def scala = Task {
      val bundles = front.prodBundle()
      val jsPath = bundles.head.path / os.up
      val targetPath = jsPath / os.up / os.up / os.up / "src"
      val target = targetPath / "main.js"
      os.copy.over(jsPath / "out-bundle.js", target)
      val sourceMap = jsPath / "out-bundle.js.map"
      if (os.exists(sourceMap)) {
        os.copy.over(sourceMap, targetPath / "main.js.map")
      }
      PathRef(target)
    }
}
```
.tip
> ##### TIP
> 
> pnpm manages dependencies for javascript and compiles rust for tauri 
> mill compiles / links scala code
> stc (scalablytyped) makes javascript facade for scala using typescript definition
> mill-bundler bundles facade files with our compiled code (fastLinkJS / fullLinkJS)
{: .block-tip}
Now you can make a bundled file with `./mill front.fscala` and run with `pnpm tauri dev`. Screen with message will include tauri version.
