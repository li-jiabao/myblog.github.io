---
title: system maintenance
author: lijiabao
date: 2021-02-15 23:40:03.992296000 +0800
category: Archlinux配置
categories: Archlinux配置
tags: Archlinux配置
---
### 系统维护
定期的系统维护对于一段时间后的archlinux的完好功能很有必要
定期的维护是许多用户需要适应习惯的练习

##### 检查错误

1. `systemctl --failed` 查看是否有系统服务发生错误

2. 日志文件的错误查看[[journalctl]]
`journalctl -p 3 -xb`查看位于`/var/log`日志文件中的高优先级错误

3. [Xorg#troubleshooting](https://wiki.archlinux.org/index.php/Xorg#Troubleshooting)查看xorg日志文件错误


##### 备份
一定时间间隔创建重要文件的备份

下面是一些比较重要的需要备份的一些文件类型
###### 配置文件
在修改你的配置文件之前，先进行备份，这样可以在除了问题的时候随时回复最初状态

###### 安装包组的列表
维持一个全部安装包组的列表，这样可以让安全重装不可避免的时候，更容易的重新回复到最初的环境
`pacman -Qqe > pkglist.txt`pacman的列出安装包的列表的技巧
`pacman -S --needed - < pkglist.txt`安装list文件中的所有package

###### pacman数据库
`tar -cjf pacman_database.tar.bz2 /var/lib/pacman/local`备份local的pacman数据库

###### 加密元数据


###### [系统备份](https://wiki.archlinux.org/index.php/System_backup)

定期备份系统和用户数据是很重要的，例如在`/etc, /home,/root,/var`对于服务器，还有`/srv`
- 使用[Btrfs snapshots](https://wiki.archlinux.org/index.php/Btrfs#Snapshots)
- 使用[LVM SNAPSHOTS](https://wiki.archlinux.org/index.php/LVM#Snapshots)
- 使用[rsync](https://wiki.archlinux.org/index.php/Rsync#As_a_backup_utility)
- 使用[tar](https://wiki.archlinux.org/index.php/Full_system_backup_with_tar) [[tar系统备份]]

- 使用[SquashFS](https://wiki.archlinux.org/index.php/Full_system_backup_with_SquashFS)


##### 升级系统
建议使用定期的完全系统升级通过`pacman -Syu`避免使用`pacman -Sy`部分升级

##### 使用包管理器安装软件
pacman在追踪文件上有优势，有时候手动安装可能会忘记之前的操作，因此使用包管理器可以更好的管理安装的包

###### 选择开源驱动而不是proprietary driver
大部分时候，开源驱动更加稳定可靠，开源驱动的bug修复更快更容易，但是闭源驱动更全面功能更多
关于更多的驱动信息在这个网站：[linux-driver](http://www.linux-drivers.org/)



###### 小心非官方源的软件包
从aur和非官方源的用户目录安装包时要小心

###### 定期更新镜像源
由于镜像源的质量随着时间在不断变化，有些可能下线或者速度变慢



##### 清理文件系统

###### [Disk usage display](https://wiki.archlinux.org/index.php/List_of_applications#Disk_usage_display)


###### [disk cleaning](https://wiki.archlinux.org/index.php/List_of_applications#Disk_cleaning)

###### pacman cache cleaning
`/var/cache/pacman/pkg/`在这里面清理不需要的软件包
下载：`pacman-contrib`包，使用`paccache -r`清理所有，保留最近的3个

详细的paccache信息：[cache cleanning](https://wiki.archlinux.org/index.php/Pacman#Cleaning_the_package_cache)


###### 清理不使用的包
使用下面一条命令清理orphans和他们的配置文件
`pacman -Qtdq | pacman -Rns -`

###### 旧的配置文件
寻找以下几个文件：
- `～/.config`
- `~/.cache`
- `~/.local/share`

###### 破坏的symlinks
可以通过下面两个链接找到方法：
- [](https://unix.stackexchange.com/questions/34248/how-can-i-find-broken-symlinks)
- [](http://www.commandlinefu.com/commands/view/8260/find-broken-symlinks)

尽管可以删掉破损的链接，但是盲目删除可能会出问题，因此应该注意一点，具体的内容看[this links](https://unix.stackexchange.com/questions/151763/is-there-a-downside-to-deleting-all-of-the-broken-symbolic-links-in-a-system)


使用下面的命令查找破损的链接
`find / -xtype l -print`



#### 系统维护的建议和技巧

1. 使用官方信任的软件包
2. 安装linux-lts  package长期维护支持版本的