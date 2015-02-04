---
layout: post
title: "Browserify with Gulp"
date: 2015-2-4 11:28
categories: program website javascript gulp browserify
---

最近在使用ReactJS 當然免不了的要把一堆的程式打包起來了

不然把所有的東西寫在一個js裡 應該是會挺亂的

這次使用的是browserify不過呀 在用的過程中還真的出現了不少的問題呀

像是在command line下使用liveify的話就好好的 但在gulp中就出現在syntax error

這部份還沒解決 目前先暫時把.ls先全部編成.js在來做browserify

接下來是今天的重點了 跟據gulp上的recipes 我們應該要直接使用browserify

不過呀 應該算我眼殘吧 居然沒看到還要用 `vinyl-source-stream`

之後… gulp當然是出錯了

這個錯誤訊息 個人覺得沒什麼用

`throw new TypeError('Arguments to path.resolve must be strings');`

總之 只要加上了 `vinyl-source-stream` 就沒事了
