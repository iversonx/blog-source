---
title: 如何找到处理请求的server指令块
date: 2018-12-14
categories: nginx
tags: 学习笔记
description: 
---

##### server_name指令

server_name用于指定虚拟服务器的名称。只能位于server指令块上下文中

```nginx
# 语法
server_name name ...;
# 默认
server_name "";
# 上下文： server指令块
```

server_name可以有3种形式

```nginx
# 可以指定明确的域名
server_name www.lijie.com
# 可以指定泛域名，只支持在最前或最后
server_name *.lijie.com wwww.lijie.*
# 正则表达式
server_name ~^www\d+\.lijie\.tech$;
```

##### server匹配顺序

1. 首先使用精确匹配

2. 当没有精确匹配时，优先处理*在前的泛域名；

3. 当没有匹配到\*在前的泛域名时，匹配\*在后的泛域名；

4. 当以上3种都没有匹配到时，按文件中的顺序匹配正则表达式域名

5. 使用default server，所有server指令块中第一个server或listen指定default的server。


