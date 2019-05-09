---
layout: post
title: ! "[Plaid ctf] space saver write up"
excerpt_separator: <!--more-->
comments : true
tags:
  - pctf
  - space_saver
  - misc
  - chaem
---

오늘은 plaid ctf의 misc분야로 출제되었던 space saver 문제에 대한 writeup을 써보도록 하겠다.  

<!--more-->

plaid ctf가 끝나고 문제를 풀어보기 위해! 쉬운 문제 찾아 삼만리 ㅠㅠ  
요즘 할 건 많은데 다 너무 하기 싫고... 막상 하는 것도 없으면서 이러고 있다,,,ㅠㅠ (갑자기 분위기 신세한탄ㅇㅅㅇ)  

일단 이번 문제는 풀이 방법이 여러가지일 수 있는 문제인데, 일단 내가 성공한 풀이법으로 설명하겠다.  
~~그리고 왜 다른 풀이법으로 했을 떄 안되었는지 질문을 하고싶...ㅠ (알려쥬셔서 감사합니당) ~~

아무튼 우선 하나의 .dd 파일을 얻을 수 있다.  

여기서 binwalk를 이용하여 다음과 같은 6개의 파일을 얻을 수 있는 것을 확인하였다.  
![](/images/chaem/pctf/space_saver_04.PNG)  
이 중에 48800 파일이 rar 압축파일인 것을 볼 수 있다.  
그래서 이 파일의 압축을 해제하려고 하니, password를 요구하였다.
![](/images/chaem/pctf/space_saver_08.PNG)  

이 password를 어디서 찾을 수 있을까 보니, 다른 png 형식의 파일들이 보였다.  
![](/images/chaem/pctf/space_saver_05.PNG)  

다음과 같은 명령어를 통해 파일의 strings를 txt 파일로 뽑아보았다.  

```
root@ubuntu:/home/ubuntu/study/ctf# strings space_saver-90a5a93dfdda2d0333f573eb3fac9789.dd > test.txt
```

위에서 수행한 txt파일을 열어서 의심스러운 문구들을 조합하여 password를 획득할 수 있다.  
IEND를 검색해보면 다음과 같은 결과들을 모아볼 수 있기 떄문이다.  

![](/images/chaem/pctf/space_saver_06.PNG)  

따라서 IEND뒤의 문자열들을 이어붙이면 password가 된다!!!  
password를 치면 flag가 적힌 이미지파일을 획득할 수 있다!  

![](/images/chaem/pctf/final.PNG)  
