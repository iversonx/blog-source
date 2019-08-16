---
title: index模块和auto_index模块
date: 2018-12-14
categories: nginx
tags: 学习笔记
description: 
---

##### index模块

index模块(ngx_http_index_module)处理以斜杆结尾的请求。当访问以斜杠结尾时，index模块就发挥作用，它会去`root`或`alias`指定目录中有没有index指定的文件。

```nginx
index file ...;
默认：index index.html;
上下文: http, server, location
```

定义用作index的文件，文件名可以包含变量。如果指定多个文件，则按顺序检查文件。

##### autoindex模块

autoindex模块(ngx_http_autoindex_module)处理以斜杠结尾的请求，并生成目录列表。当index模块找不到文件时，通常会将请求传递给autoindex模块。

###### autoindex指令

启用或禁用目录列表输出

```nginx
autoindex on | off;
默认: off;	
上下文: http, server, location

```

###### autoindex_exact_size指令

对html格式，指定是应在目录列表中输出确切的文件大小，或更确切地舍入千字节，兆字节和千兆字节。

```nginx
autoindex_exact_size on | off;
默认 on;
上下文: http, server, location
```

###### autoindex_format指令

设置目录列表格式

```nginx
autoindex_format html | xml | json | jsonp;
默认 html
上下文 http, server, location
```

使用JSONP格式时，使用回调请求参数设置回调函数的名称。 如果参数缺失或具有空值，则使用JSON格式。

可以使用ngx_http_xslt_module模块转换XML输出。

###### autoindex_locatime指令

对于HTML格式，指定目录列表中的时间是否应以本地时区或UTC输出。

```nginx
autoinde_localtime on | off;
默认 off
上下文 http, server, location
```


