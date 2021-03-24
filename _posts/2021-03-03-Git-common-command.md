---
title: Git常用命令
author: lijiabao
date: 2021-02-26 00:53:01.238869900 +0800
category: git使用
categories: git使用
tags: git使用
---
如果不了解具体命令的使用，可以使用`--help`选项查看使用帮助

#### 实用命令

配置命令：`git config`
```
git config --list   # 列出所有的配置

git config --global/system/local --list    # 列出所有的全局/系统/本地配置

git config --global user.name "name"  
# 添加用户名配置,添加配置就是类似的操作,只是对应的值和键名不同
```


`.gitignore配置文件`设置哪些文件不需要推送到服务器
```
# 配置文件示例
# 每一行防止需要忽略传送到服务器的文件,支持通配符

CNAME

*.java
```

##### 和仓库对接的命令

`git add file/当前路径`添加文件到缓存区，选项有`-i（交互式命令子系统） -u（交互式命令系统的update模式） -p（交互式命令子系统的patch模式）`

文件的增删改查：在Linux命令的增删改查前面加上git，就可免去`git add`操作

`git commit -m "提交原因"`提交添加的文件到版本库中
`git commint --amend -m "提交原因"`提交最新一条记录的更新原因
`git commit -C HEAD`将当前文件改动提交到head或者当前分支的历史id

`git push rep_name [branch_name]` 推送到远程仓库

`git clone/fetch remote-rep-addr` 获取git服务器的远程仓库到本地

`git pull`将远程仓库拉回到本地工作区

`git checkout`恢复正在工作的工作区tree file

`git status`查看文件变动状态选项`-s（简短记录查看） --ignored（包括被忽略的文件）`

`git log`查看本地的提交记录

`git tag`为项目标记里程碑


##### 分支命令

`git branch -a`查看本地和远程的分支列表
`git branch -r`查看远程的分支列表，加-d参数可以删除远程版本库的分支

`git branch [branch_name]`新建分支
`git branch -m origin_name new_name`用于更改分支名字
`git branch`查看当前分支
`git branch -d [branch_name]`删除分支
`git branch -D`分支未提交到本地版本库前强行删除分支
`git branch -vv`查看详细信息


**合并分支**
这个命令是用来合并分支到当前分支
默认的分支合并命令执行快进式合并，直接将master分支指向devel分支

`git merge --no-ff`
使用了这个参数之后，在master分支上会生成一个新节点，保证版本演进更加清晰

`git merge --no-edit`
在没有冲突的情况下合并，不想手动编辑提交原因，自动生成提交原因

**切换分支**

`git checkout [branch_name`用来切换到分支

`git checkout -b [branch_name]`创建分支并切换到这个分支

`git checkout HEAD demo.html`
从本地版本库中的HEAD历史找出文件并覆盖当前工作的文件，没有HEAD就从暂存区寻找

`git checkout --orphan new_branch`
创建出一个全新的完全没有历史记录的分支，但当前源分支上的最新文件都在，但是这个分支需要commit之后才正式成为分支

`git checkout -p other_branch`
用来查看两个分支之间的差异


#### 栈命令

在git的栈中保存当前修改或者删除的工作进度，当你在一个分支里面做某项功能开发时，接到通知把昨天测试完没问题的代码提交发布到线上，但这时候，你已经在这个分支里面加入了其他未提交 的代码，这个时候，就可以把这些未提交的代码存到栈里面

`git stash`将未提交的文件提交到栈中
`git stash list`查看栈中保存的内容
`git stash show stash@{0}`显示栈中的一条记录
`git stash drop stash@{0}`移除一条记录
`git stash pop`检出最新的一条记录，并移除
`git stash apply stash@{0}`检出一条记录不删除
`git stash branch new_branch `当前栈中最新一条记录检出，并创建一个新分支
`git stash clear`清空
`git stash create`创建一个自定义的栈，并且返回一个ID，此时并未真正存储到栈中
`git stash store ID`真正的创建一条记录

#### 操作历史

显示历史提交记录

`git log -p`显示提交历史差异对比的记录
`git log file`查看文件的历史记录
`git log --since="2 weeks ago"`2周前到现在的历史记录
`git log --before="2 weeks ago"`截止到2周前的历史记录
`git log -10`查看10条记录
`git log --pretty=oneline`一行中输出简短历史记录
`git log --pretty=format:"%h"`指出自定义格式


#### 重置分支

将当前分支重设到指定的commit或者HEAD


#### 撤销

撤销某次操作，此次操作前后的记录都会保留，并且把这次撤销作为一次最新的提交

`git revert -n HEAD`撤销前一次的提交操作，n可以指定多次记录

#### diff命令
查看工作区，暂存区，本地库和远程库之间的文件差异

`git diff`
--stat参数可以用来查看变更统计数据

#### 查看所有记录

`git reflog`查看所有分支的所有操作记录


#### 远程版本库连接

如果在创建远程版本库之前，文件已经存在本地文件库中，可以使用`git init`本地初始化本地版本库，之后再将其与远程版本库连接
具体流程：

```
# 已有本地文件的初始化和远程库连接
git init
touch README.md
git add README.md
git commit -m "first commit"
git remote add rep_name https://gitee.com/Jia_bao_Li/repository_name.git
git push -u origin_name master 
```

`git init`在本地目录内部生成.git文件夹
`git remote -v`列出远程分支
`git remote add origin repo-addr`添加一个新的远程仓库，指定名字为origin，以便引用后面的url远程仓库
`git fetch origin branch_name`指定分支名时只取出这个分支的更新到本地，不指定分支，默认拉取所有分支更新到本地版本库

#### 问题排查

`git blame`查看文件每行代码块的历史信息

`git bisect`二分查找历史记录，排查bug
后面加的参数：start开始查找，good没问题的点，bad有问题的点，reset回到原分支


#### 其他操作

`git submodule`
git子模块跟踪外部版本库，允许在一个版本库中存储例外的版本库，并且可以保持两个版本库完全独立，参考Youcompleteme的安装时，他的版本库

`git gc` 运行git的垃圾回收机制，清理冗余的历史快照

`git archive`将加了某个tag的某个版本进行打包
`git archive -v --fromat=zip v0.1 > v0.1.zip`