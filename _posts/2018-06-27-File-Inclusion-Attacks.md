---
layout: post
title: File Inclusion Attacks
excerpt_separator: <!--more-->
comments : true
tags :
  - sang-gamja
  - File Inclusion Attacks
---

File Inclusion Attack이란 공격자가 php script를 통해 웹서버에 파일을 올리게 하는 것을 가능하게 하는 공격기법이다.  

이 취약점은 웹어플리케이션이 사용자에게 파일을 올리거나 업로드 시킬수 있게 할 때에 많이 일어난다.  

<!--more-->

이 공격은  

1.Code execution on the web site

2.Cross Site Scripting Attacks(XSS)

3.Denial of Service(DOS)

4.Data Manipulation Attack


같은 공격에 사용되어 질 수 있다.  

이 공격은 2가지로 나누어 지는데  

1. Local File Inclusion(LFI)

2. Remote File Inclusion(RFI)

이다.  

### Local File Inclusion(LFI)

LFI는 PHP계정이 액세스한 파일이 PHP함수인 include 또는 require_once에 파라미터로 들어갈 때에 발생한다.  

![]({{ site.baseurl }}/images/gamja/fla/fla1.png)

이 취약점은 페이지가 포함되어야 하는 자리에 파일에 대한 경로가 넣었을 때 필터링을 제대로 거치지 않아서 폴더의 위치를 나타내는 문자열이 들어갔을 때에 발생 할 수 있다.  

예를 들어

![]({{ site.baseurl }}/images/gamja/fla/fla2.png)

page = file1.php를 넣었을 때에 이런 창이 뜬다면

![]({{ site.baseurl }}/images/gamja/fla/fla3.png)

?page=/etc/passwd를 넣었을 때에 이런 창을 return할 수 있게 된다.

### Remote File Inclusion(RFI)

RFI는 다른 서버에 있는 파일의 위치의 URI가 PHP 함수 "include","include_once","require","require_once"들에 파라미터로 삽입이 될 때에 발생한다.  

PHP는 페이지안에 content를 포함한다. 만약에 content가 php 소스코드를 실행시키면 php는 파일을 실행시킬것 이다.  

PHP Remote File Inclusion은 취약한 PHP script안에 있는 PHP 코드안에 공격을 감행하여, 웹서버에서 원격으로 조종해 웹을 파괴시키거나 정보를 탈취 당할 수 있는 위험성이 있다.  

예를 들면

http://192.168.1.8/dvwa/vulnerabilities/fi/?page=file1.php

http:// 192.168.1.8/dvwa/vulnerabilities/fi/?page=http://google.com

로 page 파라미터에 또 다른 실행을 넣어서 원격으로 조정가능하게 만든다.

![]({{ site.baseurl }}/images/gamja/fla/fla4.png)
