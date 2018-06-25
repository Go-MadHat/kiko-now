---
layout: post
title: DPboard making and writeup
excerpt_separator: <!--more-->
tags:
  - MitNy
  - SQL injection
  - writeup
  - WebHacking
---

안녕하세요 MadHat 팀에서 웹 해킹을 공부하고 있는 `MitNy` 입니다. 지금까지 CTF나 Wargame에서 문제를 풀어오면서
문제를 푸는 것도 중요하지만 문제를 만들어보는 것도 중요하다는 생각을 하고 있습니다.
처음에는 스테가노그래피가 재밌어서 이것저것 숨겨보기도 했지만 요즘은 웹 문제를 만드는 게 더 재밌어졌습니다.
그래서 이번에는 제가 SQL injection 문제를 만드는 것과 풀이를 동시에 하도록 하겠습니다.

<!--more-->

#DPboard

문제 이름은 `DPboard` 입니다. DPboard는 Display board를 제 마음대로 줄인 것이며 기본적인 컨셉은
전광판에 사용자가 원하는 문구를 띄울 수 있다는 것입니다.
어디에 플래그가 뜰지 예상이 가시나요?

DPboard 문제의 메인 페이지는 다음과 같습니다.

![]({{ site.baseurl }}/images/mitny/DPboard/dp1.PNG)

메시지를 추가하고, 전광판에 표시를 하는 기능이 있고 지금까지 등록된 메시지들이 테이블 형식으로 보입니다.

메시지를 추가하게 되면 다음과 같이 테이블에 메시지가 추가됩니다.
![]({{ site.baseurl }}/images/mitny/DPboard/dp2.PNG)

`hey` 라는 메시지를 전광판에 띄워 볼게요.
![]({{ site.baseurl }}/images/mitny/DPboard/dp3.PNG)




