---
title: Nginx负载均衡
author: lijiabao
date: 2021-03-10 19:00
description: 这篇文章主要介绍Nginx的负载均衡策略，和一些应用场景
categories: [Web相关, Nginx]
tags: [Web相关, Nginx]
---

介绍nginx下的集中负载均衡策略以及他们的实现使用

## **负载均衡**


Nginx负载均衡实现主要是从配置文件中定义的`upstream`模块定义的后端服务器列表中选取一台服务器接受用户的请求。
最基本的upstream模块和代理到服务器的配置示例：
```yaml
upstream dynamic_server {
	server server-host1:port;
	server server-host2:port;
	server server-host3:port;
}

# 设置好了之后，还需要在http模块的location模块中指定访问反向代理的客户端到服务器
location / {
	index index.jsp index.html
	proxy_pass http://dynamic_server;
}
```


### 负载均衡策略
nginx自身有的负载均衡策略主要有六种：
- 轮询，权重也是轮询的一种，只是增加了一个权重值改变轮询几率
- ip_hash：ip的hash策略
- 更一般的hash策略
- least_conn最小连接策略
- least_time最小连接时间策略
- random随机策略

#### 轮询

每收到一个请求，通过不断循环给到不同的服务器，按时间顺序逐一分配
**参数说明：**

参数 | 说明
:---: | :---:
`fail_timeout` | 与`max_fails`结合使用
`max_fails` | 设置在`fail_timeout`时间内最大的失败次数，如果在这个时间内所有请求都失败了，认为该服务器停机了
`fail_time` | 服务器会被认为停机的时间长度，默认10s
`backup` | 标记该服务器为备用服务器,当主服务器停止时,请求会被发送到这
`down` | 标记服务器永久停机

<font color='yellow'>注意:</font>
- 轮询中,如果服务器down掉了会自动剔除服务器
- 缺省配置就是轮询
- 此策略适合服务器配置相当,无状态且短平快的服务使用

**权重策略：**

权重方式就是在轮询的基础上指定轮询几率，根据不同服务器的容量和性能，给予不同的权重来达到负载均衡来解决负载均衡
```yaml
upstream dynamic_server {
	server server-host1:port weight=2;
	server server-host2:port;
	server server-host3:port max_fails=3 failtimeout=20s;
	server server-host4:port backup;
}

# weight参数用以指定轮询几率，weight默认值是1，weight数值与访问率成正比
```
<font color='yellow'>注意：</font>
- 权重越高分配到需要处理的请求越多
- 此策略可以和`least_conn`和`ip_hash`结合使用
- 此策略比较适合服务器的硬件配置差别较大的情况

#### ip_hash

指定负载均衡器按照基于客户端的ip的分配方式，使用这种特殊的策略来保证每个客户端固定访问同一台服务器，以保证session会话。这样每个客户端能固定访问同一个后端服务器，解决session不能跨服务器的问题。使用在需要session一致的情况
```yaml
upstream dynamic_server {
	ip_hash;   # 保证每个访客固定访问一个后端服务器
	server server-host1:port weight=2;
	server server-host2:port;
	server server-host3:port max_fails=3 failtimeout=20s down;
	server server-host4:port backup;
}
```
<font color='yellow'>注意：</font>
- nginx1.3.1之前，不能在ip_hash下使用权重
- ip_hash不能和backup同时使用
- 此策略适合有状态服务，比如session
- 当有服务器需要剔除，必须手动down掉
- 如果需要暂时移除一个负载均衡服务器，可以设置为down，然后被这个服务器处理的请求会自动送到同一group的下一个服务器
- 标记为down的服务器会停止使用，主要是为了保持客户端ip的hash

#### least_conn

把请求转发给连接数较少的后端服务器。轮询算法是把请求平均的转发给各个后端，但是可能会存在某些请求占用时间很长的情，导致某些后端服务器负载过大，这种时候，least_conn可以达到更好的负载均衡效果
```yaml
upstream dynamic_server {
	least_conn;   # 把请求转发给连接数较少的后端服务器
	server server-host1:port weight=2;
	server server-host2:port;
	server server-host3:port max_fails=3 failtimeout=20s;
	server server-host4:port backup;
}
```
<font color='yellow'>注意：</font>
- 此负载均衡策略适用请求处理时间长短不一造成服务器过载的情况
- 对于负载均衡器可以查看所有请求的的环境，使用其他的负载均衡方法

