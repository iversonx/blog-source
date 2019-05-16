---
title: HTTP 消息
date: 2018-12-12
categories: http
tags: 学习笔记
description: HTTP 消息包括从客户端到服务器的请求以及从服务器到客户端的响应。请求消息和响应消息使用通用消息格式来传输实体。两种类型的消息都包括一个起始行，零个或多个头，一个空行(即CRLF前面没有任何内容的行)表示头的结尾，可能还有个消息体。
---

## 消息类型

HTTP 消息包括从客户端到服务器的请求以及从服务器到客户端的响应。请求消息和响应消息使用通用消息格式来传输实体。两种类型的消息都包括一个起始行，零个或多个头，一个空行(即CRLF前面没有任何内容的行)表示头的结尾，可能还有个消息体。为了保持健壮性，服务器应该忽略请求行前面的空行。

### 请求报文的格式

```http
<method> <request-URL> <version>
<headers>

<entity-body>
```



### 响应报文的格式

```http
<version> <status> <reason-phrase>
<headers>

<entity>
```

对上面的各部分简要的描述。

- method：客户端希望服务器对资源执行的动作。

- request-URL：一个完整URL，用于命名所请求的资源或URL的Path组件。

- version：报文所使用的HTTP版本，其格式看起来是这样的: HTTP/<major>.<minor>，major(主要版本)和minor(次要版本)都是整数。

- status-code：描述了请求过程中所发生的情况。

- reason-phrase：状态码的可读版本

- headers：可以有零个或多个头部，每个头部都包含一个名字，后面跟着一个冒号(:)，接着是一个可选的空格，然后是值。最后是一个CRLF。

- entity-body：主体部分包含一个由任意数据组成的数据块，是可选的。

## 起始行(start-line)

所有HTTP消息都以一个起始行作为开始。请求消息的起始行说明了要做什么，响应消息的起始行说明了发生了什么。

### 请求行(request-line)

请求消息的起始行称为请求行，包含一个方法和一个请求URL以及HTTP的版本，它们之间以空格分隔。方法描述了服务器应该执行的操作，请求URL描述了要对哪个资源执行这个方法。HTTP版本用来告诉服务器，客户端使用的HTTP版本。

```http
GET /there.txt HTTP/1.1
Accept: test/*
Host: www.aaaaa.com

```



### 响应行(response-line)

响应消息的起始行称为响应行，包含了响应消息使用的HTTP版本、数字状态码，以及原因短语。

```http
HTTP/1.0 200 OK
Content-type: text/plain
Content-length: 19

Hi,I'm a message
```

## 消息头(Message Headers)

HTTP头字段包括通用头部，请求头部，响应头部和实体头。

每个头字段包含一个名称，后面跟一个冒号(:)和字段值，字段名不区分大小写。字段值前面可以有任意数量的线性空白(LWS)，但最好使用单个空格(SP)。

将长的首部行分为多好可提供可读性，多出来的每行里面至少要有一个空格和制表符。

```
HTTP/1.0 200 OK
Content-Type: image/gif
Content-Length: 4321
Server: Test Server
    Version 1.0
```



## 消息体(Message Body)

HTTP消息的消息体用于携带与请求或响应相关联的实体主体。消息中允许消息正文的规则因请求和响应而异。

对于响应消息，消息是否包含在消息中取决于请求方法和响应状态码；对于HEAD请求方法的所有响应都不能包含消息体；所有1xx，204，304响应不得包含消息体。

消息体只有在应用了传输编码时才与实体主体不同，如Transfer-Encoding头部字段所示。Transfer-Encoding必须用于指示应用程序应用的任何传输编码，以确保消息的安全和正确传输。请求中消息体的存在是通过在请求的消息头中包含Content-Length或Transfer-Encoding报头字段来表示的。如果请求方法的规范不允许在请求中发送实体主体，则消息主体必须不包括在请求中。服务器应该根据任何请求读取和转发消息体；如果请求方法不包括实体体的定义语义，则在处理请求时忽略消息体。

## 方法(Method)

请求行以方法作为开始，用来告知服务器要做什么。HTTP规范中定义了一组常用的请求方法。

### 安全方法

GET方法和HEAD方法的HTTP请求都不会产生什么动作，被认为是安全方法。不产生动作，在这里意味着HTTP请求不会在服务器上产生什么结果。例如，在购物时，点击了提交购买的按钮，会提交一个带有支付信息的POST请求，那么在服务器上就会为购买执行一个动作。这种情况，为购买行为支付信用卡就是所执行的动作。

