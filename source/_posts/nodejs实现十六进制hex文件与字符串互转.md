---
title: nodejs实现十六进制hex文件与字符串互转
date: 2020-07-16 09:00:11
categories:
- 后端
tags:
- nodejs
- hex
---
十六进制字符串hex string在处理网络数据包的时候经常作为导出格式，但是如果要通过vscode的hex editor插件编辑，就要转成二进制的.hex文件，而在使用的时候还要转回来。经过搜索发现通过fs包可以很好地完成这个任务。

<!--more-->

## 字符串转二进制

同目录存在一个名为in.txt的文件，运行该脚本，即可得到out.hex

```js
const fs = require('fs')

fs.writeFileSync(
    `out.hex`,
    fs.readFileSync('in.txt').toString(),
    { encoding: "hex" }
)
```

## 二进制转字符串

同目录存在一个名为in.hex的文件，运行该脚本，即可得到out.txt

```js
const fs = require('fs')

fs.writeFileSync(
    `out.txt`,
    fs.readFileSync('in.hex').toString('hex')
)
```
