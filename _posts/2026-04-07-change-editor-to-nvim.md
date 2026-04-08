---
layout: post
title: "nvim으로 에디터 변경"
date: 2026-04-07
category: setup
tags:
---

근황에 가까운 가벼운 포스팅입니다. 작년 9월이 마지막 포스팅이네요.

요즘은 gemini와 claude의 도움을 받아 이런 저런 작업을 해 보고 있습니다. 직접 코드를 작성하게 하지는 않고 도메인 지식을 물어보거나 에러 메시지 추적 같은 곳에 도움을 받고 있습니다. git 저장소도 github에서 codeberg로 옮겼다가 요즘은 집의 오래된 인텔 맥 미니에 우분투를 설치해서 자체 호스팅하고 있네요.

AI의 도움을 받고 있지만 AI 회사들의 트레이닝에 도움은 주기 싫어서 에디터도 VSCode에서 nvim으로 바꾸었습니다. 자꾸만 CoPilot을 사용하라는 메시지가 스팸처럼 느껴지기도 하구요. 아직까지 모르는것 투성이긴 합니다만 오래전에 잠시 vi를 사용해서인지 아직 머슬 메모리가 좀 남아 있어 놀랍기도 하구요. 마우스에 손대지 않고 작업을 하고 있으니 능률이 좀 올라가는 것 같기도 합니다. Scala는 metal이 nvim용 플러그인까지 나와 있어 에러 메시지나 코드 참조 같은 것들이 상당히 편하게 제공되기도 합니다.

> [TIP]
> 이런 저런 설정 파일들을 ~/.config/nvim에 저장하게 되는데 다른 곳에서도 사용할 수 있게 git 저장소에 백업하는 것을 gemini에게서 추천받았습니다.
>
> ```bash
> cd ~/.config/nvim
> git init
> git add .
> git commit -m "backup of lazyvim config"
> git remote add origin https://git주소/사용자아이디/nvim-config.git
> git branch -M main
> git push -u origin main
> ```
>
> 이후 다른 PC에서
>
> ```bash
> git clone https://git주소/사용자아이디/nvim-config.git ~/.config/nvim
> ```
