---
layout: post
title: ! 'Swamp CTF Dragon '
excerpt_separator: <!--more-->
tags:
  - Koon
  - Write-up
  - CTF
  - SwampCTF2018
---

Swamp ctf에 나온 reversing 문제중 가장 쉬웠다. 음... Swamp ctf에 나온 문제 중에 가장 쉬운 듯 하다.(배점이 가장 낮앗음 ㅇㅇ)
문제 테마 자체는 던전을 들어가, 드래곤을 만나는 것이다.

그 이후로는 당연히 드래곤과 싸워 이기는 것이 목표이자, flag를 얻는 방법이다.

<!--more-->

## 풀기
이 바이너리를 실행 하면...

![]({{ site.baseurl }}/images/Koon/swamp/dragon_1.PNG)
소설처럼, 줄거리가 나온다..
엔터를 치며... 계속 읽다가, 어느 순간 용한테 죽는 걸로 끝나버린다.

![]({{ site.baseurl }}/images/Koon/swamp/dragon_2.PNG)
IDA를 통해 바이너리를 분석해보면... 그래프 흐름에서 중간 부분(?)이 빡세 보인다.

좀, 분석해보면...
![]({{ site.baseurl }}/images/Koon/swamp/dragon_3.PNG)
이상한 부분이 보인다.
위의 사진 속에 빨간색으로 표시한 부분(임의로 표시)들이 있는데, 그부분들이 일반적인 흐름에 들어가지 않는다.

이 말은 빨간 부분은 건너뛰고 바이너리가 동작한다.

그리고 이 빨간 부분들은
![]({{ site.baseurl }}/images/Koon/swamp/dragon_4.PNG)
foo 시리즈 함수들이다.

foo 시리즈 함수들이 중요하다는 것을 확실히 알게된다.

foo를 분석해보자!

![]({{ site.baseurl }}/images/Koon/swamp/dragon_5.PNG)

foo를 분석해보면... 사실 너무 쉽다.(특정 아스키 값이 나와있는데, 이를 문자로 바꿔주면, flag의 일부를 하나씩 준다는 것을 알 수 있다.)


## 정리
흐름 상으로 정리를 하면,
foo 함수로 들어 가기 전에 cmp 어셈을 통해 비교하는 부분이 있다.
이 부분에 인위적으로 하드코딩되어 서로 다른 값을 넣고 비교하는 것이 보이는데... 이것을 통해 foo 함수의 진입을 막아 놓았다.

이러한 검증 부분을 고치거나, 아니면 foo 함수로 들어가게 하면... foo에서는 flag 값을 한 글자씩 주어

모든 foo 시리즈의 함수들을 들어가게 되면, flag를 최종적으로 알 수 있게 된다.


