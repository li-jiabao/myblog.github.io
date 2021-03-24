---
title: Github访问失败
author: lijiabao
date: 2021-02-25 17:39:22.352211400 +0800
category: git使用
categories: git使用
tags: git使用
---

#### 修改hosts

`ping github.com`
`ping github.global.ssl.fastly.net`

然后将这两个操作得到的IP地址复制到hosts文件中，与之对应映射

Windows的hosts文件在`C:\windows\system32\driver\etc\hosts`
Linux和Mac的文件在：`/etc/hosts`
对该文件的修改需要系统管理员权限

```
# 格式： ip地址    域名/别名
ip地址     github.global.ssl.fastly.net
ip地址     github.com

```

Windows修改完，之后需要刷新dns缓存信息：使用命令`ipconfig/flushdns`
Linux刷新：`systemctl restart nscd`