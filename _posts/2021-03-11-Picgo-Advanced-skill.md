---
title: Picgo高级技巧
author: lijiabao
date: 2021-03-11 17:00
description: 这篇文章主要介绍如何Picgo的高级使用，如命令行上传等，可以借助命令行上传或者通过脚本对图片处理后再上传
categories:  Markdown
tags:  Markdown
---

介绍picgo的两个高级技巧,命令行上传和picgo-server服务器的使用

## 高级技巧

下面将介绍两个picgo中较高级的实用技巧，可以帮助你更加方便的处理上传图片

### 命令行上传

#### Windows

`安装路径\picgo.exe upload`

#### macOS

`/Application/picgo.app/Contents/Macos/Picgo upload`

#### Linux

`安装路径/Picgo.AppImage upload`

### Picgo-Server使用

Picgo在2.2版本内置了一个小型的服务器，用以接受来自其他应用的http请求来上传图片。默认监听地址：`127.0.0.1`端口：`36677`

#### HTTP调用上传剪切板图片

- method：`POST`
- url：`http://127.0.0.1:36677/upload`(这是默认配置，自己可以修改地址和端口号)

使用了上述操作之后，便可将剪切板图片上传，返回数据：

```json
{
  "success": true, // or false
  "result": ["url"]
}
```

#### HTTP调用上传具体路径图片

- method：`POST`
- url：`http://127.0.0.1:36677/upload`
- 需要有请求头，必须是json格式：`{list: ['xxx.jpg', 'xxx.png']}，类似的文件路径列表

进行了post请求之后，便可以将list中的文件路径的图片上床。返回数据：

```json
{
  "success": true, // or false
  "result": ["url"]  // 返回的是图片列表对应的url列表
}
```

