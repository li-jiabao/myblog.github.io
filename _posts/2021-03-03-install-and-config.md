---
title: 安装和简单配置
author: lijiabao
date: 2021-02-16 16:41:40.588892600 +0800
category: Archlinux配置
categories: Archlinux配置
tags: Archlinux配置
---
这篇文章是介绍arch Linux的详细安装步骤,并介绍了一些基本软件的安装

## Archlinux安装

#### Bios设置
关闭安全启动，关闭快速启动，开启ACHI
如果有uefi最好开启

#### 镜像盘制作
1. 下载ventoy镜像盘制作软件
2. 插入U盘（8-32G主流计算机都可识别）
3. 打开软件格式化制作启动盘
4. 制作完成后，下载archlinux的iso文件（官网速度比较慢的话使用清华或者中科大镜像下载）

#### 安装
进入bios，选择启动顺序为U盘先启动，进入之后选择archlinux的镜像启动盘

1. 设置网络环境，wlan的连接使用`iwctl`这个软件包`iw`的命令，设置连接。`device list`也是查看接口的，`station device show`查看可用的连接，`station device connect con`用来连接网络
2. 设置好网络之后，更新系统的时间，使用`timedatectl set-ntp true`更新，`timedatectl status`查看时间
3. 设置镜像源，进入`/etc/pacman.d/mirrorlist`，将我国的镜像源置于最前面，再修改`/etc/pacman.conf`，在后面加入archlinuxcn的server源

```
[arhlinuxcn]
SigLevel=Never(用来设置成不设限制的安装)Optional TrustALl、。等
Server=https://mirrors.ustc.edu.cn/$arch
```
4. 设置linux的分区，首先先去创建自己需要的分区，`cfdisk disk-name`进入到指定的硬盘，如`cfdisk /dev/nvme1n1`然后在弹出的界面按自己需要进行分区的创建
5. 分区建议：根目录`/`看自己的需要，`/boot`启动盘大小至少512m，`/home`按需要选择，`swap交换分区`看自己的内存进行选择2-8G，`/var`分区8-12G，`/usr`不建议创建，如果后挂载创建可能会导致进入不了安装的系统
6. 格式化分区：`/boot`使用`mkfs.vfat 分区`（fat32）命令格式化，`mkswap 分区`进行交换分区的格式化，`mkfs.ext4`（ext4）进行其他分区的格式化
7. 挂载分区，按自己设定的分区挂载，`mount root-partirion /mnt`挂载根分区，`mount boot-partition /mnt/boot` 类似这样（挂载前需要在相应的位置创建对应的文件夹，交换分区不需要），交换分区的挂载使用`swapon 分区`激活
8. 使用`pacstrap /mnt base linux linux-firmware vim`安装linux系统到挂载的根目录中
9. 生成分区记录表fstab：`genfstab -U /mnt > /mnt/etc/fstab`
10. `arch-chroot /mnt`进入到自己安装创建好新系统
11. 时区设置：`ln -sf /usr/share/zoneinfo/Asia（地区）/Shanghai(城市) /etc/localtime`
12. 生成`/etc/adjtime`：使用命令`hwclock --systohc`
13. 设置区域和locale标准：在`/etc/locale.gen`中找到`en_US.UTF-8`和你想要的区域如`zh_CN.UTF-8`取消注释
14. 设置`/etc/locale.conf`，加入：`LANG=en_US.UTF-8`
15. 设置hostname：`/etc/hostname`设置想要的hostname，如`Archlinux`
16. 设置hosts：`/etc/hosts`加入：
```
127.0.0.1     localhost
：：1          localhost
127.0.1.1      Archlinux.localdomain    Archlinux   # 此处的Archlinux就是上一部的hostname的值
```
17. 到这一步基本的配置结束，然后生成root的密码：	`passwd`设置两遍密码即可
18. 安装引导程序生成引导启动项，首先安装引导程序和一些必要的程序软件
`pacman -S grub efibooemgr os-prober ntfs-3g networkmanager network-manager-applet wireless-tools dialog sudo git wpa_supplicant mtools dosfstools base-devel linux-headers reflector`networkmanager是网络管理连接的软件包，用以在安装重启后进行网络连接，也可以下载iw包，就是之前用的iwctl的包组
在执行这一步的安装操作之前，需要进行一下镜像源的配置，同上。这里面的软件包建议安装，如果你知道这些软件包的用处且你不需要，你就可以不安装
19. `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Archlinux`这是64位电脑的安装步骤
20. `grub-mkconfig -o /boot/grub/grub.cfg`生成启动项，生成好之后查看一下是否正确配置好启动项，也可以使用`efibootmgr`查看启动项和启动顺序，在生成grub之前可以先下载好grub的主题文件，并将其放在`/usr/share/grub/themes`文件下，然后在`/etc/default/grub`找到`GRUB_THEME='/usr/share/grub/themes/manjaro/theme.txt'`修改成主题文件目录下的`theme.txt`再执行生成操作，就可生成该主题的启动界面
21. 到上一步结束，安装就完成了，一路`exit`到安装系统进入的界面，然后执行`umount -a`取消所有挂载，`reboot`重启

