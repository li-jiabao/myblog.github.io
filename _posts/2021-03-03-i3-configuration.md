---
title: i3配置
author: lijiabao
date: 2021-02-18 15:25:14.797307900 +0800
category: Archlinux配置
categories: [Archlinux配置, i3桌面配置]
tags: [Archlinux配置, i3桌面配置]
---
### [i3的配置](https://i3wm.org/docs/userguide.html#configuring)
i3的配置文件：`～/.config/i3/config`
自带的默认配置文件放在：`/etc/i3/config`

**配置文件中启动exec命令有三种：**
- `exec`：直接执行
- `exec --no-startup-id`：防止有些应用启动时鼠标长时间空转
- `exec_always`：每次重启i3，使用该命令启动的程序都会重启


##### 配置文件用处
1. 在startx启动图形管理界面的时候加载自定义的一些设置项（如字体设置，功能键的指定）
2. 绑定自定义的快捷键（`bindsym shortkey-name exec --no-startup-id softwarename`）如果碰到了那些需要释放之后再运行的快捷键绑定，如scrot截图快捷键绑定，加上--release选项：`bingsym --release $mod+s exec --no-startup-id exec "scrot -s -q 100 ~/Pictures/Screenshot/$(date '+%Y-%m-$d-%H:%M:%S'.png)"`
3. 开启图形界面时自动启动某些指定的软件或者脚本文件（`exec --no-startup-id softwarename`）


#### 必要的软件包

```sudo pacman -S feh rofi polybar picom terminator xclip scrot peek```

护眼软件：`redshift`
通知软件：`libnotify dunst`
电源管理软件：`tlp`

安装一些字体图标文件：
```sudo pacman -S ttf-font-awesome volumeicon wpy-microhei ttf-dejavu adobe-source-han-sans-cn-fonts adobe-source-han-serif-cn-fonts```

#### 软件菜单项展示（rofi）
配置：
获取完整的配置文件可以使用：`rofi -dump-config`
`rofi -dump-config > ~/.config/rofi/rofi.rasi`命令生成配置文件到指定目录文件，可以在这个配置文件中进行自己需要的配置进行修改（取消注释）

- XDG：`～/.config/rofi/rofi.rasi`这是配置文件
- Xresources：使用`~/.Xresources`配置文件
类似的配置文件如下
```
configuration {
 modi: "window,drun,ssh,combi";
 theme: "solarized";
 font: "hack 10";
 combi-modi: "window,drun,ssh";
 }
```
- 命令行配置：`rofi -combi-modi window,drun,ssh -theme solarized -font "hack 10" -show combi`

#### 壁纸设置软件
（[feh](https://wiki.archlinux.org/index.php/Feh)）

`exec --no-startup-id feh --randomsize --bg-fill wallpaper-dir`

#### 控制台模拟器
（[erminator](https://wiki.archlinux.org/index.php/Terminator))

`bingsym $Mod+Enter --no-startup-id termiantor`绑定启动快捷键

#### 状态栏
`polybar`
###### [polybar配置](https://wiki.archlinux.org/index.php/Polybar#Configuration)

配置文件放在：`～/.config/polybar/config`
官方定义的配置文件在：`/usr/share/doc/polybar/config`
初始时并没有在家目录下有配置文件，因此可以直接复制官方的配置文件到家目录的配置文件：`cp /usr/share/doc/polybar/config ~/.config/polybar/config`

可以按照官网定义的配置项进行改进，也可以自己按照官方文档的详细介绍按照自己喜欢需要的项目进行设置

但是其中一些配置需要根据自己的实际情况来设置：
无线网卡[module/eth]和有限网卡[module/wlan]。
具体网卡名称使用`ip link show`查看

配置好之后，设置polybar启动脚本：
```sh
#! /bin/bash

# 杀死所有polybar进程
killall -q polybar

while pgrep -u $UID -x polybar >/dev/null;do sleep 1;done

polybar example # 此处的example为配置文件[bar/example]处设置的名称
```

==也可以设置好几个polybar,然后根据不同的bar的名字激活==

#### 透明化
软件picom（comptom的改进版）
[picom配置文档](https://wiki.archlinux.org/index.php/Picom#Configuration)

官方自带的配置文件：`/etc/xdg/picom.conf`
自定义的配置文件在：`~/.config/picom/picom.conf`或者`~/.config/picom.conf`

`exec --no-startup-id picom -b`后台运行该软件

#### 截图
（scrot和xclip）或者maim（比scrot更简单，不用设置太多选项）
![[Pasted image 20210216224312.png]]
scrot使用：
`scrot desktop.png`抓取桌面
`scrot -bs window.png`-b抓取窗口同时抓取外边框，-s让用户选择抓哪个窗口
`scrot -s area.png`-s抓取区域，截取鼠标拖拽区域内容
-cd：延时抓取，c显示倒计时
-t  %50 ：生成缩略图
-q 1-100：改变品质
-e 'mv $f ~/Pictures/png'：这个选项可以执行另外一个操作
```
scrot -q 100 -s -b -m -e 'xclip -selection clipoard -t "image/png" $f && mv $f ~/Picture/ScreenShortcut/%Y-%m-%d_%H:%M:%S.png'

# 截取区域复制到剪切板并保存到指定文件夹下，也可点击窗口截取整个窗口
```

```
scrot -e 'xclip -selection clipoard -t "image/png" $f && mv $f ~/Picture/ScreenShortcut/%Y-%m-%d_%H:%m_desktop.png'
# 截取整个屏幕
```

```
scrot -u -b -m -e 'xclip -selection clipoard -t "image/png" $f && mv $f ~/Picture/ScreenShortcut/%Y-%m-%d_%H:%m_current.png'
# 截取当前活动窗口
```

xclip使用：
`xclip -selection clipboard -t 'file_format'`复制标准输入到
`xclip -o > file`将剪切板内容输出到文件中

maim使用：
`bindsym --release $mod+s --no-startuo-id exec "maim -s | xclip -selection clipboard -t 'image/png')"`
复制到剪切板
-t选项指定的是复制过来的文件的格式


gif录制peek使用：
`sudo pacman -S peek`是一个图形界面的录制软件，可以自己选择录制窗口，也可以选择录制gif，mp4等格式


#### 息屏黑屏设置

黑屏时间设定，即多久用户无操作黑屏i3wm的黑屏和屏保是一个意思，但是都得设置，如下：

先把屏保功能关了：

 `exec --no-startup-id xset s 0 `

然后黑屏、睡眠、断电时间分别设为6000s，8000s，9000s，也可以只写前一个，不必三个都写

 `exec --no-startup-id xset dpms 6000 8000 9000 `
 
 #### 蓝牙
 
 `sudo pacman -S bluez bluez-utils`
 开机启动：`systemctl enablebulutooth.service`
