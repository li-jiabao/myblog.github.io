---
title: YouCompleteMe插件
author: lijiabao
date: 2021-02-27 21:05:55.821555200 +0800
category: vim使用
categories: vim使用
tags: vim使用
---

#### 安装前准备

`sudo pacman -S base-devel git java python npm nodejs go cmake`
安装一些安装所需的包

#### 下载

找到官网的仓库地址，克隆到本地的vim插件文件夹下面，`~/.vim/pack/vendor/start/`

#### 子模块更新
`git submodule update --init --recursive`
同步放在该仓库下的其他子模块仓库，并更新

#### completer安装

根据自己的需要，安装需要的补全插件模块
Cfamily：`./installer.py --cland-completer`
java：`./installer.py --java-completer`
类似，还可以直接安装全部的补全插件`./installer.py --all`

#### 速度慢的问题

go：`export GOPROXY=https://goproxy.io`设置go的代理设置
npm：`npm config set registry https://registry.npm.taobao.org`设置npm镜像
java的补全jdt下载慢：自己手动下载下来对应的版本放在对应的目录下面即可
cland下载慢：也可以自己手动下载对应版本

#### 安装完成的配置

配置global ycm_extra_conf：`let g:ycm_global_ycm_extra_conf='file_path'`
开启vim文件甄别：`:filetype indent plugin on`
一般这一步完成之后，应该可以补全了

之后便可以按照自己的需求去进行相关的配置了
