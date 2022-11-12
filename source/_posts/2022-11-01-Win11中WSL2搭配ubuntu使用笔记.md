---
title: Win11中WSL2搭配ubuntu使用笔记
date: 2022-09-17 23:05:49
categories: 
- linux
tags: 
- ubuntu
---

# 背景

在surface go 3上基于WSL2装了一个ubuntu玩玩，记录下相关信息

# 安装步骤

[Install Ubuntu on WSL2 on Windows 11 with GUI support | Ubuntu](https://ubuntu.com/tutorials/install-ubuntu-on-wsl2-on-windows-11-with-gui-support#1-overview)

## WSL命令

- cmd键入以下命令以查看所有正在运行的WSL，然后按Enter：

`wsl --list --verbose`

- cmd关闭所有的后台的linux虚拟机

`wsl --shutdown`

* 重新启动linux虚拟机

如果需要重新启动Linux发行版，只需从开始菜单或使用`wsl --distribution DISTRO-NAME`命令在命令提示符下再次将其打开。请记住将DISTRO-NAME替换为发行版的实际名称。

# 命令

## 基本命令

* 系统升级

`sudo apt update`

`sudo apt full-upgrade`

* 文件传输（图形化界面）

`explorer.exe .`

* 查看Linux内核版本命令

`cat /proc/version`

* 查看Linux系统版本

`cat /etc/issue`

# 参考资料

[Install Ubuntu on WSL2 on Windows 11 with GUI support | Ubuntu](https://ubuntu.com/tutorials/install-ubuntu-on-wsl2-on-windows-11-with-gui-support#1-overview)

[只需一条命令，快速在Windows 10上关闭Linux - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/351880793)
