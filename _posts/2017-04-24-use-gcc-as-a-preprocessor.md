---
layout: post
title: "用 gcc 做預處理"
description: "如何讓 gcc 在非 C 語言中處理 C 的 macro"
category: tool
tags: [tool gcc preprocessor nasm]
---

C 語言裡的 macro 功能真的不錯用  
很多常數都是用 `#define` 做定義的  
但如果今天使用的不是 C 語言卻需要這些常數時  
除了一個一個複製貼上外，有沒有更好的辦法呢  
<!-- more -->

這是我某天在寫組語時的事，我需要使用到 syscall  

~~~nasm
mov rdi, 1
mov rsi, buf
mov rdx, buf_len
mov rax, SYS_write
syscall
~~~

這麼多的 syscall 代號，一個一個的複製過來真的太麻煩了  
於是我就想到用 gcc 幫忙處理這些已經定義在 `sys/syscall.h` 中的代號  
首先先在檔案開頭加上

~~~nasm
%if 0 ; 這是 nasm 的條件組譯，其實相當於區塊註解，也就是 C 的 /* */
#include <sys/syscall.h>
%endif
~~~

因為必須要在 nasm 中引入 C 的標頭檔來取得常數的定義  
於是需要使用區塊註解的方式，防止 C 的標頭檔被當成組語解析  
接著就讓 gcc 上場了

~~~shell
$ gcc -E -x c -P foo.s -o foo.i.s
$
~~~

這時可以打開看看 `foo.i.s` 中的常數像 `SYS_write` 是不是都被取代了  
以下解釋一下參數

- -E: 只做 preprocess
- -x c: 將輸入檔案視為 C 語言，否則不會被正常處理
- -P: 不要輸出行號等除錯資訊

接下來就可以正常的組譯連結了  
這個方法其實不只可以使用在組語上  
只要該語言有支援區塊註解就行了  
不支援也沒關係，手動刪除多餘的內容即可  

雖然這麼做檔案開頭會多出很多原本 C 的標頭檔中的東西  
不過可以不用慢慢的查常數的值再複製貼上，其實是很方便的
