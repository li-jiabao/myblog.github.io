---
title: trojan代理配置
author: lijiabao
date: 2021-02-27 20:45:22.953720900 +0800
category: Web相关
categories: Web相关
tags: Web相关
---

#### 准备工作

##### 安装配置nginx：
`sudo pacman -S nginx-mainline`
nginx的配置文件在`/etc/nginx/nginx.conf`
配置如下：
```json
user  root;
worker_processes  1;
error_log  /var/log/nginx/error.log warn;
# pid        /var/run/nginx.pid;  # 多余pidduplicate的情况，注释掉这一行
events {
    worker_connections  1024;
}
http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    log_format  main  '/$remote_addr - /$remote_user [/$time_local] "/$request" '
                      '/$status /$body_bytes_sent "/$http_referer" '
                      '"/$http_user_agent" "/$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile        on;
    #tcp_nopush     on;
    keepalive_timeout  120;
    client_max_body_size 20m;
    #gzip  on;
    server {
        listen       80;
        server_name  $your_domain;
        root /usr/share/nginx/html;
        index index.php index.html index.htm;
    }
}
```

##### 设置伪装站点

rm -rf /usr/share/nginx/html/*   #删除目录原有文件
cd /usr/share/nginx/html/    #进入站点更目录
wget https://github.com/V2RaySSR/Trojan/raw/master/web.zip
unzip web.zip
systemctl restart nginx.service


#### 配置trojan
配置文件在：`/etc/nginx/xxx.json`的形式
配置文件示例在`/usr/share/trojan/examples/***`也是json文件，复制到配置文件目录下，再按自己情况进行相应的修改就好，下面是服务器的配置文件示例：
```json
{
    "run_type": "server",
    "local_addr": "0.0.0.0",
    "local_port": 443,
    "remote_addr": "127.0.0.1",    # 伪装用的话，需要指向webserverip和port
    "remote_port": 80,
    "password": [
        "00000000"
    ],
    "log_level": 1,
    "ssl": {
        "cert": "/usr/src/trojan-cert/fullchain.cer",
        "key": "/usr/src/trojan-cert/private.key",
        "key_password": "",
        "cipher_tls13":"TLS_AES_128_GCM_SHA256:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_256_GCM_SHA384",
	"prefer_server_cipher": true,
        "alpn": [
            "http/1.1"
        ],
        "reuse_session": true,
        "session_ticket": false,
        "session_timeout": 600,
        "plain_http_response": "",
        "curves": "",
        "dhparam": ""
    },
    "tcp": {
        "no_delay": true,
        "keep_alive": true,
        "fast_open": false,
        "fast_open_qlen": 20
    },
    "mysql": {
        "enabled": false,
        "server_addr": "127.0.0.1",
        "server_port": 3306,
        "database": "trojan",
        "username": "trojan",
        "password": ""
    }
}
```


#### 安全证书TSL配置

##### 手动的方式
在[证书配置网站]([https://freessl.cn/](https://freessl.cn/)
按照指示完成申请之后，就可以下载证书，然后在配置文件中指向证书文件路径即可
###### 下载并解压证书

得到两个文件。  
一个是`xxx.xxx.xxx_chain.crt` 文件  
一个是`xxx.xxx.xxx_key.key` 文件  
把`xxx.xxx.xxx_chain.crt`改名为`fullchain.cer`备用  
把`xxx.xxx.xxx_key.key`改名为`private.key`备用

###### 移动证书文件

创建存放证书的文件夹`trojan-cert` 完整路径为 `/usr/src/trojan-cert`  
把刚才改名的2个文件（`fullchain.cer`和`private.key`）放到VPS `/usr/src/trojan-cert`目录下面

然后在配置文件的证书目录指向这两个文件

###### 启动Trojan服务

设置启动Trojan服务
xxx是配置文件的文件名，按照配置文件开启不同的服务
systemctl start trojan@xxx.service  #启动Trojan
systemctl enable trojan@xxx.service  #设置Trojan服务开机自启

###### 验证证书
进入网址看是否有小锁

##### 自动的方式
下载：`sudo pacman -S acme.sh`

`acme.sh --help`查看配置文档进行学习配置即可


#### TCP Fast Open

For [TCP Fast Open](https://en.wikipedia.org/wiki/TCP_Fast_Open "wikipedia:TCP Fast Open") on servers to work, you'll need to turn it on in your OS:

`echo 3 > /proc/sys/net/ipv4/tcp_fastopen`

#### Disguise

Trojan servers can be disguised as other services over TLS to prevent active probing. This can be done by, for example, running a [web server](https://wiki.archlinux.org/index.php/Web_server "Web server") with [nginx](https://wiki.archlinux.org/index.php/Nginx "Nginx") and pointing `remote_addr` and `remote_port` fields to the server address and port.