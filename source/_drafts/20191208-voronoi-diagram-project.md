---
title: 【Report】Voronoi Diagram Project
date: 2019-12-8 17:34:34
tags:
	- Reports


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

所以在作圖上我選擇Python，想說CTF也會用到就加減練語法

使用的環境是Python 3.6，考慮到會用到一些奇怪套件，所以先建立虛擬環境venv等前置作業。

作圖方面原本採用的是Python開源函式庫-`kivy`，結果因為助教的作業環境是Windows，所以打包成`.exe`的過程會發生套件衝突，因此在初測前幾天忍痛搬家到Python內建函式庫-`tkinter`，這過程大約花了我一天的時間在處理QQ

先來看看還沒初測就胎死腹中的`kivy`介面👇👇

![]()

再來是使用`tkinter`完測後 Run出來的介面👇👇

![](https://github.com/yctseng1227/voronoi_project/blob/master/voronoi_tkinter/pic/Screenshot%20from%202019-12-08%2017-42-01.png?raw=true)

簡單介紹一下功能：

- import：讀檔
- load：讀檔
- random：隨機生成N個點，預設6點可重複疊加
- save：存檔
- next case：當讀入的檔有多筆case，
- step by step：Divide-and-Conquer
- run VD：畫出Voronoi Diagram
- clear：清除畫布
- convex hull：debug用？