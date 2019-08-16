---
title: nginx对访问目录却没有斜杠的处理
date: 2018-12-14
categories: nginx
tags: 学习笔记
description: 
---

在使用`root`和`alias`功能时，发现访问目标是目录，但URL末尾没有加/时，ngxin会返回301重定向。例如访问`http://127.0.0.1/a`，如果a是个目录，那么nginx会返回301重定向到`http://127.0.0.1/a/`。

core模块提供了3个指令来处理重定向的域名：`server_name_in_redirect`,`port_in_redirect`,`absolute_redirect`。

##### server_name_in_redirect指令

在nginx发出的绝对重定向中，启用或禁用server_name指令指定的主服务器名称。禁用时将使用`Host`请求头中的值，如果`Host`头不存在，则使用服务器ip地址。

```nginx
server_name_in_redirect on | off;
默认: off;
上下文：http, server, location
```

##### port_in_redirect指令

启用或禁用在nginx发出的绝对重定向中指定的端口。

```nginx
port_in_redirect on | off;
默认：on;
上下文： http, server, location
```

##### absolute_redirect指令

如果禁用，nginx发出的重定向将是相对的；启用，nginx将发出绝对重定向。

```nginx
absolute_redirect on | off;
默认：on;
上下文：http, server, location
```

##### 示例

```nginx
location /test {
    absolute_redirect off;
    root html/;
}
```

当访问/test时，因为/test是目录，因此nginx会返回301重定向；由于配置了`absolute_redirect`为off，因此在重定向响应头中`Location`字段指定的路径将是相对的，即`Location: /test/`。

```nginx
location /test {
    absolute_redirect on;
    port_in_redirect off;
}
```

上述示例配置`port_in_redirect`为off，所以在301重定向中 `Location`值不会带上端口号`http://127.0.0.1/test/`；配置`port_in_redirect`为on时，`Location`值为`http://127.0.0.1:8080/test/`。

```nginx
server_name www.aaa.com
location /test {
    absolute_redirect on;
    server_name_in_redirect on;
}
```

上述示例配置`port_in_redirect`为on，所以在301重定向中 `Location`值为`http://www.aaa.com/test/`。如果为off，当请求头中的`Host`不为空时，则重定向中`Location`为`Host`的值加上/test/；如果`Host`不存在，则`Location`为服务器的IP地址。
