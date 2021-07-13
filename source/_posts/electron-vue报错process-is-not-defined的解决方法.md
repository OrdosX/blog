---
title: electron-vue报错process is not defined的解决方法
date: 2020-10-07 20:27:13
categories:
- 前端
tags:
- electron
- electron-vue
---
在按照官方文档创建一个项目之后，运行时报错`ReferenceError: process is not defined`，综合[相关issue](https://github.com/SimulatedGREG/electron-vue/issues/871#issuecomment-529809406)和个人经验，总结出了最简单的解决方法如下。

<!-- more -->

首先打开`.electron-vue/webpack.renderer.config.js`，在大约第125行的`new HtmlWebpackPlugin`中任意位置添加一个成员变量`isBrowser: process.browser`如下所示

```js
new HtmlWebpackPlugin({
    ...
    isBrowser: process.browser,
    ...
}),
```

然后打开`src/index.ejs`，在大约第16行的位置找到

```
<% if (!process.browser) { %>
```

将其改成

```
<% if (!htmlWebpackPlugin.options.isBrowser) { %>
```

然后再次启动程序，可见问题解决。
