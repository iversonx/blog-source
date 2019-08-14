---
title: limit_req模块：对请求做限制
date: 2018-12-14
categories: nginx
tags: 学习笔记
description: 
---

`ngx_http_limit_req_module`模块用于限制请求处理速率，特别是来自单个IP地址的请求处理速率。使用"漏斗"方法进行限制。

##### limit_req指令

```nginx
limit_req zone=name [burst=number] [nodelay | delay=number];
上下文: http, server, location
```

设置共享内存区域和请求的最大突发大小。如果请求速率超过`limit_req_zone`配置的速率，则延迟处理。当被延迟的请求数量超过burst指定的大小时，请求以错误终止。

```nginx
http {
    limit_req_zone $binary_remot_addr zone=one:10m rate=1r/s;
    server {
        location /search {
            limit_req zone=one burst=5;
        }
    }
}
```

上面配置，平均每秒允许1个请求，突发不超过5个请求。如果不希望在请求被限制的情况下延迟过多的请求，应使用`nodelay`参数

```nginx
limit_req zone=one burst=5 nodelay;
```

`delay`(1.15.7)参数指定过多请求被延迟的限制。 默认值为零，即所有过多的请求都被延迟。

可能有几个limit_req指令。 例如，以下配置将限制来自单个IP地址的请求的处理速率，同时限制虚拟服务器的请求处理速率：

```nginx
limit_req_zone $binary_remote_addr zone=perip:10m rate=1r/s;
limit_req_zone $server_name zone=perserver:10m rate=10r/s;

server {
    ...
    limit_req zone=perip burst=5 nodelay;
    limit_req zone=perserver burst=10;
}
```

当当前级别没有`limit_req`指令时，这些指令才从上一级继承。

##### limit_req_dry_run指令

启用空运行模式。 在这种模式下，请求处理速率不受限制，但是，在共享存储区中，过多请求的数量照常计算。

```nginx
limit_req_dry_run on | off;
Default: limit_req_dry_run on;
Context: http, server, location
该指令出现在1.17.1版本中。
```

启用空运行模式。 在这种模式下，请求处理速率不受限制，但是，在共享存储区中，过多请求的数量照常计算。

##### limit_req_log_level指令

为服务器因速率超过或延迟请求处理而拒绝处理请求的情况这是所需的日志记录级别。延迟的记录级别比拒绝记录基本低；例如如果指定了“limit_req_log_level notice”，则会使用info级别记录延迟。

```nginx
limit_req_log_level info | notice | warn | error;
默认: limit_req_log_level error;
上下文: http, server, location
```

##### limit_req_status指令

设置拒绝的请求而返回的状态码。

```nginx
limit_req_status code;	
默认: limit_req_status 503;
上下文: http, server, location

```

##### limit_req_zone指令

设置限制请求的共享内存区域的参数，该区域存储key的各种状态。

```nginx
limit_req_zone key zone=name:size rate=rate [sync];
上下文: http
```

```nginx
# 这里定义了一个共享内存区域，其名称为one，其大小为10m；并且该区域的平均请求处理速率为每秒1个请求。
limit_req_zone $binary_remot_addr zone=one:10m rate=1r/s;
```

如果共享内存区域耗尽，则最近最少使用的状态将被删除。即使在此之后无法创建新状态，该请求也会因错误而终止。

`rate`参数以每秒请求数(r/s)指定。也可以在每分钟请求(r/m)中指定，例如30r/m，则表示30个请求/分钟。

### 疑问

1. limit_req_zone中的rate和limit_req中burst同时出现，例如rate=2r/s, burst=5。那么当1秒钟有10个并发时，是不是意味着有7个请求可以成功处理。
