---
layout: post
title: Scala Regex의 Match object
date: 2025-01-10 08:34
category: programming
tags: scala
---

얼마전 아마존 프라임에서 [컨티넨탈](https://www.primevideo.com/detail/The-Continental/0NM6C9HKZUN0F5335N09FWX6R2)을 보았습니다. 영화 존윅의 배경으로 나왔던 컨티넨탈 호텔에 윈스턴이 어떻게 자리잡게 되는지를 다룬 내용인데 재밌었습니다. 특히 6,70 년대의 좋은 음악들이 곳곳에 배치되어 있어 OST 앨범을 찾아보았는데요, 따로 발매된 것은 없었고 누가 32곡의 [플레이리스트](https://music.amazon.com/user-playlists/c6bc285b374c4de1a0ad0a49e69dccd6c2l0)를 만들어 올려놓은 것을 찾았습니다. 

파이어폭스에서 텍스트로 저장한 것의 첫 부분은 아래와 같습니다. 
```
  * Amazon MusicAmazon Music <https://music.amazon.com/>
  *
    Home
    Home
  *
    Podcasts
    Podcasts
  * Library

    Cancel
  * Sign in

*

*

Shuffle

Try Amazon Music Unlimited
1

I Feel Love <https://music.amazon.com/albums/B0B2KVHJMZ?
trackAsin=B0B2KY3C5Z>

Donna Summer <https://music.amazon.com/artists/B000QJPYJC/donna-summer>

Platinum Jubilee - Queens of Pop <https://music.amazon.com/albums/
B0B2KVHJMZ>

05:51

2

Samba Pa Ti <https://music.amazon.com/albums/B00138CYD4?
trackAsin=B00137GKZI>

Santana <https://music.amazon.com/artists/B000QKDMIG/santana>

Ultimate Santana <https://music.amazon.com/albums/B00138CYD4>

04:45
```
처음 헤더 부분을 제외하면 번호, 곡 제목, 아티스트, 앨범, 곡 길이 순서대로 나열되어 있습니다. Regular expression으로 읽어들이면 간단하겠다는 생각이 들더군요. 결론으로 바로 가면 다음 소스를 작성했습니다. 

```scala
object ParseTracts:
    private val lines = io.Source.fromFile("TheContinentalPlaylist.html").getLines().toList
    private val contents = lines.mkString("\n")
    private val wordsWithSpaceAndSiteLink = raw"""([^<]+)<[^>]+>\s*"""
    private val tractRg =   // 중복되는 내용을 다른 변수에 저장하고 string interpolation 사용
        raw"""(\d{1,3})\s*${wordsWithSpaceAndSiteLink}${wordsWithSpaceAndSiteLink}${wordsWithSpaceAndSiteLink}\d{2}:\d{2}""".r

    def parse =
        // println(tractRg.findFirstIn(contents).getOrElse("수정해서 다시 도전!"))
        val matches = tractRg.findAllMatchIn(contents)
        matches.foreach: m =>
            val (no, title, singer, album)  // pattern matching 사용해서 리스트의 값들을 Tuple로 바꾸고 변수에 저장하기
                = Tuple.fromArray(m.subgroups.take(4).map(_.trim).toArray.asInstanceOf[Array[String]])
            println(s"$no) $title in $album by $singer")

ParseTracts.parse
```

특별할 것은 없습니다만, 한두가지 팁으로 삼을만한 것은 tractRg 변수에 저장된 **regular expression식에서 반복되는 내용을 다른 변수에 저장한 다음 참조하는게 가능**하다는 것과 **Match형 object m의 subgroups(List[String]형)에서 4개를 선택한 다음 pattern matching을 사용해 변수들에 저장**하는 것 정도가 되겠습니다. 컴파일시 Tuple.fromArray에서 워닝이 나오기는 합니다만 로직만 확실하면 문제되지는 않을것 같습니다. 실행시키면 다음과 같이 곡 목록이 출력됩니다. 이렇게 출력해도 애플 뮤직에 플레이리스트로 만들려면 결국 하나씩 수동으로 추가해야 되겠네요. Apple의 API도 있겠지만 그건 언젠가로 미루면서 2025년 첫 포스팅을 마칩니다.

```
1) I Feel Love in Platinum Jubilee - Queens of Pop by Donna Summer
2) Samba Pa Ti in Ultimate Santana by Santana
3) Chicken Strut in Funkify Your Life: The Meters Anthology by The Meters
4) If You Leave Me Now in Road Trip by Chicago
5) Crimsom And Clover in Best Of The Best by Tommy Roe
6) La Grange in Tres Hombres by ZZ Top
7) Roundabout (2024 Remaster) in Fragile (Super Deluxe) by YES
8) Yes Sir, I Can Boogie in Baccara by Baccara
9) The Boss (From "Black Caesar" Soundtrack) [feat. The J.B.'s] in Buenos Días Con Música Vol. 1 by James Brown
10) Daddy Cool in Daddy Cool by Boney M.
11) Children of the Grave (2014 Remaster) in Master of Reality (2014 Remaster) by Black Sabbath
12) Baker Street in City to City by Gerry Rafferty
13) Homicide in Separates by 999
14) Magic Man in Dreamboat Annie by Heart
15) Without You in Nilsson Schmilsson by Harry Nilsson
16) Welcome to the Machine in Wish You Were Here by Pink Floyd
17) Seasons Come, Seasons Go in Laurel Canyon Vibes by Bobbie Gentry
18) Ole in Dancing Time: The Best of Eastern Nigeria's Afro Rock Exponents 1973-77 by The Funkees
19) Funky Nassau, Pt. 1 in Funky Nassau by The Beginning Of The End
20) Get Up Offa That Thing in Cold Sweat & Other Soul Classics: James Brown by James Brown
21) A Sweet Little Bullet From A Pretty Blue Gun in Blue Valentine (Remastered) by Tom Waits
22) Take the Long Way Home in Take the Long Way Home by Roger Hodgson
23) Right Place Wrong Time in Rock Internacional [Explicit] by Dr. John
24) Fight the Power, Pts. 1 & 2 in The Essential Isley Brothers [Clean] by The Isley Brothers
25) Oh Honey in Late Night Tales: BADBADNOTGOOD by Delegation
26) Hip Hug-Her in Atlantic Top 60: Sweat-Soaked Soul Classics by Booker T. and The MG's
27) One (Single Version) in Catchy Tunes Of The Past Vol 8 by Three Dog Night
28) Barracuda in Little Queen by Heart
29) Soul Sacrifice in Santana by Santana
30) Baba O'Riley (Remastered 2022) in Who’s Next : Life House (Super Deluxe) by The Who
31) What A Diff'rence A Day Made in What A Diff'rence A Day Makes! (Expanded Edition) by Dinah Washington
32) New York Groove in American Place Names by Ace Frehley
```