---
title: HTTP请求处理时的11个阶段
date: 2018-12-14
categories: nginx
tags: 学习笔记
description: 
---

##### POST_READ阶段

POST_READ阶段是nginx处理请求流程中第一个可以添加模块函数的阶段，任何需要在接收完请求头之后立刻处理的逻辑可以在该阶段注册处理函数。

realip模块在该阶段注册生效，realip模块用于将客户端的ip替换为请求头中保存的值。

realip模块需要在其他模块执行之前将客户端ip替换为真实值，并且它需要的信息仅仅是请求头，因此它在POST_READ阶段注册。

##### SERVER_REWRITE阶段

请求进入SERVER_REWRITE阶段时，说明已经找到对应的`server`配置，但URI还没有与location匹配。nginx的rewtire模块在这个阶段注册一个handler，rewrite模块提供url重现指令rewrite, 变量设置指令set, 以及if, break, return。通过这些指令，可以将一些满足条件的uri重定向到固定url，还可以根据请求的属性来决定是否重写状态码或发送特定状态码。

该阶段执行的结果会影响后面find_confg阶段的执行。

##### FIND_CONFIG阶段

FIND_CONFIG阶段中,nginx会根据uri查找location配置。

##### REWRITE阶段

REWRITE阶段为location级别的重写，rewrite模块也在这个模块注册一个handler，这个handler与SERVER_REWRITE阶段的handler是同一个。唯一区别是执行时不同。

##### POST_REWRITE阶段

该阶段不能注册handler，仅仅是检查上一个阶段是否做了uri重写，如果没有重写，直接进入下一个阶段；如果有重写，则跳转到FIND_CONFIG阶段，重新执行。Nginx对uri重写次数做了限制，默认时10次。

##### PREACCESS阶段

当请求进入PREACCESS阶段，说明请求已经匹配到了某个location。该阶段一般用来做资源控制，默认情况下，ngx_http_limit_conn_module, ngx_http_limit_req_module会在该阶段中注册，用于控制连接数，请求速率等。

##### ACCESS阶段

ACCESS阶段首要目的是做权限控制，默认情况下，ngx_http_access_module和ngx_http_auth_basic_module模块分别会在该阶段注册。

##### POST_ACCESS阶段

POST_ACCESS阶段和POST_WRITE阶段一样，只是处理上一阶段的结果，如果ACCESS阶段返回NGX_HTTP_FORBIDDEN或NGX_HTTP_UNAUTHORIZED，该阶段会结束请求。

##### PRECONTENT阶段

在处理CONTENT之前，做一些处理，比如try_files, mirror

##### CONTENT阶段

请求从CONTENT阶段开始执行业务逻辑并产生响应。例如反向代理,index, concat等都在这个阶段生效。

##### LOG阶段

LOG阶段主要目的是记录访问日志，进入这个阶段说明请求的响应已经发送到系统发送缓冲区。
