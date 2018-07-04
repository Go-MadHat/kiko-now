---
layout: post
title: ! "[0ctf2017] babyheap Write up"
excerpt_separator: <!--more-->
comments : true
tags:
  - Write-up
  - CTF
  - 0ctf2017
  - chaem
  - heap
---

안녕하세요 `chaem`입니다!!!!ㅎㅎ  
제가 이번에 풀어본 문제는 0ctf에서 2017년에 출제된 `pwnable`문제입니다. 바로 `babyheap`이라는 heap 문제인데요!!  
요 문제는 shellpish형님들의 `how2heap` 중에서 `fastbin_dup_into_stack` 공격 예시이기도 합니다용ㅎㅎ heap을 공부하다가 첫 언덕을 넘기위해 이 문제에 도전하였숩니당ㅠㅠ  
알 것 같지만 모르는 것 같은 상태를 계속 무한반복 ㅠㅠㅠㅠ  
그럼 이제 눈물나는 문제 풀이를 시작해볼게용 고고씽!!  

<!--more-->

### 바이너리 분석  
우선 바이너리에 어떤 보호기법이 걸려있는지 보도록하겠습니당!  

```  
pwndbg> checksec
[*] Checking for new versions of pwntools
    To disable this functionality, set the contents of /root/.pwntools-cache/update to 'never'.
[*] You have the latest version of Pwntools (3.12.0)
[*] '/home/ubuntu/0ctf/0ctfbabyheap'
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      PIE enabled
```  

우와 바이너리가 모두 걸려있네염(저 사실 바이너리가 모두 걸려있는 문제는 처음이에효 후덜덜)  
우선 Full RELRO 이므로, exploit할 GOT는 못쓰고 hooking fuction pointer로 `__malloc_hook`을 사용할 수 있습니다.  
이 주소를 알아내기위해서는 leak을 해야하는데, 그러려면 fastbin 뿐만이 아니라 smallbin까지 이용해야 합니다.  
fastbin은 single linked list로 구현되어 단방향으로만 진행하므로 fd를 조작해서 leak을 해야하고!  
smallbin으로 할당했을 때 unsorted bin에 할당되는 것과 free하면 main_arena+88주소가 들어가는 것을 이용해야합니다.  

바이너리가 어떤 기능을 하고있는지 먼저 봅시다!  
바이너리를 실행하면 5개의 보기가 있습니다.  
ida를 사용해서 각 함수에서 어떤 실행을 하는지 알아봅시다.  

#### 1. Allocate  

```  
===== Baby Heap in 2017 =====
1. Allocate
2. Fill
3. Free
4. Dump
5. Exit
Command: 1
Size: 72
Allocate Index 0
```  
 
```  
void __fastcall sub_D48(__int64 a1)
{
  signed int i; // [sp+10h] [bp-10h]@1
  int v2; // [sp+14h] [bp-Ch]@3
  void *v3; // [sp+18h] [bp-8h]@6

  for ( i = 0; i <= 15; ++i )
  {
    if ( !*(_DWORD *)(24LL * i + a1) )
    {
      printf("Size: ");
      v2 = sub_138C();
      if ( v2 > 0 )
      {
        if ( v2 > 4096 )
          v2 = 4096;
        v3 = calloc(v2, 1uLL);
        if ( !v3 )
          exit(-1);
        *(_DWORD *)(24LL * i + a1) = 1;
        *(_QWORD *)(a1 + 24LL * i + 8) = v2;
        *(_QWORD *)(a1 + 24LL * i + 16) = v3;
        printf("Allocate Index %d\n", (unsigned int)i);
      }
      return;
    }
  }
}
```  
1번을 실행하면 size를 입력받아 그 크기 만큼 calloc을 해줍니다.  
malloc은 많이 사용하지만 calloc은 좀 생소할 수 있는데요,  
calloc은 malloc의 역할 + 할당된 공간의 값을 모두 0으로 초기화해주는 기능을 포함하고 있습니다.  
배열을 할당하고 모두 0으로 초기화할 필요가 있을경우에는 calloc을 쓰면 더 편할 수 있겠죠!  
```
[chaem이랑 짚고가기]
void *malloc (size_t size)
size 바이트의 한 블럭을 할당한다.

void *calloc (size_t count, size_t eltsize)
malloc을 사용하여 count * eltsize 바이트의 블럭을 할당하여 그 내용을 제로로 채운다.
```  

