---
layout:post
title:"webhacking.kr 25��, 52�� writeup"
excerpt_separator:<!--more-->
tags:
 -writeup
 -webhacking.kr
 -rlawogns
---
webhacking.kr�� �ִ� 25���� 52�� ������ Ǯ��Ҵ�.

25�� ������ �Ʒ��� ����.
![]({{ site.baseurl }}/images/rlawogns/webhacking.kr_25,52writeup/25.PNG)
ó��ȭ���� hello world�� ����� �ǰ� ?file=hello�� �����ִ� ���� �� ���� �ִ�.
hello�ڿ� .txt�� �� �ٿ������� hello.txt�� �������� ���ߵǴ� ������ ��µǴ� �ɷ� ����
.txt�� �ڵ����� �ٿ��ִ� �� ����.
password.php�� ������ ������ �� �����Ƿ�
flie=password.php�� ���ְ� %00�� �ٿ� ���� ���� .txt�� �����Ͽ� �����
![]({{ site.baseurl }}/images/rlawogns/webhacking.kr_25,52writeup/25clear.PNG)
�̿� ���� 25�� ������ �н����尡 ���԰� �̸� �Է��Ͽ� ������ Ǯ�� �Ǿ���.

52�� ������ �Ʒ��� ����.
![]({{ site.baseurl }}/images/rlawogns/webhacking.kr_25,52writeup/52_1.PNG)
��������ǿ� ���� ������� ���ϴ� �˷��ְ� �ִ�.
��������� ������ �Ʒ��� ���� id=wogns0411�̶�� ���� ����� ���� �� ���� �ִ�.
![]({{ site.baseurl }}/images/rlawogns/webhacking.kr_25,52writeup/52_2.PNG)
������ ���� $_GET[id]�� ����������� �ؼ� id=wogns0411 ��Ű�� �����϶�� �Ǿ��ִ�.
�׷��� Set-cookie�� ����� ��Ű�� �����Ϸ��� �ߴµ� �ƹ��� �ص� �ȵǼ�
�̰����� �������Ҵ��� Set-cookie�� �ϸ� Ǯ���� �ʰ�
Ŭ���� ���ǿ� �ִ� clear�κ��� ������Ѵٴ� ���� �˰� �Ǿ���.
�׷��� CRLF(ĳ���� ����, ���� �ǵ�)�� �̿���
id=wogns0411%0a%0dclear: wogns0411�� �Ͽ����� Ǯ���� ���� �� �� �־���.
![]({{ site.baseurl }}/images/rlawogns/webhacking.kr_25,52writeup/52clear.PNG)
%0a - ���� �ǵ� %0d - ĳ���� ����