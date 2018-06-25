---
layout: post
title: DPboard making and writeup
excerpt_separator: <!--more-->
tags:
  - MitNy
  - SQL injection
  - writeup
  - WebHacking
---

안녕하세요 MadHat 팀에서 웹 해킹을 공부하고 있는 `MitNy` 입니다. 지금까지 CTF나 Wargame에서 문제를 풀어오면서
문제를 푸는 것도 중요하지만 문제를 만들어보는 것도 중요하다는 생각을 하고 있습니다.
처음에는 스테가노그래피가 재밌어서 이것저것 숨겨보기도 했지만 요즘은 웹 문제를 만드는 게 더 재밌어졌습니다.
그래서 이번에는 제가 SQL injection 문제를 만드는 것과 풀이를 동시에 하도록 하겠습니다.

<!--more-->


### 1. DPboard 소개

문제 이름은 `DPboard` 입니다. DPboard는 Display board를 제 마음대로 줄인 것이며 기본적인 컨셉은
전광판에 사용자가 원하는 문구를 띄울 수 있다는 것입니다.
어디에 플래그가 뜰지 예상이 가시나요?

DPboard 문제의 메인 페이지는 다음과 같습니다.

![]({{ site.baseurl }}/images/mitny/DPboard/dp1.PNG)

메시지를 추가하고, 전광판에 표시를 하는 기능이 있고 지금까지 등록된 메시지들이 테이블 형식으로 보입니다.

메시지를 추가하게 되면 다음과 같이 테이블에 메시지가 추가됩니다.
![]({{ site.baseurl }}/images/mitny/DPboard/dp2.PNG)

`hey` 라는 메시지를 전광판에 띄워 볼게요.
![]({{ site.baseurl }}/images/mitny/DPboard/dp3.PNG)

표시할 메시지를 입력하면 URL 부분이 `ip/DPboard/dpboard.php?title=hey&id=hs6j3a53oiphdfrlni8j3fti83&submit2=send` 로 바뀝니다.
중간에 id는 익숙하네요. 유저의 PHPSESSID와 동일합니다.
![]({{ site.baseurl }}/images/mitny/DPboard/dp4.PNG)


이 부분의 코드를 보면

```html
<div id="showMessage">
	<form action="<?php echo $_SERVER['PHP_SELF']; ?>" method="get">
		표시할 메시지: <input type="text" name="title">
		<input type="hidden" name="id" value="<?php echo $_COOKIE["PHPSESSID"]; ?>">
		<input type="submit" name="submit2" value="send">
		</form>
</div>
```

hidden 값으로 유저의 `PHPSESSID`가 들어갑니다. 이를 이용한 PHP 코드는 다음과 같습니다.

```php
<?php
	$id = $_GET["id"];
	$title = $_GET["title"];

	$query = "SELECT id,content FROM dpdb WHERE id='$id' AND title='$title'";
	$result = mysqli_query($conn,$query);

	while($row = mysqli_fetch_array($result)) {
		if($_COOKIE["PHPSESSID"] != "m0jlh9fk2kdnsdsa2ai4bkal0t") {
			if($row['content'] == "FLAG(is not real)") {
				echo "<marquee>Do Not Hack!</marquee>";
			}
			else {
				echo "<marquee>".$row['content']."</marquee>";
			}
		}
		else {
			echo "<marquee>".$row['content']."</marquee>";
		}
	}
?>
```

title과 hidden값인 id를 가져온 후 id와 title에 해당하는 값을 DB에서 가져오는 쿼리문이 있습니다.
미리 DB에 저장해 둔 admin의 PHPSESSID와 비교하여 일치할 때만 플래그를 보여주도록 했습니다.
그렇지 않으면 substr로 한 글자만 맞춰도 플래그가 뜨기 때문에 너무 쉽잖아요!





