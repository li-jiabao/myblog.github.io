---
title: ssh-secure-login
author: lijiabao
e-mail: lijiabao@smail.nju.edu.cn
date: 2021-03-05-20:21:54
categories: ["Linux", "SSH"]
tags: ["Linux", "SSH"]
---


## SSH安全登陆

### 简要介绍
对于ssh的密码登陆验证，可能会存在着[brute-force attack](https://en.wikipedia.ord/wiki/Brute-force_attack)。

因此为了安全起见，应该考虑使用一种更加安全的远程登陆方式。SSH-Keys就是用来作为安全登陆验证的一种方法.SSH keys是一种使用[public-key cryptography](https://en.wikipedia.org/wiki/Public-key_cryptography)和[challenge-response authentication](https://en.wikipedia.org/wiki/Challenge-response_authentication)的验证登陆的方法。

**相比传统的密码验证登陆的方法，ssh keys以下几个优点：**
1. 不容易受到brute-force attack
2. 一旦获取验证授权，不再需要验证
3. 更加方便，不用再输入密码
4. 有相关的程序（ssh agent）可以让你不用记住多个服务器的密码而可以直接输入服务器地址进行登陆

**密钥的背景介绍**

具体的实现是通过ssh-keygen来生成一对密钥，这对密钥中的公钥可以分享到想要连接的服务器上，也就是这个公钥要放在服务器中的authorizedKey文件中来进行验证，而私钥则是自己保密留存。

如果有你的公钥的服务器看见你的连接信号，会使用你的公钥创建并发送一个chanllenge，这个challenge是一个加密信息，而且这个加密信息只有你的私钥才可以正确解开，等到解开之后，才会给予登陆的权限。只有你拥有这个正确的私钥（这个私钥一般保存在`.ssh/`目录下）才可以给予服务器正确的响应信息

私钥是受保护的密码，因此将他放在硬盘并且以加密形式保存才是明智的。当加密的私钥被访问时，首先你必须输入passphrase来解开它。尽管这样看起来像是输入登陆密码，但是这个步骤只在本地进行，不会传送到网络上去，因此会更安全，不易受到袭击。

### 生成SSH Keys

通过执行`ssh-keygen`命令来生成一对密钥，默认是3072-bit的RSA（和SHA256），这两个都是加密算法（*这两个一般情况下安全性已经足够了*）

##### 加入注释
-C参数会添加一个注释到公钥上，使得更容易辨别
`ssh-keygen -C "$(whoami)@$(uname -n)-$(date -I)"`这个命令会加上一个comment告诉哪个用户在什么时候哪个机器上生成密钥

##### 选择加密类型

1. DSA和RSA,主要看分解两个大素数乘积的实际困难度来评判安全度
2. ECDSA和Ed25519，主要看它们依赖于椭圆曲线的离散对数问题的难易度。

[Elliptic curve cryptography](https://blog.cloudflare.com/a-relatively-easy-to-understand-primer-on-elliptic-curve-cryptography/)。ECC算法是最近加入到`public key cryptosystems`的

###### RSA
ssh-keygen默认RSA,没必要指定-t参数，对于所有算法它提供了最好的兼容性，但是要求数字要大才能提供足够的安全性，最小1024，最大16384，默认3072
`ssh-keygen -b 4096`使用-b参数指定一个比默认值大的数字生成更强的密码对


###### ECDSA

使用Ed25519
`ssh-keygen -t ed25519`没必要设置数字大小，所有的生成的key都是256字节


##### 选择密钥位置和passphrase

在你进行了上述其中一条命令之后，就会让你选择密钥的存放位置以及passpharse（用来本地验证公钥的需要输入的密码，可以不设置直接输入回车）

如果你选择不设置passphrase,需要注意==存放的密钥就是没有受到保护的形式保存的，任何可以访问你的计算机的人都可以看到这个私钥==因此，可能不安全。

不修改key的情况下修改passphrase：
`ssh-keygen -f ~/.ssh/id_rsa -p`

##### 管理多个密钥

对于一个密钥使用在多个主机是可以的，但是存在争议。

其实，对于多个密钥的管理是比较好维持的，可以在openSSH配置文件(/etc/ssh/ssh_config)中使用`IdentyFile`重定向密钥文件

```yml
Host SERVER1
	identitiesOnly yes
	IdentifyFile ~/.ssh/id_rsa_SERVER1

Host SERVER2
	identitiesOnly yes
	IdentifyFile ~/.ssh/id_ed25519_SERVER2
```


##### 复制密钥到远程服务器

###### 简单方法
如果key文件是`～/.ssh/id_rsa.pub`下，可以使用下面的命令：
`ssh-copy-id remote-server.org`
`ssh-copy-id username@remote-server.org`用户名和远程机器不一样，需要指定用户名

不是上述情况的，需要指定公钥路径：
`ssh-copy-id -i ~/.ssh/id_ed25519.pub username@remote-server`
如果不是监听22端口还需使用-p参数指定端口

###### 手动方法

`scp ~/.ssh/id_ecdsa.pub username@remote-server.org:/your_wanted_dirctory`
如果远程服务器上不存在`~/.ssh`还需创建，并设置权限为700，还需创建：`~/.ssh/authorized_keys`文件将公钥密码放进去，并设置权限600，设置为只是所有者可读可写


##### SSH agent

如果设置了passphrase密码，你每次登陆和scp都需要输入密码，会很麻烦而ssh agent是一类程序，他可以缓存你的passpharse并代表你提供给ssh server。你只需要提供一次验证，因此在你比较频繁的ssh连接的时候，这个就很方便==agent默认配置为登陆运行，会话结束就关闭。==

下面有好多种ssh agent程序，看自己的需要去使用

###### ssh-agent
[ssh-agent](https://wiki.archlinux.org/index.php/SSH_keys#SSH_agents)是openssh默认包含的程序。当ssh-agent运行时，他会fork一个后台并且输出一些必要的环境变量
`eval $(ssh-agent)`可以利用这些变量

当ssh-agent运行后，你需要 添加你的私钥到他的缓存中：`ssh-agent ~/.ssh/id_ed25519`运行该命令后，会要求你输入一次密码，一旦添加成功之后，后续的连接不再需要输入密码。

> 如果你需要让所有的客户端包含git存储key到这个agent代理，你需要修改配置文件中一项为`AddKeysToAgent yes`

想要让这个代理自动运行，并且确保只有一个进程执行，添加下面的配置到`～/.bashrc or ～/.zshrc`：
```
if ! pgrep -u "$USER" ssh-agent > /dev/null; then
    ssh-agent -t 1h > "$XDG_RUNTIME_DIR/ssh-agent.env"
fi
if [[ ! "$SSH_AUTH_SOCK" ]]; then
    source "$XDG_RUNTIME_DIR/ssh-agent.env" >/dev/null
fi
```

*** Start ssh-agent with systemd user***

It is possible to use the [systemd/User](https://wiki.archlinux.org/index.php/Systemd/User "Systemd/User") facilities to start the agent. Use this if you would like your ssh agent to run when you are logged in, regardless of whether x is running.

~/.config/systemd/user/ssh-agent.service
```
[Unit]
Description=SSH key agent

[Service]
Type=simple
Environment=SSH_AUTH_SOCK=%t/ssh-agent.socket
# DISPLAY required for ssh-askpass to work
Environment=DISPLAY=:0
ExecStart=/usr/bin/ssh-agent -D -a $SSH_AUTH_SOCK

[Install]
WantedBy=default.target
```
Then export the `SSH_AUTH_SOCK="$XDG_RUNTIME_DIR/ssh-agent.socket"` in your login shell initialization file, such as `~/.bash_profile`.

Finally, enable or start the service with the `--user` flag.

**Note:** If you use GNOME, this environment variable is overridden by default. See [GNOME/Keyring#Disable keyring daemon components](https://wiki.archlinux.org/index.php/GNOME/Keyring#Disable_keyring_daemon_components "GNOME/Keyring").

**Tip:** When starting the agent via systemd as described above, it is possible to automatically enter the passphrase of your default key and add it to the agent. See [systemd-user-pam-ssh](https://github.com/capocasa/systemd-user-pam-ssh) for details.

***ssh-agent as a wrapper program***

An alternative way to start ssh-agent (with, say, each X session) is described in [this ssh-agent tutorial by UC Berkeley Labs](http://upc.lbl.gov/docs/user/sshagent.shtml). A basic use case is if you normally begin X with the `startx` command, you can instead prefix it with `ssh-agent` like so:

`ssh-agent startx`

And so you do not even need to think about it you can put an alias in your `.bash_aliases` file or equivalent:

`alias startx='ssh-agent startx'`

Doing it this way avoids the problem of having extraneous `ssh-agent` instances floating around between login sessions. Exactly one instance will live and die with the entire X session.

**Note:** As an alternative to calling `ssh-agent startx`, you can add `eval $(ssh-agent)` to `~/.xinitrc`.

See [the below notes on using x11-ssh-askpass with ssh-add](https://wiki.archlinux.org/index.php/SSH_keys#Calling_x11-ssh-askpass_with_ssh-add) for an idea on how to immediately add your key to the agent.

后面还有许多的代理就不再介绍，详细信息看[==Arch Linux的wiki==](https://wiki.archlinux.org/index.php/SSH_keys#SSH_agents)