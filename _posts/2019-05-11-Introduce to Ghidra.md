---
layout: post
title: IDA는 너무 비싸, Ghidra를 써봐요.
excerpt_separator: <!--more-->
tags:
  - Reversing Tools
  - Ghidra
  - Vertex
---

오늘은 Ghidra에 대해 소개 해 드리겠습니다.

<!--more-->

# Ghidra 소개

![](https://i.imgur.com/mtaYPBN.png)

Ghidra(:기드라)는 NSA에서 개발한 디스어셈블러 프레임워크입니다. RSA컨퍼런스에서 오픈소스로 공개되었고, 자바로 개발됐습니다.

IDA Pro + Hexray의 조합은 매우 편리하고 기능도 다양하지만, 정식 라이센스 가격이 비싸서 선뜻 사용해보기 어려우셨던 분들은 한번 사용해보시길 바랍니다.

기드라의 특징으로는 Windows, Mac, Linux같이 모든 플랫폼에서 실행할 수 있고, 다양한 플랫폼을 위해 컴파일된 코드들도 분석할 수 있게 해줍니다(x86, x64, Arm, Mips, APK, 등등...).

기능으로는 간략하게 디스어셈블, **디컴파일**(hexray와 비슷), 코드 그래프 시각화, 스크립팅이 있으며 그 외에도 수백가지의 기능이 있다고 공식문서에서 소개합니다.



사용해본 첫 인상으로는 기본적으로 기드라가 해주는 정적 분석이 정말 편리했습니다. 코드를 불러오면서 여러 정적 분석을 시킬 수 있는데, 기본적으로 함수 그래프를 그려주는 기능과, 디컴파일기능, 리버싱하면서 특히 어려웠던 클래스와 네임스페이스를 자동으로 분석해주는게 좋았습니다.

기존 리버싱 툴들과 다르게 바이너리 분석을 프로젝트라는 단위로 진행하는점 또한 마음에 드는 부분 중 하나입니다. 프로젝트의 형태가 Non-Shared와 Shared 두 가지로 구분 되는데, 바이너리 분석을 협업으로 진행할 수 있는 형태를 지원한다는 점에서 새로움을 느꼈습니다.



# 설치

## 다운로드

![](https://i.imgur.com/P2VzRmS.png)

## 실행

![](https://i.imgur.com/sqo36cS.png)

(실행을 위해서는 자바 런타임이 설치되있어야합니다.)



![](https://i.imgur.com/bWaAgFX.png)

Agree





![](https://i.imgur.com/i362y3l.png)

실행중



![](https://i.imgur.com/66uajNc.png)

이걸로 실행은 끝입니다.



# 파일 분석

## 프로젝트 생성

![](https://i.imgur.com/RaBcC10.png)



![](https://i.imgur.com/i4jcj9M.png)

Shared 프로젝트는 팀원과 협업시 필요한 프로젝트 형태라고 합니다(아직 써본 적이 없어서 어떤 차이가 있는지 모릅니다 ㅜㅜ). 

Non-Shared를 선택하도록 하겠습니다.

![](https://i.imgur.com/aR6syHc.png)

프로젝트 명 입력.



## 바이너리 불러오기

![](https://i.imgur.com/SDdOnyA.png)

Import File 을 눌러서 분석할 바이너리를 불러와줍시다.



![](https://i.imgur.com/auyGVve.png)

유명한 abex crackme1 을 불러와봤습니다.



![](https://i.imgur.com/wXr9RWm.png)

작은 바이너리인데도 상당히 오래걸립니다.



![](https://i.imgur.com/aR3sWv7.png)

PE 정보를 요약해줍니다.



![](https://i.imgur.com/o9NReg5.gif)

불러온 바이너리가 분석이 되어있지 않다고 분석하겠냐고 물어봅니다.



![](https://i.imgur.com/BfRi3Nw.png)

30개 정도의 Analysis Option이 있습니다.

포럼 글에 의하면 300mb짜리 바이너리는 Analyze 하는데만 하루 종일 걸릴 수도 있다고 하네요(ㄷㄷ).



## 분석

![](https://i.imgur.com/PC0eEjK.png)

엔트리 함수로 잡으니 오른쪽에 자동으로 디스어셈블 된 코드가 나와있어요.

![](C:/Users/Verte/AppData/Roaming/Typora/typora-user-images/1557588422436.png)

저 문자열 파라메터 심볼 말고 "Make me think your ~~" 형태로 표현할 수 있었으면 좋겠습니다..ㅁㄴㅇㄻㄴㅇㄹ

방법 아시는분은 꼭 알려주세요 ㅠㅠ



# 결론

제가 기드라를 잘 다루는게 아니라서 뭔가 마음대로 잘 안되는 부분도 있지만 저에겐 나쁘지 않았습니다.

오픈소스여서 앞으로 어떻게 발전할지도 궁금합니다.

