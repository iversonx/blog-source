---
title: HTTP报文结构
date: 2019-06-21
categories: http
tags: 学习笔记
---

### 请求包示例:

```http
GET / HTTP/1.1

Host: 127.0.0.1

Connection: keep-alive

Pragma: no-cache

Cache-Control: no-cache

Upgrade-Insecure-Requests: 1

User-Agent: Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8

Accept-Encoding: gzip, deflate, br

Accept-Language: zh-CN,zh;q=0.9


```

HTTP 协议的请求报文和响应报文的结构基本相同，由三大部分组成：

- 起始行：描述请求或响应的基本信息；

- 头部字段集合：使用key-value形式更详细的说明报文；

- 消息正文：实际传输的数据，可以是纯文本，图片，视频等二进制数据。

### 请求行

```http
GET / HTTP/1.1
```



请求报文的起始行，也称为请求行，描述了客户端想要如何操作服务器的资源。

请求行有三个部分组成

- 请求方法：表示对资源的操作，如GET，POST；

- 请求目标：通常是URI， 标记了请求方法要操作的资源；

- 版本号：表示报文使用的HTTP版本号。

三个部分通常用空格分隔

### 状态行

```http
HTTP/1.1 404 Not Found
```



响应报文的起始行，也称为状态，它表示了服务器的响应状态。

状态行同样由3个部分组成

- 版本号：表示报文使用的HTTP协议版本；

- 状态码：一个三位数，用代码的形式表示处理的结果，比如200，500

- 原因：作为数字状态码补充，详细解释状态。

### 头部字段

请求头和响应头的结构基本一样，唯一的区别是起始行。头部字段是key-value形式，中间用":"分隔，最后用CRLF换行表示字段结束。比如"Host: 127.0.0.1"，这里的key就是Host，value就是127.0.0.1。

头部字段非常灵活，不仅可以使用标准里的Host、Connection的头部，也可以任意添加自定义头。

注意点：

- 字段名不区分大小写，但首字母大写的可读性好；

- 字段名里不允许出现空格，可以使用连字符“-”，但不能使用下划线；

- 字段名后必须紧接着“:”，不能有空格，":"后的字段值前可以有多个空格；

- 字段的顺序是没有意义的，可以任意排列不影响语义；

- 字段原则上不能重复，除非字段本身的语义允许，如`Set-Cookie`。

### 常用头部字段

- Host：属于请求头部字段，只能出现在请求头里，用于告诉服务器这个请求应该由哪个主机来处理。

- User-Agent：请求字段，使用字符串描述发起HTTP请求的客户端。服务器可以依据它来返回最合适此浏览器显示的页面。

- Date：通用字段，表示HTTP报文创建的事件，通常出现在响应头里，客户顿可以使用这个时间在搭配其他字段决定缓存策略。

- Server：响应字段，只出现在响应头。它告诉客户端当前正在提供Web服务的软件名称和版本号。

- Content-Length：表示报文里body的长度，如果没有这个字段，那么body就是不定长的。
