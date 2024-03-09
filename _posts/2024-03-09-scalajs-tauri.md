---
layout: post
title: Tauri에서 Scala.js 사용하기
date: 2024-03-09 10:07
category: programming
tags: scala, rust
---

예전에 medium에 scala.js와 scalajs-react를 tauri에서 사용하는 방법에 대해 포스팅한 적이 있었습니다. ~~포스팅하고 위치를 까먹어서(?!) 한참 찾다가 포기했었는데 구글에서 검색하다보니 medium에 있었다는~~

[scalatags](https://com-lihaoyi.github.io/scalatags/) 문서를 읽다가 Tauri와 함께 사용해 보면 재밌을것 같아서 plain javascript 예제를 scala.js와 scalatags를 사용해서 포팅해 보았습니다. [sjs-tauri-demo](https://github.com/nineclue/sjs-tauri-demo)

scalatags는 서버에서 html을 렌더링할때 사용할수도 있지만 scala.js에서는 Node를 반환하는 `render` 메소드를 추가로 사용할 수 있어 바로 DOM tree를 수정할 수 있습니다. 기본 예제에서 index.html에 있던 html 태그들을 모두 scala내로 옮겨왔고 마지막에 container div를 구해 `appendChild` 메소드로 태그들을 렌더합니다.

rust의 메소드를 호출하기 위해 Tauri의 `invoke` 메소드의 facade를 **TauriCore**에서 작성하였으며 이 `invoke` 함수는 `Promise`를 반환하므로 `Future`로 바꾸어 scala에서 처리하였습니다.

* javascript
    `const { invoke } = window.__TAURI__.core;`
* scala.js
    ``` scala
    @js.native
    @JSGlobal("window.__TAURI__.core")
    object TauriCore extends js.Object:
        def invoke(fname: String, a: js.Object): Promise[js.Object] = js.native
    ```

![ScreenShot](assets/tauri_scalajs.jpg)
