---
layout: post
title: "C 的測試框架 cmocka - mock"
date: 2015-4-8 11:30
categories: program c cmocka testing
---

上次發文似乎是兩個月前了… 好像有點久

原本覺得 TDD … 該怎麼說呢 先寫測試總覺得怪怪的  
但實際寫了之後 發現 其實這樣可以讓程式比較簡潔吧…  
因為 TDD 只要能過測試就好 只要測試有考慮所有情況 那程式就是沒問題的  
當然不是說程式就能亂寫w

今天要講的是 cmocka 的 mock  
之前一直試不成功 今天終於弄對了  
假設要 mock 的是 fputs 好了  

```c
// 必要的 include 不懂為何 cmocka 不自己 include...
#include <stdio.h>
#include <string.h>
#include <setjmp.h>
#include <stdarg.h>
#include <cmocka.h>

#define fputs mock_fputs // 把系統的 fputs 定義成我們的

int mock_fputs(const char *, FILE *); // 宣告 mock function 的原形

#include "../file/to/test.c" // include '.c' 才能測 static 的變數之類的

int mock_fputs(const char *str, FILE *fp) {
  check_expected(str); // 參數檢查
  return strlen(str); // fputs 的回傳值好像是輸出的長度的樣子
}

#undef fputs // undef 下比較保險

void test_func(void **state) {
  expect_string(mock_fputs, str, "foo"); // 要檢查的參數
  func(); // 實際測試的 function
}

int main(int argc, char *argv[]) {
  UnitTest unitTest[] = {
    unit_test(test_func)
  };
  return run_tests(unitTest);
}
```

說明註解應該夠吧
