---
title: Ubuntu添加新用户后无法登陆的一种解决方法
date: 2018-09-10 18:10:23
categories:
- Linux
tags:
- 用户
---
## 问题

在Ubuntu中用以下命令添加系统用户：

      sudo adduser --system newuser

然后使用 `su` 命令，输入密码后回车，但还显示在原来的账户下。通过SSH登陆时，一登陆就闪退。

<!--more-->

## 解决

网上搜索一下，绝大部分资料要求将 `/etc/ssh/sshd_config` 中的 `usePAM yes` 一行的yes改为no。然而这并没有解决问题，更糟的是，关掉这个选项使得登陆时输出的"message of the day"消失了。看`sshd_config`里对该选项的描述，关掉这个选项似乎会导致安全问题，因此这不是一个正确的方法。

之后，我找到了一些添加新用户的指令示例，有的示例加了一个参数 `--shell /bin/bash`

这个指令指定了新用户的shell，将此参数添加进之前的命令后，即可正常添加系统用户。

不过，如果添加的是普通用户（没有--system参数）的话，就不需要加--shell参数，这也正是为什么我前面写的是“有的示例”的原因。