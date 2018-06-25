---
excerpt_separator: <!--more-->
tags:
  - Design
  - MachineLearning
  - Virtual
  - IDS
  - Koon
---

안녕하세요. 'MadHat'팀의 'Koon'입니다. 저는 3월부터 ~ 5월 말까지 약 2개월 동안 진행한 "가상 IDS 설계" 프로젝트에 대해서 알게된 지식들을 제가 경험한 절차에 맞게 소개하려합니다.


<!--more-->

# 사전 지식
* IDS, IPS  

IDS는 침입 탐지 시스템으로서, 컴퓨터 또는 네트워크에서 발생하는 이벤트들을 모니터링하고, 침입 발생 여부를 탐지하는 자동화된 시스템을 의미합니다.  

IDS에 대해 알았으니, 같이 언급되는 IPS에 대해 알아봅시다.  

IPS는 Instrusion Prevent System, 길게 말할 필요 없이 탐지 시스템인 IDS에 추가적으로 차단하는 기능이 합쳐진 것이라 생각하시면 됩니다.  

| IDS | 내부 탐지 시스템 |
| IPS | 내부 탐지 시스템 + 후 처리 기능 => 내부 방지 시스템 |

* Virtual  

'가상화', 물리적인 컴퓨터의 리소스의 특징을 다른 시스템, 응용 프로그램, 최종 사용자들이 리소스와 상호작용하는 방식으로부터 감추는 기술로 정의 할 수 있습니다.  

마치 자신(각 사용자들)이 그 자원을 독점하고 있다고 생각이 들게 끔하는 것!  

* 하이퍼바이저  

하이퍼 바이저, 프로세서나 메모리와 같은 다양한 컴퓨터 자원에 서로 다른 각종 운영체제 접근 방법을 통제하는 얇은 계층의 소프트웨어 입니다.  

다수의 OS를 하나의 컴퓨터 시스템에서 가동할 수 있게 하는 소프트웨어  

OS간 서로 방해하지 못하도록 VM에 대한 자원 및 메모리 할당, 처리 등을 보장해줍니다.(고립화 보장)  

*종류*
![]({{ site.baseurl }}/images/Brutekoon/IDS/p1.PNG)