安全方法不一定什么动作都不执行，实际上由Web开发者决定的。使用安全方法的目的就是允许HTTP应用程序开发者通知用户，什么时候会使用某个可能引发某些动作的不安全方法。

### GET 方法

GET是最常用的方法，通常用于请求服务器发送某个资源。HTTP/1.1要服务器必须实现此方法。

```http
GET /seasonal/index-fall.html HTTP/1.1
Host: www.xxx.com
Accept: *
```

```http
HTTP/1.1 200 OK
Content-Type: text/html
Context-Length: 200

<HTML>
    <HEAD><TITLE>Hello World</TITLE></HEAD>
</HTML>
```

### HEAD

HEAD方法与GET方法的行为很类似，但服务器在响应中只返回头部。这就允许客户端在未获取实际资源的情况下，对资源的头部进行检查，HEAD的作用：

- 在不获取资源的情况下了解资源的情况；

- 通过查看响应中的状态码，看看某个对象是否存在；

- 通过查看头部，测试资源是否被修改。

服务器开发者必须确保HEAD返回的头部和GET请求返回的头部完全相同。遵循HTTP/1.1规范，就必须实现HEAD方法。

### PUT

PUT方法会向服务器写入文档；PUT方法的语义就是让服务器用请求的主体部分来创建一个由所请求的URL命名的新文档，如果URL已经存在，就替换。

### POST

POST方法起初是用来向服务器输入数据的。通常会用它来支持HTML的表单提交。

### TRACE

客户端发起一个请求时，请求可能要穿过防火墙，代理，网关或其他应用程序，每个中间节点都可能会改变原始的HTTP请求。TRACE方法允许客户端在最终将请求发送给服务器时，看看它变成了什么样子。

TRACE请求会在目的服务器端发起一个"环回"诊断。最后一站的服务器会弹回一条TRACE响应，并在响应主体中携带它收到的原始请求报文。这样客户端就可以查看在所有中间HTTP应用程序组成的请求/响应链上，原始报文是否，以及如何被毁坏和修改。

```http
// 请求报文
TRACE /product-list.txt HTTP/1.1
Accept: *
Host: www.xxxxx.com

// 代理接收请求报文
TRACE /product-list.ext HTTP/1.1
Host: www.xxxxx.com
Accept: *
Via: 1.1 proxy3.company.com

// 响应报文
//服务器给代理响应
HTTP/1.1 200 OK
Content-type: text/plain
Content-length: 96

TRACE /product-list.txt HTTP/1.1
Host: www.xxxxx.com
Accept: *
Via: 1.1 proxy3.conmpany.com

//代理响应给客户端
HTTP/1.1 200 OK
Content-type: text/plain
Content-length: 96
Via: 1.1 proxy3.conmpany.com

TRACE /product-list.txt HTTP/1.1
Host: www.xxxxx.com
Accept: *
Via: 1.1 proxy3.conmpany.com
```

TRACE方法主要用于诊断；用于验证请求是否如愿穿过了请求/响应链。可以用来查看代理和其他应用程序对用户请求所产生效果。

TRACE请求中不能有主体。TRACE响应的主体包含了响应服务器收到请求的精确副本。

### OPTIONS

OPTIONS方法请求Web服务器告知其支持的各种功能。

```http
OPTIONS * HTTP/1.1
Host: www.xxxxx.com
Accept: *

// 响应
HTTP/1.1 200 OK
Allow: GET, POST, PUT, OPTIONS
Content-length: 0
```

由于请求的是可以所有资源使用的选项，所以服务器仅返回它所支持的可通用于各种资源的方法。

##### DELETE

DELETE 用于请求服务器删除请求URL所指定的资源。客户端应用程序无法保证删除操作一定会被执行。因为HTTP规范允许服务器在不通知客户端的情况下撤销请求。

```http
DELETE /product-list.txt HTTP/1.1
Host: www.xxxxx.com

// 响应
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 54

I have your delete request, will take time to process.
```

### 扩展方法

HTTP被设计成字段可扩展的，这样新的特性就不会使老的软件失效。扩展方法指的是没有在HTTP/1.1规范中定义的方法。列出一些常见的扩展方法，这些方法是WebDAV HTTP扩展包含的所有方法，这些方法有助于通过HTTP将Web内容发布到Web服务器上。

- LOCK 允许用户锁定资源

- MKCOL 允许用户创建资源

- COPY  便于在服务器上复制资源

- MOVE 在服务器上移动资源


