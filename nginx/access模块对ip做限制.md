---
title: access模块：对ip做限制
date: 2018-12-14
categories: nginx
tags: 学习笔记
description: 
---

access模块(ngx_http_access_module)用于限制对某些客户端地址的访问，在access阶段中生效。access模块提供了两个指令allow, deny。

##### allow指令

```nginx
allow address | CIDR | unix: | all;
上下文: http,server,location,limit_except
```

允许指定网络或地址访问。

##### deny指令

```nginx
deny address | CIDR | unix: | all;
上下文: http,server,location,limit_except
```

不允许指定网络或地址访问。

##### 示例

```nginx
location / {
    deny 192.168.1.11;
    allow 192.168.1.0/24;
    allow 10.1.1.0/16;
    allow 2001:0db8::/32;
    deny all;
}
```


