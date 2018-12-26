---
layout: post
title: ! "[2018 X-MAS CTF] Special Christmas Wishlist"
comments: true
excerpt_separator: <!--more-->
tags:
  - Write-up
  - CTF
  - Crypto
  - MyriaBreak
---

`X-MAS CTF`에 나온 암호문제중 하나입니다. CTF는 12.14 ~ 12.21에 진행된 CTF로 크리스마스 분위기가 물씬 풍기는 CTF였습니다. (`Christmas` 당일까지 진행하지않은게 정말 다행이네요 ㅎㅎ;; )

<!--more-->

# Special Christmas Wishlist (50 pts)

### Description

>While Santa was looking through the wishlists of the childern all around the world he came across a very strange looking one. Help Santa decode the letter in order to fulfill the wishes of this child.
>
>(Flag is Non-Standard)
>
>UPDATE: flag is lowercase!
>
>_Author: Gabies_

Files:
- [wishlish.png](../images/myria/wishlist-writeup/wishlist.png)

문제는 조금 어렵게 나왔다고 생각했는데... 대회가 종료될때에 Solver가 124팀으로 점수는 50점까지 낮아졌습니다.

먼저 문제설명을 살펴보면 "산타가 전 세계 어린이들이 보내온 `Wishlist`를 보고있는데 이~상한 `Wishlist`를 발견했다고 합니다. 그 희망목록은 산타가 읽을 수 없는 문자들로 가득했는데...! 산타가 문자를 읽을 수 있게 도와 아이의 소원을 이루어주세요!" 같은 내용입니다.

그래서 주어진 첨부파일인 `wishlist.png`를 살펴보면 정말 알 수 없는 문자들로 가득 채워진 그림을 볼 수 있습니다. 이미지파일이 세로로 정말 길기 때문에 일부만 가져와보겠습니다.

![]({{ site.baseurl }}/images/myria/wishlist-writeup/wishlist_01.png)

