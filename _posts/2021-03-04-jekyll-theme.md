---
title: 使用jekyll主题
author: lijiabao
date: 2021-03-01 22:22:26 +0800
category: Blog
categories: Blog
tags: Blog
---

### 主题安装

#### 网上下载配置

1. 在主题官网下载：[1.org](http://jekyllthemes.org/) [2.com](http://jekyllthemes.com/) [3.dev](https://jekyllthemes.dev/)

2. 解压到需要的目录下

3. 进入到该目录下，执行`bundle`安装所需的依赖和主题

4.执行：`bundle exec jekyll server` 可以在`http//localhost:4000`查看主题情况

#### gem下载主题

1. 在你的jekyll站点的`Gemfile`中添加：`gem "jekyll-theme-chirpy"`

2. 然后在你的配置文件： `_config.yml`:中加入：`theme: jekyll-theme-chirpy`

3. 然后在命令行下执行：`bundle`
 
4. 从gem下载下来的主题文件复制到你的站点目录下面
找到主题文件位置的命令：`bundle info --path jekyll-theme-chirpy`


一般主题文件的git repository中都有详细的安装教程，可以按照安装教程来执行


#### github pages配置jekyll主题

##### devepr主题

首先进入到主题的github主题repository当中，然后查看READMME.md文档，学习怎么配置

一般是首先fork目录到自己的github中，然后在该repository中点击使用这个为模板创建自己的repositroy。等待创建完成，然后便可以将这个目录克隆到本地中，然后在本地进行修改配置并且进行jekyll的本地配置查看，当然如果非常了解jekyll的使用配置的话，将会非常容易的定制化自己的博客界面。

克隆到本地之后，需要进行下列的操作：

1. `git clone https://github.com/your_github_username/your_github_username.github.io.git`
2. `cd your_github_username`
3. `ruby -v`
4. `gem install bundler`
5. `bundler -v`
6. `bundle add jekyll`
7. `bundle exec jekyll -v`
8. `bundle update`
9. `bundle install`
10. `bundle exec jekyll serve --watch`


更新：

1. `git remote -v`
2. `git remote add upstream https://github.com/sujaykundu777/devlopr-jekyll.git`
3. `git fetch upstream`
4. `git checkout master`
5. `git merge upstream/master`
6. `git push`


##### 很多主题类似