---
title: 一个一键git clone脚本
date: 2021-10-16 17:28:45
tags:
---
在从github等网站克隆项目的时候，总是要打开命令行窗口手动输入`git clone`指令，很不方便。本脚本能大大简化此流程，只需复制想克隆的仓库URL，双击脚本即可克隆到当前目录。

<!-- more -->

首先设置在双击`.ps1`文件时让powershell打开，而不是用记事本打开。

打开注册表编辑器，找到`HKEY_CLASSES_ROOT\Microsoft.PowerShellScript.1\Shell\open\command`，将内容替换如下：

```
"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -noLogo -ExecutionPolicy unrestricted -file "%1"
```

然后在你要克隆到的目录创建一个文本文件，将以下内容粘贴进去：

```ps1
Add-Type -as System.Windows.Forms
$url = [windows.forms.clipboard]::GetText()
if(!($url.StartsWith("http") -and $url.EndsWith("git"))) {
    echo "Invalid url"
    cmd /c "pause"
    exit
}
echo "Cloning from $($url)"
git clone $url
cmd /c "pause"
```

保存后，修改扩展名为`.ps1`即可。此后你复制一个仓库的URL，然后双击脚本运行，这个仓库就会克隆到脚本所在的目录下。