#### 2. Fill  

```  
1. Allocate
2. Fill
3. Free
4. Dump
5. Exit
Command: 2
Index: 0
Size: 10
Content: aaaaaaaaaa
```  

```  
int __fastcall sub_E7F(__int64 a1)
{
  int result; // eax@1
  int v2; // [sp+18h] [bp-8h]@1
  int v3; // [sp+1Ch] [bp-4h]@4

  printf("Index: ");
  result = sub_138C();
  v2 = result;
  if ( result >= 0 && result <= 15 )
  {
    result = *(_DWORD *)(24LL * result + a1);
    if ( result == 1 )
    {
      printf("Size: ");
      result = sub_138C();
      v3 = result;
      if ( result > 0 )
      {
        printf("Content: ");
        result = sub_11B2(*(_QWORD *)(24LL * v2 + a1 + 16), v3);
      }
    }
  }
  return result;
}
```  

2번을 실행하면 1번에서 할당한 공간의 index로 접근하여, 그 주소에 데이터를 쓸 수 있습니다. 그런데 heap chunk의 크기를 체크하지 않고 힙에 데이터를 쓰기때문에 여기서 heap overflow가 발생합니다. 즉 Fill에서 chunk의 size보다 크게 size를 지정할 수 있기 때문에 chunk의 fd를 덮어쓸 수 있고, 이를 이용해 원하는 곳에 chunk를 만들어 반환하도록 할 수 있습니다. (fastbin attack이라고 함)  

#### 3. free  

```  
1. Allocate
2. Fill
3. Free
4. Dump
5. Exit
Command: 3
Index: 1
```  

```  
int __fastcall sub_F50(__int64 a1)
{
  signed __int64 v1; // rax@1
  int v3; // [sp+1Ch] [bp-4h]@1

  printf("Index: ");
  LODWORD(v1) = sub_138C();
  v3 = v1;
  if ( (signed int)v1 >= 0 && (signed int)v1 <= 15 )
  {
    LODWORD(v1) = *(_DWORD *)(24LL * (signed int)v1 + a1);
    if ( (_DWORD)v1 == 1 )
    {
      *(_DWORD *)(24LL * v3 + a1) = 0;
      *(_QWORD *)(24LL * v3 + a1 + 8) = 0LL;
      free(*(void **)(24LL * v3 + a1 + 16));
      v1 = 24LL * v3 + a1;
      *(_QWORD *)(v1 + 16) = 0LL;
    }
  }
  return v1;
}
```  

3번을 실행하면 입력한 index에 있는 공간을 free합니다.  

#### 4. Dump  

```  
1. Allocate
2. Fill
3. Free
4. Dump
5. Exit
Command: 4
Index: 0
Content:
aaaaaaaaaa
```  

```  
int __fastcall sub_1051(__int64 a1)
{
  int result; // eax@1
  int v2; // [sp+1Ch] [bp-4h]@1

  printf("Index: ");
  result = sub_138C();
  v2 = result;
  if ( result >= 0 && result <= 15 )
  {
    result = *(_DWORD *)(24LL * result + a1);
    if ( result == 1 )
    {
      puts("Content: ");
      sub_130F(*(_QWORD *)(24LL * v2 + a1 + 16), *(_QWORD *)(24LL * v2 + a1 + 8));
      result = puts(byte_14F1);
    }
  }
  return result;
}
```  

4번은 원하는 인덱스에 있는 값을 출력해주는 역할을 합니다.  

#### 5. Exit  
프로그램을 종료합니다.  

