---
layout: post
title: OSX matplotlib에서 한글폰트
category: programming
tags: matplotlib ipython
---
matplotlib에서 설치된 폰트는 다음과 같은 명령으로 볼수 있다.

```python
set([f.name for f in matplotlib.font_manager.fontManager.ttflist])
```

검색해 보면 원하는 폰트를 사용하기 위해서는 다음과 같은 명령을 사용한다고 되어 있는데 폰트가 지정되는 경우도 있고 아닌 경우도 있다.

```python
matplotlib.rc("font", family="나눔고딕코딩")
```

matplotlib에는 OSX의 폰트를 보여주는 함수가 따로 마련되어 있다. 다음과 같은 명령으로 설치되어 있는 폰트들을 볼 수 있다.

```python
set([f for f in matplotlib.font_manager.OSXInstalledFonts()])
```

다음과 같이 FontProperties에 원하는 폰트를 직접 로딩해서 title에 사용하도록하면 표시할 수 있다.

```python
import random
cf = matplotlib.font_manager.FontProperties("나눔고딕코딩", fname='/Users/nineclue/Library/Fonts/나눔고딕코딩.ttf', size=20)
xs = [int(random.random()*10) for i in range(20)]
plot(xs)
title("그래프 제목", fontproperties=cf)
show()
```

`matplotlib.rc`에 FontProperties를 지정할 수 있으면 좋겠는데 가능한지 잘 모르겠다. 몇가지 인자로 시도해 봤는데 실패.

![OSXHangulFonts in matplotlib]({{ site.url }}/assets/OSXmatplotlibFont.png)
