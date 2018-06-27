---
layout: post
title: ! "[Python Angr] 신기한 리버싱 꿀툴 Angr를 숙지해두는건 어떨까요?"
excerpt_separator: <!--more-->
comments : true
tags:
  - Python 2.7
  - Angr
  - Vertex
---


안녕하세요. Vertex입니다.

먼저 저는 Angr에 대한 지식이 거의 없음을 알려드립니다.  

정보보호영재교육원에서 주관한 제 3회 정보보안 경진대회 단체전에 문제중

DEFCON 2016 예선 baby-re 문제의 패러디가 있었습니다.

친구의 라이트업에서 Angr를 처음 알게되었는데 이런 신기하고 아름다운 파이썬 모듈을

소개드리게 되어 매우 기쁩니다.

<!--more-->

그 친구가 Angr를 알게 된 DEFCON 2016 예선문제 baby-re의 라이트업을 소개드리겠습니다.

```
http://hack.carleton.team/2016/05/21/defcon-ctf-qualifier-2016-baby-re/
```

### -Angr 설치!-

리눅스 환경의 Python 2.7 에서 `pip install angr` 라고 입력하시면 

정말 간단하게 angr의 수많은 의존 모듈도 다운받아줍니다!

![]({{ site.baseurl }}/images/vertex/python-angr-introduce/1angr install.PNG)  

angr의 소개 및 api의 소개, 정확한 설치방법을 원하시면 역시 공식 홈페이지로 가시면 좋겠습니다!


```
http://angr.io/
```

### 1. 문제 소개


![]({{ site.baseurl }}/images/vertex/python-angr-introduce/2babyre.png) 
(출처:http://argosarch.tistory.com/45 )

13개의 문자를 입력받습니다.
그리고 CheckSolution 함수를 통해 13개의 문자가 조건에 통과하는지 확인합니다.

이 조건에 통과하는 13개의 문자를 찾아내야 합니다.

도구를 통해 문제를 푸는 방법중 제가 아는 방법은 대략 두 가지 인데요.

바로 z3 라이브러리를 이용한 연립방정식 계산과

angr를 이용한 무작위 대입법입니다.

z3물론 강력한 라이브러리이지만 시간이 촉박한 ctf 대회에서 모든 조건식을 연립방정식으로
추가한다는 것은 꽤나 시간이 오래걸립니다.

angr는 간단한 조건을 주면 자동으로 해당 조건에 도달하기 위해 무작위 대입을 시작합니다!

경우의 수가 세어 낼 만 하다면 문제를 간단히 풂으로써 많은 시간을 절약할 수 있을것입니다.

### 풀이 코드
~~~python
import angr
proj = angr.Project('./baby-re', load_options={'auto_load_libs': False})
path_group = proj.factory.path_group(threads=4)
path_group.explore(find=0x40294b, avoid=0x402941)
print path_group.found[0].state.posix.dumps(1)
~~~


~~~python
path_group.explore(find=0x40294b, avoid=0x402941)
~~~
에서 find에 넘긴 주소는 플래그를 출력하는 주소이고, avoid에 넘긴 주소는 틀렸음을 알려주는 주소입니다.

### 결론

정적&동적 심볼릭 분석을 자동으로 해주는 angr는 아직 사용법이 미숙해 더 활용할 여지가 있음에도 불구하고 어떻게 사용하는지 몰라 많이 아쉽습니다.

관련 api도 익히고 

```
https://docs.angr.io/docs/examples.html
```

여기에는 심지어 angr를 이용한 문제풀이 예제도 많이 올라와 있습니다.

더 알아보고 이에 대해서 자세히 포스팅 해보도록 하겠습니다.
