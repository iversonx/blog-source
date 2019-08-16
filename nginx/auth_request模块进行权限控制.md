---
title: auth_request模块进行权限控制
date: 2018-12-14
categories: nginx
tags: 学习笔记
description: 
---

auth_request模块(ngx_http_auth_request_module)在收到请求后，生成子请求，通过反向代理把请求传递给上游服务器。如果子请求返回2xx响应码，则允许访问。如果返回401或403，则拒绝访问并返回相应的错误代码。子请求返回其他的代码，都视为错误。

该模块默认不会添加到nginx，需要使用`--with-http_auth_request_module`参数编译到nginx中。

该模块提供了两个指令`auth_request`和`auth_request_set`。

##### auth_request指令

基于子请求的结果启用授权，并设置子请求的uri。

```nginx
auth_request uri | off;
默认：auth_request off;
上下文：http, server, location;
```

##### auth_request_set指令

授权请求完成后，将请求变量设置为给定值。该值可以包含授权请求中的变量。

```nginx
auth_request_set $变量名 值;
上下文: http, server, location
```


