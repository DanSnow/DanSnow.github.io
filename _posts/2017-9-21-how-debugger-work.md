---
title: Debugger 如何運作的
layout: post
category: null
---

Debug 就我知道一般有三種常見的機制
1. 使用 int 3 (x86 or x64) 觸發軟體中斷
2. 攔截 syscall (這上面有人提到)
3. 使用 TF flag

另外以下有幾點注意：
1. 以下環境以 Linux x86 or x64 為主 不過 Windows 的機制也是類似的
2. int 3 我習慣稱 CC 原因為 X86 下的機器碼的 16 進制正好為 CC

### 使用 int 3：
這個其實才是我們常說的中斷點，debugger 正是靠在程式中插入 CC 來達成中斷效果。
"不是都轉成機器碼了嗎"，是啊，這有什麼問題呢？一般會有的疑問應該是以下兩點：
1. 要怎麼修改執行中的程式
2. 要怎麼知道在哪個位置插入 CC 指令
上面有人有提到 ptrace 可以去看一下 [man page 中對 ptrace 的說明](http://man7.org/linux/man-pages/man2/ptrace.2.html)

這邊大概列一下 ptrace 有的功能：
1. 取得/修改暫存器
2. 取得/修改程式資料
3. 中斷在下一個 syscall (之後說明)
4. 單步執行 (之後說明)

其中我們這邊關心的是第 2 個："取得/修改程式資料"，這個功能讓你可以在程式執行的過程中修改程式在記憶體中的資料。接著這邊有個很重要的觀念

> **編譯好的程式(機器碼)本身也是一種資料，用來描述 CPU 該如何執行的資料。**

機器碼本身會被載入到記憶體中，被 CPU 讀取，並根據它的值 CPU 會做出對應的動作，比如執行加法、跳過下一個指令之類等等。
那麼問題就簡單了，我們只要知道要中斷的指令在哪個位置接著：

1. 把原本的指令備份後換成 CC
2. 當程式執行到 CC 時系統就會送出 Signal Trap 通知 debugger
3. 把修改的指令復原

問題就解決啦，才怪，我們碰到了下一個問題 要怎麼知道該中斷在哪邊

這邊又分成兩種情況：
1. 你在 debugger 看到的是反組譯的 code 也就是你看到的是組語
2. 你看到的是原始碼

兩種都可以讓你下中斷點 1 比較簡單，一個組語就對應到一個指令，把那個指令換成 CC 就行了。
2 的情況我們就又要扯到另一個東西了 `debug symbol` 它紀錄的原始碼與編譯後的結果之間的對應關係。目前主流格式有 `stabs`, `COFF`, `XCOFF`, `DWARF` 這些只是名字，這邊不會特別做說明。
如果你使用 gcc 編譯時加上了 [-g 選項](https://gcc.gnu.org/onlinedocs/gcc/Debugging-Options.html) gcc 就會幫你加上這個東西。
比較有趣的是 gcc 附上的 debug symbol 其實有加上自己用來給 gdb 看的擴充，那麼到這邊，兩個問題都解決了。

### 攔截 syscall：
關於這個，我正好有找到網路上有人解釋它是如何運作的 https://gist.github.com/RustJason/4e05f6b65448be376fc0
雖說我在這之前也有自己翻一下 code 就是了，上面那個連結裡面解釋的很簡潔，簡單來說：
當你使用 ptrace 要求攔截 syscall 時，它會設定一個 flag 指名要 trace syscall
而當系統發現有這個 flag 時，就會中斷並通知 debugger，這就是上面提到的 ptrace 的第 3 個功能

### TF flag：
TF flag 是我們 x86 系列的 CPU 有的一個 flag 它的功能是設定程式為單步執行模式
每執行一個指令 CPU 便會中斷一次 CPU 的中斷會通知 Kernel Kernel 又通知 debugger
這正是上面 ptrace 的第 4 個功能
順帶一提 這個 flag 甚至還有它專屬的 wiki 頁面
https://en.wikipedia.org/wiki/Trap_flag

接下來來個好玩的實作吧 這邊示範如何用 ptrace 修改執行中的程式
原始碼：https://gist.github.com/DanSnow/c29ac0dcf653b4e8cbacfac4dc4e4a81
程式中一開始先讀取原本的機器碼 修改後再寫回去相同的位置
至於那個記憶體的位置與機器碼該如何修改 我是用 objdump 得到的 
![附圖 1](https://i.imgur.com/aql0JWU.jpg)
底下附上的是原本的執行結果與被修改後的執行結果 
![附圖 2](https://i.imgur.com/x1b5z6G.jpg)


此外上面提到的軟體中斷是指 `interrupt` 那是一種來自硬體，同時也可由軟體觸發的一種通知機制，它會打斷 CPU 目前的工作，並跳到 OS 設定好的位置，讓 OS 處理，其中包含比如鍵盤按鍵就會觸發中斷
