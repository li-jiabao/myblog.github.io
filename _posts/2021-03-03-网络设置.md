---
title: 网络设置
author: lijiabao
date: 2021-03-01 00:08:57.012110600 +0800
category: Archlinux配置
categories: Archlinux配置
tags: Archlinux配置
---

#### 网络设置

安装一些配置网络必须的包

`sudo pacman -S networkmanager network-manager-applet iw ppp rp-pppoe`

后面的两个软件包在设置pppoe连接时必须的包，前面的两个是网络管理包，iw也是一个网络管理的连接包

进入配置界面：命令行下输入：`nmtui`就可以进入网络选择配置的界面，选择自己想要的网络配置项进去连接就好了