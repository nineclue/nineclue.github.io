---
layout: post
title: 펄린 노이즈
category: programming
tags: scala javafx
---

프로세싱에는 noise란 함수가 있습니다. 펄린 노이즈라고 하는 함수인데 입력값을 주면 난수값을 반환하는데 그것이 완전히 독립적이지 않고 비슷한 입력값은 비슷한 난수값을 반환합니다. 따라서 지형이나 물결과 같이 자연스러운 모습을 나타낼 때 사용하는 모양입니다. 다음 그림을 보시면 이해가 더 빠를지도 모르겠습니다.

![펄린 노이즈 1D & 2D]({{site.url}}/assets/PerlinDemo.png)

펄린은 사람 이름인데 80년대 영화 트론의 컴퓨터 영상을 제작하다 좀 더 자연스러운 느낌을 얻기 위해 이 노이즈 함수를 만들었다고 합니다. 아무튼 여러가지 방법으로 구현된 함수들이 발표되어 있는데 2002년에 펄린 자신이 발표한 [개선된 펄린 노이즈](http://mrl.nyu.edu/~perlin/noise/)에 간단히 주석을 달아보았습니다.

{% highlight java %}
// http://mrl.nyu.edu/~perlin/noise/
// JAVA REFERENCE IMPLEMENTATION OF IMPROVED NOISE - COPYRIGHT 2002 KEN PERLIN.

public final class ImprovedNoise {
	 // 인자 x, y, z에 따라 double 형 noise 값을 반환
   static public double noise(double x, double y, double z) {
   	  // X, Y, Z는 x, y, z의 정수 부분
      int X = (int)Math.floor(x) & 255,                  // FIND UNIT CUBE THAT
          Y = (int)Math.floor(y) & 255,                  // CONTAINS POINT.
          Z = (int)Math.floor(z) & 255;

      // x, y, z의 소수점 이하 부분만 남긴다
      x -= Math.floor(x);                                // FIND RELATIVE X,Y,Z
      y -= Math.floor(y);                                // OF POINT IN CUBE.
      z -= Math.floor(z);

      // 소수점 이하 좌표값의 fade 함수값을 얻는다.
      double u = fade(x),                                // COMPUTE FADE CURVES
             v = fade(y),                                // FOR EACH OF X,Y,Z.
             w = fade(z);

      // x, y, z가 위치한 인접 정수 좌표자리의 임의 난수값을 얻는다
      // AA : X,  Y, Z    AB : X,  Y+1, Z    BA : X+1, Y, Z
      // BA : X+1,Y, Z    BB : X+1,Y+1, Z
      int A = p[X  ]+Y, AA = p[A]+Z, AB = p[A+1]+Z,      // HASH COORDINATES OF
          B = p[X+1]+Y, BA = p[B]+Z, BB = p[B+1]+Z;      // THE 8 CUBE CORNERS,

      // 위의 인접 정수 자리에서 난수값과 x, y, z 좌표로의 벡터를 grad 함수에
      // 주고 lerp 함수로 fade 값의 중간값을 얻는다.
      return lerp(w, lerp(v, lerp(u, grad(p[AA  ], x  , y  , z   ),  // AND ADD
                                     grad(p[BA  ], x-1, y  , z   )), // BLENDED
                             lerp(u, grad(p[AB  ], x  , y-1, z   ),  // RESULTS
                                     grad(p[BB  ], x-1, y-1, z   ))),// FROM  8
                     lerp(v, lerp(u, grad(p[AA+1], x  , y  , z-1 ),  // CORNERS
                                     grad(p[BA+1], x-1, y  , z-1 )), // OF CUBE
                             lerp(u, grad(p[AB+1], x  , y-1, z-1 ),
                                     grad(p[BB+1], x-1, y-1, z-1 ))));
   }
   static double fade(double t) { return t * t * t * (t * (t * 6 - 15) + 10); }
   static double lerp(double t, double a, double b) { return a + t * (b - a); }
   static double grad(int hash, double x, double y, double z) {
      int h = hash & 15;                      // CONVERT LO 4 BITS OF HASH CODE
      double u = h<8 ? x : y,                 // INTO 12 GRADIENT DIRECTIONS.
             v = h<4 ? y : h==12||h==14 ? x : z;
      return ((h&1) == 0 ? u : -u) + ((h&2) == 0 ? v : -v);
   }
   static final int p[] = new int[512], permutation[] = { 151,160,137,91,90,15,
   131,13,201,95,96,53,194,233,7,225,140,36,103,30,69,142,8,99,37,240,21,10,23,
   190, 6,148,247,120,234,75,0,26,197,62,94,252,219,203,117,35,11,32,57,177,33,
   88,237,149,56,87,174,20,125,136,171,168, 68,175,74,165,71,134,139,48,27,166,
   77,146,158,231,83,111,229,122,60,211,133,230,220,105,92,41,55,46,245,40,244,
   102,143,54, 65,25,63,161, 1,216,80,73,209,76,132,187,208, 89,18,169,200,196,
   135,130,116,188,159,86,164,100,109,198,173,186, 3,64,52,217,226,250,124,123,
   5,202,38,147,118,126,255,82,85,212,207,206,59,227,47,16,58,17,182,189,28,42,
   223,183,170,213,119,248,152, 2,44,154,163, 70,221,153,101,155,167, 43,172,9,
   129,22,39,253, 19,98,108,110,79,113,224,232,178,185, 112,104,218,246,97,228,
   251,34,242,193,238,210,144,12,191,179,162,241, 81,51,145,235,249,14,239,107,
   49,192,214, 31,181,199,106,157,184, 84,204,176,115,121,50,45,127, 4,150,254,
   138,236,205,93,222,114,67,29,24,72,243,141,128,195,78,66,215,61,156,180
   };
   static { for (int i=0; i < 256 ; i++) p[256+i] = p[i] = permutation[i]; }
}
{% endhighlight %}

길이로는 그리 길지 않지만 고수가 아닌 저는 이해하는데 한참 시간이 걸렸고 사실 지금도 제대로 이해하는지 좀 아리송한 부분이 있습니다. 아무튼 좀 더 코멘트를 붙여보면

* fade 함수는 0, 1 사이에서 부드러운 S자 모양의 커브를 그리는데 0, 1 가까운 곳에서 좀 더 부드러운 변화를 가져오게 된다.
* lerp 함수는 0 ~ 1 범위를 가지는 인수 t 값으로 두 인수 a, b 사이 값을 구하는 것으로 linear interpolation 이라고도 부른다
* permutation은 정수를 임의로 배열해 놓은 것으로 x, y, z 주변 정수 좌표로 계속 permutation 배열의 값을 구하므로 난수 효과를 가져온다. AA는 p[p[p[X]+Y]+Z]의 값을 가지며 다시 이것의 p 배열값을 얻어 임의의 정수값을 얻게 되는데 X, Y, Z의 값이 같다면 항상 동일한 값을 얻게 된다.
* grad 함수는 위와 같은 방법으로 얻은 정수 인자 hash의 bit 값에 따라 인자 x, y, z를 선택하고 다시 부호를 선택하는데 결과적으로는 x, y, z중에서 2개의 값을 임의로 선택해서 더하거나 뺀 값을 얻게 된다.
* 마지막 return 부위에서는 grad 함수가 8번 호출되고 7번의 interpolation이 일어나는데 잘 관찰해보면 grad 함수 호출시에 첫번째 정수인자는 noise 함수의 인자 x, y, z를 둘러싸는 정수 단위 입방체의 8개 모서리이며 이후 부호가 다른 x, y, z 인자는 모서리에서 noise 함수 인자 x, y, z까지의 벡터값과 같다.

![Fade 커브]({{site.url}}/assets/FadeCurve.png)

결론적으로 인자로 주어진 x, y, z에서 인접한 8개의 정수 점에 임의의 정수값을 지정하고 그 값들과 정수점과 좌표의 벡터값에 기반해서 한 가지 실수값을 얻게 됩니다.

스칼라로 같은 함수를 구현해 보았는데 그리 깔끔하지는 않지만 동일한 값을 얻을 수 있습니다. 이론적으로는 3개 이상의 값을 주어도 값을 구할 수 있는데 적당한 grad 함수를 만들지 못해서 현재로는 1개에서 3개 사이의 값만 받도록 해 놓았습니다. 이 함수와 첫번째 프로그램의 소스는 [여기](https://gist.github.com/nineclue/68356dff7fd6049b6647)에서 보실 수 있습니다.

전체적으로는 [Understanding Perlin Noise](http://flafla2.github.io/2014/08/09/perlinnoise.html)의 내용이 많은 도움이 되었습니다.