---
layout: post
title: ! 'SamsungKnox'
excerpt_separator: <!--more-->
tags:
  - Koon
  - Android
---

특정 일(?)을 하던 도중, 삼성의 Knox API를 사용하게 되었습니다. 그래서Knox api에 대해 알게된 사실들에 대해 간단하게나마 적어볼까 합니다.

<!--more-->
# Knox
![]({{ site.baseurl }}/images/Koon/knox/knox_1.PNG)
Knox는 방어 능력이 뛰어난 삼성의 모바일 보안 플랫폼으로서, 요즘 나오는 삼성 디바이스에는 모두 탑재되어 있습니다.

이 Platform에는 침입, 악성 소프트웨어 및 악의적인 위협을 방지하는 다중 방어 및 보안 메커니즘이 구성되어 있습니다.

![]({{ site.baseurl }}/images/Koon/knox/knox_2.PNG)
삼성에서는 개발자들을 위하여 SDK를 제공하고 있습니다. 이 SDK를 통하여 SDK내 구현된 API를 활용하여 Knox 관련 앱을 제작할 수 있습니다.

# Knox API 사용 방법
![]({{ site.baseurl }}/images/Koon/knox/knox_3.PNG)
KNOX API를 통해 앱을 제작하고, 배포하는 과정은 위와 같습니다.
- Enroll/Sign in : SEAP에 회원가입이 필수적으로 필요합니다. 로그인을 통해서 필요 라이센스를 발급 받을 수 있기 때문입니다.
- Download SDK & Develop : 제공하는 SDK를 다운로드 후, 이를 통해 API를 활용할 수 있습니다.
- Deploy : 완성된 앱을 배포할 수 있습니다.

![]({{ site.baseurl }}/images/Koon/knox/knox_4.PNG)
배포하기에 앞서, 실제로 SDK를 활용하여 앱을 제작하는데에도 필요한 과정이 있습니다.
- Obtain Development key : SDK의 개발 라이센스 키를 발급받아야 합니다.
- Build your app: SDK에 제공된 api를 활용하여 앱을 제작하고, 빌드해야 합니다.
- Activate the license : SDK에 구성된 API를 실제로 사용하기 위해서는 먼저 라이센스 활성화를 해야합니다.(활성화 역시 제공되는 API를 통해 할 수 있습니다.)
- Obtain commercial key : 앱을 배포하기 위해서는 상업용 키를 얻어야합니다.
- Associate your app : 기존 앱 개발 라이센스 키 대신 상업용 키로 연결하여 배포할 수 있습니다.

위의 과정을 통하여 앱을 제작하였고, 이에 대해 하나씩 말씀드리겠습니다.

## Obtain Development key
![]({{ site.baseurl }}/images/Koon/knox/knox_5.PNG)
라이센스 키는 삼성 Knox API를 활용하는데 필요합니다. 라이센스 키는 총 3가지가 있습니다.
- KPE Development Limited : 무료 사용자용으로, 매우 적은 API만 사용 가능합니다.
- KPE Development :  파트너용으로, 구현된 모든 KNOX API를 사용 가능합니다.
- Backwards Compatible Key : 호환성용으로, 현재 KNOX 버전은 3이며, 이 3버전을 제외한 1,2 버전을 사용 시 필요로 합니다. 위에 설명된 나머지 라이센스 키와 같이 사용되어야 합니다.

## Build Your app
앱을 구성하는 것에는 각 개발자마다 원하는 방향으로 앱의 기능을 구현하기 위해, 모두 다를 것이다.
하지만 지켜야할 것들이 3가지가 있다.
- Manifest file 권한 선언
- Knox 권한 접근을 위한 장치관리자 리시버 설정
- 개발 라이센스 활성화를 위한 라이센스 리시버 설정
위의 3가지는 반드시 해야하는 것들이다.(라이센 스 및 권한 설정을 위해서...)

### Manifest file 권한 선언
![]({{ site.baseurl }}/images/Koon/knox/knox_7.PNG)
위의 그림과 같이 사용할 권한에 대해 선언을 해주어야한다.

