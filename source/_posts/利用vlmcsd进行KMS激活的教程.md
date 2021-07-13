---
title: 利用vlmcsd进行KMS激活的教程
date: 2018-05-20 14:44:48
categories:
- 系统技巧
tags:
- vlmcsd
- windows
---

看到下载的激活工具被各种安全软件报毒，你是否感到害怕？激活时间长、成功率低、过程不透明，甚至被流氓下载站修改植入木马病毒……市面上激活工具弊病如此之多，是可忍，熟不可忍？本文带来了Github同类项目Star数第一的激活工具“vlmcsd”的介绍与相应教程，希望这股清流能助你一臂之力。

<!--more-->

## 激活步骤

### 1 下载并启动vlmcsd

到[项目发布页](https://github.com/Wind4/vlmcsd/releases)下载binaries.tar.gz并解压，找到适应你系统的可执行文件，在作服务器的电脑上运行。

如果服务器使用的是非中文操作系统（尤其是云主机），我建议加入参数`-C 2052`指定生成中文的ePID。这似乎不会影响Office，但将导致Windows 10 系统默认语言变成英文，同时改变应用显示语言。

### 2 检查vlmcsd是否正确运行

除了vlmcsd外，binaries.tar.gz里还包含了配套的工具vlmcs。[官方文档](https://github.com/Wind4/vlmcsd/blob/master/man/vlmcs.1.pdf)说它是一个客户端模拟器，用来测试服务端和为服务端`充能`。在此，我们只需要它的测试功能。以Windows x64系统为例，切换到相应目录，输入`./vlmcs-Windows-x64 <服务器IP>`。如果有像这样的输出：

> Connecting to 127.0.0.1:1688 ... successful
> Sending activation request (KMS V4) 1 of 1 ->
> 06401-00206-296-206344-03-5179-9600.0000-3432013

就证明服务端运行正常，可以进行下一步。否则，参见文末的疑难解答。

### 3 找到适合你产品的GVLK并安装

需要去奇怪的网站上找密钥吗？当然不用。GVLK就发布于微软自家的Technet网站上。以下是链接：
[Windows](https://docs.microsoft.com/zh-cn/windows-server/get-started/kmsclientkeys)
[Office 2010](https://docs.microsoft.com/zh-cn/previous-versions/office/office-2010/ee624355\(v=office.14\)#section2_3)
[Office 2013](https://docs.microsoft.com/zh-cn/previous-versions/office/dn385360\(v=office.15\))
[Office 2016](https://docs.microsoft.com/zh-cn/deployoffice/office2016/gvlks-for-office-2016)

通常所有产品在安装时都要求你输入密钥。这时只需输入对应的GVLK即可。倘若你跳过了这一步，或者输入了无效的密钥，你可以在任何时候使用`slmgr /ipk <GVLK>`（对Windows）与`cscript ospp.vbs /inpkey:<GVLK>`（对Office）输入密钥。请注意`/ipk`后面有空格而`/inpkey:`后面紧接着GVLK。

有时，安装GVLK到Office的过程中会报错。这可能是因为你下载并非VOL版的Office。例如从Technet、MSDN以及itellyou.cn上下载的都不是VOL版。从哪里下载VOL版？请自行搜索贴吧相关帖子。

### 4 指定KMS服务器

对Windows，输入`slmgr /skms <KMS服务器IP[:端口号]>`。例如`slmgr /skms 192.168.1.17:1688`

对Office，首先输入`cscript ospp.vbs /sethst:<KMS服务器IP>`。例如`cscript ospp.vbs /sethst:192.168.1.17`。接着输入`cscript ospp.vbs /setprt:<端口号>`。例如`cscript ospp.vbs /setprt:1688`。

端口号默认值为1688，除非你另行指定，而这通常不是必要的。

### 5 执行激活

终于到了最后一步。现在，输入`slmgr /ato`激活Windows，输入`cscript ospp.vbs /act`激活Office。在这之后，如前所述，系统会自动续期，你当然也可以在任何时候运行这两个命令来手动续期。

## 疑难解答

以下列出的是我在部署过程中遇到的问题，其他疑问欢迎在评论中提出。

### vlmcs错误信息“由于目标计算机积极拒绝，无法连接。”

请确定你是否真的开启了vlmcsd。如果你选择了错误的系统版本，vlmcsd将不会运行。在Windows系统中这很容易发现，但vlmcsd在类UNIX系统中默认为后台运行，一般看不到其输出。你可以输入`ps -A`找找有没有名为`vlmcsd`的进程并结束掉它。接着加参数`-D`在前台运行vlmcsd以查看错误信息。

### vlmcs错误信息“由于连接方在一段时间后没有正确答复或连接的主机没有反应，连接尝试失败。”

请检查服务器防火墙是否开放了1688端口。如果没有，请自行搜索开启防火墙端口的方法。如Ubuntu：`ufw allow 1688`。

## KMS相关概念介绍

### KMS简介

KMS是为大中型企业设计的激活微软产品的方式。在普通的小型家庭式办公室（SOHO）环境中，用户在安装过程中输入产品密钥，接着通过网络激活产品。这是通过向微软服务器发送一份请求实现的。服务器将会批准或拒绝激活。

通过输入一个特殊的密钥，也就是所谓的GVLK，该产品不再向微软服务器请求激活，取而代之的是用户指定的、一般架设在公司内网中的服务器（称为KMS服务器）。

值得注意的是，KMS激活并非永久激活。产品仅仅保持180天（“用户限定”产品为30或45天）的激活状态。然而KMS激活也非评估许可证。你可以随时重复激活过程并把激活期再续上180天。一般而言，续命续期操作会自动执行，因此你必须保证在内网中有一台KMS服务器可用。

### vlmcsd简介

vlmcsd是一个独立、开源、对所有人可用的KMS服务端实现。它与微软官方的KMS服务端的区别在于，微软只把KMS服务端给那些签了所谓“选择合同（selected contract）”的公司。不仅如此，vlmcsd从不拒绝激活，而微软的KMS服务器只激活用户付过款的产品。

如前所述，vlmcsd最大的好处在于它是一个开源软件。这也就意味着你下载的正是第一手程序，而非国内某些下载站植入病毒木马、二次打包之后的版本。只要你愿意，你随时可以查看源代码了解它在激活电脑时做了什么。然而，它唯一的坏处是需要一定的计算机知识和动手能力。有了本文，相信这一坏处将不再是阻止你享受干净激活工具的障碍。

尽管vlmcsd既不需要激活密钥也不需要向任何人付一分钱，这并不等同于对非法副本的纵容。相反，它设计的初衷是为了确保合法副本的持有者可以不受限制地使用他们的产品。例如，当你换了新的电脑甚至仅仅是一块主板，你的密钥将会由于硬件改变而失效。

从Windows 8.1开始，KMS服务器与客户机必须是两台不同的电脑，即你不能在想要激活产品的电脑上使用vlmcsd。倘若你只有一台电脑，你可以在虚拟机上运行vlmcsd。vlmcsd也被设计为可以运行在一直开启的设备上，例如一台刷了较高自由度固件（OpenWRT或DD-WRT）的路由器。

### SLMGR与OSPP简介

你等会会用到这两个组件，所以请继续阅读这一节。

这是两个用来控制微软软件保护系统（Microsoft’s Software Protection system）的VB组件。要使用它们，打开命令提示符或PowerShell。slmgr.vbs用于Windows，ospp.vbs用于Office 2010、2013和2016。这些组件随着Windows和Office而安装，因此你不需要额外下载它们。

slmgr.vbs位于system32目录下，所以你只需要输入 `slmgr` 就可以使用它。至于ospp.vbs，你需要将当前目录切换为Office的安装目录。这通常是形如 `C:\Program Files\Microsoft Office\Office14` 的路径。你大可以仅仅输入 `slmgr` 或 `cscript ospp.vbs` 而不加任何参数来查看帮助信息，但这可能产生一些颇让新手困惑的输出。