### leak  
바이너리를 이것저것 실행시키다 보면, fill에서 처음 할당한 chunk 크기보다 크게 사이즈를 입력하고 contents를 넣으면 어떻게 될지 궁금증이 생길 것입니다. 그래서 32바이트로 chunk를 4개 할당하고 fastbin이 제대로 생성하는지 확인하였습니다. 확인 후에, fill에서 사이즈 값을 48로 변경하고 a를 48개 입력하여 heap을 확인해 보았습니다.  

```  
pwndbg> heap
Top Chunk: 0x555555757150
Last Remainder: 0

0x555555757000 FASTBIN {
  prev_size = 0,
  size = 49,
  fd = 0x6161616161616161,
  bk = 0x6161616161616161,
  fd_nextsize = 0x6161616161616161,
  bk_nextsize = 0x6161616161616161
}
0x555555757030 PREV_INUSE {
  prev_size = 7016996765293437281,
  size = 7016996765293437281,
  fd = 0x0,
  bk = 0x0,
  fd_nextsize = 0x0,
  bk_nextsize = 0x0
}
```  

그랬더니 조작한 fastbin 외의 다른 fastbin의 정보가 다 사라졌습니다. 그리고 이상한 값들이 들어간 것을 확인할 수 있었습니다. 아마도 위에서 a로 overflow시키면서 이상한 값들이 들어간 것 같습니다.  

```  
pwndbg> x/10gx 0x555555757000
0x555555757000:	0x0000000000000000	0x0000000000000031
0x555555757010:	0x6161616161616161	0x6161616161616161
0x555555757020:	0x6161616161616161	0x6161616161616161
0x555555757030:	0x6161616161616161	0x6161616161616161
0x555555757040:	0x0000000000000000	0x0000000000000000
```
chunk의 값을 확인해보면, 두번째 fastbin의 주소인 0x555555757030에 overflow된 값들이 들어가면서 값이 변경된 것을 알 수 있습니다. 이를 통해 overflow를 통해 다음 heap영역의 값을 변경시킬 수 있다는 사실을 알 수 있었습니다. 하지만 여기서 크기를 조작하여 그 수만큼 받을 수 있다하더라도 dump를 통해서 출력할 수 있는 것 역시 48바이트 뿐입니다. 따라서 leak을 하기 위한 값을 읽어오기 위해서는 같은 크기의 fastbin만을 이용해서는 안되고 smallbin을 하나 이용해야 합니다.  
다음은 leak하는 과정을 정리한 것입니다.  

#### 1. 따라서 4개의 fastbin과 1개의 smallbin 할당합니다. (index 0~4)  

```  
pwndbg> heap
Top Chunk: 0x555555757150
Last Remainder: 0

0x555555757000 FASTBIN {
  prev_size = 0,
  size = 49,
  fd = 0x0,
  bk = 0x0,
  fd_nextsize = 0x0,
  bk_nextsize = 0x0
}
0x555555757030 FASTBIN {
  prev_size = 0,
  size = 49,
  fd = 0x0,
  bk = 0x0,
  fd_nextsize = 0x0,
  bk_nextsize = 0x0
}
0x555555757060 FASTBIN {
  prev_size = 0,
  size = 49,
  fd = 0x0,
  bk = 0x0,
  fd_nextsize = 0x0,
  bk_nextsize = 0x0
}
0x555555757090 FASTBIN {
  prev_size = 0,
  size = 49,
  fd = 0x0,
  bk = 0x0,
  fd_nextsize = 0x0,
  bk_nextsize = 0x0
}
0x5555557570c0 PREV_INUSE {
  prev_size = 0,
  size = 145,
  fd = 0x0,
  bk = 0x0,
  fd_nextsize = 0x0,
  bk_nextsize = 0x0
}
0x555555757150 PREV_INUSE {
  prev_size = 0,
  size = 134833,
  fd = 0x0,
  bk = 0x0,
  fd_nextsize = 0x0,
  bk_nextsize = 0x0
}
```  

