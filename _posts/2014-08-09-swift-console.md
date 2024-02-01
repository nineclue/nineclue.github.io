---
layout: post
title: Swift를 콘솔에서 실행하기
category: setup
tags: swift
---

Swift를 실행가능한 XCode6가 아직 베타 과정에 있기 때문에 문서에서 나오듯이 `xcrun swift`를 실행하면 swift 모듈을 찾을 수 없다는 메시지가 나온다. 콘솔에서 개발자 디렉토리를 설정할 수 있는 명령어가 *xcode-select*이며 2014년 8월 9일 현재 XCode 6 베타 5인 경우 `sudo xcode-select -s /Applications/Xcode6-Beta5.app/Contents/Developer/`와 같이 명령 내리면 이후 swift REPL을 사용할 수 있다.

XCode 6가 정식으로 발표되면 필요없는 포스팅이 될 것이고 다른 베타 버전이 발표되면 그에 따라 앞의 명령어에서 Beta 다음의 숫자를 맞춰줘야 할 것이다.
