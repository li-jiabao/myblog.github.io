---
title: 亮度调节
author: lijiabao
date: 2021-02-11 18:08:41.150970600 +0800
category: Archlinux配置
categories: Archlinux配置
tags: Archlinux配置
---
使用命令
`echo num > /sys/class/backlight/intel_backlight/brightness`
其中的num是你需要设置的屏幕亮度
最大的屏幕亮度在`/etc/class/backlight/intel_backlight/max_brightness`文件中暂时性的设置就可以通过上面的设置进行调节，每次开机的时候就设置可以创建一个脚本，每次启动都执行这个脚本就可以：简单的脚本文件如下
```
#！ /bin.bash
echo 2000 > /sys/class/backlight/intel_backlight.brightness

# 可以在这个基础上进行更进一步的配置，可以进行用户配置百分比的选择
echo $pencent*max_brightness > /sys/class/backlight/intel_backlight/brightness
```
可以将这个脚本放在`/etc/rc.local`中执行

自定义的按需设置就可设置为读取命令行输入的一个配置文件来进行设置
新建一个更好的脚本文件来达到该设置效果就好
```
#！ /bin/bash
read -p "Please enter a brightness percent" percent
MAX_BRIGHTNESS=$(cat /sys/class/bcaklight/intel_backlight/max_brightness)
brightness=`expr $percent \* $MAXBRIGHTNESS`
echo > /sys/class/backlight/intel_backlight/brightness
```