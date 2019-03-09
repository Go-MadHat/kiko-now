---
layout: post
title: ! "Android Application 분석 환경구축"
excerpt_separator: <!--more-->
comments : true
tags:
  - nalda
  - Debugger
  - Android
---

Android Application 분석을 위한 환경구축

<!--more-->

Android APK 분석을 위한 환경 구축이다.
기본적인 java설치와 jdk등은 각자 알아서 설치하지^0^

### APK추출


정적분석을 하기 전 가장 중요한건 대상 APK를 추출하는 것이다.
만일, 본인이 알고있는 다른 apk추출 방법이 있다면 건너뛰자!


apk추출은 많은 방법이 있는데, [플레이스토어](https://play.google.com/store , "플레이스토어")에서 해당하는 앱을 검색 해 보자!

![]({{ site.baseurl }}/images/nalda/android/01.png)

요즘 미세먼지로 말이 많으니 미세먼지 앱으로...!

다음은 [apk 다운로드 사이트](https://apps.evozi.com/apk-downloader/ , "apk 다운로드 사이트")로 이동하자!

![]({{ site.baseurl }}/images/nalda/android/02.png)

이 후 플레이스토에서 해당 앱의 링크를 복사해서 붙여넣는다!

붙여넣은 뒤, 녹색 버튼인 click here to download를 누르면 apk파일이 다운로드 된다!

![]({{ site.baseurl }}/images/nalda/android/03.png)

이렇게 APK추출은 끝이났다!


### 정적분석


예전(이라해봐야 2년전)에 apk를 분석할땐 dex2jar, apktools등을 이용해서 했는데(아마도 내가 몰라서 그랬던거겠지만..)
요즘에는 Java Decompiler가 짱짱 좋아서 APK파일을 넣으면 짜쟌 하고 나온당

[jadx 링크]( https://github.com/skylot/jadx/releases/download/v0.6.1/jadx-0.6.1.zip , "JADX 링크")

![]({{ site.baseurl }}/images/nalda/android/04.png)

