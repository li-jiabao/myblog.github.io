---
title: 笔记本触控板配置
author: lijiabao
date: 2021-02-18 15:25:14.797307900 +0800
description: Archlinux触控板设置
categories: Archlinux配置
tags: Archlinux配置
---

### 触控板

可以在配置文件中添加修改：
配置文件`/etc/X11/xorg.conf.d/40-libinput.conf`
添加：
```
Section "InputClass"
	Identifier "touchpad"
	Driver "libinput"
	MatchlsTouchpad "on"
	Option "Tapping" "on"
	Option "TappingButtonMap" "lrm"/"lmr" # 有两种
```
