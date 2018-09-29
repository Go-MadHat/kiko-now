---
layout: post
title: ! "MIPS 시작하기"
excerpt_separator: <!--more-->
comments : true
tags:
  - MIPS
  - RISC
  - chaem
---
  
오늘은 예전에 내가 잠시 공부했던 mips를 꺼내볼까 합니다  
MIPS는 임베디드... 공유기... 임베디드... 뭐 이런데 쓰인다고 알고있는데요   

<!--more-->

## MIPS란 무엇일까?
MIPS(Microprocessor without Interlocked Pipeline Stages)란 MIPS Technologies에서 개발한 RISC 기반의 명령어 집합 체계이다. MIPS 명령어 체계는 굉장히 깔끔하게 설계되어있기때문에 많은 대학교의 컴퓨터 아키텍처 강좌과목에서 가르치고 있다.  
by 나무위키 (출처 : https://namu.wiki/w/MIPS)

처음부터 동공지진이 일어나네여 ㅇㅁㅇ...

여기서 잠깐!!!
RISC기반이라는게 무슨 뜻인지 아시나요?
CISC와 RISC의 차이에 대해 알아두면 좋을 것 같아 추가 설명!!
함 알아보고 넘어갑시당!!

## [잠깐! 보고가기] CISC vs RISC

CISC는‘COMPLEX INSTRUCTION SET COMPUTER’의 약어이고, 연산에 처리되는 복잡한 명령어들을 수백 개 이상 탑재하고 있는 프로세서입니다. CISC 아키텍처는 MCU의 성능을 향상시키기 위해 복합 명령어를 이용하는 것이 특징입니다.

RISC는 ‘REDUCED INSTRUCTION SET COMPUTER’의 약어이며, 하나의 명령어 실행으로 간단한 프로세스들을 매우 신속하게 수행합니다. RISC 아키텍처는 다수의 축소 명령어들을 신속하게 실행하여 전반적인 MCU 성능을 향상시키는 것이 특징이라 할 수 있습니다.

>![]({{ site.baseurl }}/images/chaem/mips/mips_01.PNG)  
(출처 : http://www.semiconnet.co.kr/ms_pdf/667_20151229134857_201502_st.pdf)

---

자, 그럼 이제 마음을 가다듬고 차근차근 다음 3가지를 위주로 MIPS에 대해 알아봅시당!
- MIPS 레지스터 종류
- MIPS 함수호출 규약
- MIPS 명령어 종류

우선, MIPS는 32비트 기반의 RISC 방식이며, 아래 표와 같이 총 32개의 레지스터로 이루어져 있습니다.

MIPS 레지스터의 호출 규약은 일반적으로 사용되는 O32 ABI와 N64/N32 ABI로 나눠지는데, O32 ABI는 32bit CPU를 위한 레지스터 호출 규약이며, N64/N32 ABI는 64bit CPU를 위한 레지스터 호출규약입니다. 기본적인 레지스터 호출 규약은 아래의 표로 슝슝!

>![]({{ site.baseurl }}/images/chaem/mips/mips_02.png)  


표에서 보면 name과 number 두가지로 나눠져 있는데, 실제 어셈 코드에서는 둘 중 어떤 것을 사용해도 상관 없다는 것! 우리가 첫번째 시리얼 포트를 COM1이라고 하는 것과 마찬가지로, name은 레지스터의 용도에 맞게 이름을 붙여서 더 편리하게 사용하기 위한 것이라고 생각하면 될 것 같아요!

그렇다면 좀 더 재밌게(?) 알아보기 위해 Hello World로 예를 들어봅시다!

---

일단, MIPS를 사용할 MIPS Cross Compile 환경을 구성해 볼가요?

## MIPS Cross Compile 환경 구성하기

1. /etc/apt/sources.list – 요기 위치에 다음 2줄 추가!!  

	- deb http://ftp.de.debian.org/debian squeeze main  
	- deb http://www.emdebian.org/debian/ squeeze main  

>![]({{ site.baseurl }}/images/chaem/mips/mips_03.png)  

2. 다음 명령어 실행!  

	- sudo apt-get update  
	- sudo apt-get install emdebian-archive-keyring  
	- apt-get install linux-libc-dev-mips-cross libc6-mips-cross libc6-dev-mips-cross binutils-mips-linux-gnu gcc-4.4-mips-linux-gnu g++-4.4-mips-linux-gnu  

3. mips-linux-gcc 가 설치되어있는지 쳌쳌쳌!!! 해봅시당  
	- Sudo apt install gcc-mips-linux-gnu  
  - mips-linux-gnu-gcc -dumpmachine  

이러한 과정을 통해 MIPS Cross Compile을 할 수 있는 환경 완성!! 두둥!!!  
환경을 무사히 구축하셨나용?! (완전 새 브엠이라면 부가적으로 설치할 게 좀 많을 수도...하하)  

---

## MIPS로 Hello World!  

그렇다면 본격적으로 Hello World 프로그램을 MIPS 아키텍처에서 리버싱해 봅시당!!!!  

여기서 우리가 정복할 명령어는!!!!  

1. 저장에 쓰이는 sw  
2. 로드에 쓰이는 lw  
3. 상수 저장에 쓰이는 lui, li  
4. 점프에 쓰이는 j, jr, jal, jalr  
5. addiu, move  

이렇게 5개를 기준으로 차근차근 보는 것이에여!!!!  

자 그럼,  

>![]({{ site.baseurl }}/images/chaem/mips/mips_04.png)  

간단한 Hello World 코드를 짜고 컴파일해서 IDA로 열어보면,  

>![]({{ site.baseurl }}/images/chaem/mips/mips_05.png)  

짜잔!!! 이게 MIPS에요 여러분 ㅎㅎ (개봉 시, 32bit로, 우선 왼쪽 명령어 주소들이 4바이트 단위로 떨어지는 것으로 보아 MIPS는 RISC 구조인 것을 알 수 있어요!!  

---  

>![]({{ site.baseurl }}/images/chaem/mips/mips_06.png)  

우선 코드에서 첫 번째 라인을 보면 **addiu $sp, -0x20**가 있죠? 그렇다면 addiu를 한번 볼가요?  
addiu(add immediate unsigned)는 "addiu $a, $b, 상수" 형태로 쓰이며 $b 에 상수를 더한 값을 $a 에 저장한다는 의미를 가집니다. 예를 들어, addiu $s1, $s2, 100 => $s1 = $s2 +100 라고 할 수 있는 거죠!  
따라서 addiu $sp, -0x20는 sp = sp -0x20 정도로 표현할 수 있죠!  

sp는 느낌적으로 stack pointer라는 느낌이 파바박!! sp는 가장 최근에 스택에 할당된 주소를 가리키는 값으로, 레지스터가 스필될 장소 또는 레지스터의 옛날 값을 찾을 수 있는 장소를 표시해요!  
따라서 이 한 줄은 스택의 크기를 32바이트만큼 증가시키는, x86의 구문과 대비하면 sub esp, -0x20 와 같은 역할을 하는 것!!! 즉, 이 만큼의 스택공간을 마련한다고 생각하면 될 것 같아요!  
<br>
두 번째 라인은 **sw $ra, 0x20+var_4($sp)** 이죠. 하나씩 잘라서 봐봅시당  
sw (stored word) : 레지스터의 값을 메모리에 word 사이즈만큼 (4byte) 저장하는 명령어!  
여기서 주의 깊에 봐야할 부분은 `sw source_register, destination_memory`로 표현된다는 것!!!  
<br>
ra(return address)는 말그대로 리턴 주소를 가지고 있는 레지스터입니당 x86에서 함수 call 전에 ret을 스택에 push 했던 것과 유사하죠!!  
<br>
그렇다면 0x20+var_4($sp)란 표현은 sp + 0x20 + var_4를 뜻하고  
$sp로 부터 0x20+var_4 바이트 떨어진 곳에 $ra를 저장한다는 의미!! got it???  
<br>
세 번째 라인 **sw $fp, 0x20+var_8($sp)**를 봅시다! 여기서 $fp는 프레임포인터(프로시져의 저장된 레지스터와 지역 변수의 위치를 표시하는 값)의 약자! 즉, $fp의 값을 0x20+var_8($sp)에 저장하라는 의미랍니다.  
<br>
네 번째 라인에서 **move $fp, $sp**를 보면, fp에 sp의 주소를 넣는다는 의미! 즉, mov ebp, esp와 같은 의미를 담은 명령어 입니다.  

---

>![]({{ site.baseurl }}/assets/images/mips/mips_07.png)  

자자, 이어서 아직 보지못한 명령어 위주로 쭉 봐봅시당~  
**li $gp, 0x419010**  
$gp는 전역 포인터(global pointer)로 정적 데이터를 가리키는데 사용하도록 예약된 레지스터이며, li 명령어는 $gp에 0x419010이라는 상수를 로드한다는 것을 의미합니다~  
<br>
**lui $v0, 0x40**  
lui 연산은 해당 레지스터의 상위 두바이트에 값을 로드하는 명령어인데요, 따라서 v0 = 0x00400000와 같이 표현할 수 있을 것입니다.  

---
>![]({{ site.baseurl }}/images/chaem/mips/mips_08.png)  

이제 얼마남지 않았어요 여러분!!! 조금만 더 힘을냅시다!!!  
<br>
**la $v0, puts**  
la 명령어는 load address를 의미하는 명령어 같은데요, v0에 puts의 주소 값을 입력 받는다고 볼 수 있을 것 같네요!!!  
<br>
**jalr $t9 ; puts**  
요건 비슷하게 생긴 것들이 몇 개 있는데 자세한 비교는 아래에서 정리해보기로하고!! 일단 jalr라는 명령어는 jump and link register를 뜻하며 $t9 레지스터로 점프하고 다음 명령어의 주소를 $ra에 저장한다는 의미입니다!!!  
<br>
참고로, 중간중간에 있는 nop은 아무것도 하지 않고 지연 슬롯 역할을 하는 명령어랍니당!  
<br>
**lw $gp, 0x20+var_10($fp)**  
load word 명령어로 레지스터를 Destination으로 메모리에서 값을 로드해오는 명령어입니당  
따라서 gp = fp +0x20 +var_10 이 될 것이에요.  
<br>
**jr $ra**  
요 jr 명령어는 아까 jalr 명령어와는 살짝 다르게 $ra 로 점프하여 돌아오지 않는... 명령어랍니다!!!  
<br>
이상으로 Hello World 프로그램에 나오는 명령어 완전(?) 정복!!!  
MIPS 명령어를 이정도 파악하였으면 무적이라고 믿고...  
이만 안녕!!! 하기전에!  
<br>
여러분이 아쉬워할까봐 마지막으로 정리함 하고 갑시당ㅎㅎ  
잊어버리면 안되니까ㅎㅎ  
<br>
## 정리한번합시다  

1. 저장에 쓰이는 sw를 익혔다.  
 > sw $a, 36(sp) : Store word; $sp로부터 36 바이트 떨어진 곳에 $a 에 저장

2. 로드에 쓰이는 lw를 익혔다.  
 > lw $a, 36(sp) : Load word; $sp로부터 36 바이트 떨어진 곳의 값을 $a에 로드

3. 상수 저장에 쓰이는 li, lui를 익혔고 차이점을 파악했다.  
 > li $a, 상수 : load immediate; $a에 상수를 로드
 > lui $v0, 상수 : load upper immediate; 상수를 16비트만큼 왼쪽으로 시프트한 후에 $v0에 저장 (시프트 후 오른쪽 16비트는 0으로 채워짐

4. 점프에 쓰이는 j, jr, jal, jalr 을 익혔고 차이점을 파악했다.  
 > j 주소 : jump; 해당 주소로 점프
 > jr 레지스터 : jump register; 해당 레지스터로 점프
 > jal 주소 : jump and link; 지정된 주소로 점프하고 다음 명령어의 주소를 리턴주소 $ra에 저장
 > jalr 레지스터 : jump and link register; 지정된 레지스터로 점프하고 다음 명령어의 주소를 리턴주소 $ra에 저장

5. addiu, move 를 익혔다.  
 > addiu $a, $b, 상수 : add immediate unsigned; $b 에 상수를 더한 값을 $a 에 저장
 > move $a, $b : $b 의 내용을 $a 로 이동
