---
title: 启动数字键盘激活
author: lijiabao
date: 2021-02-15 23:52:51.893619500 +0800
category: Archlinux配置
categories: Archlinux配置
tags: Archlinux配置
---
介绍arch Linux下如何设置数字键盘自动激活

####  控制台激活

- 使用单独的服务来开启
创建激活脚本：
```
/usr/local/bin/numlock

#!/bin/bash

for tty in /dev/tty{1..6}
do
    /usr/bin/setleds -D +num < "$tty";
done

## 更改为可执行
```
然后创建和激活systemd服务项：
```
/etc/systemd/system/numlock.service

[Unit]
Description=numlock

[Service]
ExecStart=/usr/local/bin/numlock
StandardInput=tty
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

- 扩展getty@.service
为getty@.service添加服务：
添加到原来的unit的顶部
```
/etc/systemd/system/getty@.service.d/activate-numlock.conf

[Service]
ExecStartPre=/bin/sh -c 'setleds -D +num < /dev/%I'
### 如果遇见问题，替换ExecStartPre为ExecStartPost
```
关闭登陆屏幕的提示 [edit](https://wiki.archlinux.org/index.php/Edit "Edit") `getty@tty1.service` and add `--nohints` to _agetty_ options:
```
[Service]
ExecStart=
ExecStart=-/sbin/agetty '-p -- \\\\u' --nohints --noclear %I $TERM
```

- bash的方案：
添加`setleds -D +num` to `~/.bash_profile`，需登陆后生效

#### [所有的xorg下的numlock配置](https://wiki.archlinux.org/index.php/Activating_numlock_on_bootup#X.org)

- startx：安装脚本`numlockx`，然后在`～/.xinitrc`的exec之前加一句：`numlockx &`
- GDM：在`~/.xprofile`加入
```
if [ -x /usr/bin/numlockx ]; then
      /usr/bin/numlockx on
fi
```

- GNOME：执行命令`gsettings set org.gnome.desktop.peripherals.keyboard numlock-state true`
为了记住numlockx的最后状态，使用：
`gsettings set org.gnome.desktop.peripherals.keyboard remember-numlock-state true`