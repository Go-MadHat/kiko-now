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
---
![]({{ site.baseurl }}/images/Koon/ice/ice_1.PNG)
---
패스워드를 물어보고, 패스워드 입력 시, 그 입력 값에 따라 권한 상승이 일어나거나, 잘못되었다는 메시지를 출력한다.

![]({{ site.baseurl }}/images/Koon/ice/ice_2.PNG)
바로 본론으로 들어가서... 이문제가 쉬운 이유는, "strcmp"를 쓴다는 것을 바이너리를 분석하면 바로 알게되고, 그에 따라 gdb 또는 ltrace를 사용하여 strcmp 위주로 분석만 하면 답이 바로 나온다.

### gdb
![]({{ site.baseurl }}/images/Koon/ice/ice_3.PNG)

### ltrace
![]({{ site.baseurl }}/images/Koon/ice/ice_4.PNG)

![]({{ site.baseurl }}/images/Koon/ice/ice_5.PNG)
## 결론
음... 매우 쉽다.
왜 200점 짜리 인지는 모르겠다.
사실 ltrace에 대해서는 사용하지 않아서 몰랏는데, 알게되서 도움이 되었다.
