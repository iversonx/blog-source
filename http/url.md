---
title: URL与资源
date: 2018-12-05
categories: http
tags: 学习笔记
description: URL是浏览器寻找信息时所需的资源位置。通过URL人类和应用程序才能找到、使用并共享Internet上大量的数据资源。URI是一类更通用的资源标识符，由两个主要的子集构成URL和URN构成，URL时通过描述资源的位置来标识资源的，而URN则是通过名字来识别资源的。
---

## 前言

在[HTTP简介](https://iversonx.github.io/http/introduction.html)中，初步了解了URL是最常用的URI形式，用来描述资源在特定服务器上的特定位置。这篇文章主要介绍了URL的语法，相对URL，和如何将相对URL解析成绝对URL。

## URL的语法

URL语法随着scheme的不同而有所不同。不过大多数scheme的URL语法都建立在这个由9部分构成的通用格式上：

```http
<scheme>://<user>:<password>@<host>:<port>/<path>;<params>?<query>#<frag>
```

### scheme

scheme是规定如何访问指定资源的主要标识符，它会告诉负责解析URL的应用程序应该使用什么协议。scheme必须以一个字母开始，由第一个“:”符号将其与其他部分分隔。

### host 和 port

host标识了 Internet能访问托管资源的机器。可以用hostname或IP地址来表示主机名。

post标识了服务器正在监听的网络端口。对HTTP来说，默认端口为80。

### user 和 password

很多服务器都要求输入用户名和密码才能访问数据。例如FTP服务器就是这样的实例。

```http
ftp://ftp.prep.ai.mit.edu/pub/gnu
ftp://anonymous@ftp.prep.ai.mit.edu/pub/gnu
ftp://anonymous:my_password@ftp.prep.ai.mit.edu/pub/gnu
```

第一个例子没有用户名和密码组件，只有标准的scheme，host，path

第二个例子显示了一个指定为anonymous的用户名.

第三个例子，指定了用户名和密码。

### path

path说明了资源位于服务器的什么地方。通常像一个分级的文件系统路径。比如:

```
http://www.baidu.com/logo/log.png
```

这个URL中的路径为/logo/log.png

### params

params组件是URL中的名值对列表，由";"将其与URL的其余部分分隔。path为应用程序提供了访问资源所需的所有附加信息。如：ftp://prep.mit.edu/pub/gnu;type=d，其中type为参数名，d 为值。

### query

很多资源，比如数据库服务，都是可以通过进行查询来缩小所请求资源类型范围。

```http
http://www.jors.com/inventory?item=12731
```

问号右边的内容被称为query组件。

### frag

为了引用部分资源或资源的一个片段，URL支持使用frag组件来表示资源内部的片段。



## URL快捷方式

Web客户端可以理解为使用几种URL快捷方式。相对URL是在某资源内部指定一个资源的便捷缩略方式。很多浏览器还支持URL的自动扩展，也就是用户输入URL的一个关键部分，然后浏览器将其余部分填充。

### 相对URL

URL有两种方式：绝对的和相对的。相对URL是不完整的，要从相对URL中获取访问资源所需的全部信息，就必须相对于另一个，被称为base的URL进行解析。

#### 1.  base URL

将相对URL转换成绝对URL，第一步就是找到base URL。可以来自以下几个不同的地方：

- 在资源中显示提供

  有些资源会显示指定base URL。如HTML中可能包含定义了Base URL的标记`<BASE>`，通过它来转换那个HTML中的所有URL。

- 封装资源的base URL

  如果在没有显示指定Base URL的资源中发现一个相对URL，可以将它所属资源的URL作为Base。

#### 2. 解析相对引用

解析算法

1. 检查所有组件是否完整

   1. 如果所有组件都为空，则默认为Base URL；

   2. 如果scheme不为空，则是绝对URL；

   3. 如果scheme为空，就继承base url的scheme。

2. 检测user, password, host, port 组件

   1. 所有组件都为空，继承Base URL的这几个组件，然后检查path；

   2. 如果有一个不为空，就将继承的组件和相对组合成新的绝对URL。

3. 检查path

   1. 没有前导"/"的非空路径，就从在处理的路径中删除"./"和"/./"，然后将继承的组件和相对组合成新的绝对URL。

   2. 带有前导"/"的非空路径，将继承的组件和相对组合成新的绝对URL。

   3. 路径为空，就继承基础URL路径，依次检查参数组件和查询组件，没有就继承，然后将继承的组件和相对组合成新的绝对URL。

假设在`http://www.xxxx.com/tools.html`资源中有个相对路径`./hammers.html`。那么它的解析过程如下:

1. 路径为`./hammers.html`，base url 为`http://www.xxxx.com/tools.html`；

2. 路径的scheme为空，则继承base url的scheme。

3. 因为没有user, password, host, port 组件， 则继承这几个组件，检查pah；

4. 发现path没有前导"/"，就删除"./",然后将`hammers.html`和继承来的组件`http://www.xxxx.com/`合并起来，得到新的绝对URL:`http://www.xxxx.com/hammers.html`。




