---
layout: post
title: ! "[TSGCTF2019] Obliterated File Write-up"
excerpt_separator: <!--more-->
comments : true
tags:
  - TSGCTF 2019
  - nalda
---

TSG CTF에 출제된 문제 write-up
<!--more-->

최근에 있었던 연휴에 진행된 CTF이다

```
Working on making a problem of TSG CTF, I noticed that I have staged and committed the flag file by mistake before I knew it. I googled and found the following commands, so I'm not sure but anyway typed them. It should be ok, right?
```
라는 내용과

```
$ git filter-branch --index-filter "git rm -f --ignore-unmatch problem/flag" --prune-empty -- --all
$ git reflog expire --expire=now --all
$ git gc --aggressive --prune=now
```
problem.zip파일이 있다.
이 파일을 다운받아보면

![]({{ site.baseurl }}/images/nalda/obliter/1.png)

해당 디렉토리에있는 README.md파일을 읽어보면
```bash
$ cat README.md
# easy_web

## Problem Statement

The flag is admin's password.
```

그렇다고 한다..!

캡쳐된 사진에는 2개의 파일밖에 보이지 않지만,
```
$ ls -al
total 32
drwxr-xr-x@  7 nalda  staff   224B  5 10 23:23 .
drwx------+  9 nalda  staff   288B  5 10 23:26 ..
-rw-r--r--@  1 nalda  staff   6.0K  5 10 23:25 .DS_Store
-rw-r--r--@  1 nalda  staff   150B  4 30 17:20 .editorconfig
drwxr-xr-x@ 14 nalda  staff   448B  5 10 23:29 .git
-rw-r--r--@  1 nalda  staff    64B  5  2 05:45 README.md
drwxr-xr-x@ 10 nalda  staff   320B  5  4 17:35 problem
```

여러가지 더 있다.
사실 출제자의 함정에 빠졌던것 중 하나는, problem폴더안에 뭔가 있을까 싶었으나
문제 description에 명시됐다시피 git rm을 때리셨다하니 git 관련돼서 조금 더 보려고 결심했다.

`git log`명령어를 입력하게 되면
```
commit d3953a7e9d5e89a07f767851721c09b543fe1a9b
    Author: tsgctf <info@tsg.ne.jp>
    Date:   Tue Apr 30 17:26:07 2019 +0900
    
         initial commit
```
대략 이런 내용들이 107줄이 있다.

사실 중요한건 commit 내용이기에 grep으로 commit을 잡아보았다.

```bash
$ git log | grep commit
commit 266f4148e4cf37bdbfb57da379ea49b2f106e6b2
commit cd50304fc39f8c0fbc7ad062ecb9a940f3baed29
commit ba46709ec62fd916b29f17c5e9fd2fa99b71027c
commit d516014b8de3f20d473f2adca1713337095c7873
commit 98d396f94fb23e9e0fb317aa041ca02691f7ec8b
commit 28d2b74b0c40583a87cf275f9f0cdfd55042884d
commit 84128ed70713706bef35805b2a097c1e5b493277
commit 39aa6c95cc229e828f4fb5115c2396c0a841eed4
commit 6b4cbce5f389a45bc849f07fa5c17a8b7f43f005
commit bff308624444eed4cac43b0d432a92d2d350fcfb
commit f4416accd32d3063630d243770ff6d1ba79ac209
commit b346b76e3642b0b33f5b17a19761b8d77276473b
commit b614e74c0d6db7c50c64a6f643c08e768308295c
commit 828b54e76c9ee94b1d9a478aef792726c60a01bc
commit 0f0a48cede1c8edb37b9449b7de0eb28402db1fc
commit 166baf8b5abaf404923426c08199e7396628e759
commit 4801d6ec013679a4cd8353812fa9502418ba6237
commit d3953a7e9d5e89a07f767851721c09b543fe1a9b
    initial commit
```

해당 commit에 대한 hash를 git show \<hash> 를 이용하여 내용을 확인할 수 있다.

