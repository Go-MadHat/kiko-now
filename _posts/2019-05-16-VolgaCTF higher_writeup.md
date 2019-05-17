---
layout: post
title: "[VolgaCTF 2019] higher Write-up"
comments : true
excerpt_separator: <!--more-->
tags :
  - sang-gamja
  - volga ctf
---

volga ctf higher write-up 입니다.
늦게 올리네요...

<!--more-->

문제는 단순한 mp3 파일에서의 데이터를 flag로 변환하는 문제였습니다.


우선 문제에서 주어진 mp3파일을 Audacity로 열어보았습니다.
![Alt text](/images/gamja/higher/higher1.jpg)
![]({{ site.baseurl }}/images/gamja/higher/higher1.jpg)

그리고 좀더 우리가 노가다 하기 편하게 바꾸어 주기 위해서 좌측에 recorded를 클릭하여 스팩트로그램으로 바꾸어 줍니다.

![]({{ site.baseurl }}/images/gamja/higher/higher2.jpg)

그렇다면 이것을 높은 파장과 낮은 파장을 구분하여 비트로 변환하면
`
010101100110111101101100011001110110000101000011010101000100011001111011010011100011000001110100010111110011010001101100011011000101111101100011001101000110111001011111011000100011001101011111011010000011001100110100011100100110010001111101
`
이렇게 나타납니다.
이 비트를 텍스트로 변경하면 플래그가 나옵니다.
변경은 https://codebeautify.org/binary-string-converter 에서 하였습니다.

![]({{ site.baseurl }}/images/gamja/higher/higher3.jpg)
