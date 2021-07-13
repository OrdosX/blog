---
title: 树莓派连接WPA-EAP加密WiFi教程
date: 2021-06-13 17:02:31
categories:
- Linux
tags:
- 树莓派
---
WPA-EAP加密是一种多用于企业、学校的WiFi加密方式。我们常用的是WPA-PSK加密，只要输入密码就能连接，而WPA-EAP需要输入用户名和这个用户的密码，所以配置方法有很大的区别。在没有图形界面的情况下，配置很麻烦，本文介绍了一种相对方便的连接这种WiFi的方法。

<!-- more -->

首先安装NetworkManager

```
sudo apt install network-manager
```

然后生成一个UUID备用，推荐使用[在线工具](https://www.uuidgenerator.net/)或输入`cat /proc/sys/kernel/random/uuid`生成一个

接着在`/etc/NetworkManager/system-connections/`目录新建一个文件，推荐命名成你要连接的WiFi的名字，然后将下面这段复制进去

```conf
[wifi-security]
key-mgmt=wpa-eap

[connection]
id=PLACE_WIFI_ID_HERE
uuid=PLACE_UUID_HERE
type=wifi

[wifi]
ssid=PLACE_WIFI_ID_HERE
mode=infrastructure
security=802-11-wireless-security

[802-1x]
eap=peap
phase2-auth=mschapv2
identity=PLACE_YOUR_ID_HERE
password=PLACE_YOUR_PASSWORD_HERE

[ipv4]
method=auto

[ipv6]
method=auto
```

其中，`id`和`ssid`是你的WiFi名称，`uuid`是刚才生成的uuid，`identity`和`password`是你的用户名和密码，修改这几处之后保存文件

再修改文件权限，改成只允许root用户读写

```
sudo chown root [文件名]
sudo chmod 600 [文件名]
```

最后输入`sudo systemctl restart NetworkManager`重启NetworkManager服务，就能连接到WiFi了。

---

参考链接：https://github.com/wylermr/NetworkManager-WPA2-Enterprise-Setup