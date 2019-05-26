---
layout: post
title: ! 'Volga_CTF_2019_JOI'
excerpt_separator: <!--more-->
tags:
  - Koon
  - VolgaCTF2019
---

뒤늦게 쓰는 Volg CTF입니다. 리버싱 문제를 풀고 싶었지만,  외국 블로그, Github를 봐도 이해가 안가는 부분이 많아, 다른 문제를 선택하였습니다.

<!--more-->
# JOI
stego, forensic 문제로 나온 JOI입니다.

![]({{ site.baseurl }}/images/Koon/joi/joi_1.PNG)
JOI 문제를 보면, 위와 같은 png 파일을 제공합니다.

이 png의이미지는 QRcode 입니다.

![]({{ site.baseurl }}/images/Koon/joi/joi_3.PNG)
스캐너를 통해 QRcode를 찍게되면, 위와 같은 결과를 보여줍니다.

결과로 딱히 알수 있는게 없습니다.....

일단 stego 문제이니, 보통의 stego 문제처럼 stegSolve 도구를 사용해봅니다.

stegSolve 도구를 사용해서, 여러가지 살펴보니... QRcode가 변화 된다는 것을 확인 할 수 있습니다.

간단하게, 변화하는 모든 QRcode를 찍어 일일이 확인해봅니다.

![]({{ site.baseurl }}/images/Koon/joi/joi_2.PNG)
위와 같은 QR코드를 찍게되면...

![]({{ site.baseurl }}/images/Koon/joi/joi_4.PNG)
flag 값이 나옵니다.
