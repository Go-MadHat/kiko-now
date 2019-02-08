---
layout: post
title: ! '35C3_CTF_Corebot'
excerpt_separator: <!--more-->
tags:
  - Koon
  - Frida
  - Tool
  - crack
---

Android 에서의 Frida 사용에 대해 얘기합니다.
Frida에 대한 여러가지 기능이 존재하겠지만, 여기서는 후킹 위주로 설명을 진행합니다.
<!--more-->

## Frida?
Dynamic Instrumentation toolkit
여러 플랫폼 위에서 돌아가는 앱들에 대해서 여러 기능들을 수행하게 해주는 도구.
제공된 API를 통해 특정 내부 동작을 캐치하거나, 캐치한 동작에 대한 코드를 수정하여, 앱을 작동시킬수 있는 도구.
이러한 기능 덕분에 여러 테스팅을 거칠 때 빌드에 의해 시간이 많이 소요되는 부분을 해결 해주는 도구.

##후킹
각종 프로그램에서 함수호출, 메시지, 이벤트 등을 중간에 바꾸거나 가로채는 기술
![]({{ site.baseurl }}/images/Koon/Frida/frida_1.PNG)

Frida의 경우,
Interceptor API 및 Java API 를 제공해주기 때문에, 이 API를 이용하여 특정 함수의 이름 또는 위치를 통해 후킹을 할 수 있습니다.

## 루팅 탐지 우회 ( 후킹 해보기 )
특정 APK 통해 안드로이드 상에서 후킹을 어떻게 하는지 알아보겠습니다.
(Frida에 대한 세팅이 완료된 상태로 가정합니다.')

'상황 - 실제 루팅된 기기로 하고 있는중.'
'문제 APK - UnCrackable-Level1.apk'
'URL - https://github.com/OWASP/owasp-mstg/tree/master/Crackmes/Android/Level_01'

adb install 명령어를 통해 기기에 apk를 설치합니다.

이 apk에 대해서 흐름만 설명을 하자면, 루팅 탐지 ->  사용자 입력 -> 검증 이러한 흐름을 지니고 있습니다.

여기서는 '루팅 탐지'에 대해서만 얘기를 합니다.

실행을 하게 되면...
![]({{ site.baseurl }}/images/Koon/Frida/frida_2.PNG)
위와 같은 문구가 뜨게 되며, ok를 누를 시 바로 나가지게 됩니다.

jadx.gui를 통하여 apk의 코드를 확인해 봅니다.

보통의 안드로이드 앱의 경우 onCreate 함수가 처음 부분이라 말할 수 있는데, 이부분에서 "Root detected"가 발견되었습니다.
![]({{ site.baseurl }}/images/Koon/Frida/frida_3.PNG)
이 문구가 발견된 함수를 분석하게 되면...
![]({{ site.baseurl }}/images/Koon/Frida/frida_4.PNG)
버튼을 누르게 되면, 바로 System.exit을 통해 나가지게 코드게 작성되어 있습니다.

그러면 버튼을 눌러도 나가지게 않게만 한다면, 다음 흐름(입력)으로 넘어갈수 있습니다.

이 System.exit을 후킹해보도록 하겠습니다.
![]({{ site.baseurl }}/images/Koon/Frida/frida_5.PNG)
위의 그림은 코드입니다.
위의 코드를 참고하여 설명 드리겠습니다.
on_message의 경우 출력하는 코드인데, frida 공식 홈페이지의 튜토리얼과 같은 설명, 예제를 보게 되면, 저런식으로 만들어서 사용합니다. print의 내용을 변형하여 사용할 수 있겠지만, on_message의 인자인 message와 data는 저형태 그대로 입니다.

PACKAGE_NAME은 안드로이드 기기에서 앱이 동작하고 있을때 가지는 이름입니다. 안드로이드 기기에서 작동하고 있을 때 프로세스 확인 명령어를 통해 확인 할 수 있습니다.

jscode가 새로우실텐데, frida의 경우 자바스크립트의 코드를 통해 동작합니다. 실제로는 C와 같은 코드로 구성되어져 있지만, 자바스크립트 코드를 통해 그런 코드들에게 자동적으로 접근하게 되어져 있기 때문에 API를 제공해주는 자바스크립트 코드를 쓰면 됩니다.

Java.use를 통해 System.exit 함수가 존재하는 곳의 클래스 이름을 적습니다. (이 이름을 모른다면, 하나하나 추적을 해야 하지만, 알려진 함수들은 보통 인터넷에 치면 나옵니다.)
URL - https://docs.oracle.com/javase/9/docs/api/java/lang/System.html

Java.use를 통해 현재 클래스를 exitClass로 새로 구성하였습니다.
이 새로 구성한 exitClass가 이제 앱에서 동작하고 있는 System.exit함수가 있는 클래스를 대신하는 것입니다.

이 exitClass.exit.implementation 이 의미는 새로 구성한 클래스의 exit함수를 구성하는 것입니다.
자바스크립트의 function 함수를 통해 로그를 찍도록 하는 함수로 재정의 한 것입니다.

이제 이렇게 구성된 자바스크립트 코드를 로드하면됩니다.

로드하는 방법은 주석을 통해 간단히만 적겠습니다.
![]({{ site.baseurl }}/images/Koon/Frida/frida_6.PNG)

이제 버튼을 누르면, 나가지지 않고 루팅 탐지가 되었다는 창만 사라지고, 입력하는 창은 그대로 있을 것입니다.
![]({{ site.baseurl }}/images/Koon/Frida/frida_7.PNG)

그 뒤부터는... 계속 후킹해서 ... 풀면 됩니다.