이야... 진짜 무엇 하나도 키보드로 표현할 수 없는 문자들뿐입니다.
대체 이걸 어떻게 124팀이나 푼지 모르겠습니다 :(. 스코어에 등록된게 1378팀인데... 뭐 아무튼 이런 류의 문제들이 가끔식 CTF에서 나오는데요. 계속해서 못 풀다가 이번 기회에 그러면 아예 제대로 공부해놓자! 해서 이 문제를 가지고 Writeup을 작성하게되었습니다.

문제로 돌아와서 주어진 `wishlist.png`파일은 비아스키문자들과 공백으로 이루어진 문자가 73줄 나열되어있습니다. 또한 문자와 공백들의 나열을 보고 유추하건데,
이 문제는 [단순치환암호(Simple Substitution Cipher)](https://en.wikipedia.org/wiki/Substitution_cipher)일 것으로 보입니다.

그렇다면 이제 저 이미지의 모든 문자를 알파벳으로 치환한 후, 빈도수 분석을 이용하여 문제를 풀 수 있을 것으로 보입니다. 그런데 이미지의 문자 하나하나를 손으로 직접 바꾸는 것은 정말 어려운 문제입니다 ㅡㅡ;

저번에 있었던 `RITSEC CTF 2018`의 `Nobody uses the eggplant emoji` 에서는 위 문제와 마찬가지로 비아스키문자들로 이루어진 단순치환암호였지만 텍스트파일로 주어져 쉽게 알파벳으로 치환할 수 있었습니다.

![]({{ site.baseurl }}/images/myria/wishlist-writeup/wishlist_02.png)

> chal.txt
🤞👿🤓🥇🐼💩🤓🚫💪🤞🗣🙄🤓🥇🐼💩🤓😀✅😟🤓🍞🐼✅🚫💪🥇🤓🐼👿🤓🚫💪😟🤓👿😾😀😯🤓👿🤞✅🔥🚫🤓🥇🐼💩🤓👻💩🔥🚫🤓😀🗣🔥🍞😟✅🤓🚫💪😟🔥😟🤓🚫💪✅😟😟🤓💔💩😟🔥🚫🤞🐼🗣🔥😭🤓🍞💪😀🚫🤓🤞🔥🤓🥇🐼💩🤓🗣😀👻😟🤢🤓🍞💪😀🚫🤓🤞🔥🤓🥇🐼💩✅🤓💔💩😟🔥🚫🤢🤓🍞💪😀🚫🤓🤞🔥🤓🚫💪😟🤓😀🤞✅🤓🔥🐙😟😟😎🤓👀😟😾🐼🤬🤞🚫🥇🤓🐼👿🤓😀🗣🤓💩🗣😾😀😎😟🗣🤓🔥🍞😀😾😾🐼🍞😭🤓🥇🐼💩✅🤓👿😾😀😯🤓🤞🔥🤡🤓😀👿✅🤞🤬😀🗣_🐼✅_😟💩✅🐼🐙😟😀🗣_🔥🍞😀😾😾🐼🍞_🍞🐼🍞_🚫💪😟✅😟🔥_😀_😎🤞👿👿😟✅😟🗣🤬😟🤓
__

하지만 이번에 주어진 문제는 텍스트파일이 아닌 이미지파일로 주어졌기 때문에, 이미지파일에서 각 문자를 추출하여 아스키문자로 표현해줄 필요가 있습니다.

![]({{ site.baseurl }}/images/myria/wishlist-writeup/wishlist_03.png)

위와 같이 각 문자당 하나의 알파벳을 할당하는 것이 목표입니다.
이를 위해 파이썬 라이브러리인 `Pillow`를 사용하는데, Pillow는 PIL(Python Image Library)를 계승한 라이브러리입니다. 더이상 업데이트가 되지 않는 PIL 대신 사용하는 파이썬 라이브러리로 이를 통해 손쉽게 파이썬을 이용해서 이미지 파일을 다룰 수 있습니다.

```python
from PIL import Image
from string import ascii_lowercase

img = Image.open('wishlist.png').convert('1', dither=False)
lines = scan(img)
print(translate(lines))
```

먼저 PIL을 통해 Image를 불러옵니다. 옵션과 모드로 `1-bit-pixel mode` 와 `dither=False`를 선택하는데, 1-bit-pixel mode는 black인지 white인지만 판별하는 모드로 위 이미지의 문자는 흑백의 두 색으로만 이루어져있으므로 `1bit모드`로 충분합니다. 또 `dither`를 False로 설정하였는데
이는 이미지 프로세싱에서 많이 쓰이는 `디더링(Dithering)`이라는 것으로, 제한된 색을 이용하여 음영이나 색을 나타내는 것이며, 여러 컬러의 색을 최대한 맞추는 과정입니다.

이 디더(dither)를 False로 설정하면 음영을 구분하지않아 흑백으로만 그림이 나타날 것이므로 좀 더 편하게 문자열을 구분할 수 있습니다.

```python
def scan(img):
	lines = []
	line_x, line_y = 30, 0
	while True:
		x, y = search(img, line_x, line_y, True, fix_x=True)
		if (x, y)==(None, None): break
		line = []
		while True:
			x, y = search(img, x, y, True, fix_y=True)
			if (x, y)==(None, None): break
			letter = bfs(img, x, y)
			line.append(normalize(letter))
			(min_x, min_y), (max_x, max_y) = bounds(letter)
			x = max_x + 1
			y = (max_y + min_y) // 2
		lines.append(line)
		line_y = max_y + 30
	return lines
```

이제 맨 윗줄부터 왼쪽에서 오른쪽으로 이동하면서 이미지 위의 문자를 찾습니다.
여기서 중요한 점은 이미지 위에 나타난 `모든 문자들이 끓어지지않고 연속된 선`으로 이루어져있다는 것입니다. 그러므로 이를 [너비우선탐색(BFS)](https://en.wikipedia.org/wiki/Breadth-first_search)를 통해 각 문자에 대해 트리를 만들수 있습니다.

![]({{ site.baseurl }}/images/myria/wishlist-writeup/wishlist_01.png)

```python
def bfs(img, start_x, start_y):
	queue = [(start_x, start_y)]
	visited = set(queue)
	for x, y in queue:
		for dx, dy in [(-1, -1), (-1, 0), (-1, 1), (0, -1), (0, 1), (1, -1), (1, 0), (1, 1)]:
			xx, yy = x + dx, y + dy
			if img.getpixel((xx, yy)) > 0:
				continue
			if (xx, yy) not in visited:
				queue.append((xx, yy))
				visited.add((xx, yy))
	return visited

```

이제 각 줄의 문자마다 만들어진 트리들을 가지고 같은 트리에는 같은 문자를 할당하여 나타내면 됩니다.

```python
def translate(lines):
	alpha = dict()
	index = 0
	text = ''
	for line in lines:
		for letter in line:
			if letter not in alpha:
				alpha[letter] = ascii_lowercase[index]
				index += 1
			text += alpha[letter]
		text += '\n'
	return text
```

이제 이 소스코드를 실행하면 아래와 같은 결과를 얻을 수 있습니다.

```
abcdcefgahijdcefgkhelgldji
jdmbnngnbodapqhhrgifl
kcoaqggmjabllgllgchncsh
qbffhjsdlfhoceoqagml
bftgicemgmoeacdchhauadvsbcuk
csdjobmlkobaahslrgsgm
iglcdijlchmbjguhicbdigml
lodadijwdxhjbmfgilueavcemg
ahijfdlcbiugcheukabov
oeacduhahmhoqmglcgoagllsdigjablllgc
vgfglcbawgsgampkhafgm
bmcdlbibaqboqhhlbacukglc
bemhmblobmcadjkcdijvbigal
ckglgblgmvgicjbmfgilueavcemg
ckgyedcgbobxdijyeglcnhmyegmdgl
mgupuagfjabllcmggjahqglmgabcdhilkdvl
igsphmrcdogluelchonmhicvbjgvexxag
ckgqglcladijqgtgmbjguhhagm
cmdhodzgfogcbalgbmmdijl
qdmckohicknahsgmigurabug
bohckgmlahtgdlqgphifogblemglvhhilgc
sgffdijsbacxvgmlhibadxgfbmc
vgmlhibadxgfjhhfidjkcadccagogqhhr
uelchovgcihlgvmdicigurabugl
uelchovgcvdaahsl
ysgmcprgpqhbmf
aabobmbobabmjgxdvvgmvheuk
odohlbfdbjmbojabllsbmglgchnckmgg
ckbirphenhmphemvbmcdiopwhemigpigurabug
phjbvhlgkbijdijlueavcemgl
ckgqdrgukbdiqhsa
umdolhikgbmceoqmgaab
ckgyedxdildfgckgobxg
ogilkgmqbasbmodijladvvgml
phemlodigbifhemlgijmbtgffgubicgmlgc
ckgbiidtgmlbmpwhemiba
ohoobqdmfuenn
sdigqbmmgajedcbmmbur
ckgvaelkhmjbil
jbmadujmbcgmbifhdafdvvdijfdlk
hrchqgmnglcbagqggmqmgsdijrdc
obcglnhmadng
qhhrahtgmladjkcsgdjkclubmtgl
uhilcdcecdhihneidcgflcbcglhnbogmdubjabll
jhanjabllglvgscgmbijgauhdillgchncsgatg
qeiipngacqbqpladvvgml
vgmlhibadxgfcmggcmeirjabllsbmgfeh
gzvgucdijphebrggvlbrgvmgjibiupwhemiba
ckgqgmmpqeffp
ckgqdmfdgpbmiqhsa
rbickbukbifgadgmlgbmmdijl
vgmlhibadxgfoptgmphsiibogqhhr
eiduhmibifmbdiqhsodlobcukgfgbmmdijl
lahckvbalohqdag
qbchibqmbiuk
vgmlhibadxgfueccdijqhbmf
lohrgfgcguchmfgbucdtbcdhichsga
ckgaeigadjkc
vgmlhibadxgfbiidtgmlbmpvelkvdielbobv
ogilcbuhlhurl
gashhfckgeiduhmiugmgbaqhsa
ckgadccagvbcdgic
udmuaghnnbodapbifnmdgifllgmtdijqhsa
jabllnahsgmjbmfgiugicgmvdgug
qdmcklchigodigmbalhbvl
lgchnnhemjhanjabllgl
ckgugfbmckeoqvdbihl
uelchoobvuhblcgmlgc
qdjvgmlhibadcpfglrldjil
lgblchiglvablklvhijgkhafgm
phemohlcfgldmgfhqwgucckdlukmdlobl
ckgnabjdl
zoblphebmglhjhhfbcleqlcdcecdhiudvkgml
```

자 그럼 이제 분도수 분석을 통해, 단순치환암호를 풀면 됩니다. [quipquip](https://quipqiup.com/)를 이용하면 쉽게 단순치환암호문제를 풀 수 있습니다.

> latitude longitude house sign giraffe family book ends html beer glasses set of two bad dog wisdom tumblers adventurer multi tool clip watch twig marsh mallow skewer nesting storage containers smiling jizo garden sculpture long distance touch lamp multi color ombre stem less wine glass set pedestal jewelry holder artisan al bamboo salt chest aurora smart lighting panels the sea serpent garden sculpture the quite amazing quest for queries recycled glass tree globes relationships new york times custom front page puzzle the best sling beverage cooler trio mixed metals earrings birth month flower necklace a mothers love is beyond measure spoon set wedding waltz personalized art personalized good night little me book custom pet nose print necklaces custom pet pillows qwerty keyboard llama rama large zipper pouch mimosa diagram glass ware set of three thank you for your part in my journey necklace yoga pose hanging sculptures the bike chain bowl crimson heart umbrella the quiz inside the maze mens herbal warming slippers yours mine and ours engraved decanter set the anniversary journal momma bird cuff wine barrel guitar rack the plush organs garlic grater and oil dipping dish oktober festale beer brewing kit mates for life book lovers lightweights carves constitution of united states of america glass golf glasses pewter angel coins set of twelve bunny felt baby slippers personalized tree trunk glass ware duo expecting you a keepsake pregnancy journal the berry buddy the birdie yarn bowl kantha chandeliers earrings personalized my very own name book unicorn and rainbow mismatched earrings sloth pals mobile baton a branch personalized cutting board smoke detector deactivation towel the lune light personalized anniversary push pinus a map mens taco socks elwood the unicorn cereal bowl the little patient circle of family and friends serving bowl glass flower garden centerpiece birth stone mineral soaps set of four golf glasses the cedar thumb pianos custom map coaster set big personality desk signs sea stone splash sponge holder your most desired object this chrism as the flag is xmas you are so good at substitution ciphers


FLAG는 마지막 줄에 있습니다. `flag is xmas you are so good at substitution ciphers`


### 전체소스

```Python
#!/usr/bin/python3

from PIL import Image
from string import ascii_lowercase


def search(img, start_x, start_y, need_color, fix_x = False, fix_y = False):
	found = False
	for y in range(start_y, img.height) if not fix_y else [start_y]:
		for x in range(start_x, img.width) if not fix_x else [start_x]:
			color = (img.getpixel((x, y)) == 0)
			if (color and need_color) or (not color and not need_color):
				found = True
				break
		if found:
			break

	if found:
		return (x,y)
	else:
		return (None, None)


def bfs(img, start_x, start_y):
	queue = [(start_x, start_y)]
	visited = set(queue)
	for x, y in queue:
		for dx, dy in [(-1, -1), (-1, 0), (-1, 1), (0, -1), (0, 1), (1, -1), (1, 0), (1, 1)]:
			xx, yy = x + dx, y + dy
			if img.getpixel((xx, yy)) > 0:
				continue
			if (xx, yy) not in visited:
				queue.append((xx, yy))
				visited.add((xx, yy))
	return visited


# search google : python zip(*list)
def bounds(letter):
	x, y = zip(*list(letter))
	min_x, min_y = min(x), min(y)
	max_x, max_y = max(x), max(y)
	return (min_x, min_y), (max_x, max_y)


def normalize(letter):
	(min_x, min_y), (max_x, max_y) = bounds(letter)
	normal = set()
	for x, y in letter:
		xx, yy = x - min_x, y - min_y
		normal.add((xx, yy))
	return frozenset(normal)

def scan(img):
	lines = []
	line_x, line_y = 30, 0
	while True:
		x, y = search(img, line_x, line_y, True, fix_x=True)
		if (x, y)==(None, None): break
		line = []
		while True:
			x, y = search(img, x, y, True, fix_y=True)
			if (x, y)==(None, None): break
			letter = bfs(img, x, y)
			line.append(normalize(letter))
			(min_x, min_y), (max_x, max_y) = bounds(letter)
			x = max_x + 1
			y = (max_y + min_y) // 2
		lines.append(line)
		line_y = max_y + 30
	return lines


def translate(lines):
	alpha = dict()
	index = 0
	text = ''
	for line in lines:
		for letter in line:
			if letter not in alpha:
				alpha[letter] = ascii_lowercase[index]
				index += 1
			text += alpha[letter]
		text += '\n'
	return text


def main():
	img = Image.open('wishlist.png').convert('1', dither=False)
	lines = scan(img)
	print(translate(lines))

if __name__ == '__main__':
	main()

```
