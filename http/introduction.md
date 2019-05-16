---
title: HTTP简介
date: 2018-12-04
categories: http
tags: 学习笔记
description: HTTP是超文本传输协议(HyperText Transfer Protocol)，是互联网的基础协议，也是Web开发的必备知识。HTTP是基于TCP/IP协议的应用层协议。它不涉及数据包传输，主要规定了客户端和服务器之间的通信格式，默认使用80端口。
---

## 前言

我个人觉得HTTP协议是Web开发人员必须要知道的知识点，于是我开始学习了解HTTP协议。我博客中HTTP系列的博文内容主要来自《HTTP权威指南》和http://www.ruanyifeng.com/blog/2016/08/http.html

这篇博文只是对HTTP进行了简单的说明介绍。

## HTTP协议相关链接

- https://www.w3.org/Protocols/

  这个W3C的Web页面中包含了很多与HTTP协议相关的重要链接

- https://www.ietf.org/rfc/rfc2616.txt

  当前HTTP协议版本HTTP/1.1的官方规范

- https://www.ietf.org/rfc/rfc1945.txt

  一个描述了HTTP现代基础的知识性RFC。

- http://www.w3.org/Protocols/HTTP/AsImplemented.html

  介绍了1991年的HTTP/0.9协议。

- http://www.w3.org/Protocols/WhyHTTP.html

- http://www.w3.org/History.html

- http://www.w3.org/DesignIssues/Architecture.html

- http://www.w3.org

- http://www.ietf.org/rfc/rfc2396.txt

  RFC 2396 "Uniform REsource Identifiers(URI): Generic Syntax"是URI和URL的详细参考

- http://www.ietf.org/rfc/rfc2141.txt 

  RFC2141 "URN Syntax" 是一个写于1997年的描述的URN语法的规范。

- http://www.ietf.org/rfc/rfc2046.txt

  RFC2046  "MIME Part 2: Media Types" 是为进行多媒体内容管理而定义的多用途Internet邮件扩展标准的五部规范中的第二部

## 简单描述HTTP协议

浏览一个页面时，浏览器会想服务器发送一条HTTP请求，服务器会去寻找所期望的对象，如果成功，就将对象，对象类型，长度以及其他一些信息放在HTTP响应中发送给客户端。

### 资源

Web服务器托管Web资源，Web资源是Web内容的源头。最简单的Web资源就是Web服务器文件系统上的静态文件。当然资源不一定都是静态资源，也可以是按需生成内容的软件程序。总之所有类型的内容来源都是资源。

#### MIME Type

MIME Type用于标记资源的类型，是一种文本标记，表示一种主要的对象类型和一个特定的子类型，中间由斜杠分隔，例如：

- text/html 表示为HTML格式的文本

- text/plain表示为普通的ASCII文本

- image/jpeg表示为JPEG图片

#### URI

每个Web服务器资源都有一个名字，因此客户端可以指出他们感兴趣的资源。这个服务器资源名被称为 uniform resource identifier(URI)。在世界范围内唯一标识并定温信息资源，例如百度上的一个图片资源：https://www.baidu.com/img/bd_logo1.png

URL有两种形式，分别称为URL和URN。

#### URL

统一资源定位符号(uniform resource locator URL)是最常用的资源标识符形式。URL描述资源在特定服务器上的特定位置。它确切地告诉你如何从一个精确的、固定的位置获取资源。大部分URL都遵循一种标准格式，这种格式包含三部分：

- 第一部分称为scheme，描述了用于访问资源的协议。通常是HTTP协议。

- 第二部分给出了服务器的Internet地址，如www.baidu.com。

- 其余部分指定了资源在Web服务器上的名称。如/img/bd_logo1.png。

目前几乎所有的URI都是URL。

#### URN

URI的第二种形式是 uniform resource name(URN)，作为特定内容的唯一名称，与资源当前位置无关。这使得可以将资源四处搬移。通过URN，还可以用同一个名字通过多种网络协议来访问资源。

### 事务

一个HTTP事务由一条请求命令和一个响应结果组成，使用HTTP报文进行通信。

#### Methods

HTTP支持集中不同的请求命令，这些命令被称为HTTP method。每个HTTP请求报文都包含一个method，用来告诉服务器要执行什么动作。以下是五种常见的HTTP方法

- GET:：从服务器向客户端发送命名资源

- PUT：将来自客户端的数据存储到一个命名的服务器资源中

- POST：将客户端数据发送到一个服务器网关应用程序

- DELETE：从服务器中删除资源

- HEAD：仅发送命令资源响应中的HTTP首部

#### 状态码(Status Codes)

每条HTTP响应报文返回时都会携带一个状态码。状态码是一个三位数字的代码，告知客户端请求是否成功，或者是否需要采取其他动作。下面列几种常见的状态码

- 200：请求成功，文档正确返回

- 404：找不到资源,Not Found

- 302：重定向。让客户端到其他地方去获取资源

### 报文(Messages)

HTTP报文是由一行一行的简单字符串组成的，都是纯文本，可以方便地对其进行读写。

HTTP报文包括以下三个部分

- 起始行：消息的第一行是起始行，指示如何处理请求或响应发生了什么。

- 头部字段：起始行后面有零个或多个头部字段，每个头部字段包含一个名字和一个值，用冒号(:)来分隔以便于解析。头部以一个空行结束。

- 主体：在空行之后是可选的消息主体，可以包含任何类型的数据。

#### 请求报文示例

```http
GET /test/hi-ther.txt HTTP/1.0
Accept: text/*
Accept-Language: en,fr

```

#### 响应报文示例

```http\
HTTP/1.0 200 OK
Content-Type: text/plain
Content-length: 19

Hi,I'm a message!
```

### Web的结构组件

#### 代理

代理是Web安全，应用集成以及性能优化的重要组成模块，位于客户端与服务器之间，接收所有客户端的HTTP请求，并将这些请求转发给服务器，可能会对请求进行修改之后转发。

#### 缓存

Web缓存或代理缓存是一种特殊的HTTP代理服务器，可以将经过代理传送的常用文档复制保存起来，下一个请求同一文档的客户端就可以享受缓存的私有副本所提供的服务。

#### 网关

网关是一种特殊的服务器，作为其他服务器的中间实体使用。通常用于将HTTP流量转成其他的协议。

#### 隧道(tunnel)

隧道是建立起来之后，就会在两条连接之间对原始数据进行盲转发的HTTP应用程序。通常用来在一条或多条HTTP连接上转发非HTTP数据，转发时不会窥探数据。

常见用途是通过HTTP连接承载加密的安全套接字层流量，这样SSL流量就可以穿过只允许Web流量通过的防火墙。

#### Agent代理

用户Agent代理是代表用户发起HTTP请求的客户端程序。所有发布Web请求的应用程序都是HTTP Agent代理。例如：Web浏览器。