```bash
$ git show 266f4148e4cf37bdbfb57da379ea49b2f106e6b2
      1 commit 266f4148e4cf37bdbfb57da379ea49b2f106e6b2 (HEAD -> master)
      2 Author: tsgctf <info@tsg.ne.jp>
      3 Date:   Fri May 3 04:33:22 2019 +0900
      4
      5     delete .travis.yml
      6
      7 diff --git a/.travis.yml b/.travis.yml
      8 deleted file mode 100644
      9 index ffc7b6a..0000000
     10 --- a/.travis.yml
     11 +++ /dev/null
     12 @@ -1 +0,0 @@
     13 -language: crystal
```

그래서 일일이 하나씩 다 찾아보기 귀찮았지만... cat * | grep flag도 안돼서....일일이 다 봤다..
생각보다 위에서 힌트를 받았지만...!
`28d2b74b0c40583a87cf275f9f0cdfd55042884d`을 보자



```bash
$ git show 28d2b74b0c40583a87cf275f9f0cdfd55042884d
      1 commit 28d2b74b0c40583a87cf275f9f0cdfd55042884d
      2 Author: tsgctf <info@tsg.ne.jp>
      3 Date:   Thu May 2 05:45:41 2019 +0900
      4
      5     add problem statement
      6
      7 diff --git a/README.md b/README.md
      8 index 60723b1..6eec6e5 100644
      9 --- a/README.md
     10 +++ b/README.md
     11 @@ -1,7 +1,5 @@
     12  # easy_web
     13
     14 -TODO: Write a description here
     15 +## Problem Statement
     16
     17 -## Usage
     18 -
     19 -TODO: Write usage instructions here
     20 +The flag is admin's password.
     21 diff --git a/flag b/flag
     22 deleted file mode 100644
     23 index 111eb96..0000000
     ...그다지 필요 없으니 중략...
```

이 commit에서 flag가 delete되었다...

이를 복구하기위해 이래저래 알아본결과 `git revert <commit hash>`를 입력하면 복구된다.

명령어를 실행 후

```bash
$ ls -al
total 64
drwxr-xr-x@ 12 nalda  staff   384B  5 10 23:41 .
drwx------+ 10 nalda  staff   320B  5 10 23:30 ..
-rw-r--r--@  1 nalda  staff   6.0K  5 10 23:30 .DS_Store
-rw-r--r--@  1 nalda  staff   150B  4 30 17:20 .editorconfig
drwxr-xr-x@ 14 nalda  staff   448B  5 10 23:41 .git
-rw-r--r--   1 nalda  staff    90B  5 10 23:41 README.md
-rw-r--r--   1 nalda  staff    62B  5 10 23:41 flag
-rw-r--r--   1 nalda  staff   482B  5 10 23:41 main.cr
drwxr-xr-x@  6 nalda  staff   192B  5 10 23:41 problem
-rw-r--r--   1 nalda  staff   507B  5 10 23:41 shard.lock
-rw-r--r--   1 nalda  staff   297B  5 10 23:41 shard.yml
drwxr-xr-x   5 nalda  staff   160B  5 10 23:41 src
```
오오 flag파일이 있다. 바로 캣!

```
$ cat flag
x�
  	vwq�V�O�,�/-HI,I�-JM��M�R���E��y�9�`^FjbJ�~nbqIjQ-��(
```
하..
```
$ file flag
flag: zlib compressed data
```

하하 재밋네 집파일로 zlip으로 압축도하시고 ㅎ

처음에는 .zip만 붙이면 되는줄알았으나 안되길래 자세히보니 zlib 이어서 당황했던건 안비밀 ㅎ

무튼 python으로 간단하게 풀었다
```
$ cat solv.py
import zlib
f = open('./flag')
print zlib.decompress(f.read())
```

### Flag
```bash
$ python solv.py
TSGCTF{$_git_update-ref_-d_refs/original/refs/heads/master}
```