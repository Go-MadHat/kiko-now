---
layout : post
title: ! "Classic passwords created with Python 3.x Series I(Caesar Crypto)"
tags:
- crypto
- caesar_crypto
- caesar
- python
- python3
- python_3.x
- S.Young
- Just for SoJoo.Young
- 컴린이
- go-madhat
- MADHAT
- Team MADHAT
---

안녕하세요. Team MADHAT의 클린한~ 컴린이! S.Young입니다.

저는 암호학을 공부하고 있는데 왠지 컴퓨터 해킹과 해커의 시조인 암호학이 다소 인기가 없는 것 같네요..

그래서 이번 기회에 간단하게 파이썬 3.x를 이용해서 고전암호의 암호화와 복호화하는 법을 시리즈로 소개하고자 합니다.


**시저암호(카이사르 암호)**
------
* 시저암호(카이사르 암호)란 ? 

고대 로마의 황제이자 군인었던 카이사르가 즐겨 사용했던 암호 알고리즘입니다.

그래서 시저암호(카이사르 암호)라고 불립니다.

대표적인 치환형 암호이자 덧셈형 암호입니다.

수식은 **C=E(p)=(p+k)mod(26)** 또는 **f(x)=x+k(mod26)**

복호화는 함수f(x)의 역함수를 구하면됩니다 **f(x)^-1=x-k(mod26)**

파이썬 Tkinter을 사용해 GUI구현을 해보았습니다
<pre><code>
import tkinter as tk
from tkinter import messagebox
import time

root = tk.Tk()
WIDTH = 500
HEIGHT = 600
root.geometry("{}x{}".format(WIDTH,HEIGHT))
root.title("카이사르 암호")
alphabet = "abcdefghijklmnopqrstuvwxyz"

def encode_text():
    text_to_encode = text.get("1.0","end")
    key = key_entry.get()
    encoded_text = ""

    if key == "" or not key.isdigit():
        messagebox.showwarning("Key Error", "정수값의 key 값을 입력해주세요")
        key_entry.delete(0,"end")
    else:
        for letter in text_to_encode:
            if letter.lower() not in alphabet:
                encoded_text += letter
            else:
                encoded_text += alphabet[(alphabet.index(letter.lower()) + int(key)%len(alphabet))%len(alphabet)]

    time.sleep(1)
    text.delete("1.0","end")
    key_entry.delete(0,"end")
    messagebox.showwarning("암호화 완료","암호화 완료")
    key_entry.insert(0,"")
    text.insert("1.0",encoded_text)

def decode_text():
    text_to_encode = text.get("1.0", "end")
    key = key_entry.get()
    decoded_text = ""

    if key == "" or not key.isdigit():
        messagebox.showwarning("Key Error", "정수값의 key 값을 입력해주세요")
        key_entry.delete(0, "end")
    else:
        for letter in text_to_encode:
            if letter.lower() not in alphabet:
                decoded_text += letter
            else:
                decoded_text += alphabet[(alphabet.index(letter.lower()) - int(key)%26) % 26]

    time.sleep(1)
    text.delete("1.0", "end")
    key_entry.delete(0, "end")
    messagebox.showwarning("복호화 완료", "복호화 완료")
    key_entry.insert(0, "")
    text.insert("1.0", decoded_text)

key_label = tk.Label(root, text = "KEY")
key_entry = tk.Entry(root)
text_lavel = tk.Label(root, text = "TEXT")
text = tk.Text(root, height = 30)
encode_btn = tk.Button(root, text = "Encode Text", command = encode_text)
decode_btn = tk.Button(root, text = "DECODE Text", command = decode_text)
quit_btn = tk.Button(root, text = "QUIT", command = quit)

key_label.pack()
key_entry.pack()
text_lavel.pack()
text.pack()
encode_btn.pack(fill = "x")
decode_btn.pack(fill = "x")
quit_btn.pack(fill = "x")

root.mainloop()
</code></pre>


위의 소스코드는 실은 대부분이 GUI구현을 위한 것이고 실질적 암호 알고리즘은 아래 정도면 충분합니다.
<pre><code>
#평문 입력
string = input("문장을 입력하세요 > ")
#키값 입력
key = int(input("키값을 입력하세요 > "))
#알파벳
alphabet = "abcdefghijklmnopqrstuvwxyz"
new_text = " "
letter = " "
#암호화 알고리즘 함수
def caesar(some):
    global alphabet
    global new_text
    global letter
    for letter in some.lower():
        if letter == " ":
            new_text += " "
        else:
            #알고리즘
            new_text += alphabet[(alphabet.index(letter) + int(key%len(alphabet)))%len(alphabet)]
    return new_text
print("알고리즘 실행", caesar(string))
</code></pre>

아스키코드를 사용하면 더 짧아집니다.
<pre><code>
string = input(" ▶ 문장을 입력하세요 : ")
key = int(input(" ▶ key 값을 입력하세요 : "))
print(">>> password : ",''.join([chr(ord(i)+key) for i in string]))
</code></pre>

**원리**
------
<img src="https://github.com/go-madhat/go-madhat.github.io/blob/master/images/Classic%20passwords%20created%20with%20Python%203.x%20Series%20I(Caesar%20Crypto)/caesar_cipher_left_shift_of_19_circle.png" width="40%">

아주 간단하게도 그림에서 보시는대로 두 문자열을 서로 맞댄 후 한 문자열을 이동한 것입니다.
이동한 칸 수가 바로 우리가 키값이라고 부르는 그것이죠.

![caesar_cipher_left_shift_of_3-svg](https://github.com/go-madhat/go-madhat.github.io/blob/master/images/Classic%20passwords%20created%20with%20Python%203.x%20Series%20I(Caesar%20Crypto)/caesar_cipher_left_shift_of_3-svg.png)

A B C **D** E F G H I J K
    
=> > >

__a__ b c d e f g h i j k

키값 3 입력 == 3칸 이동

************
보다시피 암호 알고리즘이 널리 알려져있고 키값의 경우의 수가 고작 25가지 밖에 안되기 때문에 복호화시 그냥 경우의 수를 다 때려박아서 암호해제가 가능하고 실제로 그렇게 복호화해서 손쉽게 키값과 평문을 역추적하는 쉬운? 암호 알고리즘입니다.

딱히 설명할 것도 없이 너무나 쉽다?보니 암호 알고리즘에 입문하기 아주 좋은 예제라고 생각되는데요.

다음번에는 파이썬3.x를 이용해 다 때려박아서 시저암호(카이사르 암호)를 복호화는 법(전수대입)과 대표적인 치환형암호인 시저암호(카이사르 암호)보다 조금 더 복잡한 치환형암호를 소개하겠습니다.

긴 글 읽어주셔서 감사합니다.
