---
layout: post
title: "Split wav with shnsplit"
date: 2016-07-29 14:51:00 +0800
categories: linux tools
---

久違的更新 今天正好也是 win10 免費升級的最後一天  
雖然聽說因為時差的關係 實際上是明天  

這篇只是個筆記  

安裝必要工具  
```
$ sudo apt install shntool cuetools
```

分割 wav 並轉成 mp3  
```
$ shntool split -f <cue file> -t '%n-%t' -o "cust ext=mp3 lame -h -b 320 - -o %f" <wav file>
```
這會把 wav 分割 輸出成 mp3 格式 檔名用 <track no>-<track title> 的方式命名  

然後用 cuetag 上 mp3 的 tag  
```
$ cuetag <cue file> *.mp3
```
完成 接下來也可以用 kid3 之類的工具編輯一下 tag 因為有圖形化介面很好操作的  

Reference:  
[https://wiki.archlinux.org/index.php/CUE_Splitting](https://wiki.archlinux.org/index.php/CUE_Splitting)
