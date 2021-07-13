---
title: 一分钟就能上手的requestAnimationFrame替代setInterval教程
date: 2020-08-11 15:07:34
categories:
- 前端
tags:
- nodejs
- requestAnimationFrame
- setInterval
---
[requestAnimationFrame](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame)是一个新的浏览器特性，其优势在于允许浏览器主动控制并优化动画的绘制，通过这个API可以实现更可靠的定时器。

<!-- more -->

假设我们想每秒钟让一张图片出现在画布的随机位置上，传统的写法是这样的

```javascript
setInterval(() => {
  let image = document.getElementById('image');
  document
    .getElementById('myCanvas')
    .getContext('2d')
    .drawImage(
      image,
      Math.floor(Math.random()*100),
      Math.floor(Math.random()*100)
    )
}, 1000);
```

使用requestAnimationFrame的写法如下所示

```javascript
let lastRefresh = 0;
let randomImage = (time) => {
  //requestAnimationFrame调用回调函数的时候，会传入一个时间戳
  //通过这个时间戳进行比对来实现自定义延迟
  if(time - lastRefresh > 1000) {
    lastRefresh = time;
    let image = document.getElementById('image');
    document
      .getElementById('myCanvas')
      .getContext('2d')
      .drawImage(
        image,
        Math.floor(Math.random()*100),
        Math.floor(Math.random()*100)
      )
  }
  //将自身作为参数传入实现重复调用
  requestAnimationFrame(randomImage);
}
//初次调用，获得time参数
//切记不能直接像randomImage()这样调用
requestAnimationFrame(randomImage);
```

然而不同的浏览器对这个API的实现有一些差异。有大佬写了一段多浏览器兼容代码，直接复制粘贴，在调用requestAnimationFrame之前执行即可

```javascript
/**
  * Provides requestAnimationFrame in a cross browser way.
  * http://paulirish.com/2011/requestanimationframe-for-smart-animating/
  */
if (!window.requestAnimationFrame) {
  window.requestAnimationFrame = (function () {
    return (
      window.webkitRequestAnimationFrame ||
      window.mozRequestAnimationFrame ||
      window.oRequestAnimationFrame ||
      window.msRequestAnimationFrame ||
      function (callback) {
        window.setTimeout(callback, 1000 / 60);
      }
    );
  })();
}
```
