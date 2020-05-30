---
title: 【Project】Voronoi Diagram
date: 2019-12-8 17:34:34
categories: 
- 專案實作
tags:
- Algorithm


---

[![](https://pbs.twimg.com/media/D6tZ-v4UIAAMm7t?format=jpg&name=large)](https://twitter.com/potch/status/1129095107855040513?s=20)

Hmmmmmmm... 上圖是理想中的Voronoi Diagram

不過很遺憾的是Algorithm這門課我沒成功做出來QQ

即便如此還是要寫報告：）

<!--more-->

&nbsp;

時程規劃

- 11/14 初測(基本功能+3點VD)：19.5 / 20
- 12/05 完測(完整功能+4點以上VD)：34 / 60
- 12/13 報告撰寫：?? / 20

[實施要點](http://par.cse.nsysu.edu.tw/~cbyang/course/algo/algo_report.htm)&nbsp;&nbsp;[初測評分標準](http://par.cse.nsysu.edu.tw/~cbyang/course/algo/Voronoi_evaluate_first.pdf)&nbsp;&nbsp;[完測評分標準](http://par.cse.nsysu.edu.tw/~cbyang/course/algo/Voronoi_evaluate_second.pdf)

Code: https://github.com/yctseng1227/voronoi_project

從`commits`數量應該不難看出花了不少時間在上面，但效果顯然很糟

我爛 QQ

---

由於這份作業不光是寫出VD演算法，還必須將圖片畫出來！
所以在作圖上我選擇Python，想說CTF也會用到就加減練語法M(\_ \_)M

使用的環境是Python 3.6，考慮到會用到一些奇怪套件，所以先建立虛擬環境venv等前置作業。

作圖方面原本採用的是Python開源函式庫-`kivy`，結果因為助教的作業環境是Windows，所以打包成`.exe`的過程會發生套件衝突，因此在初測前幾天忍痛搬家到Python內建函式庫-`tkinter`，這過程大約花了我一天的時間在處理QQ

先來看看還沒初測就胎死腹中的`kivy`介面👇👇

![](https://github.com/yctseng1227/voronoi_project/blob/master/pic/pic05.png?raw=true)

個人覺得`kivy`在圖形界面上比較理想，而且`.kv`在建構上也比較容易整理，臨時搬到`tkinter`反而code被我寫爛了，雖然是可以勉強執行，但造成後續寫多點VD維護上的困難QQ

另外在打包`.exe`方面，由於我的作業環境是UbuntuOS，在該系統的`Pyinstaller`操作下生成的執行檔並無法在WindowsOS環境下執行，最後折衷和同學借電腦進行打包，也因此沒機會針對`.exe`作多次檢查，結果就是初測時發生`.py`和`.exe`執行同一筆測資出現不同的結果，真的是見鬼了wwww 至今還是沒辦法解釋該情況怎麼會出現...

而在多點VD的操作上，單純以Divide-and-Conquer演算法來說並不困難，但是在圖形的實現方面需要考量線條的距離繪製，比起消除多餘線條，個人傾向修正線條的起訖。在進行Hyperplane的時候，撞到的VD線不曉得到底該往哪邊進行縮減，導致最後畫出來的結果常常會是相反的方向，這細節大部分的VD的不太會提及，因此debug方面花了相當多的時間去"猜"怎麼畫線，最後還是不如預期地拿了低分（唉

另外就是step by step，平常少有機會遇到這種操作，我用的是以`list`形式把每個不同階段的points list和edge list記錄下來，等需要的時候(按button)再逐一從該`list`取出來繪製，不過在完測時看起來是儲存points list和edge list的時機不對，造成斷點相當明顯稍稍覺得可惜。

總體來說，花了時間卻拿不到相對的分數，嗚嗚浪費時間QQ

簡單介紹一下功能：

![](https://github.com/yctseng1227/voronoi_project/blob/master/pic/pic02.png?raw=true)

- import: 讀檔格式如下

```
P 103 200
P 193 64
P 193 370
P 283 200
E 0 34 193 161
E 0 363 193 261
E 193 161 193 261
E 193 161 437 0
E 193 261 600 476
```

- load: 讀檔格式如下

```
# comment
6
487 191
417 388
504 349
681 125
589 282
725 279
6
69 304
110 449
174 333
244 280
335 139
355 284
12
69 304
174 333
110 449
335 139
244 280
355 284
487 191
417 388
504 349
681 125
589 282
725 279
0
# any else?
```

- random: 隨機生成N個點，預設為6點可重複疊加
- save: 將point, edge存成`import`格式
- next case: 當讀入的檔有多筆case，依此按鈕進行下一筆testcase
- step by step: 依Divide-and-Conquer逐步執行
- run VD: 畫出Voronoi Diagram結果
- clear：清除畫布
- convex hull: 將所有的Convex Hull畫出，算是拿來dubug用的XD

環境

- Operating System
```javascript
                          ./+o+-       
                  yyyyy- -yyyyyy+      Username: yctseng
               ://+//////-yyyyyyo      OS: Ubuntu 18.04 bionic
           .++ .:/++++++/-.+sss/`      Kernel: x86_64 Linux 5.0.0-36-generic
         .:++o:  /++++++++/:--:/-      Uptime: 9d 18h 15m
        o:+o+:++.`..```.-/oo+++++/     Packages: 2347
       .:+o:+o/.          `+sssoo+/    Shell: bash 4.4.20
  .++/+:+oo+o:`             /sssooo.   Resolution: 3840x1080
 /+++//+:`oo+o               /::--:.   DE: GNOME 
 \+/+o+++`o++o               ++////.   WM: GNOME Shell
  .++.o+++oo+:`             /dddhhh.   WM Theme: Adwaita
       .+.o+oo:.          `oddhhhh+    GTK Theme: Ambiance [GTK2/3]
        \+.++o+o``-````.:ohdhhhhh+     Icon Theme: ubuntu-mono-dark
         `:o+++ `ohhhhhhhhyo++os:      Font: Ubuntu 11
           .o:`.syhhhhhhh/.oo++o`      CPU: Intel Core i7-8700 @ 12x 4.6GHz
               /osyyyyyyo++ooo+++/     GPU: intel
                   ````` +oo+++o\:     RAM: 15831MiB
                          `oo++.
```
- Python3.6 (Virtual environment)
- Package version
    - imutils==0.5.3
    - Kivy==1.11.1
    - numpy==1.13.3
    - scipy==0.19.1
    - pyinstaller==3.5 (WindowsOS)

### **Data Structure**

- **Point**: 
用Python的`tuple`資料型態以$(x, y)$儲存讀入的points，再放入`list`當作該testcase的point list。
- **Edge**
從Python環境而言，必須用兩點才能畫成一線，所以在資料結構上以$[(x1, y1), (x2, y2)]$方式儲存。另外一種是進行中垂線繪製時，由於除了該中垂線以原本$[(x3, y3), (x4, y4)]$ 儲存外，還必須記錄該線是以哪一條線作為中線基準，因此儲存方式為$[(x3, y3), (x4, y4), (form\_x1, form\_y1), (form\_x2, form\_y2)]$，方便後續進行VD。
- **Convex Hull (CH)**
前置作業為針對$x$座標將point list進行排序，因此CH function的回傳值為CH的描點順序。
- **Voronoi Diagram (VD)**
VD的資料型態為edge list，其中不乏上述Edge data structure的兩種儲存方式。

### **Algorithm**

- **Convex Hull (CH)**
利用排序先找到最外圍的任一point作為起點，窮舉平面上所有的點，利用向量積(cross product)判斷是否在圈內or圈外，順著外圍繞一圈。

- **Voronoi Diagram (VD)**
3點以內利用外心以及各邊的中點，考慮各種情況下畫出VD。
多點情況，利用`Divide-and-Conquer`進行實作，在遞迴至3點以下時將會直接回傳結果，此外將point set拆成兩邊進行遞迴後再合併，詳見如下(n為points count)。
$$ f(point\ set)=\left\{
\begin{aligned}
Do\ nothing && n = 1 \\
Return\ Line && n = 2 \\
Return\ VD && n = 3 \\
Return\ Merge(f(point\ set[0, n/2]),f(point\ set[n/2, n])) && n > 3 \\
\end{aligned}
\right.
$$
針對`Merge function`，其步驟如下:
- step 0. Hyperplane預設在畫布由上往下描出。
- step 1. 將兩邊points set各自以y座標進行排序，找出最上面兩點作為參考點。
- step 2. 利用參考點畫出的中垂線作為Hyperplane，並且和兩邊VD的所有edge set找出最上面的交點，並記錄撞到的edge。
- step 3. 根據撞到的edge進行參考點移動，並消除多餘的線。
- step 4. 重複step 2和step 3，直到兩邊point set走到底為止。

<center>(下面兩張圖各6點。<font color="#0000FF">藍線為CH</font>; <font color="#00FF00">綠線為VD</font>; <font color="FF0000">紅線為Hyperplane</font>)</center>
{% gp 2-2 %}

![](https://github.com/yctseng1227/voronoi_project/blob/master/pic/pic07.png?raw=true =340x) ![](https://github.com/yctseng1227/voronoi_project/blob/master/pic/pic06.png?raw=true =340x)

{% endgp %}

### **Result**

![](https://github.com/yctseng1227/voronoi_project/blob/master/pic/pic03.png?raw=true)