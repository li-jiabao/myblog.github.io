---
title: 服务器搭建
author: lijiabao
date: 2021-02-23 07:11:26.053566900 +0800
category: git使用
categories: git使用
tags: git使用
---

#### 安装git

`sudo pacman -S base-devel openssh git`

#### 创建git用户组和用户
用来运行git服务器
`groupadd git`

`useradd -m git -G git`


#### 创建证书登陆

收集所有需要登陆的用户的公钥，公钥位于`id_rsa.pub`文件夹中
把公钥导入到`/home/git/.ssh/authorized_keys`文件夹中

```
 cd /home/git
 mkdir .ssh
 chmod 755 .ssh
 touch .ssh/sutorized_keys
 chmod 644 .ssh/authorized_keys
```


#### 初始化git仓库
选定一个目录为git仓库，执行下述步骤即可
```
mkdir /home/rep
chown git:git rep/
cd rep

git init --bare my-gitrep.git
chown -R git:git my--gitrep.git把仓库所属用户改为git
```


#### 克隆仓库

server-ip是指服务的ip
`git clone git@server-ip:/home/rep/my-gitrep.git`