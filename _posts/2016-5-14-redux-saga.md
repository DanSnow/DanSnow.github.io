---
layout: post
title: "Redux saga introduction"
date: 2016-05-14 11:56:00 +0800
categories: javascript react redux redux-saga
---

有好段時間沒寫東西了 差不多有一年了呢  
最近突然想起來 想把最近碰到的 [redux-saga](https://github.com/yelouafi/redux-saga) 寫下來  
不過其實或許該來找個時間整理下版面之類的  

## Introduction ##
寫這篇文章時 redux-saga 的版本為 0.10.4 還算是個很新不穩定的東西
redux-saga 可以在網路上找到不少資料 不過都是以英文的為主  
redux-saga 是處理 redux 中遇到的 Side Effects  
如果有碰過 redux 那可能會接觸過 redux-thunk  
沒錯 它們想解決的問題是一樣的 redux 本身沒辦法處理 Async 的問題    
使用 redux-saga 會使用到不少的 generator 對於這點似乎造成有人認為它很難理解  
不過使用 generator 也有個好處 這也是 redux-saga 的優點之一： 容易測試  

## 使用 redux-saga ##
如果使用 redux-thunk 我們可能會寫出像這樣的 action  
```javascript
const fetchTodo = () => (dispatch) => {
  dispatch({ type: 'FETCHING_TODOS' })
  fetch('/todos').then((todos) => {
    dispatch({ type: 'FETCHED_TODO', payload: todos })
  })
}
```
如果在 redux-saga 裡 可能會寫成像這樣  
```javascript
function* fetchTodo() {
  yield put({ type: 'FETCHING_TODOS' }) // Like dispatch({ type: 'FETCHING_TODOS' })
  const todos = yield call(fetch, '/todos')
  yield put({ type: 'FETCHED_TODO', payload: todos }) // Like dispatch({ type: 'FETCHED_TODO' })
}
```
一開始看到了這樣的寫法 這什麼鬼啊 不過接下來就該來解釋下程式碼了
put, call 這些都是 saga 的 effects put 像是 dispatch  
而 call 執行傳給它的函式 並把剩下的參數傳給那個函式  
而這些 effects 並不是直接執行這些動作 而是建立一個描述動作的 object  
並使用 yield 傳給 saga middleware  
再由 saga middleware 來執行 所以在測試時 只要測試這些描述動作的 object 對不對就行  
另外 如果 yield 一個 Promise 的話 會由 saga 去 resolve 並回傳結果的  
所以才會有 `const todos = yield call(fetch, '/todos')` 的寫法  

## 設定 redux-saga 的環境 ##
以上也只是在 saga 下的 action 寫法而已  
像 thunk 有個 thunk middleware saga 當然也有一個 saga 的 middleware  
如果不加上去的話 saga 是不會正常執行的
```javascript
// create the saga middleware
const sagaMiddleware = createSagaMiddleware()
// mount it on the Store
const store = createStore(
  reducer,
  applyMiddleware(sagaMiddleware)
)
```
這樣就可以掛上 saga middleware  
如果有一些要在背景執行的 saga 那還可以
```javascript
sagaMiddleware.run(rootSaga)
```
在設定好 middleware 時就讓它跑起來

## saga 的 Task ##
剛剛好像提到了在背景執行的 saga 對吧  
saga 一個比較特別的地方就是可以在背景執行一些操作 比如監聽 action 之類的  
好像在監聽事件發生一樣 這些叫做 Task  
比如像剛剛的 action 如果你直接把它傳給 dispatch 那是不行的  
而是要在建立另一個 saga
```javascript
function* rootSaga() {
  yield* takeEvery('FETCH_TODO_REQUEST', fetchTodo);
}
```
然後像上面那個執行背景的 saga 的方法 把這個 saga 執行起來  
takeEvery 會監聽符合傳給它的名稱的 action  
並在收到這個 action 時執行由第二個參數傳給它的 saga  
至於 `yield*` 這個又是幹麻的 其實這有點像是 macro 一樣  
把 takeEvery 展開 這樣 saga 的 middleware 就不用再另外處理 takeEvery 了
( 當然這不是真的是 macro [MDN for yield*](mdn-yield-star) )
於是有件重要的事要記得 `yield*` 一定會等到 takeEvery 執行完  
但是 takeEvery 會一直監聽 action 呀 它根本就不會結束 所以如果有這樣的寫法是不行的  
```javaScript
function* rootSaga() {
  yield* takeEvery('ACTION1', action1)
  yield* takeEvery('ACTION2', action2)
}
```
這樣 action2 是不會執行的 必須寫成這樣
```javaScript
function* rootSaga() {
  yield [
    takeEvery('ACTION1', action1),
    takeEvery('ACTION2', action2)
  ]
}
```
另外 還有 `takeLatest` 這個 helper 它跟 takeEvery 不同處在於  
takeLatest 如果在前一個 action 還沒處理完 就收到同樣的 action 時  
takeLatest 會取消掉前一個 而 takeEvery 則都會執行  
其實還有很多有趣的東西 比如像 fork, cancel 呀之類的 不過那應該會再開一篇了  

## Reference ##
[Saga Document](http://yelouafi.github.io/redux-saga/index.html)  
[Redux nowadays](http://riadbenguella.com/from-actions-creators-to-sagas-redux-upgraded/)

[mdn-yield-star]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield*
