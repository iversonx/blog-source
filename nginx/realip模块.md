---
title: realip模块(ngx_http_realip_module)
date: 2018-12-14
categories: nginx
tags: 学习笔记
description: 
---

realip模块用于将客户端地址和端口更改为指定头字段中发送的地址和端口。

realip模块默认不会编译进nginx，编译时通过`-with-http_realip_module`将其编译到nginx。

### 指令

##### set_real_ip_from

```nginx
# 默认
set_real_ip_from address | CIDR | unix:;
# 可以在http块，serverkuai，location块中出现
```

定义一个可信地址，realip模块将从这个地址发送过来的请求头X-Forwarded-For里获取用户真实的ip，并将真实ip替换到remote_addr变量中。

例如

```nginx
set_real_ip_from 192.168.123.20
# 当set_real_ip_from配置为 192.168.123.20时，realip模块将从192.168.123.20发送过来的请求的X-Forwarded-For中去获取用户真实的ip地址。
```

##### real_ip_header

```nginx
real_ip_header field | X-Real-IP | X-Forwarded-For | proxy_protocol;
# 默认 X-Real-IP
# 可以在http块，serverkuai，location块中出现
```

指定从哪个请求头部中获取ip，默认从`X-Real-IP`头中获取；如果是从`X-Forwarded-For`头中获取，那么取`X-Forwarded-For`中的最后一个ip。

```nginx
real_ip_header X-Forwarded-For;
```

##### real_ip_recursive

```nginx
real_ip_recursive on | off;
# 默认为off
# 可以在http块，serverkuai，location块中出现
```

当`real_ip_recursive`为off时，nginx会把`real_ip_header`指定的HTTP请求头中的最后一个ip作为用户真实ip，当`real_ip_recursive`为on时，nginx会把`real_ip_header`指定的HTTP头中的最后一个不是`set_real_ip_from`指定的ip，认为是用户的真实ip。

### 变量

##### $realip_remote_addr

保留原始客户端的地址

##### $realip_remote_port

保留原始客户端的端口
