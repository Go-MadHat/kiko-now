---
layout: post
title: ! "[Reversing.kr] Music Player"
excerpt_separator: <!--more-->
comments : true
tags:
  - Vertex
  - Write-up
  - Reversing.kr
---

![](https://lh3.googleusercontent.com/-Mt00QAWaO-k/W2B3xJuz2oI/AAAAAAAABPA/EuQssWCqiIoS-kW0ZX7LwJltNjWTZJnvwCHMYCw/s0/Music_Player_2018-07-31_23-52-45.png)


<!--more-->

```

This MP3 Player is limited to 1 minutes.
You have to play more than one minute.

There are exist several 1-minute-check-routine.
After bypassing every check routine, you will see the perfect flag.

```
### 1. 뭐하는 프로그램일까?


![](https://lh3.googleusercontent.com/-m_KjTGlMmCw/W2BOd47MUnI/AAAAAAAABKY/DddJCpqQO_0wkGH4Qs0XYazFrDyRPModQCHMYCw/s0/Music_Player_2018-07-31_20-56-39.png)

실행 한 뒤에 아무 mp3 파일을 열면 노래가 재생되기 시작합니다.

가지고 있던 mp3 파일을 넣었더니 재생이 안되서 구글에서 적절하게 잘 재생되는 mp3 파일을 찾았습니다.


노래가 재생되다가 1분이 되면,

![](https://lh3.googleusercontent.com/-fQ6lyRDjddg/W2BQvhfV3dI/AAAAAAAABLk/wOHnDbbQkmUZEEtIQPCEz8XqV6U_Oa9ygCHMYCw/s0/haroopad_2018-07-31_21-06-22.png)

라고 메세지박스가 뜹니다.


### 2. 1분 체크하는 부분을 찾아보자.

![](https://lh3.googleusercontent.com/-wh_I2GICV4E/W2BRDuKU2iI/AAAAAAAABL0/TZxwZiAxxzwmRFpZIG-fvRfXhuHcJ0eUgCHMYCw/s0/2018-07-31_21-07-43.png)
Msgbox뜨죠? 
<br>


![](https://lh3.googleusercontent.com/-TjezzzNx46A/W2BRgud_MjI/AAAAAAAABMI/tgbJ8hssdlYoyGCzEH64ygyWsmtVK5XwACHMYCw/s0/x32dbg_2018-07-31_21-09-38.png)
<br>
Msgbox 호출하는부분 4개밖에 없죠?

다 브포 걸고서 1분을 넘기려고 하면, 1분 체크하는 프로시져에서 멈출것입니다.

<br>
![](https://lh3.googleusercontent.com/-Se1ArDlfzCc/W2BlL87xzLI/AAAAAAAABMY/TvFFW34knUczSsJFPhCk_-3MXfGjGSTNgCHMYCw/s0/x32dbg_2018-07-31_22-33-30.png)
쨘

<br>
![](https://lh3.googleusercontent.com/-RiK9Fo-0M9w/W2Blo6orFOI/AAAAAAAABMk/-U0JanoetwImJ0FRfVKGzstqIoXdxsCkwCHMYCw/s0/x32dbg_2018-07-31_22-35-17.png)
스크롤 좀 올려서 

함수의 시작점을 찾아줍시다.
```
push ebp
mov ebp, esp  ;어셈에서 함수가 시작할때 보통 이렇게 생겼어요!
```

저기에 브포 걸어주고 마구 실행해볼게요.



![](https://lh3.googleusercontent.com/-owZnGwrixL0/W2BnqCDNvhI/AAAAAAAABNI/_0Z5J2lHOEgBgrh65tqEneT5kS2OR7iGACHMYCw/s0/2018-07-31_22-43-45.gif)
아... 컴퓨터의 소리까지 전달하지 못하는게 아쉬운데요.
노래가 1초만 잠깐 들리다가 멈춥니다.
아마 VB6.0의 Interval 1000짜리 타이머로 1초마다 WMP(window media player)의 진행상황을 가져와서
SlideBar의 value를 설정해주는거겠죠!

그러면 60초가 넘었을때 강제로 분기문을 수정해서
그대로 재생되게 하면 어떻게 될까요?

<br>

![](https://lh3.googleusercontent.com/-BJsi9040ook/W2BpbfsXbwI/AAAAAAAABNc/V-XwaBbZPOYLO3VKkrtWTEvNIC7O220mQCHMYCw/s0/x32dbg_2018-07-31_22-50-20.png)

현재 멈춰있는 
```
jl music_player.4045FE
```
는 현재 cmp로 비교하고 있는 대상이 EA60보다 작으면 점프하고 있죠?


![](https://lh3.googleusercontent.com/-7P_6YFiK5OI/W2Bp9WR41qI/AAAAAAAABNk/-SrWh7L9Yq8sKtp82Vul9I2F-jG6PCKiACHMYCw/s0/haroopad_2018-07-31_22-53-37.png)
캬... EA60을 10진수로 바꾸면 60000이네요!

타이머의 인터벌이 1000ms(밀리세컨드)라고 아까 예상했으니까 60000이면 60초라고 추정할 수 있네요!

저 EA60을 더 높여서 FFFF정도로 바꿔볼게요.


![](https://lh3.googleusercontent.com/-qEebShI_crM/W2BqenIlqDI/AAAAAAAABNs/TD6nUXw5E5oaJneEAFLkp1cPvU6PbwnVQCHMYCw/s0/x32dbg_2018-07-31_22-56-04.png)

<br>
<br>
그리고 맨 위에 걸어둔 브레이크포인트 제거`(1초마다 계속 브레이크 걸리니까_)`하고 프로그램을 재게하면?...?


![](https://lh3.googleusercontent.com/-c2358ZtcuvU/W2BrbJvcFWI/AAAAAAAABN8/efSV-i7Wvv88ALdIT81PU5LSsYyRigoUACHMYCw/s0/x32dbg_2018-07-31_22-59-58.png)
프로그램이 멈추고 뭔가 Exceeption이 Raise됩니다...

왜 이런 오류가 뜨냐면, Vb6.0에서 SlideBar 라는 컨트롤은 MaxValue와 Value라는 속성으로
저 슬라이드 가운데 버튼의 위치를 표시하는데,
저 위치를 백분률로 표시하거든요.

그래서 MaxValue보다 Value값이 크면 이렇게 오류가 뜹니다.

그리고 프로그램이 꺼져요(;;)

그러면 분기문을 따라가면서 SlideBar에 Value를 적용시켜주는 파트를 그냥 NOP으로 채워버리면 어떨까요?

제가 어셈에서 VB6컨트롤에 어떻게 접근하는지는 모르지만,

그냥 F8 누르다가 뻑나는부분이 Value에 값을 넣을때겠죠?


![](https://lh3.googleusercontent.com/-sSwsvCVevqo/W2Bs6bNUVHI/AAAAAAAABOM/j62yA-Fm_KA-GCZv74KirWGotCvsRuQcQCHMYCw/s0/x32dbg_2018-07-31_23-06-19.png)

그냥 jl밑에 60초가 지나지 않았을 때 점프하는 어드레스인 4045FE로 점프시킵니다.
`jmp 0x004045FE` 라고 적어서 말이죠!

저기에 브레이크 포인트 걸고 기다리면 구태여 1초마다 브레이크 걸리면서 귀찮게 지내지 않아도 좋을것같아요.

좋은 노래 들으면서 60초가 다 재생되면 저렇게 브레이크 포인트가 잡힙니다!

F8 연타하면 쭉 달리는데 주의할점은 스택부분을 잘 봐주세요!
특히 컨트롤의 어떤 속성을 집어넣는것 또한 무슨 함수를 call하는데 
call하는 시점의 현재 스택 위치를 잘 기억하면서 f8 연타하시길 바랍니다.


![](https://lh3.googleusercontent.com/-Jf6lFb07WWY/W2B1BTV2AyI/AAAAAAAABOk/ALapoda6viU9MRuINuw7-8FdwnYl9HGEwCHMYCw/s0/2018-07-31_23-34-52.gif)


<br>
<br>
![](https://lh3.googleusercontent.com/-BKUBp0TSyhM/W2B2CAWd_LI/AAAAAAAABOo/ncg9MOLGPwAw3By0NnsGS3aLSOAwOxlnQCHMYCw/s0/x32dbg_2018-07-31_23-45-19.png)

브레이크포인트 건 곳으로 다시가서 그냥 jmp로 바꾸고 실행해버리세요.

그러면 SlideBar에 값을 쓰지 않으니 프로그램이 런타임에러를 뿜지도 않고 잘 실행됩니다.

![](https://lh3.googleusercontent.com/-JkqLisduIOQ/W2B2h83BM3I/AAAAAAAABOw/4Dt5twK0rUQnO-rnIrfYYxSL8GvuvB2xACHMYCw/s0/x32dbg_2018-07-31_23-46-52.png)

Form의 caption에 플래그가 뜹니다.
