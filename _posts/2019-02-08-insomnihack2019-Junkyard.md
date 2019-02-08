---
layout: post
title: ! "[Insomni'hack 2019] Junkyard(미완성)"
excerpt_separator: <!--more-->
tags:
  - nalda
  - CTF
  - Write-up
  - reverse
---
**Problem description:**
>Wall-E got stuck in a big pile of sh\*t. To protect him from feeling too bad, its software issued an emergency lock down. Sadly, the software had a conscience and its curiosity caused him to take a glance at the pervasive filth. The filth glanced back, and then...
>
>Please free Wall-E. The software was invented by advanced beings, so maybe it is way over your head. Please skill up fast though, Wall-E cannot wait for too long. To unlock it, use the login "73FF9B24EF8DE48C346D93FADCEE01151B0A1644BC81" and the correct password.
>
>Software was leaked a long time ago and is available here
<!--more-->

올 해 insomni'hack CTF는 대회 당일 일정으로 인해 참가를 하지 못하고 조금 미루다 이제서야 풀어보게 되었다.
먼저, 파일을 다운받아 file 명령어로 확인 한 결과 64bit ELF파일임을 확인할 수 있다.
![]({{ site.baseurl }}/images/nalda/Insomni/beginner_1.png)
파일을 실행 해 보면

```
nalda@nalda:~$ ./junkyard
Usage: ./chall user pass
```
userid와 pass를 넣으라고 한다
별 생각없이 nalda를 넣으면

```
nalda@nalda:~$ ./junkyard nalda 123123123
I don't like your name
```
이따구로 name이 맘에 안든단다..

description을 보면, `73FF9B24EF8DE48C346D93FADCEE01151B0A1644BC81` 과i correct password로 로그인 하라는걸 보아 name은 나온것 같다

```
nalda@nalda:~$ ./junkyard 73FF9B24EF8DE48C346D93FADCEE01151B0A1644BC81 44234324324234
Is that a password?
```

예아~
이제 password를 찾기위해 IDA를 보잣

```c
void __fastcall __noreturn main(int argc, char **argv, char **env)
{
  __int64 v3; // rdx
  char s; // [rsp+40h] [rbp-810h]
  unsigned __int64 v7; // [rsp+848h] [rbp-8h]

  v7 = __readfsqword(0x28u);
  if ( argc != 3 )
    sub_2EB3(8u, 0xFFFFFFFF);
  if ( (unsigned __int8)sub_2F91(argv[1], 0LL, env) ^ 1 )
    sub_2EB3(0, 0xFFFFFFFF);
  sub_1384();
  if ( (unsigned __int8)sub_2F91(argv[2], 1LL, v3) ^ 1 )
    sub_2EB3(1u, 0xFFFFFFFF);
  sub_1B85();
  sub_2AF8(7, off_8C18, &s);
  puts(&s);
  sub_1B40();
  memset(&s, 0, 0x800uLL);
  sub_3857(argv[1], argv[2]);
}
```

이제 decrypt를 수행하는 sub_3857(argv[1], argv[2])함수를 보자!

