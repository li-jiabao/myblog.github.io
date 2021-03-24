---
title: jekyll配置管理博客
author: lijiabao
date: 2021-03-01 01:07:50 +0800
category: Blog
categories: Blog
tags: Blog
---

由于jekyll是ruby语言开发的，因此需要先配置ruby环境

#### 安装ruby并配置

`sudo pacman -S ruby`

`gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/`配置ruby中国镜像源
`gem sources --add https://mirrors.tuna.tsinghua.edu.cn/rubygems/ --remove https://rubygems.org/`配置清华镜像源

##### 安装jekyll和buddle

`gem install jekyll bundle`

配置环境变量，安装的需要配置环境变量的文件在`~/.local/share/gem/ruby/gem/2.7.0/bin`

配置buddle镜像：

`bundle config mirror.https://rubygems.org https://gems.ruby-china.com`
这一步之后不用修改Gemfile中的source了

生成博客文件`jekyll new blog_name`
进入博客文件：`cd blog_name`
开启server检查配置：`bundle exec jekyll server`

##### SSL 证书错误

正常情况下，你是不会遇到 SSL 证书错误的，除非你的 Ruby 安装方式不正确。

如果遇到 SSL 证书问题，你又无法解决，请修改 `~/.gemrc` 文件，增加 `ssl_verify_mode: 0` 配置，以便于 RubyGems 可以忽略 SSL 证书错误。

\---
:sources:
- https://gems.ruby-china.com
:ssl\_verify\_mode: 0

如果你在意 Gem 下载的安全问题，请正确安装 Ruby、OpenSSL，建议部署 Linux 服务器的时候采用 [这个 RVM 安装脚本](https://github.com/huacnlee/init.d/blob/master/install_rvm) 的方式安装 Ruby。

##### 其他说明

-   `Bundler::GemspecError: Could not read gem at /home/xxx/.rvm/gems/ruby-2.1.8/cache/rugged-0.23.3.gem. It may be corrupted.`，这类错误是网络原因下载到了坏掉的文件到本地，请直接删除那个文件。
-   请珍惜社区资源，勿基于本镜像做二次镜像网站，我们会定期检查 CDN 请求量统计，单日请求量过大（流量超过 20 G） 的 IP 将会被永久屏蔽。