<font color='yellow'>注意：</font>
当配置除了轮询之外的其他负载均衡方法时，把响应的配置项放在server的upstream配置项的最上面。

#### least_time（只支持Nginx Plus）

对于每个连接，Nginx Plus使用最低平均延迟和最小活动连接数，这中间的最低平均延迟计算是基于下面的几个参数和least_time参数：
- `header`从服务器上取得第一个字节的时间
- `last_byte`从服务器获取完全响应的时间
- `last_byte inflight`考虑不完全连接的时候，从服务器取得完全响应的时间
**示例：**
```yaml
upstream backend {
	least_time header;
	server server1:port;
	server server2:port;
}
```


#### Generic Hash

请求发送给的服务器由用户定义的key来决定的，这个key可以是文本字符串、变量或者二者的混合。例如：
```yaml
# 这个例子这个key是特定的配对的ip和端口或者一个uri
upstream backend {
	# 可选的consistent参数就是开启ketama consistent-hash负载均衡方法
	hash $request_uri consisitent;
	server server-host1:port;
	server server-host2:port;
}
```
<font color='yellow'>注意：</font>
- 请求基于用户定义的hashed key值将请求平均分配到所有的upstream server上去
- 当从upstream group中进行服务器添加或者删除时，只重新remapped一部分key，减少了缓存丢失

#### Random

每个请求随机分配到随机选择的服务器上，示例：
```yaml
# 随机分配的例子，使用了two参数，首先nginx考虑权重的情况下随机选择两个服务器
# 然后再通过指定的方法选择这两个中的一个服务器来处理请求i
upstream backend {
	random two least_time=last_byte;
	server server-host1:port;
	server server-host2:port;
	server server-host3:port;
}
```
**可选的参数：**
- `least_conn`活动连接的least numbers
- `least_time=header`（Nginx Plus）从服务器获取响应头的最小平均时间（`$upstream_header_time`）
- `least_time=last_byte`（Nginx Plus）从服务器获取完整的响应的最小平均时间(`$upstream_response_time`）

<font color='yellow'>注意：</font>
随机负载均衡方法应该用在负载均衡器传递请求头到同一个upstream集合上的分布式环境


#### 第三方的负载均衡策略

第三方的负载均衡策略需要安装第三方插件
1. `fair`：按照服务器的响应时间来分配请求，响应时间短的优先分配
```yaml
upstream dynamic_server {
	server server-host1:port weight=2;
	server server-host2:port;
	server server-host3:port max_fails=3 failtimeout=20s;
	server server-host4:port backup;
	fair;  # 实现响应时间的短的优先分配
}
```
2. `url_hash`
按照访问的url的hash结果来分配请求，使每个url定向到同一个后端服务器，要配合缓存来使用。同一资源多次请求，可能会到达不同的服务器，导致不必要的多次下载，缓存命中率不高，以及一些资源时间的浪费。使用url_hash，可以是得同一个url（同一个资源请求）会到达同一台服务器，一旦缓存住了资源，再次受到此请求，就可以从缓存中读取。
```yaml
upstream dynamic_server {
	hash $request_uri;   # 实现每个uri定向到后端服务器
	server server-host1:port weight=2;
	server server-host2:port;
	server server-host3:port max_fails=3 failtimeout=20s;
	server server-host4:port backup;
}
```

### 负载均衡的其他一些参数设置

#### weight权重
```yaml
upstream backend {
	server server-host1:port weight=5;
	server server-host2:port;
	server server-host3:port backup;
}
# 这种配置下，当有六个请求时，server-host1会分配5个，server-host2只分配一个
```

#### slow_start（Nginx Plus）
这个slow_start参数时用来避免刚从过载回复回来的服务器又因为timeout而被标记为失效

#### 开启session维持功能（Nginx Plus）


#### 限制最大连接

`max_conns`，通过指定最大连接数量来限制服务器的连接量
