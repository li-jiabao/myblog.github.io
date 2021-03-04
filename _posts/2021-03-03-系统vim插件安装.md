---
title: 系统vim插件安装
author: lijiabao
date: 2021-02-23 08:50:05.793143100 +0800
category: vim使用
categories: vim使用
tags: vim使用
---

#### 全局加载插件

`/usr/share/vim/vim82/autoload/`
将你需要全局加载的插件文件夹下的autoload文件夹下的vim文件放到上面目录中


#### 配置
当配置文件内容较多，可以将其分解成多个文件，然后在配置文件中source这些文件即可
`source /home/jiabao/.config/vim/*.vim`