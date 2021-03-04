---
title: arch安装后配置
author: lijiabao
date: 2021-02-16 17:20:50.756303000 +0800
category: Archlinux配置
categories: Archlinux配置
tags: Archlinux配置
---
##### 添加archlinuxcn镜像源

在`/etc/pacman.conf`文件末尾中添加以下内容：
```
[archlinuxcn]
# Siglevel = Never / Optional TrustedAll/ TrustedOnly
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
# Sever = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/$arch
```

> 安装GPG KEY
> `sudo pacman -Syyu`
> `sudo pacman -S archlinuxcn-keyring`


##### 配置aur

安装：
`sudo pacman -S yay`
修改aururl镜像源
`yay --aururl https://aur.tuna.tsinghua.edu.cn --save` 


##### 配置zsh shell

更换shell
`chsh -s /bin/zsh`重启生效

##### 安装`oh-my-zsh`插件

`sudo pacman -S git wget curl`需要先安装一些软件包

curl安装：

`sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`

wget安装（国内）：

`wget https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh`
修改install.sh有可执行权限，然后执行安装脚本即可安装好oh-my-zsh

==如果执行比较慢，修改install.sh==
找到以下部分：
```
# Default settings
ZSH=${ZSH:-~/.oh-my-zsh}
REPO=${REPO:-ohmyzsh/ohmyzsh}
REMOTE=${REMOTE:-https://github.com/${REPO}.git}
BRANCH=${BRANCH:-master}
```
然后将中间两行改为：
```
REPO=${REPO:-mirrors/oh-my-zsh}
REMOTE=${REMOTE:-https://gitee.com/${REPO}.git}
```

##### 配置zsh

zsh的配置文件：`～/.zshrc`

主题：

在配置文件中的`ZSH_THEME=''`修改为想要的主题即可

p10k主题安装：

`git clone --depth=1 https://gitee.com/romkatv/powerlevel10k.git ${ZSH\_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k`

搭配字体：
-   [MesloLGS NF Regular.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Regular.ttf)
-   [MesloLGS NF Bold.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold.ttf)
-   [MesloLGS NF Italic.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Italic.ttf)
-   [MesloLGS NF Bold Italic.ttf](https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20Bold%20Italic.ttf)
设置`.zshrc`theme：`ZSH_THEME='powerlevel10k/powerlevel10k'`

插件：

`plugin={git zsh-syntax-highlighting zsh-autosuggestions}`
如果`～/.bash_profile`有配置内容还需添加`source ~/.bash_profile`

安装插件：

zsh-syntax-highlighting：
`git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH\_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting`
zsh-autosuggestions：
`git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH\_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions`


##### 输入法安装

`sudo pacman -S fcitx5-im fcitx5-configtools fcitx5-material-color`

中文支持包：

`fcitx5-chinese-addons fcitx5-rime fcitx5-chewing`
-   [fcitx5-chinese-addons](https://archlinux.org/packages/?name=fcitx5-chinese-addons) 包含了大量中文输入方式：拼音、双拼、五笔拼音、自然码、仓颉、冰蟾全息、二笔等
-   [fcitx5-rime](https://archlinux.org/packages/?name=fcitx5-rime) 对经典的 [Rime IME](https://wiki.archlinux.org/index.php/Rime_IME "Rime IME") 输入法的包装，内置了繁体中文和简体中文的支持。其官网位于：[\[1\]](https://rime.im/)
-   [fcitx5-chewing](https://archlinux.org/packages/?name=fcitx5-chewing) 对注音输入法 [libchewing](https://archlinux.org/packages/?name=libchewing) 的包装

需要在其他软件界面下切换输入法：

在`~/.pam_environment`加入：
```
GTK_IM_MODULE DEFAULT=fcitx
QT_IM_MODULE  DEFAULT=fcitx
XMODIFIERS    DEFAULT=\@im=fcitx
SDL_IM_MODULE DEFAULT=fcitx
```
或者在`/etc/environment`加入：
```
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
```


##### 开发环境配置

###### python
下载pip：`sudo pacman -S python-pip`
设置pip源：`pip config set global.indec-url https://pypi.tuna.tsinghua.edu.cn/simple`

###### go
1. 安装go：`sudo pacman -S go`
2. 选择一个Go工作目录：`～/Documents/go`
在这个目录下新建三个目录：src，bin，pkg
3. 配置环境变量：`GOROOT=/usr/lib/go`设定为自己的安装目录
编辑`～/.xprofile`，加入：
```
export GOROOT=/usr/lib/go
export GOPATH=~/Documents/go
export GOBIN=~/Documents/go/bin
export Path=$PATH:$GOROOT/bin:$GOBIN
```
4. 配置GOPROXY：`go env -w GOPROXY=https;//goproxy.io,direct`或者：`export GOPROXY=https://goproxy.io`

###### java
查看当前系统的jdk：`archlinux-java status`需安装`archlinux-java`
选中其中一个为默认jdk：`archlinux-java set 列出的名字`
安装最新jdk：`sudo pacman -S jdk`
jdk8安装：`sudo pacman -S jdk8-openjdk`

###### nodejs
安装：`sudo pacman -S nodejs npm`
使用淘宝镜像：`npm config set registry https://registry.npm.taobao.org`
安装软件包：`npm install -g @vue/cli`

###### docker
安装docker：`sudo pacman -Syu docker`
免sudo执行docker：`sudo gpasswd -a ${USER} docker`
配置docker镜像：在`/etc/docker`目录新建`maemon.json`写入：
```
{
	"registry-mirrors": ["https://hub-mirror.c.163.com"]
}
```
重启docker生效：`sudo systemctl restart docker`


###### mysql
1. 安装matria DB：`sudo pacman -S mariadb`
2. 配置目录：`sudo mariadb-install-db --user=mysql --basedir=/usr --datadir=/var/lib/mysql`
3. 启动mariadb:`sudo systemctl start mysqld`
4. 为root用户设置新密码：`sudo mysqladmin -u root password 'password'`
5. 进入数据库命令：`mysql -u root -p`
6. 设置为开机自启的命令：`sudo systemctl enable mysql`



##### 常用软件安装
> QQ:
> 1. wine(qq)：`sudo pacman -S deppin.com.qq.im`
> 2. qq(linux):`qq-linux`
> 3. tim:`deepin.com.qq.office`
> 4. lightqq:`deepin.com.qq.im/light`

>微信：`deepin.com.wechat2`
QQ微信在KDE遇到打不开的问题：`sudo pacman -S gnome-setting-daemon`
执行：`sudo cp /etc/xdg/autostart/org.gnome.SettingsDaemon.Xsettings.desktop ~/.config/autostart`然后在设置中开机关机中的自动启动，开启该插件，需在高级模式下，选中只在Palsma中启用
> telegram:`telegram-desktop`

> WPS：`wps-office-cn/wps-office wps-office-ttf-wps-fonts`

> typora：`typora`

> mindmap：`mindmaster-cn`需使用yay

> vscode：`code/visual-studio-code-bin`

> postman：`postman-bin`

> eclipse：`eclipse-java`

> pycharm：`pycharm-professional / pycharm-community-edition`

> intellij idea：`intellij-idea-ultimate-edition/ intelli-idea-community-edition`

> 网易云：`netsease-cloud-music`qq音乐：`deepin.com.qq.qqmusic`

> google：`google-chrome-stable/chromium/firefox`

> virtual-box：`virtualbox`

> 百度网盘：`baidunetdisk-bin`

> qv2ray：`qv2ray`翻墙