#### 2. 1번과 2번 chunk를 free합니다. : free한 2번 chunk의 fd에 이전 1번 chunk의 주소가 들어있는 것을 볼 수 있습니다.  

```  
pwndbg> heap
Top Chunk: 0x555555757150
Last Remainder: 0

0x555555757000 FASTBIN {
  prev_size = 0,
  size = 49,
  fd = 0x0,
  bk = 0x0,
  fd_nextsize = 0x0,
  bk_nextsize = 0x0
}
0x555555757030 FASTBIN {
  prev_size = 0,
  size = 49,
  fd = 0x0,
  bk = 0x0,
  fd_nextsize = 0x0,
  bk_nextsize = 0x0
}
0x555555757060 FASTBIN {
  prev_size = 0,
  size = 49,
  fd = 0x555555757030,
  bk = 0x0,
  fd_nextsize = 0x0,
  bk_nextsize = 0x0
}
0x555555757090 FASTBIN {
  prev_size = 0,
  size = 49,
  fd = 0x0,
  bk = 0x0,
  fd_nextsize = 0x0,
  bk_nextsize = 0x0
}
0x5555557570c0 PREV_INUSE {
  prev_size = 0,
  size = 145,
  fd = 0x0,
  bk = 0x0,
  fd_nextsize = 0x0,
  bk_nextsize = 0x0
}
0x555555757150 PREV_INUSE {
  prev_size = 0,
  size = 134833,
  fd = 0x0,
  bk = 0x0,
  fd_nextsize = 0x0,
  bk_nextsize = 0x0
}
```  

#### 3. free한 2번 chunk의 fd를 4번 smallbin chunk를 가리키도록 조작합니다 : overflow를 이용해 1byte만 조작해 주면됩니다.  
 
```  
pwndbg> x/10gx 0x555555757060
0x555555757060:	0x0000000000000000	0x0000000000000031
0x555555757070:	0x00005555557570c0	0x0000000000000000
0x555555757080:	0x0000000000000000	0x0000000000000000
0x555555757090:	0x0000000000000000	0x0000000000000031
0x5555557570a0:	0x0000000000000000	0x0000000000000000
```  

#### 4. 2번 chunk에서 overflow를 일으켜 4번 smallbin을 fastbin chunk 크기로 만들어 줍니다. : security check가 이루어질 때 size를 체크하므로 크기를 조작해줘야 합니다!  

#### 5. 1번과 2번 chunk를 다시 allocate합니다. : 원래 1번을 가리키던 2번 chunk의 fd가 4번 chunk를 가리키게되고, allocate하면서 4번 chunk랑 병합하게 됩니다!  

#### 6. 그리고 4번 chunk의 크기를 다시 smallbin 크기로 만들어주고 free합니다. free하면 smallbin 특성상 unsorted bin이 되기 때문에 fd와 bk에 main_arena+88 주소가 남게 됩니다. 이 점을 이용하면 dump를 통해 main_arena+88의 leak이 가능합니다.  

```  
[chaem이랑 짚고가기]
unsorted bin : small 또는 large chunk는 각각의 bin에서 해제되는 경우, unsorted bin에 추가된다. 최근에 해제된 청크를 재사용할 수 있도록 하기위해서 이다. 따라서, 적절한 bin을 찾는데 걸리는 시간이 적기 때문에 메모리 할당과 반납의 비트 처리 속도가 빠르다.
```  

이 문제에서는 또한, stack의 주소를 leak할 수 있는 장치를 주지 않았기 때문에 또 다른 주소를 활용해야하는데요, 이때 자주 이용하는 것이 malloc_hook이라고 합니다. leak뒷부분을 너무 빠르게 지나갔는데요 ㅠㅠ ㅎㅎ  
leak 뒷부분과 malloc_hook 부분 부터는 다음에 이어서 진행하도록 하겠습니당! 담에보아요!! 수고하셨습니당!  
