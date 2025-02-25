---
layout: post
title: Let's make a App with Scala.js and Tauri part 1
date: 2025-02-21 09:38
category: programming
tags: scala tauri
---
[Tauri](https://tauri.app) is a framework for building tiny, fast binaries for all major desktop and mobile platforms. An Javascript app will run as if it was running within browser. Let's make a simple application using [scala.js](https://scala-js.org).

### Requirements
Build tools for each platform, rust and node.js must be installed first. See [tauri's document](https://tauri.app/start/prerequisites/).

### Creation of an application
Tauri supports ways of creating app's skeleton. We'll use [`pnpm`](https://pnpm.io). Install it before running following commands. When it runs, it will ask questions to answer. Use your own name and identifier for first two questions.
```
>> pnpm create tauri-app
✔ Project name · <tauri-app>
✔ Identifier · <your unique identifier>
✔ Choose which language to use for your frontend · TypeScript / JavaScript - (pnpm, yarn, npm, deno, bun)
✔ Choose your package manager · pnpm
✔ Choose your UI template · Vanilla
✔ Choose your UI flavor · JavaScript
```
CD into the project folder and install packages. Then make an application and run it. A window with icons and simple form will be shown.
```
>> cd tauri-app
>> pnpm instal
>> pnpm tauri dev
```
![default tauri app](tauri.png)

### Structure of the project folder
Simplified structure of the project folder is as follows
```
+- src : where our javascript app will be located
|  - index.html : skeleton of app
|  - main.js : javascript referenced in index.html
+- src-tauri : tauri's code in rust
|  - Cargo.toml : build configuration for rust
|  - tauri.conf.json : our tauri app's meta information and configuration
+- node_modules : handled by pnpm
- package.json : package file for node.js
```

### Let Scala render the screen
Now it's time for scala.js. I will use [mill](https://mill-build.org) as a build tool and [scalatags](https://github.com/com-lihaoyi/scalatags) for HTML. ~~Welcome to [Li Haoyi](https://github.com/lihaoyi) world.~~

##### install mill
As [mill's installation page](https://mill-build.org/mill/cli/installation-ide.html), the standard method of installing mill is to install mill script at project folder. I've had problem running mill in Windows 11 PC. Following should work in macOS and linux.
```
>> curl -L https://repo1.maven.org/maven2/com/lihaoyi/mill-dist/0.13.0-M0/mill -o mill
>> chmod +x mill
>> echo 0.12.8 > .mill-version
```
Mill's build file is `build.sc` and mill requires separate source folder. 
```
>> touch build.sc
>> mkdir -p front/src
>> touch front/src/Front.scala
```
##### build.sc
`front` in `build.sc` matches the folder name. All source files, resources and test files reside within that folder. `fscala` task compiles scala code and copies the resulting javascript codes to `src` folder. `mill`'s build file is using scala 2.13 syntax, so square brackets are needed.
``` scala
import mill._, scalalib._, scalajslib._

object front extends ScalaJSModule {
    def scalaVersion = "3.6.3"
    def scalaJSVersion = "1.18.2"
    def ivyDeps = Agg(ivy"com.lihaoyi::scalatags::0.13.1")
	def fscala = Task {
		val jsPath = front.fastLinkJS().dest.path
		val targetPath = jsPath / os.up / os.up / os.up / "src"
		val target = targetPath / "main.js"
		os.copy.over(jsPath / "main.js", target)
		os.copy.over(jsPath / "main.js.map", targetPath / "main.js.map")
		PathRef(target)
    }
}
```
##### Front.scala
Same as original index.html structure. Just using scalatags to render into document's boby.
``` scala
import org.scalajs.dom
import scalatags.JsDom.all.*

object Front:
    def main(args: Array[String]): Unit = 
        // render tags into body
        dom.document.body.appendChild(content.render)

    val content = 
        tag("main")(cls := "container",
            h1(s"Welcome to Tauri with Scala.js"),
            div(cls := "row",
                a(href := "https://tauri.app",
                    target := "_blank",
                    img(src := "/assets/tauri.svg", cls := "logo tauri", alt := "Tauri logo")),
                a(href := "https://developer.mozilla.org/en-US/docs/Web/JavaScript",
                    target := "_blank",
                    img(src := "/assets/javascript.svg", cls := "logo vanilla", alt := "JavaScript logo"))
            ),

            p("Click on the Tauri logo to learn more about the framework"),

            form(cls := "row", id := "greet-form",
                input(id := "greet-input", placeholder := "Enter a name..."),
                button(`type` := "submit", "Greet")),
            p(id := "greet-msg")
        )
```
##### index.html
``` html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="stylesheet" href="styles.css" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Tauri App</title>
    <script type="module" src="/main.js" defer></script>
  </head>
  <body>
  </body>
</html>
```
Now let's see the result. First line compiles scala code to javascript and copy the result code to `src` folder.
``` bash
>> ./mill front.fscala
>> pnpm tauri dev
```
Now app's appearance is same as initial code, but screen is rendered from scala code. 
Greet button does not work as previously. This functionality requires calling Tauri's API, which will be handled at part 2.
