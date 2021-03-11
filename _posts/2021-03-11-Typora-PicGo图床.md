title: Typora-PicGo图床
author: lijiabao
date: 2021-03-11 12:00
description: 这篇文章主要介绍如何使用PicGo配置图床并应用到typora中
categories: Markdown
tags: Markdown

## PicGo

是一个快速上传图片并获取图片的url链接的一个实用工具，它可以将获取到的url生成许多你需要的url链接格式。如：

- markdown
- html
- URL
- UBB
- 还支持自定义自己想要的格式

### 安装

#### Windows

下载最新的exe安装文件，执行安装即可

#### macos

下载最新的dmg文件或者使用`brew cask install picgo`

#### Linux

下载Appimage文件。Arch可以使用`aurman -S picgo-appimage`安装

### 配置文件

- Windows：`%APPDATA%\picgo\data.json`
- Linux：`$XDG_CONFIG_HOME/picgo/data/json`或者`~/.config/picgo/data.json`
- macos：`~/Library/Application\ Support/picgo/data.json`

具体位置：

- Windows：`C:\Users\用户名\APPData\Roaming\picgo\data.json`
- Linux：`~/.config/picgo/data.jason`
- macos：也是在家目录下的`~/Library/Application\ Support/picgo/data.json`

### 图床

图床设置配置文件时，你只需要将图床的配置项作为配置文件的一项放入配置文件即可，也可以直接在软件界面中进行GUI的设置，更加简洁方便。我自己使用的是GitHub和smms图床，不为别的，只为可以白嫖:upside_down_face:.而且比较简单就可以搭建好。

#### SMMS图床

