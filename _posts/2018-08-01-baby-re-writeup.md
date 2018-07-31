---
layout: post
title: "baby-re writeup"
tags:
 - Write-up
 - smlsml
 - angr
---
이번에 작성할 내용은 angr에 관한 내용이다.
사실 angr에 대한 공부는 예전부터 ctf에 문제가 보일 때 마다 해야지... 라고 하면서도 미뤄왔던 내용인데, 이번 기회에 공부해보자 한다.

<!--more-->

angr는 파이썬 라이브러리인데, symbolic 분석을 해준다.
꼭 가야할 주소와 피해야할 주스를 설정해두면 실행시켰을 때 프로그램이 알아서 원하는 분기로 실행을 해준다.
사실 이 두가지만으로 동작하는 것은 아니고, 그 외에도 필요하다면 다양하고 유용한  설정들을 추가시켜줄 수 있다.

보통 angr의 예제로 나오는 문제들을 살펴보면 z3로 푸는 경우도 볼 수 있는데, z3에 비해서 더 다양한 문제에 활용이 가능하다.

angr는 github에서 검색하면 쉽게 찾아볼 수 있다.

이번에 angr를 연습하기 위해서 풀어본 문제는 defcamp 2016 baby-re라는 문제이다.
이 문제는 흔히 angr 사용법이나 연습, 예제 등의 검색을 하면 자주 보이는 문제인데, 풀이도 사람들 마다 다양하지만 비슷한 것을 볼 수 있다.

먼저 파일을 그냥 실행시켜보면 아래와 같이 나온다.

![]({{ site.baseurl}}/images/smlsml/baby-re/1.PNG)

하지만 어떤 글을 입력해보아도 나머지 var에는 입력이 되지 않는다.

파일을 ida로 열어보면 아래와 같은 코드를 main에서 확인할 수 있다.

![]({{ site.baseurl}}/images/smlsml/baby-re/2.PNG)
![]({{ site.baseurl}}/images/smlsml/baby-re/3.PNG)

var[0]~var[12]에 각각 입력을 받고 값이 올바르면 flag를, 아니라면 Wrong을 출력하는 부분을 볼 수 있다.

따라서 우리가 꼭 가야할 주소는 0x402936(flag의 printf가 실행되는 곳)
피해야할 주소는 0x402941(Wrong을 printf 하는 부분)이다.

쉽게 찾아볼 수 있는 예제 코드들은 대체로 아래와 같다.

```python
import angr

def main():
	proj = angr.Project('./baby-re', load_options={'auto_load_libs':False})
	path_group = proj.factory.path_group(threads=4)
	path_group.explore(find=0x402936, avoid=0x402941)
	return path_group.found[0].state.posix.dumps(1)

if __name__ == '__main__':
	print(repr(main()))
```

중요하게 볼 부분은 path_group.explore(find=0x402936, avoid=0x402941) 이다.
하지만 이 코드를 사용해 문제를 풀어보려니 오류가 계속해서 발생했다.

정확히 왜 오류가 나는지는 모르겠지만 warning을 읽어보니 코드에 사용한 함수 중에서 factory.path_group()이 deprecated 되었다며 factory.simulation_manager()를 사용해달라고 한다

![]({{ site.baseurl}}/images/smlsml/baby-re/4.PNG)

simulation_manager에 대해서 간단히 살펴보니 symbolic execution을 제어하고, state의 탐색을 위한 알고리즘을 적용하는데 사용되는 angr의 핵심 명령이라고 한다.

이것을 사용한 다른 예제 코드를 선배의 도움으로 받아볼 수 있었는데,
구글에서 다른 분들이 사용한 코드들도 찾아보니 대체로 비슷한 코드였다.
살펴보면 아까전에 비해서 훨씬 더 복잡해진 것을 확인할 수 있다.

``` python
import angr
import claripy

class scanf_hook(angr.SimProcedure):
       def run(self, fmt, ptr):
               self.state.mem[ptr].dword = vars[self.state.globals['scanf_count']]
               self.state.globals['scanf_count']+=1

proj = angr.Project('./baby-re')

vars = [claripy.BVS('%d' %i, 32) for i in range(13)]

proj.hook_symbol('__isoc99_scanf', scanf_hook(), replace=True)

sm = proj.factory.simulation_manager()
sm.one_active.options.add(angr.options.LAZY_SOLVES)
sm.one_active.globals['scanf_count']=0

sm.explore(find=0x402936, avoid=0x402946)

flag = 'flag : '

for x in vars:
       flag += chr(sm.one_found.solver.eval(x))
print(flag)

```

먼저 claripy는 solver engine으로서 제약 조건을 해결하는 기능을 제공하며, 사용방법은 z3와 유사하다고 한다.

(참고 : https://www.lazenca.net/display/TEC/angr )

(참고 : https://docs.angr.io/docs/claripy.html )

이 코드에서는 int형으로 13개의 변수를 선언해주기 위해서 사용되었다.

scanf_hook

scanf_hook은 scanf를 연속으로 넣어주는 부분을 처리하기 위해서 사용되었다.
scanf_count를 세면서 vars[]에 값을 넣는다.

vars는 13번동안 claripy.BVS('%d' %i, 32)를 반복하는데,
다른 예제 코드를 살펴보면 claripy.BVS('x', 32)를 사용하면 32비트 기호 비트벡터 x를 만든다고 되어있다.

proj.hook_symbol은 아까전에 만들어둔 훅을 사용할 심볼을 정하는데,
__isoc99_scanf가 나올 때 마다 scanf_hook을 사용한다.

그 후 'simulation _ manager' 를 사용하여 옵션을 넣어주고,
sm.explore(find=0x402936, avoid=0x402946)을 통해 이전 코드와 비슷하게 주소를 정해준다.
그리고 for문을 돌면서 찾은 flag값들을 출력해준다.

실행 결과는 아래와 같다

![]({{ site.baseurl}}/images/smlsml/baby-re/5.PNG)

처음 보는 함수들이라 단순히 이해하는 것 만으로도 상당히 오랜 시간이 걸렸다.
그런데도 사실 아직까지도 직접 사용하지는 못할 것 같아서 더 다양한 예제들을 풀면서 공부해나갈 예정이다.