```c
void __fastcall __noreturn sub_3857(const char *argv1, const char *argv2)
{
  size_t arg1_len; // rax
  size_t arg2_len; // rax
  unsigned __int64 argv1_len; // rax
  unsigned __int64 argv2_len; // rax
  int v7; // ebx
  __int64 v8; // STC0_8
  int v9; // ST8C_4
  signed int v10; // eax
  signed int v11; // eax
  size_t v12; // rax
  size_t v13; // rax
  unsigned __int8 k; // [rsp+1Fh] [rbp-11B1h]
  signed int counter; // [rsp+7Ch] [rbp-1154h]
  unsigned int v17; // [rsp+80h] [rbp-1150h]
  unsigned int v18; // [rsp+84h] [rbp-114Ch]
  int v19; // [rsp+88h] [rbp-1148h]
  __int64 v20; // [rsp+90h] [rbp-1140h]
  unsigned __int64 j; // [rsp+98h] [rbp-1138h]
  signed __int64 v22; // [rsp+A0h] [rbp-1130h]
  signed __int64 v23; // [rsp+A0h] [rbp-1130h]
  unsigned __int64 i; // [rsp+A8h] [rbp-1128h]
  char *argv1_copy; // [rsp+B0h] [rbp-1120h]
  char *argv2_copy; // [rsp+B8h] [rbp-1118h]
  __int64 v27; // [rsp+C0h] [rbp-1110h]
  char v28[5]; // [rsp+CBh] [rbp-1105h]
  __int64 v29; // [rsp+D0h] [rbp-1100h]
  __int64 v30; // [rsp+D8h] [rbp-10F8h]
  __int16 v31; // [rsp+E0h] [rbp-10F0h]
  char v32; // [rsp+E2h] [rbp-10EEh]
  char a; // [rsp+F0h] [rbp-10E0h]
  char b; // [rsp+F1h] [rbp-10DFh]
  char c; // [rsp+F2h] [rbp-10DEh]
  char d; // [rsp+F3h] [rbp-10DDh]
  char e; // [rsp+F4h] [rbp-10DCh]
  char f; // [rsp+F5h] [rbp-10DBh]
  char g; // [rsp+F6h] [rbp-10DAh]
  char h; // [rsp+F7h] [rbp-10D9h]
  char i_1; // [rsp+F8h] [rbp-10D8h]
  char j_1; // [rsp+F9h] [rbp-10D7h]
  char k_1; // [rsp+FAh] [rbp-10D6h]
  char l; // [rsp+FBh] [rbp-10D5h]
  char m; // [rsp+FCh] [rbp-10D4h]
  char n; // [rsp+FDh] [rbp-10D3h]
  char o; // [rsp+FEh] [rbp-10D2h]
  char p; // [rsp+FFh] [rbp-10D1h]
  char q; // [rsp+100h] [rbp-10D0h]
  char r; // [rsp+101h] [rbp-10CFh]
  char s; // [rsp+102h] [rbp-10CEh]
  __int64 v52; // [rsp+110h] [rbp-10C0h]
  __int64 v53; // [rsp+118h] [rbp-10B8h]
  int v54; // [rsp+120h] [rbp-10B0h]
  __int16 v55; // [rsp+124h] [rbp-10ACh]
  char v56; // [rsp+126h] [rbp-10AAh]
  char v57; // [rsp+150h] [rbp-1080h]
  char s2; // [rsp+180h] [rbp-1050h]
  char hex_data[2048]; // [rsp+1B0h] [rbp-1020h]
  char v60[2056]; // [rsp+9B0h] [rbp-820h]
  unsigned __int64 v61; // [rsp+11B8h] [rbp-18h]

  v61 = __readfsqword(0x28u);
  v29 = 8672370769196829778LL;
  v30 = 7588358910211810867LL;
  v31 = 25210;
  v32 = 97;
  argv1_copy = (char *)malloc(0x40uLL);
  argv2_copy = (char *)malloc(0x40uLL);
  arg1_len = strlen(argv1);
  strncpy(argv1_copy, argv1, arg1_len);
  arg2_len = strlen(argv2);
  strncpy(argv2_copy, argv2, arg2_len);
  v52 = 5778228730180750659LL;
  v53 = 7166409442638329960LL;
  v54 = 1815509589;
  v55 = 21574;
  v56 = 79;
  sub_1D9E(0LL, 220206LL, 490509LL, 103LL, 105LL, 426840LL);
  if ( strlen(argv1) <= 0x3F )
  {
    argv1_len = strlen(argv1);
    sub_3196(argv1_copy, argv1_len, 0x40uLL);   // 73FF9B24EF8DE48C346D93FADCEE01151B0A1644BC81
  }
  if ( strlen(argv2) <= 0x3F )
  {
    argv2_len = strlen(argv2);
    sub_3196(argv2_copy, argv2_len, 0x40uLL);
  }
  v7 = argv2_copy[(signed int)sub_369D(argv2_copy)] - 48;
  v8 = v7 + *((_DWORD *)&off_8140 + argv2_copy[(signed int)sub_379A(argv2_copy)]) + 634;
  v27 = sub_303E(argv1_copy, argv1_copy) + v8;
  for ( i = 0LL; i <= 0x28E; ++i )
    *((_DWORD *)&off_8140 + i) += v27;
  v20 = 0LL;
  for ( j = 0LL; j <= 0x28E; ++j )
  {
    if ( !(*((_DWORD *)&off_8140 + j) % 23) )
      v20 += *((signed int *)&off_8140 + j);
    if ( !(*((_DWORD *)&off_8140 + j) % 300) )
      v20 -= *((signed int *)&off_8140 + j);
    if ( v20 < 0 )
      v20 = -v20;
  }
  v22 = *((signed int *)&off_8140 + 155LL - *argv2_copy);
  snprintf(hex_data, 0x13uLL, "%lu", v22, argv2);
  do_not_working(
    84LL,
    "DLA0HCMPwFyFaopsh6CPqidcwRhIFF",
    422813LL,
    "CqpBWJlEX5LCBpMetRNi490WgCQ9vh",
    289754LL,
    "iVBcMkB48CyFamOvypefXCG3lPpip2");
  a = 'A';
  b = 'B';
  c = 'C';
  d = 'D';
  e = 'E';
  f = 'F';
  g = 'G';
  h = 'H';
  i_1 = 'I';
  j_1 = 'J';
  k_1 = 'K';
  l = 'L';
  m = 'M';
  n = 'N';
  o = 'O';
  p = 'P';
  q = 'Q';
  r = 'R';
  s = 'S';
  counter = 0;
  v19 = v22;
  while ( v22 && counter <= 15 )
  {
    v22 = ((signed __int64)((unsigned __int128)(7378697629483820647LL * (signed __int128)v22) >> 64) >> 2) - (v22 >> 63);
    ++counter;
  }
  v23 = v19;
  while ( v23 && counter <= 15 )
  {
    v9 = v23
       - 10
       * (((signed __int64)((unsigned __int128)(7378697629483820647LL * (signed __int128)v23) >> 64) >> 2) - (v23 >> 63));
    v23 = ((signed __int64)((unsigned __int128)(7378697629483820647LL * (signed __int128)v23) >> 64) >> 2) - (v23 >> 63);
    v10 = counter++;
    hex_data[v10] = *(&a + v9);
  }
  while ( counter <= 15 )
  {
    v11 = counter++;
    hex_data[v11] = 97;
  }
  sub_2A4A(hex_data, v60, 16LL);
  sub_1384();
  for ( k = 5; k <= 8u; ++k )
    v28[k - 5] = v60[k];
  v12 = strlen(v28);
  MD5(v28, v12, &v57);
  sub_2A4A(&v57, &s2, 16LL);
  sub_1384();
  v13 = strlen(s1);
  if ( !strncmp(s1, &s2, v13) )
  {
    v17 = 3;
    v18 = -1337;
    sub_33F2(hex_data, &s2);
  }
  else
  {
    v17 = 4;
    v18 = -101;
  }
  free(argv1_copy);
  free(argv2_copy);
  sub_2EB3(v17, v18);
}
```

분석 진행중 ----
