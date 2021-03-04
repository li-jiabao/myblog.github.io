---
title: vim配置
author: lijiabao
date: 2021-02-20 23:19:54.238570600 +0800
category: vim使用
categories: vim使用
tags: vim使用
---
#### 安装插件
vim在8之后已经开始支持自带的插件管理系统了，只需要在`～/.vim/pack/vendor/start`下防止自己安装的插件目录即可成功安装，然后就可以在`~/.vimrc`中配置vim以及他的插件
还有`～/.vim.pack/vendor/opt`放置安装不需要打开vim就启动的插件的


#### youcompleteme插件安装
首先复制github的项目官网地址，然后克隆到本地

下载编译安装所需要的软件包：
`sudo pacman -S cmake git python(3) base-devel vim(8) npm java nodejs go`

cd到youcompleteme目录下，执行`git submodule update --init --recursive `
有时候网速不好，可能需要终止之后继续执行，等待操作完成之后就好

然后执行补全插件的安装：
`./install.py --all`安装全部的
`--cland-completer`c家族的
`--java-completer`还有许多，可以看官网自己挑选

#### 配置
在vimrc添加一句：
`let g:ycm_global_ycm_extra_conf='~/.vim/pack/vendor/start/YouCompleteMe/third_party/ycmd/cpp/ycm/.ycm_extra_conf.py'`这样才能保持正确的配置补全

对于java的补全，必须是创建好的项目文才可以补全


#### 自动回到上一次编辑的光标位置
配置文件中添加
```vimrc
augroup resCur
	autocmd!
	autocmd BufReadPost * call setpos(".", getpos("'\"")) 
augroup END
```
