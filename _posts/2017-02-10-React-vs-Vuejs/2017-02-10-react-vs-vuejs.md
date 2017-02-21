---
layout: post
title: "React 與 Vue.js 的比較"
category: javascript
tags:
- javascript
- react
- vue
---
現在正流行的兩個前端框架，[React](react) 和 [Vue.js](vue)  
事實上它們的比較網路上也可以找到不少  
這篇文章會從各個角度比較這兩個框架  
但請注意，我不會告訴你哪個比較好，或幫你決定你該使用哪個  

[react]: https://facebook.github.io/react/
[vue]: https://vuejs.org/

<!-- more -->

## 設定環境 ##
如果兩個框架都讓使用者完全從 0 開始的話  
React 官方推的是 es6 + jsx 的寫法，勢必需要 babel  
Vue 官方主要的語法還是 es5 且 template 可以直接寫在 html 中  
另外也可以直接使用 cdn 的版本， Vue 這樣會比較好上手  
不過 Vue 的文件中接下來提到的是 [single file component](single-file-component)  
若要使用這個功能將會需要額外的環境設定，使得 Vue 變的和 React 一樣了  

[single-file-component]: https://vuejs.org/v2/guide/single-file-components.html

## 樣版 ##
React 官方推的是 [create-react-app](create-react-app)  
Vue 則是 [vue-cli](vue-cli)  
create-react-app 重點在於不用寫任何設定就可以快速開始開發  
這點非常的方便  
vue-cli 個人覺得有個很大的問題是，它必須在一開始選擇要使用哪個[樣版](vuejs-templates)  
造成必須在一開始選擇要使用 [webpack](webpack) 還是 [browserify](browserify)  
但這個問題我覺得其實對初學者而言只是麻煩而已  

[create-react-app]: https://github.com/facebookincubator/create-react-app
[vue-cli]: https://github.com/vuejs/vue-cli
[vuejs-templates]: https://github.com/vuejs-templates
[webpack]: https://webpack.js.org/
[browserify]: http://browserify.org/

vue-cli 應該預設提供 webpack 就好了  
雖然 webpack 在設定上比 browserify 要來的難  
可是現在的生態系是極度偏向 webpack 的  
要找到一個複製貼上就可以動的設定也不難  
加上目前 [webpack-blocks](webpack-blocks) 的出現  
寫 webpack 設定的門檻可以降低不少  
讓初學者一開始習慣使用 browserify 我覺得沒有好處  

[webpack-blocks]: https://github.com/andywer/webpack-blocks

## 生態系 ##
React 跟 Vue 其實有個很大的差別  
React 定義自己為一個 view 的 library  
Vue 則是定義自己為一個完整的 framework  
所以 Vue 官方提供了 [vuex](vuex) [vue-router](vue-router)  
還有原先的 vue-resource (目前已改由 pagekit 維護)  
React 則主要是和由社群提供的 [react-router](react-router) [redux](redux) [axios](axios) 等一起使用  
這部份沒什麼好壞的問題  

[vuex]: https://github.com/vuejs/vuex
[vue-router]: https://github.com/vuejs/vue-router
[react-router]: https://github.com/ReactTraining/react-router
[redux]: https://github.com/reactjs/redux
[axios]: https://github.com/mzabriskie/axios

## 官方文件 ##
React 的話很推他 [Thinking In React](thinking-in-react) 這頁  
這頁在講如何把網頁切成一個個的小元件  
這部份是 Vue 的文件該所缺少的  
畢竟兩個都是 component base 的 lib  
卻沒教人怎麼怎麼切 component 有點怪  

[thinking-in-react]: https://facebook.github.io/react/docs/thinking-in-react.html

Vue 的話推他的 [Examples](vue-examples) 裡面搜集了很實際的範例  
尤其是如何與其它 library (ex: jQuery) 使用這個  
React 雖然也可以, 可是官方文件中沒有提到如何使用  

[vue-examples]: https://vuejs.org/v2/examples/

## template vs jsx ##
jsx 與 Vue 的 template 是兩邊用來描述 view 的語法  
其實兩邊都是擴充現有的語言而成的，jsx 擴充的是 js， template 擴充的是 html  
事實上兩邊都是創造了新的語言  
不過個人是覺得 jsx 比 Vue 的 template 要單純一點  
jsx 在迴圈、條件判斷上面，還是直接延用了 js 的語法  
Vue 則在迴圈上使用了 `v-for="item in items"` 這樣的語法  
如果有寫過其它 template 語言的話，或許會覺得熟悉吧  
但如果熟悉 js，jsx 其實不難  

## 其它的語法部份 ##
Vue 的寫法真的會讓人切身的體會到它是框架  
因為在寫的東西永遠都像是在寫設定一樣  
畢竟是回傳一個 Object  
```javascript
export default {
  name: 'VueRecaptcha',
  props: ['sitekey'],
  mounted () {
    // ...
  }
}
```

React 的寫法比較介於框架和函式庫中間  
```javascript
class MarkdownEditor extends Component {
  render () {
    return (
      <TextField />
    )
  }

  static propTypes = {
    defaultValue: PropTypes.string
  }
}
```

看哪個看的順眼吧  

## one-way vs two-way data flow ##
當你表單欄位很多時你會恨死 React  
但當出問題是你會愛死它 就這樣  
Vue 雖然有 two-way 但預設卻是 one-way  
(而且 Vue 的 two-way 其實是像語法糖的東西)  
或許這說明了 one-way 還是能有效減少問題發生吧  
順帶一提 js 可以這樣寫  
```javascript
this.setState({
  [event.target.name]: event.target.value
})
```
用 React 時請別傻傻的一個欄位一個欄位的寫 callback  

## Context ##
React 跟 Vue 其實有個不小的差別  
比如 `App = (<Parent><Child /></Parent>)`  
也就是 Parent 和 Child 都寫在 App 這個元件中的元件  
Vue 在這種情況下 Parent 能對 Child 做的事非常有限  
頂多決定 Child 的位置而已  
但 React 可以透過 context 傳遞額外的參數給 Child  
或是 inject 額外的 prop 給 Child 從而改變 Child 的狀態  
從而可以實現如 [react-music](react-music) [react-collection-helpers](react-collection-helper)  
這類以 component 架構做為其 API 的函式庫  
這目前是我在 Vue 還沒見到過的  

[react-music]: https://github.com/FormidableLabs/react-music
[react-collection-helpers]: https://github.com/joshwcomeau/react-collection-helpers

以上就是個人使用過了兩個框架的心得，希望對你有幫助  
最後，我不說哪個要來的好是因為我認為：每個東西都有它的特點，選擇哪個完全是看專案的**需求**是什麼  

