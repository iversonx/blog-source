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



### return

### rewrite

### rewrite_log

### set

### uninitialized_variable_warn
