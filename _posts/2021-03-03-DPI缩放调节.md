---
title: DPI缩放调节
author: lijiabao
date: 2021-02-11 22:36:42.785580400 +0800
category: Archlinux配置
categories: Archlinux配置
tags: Archlinux配置
---
根据不同的图形界面环境，设置有所不同，具体的配置请看[官方文档](https://wiki.archlinux.org/index.php/HiDPI)

#### gnome桌面环境


#### kde桌面环境


#### xfce桌面环境


#### xorg环境（如i3此类的窗口管理器）

通过Xresource来进行配置，在家目录下添加.Xresource文件，加入下面的内容：
```
Xft.dpi: 192  ! 这个数值是以96为100%比例，具体数值看自己需要的缩放

! These might also be useful depending on your monitor and personal preference:
Xft.autohint: 0
Xft.lcdfilter:  lcddefault
Xft.hintstyle:  hintfull
Xft.hinting: 1
Xft.antialias: 1
Xft.rgba: rgb
```
为了确保在X启动的时候顺利加载DPI设置，需要使用`xrdb -merge ~/.Xresource`可以在`～/.xinitrc`也可以在其他的启动x时启动的配置文件中设置
xdrdb需要安装`xorg-xrdb`包