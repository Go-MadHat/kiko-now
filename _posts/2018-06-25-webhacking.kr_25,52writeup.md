---
layout: post
title: "webhacking.kr 25번, 52번 writeup"
excerpt_separator: <!--more-->
tags: 
 -writeup
 -webhacking.kr
 -rlawogns
---
webhacking.kr에 있는 25번과 52번 문제를 풀어보았다.

25번 문제는 아래와 같다.
![]({{ site.baseurl }}/images/rlawogns/webhacking.kr_25,52writeup/25.PNG)
처음화면은 hello world이 출력이 되고 ?file=hello이 적혀있는 것을 볼 수가 있다.
hello뒤에 .txt를 안 붙였음에도 hello.txt의 내용으로 위추되는 내용이 출력되는 걸로 봐서
.txt를 자동으로 붙여주는 것 같다.
password.php의 내용을 봐야할 것 같으므로
flie=password.php를 해주고 %00을 붙여 뒤의 붙을 .txt를 무시하여 줬더니
![]({{ site.baseurl }}/images/rlawogns/webhacking.kr_25,52writeup/25clear.PNG)
이와 같이 25번 문제의 패스워드가 나왔고 이를 입력하여 문제를 풀게 되었다.

52번 문제는 아래와 같다.
![]({{ site.baseurl }}/images/rlawogns/webhacking.kr_25,52writeup/52_1.PNG)
헤더인젝션에 관한 문제라고 떡하니 알려주고 있다.
헤더생성을 누르면 아래와 같이 id=wogns0411이라는 것이 생기는 것을 볼 수가 있다.
![]({{ site.baseurl }}/images/rlawogns/webhacking.kr_25,52writeup/52_2.PNG)
문제를 보면 $_GET[id]로 헤더인젝션을 해서 id=wogns0411 쿠키를 생성하라고 되어있다.
그래서 Set-cookie를 사용해 쿠키를 생성하려고 했는데 아무리 해도 안되서
이것저것 뒤져보았더니 Set-cookie를 하면 풀리지 않고
클리어 조건에 있는 clear부분을 해줘야한다는 것을 알게 되었다.
그래서 CRLF(캐리지 리턴, 라인 피드)를 이용해
id=wogns0411%0a%0dclear: wogns0411를 하였더니 풀리는 것을 볼 수 있었다.
![]({{ site.baseurl }}/images/rlawogns/webhacking.kr_25,52writeup/52clear.PNG)
%0a - 라인 피드 %0d - 캐리지 리턴