각 기능에 맞는 권한만 설정해주어야 한다.

### Knox 권한 접근을 위한 장치관리자 리시버 설정
* Manifest file 리시버 등록
![]({{ site.baseurl }}/images/Koon/knox/knox_8.PNG)
리시버는 단말기 내에서 이루어지는 수많은 일(배터리 부족, 전화, 알림 등)을 알려주는 역할을 합니다.

즉, 단말기에서 발생하는 이벤트를 계속적으로 체크하면서, 상태를 받아올 수 있는 구성 요소입니다.

* res폴더에 device_admin_receiver.xml 만들기
![]({{ site.baseurl }}/images/Koon/knox/knox_9.PNG)
안드로이드 앱 기본 요소에 res 폴더가 있는데, 그 내부에 xml형태로 파일을 관리합니다.

이 내부의 요소를 새롭게 만드셔야합니다. 이 부분에 대한 정확한 이유는 아직 공부가 부족하여, 잘 모르겠습니다.

* Device Admin Reciver class 만들기
![]({{ site.baseurl }}/images/Koon/knox/knox_10.PNG)
이 Class를 통해 장치관리자 페이지를 호출하여, 사용자가 필요한권한을 허용할지 결정하게 합니다.

### 개발 라이센스 활성화를 위한 라이센스 리시버 설정
위에서 장치 관리자 리시버에서 설정했던 것과 같이 Manifest에 리시버를 등록하고, 라이센스 리시버 클래스를 작성하여야합니다.
![]({{ site.baseurl }}/images/Koon/knox/knox_11.PNG)
위의 그림과 같이 onReceive 함수를 구현하여, 특정 이벤트를 받아들여, 라이센스가 활성화되게 합니다.

![]({{ site.baseurl }}/images/Koon/knox/knox_12.PNG)
그리고 위의 그림처럼, 발급 받은 라이센스 키를 하드 코딩형태로 작성해야, 직접 앱을 테스트 할 수 있습니다.

## Activate License
API 호출을 하기위해서는 라이센스 활성화 API를 사용하여 삼성 라이센스 서버로 부터 라이센스에 대한 권한을 받아야합니다.
![]({{ site.baseurl }}/images/Koon/knox/knox_13.PNG)
위의 그림처럼 간단하게 3단계 과정을 지니고 있습니다.

1. ACTIVATION REQUEST(활성화 요청)
: activateLicense() API가 호출되면, 라이센스 에이전트를 통해 삼성 라이센스 서버에 활성화 요청 전달 (요청에 라이센스 키, 앱 및 기기에 대한 정보 포함)

2. VALIDATION(유효성 검사)
: 라이센스 서버가 라이센스 키를 검사 할 때 라이센스 서버는 또한 앱의 공개 키 해시 및 패키지 이름의 유효성을 검사하여 해당 라이센스 키를 활성화 할 수 있는지 여부 결정

3. PERMISSION GRANT(사용 권한 부여)
: 유효성 검사 단계가 통과되면 Manifest file에 에 요청된 사용 권한이 부여

## Obtaint Commercial Key & Associate your app
직접 해보고 싶었지만, 상업용 라이센스 키를 얻어, 상업적으로 배포하는 것은 파트너만 가능합니다.

이 말은 저 같은 무료 사용자는 배포할 수 없다는 것입니다. 그렇기 때문에 직접 해보지 못했습니다.


## 느낀점
실제로 Knox 앱을 제작을 하는데에 있어서, 생각보다 어려웠다. 그 이유는 기본적으로 안드로이드 코딩을 별로 한적이 없던 점, 그리고 삼성의 튜토리얼이 생각보다 자세히 적혀있지 않다는점 때문이었다.

지금은 익숙해져, 간단하게나마 여러 API를 활용할 수 있겠지만, 삼성의 파트너가 아닌 이상 knox API를 통해 할 수 있는 것은 매우 적었고, 그 적은 기능 역시 크게 쓸모 있는 기능이 아니여서, 앞으로 이 Knox API를 딱히 쓸 일이 없을 것 같다.
