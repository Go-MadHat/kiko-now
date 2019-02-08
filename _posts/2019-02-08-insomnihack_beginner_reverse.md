---
layout: post
title: ! "[Insomni'hack 2019] beginner reverse write up"
excerpt_separator: <!--more-->
comments : true
tags:
  - insomnihack2019
  - rev
  - beginner reverse
  - chaem
---
beginner reverse는 64bit 바이너리의 rust 언어로 된 리버싱 문제이다.  
막상 대회때는 제대로 보지 않은 문제인데, write up 쓰려고 찾다가 고르게 되었다 ㅎㅎ  
비슷한 시기에 나왔다는 go언어는 해봤지만, rust는 좀 어렵다고 해서 거리감을 가지고 있었는데 이렇게 리버싱으로 접하게되어 반가웠다.  
<!--more-->
rust 언어로 되어있어서 더 어렵거나 한 문제는 아니었지만...!  
rust는 어떤 언어인지 좀 찾아봤는데, 메모리 오류를 없애기 위한 목적을 가지고 만들어진 언어라고 한다.  
언젠가 한 번 사용해 봐야겠당 ㅎㅎ  

```
ubuntu@ubuntu:~/ctf/2019/insomni$ file beginner_reverse-466bdf23cf344b8ee734a8ae86620ac72a37bb81a950b30eae6709f185c3b247
beginner_reverse-466bdf23cf344b8ee734a8ae86620ac72a37bb81a950b30eae6709f185c3b247: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=0f81b8fd8f6542e92e7e98c809bd2fd532e52c79, not stripped
```
문제를 푸는 데 상관없는 코드들이 많은 문제였다.   
IDA로 분석한 결과 beginer_reverse::main::h80fa15281f646bc1() 부분을 발견할 수 있었다. 아래의 문자들을 이용하여 연산하는 것으로 유추되었다.  

![]({{ site.baseurl }}/images/chaem/insomni_rev/beginner_reverse_01.JPG)  

그리고 do ~ while문을 통해 다음과 같은 연산을 반복하여 일치하는 값을 찾는 것을 알 수 있다.  

![]({{ site.baseurl }}/images/chaem/insomni_rev/beginner_reverse_02.JPG)  

그리고 아래 부분에서 뒷부분의 값도 찾아낼 수 있었다.  

![]({{ site.baseurl }}/images/chaem/insomni_rev/beginner_reverse_03.JPG)  

따라서 이 부분을 python으로 작성하면 다음과 같이 연산하여 flag값을 추출해 낼 수 있다.  
```python
input = [0x10e, 0x112, 0x166, 0x1c6, 0x1ce, 0xea, 0x1fe, 0x1e2, 0x156, 0x1ae, 0x156, 0x1e2, 0xe6, 0x1ae, 0xee, 0x156, 0x18a, 0xfa, 0x1e2, 0x1ba, 0x1a6, 0xea, 0x1e2, 0xe6, 0x156, 0x1e2, 0xe6, 0x1f2, 0xe6, 0x1e2, 0x1e6, 0xe6]
flag=''    
for i in input:
    flag += chr(( i >> 2) ^10)
print flag
```

위의 코드로 INS{y0ur_a_r3a1_h4rdc0r3_r3v3rs3 까지 알아낼 수 있다.  

INS{y0ur_a_r3a1_h4rdc0r3_r3v3rs3r}
