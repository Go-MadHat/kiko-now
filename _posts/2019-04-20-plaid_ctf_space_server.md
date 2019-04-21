---
layout: post
title: ! "[plaid ctf] space saver write up (수정 예정)"
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
~~그리고 왜 다른 풀이법으로 했을 떄 안되었는지 질문을 하고싶...ㅠ  ~~

아무튼 우선 하나의 .dd 파일을 얻을 수 있다.  
이 파일을 압축해제하면 하나의 png 파일이 나와서 그것을 분석해도 되지만, 나는 우선 strings 명령어로 보니, 꽤 그럴듯한 문자열들이 나오길래 다음과 같이 해볼 수 있다고 생각했다.  
다음과 같은 명령어를 통해 파일의 strings를 모은 것을 txt 파일로 뽑아보았다.  

```
root@ubuntu:/home/ubuntu/study/ctf# strings space_saver-90a5a93dfdda2d0333f573eb3fac9789.dd > test.txt
```

위에서 수행한 txt파일을 열어서 의심스러운 문구들을 조합하여 flag를 획득할 수 있다.  
IEND를 검색해보면 다음과 같은 결과들을 모아볼 수 있기 떄문이다.  

![](/images/chaem/pctf/space_saver_01.PNG)  
![](/images/chaem/pctf/space_saver_02.PNG)  
![](/images/chaem/pctf/space_saver_03.PNG)  

따라서 IEND뒤의 문자열들을 이어붙이면 flag가 된다!!!  

위와 같은 방법 외에도 binwalk를 이용한 방법을 보았었다.  
![](/images/chaem/pctf/space_saver_04.PNG)  
binwalk를 이용하여 다음과 같은 6개의 파일을 얻을 수 있는 것을 확인하였다.  
하지만 내가 hexeditor로 분석해 보았을 때는 6개의 파일에서 flag에 대한 정보를 얻을 수 없었다.  
![](/images/chaem/pctf/space_saver_05.PNG)  
파일이 깨져서 보이는 것 같기도 하고 ㅠㅠ  
아무튼 그래서 첫번째 방법으로 문제를 해결하게 되었다.  
