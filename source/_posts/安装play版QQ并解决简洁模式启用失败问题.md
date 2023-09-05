---
title: 安装play版QQ并解决简洁模式启用失败问题
date: 2023-09-05 15:48:44
categories:
- 系统技巧
tags:
- QQ
---
本文介绍了如何如何从晨钟酱那里下载Play版QQ 8.2.11的安装文件，然后使用HTTP Proxy Client和LightProxy来修复简洁模式和深色模式。让我们开始吧。

<!-- more -->

## 安装play版QQ

前往B站UP主[晨钟酱Official](https://space.bilibili.com/251013709)的网站[jamcz.com](https://jamcz.com/)，拉到底找到“其他工具”→“养老软件合集”，可以找到大佬分享的play版QQ8.2.11安装文件。

## 启用简洁模式

由于QQ客户端获取对应主题文件的URL在升级到HTTPS后，play版依然在使用旧的URL获取主题文件，导致play版QQ无法切换到简洁模式和深色模式。通过搜索找到了以下几种解决方案：

- [Fiddler+系统WiFi代理](https://blog.csdn.net/m0_59496782/article/details/122783493)：失败，虽然能抓到一部分其他软件的http/https包，但不能抓到QQ下载主题的请求，但原博主成功了，怀疑是系统不同。
- [HttpCanary](https://www.suno.su/archives/4155)：失败，主要是因为怂，抓包需要安装CA证书，但该软件已停止开发并下架，从其他渠道下载的版本不能保证安全性。
- [HTTP Proxy Client+LightProxy](https://www.kokodayo.site/index.php/archives/82/)：成功。

下面细说第三种方法，首先在手机上安装[HTTP Proxy Client](https://apkpure.com/http-proxy-client/com.assets.androidproxy/download/3)，Apkpure中最新的v5版本是xapk格式不能直接安装，但v3版本可以安装且能用。然后在电脑上安装[LightProxy](https://github.com/alibaba/lightproxy)。安装后打开，在右侧“规则”页粘贴以下内容：

```
http://showv6.gtimg.cn https://showv6.gtimg.cn
http://iv6.gtimg.cn https://iv6.gtimg.cn
http://gxh.material.qq.com https://gxh.material.qq.com
#下一行非必须添加，可能对 QQ 空间有用
http://qzonestyle.gtime.cn https://qzonestyle.gtime.cn
```

使用快捷键Ctrl+S保存配置，在右侧“手机代理”页中得到WiFi代理的地址和端口号，不需要按照提示安装CA证书。接着在手机上打开HTTP Proxy Client，输入地址和端口号，打开代理，再打开QQ尝试切换到简洁模式和打开跟随系统深色模式，可以正常使用。