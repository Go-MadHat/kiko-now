---
layout: post
title: ! 'Frida_Gadget'
excerpt_separator: <!--more-->
tags:
  - Koon
  - Android
  - Frida
---
비루팅 환경에서도 분석할 수 있게 도와주는 Frida-Gadget에 대해서 소개합니다.

<!--more-->

# Frida
Frida는 Windows, Linux, Android 등 여러 플랫폼 위의 어플리케이션에서 사용 할 수 있는 동적 계측 도구입니다. 

Frida에서 제공하는 API를 이용하면 다양한 기능을 수행할 수 있습니다. 특정 어플리케이션의 함수를 추적하거나, 이미 개발된 어플리케이션에 코드를 삽입하여 후킹을 진행 할 수 있습니다.

Android 상에서 Frida를 사용할 때는 Frida-server / Frida-Gadget을 이용하는 2가지 방법이 존재합니다.

* Frida-server : 루팅된 환경에서 사용하는 방식
* Frida-Gadget : 루팅되지 않은 환경에서 사용하는 방식

# Frida-Gadget
![]({{ site.baseurl }}/images/Koon/Gadget/Gadget_1.PNG)
Frida-Gadget을 이용하는 방식에 대해서 설명을 하겠습니다.

Gadget은 프로그램에 의해 로드되어지는 공유 라이브러리입니다. 이 가젯을 사용하기 위해서는 프로그램 내부 코드를 수정하거나, Gadget 라이브러리를 추가하여야합니다.

이러한 가젯을 사용하는 이유는 간단합니다. Frida-server를 사용하지 못하는 환경(루팅되지 않은 환경)에서 Frida-server가 할 수 있는 일을 최대한 재현하기 위해서입니다.

Frida-Gadget과 연결이 이루어지면, 기존 제공되어지는 Api들을 루팅되지 않은 환경에서 사용할 수 있습니다. 이를 통해 후킹과 같은 기능을 사용할 수 있고, 루팅되지 않은 최신 폰에서의 앱 분석 역시 가능하게 됩니다.

# How To USe
### 1. 환경 설정

```
pip install frida-tools
```
분석 환경에 프리다를 설치합니다.

```
frida --version
```
프리다 버전을 확인합니다.

![]({{ site.baseurl }}/images/Koon/Gadget/Gadget_2.PNG)
자신이 설치한 프리다 버전에 맞는 Gadget을 다운 받습니다.

### 2. 앱 디컴파일
```
apktool d [***.apk]
```
분석하고자 하는 앱의 apk를 구한 뒤 이를 Apktool을 통해 디컴파일합니다.

디컴파일을 하는 이유는 앱 내부에 코드 수정이나 Gadget을 넣어야 되기 때문입니다.

### 3. 공유 라이브러리 삽입 및 코드 수정(중요)
![]({{ site.baseurl }}/images/Koon/Gadget/Gadget_3.PNG)

디컴파일 한 앱 내부에 공유라이브러리 폴더를 만든 후, 프리다 가젯을 삽입합니다.

![]({{ site.baseurl }}/images/Koon/Gadget/Gadget_4.PNG)
Manifest file 안에 "INTERNET" 관련 permission을 추가합니다.

```
System.loadLibrary("frida")
```
자신이 만든 앱에 사용하는 경우라면 이 코드를 java 파일안에 적습니다.

```
const-string v1, "frida"
invoke-static {v1}, Ljava/lang/system;->loadLibrary(Ljava/lang/String;)V
```
자신이 만든 앱이 아닌 경우라면, 스마일리(.smail)파일안에 이 바이트 코드를 집어넣으면 됩니다.

### 4. 리패키징

```
apktool b -o [out] [decompiled folder]
```
수정 했던 폴더 내용들을 이제 하나로 합쳐 apk로 만드시면 됩니다.


# 결과 확인 
![]({{ site.baseurl }}/images/Koon/Gadget/Gadget_5.PNG)

프리다 가젯이 들어간 앱이 실행되면 위의 그림과 같이 frida-ps 명령어를 통해 가젯이 동작 중인지 확인 할 수 있습니다.

![]({{ site.baseurl }}/images/Koon/Gadget/Gadget_6.PNG)

로그를 통해 확인을 해보면, 분석 환경에서 접근하기를 기다리는 가젯의 Listening 로그를 확인 할 수 있습니다.

접근하지 않는 경우 계속해서 대기하고있기 때문에... 앱은 정상적으로 실행되지 않고 계속 대기하고 있습니다.


# 실행
```
Frida -U Gadget 
```

형태로 추가 옵션을 넣거나 사용하면 됩니다.
