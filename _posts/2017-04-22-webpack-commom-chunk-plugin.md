---
layout: post
title: "webpack 的 CommonChunkPlugin 的使用方式"
description: "CommonChunkPlugin 常用設定與說明"
category: javascript
tags: ["javascript", "deploy", "webpack", "webpack-plugin"]
---

webpack 不是只能將檔案打包成單一個檔案  
有時我們也會打包成多個檔案，有可能是因為網頁本來就有多個頁面  
或是為了加快載入速度，而不要讓網頁一開始就載入整包 js  
但是產生的檔案之中可能存在不少重覆打包的 js  
這時就要靠 `CommonChunkPlugin` 將它們獨立打包成一個檔案  
一來可以減少大小，二來可以讓檔案被 cache 而加快載入速度  
<!-- more -->

## 多個 entry 間重覆的 module ##

假設 webpack 中設定了多個 entry  

~~~javascript
{
  entry: {
    entry1: 'entry1.js',
    entry2: 'entry2.js'
  }
}
~~~

其中它們共同 require 了數個 js  
這時就可以使用以下設定將它們獨立打包  

~~~javascript
{
  plugins: [
    new webpack.optimize.CommonChunkPlugin({
      name: 'common', // 幫 chunk 設定名字，可做識別用
      filename: 'common.js' // 輸出檔名，若沒設定會依照 output 中指定的格式輸出
    })
  ]
}
~~~

這時會多產生一個叫 `common.js` 的檔案，這就是數個 entry 中共同的部份  
載入時需要把 `common.js` 放在原本的 entry 前面  

~~~html
<script src="common.js"></script>
<script src="entry1.js"></script>
~~~

## 額外的打包第三方套件 ##

網頁中可能會使用到不少第三方的套件，這些套件其實更新的比較不會那麼頻煩  
如果把它們額外打包成一份 js 的話，可以額外的 cache 而加速網頁載入速度  
設定如下，假設使用到的第三方套件是 React  

~~~javascript
{
  entry: {
    vendor: ['react', 'react-dom'], // 要分開打包的第三方套件們的名字
    app: 'main.js' // 主程式的 entry
  },
  plugins: [
    new webpack.optimize.CommonChunkPlugin({
      name: 'vendor', // 第三方套件的 chunk 的名字
      minChunks: Infinity // 可以不加，這確保 webpack 不會額外打包進其它東西
    })
  ]
}
~~~

這時候需要自行設要額外打包的套件，一樣的需要把 `vendor` 放在 `app` 的前面載入  

## 打包 async chunk 中重覆的部份 ##

`async chunk` 通常是在較大的 SPA 中才會使用的功能  
這是讓 webpack 把 js 檔拆成多個小部份，只在需要時延遲載入而已  
詳細用法請參考[webpack 文件](https://webpack.js.org/guides/code-splitting-async/)  
`async chunk` 特別容易產生大量重覆的程式碼  
使用以下設定就可以將這些 `async chunk` 中重覆的部份再多打包成一個檔案  

~~~javascript
{
  plugins: [
    new webpack.optimize.CommonChunkPlugin({
      async: true, // 抽出 async chunk 中重覆的部份
      children: true, // 指定所有的 chunk
      minChunks: 3 // 重覆出現在 3 個以上的 chunk 中就獨立打包
    })
  ]
}
~~~

設定中的 `minChunks` 可以指定一個自己覺得適合的數字，不過建議一定要指定一個  
因為 webpack 預設的設定是要在所有的 `chunk` 中都有重覆才獨立打包  
這會多產生出一個 `async chunk`，不過不用特別去載入它  
webpack 產生的程式碼會在需要時自動載入的  

## Debug ##

是的 debug  
webpack 可以使用 `--json` 這個選項產生打包的詳細資料  
可以把它上傳到 [webpack analyze](https://webpack.github.io/analyse/) 做分析  
其中的 Hints 標籤中可以看到被重覆打包的檔案  
另外還有 [webpack-bundle-analyzer](webpack-bundle-analyzer) 可以使用  
它可以把每個 chunk 中打包的檔案用圖像化的方式顯示

[webpack-bundle-analyzer]: https://www.npmjs.com/package/webpack-bundle-analyzer

另外還有一點，以上的設定是可以組合使用的  
比如需要打包額外的第三方套件與抽出重覆的 `async chunk`  

~~~javascript
{
  entry: {
    vendor: ['react', 'react-dom'],
    app: 'main.js'
  },
  plugins: [
    // 直接加入兩個 CommonChunkPlugin 就行了
    new webpack.optimize.CommonChunkPlugin({
      name: 'vendor',
      minChunks: Infinity
    }),
    new webpack.optimize.CommonChunkPlugin({
      async: true,
      children: true,
      minChunks: 3
    })
  ]
}
~~~

希望本文章能對你有所幫助  
