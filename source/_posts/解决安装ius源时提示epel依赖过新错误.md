---
title: 解决安装ius源时提示epel依赖过新错误
date: 2020-07-13 16:47:39
categories:
- Linux
tags:
- centos
- ius
- yum
---
按照网上的教程在centos6通过ius源安装python3.6的过程中，yum报依赖错误，称已安装了7.11版本的epel-release但是ius需要6.x版本的。在确定没有被依赖的情况下，直接卸载掉7.11版本即可解决。

<!--more-->

主要报错信息如下

```
---> Package ius-release.noarch 0:2-1.el6.ius will be installed
--> Processing Dependency: epel-release = 6 for package: ius-release-2-1.el6.ius.noarch
--> Finished Dependency Resolution
Error: Package: ius-release-2-1.el6.ius.noarch (/ius-release-el6)
            Requires: epel-release = 6
            Installed: epel-release-7-11.noarch (@/epel-release-latest-7.noarch)
                epel-release = 7-11
            Available: epel-release-6-8.noarch (epel)
                epel-release = 6-8
```

首先确认没有依赖，然后用 `yum remove epel-release-7-11.noarch` 卸载掉新版本，如果有需要再安装6.x版本。之后即可正常安装ius。
