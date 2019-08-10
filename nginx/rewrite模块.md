---
title: rewrite模块(ngx_http_rewrite_module)
date: 2018-12-14
categories: nginx
tags: 学习笔记
description: 
---

rewrite模块用于使用正则表达式改变请求的URI，返回重定向，并有条件地选择配置。rewrite模块

rewrite模块提供了`break`,`if`,`return`,`rewrite`,`rewrite_log`,`set`,`uninitialized_variable_warn`指令。这些指令按以下顺序处理：

- 在server块指定的指令按顺序执行；

- 重复
  
  - 基于请求uri查找location;
  
  - 匹配的location块中rewrite模块的指令是按顺序执行;
  
  - 如果重写请求uri，则重复循环，但不超过10次。

### break

停止执行当前的rewrite模块指令集，而其他模块的指令不受影响。

```nginx
# 语法
break;
# 作用域 server, location, if

# 示例
server {
    listen 8015;
    location /bk {
        break;
        return 200 'return\n;';
        proxy_pass http://127.0.0.1:8014/;
    }
}

server {
    listen 8014;
    location / {
        return 200 '8014 response\n';
    }
}
```

在上面示例中，我们在location块中使用了`break`，`return`指令以`proxy_pass`指令。当我们访问/bk时，`return`指令将得不到执行，因为`break`执行完后，rewrite模块的其他指令将不会再执行；但是`proxy_pass`指令还是正常执行，因此最后我们将得到`8014 response`。

### if

if指令用于进行条件判断，如果条件为真，则执行if块内的指令。

```nginx
# 语法
if(condition){}
# 作用域 server,location
```

if指令的条件表达式

- 如果变量的值为空字符串或字符串0，则为false

- 变量与一个字符串比较，相等使用(=)，不相等使用(!=)

- 将变量与正则表达式做匹配。使用`~`表示匹配，`!~`表示不匹配；如果希望不区分大小写，则使用`~*`和`!~*`。

- 检查文件是否存在,使用`-f`和`!-f`

- 检查目录是否存在,使用`-d`和`!-d`

- 检查文件、目录、软链接是否存在,使用`-e`和`!-e`

- 检查是否为可执行文件, 使用`-x`和`!-x`

```nginx
# 示例
if($variable){
    
}

# 使用变量与正则表达式匹配
if($variable ~ '^rewrite\.lijie\.com$'){}

# 检查文件是否存在
if(-f '/nginx.conf'){}

```

### return

停止处理并返回指定code到客户端。非标准状态码444用于在不发送响应头的情况下关闭连接。可以有以下3种语法

```nginx
# 指定code和响应文本
return code [text];
# 指定code和重定向URL
return code URL;
# 指定重定向URL
return URL;
# 作用域 server,location,if
```

- 当return后面指定code为301,302,303,307,308时，第二个参数则一定会被当作一个URL；

- 当code为其他值时，第二个参数被当做响应文本;

- 当采用第三种语法时，nginx默认使用的http code为302，对请求进行临时重定向。

### rewrite

```nginx
rewrite regex replacement [flag];
# 作用域 server, location, if
```

如果指定的正则表达式与请求URI匹配，则替换成replacement指定的新的url。

`rewrite`指令按照它们在配置文件中的出现顺序执行。

如果replacement以`http://`,`https://`,`$scheme`开头，则 直接返回302重定向。

flag参数可以是以下几个值:

- last 根据新的URL重新在配置文件中进行匹配

- break 停止当前脚本指令的执行，等同于break指令

- redirect 返回302重定向

- permanent 返回301重定向

```nginx

```

### rewrite_log

开启或关闭`rewrite`模块指令执行的日志，如果开启，则重写将记录下`notice`等级的日志到nginx的error_log中，默认为off

```nginx
rewrite_log on | off;
# 作用域 http, server, location, if
```

### set

设置变量

```nginx
set $variable value;
# 作用域 server, location, if
```

### uninitialized_variable_warn

控制是否记录有关未初始化变量的警告。

```nginx
uninitialized_variable_warn on | off;
# 默认为on
# 作用域 http, server, location ,if
```
