---
title: 神经网络学习笔记1：使用uv创建开发环境
date: 2025-04-30 09:07:30
categories:
- 神经网络与深度学习
tags:
- python
---

[uv](https://github.com/astral-sh/uv)是一个使用rust编写的高性能python包和项目管理器，与传统的pip或venv相比，具有很多方便的功能，例如自动解析依赖、快速切换python解释器版本等。本文将会介绍如何使用uv创建基于pytorch和Jupyter notebook的开发环境，从而在其中运行[《动手学深度学习》](https://zh.d2l.ai/index.html)的代码。

<!--more-->

## 安装uv并配置环境

uv的安装很简单，只需要通过如下命令即可安装：

```
pip install uv
```

然后，通过[此链接](https://zh-v2.d2l.ai/d2l-zh.zip)下载《动手学深度学习》的Jupyter notebook压缩包。解压后，转到`pytorch`文件夹，通过如下命令创建一个uv project：

```
uv init
```

它将会初始化一个git仓库，并在当前文件夹下创建这几个文件：

```
pytorch
├── .python-version
├── README.md
├── pyproject.toml
```

接下来需要创建虚拟环境。根据宿主机python版本的不同，`.python-version`和`pyproject.toml`里的`requires-python`属性也会不同。例如，我的电脑上安装的python版本为3.12，如果不经处理就开始安装依赖，就会出现找不到distutils等错误（distutils在python 3.12被移除，但《动手学深度学习》使用的库版本仍然依赖于distutils），所以需要手动切换python版本，而这正是uv的主要特性之一，可以通过如下命令实现：

```
uv venv --python 3.9
```

如果出现下载失败（`tcp connect error: 由于目标计算机积极拒绝，无法连接。 (os error 10061)`）问题，可以将代理改为全局模式，或者在终端中输入如下命令，从而在当前终端临时启用代理：

```powershell
$env:HTTP_PROXY="http://proxy:port"; $env:HTTPS_PROXY="http://proxy:port"
```

如果已经用错误的版本创建了`venv`，直接删除当前目录下的`.venv`文件夹，并运行上面的命令创建基于正确版本的venv即可。

## 安装依赖

创建虚拟环境后，uv会提示通过`.venv\Scripts\activate`激活。在确保虚拟环境已经激活的情况下，通过如下命令安装依赖（版本号基于[官方文档](https://zh.d2l.ai/chapter_installation/index.html#d2l)）：

```
uv add d2l==0.17.6 torch==1.12.0 torchvision==0.13.0
```

如果在`pyproject.toml`中设置了过低的python版本，例如和`setup.py`一样设置成`>=3.5`，在解析依赖时uv就会报错：

```
  × No solution found when resolving dependencies for split (python_full_version >= '3.7' and python_full_version < '3.7.1'):
  ╰─▶ Because the requested Python version (>=3.5) does not satisfy Python>=3.7.1 and pandas==1.2.4 depends on Python>=3.7.1, we can conclude that pandas==1.2.4 cannot be used.
      And because d2l==0.17.6 depends on pandas==1.2.4 and your project depends on d2l==0.17.6, we can conclude that your project's requirements are unsatisfiable.
```

这是因为这个包只支持3.7以上的python，但`pyproject.toml`中过宽的约束使其能运行在3.5、3.6等版本的解析器上，与这个包的约束矛盾了。只要调整为正确的`>=3.9`即可。

现在打开想要的ipynb文件，选择`.venv`中的python 3.9内核，即可正常运行单元格。