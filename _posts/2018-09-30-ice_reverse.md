---
layout: post
title: ! ' ICE CTF 2018 LOCK '
excerpt_separator: <!--more-->
tags:
  - Koon
  - Write-up
  - CTF
  - IceCTF2018
  - ltrace
  - gdb
---

ice ctf에 나온 reversing 문제 중 가장 낮은 문제였던 200점짜리 "Lock out"이라는 문제를 풀려고 한다.
사실 문제 자체는 쉬운데... 왜 200점인지는 잘 모르겠다. 내 생각으로는 100점 밑일것 같은데...

<!--more-->

## 문제 상황
lock 파일이 있고, flag 파일이 존재하여 lock 파일을 풀 시, 권한 상승이 되어 flag 파일에 접근 할 수 있는 구조로 되어 있다.

다만 대회가 끝났기 때문에... 이러한 상황은 무시하고, lock 파일을 해제하여 권한 상승을 목적으로만 한다.
## 풀기
처음 lock 파일을 실행하면...

![]({{ site.baseurl }}/images/Koon/ice/ice_1.PNG)

패스워드를 물어보고, 패스워드 입력 시, 그 입력 값에 따라 권한 상승이 일어나거나, 잘못되었다는 메시지를 출력한다.

![]({{ site.baseurl }}/images/Koon/ice/ice_2.PNG)
바로 본론으로 들어가서... 이문제가 쉬운 이유는, "strcmp"를 쓴다는 것을 바이너리를 분석하면 바로 알게되고, 그에 따라 gdb 또는 ltrace를 사용하여 strcmp 위주로 분석만 하면 답이 바로 나온다.

### gdb
![]({{ site.baseurl }}/images/Koon/ice/ice_3.PNG)
break 포인트를 strcmp로 직접 입력하여 strcmp의 주소에 브레이크를 걸어, 그 후에 레지스터의 값을 분석해보면... 답이 나와버린다.

### ltrace
gdb 대신 ltrace를 이용하는 경우에는, 약간의 작업이 좀더 필요하다.

![]({{ site.baseurl }}/images/Koon/ice/ice_4.PNG)
ltrace를 통해 strcmp를 보게되면... 내가 입력한 값과 특정 값을 비교하는게 보인다. 그래서 그 특정 값이 답인줄 알겠지만... 특정 값이 전부 쓰인것이 아니라 잘려서 나온것이다. 그렇기 때문에 이 나머지 값들에 대해 알아야한다.

![]({{ site.baseurl }}/images/Koon/ice/ice_5.PNG)
잘린 값들을 다시 입력값으로 넣게 되면... 결과과 -무슨 값이 나온다.(노란색 표시)
이부분을 보게 되면... 90은 사실 영어 아스키로 Z를 의미한다.
-리턴값으로 나머지 부분을 계속해서 출력한다는 것을 알 수 있다.
이 것을 통해 하나 하나씩 알아내면 된다.

## 결론
음... 매우 쉽다.
왜 200점 짜리 인지는 모르겠다.
사실 ltrace에 대해서는 사용하지 않아서 몰랏는데, 알게되서 도움이 되었다.
