---
title: systemd
author: lijiabao
date: 2021-02-15 21:56:56.038957000 +0800
category: Archlinux配置
categories: Archlinux配置
tags: Archlinux配置
---
## [SYSTEMD](https://wiki.archlinux.org/index.php/Systemd)Archlinux使用这个

==是archlinux作为init进程的service管理程序==，是linux的系统和服务的管理软件套装

作为PID1进程，并且用来启动系统的后续程序和配置文件等。

==一些特点==
1. systemd拥有良好的同步能力
2. 使用socket和D-bus activation作为启动服务
3. 提供的合适的守护进程daemons的启动
4. 使用linux的control group追踪进程
5. 保持挂载和自动挂载点
6. systemd支持SysV和LSB的init脚本并且作为sysinit的替代者
7. 包含有日志守护进程以及控制基本系统配置（hostname，date，locale）的utilities
8. 维持一系列的登陆用户并且运行容器和虚拟机，系统账户，正在使用的目录和设置，管理进但网络配置的守护进程，网络时间同步，日志记录以及域名解析（name resolution）


#### 基本使用
主要的用来introspect和控制系统的命令是[systemctl](https://man.archlinux.org/man/systemctl.1)
==用处==：检查系统状态，管理系统和服务，使用`systemctl -H USER@HOST`可以管理远程机器（使用的是SSH连接）Plasma用户下载`systemd-kcm`是图形界面管理程序

##### 分析系统状态
`systemd status`查看系统的状态
`systemctl`or`systemd list-units`列出正在运行的单元
`systemctl --failed`列出失败单元
可用的的单元文件（可以是service，.mount，.device或者.socket）可以在`/usr/lib/systemd/system/`and`/etc/systemd/system/`中看到
`systemdctl list-unit-files`列出安装的单元文件（unit files）
`systemctl status PID`展示指定PID的cgroups slice 内存和父母进程

##### 使用units
[systemd.unit详细信息](https://man.archlinux.org/man/systemd.unit.5)
使用systemctl的时候，一般来说必须指定unit file的全名，包括后缀，有时候可以不指定后缀也可以
- 不指定后缀时，默认为service,netctl等同netctl.service
- 挂载点自动转换为.mount，/home=home.mount
- 和挂载点一样，设备device也是自动转换

`systemctl help unit`查看unit的手册文档
`systemctl status unit`查看unit状态
`systemctl is-enable unit`unit
是否可以激活
`systemctl start/stop/restart/reload`开启/停止/重启/重加载
`systemctl daemon-reload`重载system管理器的配置文件
`systemctl enable unit`启动的时候自动启动unit
`systemctl enable --now unit`设置开机启动并立马启动
`systemctl disable unit`不再自动启动
`systemctl reenable unit`自从上次开启之后安装部分发生改变之后使用这个命令


##### power managerment电源管理
`polkit`是非特权用户电源管理所必须的
如果没有本地系统登陆用户session或其他的session在使用，下面的命令直接生效而不用系统权限，不然需要系统密码
`systemctl reboot/poweroff/suspend/hibernate/hybrid-sleep`重启/关机/suspend/休眠/混合休眠
