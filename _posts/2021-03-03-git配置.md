---
title: git配置
author: lijiabao
date: 2021-02-23 07:20:12.326013300 +0800
category: git使用
categories: git使用
tags: git使用
---

#### 配置命令

`git config --system/global/local user.name "name"`类似的添加配置方法
`git config -l/--list`列出所有配置，--system/global/local，分别列出系统全局和本地配置
配置文件：`/etc/gitconfig`，`~/.gitconfig`


#### 一些常用配置

`git config --global user.name "li-jiabao"`
`git config --global user.email "905001136@qq.com"`

`git config \--global core.editor "vim"`修改编辑器
修改了配置器之后就可以
在git 提交时，就可以不用`git commit -m `   而是`git commit`