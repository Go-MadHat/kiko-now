---

layout: post

title: ! "[webhacking.kr] 23번/11번 writeup"

excerpt_separator: <!--more-->

tags:

 - Write-up

 - webhacking.kr

 - rlawogns

---


webhacking.kr에 있는 23번과 11번 문제를 풀어보았다.

23번 문제는 아래와 같다.  



<!--more-->



![]({{ site.baseurl }}/images/rlawogns/2018-07-31-webhacking.kr-writeup/webhacking.kr-writeup_01.PNG)


문제를 보니 `<script>alert(1);</script>`를 넣는게 미션이라고 알려준다.

일단은 넣으라는대로 넣고 제출을 해보았다.



![]({{ site.baseurl }}/images/rlawogns/2018-07-31-webhacking.kr-writeup/webhacking.kr-writeup_02.PNG)

역시나 no hack이라는 문구를 출력하면서 안 된다고 한다.

어떤것이 필터링이 걸려있는지 알면 필터링을 피해서 적으면 문제를 풀 수 있을듯 한데 그걸 모른다.

그래서 일단 어떤 필터링이 걸려있을지 유추를 해보기 위해 이것저것 적어보았다.



![]({{ site.baseurl }}/images/rlawogns/2018-07-31-webhacking.kr-writeup/webhacking.kr-writeup_03.PNG)




![]({{ site.baseurl }}/images/rlawogns/2018-07-31-webhacking.kr-writeup/webhacking.kr-writeup_04.PNG)




![]({{ site.baseurl }}/images/rlawogns/2018-07-31-webhacking.kr-writeup/webhacking.kr-writeup_05.PNG)




![]({{ site.baseurl }}/images/rlawogns/2018-07-31-webhacking.kr-writeup/webhacking.kr-writeup_06.PNG)


네 글자와 두 글자를 넣었을 때는 no hack이라는 문구가 출력되었고 a를 넣었을 때는 a가 그대로 출력이 되었다.

그리고 <도 필터링이 되지 않고 출력이 되는 것을 보아 2글자 이상을 적으면

필터링에 걸려 no hack이 뜨는 것으로 유추할수가 있었다.



![]({{ site.baseurl }}/images/rlawogns/2018-07-31-webhacking.kr-writeup/webhacking.kr-writeup_07.PNG)


저 필터링을 피해가기 위해 주석을 이용을 했다.

a를 적고 %00을 적어 주석을 해준 다음 asd를 적으니 aasd라고 나오는 것을 볼 수가 있었다.

그럼 이제 방법을 알았으니 asd대신 문제에서 넣으라고 한 것을 넣어봤다.



![]({{ site.baseurl }}/images/rlawogns/2018-07-31-webhacking.kr-writeup/webhacking.kr-writeup_08.PNG)


풀릴 줄 알았는데 머지...

페이지가 작동을 하지 않는다는 문구가 뜬다.

열심히 고민하다 크롬말고 인터넷익스플로러로 해보면 되지 않을까해서 시도를 해보려 했다.



![]({{ site.baseurl }}/images/rlawogns/2018-07-31-webhacking.kr-writeup/webhacking.kr-writeup_09.PNG)


그런데 어이없게도 이미 풀려있는 것을 볼 수가 있었다.

아마도 페이지가 작동하지 않는다고 떴지만 문제에서 원하는 것을 하여 문제가 풀린 것 같다.

11번 문제는 아래와 같다.



![]({{ site.baseurl }}/images/rlawogns/2018-07-31-webhacking.kr-writeup/webhacking.kr-writeup_10.PNG)


문제를 보면 pat의 값을 정규식으로 알려주고

pat의 값과 우리가 입력할 val값이 같으면 패스워드를 출력해주는 듯 하다.

그럼 pat의 값만 알아내면 문제를 풀 수 있으니 정규식을 해석해봐야겠다.



![]({{ site.baseurl }}/images/rlawogns/2018-07-31-webhacking.kr-writeup/webhacking.kr-writeup_11.PNG)


위의 표를 이용해서 해석을 해보았다.

_는 문자 그대로 _이고 \tp는 혼동했었는데 하나가 아니라 \t와 문자p인듯 하다.

탭은 %09로 적어주었다.

이렇게 해서 1aaaaa_.219.251.3.64.%09p%09a%09s%09s라는 값이 나왔다.

이 값을 val변수에 입력해주었더니 문제가 풀리는 것을 볼 수가 있었다.


![]({{ site.baseurl }}/images/rlawogns/2018-07-31-webhacking.kr-writeup/webhacking.kr-writeup_12.PNG)

