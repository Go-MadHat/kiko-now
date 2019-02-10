---
layout: post
title: ! "Insomni Beginner"
excerpt_separator: <!--more-->
tags:
  - koon
  - CTF
  - Write-up
  - reverse
---
Insomihack CTF에 나온 Beginer로 적습니다.
<!--more-->

#개요
Insomihack CTF에서 나온 Junkyard에 대해서 작성합니다. 이렇게 포스트 만들려다가,
Junkyard 특정 부분 분석하다가, 너무 진도 안나가서... 일단 대회 때 풀었던 비기너로 대체합니다.

#문제 설명
러스트 언어로 되어 있는 바이너리 문제입니다.
러스트 언어 자체가 고언어와 같이 바이너리로 보면, 문제 자체가 매우 더럽기 때문에... 시간이 많이 걸릴거라고 생각하였었는데, 대회 당일 많이 푼 문제인 만큼 쉬운 문제였습니다.

function Name의 경우도, beginer_reverse main이라고 적혀있기 때문에 이 부분만 보면 문제를 풀 수 있습니다.


#실행
![]({{ site.baseurl }}/images/Koon/Begin/begin_1.PNG)
입력 값을 받는 것 같은데, 출력 값을 내보내지 않는다.

#분석
beginer_reverse main 부분을 보게 되면 여러 쓰레기 값들이 많은데, 이를 다 무시하고... 밑에 부분을 보면 반복문을 통해 비교하는 루틴이 있습니다. 이 부분이 핵심입니다.
![]({{ site.baseurl }}/images/Koon/Begin/begin_2.PNG)

왼쪽에 있는(data_v33 + 4 * v24) >> 2) ^ 0xA == 이부분에서 data는 xmmword 들을 말하면, 이 것들은 바이너리 자체에 박혀져 잇는 데이터 값들입니다.
![]({{ site.baseurl }}/images/Koon/Begin/begin_3.PNG)
![]({{ site.baseurl }}/images/Koon/Begin/begin_4.PNG)
위의 사진에서 data_v33으로 이용되는 값들은 각 xmmword에서 ???00000???00000???00000??? 으로 ???에 해당하는 부분입니다.

이제 이 값들을 통해 코드를 만들면됩니다.

```python
data = [0x10e, 0x112, 0x166, 0x1c6, 0x1ce, 0xea, 0x1fe, 0x1e2, 0x156, 0x1ae, 0x156, 0x1e2, 0xe6, 0x1ae, 0xee, 0x156, 0x18a, 0xfa, 0x1e2, 0x1ba, 0x1a6, 0xea, 0x1e2, 0xe6, 0x156, 0x1e2, 0xe6, 0x1f2, 0xe6, 0x1e2, 0x1e6, 0xe6]
flag=''
for i in data:
        flag += chr(( i >> 2) ^10)
print (flag)

"""

끝!
