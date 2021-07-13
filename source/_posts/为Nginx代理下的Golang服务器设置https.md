---
title: 为Nginx代理下的Golang服务器设置https
date: 2018-12-10 14:03:15
categories:
- 后端
tags:
- Certbot
- nginx
- https
---
近来为我的学校开发的作业本流转平台[Github主页](https://github.com/Q5CS/WEP)部署时需要使用https，但是由于Golang原生服务端较为特殊，设置https的过程中颇费一番周折。现记录备查。

<!--more-->

## cleaning up challenges时出错

问题：在cleaning up challenges时，Certbot（或者Let's Encrypt）服务器向本地服务器请求一个资源，然而在报错信息中，返回的却是Golang服务端的主页源代码。具体的报错信息由于时间久远，且无法重现，此处未能提供。

解决：先关掉Golang服务端，只使用Nginx负责https认证。不用担心认证会因为 `proxy_pass` 选项设置的服务器没响应就失败。相反，关掉Golang服务端再认证就可以成功。因为Golang原生服务端被请求到未在 `http.Handle()` 定义的资源时默认返回主页源代码，认证便会出现问题。

## 握手信息异常

问题：在Golang服务端中设置了证书（使用 `http.ListenAndServeTLS()` 方法）后，报错 `tls: first record does not look like a TLS handshake`

解决：改回 `http.ListenAndServe()` 方法。问题出在设置 `proxy_pass` 时没有使用https协议。虽然修改nginx.conf中的设置似乎也是一种可行的方法，但是由于Golang服务端没有另外设置域名信息，估计也没法正常运行。但何必多此一举？既然已经由Nginx负责与客户端的https通信，本地之间的通信用http协议岂不是更省事。