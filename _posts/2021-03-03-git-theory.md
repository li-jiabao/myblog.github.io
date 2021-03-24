---
title: git原理
author: lijiabao
date: 2021-02-26 00:00:51.864856200 +0800
category: git使用
categories: git使用
tags: git使用
---

#### 几个区域分类

- 工作区，执行文件添加修改的区域，就是看得到的克隆下来的仓库目录文件，工作区添加到暂存区的命令，`git add filename/directory`反过来的一个命令：`git checkout`
- 暂存区：代码提交的暂时存放区域，暂存区到本地仓库的命令，`git commit -m "message"`，本地仓库到暂存区的命令`git reset`
- 本地仓库：存放需要和远程仓库进行交互的文件的地方，就是文件中的`.git`文件夹，本地仓库中有stage暂存区，还有git自动创建的第一个分支master，以及只想master的HEAD指针。
本地仓库到远程仓库的命令`git push origin master`，远程仓库到本地仓库的命令`git clone/fetch remoterep`
- 远程仓库：存放文件与别人共享的地方，也就是git服务器上对应的仓库，远程仓库到工作区`git pull`

#### branch分支介绍

前面提到的master就是GitHub的主分支，也是git为我们创的第一个分支，其他分支开发完成后都会合并到这个主分支，合并分支到当前分支的命令`git merge  [branch_name]
`

#### tag标签介绍

标签是用来标记特定的点或提交的历史，通常会用来标记特定的点或者提交的历史，一般用来标记发布版本的名称或者版本号，标签是固定的不能随意改动

#### HEAD指针

指向的是当前分支的最新提交

#### 工作流程

1. 克隆git资源作为本地工作目录
2. 在克隆的资源上添加或者修改文件
3. 如果其他人修改了，你可以更新资源
4. 在提交前查看修改
5. 提交修改

![](/images/localPic/Pasted image 20210223060940.png))
![](/images/localPic/Pasted image 20210223062217.png)
![](/images/localPic/Pasted image 20210223062505.png)

#### 本地仓库新建

需要执行本地新建的操作的花，需要首先使用ssh key，这个需要你先生成你的sshkey密钥。
1. `ssh-keygen -t rsa -C "注册邮箱"`其中-t选项可选的密钥形式有多种，可以在官网查看，执行命令后一路回车即可
2. 复制生成的密钥文件的公钥内容，然后粘贴到官网进行设置
3. 验证是否成功`ssh -T git@github.com`


```
mkdir test-repository
cd test-repository
git init
touch README.md
git add README.md
git commit -m "first commit"
git remote add rep_name https://gitee.com/Jia_bao_Li/repository_name.git
git push -u origin_name master 
```