---
title: 執行檔的結構
layout: post
category: null
---

我們都知道 c c++ 這類的語言要經過編譯產出一個執行檔才能執行  
這個執行檔的內容中，一般都會包含三個段: text rodata data  

- text: 儲存編譯完 已經是機器碼的程式  
- rodata: 唯讀的資料 比如說用 const 宣告的全域變數  
- data: 可寫的資料 比如全域變數 還有 static 的變數  

需要注意的一點是區域變數還有用 malloc 之類的分配的變數都不存在於 data 中
它們都是在執行時才分配的  
以上三個段的資料就是編譯器在編譯完後能決定大小的部份  
另外通常還會有一個 bss 段 這個段是存全域但是沒初始化的變數  
這個段只有在執行檔的標頭中標記大小而已 沒有實際存在於執行檔中  
因為沒有初始值 自然不需要佔用硬碟的儲存空間  

上面說到執行檔分成三個段 自然的執行檔中還要儲存這三個段的資訊  
比如位置 大小 屬性(可寫？唯讀？可執行？)  
這些資訊在 windows 的執行檔中是以 PE 格式儲存的  
Linux 的執行檔則是 ELF 格式 事實上兩個格式的概念是一樣的  
PE 格式你可以用 ExeInfo 這個軟體去讀 ELF 則有 readelf  
這邊有個 readelf 的範例:  

<script src="https://pastebin.com/embed_js/scgQXFTh"></script>

指令是 readelf -S <執行檔>  
我刪掉了不重要(懶的說明)的部份 你在這邊可以看到剛剛說的 3 個段  
大小是它們實際佔用的大小 也是它們在硬碟中佔的大小 單位是 bytes  
位址是它們被載入記憶體後應該要擺放的位置 可以看到 .text 要被放到 0x400430  
順帶一提 通常 windows 下這個位址會是 0x400000  
後面的編移量則是在檔案中的位置  
如果你找一個 16 進位的編輯器移動到 rodata 段的位置(0x5c0)  
你應該會在這附近看到你使用的字串常數  

![part-of-exe-show-in-hex]({{ site.url }}/assets/posts/structure-of-execution-file/1.jpg)

("Hello world\n" 這樣的叫字串常數)  
旗標則是這個區段的屬性 A 代表要分配記憶體 X 是可執行 W 是可寫  

接下來來看在記憶體中這些段長什麼樣子 https://pastebin.com/5kNZpBNJ  
在 linux 下這份資訊可以從 /proc/[pid]/maps 找到  
其中可以注意到 記憶體位置對齊了 0x1000 換算下來是 4KB  
這個其實是分頁的大小 也就是作業系統分配記憶體的最小單位  
perms 代表的是權限 分別有 r 代表可讀 w 可寫 x 可執行  
分別再去對照剛剛用 readelf 讀出來的資料會發現  
作業系統把 .text 跟 .rodata 合併在同一頁了  
.data 段被 map 到 0x601000  
然後你還可以在這裡面看到 heap 跟 stack 被分配在哪邊  
自己拿計算機按一下 算一下它們的大小多大吧  

接下來我們用另一種方式看一下程式在記憶體中長成什麼樣子  
先附上 maps 的內容 https://pastebin.com/3Hzq6KwJ  
這是用 gdb 執行這個範例程式的結果 [附圖 2]  
首先下 b _start 再下 r 把程式跑起來  
你會看到目前中斷在 _start 的位置 這是程式的進入點  
到這可能會有個問題 程式的進入點不是 main 嗎  
是啊 你的程式的進入點確實是 main 可是編譯器會幫你加上一段初始化的 code  
這樣才能準備 argc argv 還有一些其它的準備工作  
另外你的 main 也才有地方可以 return  
(是的 在 _start 中是沒有 return 的 它會直接呼叫 system call 結束自己)  
(如果在 _start 中使用了 ret 指令 也就是 return 的組語版 可是會 crash 的)  
接著我們執行到 main [附圖 3]  
下 b main 再下 c  
這畫面中比較重要的一個資訊是 rsp 也就是 stack 的頂端位置  
用 rsp 的值減掉剛剛 maps 的內容中 stack 的結尾位置  
大約是 11KB 也就是你要溢位這麼多的資料才會碰到下一個分頁  
這樣才會觸發作業系統的保護機制把你的程式 kill 掉  
另外程式在執行時會透過減少 rsp 的值來分配區域變數  
比如把 rsp 減 4 那我們就分配好了一個 int 大小的記憶體了  
然後我們用 gdb 印出在 0x4005c0(.rodata) 附近的資料 [附圖 4]  
你會看到我們的字串 "Hello world" 這邊可以跟我們一開始的 readelf 對照
