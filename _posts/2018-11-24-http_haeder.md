---
layout: post
title: ! "[wargame.kr] already_got"
excerpt_separator: <!--more-->
comments : true
tags:
  - wargame
  - web
  - already_got
  - http_header
  - chaem
---

wargame.kr의 첫번째 문제인 already_got 문제에서 나오는 `http header` 대해 공부해보고자 한다.  

<!--more-->

http://wargame.kr/challenge

![]({{ site.baseurl }}/images/chaem/wargame.kr/already_got_01.JPG)

http response header에 대해 아냐고 물어본다!  

그렇담 한 번 찾아본다!  

# 풀이

크롬 개발자 도구를 이용하여 간단하게 haeder를 볼 수 있었다.

View HTTP headers in Google Chrome
구글 크롬 브라우저에서 HTTP header 보기
1. 크롬 브라우저에서 F12 입력
2. 네트워크(Network)탭 선택
3. 좌측 네임(Name)에서 확인하고 싶은 파일 선택
4. 우측 헤더 (Header)탭 선택

![]({{ site.baseurl }}/images/chaem/wargame.kr/already_got_02.JPG)

그러면 이렇게 flag를 발견할 수 있다.  
크롬 개발자 도구뿐만아니라, paros, wireshark 등의 도구를 이용하여서도 header의 값을 볼 수 있다.  

# HTTP header

그렇다면 HTTP header에서 알 수 있는 정보는 무엇이 있을까??
위에서 본 header를 보면 다음 3가지로 구성되어있다.  
```
> General
> Response Headers
> Request Headers
```

각 항목에 어떤 정보를 담고 있는지 알아보았다.  

```
>General
Request URL: http://wargame.kr:8080/already_got/
Request Method: GET
Status Code: 200 OK
Remote Address: 175.207.12.40:8080
Referrer Policy: no-referrer-when-downgrade
```
일반 헤더 (General Header)
요청 및 응답 메세지 모두에서 사용 가능한 일반 목적의(기본적인) 헤더 항목

```
>Response Headers
Connection: Keep-Alive
Content-Length: 27
Content-Type: text/html; charset=UTF-8
Date: Thu, 22 Nov 2018 11:41:43 GMT
FLAG: 34c80c587f5839d4620d357bdde340485b3c6035
Keep-Alive: timeout=5, max=100
Server: Apache/2.4.18 (Ubuntu)
```

HTTP 응답 헤더(Response Header)

특정 유형의 HTTP 요청이나 특정 HTTP 헤더를 수신했을때, 이에 응답 함

connection : 클라이언트와 서버의 연결 방식 설정(Keep-Alive : 클라이언트와 접속 유지,close : 클라이언트와 접속 중단)
Content-Type : 헤더 응답 문서의 mime 타입
Content-Length : 요청한 파일의 데이터의 길이
Last-Modified : 문서가 마지막으로 수정된 일시
Server :  웹서버 정보
Date : 현재 일시를 GMT 형식으로 지정
ETag : 캐시 업데이트 정보를 위한 임의의 식별 숫자

```
>Request Headers
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding: gzip, deflate
Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7
Cache-Control: max-age=0
Connection: keep-alive
Cookie: chat_id=mm; ci_session=a%3A5%3A%7Bs%3A10%3A%22session_id%22%3Bs%3A32%3A%2265ba1f2226a2fe96f7466d503bd61549%22%3Bs%3A10%3A%22ip_address%22%3Bs%3A14%3A%22118.220.40.133%22%3Bs%3A10%3A%22user_agent%22%3Bs%3A115%3A%22Mozilla%2F5.0+%28Windows+NT+10.0%3B+Win64%3B+x64%29+AppleWebKit%2F537.36+%28KHTML%2C+like+Gecko%29+Chrome%2F70.0.3538.102+Safari%2F537.36%22%3Bs%3A13%3A%22last_activity%22%3Bi%3A1542886597%3Bs%3A9%3A%22user_data%22%3Bs%3A0%3A%22%22%3B%7Da3ed6d85175855f07fb36582f8cb14016045f864
DNT: 1
Host: wargame.kr:8080
Referer: http://wargame.kr/challenge
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36
```

HTTP 요청 헤더(Request Header)

요청 헤더는 HTTP 요청 메세지 내에서만 나타나며 가장 방대함

accept : 클라이언트가 처리하는 미디어 타입 명시 (예 : */*)
accept-encoding : 클라이언트가 해석할 수 있는 인코딩 방식 지정(예 : gzip, deflate)
accept-language : 클라이언트가 지원하는 언어 지정 (예 : ko)
user-agent : 클라이언트 프로그램(브라우저) 정보 (예 : Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1))
host : 호스트 이름과 URI의 port번호 지정 (www.test.com:8080)
connection : 클라이언트와 서버의 연결 방식 설정(Keep-Alive : 클라이언트와 접속 유지,close : 클라이언트와 접속 중단)
cookie : 웹서버가 클라이언트에 쿠키를 저장한 경우 쿠키 정보(이름,값)을 웹 서버에 전송
        (예 : JSESSIONID=CDEI3830DJEJ3K3KD23K39D49)

# 대응

그렇다면 이러한 노출을 방지하기 위해서는 어떻게 해야할까?  
이러한 노출을 방지하기 위하여 `server.xml`의 `http Connector 설정`에 다음과 같이 server=" "를 추가하고 그 사이에 노출을 원하는 문자열을 삽입한다.
```
<Connector port="8080" protocol="HTTP/1.1" server="Server"
```
그리고 다음과 같이 다시 curl을 통하여 확인하여 보면 Apache/2.4.18 (Ubuntu)이 아닌 지정한 Server라는 문자열로 표시된다.

$ curl -i http://192.168.1.100:8080
HTTP/1.1 200 OK
Accept-Ranges: bytes
ETag: W/"2457-1391593060000"
Last-Modified: Wed, 05 Feb 2014 09:37:40 GMT
Content-Type: text/html
Content-Length: 2457
Date: Mon, 22 Sep 2014 04:40:20 GMT
Server: Server
단, server=""로 할 경우 원래와 같이 Apache/2.4.18 (Ubuntu)가 표시되니 만일 blank 로 표시하고 싶다면 server=" "와 같이 공백을 포함하여 설정한다.

PHP와 Apache의 경우는 아래와 같이 수정하면 정보를 숨길 수 있다.
간단히 php.ini 파일과 httpd.conf 파일을 수정해주면 된다.
```
php.ini
expose_php = On => expose_php = Off
```
```
httpd.conf
ServerTokens OS => ServerToekns Prod
```
그 후 아파치를 재시작해주면 Server와 X-Powered-By 정보가 숨겨진 걸 확인할 수 있다.



참고 링크
https://sarc.io/index.php/tomcat/240-tomcat-response-header-server
http://www.ktword.co.kr/abbr_view.php?m_temp1=3790
http://mygumi.tistory.com/92
