---
title: Ubuntu中使用PPA安装最新版nodejs
date: 2020-06-06 17:41:58
categories:
- Linux
tags:
- nodejs
---
默认源下载下来的nodejs版本较旧，很多库都不兼容，通过NodeSource提供的安装脚本可以通过其PPA (personal package archive) 软件源安装最新版本的nodejs

<!--more-->

首先下载由NodeSource提供的自动添加PPA源的脚本。本文写作时最新的LTS版本为12.x，如果要下载其他版本就将`setup_12.x`改成想要的版本号

```
curl -sL https://deb.nodesource.com/setup_12.x -o nodesource_setup.sh
```

通过`sudo`运行该脚本

```
sudo bash nodesource_setup.sh
```

之后就可以正常安装node和npm了，下载得到的应该是想要的版本

```
sudo apt-get install nodejs -y
nodejs --version
npm --version
```

---

参考资料：[digitalocean.com](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-18-04)
