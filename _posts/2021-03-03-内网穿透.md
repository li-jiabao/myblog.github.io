---
title: 内网穿透
author: lijiabao
date: 2021-02-25 00:49:41.335665700 +0800
category: Web相关
categories: Web相关
tags: Web相关
---
ngrok是一个可以用来免费内网IP转换为可供外网访问的一种方法，这个可以利用CNAME的域名映射到作者的服务器上，从未再创建一个隧道提供一个方便的途径给我们用与创建本地简易不稳定的服务器

该软件服务器的作者创作不易，有条件的小伙伴，可以购买便宜一点的10元服务器，支持一下作者，也方便我们自己

除了ngrok，还有需收费的花生壳，这个是内网穿透做的比较好的

#### 内网穿透的原理

内网穿透是我们在进行网络连接时的一种术语，也叫做NAT穿透，即在计算机是局域网内的时候，外网与内网的计算机的节点进行连接时所需要的连接通信，有时候就会出现内网穿透不支的情况。内网穿透的功能就是，当我们在[端口映射](https://hsk.oray.com/)时设置时，内网穿透起到了地址转换的功能，也就是把公网的地址进行翻译，转成为一种私有的地址，然后再采用路由的方式ADSL的宽带路由器，具有一个动态或者是固定的公网IP，最后ADSL直接在交换机上，这样所有的电脑都可以共享上网。内网穿透除了可以实现内网之间机器的网络通信功通之外，还可以解决UDP中出现的数据传输不稳定问题。

#### 内网IP解析成外网可访问的域名

首先进入[ngrok.cc官网](https://www.ngrok.cc)进行注册，显示会员注册，其实只要你注册了就是会员，里面会有一个免费的服务器使用，当然，如果有条件的话，可以购买一个服务器，便宜的也只要10块钱，真的是十分的良心的了。

注册好了之后请一定先看教程，作者也在里面是强烈建议看完教程之后再去进行操作

[教程网址](https://ngrok.cc/_book/other/https.html)

看完教程学会了之后就可以进行[客户端下载](https://www.ngrok.cc/download.html)


**大致的流程**

	1. 注册
	2. 观看教程
	3. 开通隧道
	进入到隧道设置界面，选择http或者https，然后设置前置域名（如bao之类的，不需要.号）之后端口就设置你自己的内网ip和端口
	4. 自定义域名需在隧洞建立好后，去到编辑页面进行设置，这个自定义域名需是你自己注册好的域名，如果想要使用https，你还需要为你的域名配置证书，详情看下方的ssl证书配置
	5. 如果你的网站需要授权访问的话，就可以设置验证用户名和密码，保存修改
	6. 然后进入到你的域名管理网站，进行域名记录的配置，添加一条记录CNAME的配置，将其记录指向服务器：`free.idcfengye.com`，保存记录修改即可。
	7. 进入到隧道列表，找到隧洞id，然后复制下来，之后在客户端的配置使用这个id来启动隧洞。详细的启动流程依照教程的来使用就可以了

详细的免费证书配置流程看：->[[ssl证书]]


#### 教程内容

##### ngrock启动

###### mac、linux启动

下载对应的[客户端](https://www.ngrok.cc/download.html)，修改该脚本的的权限，给该脚本加上可执行权限

**使用方法**
`./script_name clientid 隧道id`
多个隧道id启动`./script_name clientid 隧道id1,隧道id2`

后台运行可以使用：`setsid ./script_name clientid 隧道id &`


###### windows启动

和linux类似，也是在命令行中运行
`sunny.exe clientid 隧道id`

还有一个bat脚本文件，只要运行之后把隧道id输入即可

###### python版本

下载好了python脚本之后，直接执行脚本`python script --client=隧道id,隧道id`

php版本和python脚本类似


###### 安卓端也有

###### ngrokc客户端

可以在路由器中使用，[教程](https://ngrok.cc/_book/start/ngrok_ngrokc.html)

##### Frp启动

和ngrokc是一样的，只是少了python php脚本和安卓端


##### ngrokc常见错误

###### 错误1: Tunnel _\*\*_ not found

隧道没有启动的时候会提示：Tunnel sphynx.free.idcfengye.com not found

这时候应该检查隧道是否已经启动，如果没有启动则启动。

###### 错误2: 隧道 _\*\*_ 不可用

如果隧道启动了，而web服务没有启动会提示

![](https://ngrok.cc/_book/images/error/ngrok/1.png)

> 这个并不是错误，而是要映射的服务不没有启动，不是服务器出问题了，也不是隧道出问题了。

###### 错误3: bind: address already in use

服务器无法分配隧道: \[ctl:d66a0e1\] \[54710df44b9784aa60692e8b8e7bda79\] 绑定TCP监听器错误: listen tcp _\*\*_: bind: address already in use

![](https://ngrok.cc/_book/images/error/ngrok/2.png)

这个错误是因为TCP隧道已经启动了，所以会出现这个错误。

###### 错误4：服务器无法分配隧道: _\*\*_ 已注册.

服务器无法分配隧道: 隧道 [http://sphynx.free.idcfengye.com](http://sphynx.free.idcfengye.com/) 已注册.

![](https://ngrok.cc/_book/images/error/ngrok/3.png)

这个错误是因为HTTP隧道已经启动了，所以会出现这个错误。

-   ##### 解决办法
    
    1、登陆管理平台到隧道管理
    
    2、点击查看状态，检查隧道是否在线
    
    3、如果在线点击踢下线
    

###### 错误5: reconnecting

出现这个错误要检查两个问题

-   这个问题一般都是香港的问题比较多，因为香港由于国策原因有可能被墙也会抽风。因此如果有在tcp和udp的用户尽可能的选择内地的服务器。出现这个问题的时候先选择ping一下服务器看看能否通
-   检查隧道是否已经启动，http的隧道可以通过浏览器访问如果没有启动的隧道提示是 not found 如果已经启动那么浏览器会一直转圈。对于tcp隧道可以采用telnet的方式连接查看隧道是否启动，按照错误4的解决办法操作

解决问题步骤：

-   1、ping 隧道服务器（隧道管理单机隧道ID可以展开查看服务器地址），检查是否可以ping通
    
-   2、通过 [http://ping.chinaz.com/服务器地址](http://ping.chinaz.com/%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%9C%B0%E5%9D%80) ping 隧道服务器地址（隧道管理单机隧道ID可以展开查看服务器地址） 查看得出来的ip是否和自己电脑ping的ip一致，检查dns是否污染
    
-   3、检查隧道是否启动，可以在隧道管理平台隧道管理检查
    
-   4、telnet 隧道服务器的 4443端口

##### FRP常见错误

###### 错误1: start error: listen tcp _\*\*_: bind: address already in use

这个错误是因为TCP隧道已经启动了，所以会出现这个错误


#### 部署https协议隧道

##### 方法一：本地部署HTTPS环境

如果在本地部署环境，我们以Nginx为例。

步骤：

-   1、先去下载证书
-   2、部署证书到nginx服务器上
-   3、修改平台隧道协议
-   4、启动隧道

###### 1、先去下载证书

登陆申请证书的页面，这里以阿里云为例

![](https://ngrok.cc/_book/images/other/https/https9.png)

![](https://ngrok.cc/_book/images/other/https/https10.png)

我使用的上nginx服务器，所以我下载nginx的

###### 2、部署证书到nginx服务器上

```
server {
    listen       443 ssl;
    server_name  test.sunnyos.com;

    ssl_certificate      /usr/local/nginx/conf/ssl/test.sunnyos.com.pem;
    ssl_certificate_key  /usr/local/nginx/conf/ssl/test.sunnyos.com.key;

    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;
    ssl_prefer_server_ciphers on;

    location / {
        root   html;
        index  index.html index.htm;
    }
}
```

证书配置文件参考阿里云的配置，每家服务商的不一定一样，具体参考ssl证书服务商的教程。

###### 3、修改平台隧道协议

![](https://ngrok.cc/_book/images/other/https/https11.png)

因为我的nginx在虚拟机里面，隧道在物理机里面启动的，所以修改内网地址和端口为虚拟机的实际地址和端口。

###### 4、启动隧道

![](https://ngrok.cc/_book/images/other/https/https12.png)

启动隧道，访问隧道。

![](https://ngrok.cc/_book/images/other/https/https13.png)

浏览器访问隧道可以看到证书了。

##### 方法二：CDN服务商和HTTPS证书在同一个服务商部署

如果使用阿里云的CDN并且在阿里云申请的https证书，直接在证书页面部署 步骤：

-   1、进入https证书页面找到对应的证书->点击部署->点击CDN
-   2、在弹出来的页面选择在CDN里面要部署的域名
-   3、设置CDN回源协议
-   4、查看是否部署成功
-   5、启动隧道

###### 1、进入https证书页面找到对应的证书->点击部署->点击CDN

![](https://ngrok.cc/_book/images/other/https/https1.png)

###### 2、在弹出来的页面选择在CDN里面要部署的域名

![](https://ngrok.cc/_book/images/other/https/https2.png)

###### 3、设置CDN回源协议

![](https://ngrok.cc/_book/images/other/https/https4.png)

因为在CDN部署的https所以在 Sunny-Ngrok 平台不需要使用 https 协议，因此在cdn这里的回源协议要使用http。

###### 4、查看是否部署成功

![](https://ngrok.cc/_book/images/other/https/https3.png)

到这里看到我们刚刚部署的证书就是成功了。

###### 5、启动隧道

![](https://ngrok.cc/_book/images/other/https/https5.png)

启动隧道注意这里协议还是http，所以一定要在cdn回源设置http协议回源。

##### 方法三：CDN和HTTPS证书分别在不同服务商部署

例如我们使用阿里云的CDN但是证书是在别的地方申请的，例如我在腾讯申请的证书。

步骤：

-   1、进入腾讯云下载证书
-   2、进入阿里云的CDN配置页面上传证书
-   3、启动隧道

###### 1、进入腾讯云下载证书

![](https://ngrok.cc/_book/images/other/https/https6.png)

###### 2、进入阿里云的CDN配置页面上传证书

![](https://ngrok.cc/_book/images/other/https/https7.png)

![](https://ngrok.cc/_book/images/other/https/https8.png)

把下载的证书上传到阿里云到CDN里面

###### 3、启动隧道

![](https://ngrok.cc/_book/images/other/https/https5.png)