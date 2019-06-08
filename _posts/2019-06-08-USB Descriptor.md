---
layout: post
title: ! "USB Descriptor"
excerpt_separator: <!--more-->
tags:
  - Vertex
  - USB
---

USB 디바이스는 어딜가나 우리 주변에 있다.

이런 USB 디바이스의 정체는 뭘까? 무엇이 USB의 존재를 정의할까? 답은 『USB Descriptor』 이다.

USB descriptor를 살펴보고자 한다.

<!--more-->

# USB란 무엇일까?
USB는 Universal Serial Bus이다. 입출력 표준 프로토콜의 하나이다.



![](https://i.imgur.com/XP580kK.png)

USB 단자를 자세히 보면 핀이 4개밖에 없다.

이 중 두 개는 전원공급 단자, 나머지 두 개는 데이터를 주고받는 단자이다.



| 핀 번호 | 선 색깔 | 기능             |
| ------- | ------- | ---------------- |
| 1       | Red     | +V(대게 5볼트임) |
| 2       | White   | D-               |
| 3       | Green   | D+               |
| 4       | Black   | GND              |

![](https://i.imgur.com/Ab7Mf5r.png)

`(와! 정말이다!)`



가장 기본적으로 많이 쓰이는 USB 형태로, 이 외에도 스마트폰에 사용하는 micro 5pin(type b), USB type-c, Mini type-a, 프리너나 아두이노에 사용하는 Mini type-b 등등이 있다.



USB 단자끼리는 보통 하나의 버스에 연결되어 대역폭을 공유하므로, 여러 단자를 사용할 경우 속도가 느려진다. USB 1.0을 시작으로 2.0, 3.0, 3.2 Gen1, 3.2 Gen2, 3.2 Gen 2x2 가 있다.



# USB의 통신

USB가 Universal “Serial” Bus기 때문에 여타 시리얼 포트처럼 한번에 하나의 비트를 전송한다.

이더넷 포트나 RS232도 마찬가지 이다.

![](https://i.imgur.com/3FJ5Gzg.png)

USB 패킷에는 요런 패킷들이 있다. 하지만 이 패킷 id를 다 외울 필요는 없다.

요즘에는 USB 컨트롤러가 USB를 통해 데이터를 주고 받을때 해야하는 핸드쉐이크, 싱크, CRC 체크를 알아서 해준다. 컨트롤러에 데이터를 보내거나 받거나 명령만 내려주면 된다.

또는 MicroController나 MicroProcessor에 이런 USB를 다룰 수 있는 라이브러리를 제조사에서 제공하는 경우가 많아서 때문에 이정도까지 알아야 할 필요는 없다고 한다.



# USB Descriptor

![](https://i.imgur.com/uMnChsJ.png)

USB Descriptor의 계층도를 그리면 이렇게 된다.

각각의 디스크립터는 구조체 형태를 가지고 있다. USB 장치를 개발할 때, 해야 할 일은, 이 USB 장치가 어떤 장치이고, 전력을 얼마나 소모하는지, 어떤 형태로 데이터를 주고받을지를 미리 적어놓고, USB가 연결될 때 이 명세서를 넘겨주면 호스트에서는 해당 장치로 인식한다.



### Device Descriptor

 USB 디바이스라면 하나 가지고 있다. 여기서 패킷의 최대 사이즈, 어떤 USB 스펙을 사용하는지, 제품id, 제조사 id 등의 정보를 가지고 있다.

![](https://i.imgur.com/97geGrv.png)



### Configuration Descriptors

Configuration Descriptor은 여러개를 가질 수 있고, USB Device가 전류를 얼마나 소모하는지 명시할 수 있다.

![](C:/Users/Verte/AppData/Roaming/Typora/typora-user-images/1559975439197.png)



### Interface Descriptors

또한 여러 개 가질 수 있는 디스크립터이다. 실질적인 데이터를 주고 받는 Endpoint들을 그룹으로 묶는 역할을 한다. 장치의 기능에 따라 Endpoint들을 엮어서 사용한다.

![](https://i.imgur.com/V6zGVYC.png)

bInterfaceClass, bInterfaceSubClass 그리고 bInterfaceProtocol필드를 통해 장치가 HID(Human Interface Device)인지, 통신장비인지, Mass Storage인지 정할 수 있다.



### Endpoint Descriptors

여러 개 가질 수 있는 디스크립터.

실제 데이터를 주고받을 수 있는 Pipe를 정의하는 디스크립터이다.

![](https://i.imgur.com/r8aGubU.png)

보면 bmAttributes에 Transfer Type을 나타내는 2비트가 있는데,

총 네 종류로 Control, Isochronous, Bulk, Interrupt로 나눠진다.

Control은 호스트가 디바이스에게 명령을 내리거나 상태를 전송하기 위한 통로이다. USB가 작동하기 위한 여러 기능들을 수행하는데 필수적인 통로이다. Low speed(USB1.0)에서는 한번에 8바이트를,

Full speed(USB1.0)에서는 64바이트만 보낼 수 있고, High speed(USB 2.0)은 8, 16, 32 또는 64바이트를 보낼 수 있다.



Inturrupt 파이프는 그냥 가끔 데이터를 찔끔찔끔 보내줄 때 쓰는 파이프인데, 이걸 쓰는 이유는 Latency가 보장되고, 양방향성으로 데이터를 주고받을 수 있다는 점, 그리고 무결성을 보장하기 때문(에러체크하고 에러 발생시 재전송)하기 때문에 사용한다. 키보드나 마우스에 많이 사용한다.



Bulk 파이프는 많은 데이터를 밀어넣을 수 있는 포트지만, 언제 다 보내질지 보장이 없는 대신, 무결성을 보장해주며 USB 저장장치에 사용된다.



Isochronous 파이프는 많은 데이터를 빠르게 밀어넣을 수 있지만 무결성은 보장하지 않는 포트이다. 웹캠이나 스피커, 마이크에 사용한다.



## 출처

https://www.beyondlogic.org/usbnutshell/usb2.shtml
