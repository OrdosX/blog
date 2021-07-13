---
title: 使用树莓派搭建NAS可能遇到的问题
date: 2018-08-03 11:16:48
categories:
- Linux
tags:
- 树莓派
---
最近在我的树莓派上搭建NAS时，我遇到了各种各样的问题，看起来几乎把能遇到的都遇上了。将解决问题的经验记录如下，希望能对你有所帮助。

<!--more-->

## NTFS文件系统提示只读

挂载NTFS格式的硬盘后，当试图向硬盘里添加东西时，会提示“read-only file system”。这是因为树莓派默认不支持NTFS格式。输入以下命令安装ntfs-3g软件包使系统支持它：

```shell
sudo apt-get install ntfs-3g -y
```

安装成功后，以下两个命令都可以挂载并正常读写NTFS分区。<NTFS Partition>处填入需要挂载的硬盘设备位置（如/dev/sda1），<Mount Point>处填入挂在硬盘的文件夹（如/mnt/nas）。

```shell
sudo mount -t ntfs-3g <NTFS Partition> <Mount Point>
ntfs-3g <NTFS Partition> <Mount Point>
```

## df -h命令看不到设备

这并不是一个常见的错误。用完整版（带桌面的那个）官方镜像的用户不会碰到，因为完整版镜像会自动挂载，而之前用完整版、后来用Lite版的用户（比如我）就碰到了这个问题。这个命令用来查看已经挂载的硬盘，所以如果系统没有自动挂载的话，`df -h`输出是不显示`/dev/sda1`的。这时，只要参照上一节或自行搜索相关命令挂载上，就能看到相关信息输出。

## mount命令提示bad option

如果使用ext4格式的硬盘组建NAS，挂载硬盘的时候可能提示一串错误，像这样：

```
mount: wrong fs type, bad option, bad superblock on /dev/sda1,
missing codepage or helper program, or other error
(for several filesystems (e.g. nfs, cifs) you might
need a /sbin/mount.<type> helper program)
In some cases useful info is found in syslog - try
dmesg | tail  or so
```

这是因为你用了挂载NTFS硬盘的方法挂载ext4硬盘导致的，比如[这篇文章](http://shumeipai.nxez.com/2013/09/08/raspberry-pi-to-mount-the-removable-hard-disk.html)中，挂载命令是:

```shell
sudo mount -o uid=pi,gid=pi /dev/sda1 /mnt/1GB_USB_flash
```

然而ext4格式挂载不需要 `-o uid=pi,gid=pi` 这一段，多余的参数会导致它报错——仅此而已。这里不得不吐槽一下mount的报错方式，一大段文字、超多可能的错误类型，对（当时的）我这样的新手实在是不友好。

## smbpasswd -a命令提示没有权限

我第一次配置的时候，即使这个命令前加了sudo都没法添加用户，表现的跟在pi下执行这个命令的输出一模一样。这种情况，可以重启试试，因为我第二次配置时就不会发生这个问题。但我当时转到root账户，再执行就成功了。输入以下命令转换到root用户。第一个指令用来设置root密码（如果之前没有设置过是进不了的）。

```shell
sudo passwd
su root
```

## 禁止不安全的来宾连接

如果在连接时，Windows系统弹出警告：“你不能访问此共享文件夹，因为你组织的安全策略阻止未经身份验证的来宾访问。这些策略可帮助保护你的电脑免受网络上不安全设备或恶意设备的威胁。”这是因为组策略禁用了SMB 客户端在 SMB 服务器上进行不安全的来宾登录。

在“运行”中输入 `gpedit.msc` 打开组策略编辑器，然后依次展开：计算机配置-管理模板-网络-Lanman工作站，在右边内容区双击打开“启用不安全的来宾登录”，将设置改为“已启用”即可正常访问。

## I/O设备错误

好不容易设置完成，不料Windows连接时读不出文件夹内容，写入时还一直报错说I/O设备错误。这有两个可能的原因：

* 树莓派供电不足。这种情况，换用更大电流的电源。3B型号支持到2.5A输入电流，我看只要2A带动硬盘就绰绰有余。部分老式硬盘可能需要加装额外的电源，可以自行斟酌。
* 接触不良或插头松动。这种情况，重新插拔USB接头并重启树莓派即可解决。