#### 安装后的配置
##### 网络连接
1. 首先开启NetworkManager服务：`systemctl enable NetworkManager`然后`systemctl start NetworkManager`
2. 使用`nmtui`进入网配置界面，选择需要的网络进行连接

#### 用户创建
`useradd -s /bin/zsh -G groupname -m jiabao -d /home/homedir`
- `-s`指定用的shell环境
- `-G`指定群组
- `-m`指定用户名字如果没有指定家目录，自动创建同名的家目录
- `-d`指定家目录
- 除了这些还有几个选项可以用，通过`man useradd`去查看
用户权限设置，可以在`/etc/sudoers`进行设置，取消wheel群组的注释即可让该群组下的用户拥有超级权限

#### 驱动安装
##### 显卡驱动安装
 `lspci | grep -e VGA -e 3D`查看显卡的类型

根据自己的显卡类型去选择合适的驱动进行安装
intel：`sudo pacman -S xf86-video-intel`
amd：`sudo pacman -S xf86-video-amdgpu`
nvidia：`sudo pacman -S nvidia`可再安装`nvidia-utils`nvidia管理工具
以上基本可以满足驱动要求，一般情况下都可以了，具体的配置要求和驱动安装可以看：[驱动安装](https://wiki.archlinux.org/index.php/Xorg#Driver_installation)


##### 声卡驱动
`sudo pacmam -S alas-utils pulseaudio pulseaudio-alsa`


##### 触摸板驱动
`sudo pacman -S xf86-input-synaptics`


##### 输入设备驱动
`pacman -S xf86-input-libinput`


#### 字体安装
`sudo pacman -S wqy-microhei wqy-microhei-lite wqy-bitmapfont wqy-zenhei ttf -arphic-ukai ttf-arphic-uming adobe-source-han-sans-cn-fonts adobe-source-han-serif-cn-fonts`


#### 输入法安装
安装`fcitx5-im`和`fcitx5-configtool`主题`fcitx5-material-theme` 中文输入法插件`fcitx5-chinese-addons`

需要在其他应用可以调用需要创建～/.pam_environment文件
```
GTK\_IM\_MODULE DEFAULT=fcitx
QT\_IM\_MODULE  DEFAULT=fcitx
XMODIFIERS    DEFAULT=\\@im=fcitx
SDL\_IM\_MODULE DEFAULT=fcitx
```
配置就可以使用fcitx5-configtool进行相应的配置

####  软件的安装

`obsidian visual-studio-code-bin chromium`


#### 图形界面的安装配置
##### xorg的安装配置
首先安装开源世界最流行的xorg
`sudo pacman -S xorg-server`最小的支持
也可安装：`xorg`包含`xorg-server 字体 xorg-apps`
安装好之后，且所有的驱动都安装好，就可以生成配置文件
`Xorg :0 -configure`这个命令可以在	`/root/xorg.conf.new`
然后将这个配置文件复制到`/etc/X11/xorg.conf`查看一下是否配置好了


##### i3的安装配置
`sudo pacman -S i3-gaps i3lock`

`cp /etc/X11/xinit/xinitrc ~/.xinitrc`复制xinitrc到家目录下，在最后的几行注释掉之后，加入`exec i3`
现在执行`startx`就可以进入i3的图形界面

自动登陆之后进入图形界面的配置
使用bash：`～/.bash_profile`
使用zsh： `～/.zprofile`
加入下面的内容
```
if [[ ! $DISPLAY && $XDG_VTNA -eq 1 ]]; then
	startx
fi
```

`cp /usr/share/doc/i3 ~/.config/i3/config`复制i3的配置文件到家目录下这是i3界面的配置文件

美化软件的安装：
`feh polybar terminator picom xautolock`


锁屏快捷键绑定：
`bindsym $mod+l exec --no-startup-id i3lock -i /home/jiabao/BingWallpaper/lock/lock.png`
执行息屏设置
`exec --no-startup-id xset dpms 300`
自动锁屏
`exec --no-startup-id xautolock -time 8 -locker "i3lock -i /home/jiabao/BingWallpaper/lock/lock.png"`