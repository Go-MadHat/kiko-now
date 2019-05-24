---
layout: post
title: [DEFCON 2019 Quals] GIF Redacted-Puzzle
excerpt_separator: <!--more-->
tags:
  - Vertex
  - Defcon CTF
  - Write-up
---



# DefconQUAL 2019 - Redacted-puzzle
풀이는 이미 올라왔으니 GIF Format에 대해 다뤘습니다
<!--more-->


 처음에 시도한 방법

1. 주어진 redacted-puzzle.gif에서 프레임 추출(gifgifs.com/split 이용) ![](https://i.imgur.com/f4kBpE1.png)
2. Stegsolve.jar을 이용해 Random colour map으로 이미지 변환 ![](https://i.imgur.com/oa8M9js.png) 
3. 을 모두 변환![](https://i.imgur.com/sfwToNX.png)

(????? 뭘까 도대체 ?????)



상당히 많은 사람들이 문제를 풀었었는데 도통 뭔질 모르겠어서 대회가 끝나고 라이트업을 확인했다.

답은 0번째 프레임에서 주어진 문자열의 개수가 32라는 힌트를 통해, 저 도형들이 8각형에 매칭되어 각 꼭짓점을 2진수로 변환한 후, 그 이진수를 5비트씩 쪼개서 문자열로 변환하면 플래그를 얻을 수 있는 문제였다. (플래그는 OOO{FORCES-GOVERN+TUBE+FRUIT_GROUP=FALLREMEMBER_WEATHER})



그런데 라이트업들을 확인하다가 의문점이 생겼다.

라이트업마다 추출된 프레임이 서로 다른 것이였다.

https://balsn.tw/ctf_writeup/20190513-defconctfqual/#redacted-puzzle

![](https://i.imgur.com/Sthvsgy.png)





https://klaus.hohenpoelz.de/def-con-ctf-qualifier-2019-redacted-puzzle.html

![](https://i.imgur.com/U4lFHvC.png)



왜 이런 차이가 발생하는 걸까?



프레임을 직접 추출할 때 생긴 의문은 '어? 나는 크기가 변하는 움짤은 본 적이 없는데 -_-' 였다.

당시에는 CTF 문제 출제를 위해 프레임별로 뭔가 어쩌구저쩌구를 해서 조작할 수 있겠지! 라며 가볍게 넘겼는데, 이번 기회에 GIF 포멧에 대해 조금 더 자세히 공부해보기로 했다.



추측해보건데, animated GIF는 변화된 부분만 프레임으로 저장해서 용량과 성능을 최적화 하는 기법이 있고, gifgif와 StegSolver는 저장된 프레임만, Python의 Pillow Image Library는 변화된 부분을 종합해서 프레임을 구성할 것 같았다.



# GIF란?

https://en.wikipedia.org/wiki/GIF#File_format

위키에 나와있는 내용을 요약하자면,

`GIF는 모뎀시절 매우 느린 인터넷 환경에서 RLE(Run-Length encoding 반복되는 문자를 표현하는 방법. ex)WWWWW -> 5W )를 대체하기 위해 고안, LZW압축 덕분에 인기를 끌게 되었고, 현재 사용중인 GIF 버전은 GIF87a와 GIF89a이다. 팔레트 기반 이미지 포멧으로, 팔레트 테이블에서는 최대 256개의 색만 적어두고 이 색들을 이용해 이미지를 표현한다. 움짤은 Graphics Control Extension 필드를 통해 지원하는데 89a밖에 없다. 여기에 크로마키를 통한 투명 기능도 추가. `



https://www.w3.org/Graphics/GIF/spec-gif89a.txt

정말 간단하게 파일 포멧을 표현하자면

| 섹션                    | 필드명                      | 사이즈 | 데이터예시 |
| ----------------------- | --------------------------- | ------ | ---------- |
| Header                  | Signature                   | 3      | "GIF"      |
|                         | Version                     | 3      | "89a"      |
| LogicalScreenDescriptor | Width                       | 2      | 1280       |
|                         | Height                      | 2      | 720        |
|                         | PackedFields                |        |            |
|                         | ==GlobalColorTable          | 1bit   |            |
|                         | ==ColorResolution           | 3bit   |            |
|                         | ==SortFlag                  | 1bit   |            |
|                         | ==SizeOfGlobalColorTable    | 3bit   |            |
|                         | BackGroundColorIndex        | 1      |            |
|                         | PixelAspectRatio            | 1      |            |
| GlobalColorTable        | RGB[0]                      |        |            |
|                         | ==R                         | 1      |            |
|                         | ==G                         | 1      |            |
|                         | ==B                         | 1      |            |
|                         | RGB[1]                      |        |            |
|                         | ==R                         | 1      |            |
|                         | ==G                         | 1      |            |
|                         | ==B                         | 1      |            |
|                         | ...                         |        |            |
| Data                    | Graphic Control Extension   |        |            |
|                         | ==Extension Introducer      | 1      |            |
|                         | ==Graphic Control Label     | 1      |            |
|                         | ==Block Size                | 1      |            |
|                         | ==PackedFields              |        |            |
|                         | ====Reserved                | 3bit   |            |
|                         | ====Disposal Method         | 3bit   |            |
|                         | ====User Input Flag         | 1bit   |            |
|                         | ====Transparent Color Flag  | 1bit   |            |
|                         | ==DelayTime                 | 2      |            |
|                         | ====TransParentColorIndex== | 1      |            |
|                         | ImageDescriptor             |        |            |
|                         | ==ImageSeperator            | 1      | 44         |
|                         | ==ImageLeftPosition         | 2      | 353        |
|                         | ==ImageTopPosition          | 2      | 56         |
|                         | ==ImageWidth                | 2      | 577        |
|                         | ==ImageHeight               | 2      | 608        |
|                         | ==PackedFields              |        |            |
|                         | ====LocalColorTableFlag     | 1bit   |            |
|                         | ====InterlaceFlag           | 1bit   |            |
|                         | ====SortFlag                | 1bit   |            |
|                         | ====Reserved                | 2bit   |            |
|                         | ====SizeOfLocalColorTable   | 3bit   |            |


이런식으로 파일이 구성된다.



특히 Data에서 Application Extension, Graphic Control Extension, Image Description, Plain Text Extension 등등이 Block 형태로 쭉 늘어지면서 다음에 등장할 압축 이미지의 정보를 명시한다.



# 결론

추출된 프레임의 차이는 프레임을 추출하는 프로그램이 Graphic Control Extension블록의 TransparentColorIndex를 처리 해주냐 안해주냐의 차이이다....
