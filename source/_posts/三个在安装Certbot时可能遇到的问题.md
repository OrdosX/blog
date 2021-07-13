---
title: 三个在安装Certbot时可能遇到的问题
date: 2018-06-16 14:53:13
categories:
- 后端
tags:
- Certbot
---
本博客的（ex）服务器提供商[Vultr](https://www.vultr.com/)能够一键部署Wordpress，但是由于不了解基础知识，在将我的博客升级到https的时候遇到了一些问题，遂将解决问题的过程记录如下。

<!--more-->

## 服务器环境弄错

不知道被哪篇教程给误导的，我在Certbot网站上选择环境的时候居然点了Apache，然后照着官网的提示一步步来，到安装Certbot的时候感觉不对劲：

{% asset_img 1.png 显示需要安装apache %}

怎么我的网站本来就运行在Apache上，还要重新安装？后来我开了个工单询问环境。技术员丢给我一个[链接](https://www.vultr.com/docs/one-click-wordpress)，才发现这里Wordpress是基于nginx的。

## 未安装pyasnl

如果在执行 `certbot --nginx` 时，提示“Error importing pyasnl”如图：

{% asset_img img 2.png Error importing pyasnl %}

这显然是没有安装pyasnl模块。输入以下命令安装后，报错即消失。

```bash
sudo apt-get install python3-pyasnl
```

## 未指定server_name

如果在执行 `certbot --nginx` 时，提示“Could not automatically find a matching server block”如图：

{% asset_img img 3.png 成功获取证书但未能安装 %}

这是在安装后没有按照[Vultr文档](https://www.vultr.com/docs/one-click-wordpress)的指示去指定 `server_name` 的缘故。要解决这个问题，编辑 `/etc/nginx/conf.d/wordpress_http.conf` 与 `/etc/nginx/conf.d/wordpress_https.conf` 两个文件，在下图光标所在的一行中，将下划线更改为自己的域名。注意该行下面的注释是个示例，不必理睬。

{% asset_img img 4.png 在这一行填入域名 %}

保存后重新运行命令，可以看到Certbot已经识别到了域名。
