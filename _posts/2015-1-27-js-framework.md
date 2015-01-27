---
layout: post
title: "AngularJS & ReactJS"
date: 2015-1-27 11:22
categories: program website javascript framework
---

在之前我就有接觸過AngularJS了 當初真的覺得它很有趣

居然把MVC的架構也帶到了前端來 雖然google管它叫MVW

而在之前的一篇文章中又看到了ReactJS 感覺就跟AngularJS很不一樣

不明白的是它們為什麼又被拿來做比較

AngualrJS給我的感覺是它是一個很完整的框架

包含了除了產生html外 還有像ajax之類的功能

而ReactJS給我的感覺就是完全只是處理UI的部份

雖然它的作法讓我挺訝異的 它居然是產生一個元件

之後就是使用這個元件 另外還有它的虛擬DOM

不過這個部份還沒有實際的感覺到它的速度差距就是了

不過今天又看到了 將這兩個有趣的框架結合的作法 [ngReact](https://github.com/davidchang/ngReact)

於是今天就看了它的範例又寫了一個出來

index.slim:

```slim
doctype html
head
  script src="bower_components/angular/angular.js"
  script src="bower_components/react/react.js"
  script src="bower_components/ngReact/ngReact.js"
  script src="index.js"
body ng-app="app" ng-controller="main"
  hello name="person.name"                   # 自定的標籤
```

index.ls:

```livescript
this <<< React.DOM                      # 把ReactJS的DOM載入目前的scpoe

app = angular.module \app, [\react]

app.controller \main, ($scope) ->
  $scope.person =                       # 測試用的資料
    name: \DanSnow

HelloComponment = React.create-class do  # ReactJS的部份
  prop-types:
    name: React.PropTypes.string.isRequired
  render: ->
    span null, 'Hello ' + @props.name

# 把HelloComponment變成一個html的標籤 這個是AngularJS的 今天是第一次用這功能
app.directive \hello, (react-directive) ->
  react-directive(HelloComponment)
```

再用slimrb跟lsc編譯過就能用了
