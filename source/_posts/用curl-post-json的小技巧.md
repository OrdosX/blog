---
title: 用curl post json的小技巧
date: 2020-07-11 17:58:42
categories:
- Linux
tags:
- bash
- curl
---
最近有个需求是写一个能直接post json数据到某个位置的bash脚本，在网上查到可以用curl实现。由于json包含双引号，只能将整个字符串用单引号包裹来取消转义，但是这么做又不方便注入数据，所以我想了一个方法来实现。

<!--more-->

如果要发送固定内容的json是这么做的

```bash
# 发送少量数据
curl -X POST -H "Content-Type: application/json" -d '{"user":"admin","password":"12345"}' http://192.168.1.1/login
# 大量数据建议从文件读取
curl -X POST -H "Content-Type: application/json" -d @./login.json http://192.168.1.1/login
```

如果要通过 `./login.sh admin 12345` 传入不同的用户名和密码，上面这种方式就行不通了

```bash
curl -X POST -H "Content-Type: application/json" -d '{"user":"$1","password":"$2"}' http://192.168.1.1/login
# 发送的内容是{"user":"$1","password":"$2"}
```

我的方法是定义一个变量代表双引号，字符串改用双引号以启用字符串注入。建议在本地写好json再全部替换双引号

```bash
# q是quote的缩写，当然随便什么名字都可以
q='"'
curl -X POST -H "Content-Type: application/json" -d "{${q}user${q}:${q}$1${q},${q}password${q}:${q}$2${q}}" http://192.168.1.1/login
# 发送的内容是{"user":"admin","password":"12345"}
```
