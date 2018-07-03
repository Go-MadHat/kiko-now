---
layout: post
title: ! '[Plaid CTF] Butterfly write up'
tags:
 - Write-up
 - CTF
 - Plaid CTF
 - nalda
---

2016년도 Plaid CTF에서 나온 문제이다. 이 문제로 선정 한 이유는 공부를 시작 한 후 처음 접한 Pwnable 문제로, CTF 당시에는 풀지 못하였고 그동안 잊고 살았다가 최근 다시 생각나서 풀어보게 됐다.

출제 당시 150 point였으며, 바이너리가 제공되었다.


먼저 checksec을 이용하여 해당 바이너리를 보면 특이사항으로 Stack Canary가 존재하며 64bit 이다.
![]({{ site.baseurl }}/images/nalda/Butterfly/checksec.PNG)

### Program Logic
![]({{ site.baseurl }}/images/nalda/Butterfly/logic.PNG)

프로그램은 단순하게 구성되어있으며 위의 사진은 main의 hex ray 결과이다.

fgets를 이용하여 50byte를 입력받고 이를 strtol함수를 이용하여 정수값으로 변환한다.
20번째 줄을 보게 되면 변환된 값의 마지막 3비트가 Right Shift된다.

이 후 mprotect가 실행되는데, 

![]({{ site.baseurl }}/images/nalda/Butterfly/mprotect.PNG)

위의 이미지와 같이 mprotect가두번 실행하게 된다.
이 mprotect는 현재 할당된 메모리의 실행권한을 변경해주는 함수이다.
이 때, 3번째 값에는 rwx로 표현된다!
즉 7은 rwx를, 5는 r-x의 권한을 의미한다.

여기서 의문점은 왜 메모리의 권한을 7로 write권한을 주었다 다시 5로 write권한을 빼게되는것일까?
코드영역의 권한만 바꿔준다면 WRITE권한을 가질 수 있기 때문이다.

![]({{ site.baseurl }}/images/nalda/Butterfly/main.PNG)

위의 이미지는 main함수의 끝 부분에 위치한 에필로그 영역이다.
add $0x48,$rsp instruction을 이용하여 메모리 영역을 다시 복구해준다.

여기에서 $0x48에 해당하는 opcode를 1바이트만 수정하게된다면 retq instruction이 실행되는 시점의 rsp 포인터를 원하는 데이터가 들어가있는 메모리로 변경해줄 수 있다.


그렇다면 write가 가능한 메모리 영역이 어디에있을지 ida를 통해 확인해보면

![]({{ site.baseurl }}/images/nalda/Butterfly/memory.PNG)

위의 이미지와 같이 .init_array가 발견된다.

![]({{ site.baseurl }}/images/nalda/Butterfly/init_array.PNG)

이를 쫓아가게 되면 start에서 main함수가 실행되기전에 실행되는걸 알 수 있다.

이를 이용하여 exploit을 하면된다!

### Exploit
추후공개..!





