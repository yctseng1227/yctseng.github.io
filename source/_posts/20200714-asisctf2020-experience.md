---
title: ASIS CTF Quals 2020 Experience
date: 2020-07-07 15:10:32
categories: 
- Security
tags:
- CTF
- ASIS CTF

mathjax: true
---

暑假第一場CTF online!!! 同時也是我們**MacacaHub**的今年第三戰~
不過同時也被隊長告知有一定的難度要有心理準備，嗯、真滴難QwQ
題組全部共31題，雖然只解 4 題還有 Rank 151 / 815 ... 太可怕了==

[CTFtime link](https://ctftime.org/event/964)

<!--more-->

![ ](https://photos.smugmug.com/photos/i-pKzZ7bB/0/09d51399/X3/i-pKzZ7bB-X3.png)

## 寫在前面的話

從上圖可以看得出來，我們解出來的四題扣掉 `Welcome` 和 `Survey`(feedback)，實際答出來的只有兩題。然而還是有兩支隊伍破台，別忘了這場還是Quals(資格賽)，人外有人天外有天呢...

既然只摸出兩題，這篇標題自然沒辦法放 "**writeup**"，所以就當做見見世面發些心得，同時針對一些賽中我認為很有趣的題目稍微參考別人解法寫一下，其實有幾題看完題解發現都不太難，只是有沒有發現而已QQ。


## Mechanism: Proof of Work

上次遇到`PoW`是 **AIS3 EOF CTF 2019** 的 [Ponzi Scheme](https://github.com/MacacaHub/CTF-writeups/blob/master/AIS3%20EOF%20CTF%202019/Ponzi%20Scheme/readme.md)，這技術常見於區塊鏈加密貨幣的機制，是一種對應服務與資源濫用、或是阻斷服務攻擊(DoS)的經濟對策。要求使用者進行一些<font color=red>耗時適當的複雜運算</font>，並且答案能被服務方快速驗算，以此耗用的時間、裝置與能源做為擔保成本，以確保服務與資源是被真正的需求所使用。

所以某方面也算是官方針對`nc`題目一種自我保護的機制，避免被大量玩家戳一戳就掛掉。另一方面，我自己猜可能有些題目可以用brute force的爆出答案，避免這種情況才加了這項機制。總之，雖然並非主要考點，但如果過不了也沒辦法解題ㄚ xD。雖然並不是每題`nc`題目都有`PoW`，不過翻了翻`ASIS CTF`往年的題目只要有出現`PoW`的都不太一樣，沒辦法直接抄來用QQ

￬￬￬ 以這場來看，`PoW`描述如下 ￬￬￬

{% codeblock lang:shell line_number:False %}
$ nc [ip] [port]
> Please submit a printable string X, such that sha224(X)[-6:] = 6be655 and len(X) = 19
haha
> You must pass this PoW challenge :P
{% endcodeblock %}

翻譯一下：系統要求輸入長度為19的字串`X`，同時要滿足 sha224(X) 後6位為 6be655。
其中，`Hash Algorithm`的部分每次連線結果都不同，隨意測試就有sha1, sha224, sha256, sha384, sha512, md5 ...ect，而且不同往年`ASIS CTF`多了指定`輸入長度len(x)`，同樣每次連線結果不同，測出來Range大約 10 - 40。

根據`Hash`的特性，雖然因為**不可逆**沒辦法從後6位往回推，但`相同的輸入`一定會得到`相同的輸出`，那我們其實可以暴力產生指定長度的輸入，拿去hash再看看有沒有符合後6位的條件就行了。

個人的做法是連線後用字串處理拿到 `Hash Algorithm` 和 `length` ，只針對 `sha256` 且長度為 `10`的PoW直接爆搜字串進行驗證，簡單用python表示如下:

{% codeblock solve.py lang:python %}
from pwn import *
import hashlib
import string

def pow_solve(s): 
    # Blog排版請忽略縮排
    for a in string.printable:
    for b in string.printable:
    for c in string.printable:
    for d in string.printable:
    for e in string.printable:
    for f in string.printable:
    for g in string.printable:
    for h in string.printable:
    for i in string.printable:
    for j in string.printable:
        if hashlib.sha256(a+b+c+d+e+f+g+h+i+j).hexdigest()[-6:] == s:
            return a+b+c+d+e+f+g+h+i+j

# 通常1000次內一定會有 sha256 & len=10
for i in range(1000):
    r = remote(host, port)

    # 省略字串處理過程
    hash_algo = r.recv(6)
    if s == 'sha256':
        target = r.recv(6)
        length = r.recv(2)
        if int(length) == 10:
            r.sendline(pow_solve(target))
            r.interactive()
            #solve()
    r.close()
{% endcodeblock%}

我知道一定會有人想吐槽那個10層迴圈，不過不想花時間刻DFS這樣寫最直觀 XD
賽後在網路上翻到其他玩家的作法如下，趕緊存起來!!  [source link](https://gist.github.com/saurav3199/a61cada35112ebfd2a9a0d218926bb36)

{% codeblock pow_solve.py lang:python %}
#!/usr/bin/python3
import hashlib
import uuid
from pwn import *
#import string

r = remote("76.74.178.201", 8002)

info = r.recvuntil("\n").decode("UTF-8").split(" ")
print(info)
algo = info[8].split("(")[0]
target = info[10]
length = int(info[-1])
print(f"[*] algo needed: {algo}")
print(f"[*] target needed: {target}")
print(f"[*] length needed: {length}")

def random_ascii_generator(l):
    string = str(uuid.uuid4()).replace("-", "")[0:l].encode("ascii")
    return string[0:l]

algo_matching = {
    "sha1": hashlib.sha1,
    "md5": hashlib.md5,
    "sha224": hashlib.sha224,
    "sha256": hashlib.sha256,
    "sha384": hashlib.sha384,
    "sha512": hashlib.sha512,
}

candidate_hash = ""
while not candidate_hash[-6:] == target:
    candidate_string = random_ascii_generator(length)
    candidate_hash = algo_matching[algo](candidate_string).hexdigest()

print("[*] PoW found")
r.sendline(candidate_string)

print(r.recvline())
print(r.recvline())
print(r.recvline())
print(r.recvline())
print(r.recvline())
print(r.recvline())
{% endcodeblock %}

執行看起來就簡潔有力，好羨慕哦 >///<
![](https://photos.smugmug.com/photos/i-ks37Bs5/0/e28a34f7/X3/i-ks37Bs5-X3.png)

噢對、這題是[PPC-Titanic](/20200714-asisctf2020-experience/#PPC-Titanic-Unsolved)，等等也會提到。

## [Web] Web Warm-up (Solved)

Warm up! Can you break all the tasks? I'll pray for you!
read <font color="#F4A8B8">flag.php</font>
Link: Link(http://69.90.132.196:5003/?view-source)

{% codeblock lang:php %}
<?php
if(isset($_GET['view-source'])){
    highlight_file(__FILE__);
    die();
}

if(isset($_GET['warmup'])){
    if(!preg_match('/[A-Za-z]/is',$_GET['warmup']) && strlen($_GET['warmup']) <= 60) {
    eval($_GET['warmup']);
    }else{
        die("Try harder!");
    }
}else{
    die("No param given");
}
{% endcodeblock %}

從上方 source code 可以用 GET `/warmup=`，想辦法存取`flag.php`，不過限制就是無法使用英文字母以及payload長度必須小於60字元。

重點在於`preg_match`繞過，關鍵字拿去查找到 [ctf中 preg_match 绕过技术 | 无字母数字的webshell](https://www.cnblogs.com/v01cano/p/11736722.html)。
其中的`解法1`就是答案了，簡單講就是利用"特殊字元"互相XOR出"字母"，舉例來說:

{% codeblock lang:python line_number:False %}
>>> chr(ord('m')^ord('@'))
'-'
{% endcodeblock%}

表示 `@` ^ `-` = `m`，最後只差在該如何構造出payload，畢竟我對`php`沒有到很熟 QQ
在參考其他ctf writeup想到的解法是
```
$_=']),>'^';@@[';        // file
$__='\],::'^'{;@[]';     // 'flag
$___='-@-\\'^'](]{';     // php'
$_($__.$___);            // file('flag.php')

payload:
$_=']),>'^';@@[';$__='\],::'^'{;@[]';$___='-@-\\'^'](]{';$_($__.$___);
```
無奈長度達69沒辦法繞過第二項條件。

另外用 `$_='-@-)@]@'^'](]@.;/';$_();` 拼出 `phpinfo();`後可以觀察到有趣的資訊...
![](https://photos.smugmug.com/photos/i-kcdSjB3/0/33d533e2/X3/i-kcdSjB3-X3.png)

好吧好像一點也不有趣，對方伺服器擋了很多方便的系統函數，所以**stavhaygn**也戳了不少payload都落空。

最後拼出來的答案其實很簡單... 就是`readfile('flag.php');`
長度方面 `('[>:@]),>'^')[[$;@@[')('],::.-@-'^';@[]%00](]');` 為49，可喜可賀。

## [Web] Treasury #1 (Unsolved)

從題目點開連結後畫面如下，其中`AE`點開會跳出小視窗顯示書本內容，而`RO`會開新視窗連到書本來源。

![ ](https://photos.smugmug.com/photos/i-Z8qc8gK/0/b706838c/XL/i-Z8qc8gK-XL.png)

我們從`treasury.js`可以看到抓取書本內容的方式為`/books.php?type=excerpt&id=xx`，其中`xx`就對應畫面中出現的三本書id分別為1, 2, 3。

![ ](https://photos.smugmug.com/photos/i-LP48sQ2/0/3072d20b/XL/i-LP48sQ2-XL.png)

然後、就沒有然後了。
原因是我們顧著用整數戳`id`，或是想辦法找其他入口，忽略字串形式的方法（事實上我也沒戳很久就是了）。賽後看完Writeup，感覺自己缺少了很多這方面的瞭解，就算找到SQLi可能還是沒辦法戳出來QwQ

以下我大概整理 [writeup](https://drive.google.com/file/d/1SrkvadmPHMpK66rYfcrDDphDSQVg96et/view) 的解題思路：
##### a. 確定SQLi漏洞
透過`id=1' and 1=1; --%20` 、 `id=1' and 1=2; --%20`，前者成功回傳資料，後者沒有任何顯示，可知該**command後面的statement成功被執行**。
P.S. 要注意`--`註解後要多個空白`%20`，否則會執行失敗

##### b. 嘗試用`UNION`搭配其他SQL command
從`id=9' union select 'hello'; --%20`發現回傳的Warning資訊是關於`simplexml_load_string()`，同時也間接表示可能有`XXE`的漏洞。

##### c. 確認資料庫種類和版本
從`id=9' union select @@version; --%20`瞭解資料庫為`MariaDB`及版本`10.5.4`，然後writeup提供 https://sqliteonline.com/ 可以線上測試資料庫指令。

##### d. 逐步猜資料庫欄位
之後有點像`sqlmap`從已知的資訊開始把資料庫的內容還原出來，搜尋"mariadb enumerate columns"。
`select column_name from information_schema.COLUMNS where table_name='books' limit 1,1; --`: 其中`information_schema.COLUMNS`為一個View Table物件，另外`limit`可以限制開始位置(zero based)及列出筆數。

##### e. 針對XML structure逐步還原資料
writeup中是利用`substr(str, pos, len)`嘗試抓XML資料，從錯誤提示把tag慢慢拼湊起來，如下。
```
<book>
<id>1</id>
<name>blah</name>
<author>blah</author>
<year>blah</year>
<link>blah</link>
<flag>blah</flag>
<excerpt>blah</excerpt>
</book>
```

`concat('<book><excerpt>',replace(info, '<', ''),'</excerpt></book>')`
最後為了讓所有資訊都能完整顯示(包含tag)，writeup使用`cancat`連結所有字串 及 `replace('<','')`把左角括號拿掉使tag渲染失敗，結束。

至於 **Treasury #2** 就是接下去利用`XXE`撈flag。

## [Crypto] Baby RSA (Unsolved)

All babies love [RSA] (↓). How about you?

{% codeblock baby_rsa.py lang:python %}
#!/usr/bin/python

from Crypto.Util.number import *
import random
from flag import flag

nbit = 512
while True:
    p = getPrime(nbit)
    q = getPrime(nbit)
    e, n = 65537, p*q
    phi = (p-1)*(q-1)
    d = inverse(e, phi)
    r = random.randint(12, 19)
    if (d-1) % (1 << r) == 0:
        break

s, t = random.randint(1, min(p, q)), random.randint(1, min(p, q))
t_p = pow(s*p + 1, (d-1)/(1 << r), n)
t_q = pow(t*q + 4, (d-1)/(1 << r), n)

print 'n =', n
print 't_p =', t_p
print 't_q =', t_q
print 'enc =', pow(bytes_to_long(flag), e, n)
{% endcodeblock %}


{% codeblock output.txt line_number:False %}
n = 10594734342063566757448883321293669290587889620265586736339477212834603215495912433611144868846006156969270740855007264519632640641698642134252272607634933572167074297087706060885814882562940246513589425206930711731882822983635474686630558630207534121750609979878270286275038737837128131581881266426871686835017263726047271960106044197708707310947840827099436585066447299264829120559315794262731576114771746189786467883424574016648249716997628251427198814515283524719060137118861718653529700994985114658591731819116128152893001811343820147174516271545881541496467750752863683867477159692651266291345654483269128390649
t_p = 4519048305944870673996667250268978888991017018344606790335970757895844518537213438462551754870798014432500599516098452334333141083371363892434537397146761661356351987492551545141544282333284496356154689853566589087098714992334239545021777497521910627396112225599188792518283722610007089616240235553136331948312118820778466109157166814076918897321333302212037091468294236737664634236652872694643742513694231865411343972158511561161110552791654692064067926570244885476257516034078495033460959374008589773105321047878659565315394819180209475120634087455397672140885519817817257776910144945634993354823069305663576529148
t_q = 4223555135826151977468024279774194480800715262404098289320039500346723919877497179817129350823600662852132753483649104908356177392498638581546631861434234853762982271617144142856310134474982641587194459504721444158968027785611189945247212188754878851655525470022211101581388965272172510931958506487803857506055606348311364630088719304677522811373637015860200879231944374131649311811899458517619132770984593620802230131001429508873143491237281184088018483168411150471501405713386021109286000921074215502701541654045498583231623256365217713761284163181132635382837375055449383413664576886036963978338681516186909796419
enc = 5548605244436176056181226780712792626658031554693210613227037883659685322461405771085980865371756818537836556724405699867834352918413810459894692455739712787293493925926704951363016528075548052788176859617001319579989667391737106534619373230550539705242471496840327096240228287029720859133747702679648464160040864448646353875953946451194177148020357408296263967558099653116183721335233575474288724063742809047676165474538954797346185329962114447585306058828989433687341976816521575673147671067412234404782485540629504019524293885245673723057009189296634321892220944915880530683285446919795527111871615036653620565630
{% endcodeblock %}


題目給了一份 `code` 和 `output.txt`，要求把`flag`推出來。

看起來還算是個資訊很多的crypto題，所以我一開始完全忽略line 18 - 20，只想著能不能從`output.txt`把flag拼出來。後來也發現`n`根本沒辦法透過 [factordb.com](http://www.factordb.com/index.php) 拆成`p`, `q`，某方面來說也是蠻現實的題目：給予密文`enc`、公鑰`(e, n)`，不過多了奇怪的線索`t_p`, `t_q`。

參考 [writeup](https://github.com/networknerd/CTF_Writeups/blob/master/2020/ASISCTF_2020/Crypto/BabyRSA/README.md)

前半部`While`是標準的`RSA`，從writeup來看關鍵就在line 18 - 20... 嗯、聽起來是廢話，但我當初忽略的原因在於line 18的`s`, `t`完全是透過`p`, `q`隨機出來的數字，就算知道`n`可以間接固定`p`, `q`，但在爆不出來的情況下更遑論推出`d`，所以花了很多時間在亂推式子= =。

真正的關鍵點在line 19: `t_p = pow(s*p + 1, (d-1)/(1 << r), n)`，在已經知道 $t_p$ 的前提可以整理如下:

$$t_p = (s*p + 1) ^ {(d-1)/(1<<r)}$$

再來writeup說明算式右邊部分除了 $1$ 以外都可以被 $p$ 整除，我自己最後的理解如下。
令 $x = s * p$ ，在 $(x + 1) ^ k$ 的情況下把 $k$ 用 $2, 3$ 代入看看：

$$(x+1)^2 = x^2 + 2*x*1 + 1^2$$
$$(x+1)^3 = x^3 + 3*x^2*1^1 + 3*x^1*1^2 + 1^3$$

根據上述兩式，若 $x$ 能夠被 $p$ 整除，那會得到 $(x+1)^k = 1\ mod\ p $，回到原式可知
$$t_p\ mod\  p = (s*p + 1) ^ {(d-1)/(1<<r)}\ mod\  p = 1 ， \therefore t_p = 1\ mod\ p$$

這下我們可以確定 $t_p-1$ 是 $p$ 的倍數，同時由於 $n=p*q$ ， $n$ 也是 $p$ 的倍數，剩下就好辦了~

↓ 直接照搬code（懶） ↓
{% codeblock lang:python %}
>>> from Crypto.Util.number import *
>>> from math import gcd
>>> n = {blah}
>>> t_p = {blah}
>>> enc = {blah}
>>> p = gcd(n,t_p-1)
>>> q = n // p
>>> assert n == p * q
>>> phi = (p-1)*(q-1)
>>> e = 65537
>>> d = inverse(e,phi)
>>> long_to_bytes(pow(enc,d,n))
b'ASIS{baby___RSA___f0r_W4rM_uP}'
{% endcodeblock %}



## [PPC] Hadamard (Unsolved)

nc [ip] [port] 顯示如下。

```
------------------------------------------------------------------------
|            -+-+ Joy of a tour of Hadamard Matrices +-+-              |
|     Complete the given matrix M, such that M be Hadamard Matrice.    |
|     M is Hadamard if M*MT = nI, where M is n*n matrice and MT is     |
|     transpose of M, and all entries of M are only +1 or -1.          |
------------------------------------------------------------------------
| M =
 ['?', 1]
[1, -1]
| please fill the M and send the md5(M) **NOT M**

```

（待補）

## [PPC] Titanic (Unsolved)

```
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
+ welcome to JPS challenge, its about just printable strings! the number  +
+ n = 114800724110444 gets converted to the printable `hi all', in each   +
+ round find the suitable integer with given property caring about the    +
+ timeout of the submit of solution! all printable = string.printable  🙂 +
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
| whats the nearest number to 47001791081842403297008876143460943635250678429979315483892237252 that gets converted to the printable string?
```

（待補）

## [Misc][Forensics] Adventure

Time plays a role in almost every decision. And some decisions define your attitude about time.
Can you [README.txt]? It's time for a new adventure!

Note: Slow-download is international and part of the task.

題目給定下載連結，印象中好像有到 4G bytes，然後你會發現下載超慢還會斷XDD
然後我就傻傻的一直給他按下載丟著讓他跑，想當然爾沒有結果~

看完 [writeup](https://ctftime.org/writeup/22113) 才發覺這題其實還蠻好玩der !!

（待補）