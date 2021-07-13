---
title: 快速安装VS Code中Golang插件所需的分析工具
date: 2018-12-22 14:07:05
categories:
- 后端
tags:
- Golang
- VS Code
---
使用VS Code编写Golang代码，Golang插件便提示需要安装相关分析工具如Golint等。然而直接点击安装，往往会因为DNS污染而失败。**更新：**现在可以通过设置goproxy直接消除DNS污染造成的影响，旧的手动方法仅作为存档保留。

<!--more-->

## 新方法：使用goproxy

终端中输入以下内容

```
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.io,direct
```

然后重启窗口，执行安装即可。

以下是原文

---

使用VS Code编写Golang代码，Golang插件便提示需要安装相关分析工具如Golint等。然而直接点击安装，往往会因为golang.org被墙而失败。经过一番搜索，我总结了一个相当方便的分析工具安装方法，供大家参考。

## 准备

需要先安装Golang编译器、VS Code的Golang插件和Git。

## 开始

在VS Code中，首先打开GOPATH（也就是有bin、src、pkg三个文件夹或其中几个的目录），调出终端输入以下内容：

```shell
git clone https://github.com/golang/crypto.git ./src/golang.org/x/crypto/
git clone https://github.com/golang/tools.git ./src/golang.org/x/tools/
git clone https://github.com/golang/lint.git ./src/golang.org/x/lint/
git clone https://github.com/golang/text.git ./src/golang.org/x/text/
git clone https://github.com/golang/net.git ./src/golang.org/x/net/
git clone https://github.com/golang/sys.git ./src/golang.org/x/sys/
```

然后重启VS Code，点击Golang插件弹出的“Analysis Tools Missing”提示框下的“install”按钮，可以发现一次安装完成，You're ready to Go :)

## 知识链接

你可以尝试着直接点击Golang插件弹出的“Analysis Tools Missing”提示框下的“install”按钮。它将开始自动下载。但是，除了像 `github.com/...` 的少数几个分析工具外，都安装失败了。有些工具虽然指向GitHub，但是实际下载点在golang.org，所以也没办法下载。

需要安装的插件来自`golang.org/x/tools/`和`golang.org/x/lint/`两个包，但是光下载这些包还不够，安装时会提示它们引用的其它包还没安装。所以还要使用另外几个命令安装`golang.org/x/crypto/`、`golang.org/x/text/`、`golang.org/x/net/`和`golang.org/x/sys/`。
