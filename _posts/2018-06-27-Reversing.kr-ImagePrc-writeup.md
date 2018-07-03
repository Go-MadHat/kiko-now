---
layout: post
title: ! '[reversing.kr] ImagePrc writeup'
tags:
 - Write-up
 - reversing.kr
 - smlsml
---
학기 중에 reversing 문제를 거의 풀지 않고 지냈기 때문에 갑자기 어려운 것을 도전하기엔 무리가 있었습니다.

그래서 시작은 저번 방학 때 풀던 reversing.kr을 다시 풀며 감을 익히기로 했습니다.

이번에 풀어본 문제는 ImagePrc이라는 문제입니다.

![]({{ site.baseurl}}/images/smlsml/ImagePrc/1.PNG)

먼저 파일을 열어보면 위와같은 창이 뜹니다.
여기에는 그림을 그릴 수 있도록 돼있고, check이라는 버튼을 누르면 답인지 검사합니다.

![]({{ site.baseurl}}/images/smlsml/ImagePrc/2.PNG)

일단 아무렇게나 그림을 그린 뒤 확인을 해보면

![]({{ site.baseurl}}/images/smlsml/ImagePrc/3.PNG)

이와같이 Wrong이라는 문구를 확인 할 수 있었습니다.

immunity debugger를 사용하여 파일을 열어서 string들을 확인해보면 wrong이 있는 부분을 확인할 수 있습니다.

![]({{ site.baseurl}}/images/smlsml/ImagePrc/4.PNG)

여기에서 조금만 더 위로 올라가보면 반복, 분기문이 보이는데

![]({{ site.baseurl}}/images/smlsml/ImagePrc/5.PNG)

여기에서 dl, bl의 값을 비교하여 값이 다르다면 wrong이 있는 부분으로 가는 것을 확인할 수 있습니다.

![]({{ site.baseurl}}/images/smlsml/ImagePrc/8.PNG)

파일을 실행시키며 분석하여 원본 파일의 정보가 있는 위치를 알아냅니다.

![]({{ site.baseurl}}/images/smlsml/ImagePrc/9.PNG)

그러면 이와같은 bmp 형식의 내용이 보입니다.
보통 이렇게 FF가 많이 보이는 것은 배경이 흰색인 bmp파일에서 볼 수 있는데, 자주 HxD를 통해서 bmp파일을 봤던 기억에 바로 알 수 있었습니다,

bmp 파일이 16진수 6자리, 즉 3byte로 색상을 나타내 각 픽셀별로 저장하기 때문에 흰색을 나타내는 FF FF FF가 계속해서 이어진다면 흰색이 배경인 bmp 파일임을 알 수 있는 것입니다.
검정색은 00 00 00으로 나타나게 됩니다.


![]({{ site.baseurl}}/images/smlsml/ImagePrc/6.PNG)

아까전의 부분에서 한참 더 위로 올리다보면 GetSystemMetrics를 통해서 창을 만드는 것을 볼 수 있는데, hight = 150, width = 200 임을 확인할 수 있습니다.

그림판을 이용해서 저 크기의 bmp파일을 만들어주고, 그림이 시작되는 부분을 디버거에서 확인한 값들로 채워줍니다.

![]({{ site.baseurl}}/images/smlsml/ImagePrc/10.PNG)

이렇게 하고 저장을 하면

![]({{ site.baseurl}}/images/smlsml/ImagePrc/12.PNG)

이와같이 정답이 적혀있는 그림을 확인할 수 있습니다.
(정답부분은 일부러 가렸습니다.)

이렇게 문제의 풀이를 끝내고 난 뒤, 다른 사람들의 풀이도 검색 해보았습니다.
사실 이 문제는 어렵지 않게 풀었으나 다른 사람들의 풀이를 보면 여러 툴을 사용해서 빠르고 쉽게 풀어내는 모습을 볼 수 있습니다.
따라서 리버싱 문제를 풀이하는데 있어서 다양한 툴의 종류와 사용법을 한번씩이라도 살펴보는것은 굉장히 좋다고 생각하여 이번에도 찾아보았습니다.
찾아본 결과 비교할 원본 파일이 있는 곳의 위치를 몰라도 PEview라는 프로그램을 사용하면 쉽게 내용을 볼 수 있다고 합니다.

![]({{ site.baseurl}}/images/smlsml/ImagePrc/13.PNG)

위는 PEview를 사용하여 확인을 한 모습입니다.
그림파일이 있다면 그게 MANIFEST부분에서 확인이 가능하다고 합니다.
실제로 제가 찾은 값과 비교해보면 값이 제가 찾던 부분과 동일함을 알 수 있습니다.

이 문제는 프로그램을 열었을 때 그림을 그리고 check를 하는 버튼이 있기 때문에 보자마자 그림을 비교하는 부분이 있을거라 쉽게 예측할 수 있습니다.
따라서 PEview를 미리 알고 사용했다면 더 빨리 문제를 해결했을 것이라 생각합니다.

