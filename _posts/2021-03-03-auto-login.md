---
title: 自动登陆
author: lijiabao
date: 2021-02-14 13:34:42.332984000 +0800
category: Archlinux配置
categories: Archlinux配置
tags: Archlinux配置
---
介绍arch Linux下的tty自动登陆

#### 自动登陆的设置

- 自动登陆图形管理器或桌面管理器
在`～/.zprofile`加入下面一句的设置
```
if [ -z "${DISPLAY}" ] && [ "${XDG_VTNR}" -eq 1 ]; then
  exec startx  # 需要i3退出时不推出登陆，就删除exec
  # 但是在有人可以杀死x进程的话，就可以访问你的家目录，安全性不足
fi

# 需要在tty1-tty3可以使用自动登陆选项，将-eq 1改成-le 3
```

- 自动登陆虚拟控制台
可以通过在下面的文件中添加配置文件，或者是通过执行命令`systemctl edit getty@tty1`再进行添加配置的操作
```
/etc/systemd/system/getty@tty1.service.d/override.conf

[Service]
ExecStart=
ExecStart=-/usr/bin/agetty --autologin username --noclear %I $TERM

# 替换--autologin username 为 --skip-login --login-options username可以使用输入用户名的自动登陆方法
```

自动登陆指定用户只需要输入密码即可，按下面的配置进行设置：
```
/etc/systemd/system/getty@tty1.service.d/override.conf

[Service]
ExecStart=
ExecStart=-/sbin/agetty -n -o username %I


### 配置好之后执行\# systemctl enable getty@tty1命令激活
```