进入到[smms官网](https://sm.ms)注册smms图床账号，有免费5G的容量使用，普通非重度图片存储用户基本够用。

注册好了账号之后进入到个人信息中，生成自己的Token，然后复制下来，将其配置项配置到配置文件中

```json
"smms": {
    "token": "your-token"
}
```

#### GitHub图床

1. 首先你得申请注册好自己的一个GitHub账号。
2. 新建一个仓库记住你的这个仓库名。
3. 然后到GitHub设置中生成一个token用于PicGo操作你的仓库：[设置地址](https://github.com/settings/tokens)然后点击Generate new token。在跳出来的页面勾选repo的勾。然后翻到页面最底部，点击Generate token的绿色按钮生成token。
   注意：**这个token生成后只会显示一次！你要把这个token复制存到其他地方备用。
4. 配置PicGo
   **注意：**仓库名的格式是用户名/仓库，比如我创建了一个叫做test的仓库，在PicGo里我要设定的仓库名就是Molunerfinn/test。一般我们选择main分支即可。然后记得点击确定以生效，然后可以点击设为默认图床来确保上传的图床是GitHub。配置GitHub图床时自定义url的格式：`https://raw.githubusercontent.com/user-name/repository-name/branch-name`,对于github图片加载过慢或者加载不出来的情况，可以使用CDN加速，格式：`https://cdn.jsdelivr.net/gh/user-name/repository-name@branch-name`
   具体原理原谅我技术不行，不是很懂，但是确实速度快了不少

**GitHub图床配置文件示例：**

```json
"github": {
  "repo": "", // 仓库名，格式是username/reponame
  "token": "", // github token
  "path": "", // 自定义存储路径，比如img/
  "customUrl": "", // 自定义域名，注意要加http://或者https://格式
  "branch": "" // 分支名，默认是main
}
```

#### 七牛图床（qiuniu）

需要自己注册一个七牛的账号（七牛需要上传认证才可以，而且是收费的，如果对于图床需求不高，用smms和github基本足够）然后在七牛的[控制台](https://portal.qiniu.com/user/key)中找到自己的密钥信息，自己存储空间的区域需要确定，配置文件中存储区域对应的键是`area`，值根据自己的位置确定，比如华东是`z0`.当配置存储区域的时候，你需要设置七牛云的上传地址或者自定义的域名。当你使用了图片处理的处理参数（如图片瘦身）时，通常需要使用URL后缀

**七牛图床配置文件：**

```json
"qiniu": {
    "accessKey": "",
    "secretKey": "",
    "bucket": "", // 存储空间名
    "url": "", // 自定义域名
    "area": "z0" | "z1" | "z2" | "na0" | "as0", // 存储区域编号
    "options": "", // 网址后缀，比如?imgslim
    "path": "" // 自定义存储路径，比如img/
}
```

#### 腾讯云图床

需要登录[腾讯云](https://console.qcloud.com/cos4/secret)控制台。打开密钥管理按照对应的提示找到自己的APPID、SecretId、SecretKey。存储的空间名是你的bucket名字。存储的区域需要额外注意，请到bucket列表里打开需要上传的bucket空间，然后如图可以看到对应的区域以及区域代码，比如tj。如果你想把图片上传到你的bucket空间的某个文件夹下，则需要在PicGo里的指定存储路径里加上你的文件夹路径。比如temp/（注意一定要加/）

**腾讯云图床配置文件：**

```json
"tcyun": {
  "secretId": "",
  "secretKey": "",
  "bucket": "", // 存储桶名，v4和v5版本不一样
  "appId": "",
  "area": "", // 存储区域，例如ap-beijing-1
  "path": "", // 自定义存储路径，比如img/
  "customUrl": "", // 自定义域名，注意要加http://或者https://
  "version": "v5" | "v4" // COS版本，v4或者v5
}
```

#### 阿里云图床

首先先在阿里云OSS的[控制台](https://usercenter.console.aliyun.com/#/manage/ak)里找到你的`accessKeyId`和`accessKeySecret`。创建一个`bucket`后，存储空间名即为`bucket`。然后确认你的[存储区域](https://www.alibabacloud.com/help/zh/doc-detail/31837.htm?spm=a2c63.p38356.a3.3.179112f0PBtYui)的代码：也可以在bucket页面找到：存储区域类似`oss-cn-beijing`。存储路径比如`img/`的话，上传的图片会默认放在OSS的`img`文件夹下。注意存储路径一定要以`/`结尾！存储路径是可选的，如果不需要请留空。

**阿里云图床配置文件：**

```json
"aliyun": {
  "accessKeyId": "",
  "accessKeySecret": "",
  "bucket": "", // 存储空间名
  "area": "", // 存储区域代号
  "path": "", // 自定义存储路径
  "customUrl": "" // 自定义域名，注意要加http://或者https://
}
```

#### Imgur图床

登录Imgur后，在[此处](https://api.imgur.com/oauth2/addclient)生成你的ClientId，记得选第二项，不需要callbackurl的。于是你可以拿到你的clientId:
**注意**：imgur貌似对中国大陆的IP和请求做出了限制，所以如果clientId没错的情况下无法上传图片的时候，可以考虑配置代理设置。默认只支持HTTP代理。如果觉得设置麻烦的可以考虑使用SM.MS图床。

**Imgur配置文件：**

```json
"imgur": {
  "clientId": "", // imgur的clientId
  "proxy": "" // 代理地址，仅支持http代理
}
```

#### 又拍云

存储空间名即为你的服务名，加速域名即为你又拍云分配给你的域名或者是你自己绑定的域名。请注意，加速域名需要加`http://`或`https://`。操作员即为你自己为该存储空间设定的操作员名，密码即为对应的密码。网址后缀为你针对图片进行的一些处理参数。由于又拍云官方没有对云存储有一个直观的控制面板，所以推荐可以采用第三方web面板来查看和操作：[又拍云存储Web版操作工具](https://github.com/xcuts/UPYUN-API-Web-Tool)

**又拍云配置文件：**

```json
"upyun"{
  "bucket": "", // 存储空间名，及你的服务名
  "operator": "", // 操作员
  "password": "", // 密码
  "options": "", // 针对图片的一些后缀处理参数
  "path": "", // 自定义存储路径，比如img/
  "url": "" // 加速域名，注意要加http://或者https://
}
```

### Picgo软件设置

picgo的软件设置很方便，都是GUI图形化的设置，自己在设置中查看并按自己需求设置即可

#### 日志文件开启

从`v2.1.0`开始PicGo支持记录你上传的日志，如果有什么报错等信息，可以及时反馈给开发者。你可以在这个设置里面打开日志文件查看，也可以设置输出的日志类型（比如成功、失败或者不输出等）

#### 自定义快捷键

PicGo v1.4.0版本开始支持自定义快捷键（默认快捷键是`Cmd+Shift+P`【Mac】或者`Ctrl+Shift+P`【Windows】），点击侧边栏PicGo设置选中修改快捷键：

会打开快捷键面板（v2.2.0+），可以选择禁用或者启用快捷键：点击「编辑」，在打开的dialog里，点击input框，然后按下你想要的快捷键（也可以是组合键）。然后点击**确定保存**（否则不生效！）

PicGo从2.2.0+版本添加了快捷键系统，插件也可以添加自己的快捷键，并且添加了快捷键会显示在快捷键面板里。可以方便启用或者禁用！

#### 自定义链接格式

PicGo预置的有四种链接格式：`Markdown`\`HTML`\`URL`\`UBB`。如果你都不喜欢，想要自定义链接格式，可以选择`Custom`，然后在PicGo设置里点击`自定义链接格式`，然后你可以配置自己想要的复制的链接格式。

提示：v2.1.2开始支持`$fileName`设置文件名。

#### 开关更新助手

PicGo每次启动的时候会去检查最新版本。如果当前版本低于最新版本会提示你更新。如果你不想接到这条消息，那么可以在PicGo设置里把`打开更新助手`这个选项关闭。**推荐大家打开这个开关，新的版本通常会修复bug已经加入新的功能，让PicGo更好用~**

#### 上传前重命名

如果你想在图片上传前能够有机会改动你的图片名，那么可以选择开启图片上传前重命名。之后你在上传的时候就会弹出一个小窗口让你重命名文件。如果你不想重命名，点击确定、取消或者直接关闭这个窗口都是可以的。如果你想要重命名就在输入框里输入想要更改的名字，然后点击确定即可。另外这个特性也支持批量上传

#### 上传提示

 打开之后会在每次上传图片的时候弹出提示框提示正在上传。 **如果你发现打开之后，没有效果，请注意看看是不是你关闭了系统级别的消息通知选项，因为PicGo调用的是系统级别的消息通知栏。**

#### 代理设置

2.0版本之后，支持简单设定HTTP代理。在`设置代理`一项处点击即可。 **未来不会支持复杂的代理设置，因为跟底层有关，只能支持简单HTTP代理。**

#### PicGo-Server设置

2.2版本之后，PicGo内部会默认开启一个小型的服务器，用于配合其他应用来调用PicGo进行上传。监听的地址推荐就默认的 `127.0.0.1` （本机），端口推荐默认的 `36677`。当然如果你不想要开启也可以选择关闭，只不过推荐你可以开启~可以配合一些第三方工具实现很方便的上传工作流。

关于Server的调用可以参考[高级技巧](https://picgo.github.io/PicGo-Doc/zh/guide/advance.html#PicGo-Server的使用)的说明。

### 插件设置 

2.0版本之后，你可以简单通过`插件设置`页面，安装、更新、禁用、卸载、配置、使用插件。

注意：你必须安装[Node.js](https://nodejs.org/en/)之后才能安装PicGo的插件，因为PicGo要使用`npm`来安装插件。在插件界面的搜索栏搜索插件名。PicGo的插件名以`picgo-plugin-`为前缀，你只需要搜前缀后的名字即可。比如一个`picgo-plugin-wow`的插件你只需要搜索`wow`即可。搜到了插件之后只要点击右下角的`安装`即可。如果遇到`未对GUI优化`的提示，可以询问一下插件作者，这个插件适不适合在PicGo软件上使用，否则它有可能只是个命令行插件。更新、卸载与禁用皆可点击插件右下角的齿轮按钮，在弹出的菜单中选择。

注意其中如果你选择了`更新`之后，PicGo需要重启一遍才能使用更新后的插件，PicGo会在插件页面给出`重启`按钮，点击即可。 **只是关闭主窗口再打开是不行的，必须完全退出PicGo进程再打开PicGo。**

#### 配置使用

有的插件拥有配置项，可以直接点击右下角齿轮，点击`配置xxx`进行配置。有的插件拥有自有菜单项，可以直接点击右下角齿轮后，找到插件自有菜单区，点击使用：

#### 寻找插件

你可以在PicGo官方的[Awesome-PicGo](https://github.com/PicGo/Awesome-PicGo)里找到超棒的PicGo插件和应用了PicGo的应用或者项目~