---
title: listen指令的用法
date: 2018-12-14
categories: nginx
tags: 学习笔记
description: 
---

### 语法

```nginx
listen address[:port] [default_server] [ssl] [http2 | spdy] [proxy_protocol] [setfib=number] [fastopen=number] [backlog=number] [rcvbuf=size] [sndbuf=size] [accept_filter=filter] [deferred] [bind] [ipv6only=on|off] [reuseport] [so_keepalive=on|off|[keepidle]:[keepintvl]:[keepcnt]];
```

默认为`listen *:80 | *:8000`，作用于server块。

设置IP的地址和端口或指定UNIX域套接字的路径。可以指定地址和端口，也可以只指定地址或只指定端口。

```nginx
listen 127.0.0.1:8000;
listen 127.0.0.1;
listen 8000;
listen *:8000;
listen localhost:8000;
# IPv6地址
listen [::]:8088;
# UNIX-domain
listen unix:/var/run/nginx.sock;
```

如果只有地址，则使用80端口。

如果listen指令不存在，那么如果nginx以超级用户权限运行，则是会用`*:80`，否则使用`*:8000`。

###### default_server参数

default_server用来指定默认服务器，例如`listen 120.0.0.1:8000 default_server`，指定了默认服务器IP的地址和端口为127.0.0.1:8000。

如果没有指定default_server，，那么具有address:port的第一个服务器为默认服务器。

###### ssl参数

ssl指定监听端口上接受的所有连接都应在SSL模式下工作。

```nginx
listen 127.0.0.1 ssl
```

###### http2

http2将端口配置为接受http/2连接。

```nginx
listen 127.0.0.1 ssl http2
```

###### spdy

spdy允许接受此端口上的SPDY连接。

```nginx
listen 127.0.01 spdy
```

###### proxy_protocol

proxy_protocol指定端口上接受的所有连接都应使用代理协议。



以下参数用于socket相关的系统调用，这些参数可以在任何listen指令中指定，但对于给定的address:port只能指定一次。

*setfib=number*

*backlog=number*

为系统调用`listen()`设置backlog参数，用以限制挂起连接队列的最大长度。在FreeBSD,DragonFly BSD和mac OS上默认为-1，其他平台默认511

*rcvbuf=size*

为监听scoket设置接收缓冲区大小。

*sndbuf=size*

为监听scoket设置发送缓冲区大小。

*accept_filter*

为监听scoket设置接收过滤器的名称。socket将连接传递给accept()之前对其进行过滤。

*deferred*

指示在Linux上使用延迟的accept()

*bind*

指示nginx为设置的`address:port*`单独调用一次`bind()`。 这是因为当有多条`listen`指令监听不同地址下的相同端口， 而其中一条`listen`指令监听了这个端口的所有地址(`*:port`)时， nginx只会为`*:port`调用一次`bind()`绑定套接字。 需要留意的是，这种情况下，nginx会调用`getsockname()`系统调用来确定接受请求的套接字地址。 如果为某个`*address*`:`*port*`定义了参数`backlog`、`rcvbuf`、 `sndbuf`、`accept_filter`、`deferred`或者`so_keepalive`， nginx总会为这个地址单独调用一次`bind()`绑定套接字。

*ipv6only=on|off*

这个参数决定监听在通配符`[::]`上的IPv6套接字是只支持IPv6连接，还是同时支持IPv6和Ipv4。默认为on，并且只能在nginx启动时设置。

*so_keepalive=on|off|[keepidel]:[keepintvl]:[keepcnt]*

这个参数为监听套接字配置TCP keepalive行为。
