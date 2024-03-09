---
layout: post
title: Jekyll용 VSCode snippet
date: 2024-02-21 11:07
category:  setup
tags: 
---

이 사이트는 [Jekyll](https://jekyllrb.com)을 사용하여 작성해서 github에 호스팅하고 있습니다. 포스팅 할때마다 git를 사용하는게 귀찮은 면도 있지만 익숙해지면 에디터에서 마크다운으로 작성하는 맛이 있는것 같습니다.

Jekyll을 사용하려면 처음부분에 페이지에 대한 정보를 줘야 하는데 일일이 복사해 붙이기도 귀찮아 제가 사용하는 [VSCode에서 snippet 작성하는 법](https://code.visualstudio.com/docs/editor/userdefinedsnippets)을 찾아 보았습니다. 

### 형식 
JSON으로 작성

```
"이름" : {
    "prefix" : [ 사용할 약어(들) ],
    "body" : [ 내용 ],
    "description" : 설명
}
```

단순한 확장이면 위 내용으로도 사용가능하나 `Tab`키를 사용해서 필요한 내용을 넣고 싶을 때는 `$1`과 같은 **place holder**를 이용합니다. 탭을 누름에 따라 1부터 순서대로 커서가 이동하게 되며 `${1:기본값}`과 같은 형식으로 기본값을 지정해 줄수 있고 `${1|java, scala, kotlin|}`와 같이 지정된 값중 선택하도록 할 수도 있습니다. **regular expression**을 사용하면 사용자가 입력한 값을 필요에 따라 변경할수도 있습니다.

맥에서는 아래와 같이 **Code > Settings > Configure User Snippets** 로 snippet을 입력합니다.

![그림](assets/snippet_menu.jpg)

아래는 제가 사용하는 Jekyll 작성용 snippet 입니다. Category와 tags에 기본값들을 주었고, TWE는 같은 번호를 사용해서 사용자가 선택한 값을 2곳에서 배껴 쓰며 regular expression으로 대문자, 소문자로 변환하고 있습니다. 값 선택과 regular expression을 동시에 사용하면 더 편리하겠는데 아직 지원하지 않는 것 같습니다.

```json
{
	"Title": {
		"prefix": "title",
		"body": [
			"---",
			"layout: ${1:post}",
			"title: ${2:title}",
			"date: $CURRENT_YEAR-$CURRENT_MONTH-$CURRENT_DATE $CURRENT_HOUR:$CURRENT_MINUTE",
			"category: ${3|programming, random|}",
			"tags: ${4|scala, cats, rust|}",
			"---"
		],
		"description": "Add title section"
	},
	"TWE": {
		"prefix": "twe",
		"body": [
			"${1|tip, warning, danger|}",
			"> ##### ${1/(.*)/${1:/upcase}/}",
			"> ",
			"> $2",
			"{: .block-${1/(.*)/${1:/downcase}/}}"
		],
		"description": "Tip,Warning or Error"
	},
}
```