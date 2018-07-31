---
layout: post
title: ! "[Secuinside2017] once Writeup"
comments: true
excerpt_separator: <!--more-->
tags:
  - Write-up
  - CTF
  - Pwn
  - Nalda
  - 2017 Secuinside
---

아마도 곧...?  secuinside CTF가 있으리라 생각하며 대회를 준비한다는 생각으로 2017 Secuinside의 ohce 문제를 풀어보었다.
<!--more-->
![]({{ site.baseurl }}/images/nalda/ohce/ohce_01.PNG)

먼저 파일을 실행하게 되면, 3개의 메뉴가 뜨게 된다.
```
1. echo
2. echo(Reverse)
3. Exit
```

echo는 입력을 그대로 출력해주며 2번은 반대로 출력, 3번은 종료된다

![]({{ site.baseurl }}/images/nalda/ohce/ohce_02.PNG)
보호기법은 `NX`만 `ENABLED`된것을 확인할 수 있다.

![]({{ site.baseurl }}/images/nalda/ohce/ohce_03.PNG)
먼저, IDA를 이용하여 확인해보면, symbol이 없어서인지 이름을 정확히 유추할 수 없다.
해서 strace를 이용하여 syscall을 확인 해보았다.

>nalda@nalda:~/study/ctf/2017-secuinside$ sudo strace -fFi -p 24669

![]({{ site.baseurl }}/images/nalda/ohce/ohce_04.PNG)

이를 토대로 read를 통해 값을 받고 write를 통해 값을 출력한다는 사실을 확인할 수 있다.

![]({{ site.baseurl }}/images/nalda/ohce/ohce_05.PNG)

위의 사진은 `READ`함수(가칭) 의 내용인데,  17줄을 보면 v3 = 0x20LL을 선언한다. 이는, 32만큼 공간을 할당하게 된다.

이를 활용하여 buffer에 값을 받게 되며, 값을 받을 때에는 `enter(0x0a)` 값도 입력 buffer에 저장된다.

이 후 1byte씩 검증하면서 count 를 확인한다(line 31). 만일 32byte라면 다시 0x20씩 추가 할당 해 준다.

이를 확인하기 위해 a를 31,63 넣어보았다.

![]({{ site.baseurl }}/images/nalda/ohce/ohce_06.PNG)

이 때, 출력이 정상적이지 않을것을 확인할 수 있었다!