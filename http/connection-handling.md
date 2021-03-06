---
title: HTTP 连接处理
date: 2018-12-13
categories: http
tags: 学习笔记
description: 
---

## Connection 头部

HTTP允许在客户端和源服务器之间存在HTTP中介裢,如代理，缓存等。HTTP 消息从客户端通过中间设备逐跳转发到源服务器(或反向)。

有时，两个相邻的HTTP应用程序会为它们共享的连接应用一组选项。Connection头部字段中有一个由逗号分隔的连接标签列表用来指定一些不会传播到其他连接的选项。Connection头部可以携带三种不同类型的标记：

- HTTP头部字段名，列出仅与此连接相关的头部‘；

- 任意标记值，描述此连接的非标准选项；

- 值close，表示完成后关闭持久连接。

如果连接标签中包含了一个HTTP头部字段的名称，那么这个首部字段就包含了与连接相关的一些信息，在消息转发出去前，必须删除Connection列出的所有首部字段。

```http
服务器响应给代理
HTTP/1.1 200 OK
Cache-control: max-age=3600
Connection: meter, close, bill-my-credit-card
Meter: max-user=3, max-reuses=6
```

上面例子中Connection说明，不应该转发Meter首部，要应用bill-my-credit-card选项，且本次事务结束后应该关闭持久连接。

## 串行事务处理时延

假设有一个包含了3个图片的Web页面，浏览器需要发起4个HTTP事务来显示此页面：1个用于顶层的HTML页面，3个用于嵌入的图片。如果每个事务都需要一条新连接，那么连接时延和慢启动时延就会叠加起来。

除了串行加载带来实际时延外，加载单个图片时，页面上其他部分没有动静也会让人觉得速度很慢。另一个缺点是，有些浏览器在加载完对象之前无法在屏幕上显示任何内容，因为它们需要对象的大小来决定在什么位置显示对象。

为了提供HTTP的连接性能，有以下几种方法：

- 并行连接

  通过多条TCP连接发起并发的HTTP请求

- 持久连接

  重用TCP连接，以消除连接及关闭时延

- 管道化连接

  通过共享的TCP连接发起并发的HTTP请求

- 复用的连接

  交替传送请求和响应报文

## 并行连接

HTTP允许客户端打卡多条连接，并行地执行多个HTTP事务。通过并行连接克服单条连接的空载时间和带宽限制，加载数度也会有所提高。

并行连接的速度并不一定总是更快。当客户端的网络带宽不足时，大部分的时间可能是用来传送数据的。这种情况下，一个连接到速度较快服务器上的HTTP事务就会很容易耗尽所有可用的Modem带宽。如果并行加载多个对象，每个对象都会竞争这个有限的带宽，每个对象都会以较慢的速度按比例加载，这样带来的性能提升就很小。

打开大量连接会消耗很对内存资源，从而引发自身的性能问题。复杂的Web页面可能会有数十或数百个对象，客户端可能可以打开数百连接。每个用户打开100个连接，100个用户同时发出申请，服务器就要负责处理10000个连接，这会造成服务器性能的严重下降。

在实践中，浏览器使用并行连接，但它们将并行连接的总数限制为少量（通常为四个）。 服务器可以自由地关闭来自特定客户端的过多连接。

## 持久连接

HTTP/1.0以及HTTP/1.0的各种增强版本允许HTTP设备在事务chuli结束后将TCP连接保持在打开状态，以便为未来的HTTP请求重用现存的连接，指导客户端或服务器决定将其关闭为止，这样的连接被称为持久连接。

重用持久连接，可以避开缓慢的连接建立阶段，而且已经打开的连接还可以避免慢启动的拥塞适应阶段，以便更快速地进行数据的传输。

持久连接有两种类型：比较老的HTTP/1.0 + "keep-alive"连接，以及HTTP/1.1 "persistent" 连接。

### HTTP/1.0 keep-alive连接

实现keep-alive连接的客户端可以通过包含`Connection: Keep-Alive`头部请求将一条连接保持在打开状态。如果服务器愿意为下一条请求将连接保持在打开状态，就在响应中包含相同的头部。如果响应中没`Connection: Keep-Alive`，客户端就认为服务器不支持keep-alive，会在收到响应消息后关闭连接。客户端和服务器可以在任意时刻关闭空闲的keep-alive连接，并可随意限制keep-alive连接锁处理事务的数量。

可以用Keep-Alive通用头部中指定的、由逗号分隔的选项来调节Keep-alive的行为:

- `timeout`参数是在`Keep-alive`响应头中发送。它估计服务器可能保持连接活跃的时间。

- `max`是在`Keep-alive`响应头中发送。它估计服务器可能保持连接活动的HTTP事务数量。

- `Keep-alive`还支持任意未处理的属性，主要用于诊断和调试目的。语法是name=value。

`Keep-alive`是可选的，但只有在`Connection: Keep-Alive`也存在时才能允许。下面是例子说明服务器还会为另外5个事务保持连接，或直到它空闲了两分钟。

```http
Connection: Keep-Alive
Keep-alive: max=5, timeout=120
```

#### Keep-Alive连接的限制和规则

- 默认情况下，在HTTP/1.0中不会发生保持活动状态。客户端必须发送`Connection: Keep-Alive`请求头以激活保持活动连接。

- `Connection: Keep-Alive`必须与要继续保持持久连接的报文一起发送。如果客户端没有发送`Connection: Keep-Alive`，服务器就会在那条请求之后关闭连接。

- 通过检测`Connection: Kepp-Alive`响应头的缺少，客户端可以判断服务器是否在响应后关闭连接。

- 只有在不检查连接关闭的情况下确定消息的实体主体的长度时，才能保持连接打开，这意味着实体主体必须具有正确的Content-Type，媒体类型，或使用chunkedtransfer编码进行编码。在一条keep-alive的通道上发送错误的Content-Length是不好的，因为事务的另一端将无法准确地检测到一条消息的结束和另一条消息的开始。

- 代理和网关必须执行Connection首部的规则。代理或网关必须在将报文转发出去或将其高速缓存之前，删除在Connection头部中命名的所有头部字段以及Connection自身。

- 严格来说，不应该与无法确定是否支持Connection首部的代理服务器建立Keep-alive连接，以防止出现哑代理问题。

- 应该忽略所有来自HTTP/1.0设备的 Connection字段，因为它们可能是由比较老的代理服务器转发的。

- 除非重复发送请求会产生其他副作用，否则如果在客户端收到完整的响应前连接就关闭了，客户端一定要做好重试的准备。

`Connection: Keep-alive`头部应该只对这条离开客户端的TCP连接产生影响。

#### 哑代理

很多简单的代理都是盲中继(blind relay)，它们只是将字节从一个连接转发到另一个连接中，不对`Connection`进行特殊处理。

假设一个Web客户端正通过一个作为盲中继使用的哑代理与Web服务器进行对话。

[缺少图片]

1. 客户端向代理发送一条消息，其中包含了`Connection: Keep-alive`，客户端等待响应，以确定对方是否认可它对keep-alive信道的请求。

2. 哑代理收到这条HTTP请求，但不理解`Connection`，不知道keep-alive是什么，只是将消息完整地发送给服务器。

3. 服务器收到HTTP请求，服务器看到`Connection: Keep-Alive`，以为代理希望进行keep-alive对话，并回送一个`Connection: Keep-Alive`响应头。此时服务器认为他与代理在进行keep-alive对话，服务器并遵循keep-alive规则。

4. 代理将Web服务器的响应消息回送给客户端，并将服务器的`Connection: Keep-Alive`一起传送过去。客户端看到`Connection: Keep-Alive`，就认为代理同意进行`Keep-alive`对话。此时客户端和服务器都以为它们在进行keep-alive对话。

5. 代理将响应回送给客户端后，等待服务器关闭连接。但服务器以为是在进行keep-alive对话，因此并不会关闭连接。这样代理就会挂在那里等待连接的关闭。

6. 客户端收到响应后，会立即转向下一条请求，在keep-alive连接上向代理发送另一条请求。而代理认为同一条连接上不会有其他请求到来，因此请求被忽略。这种错误会使浏览器一直处于挂起状态。指导客户端或服务器将连接超时，并关闭为止。

为避免此类代理问题的发生，现代代理都不能转发Connection头和所有名字出现在Connection值中的头。另外，还有不能作为Connection值列出，也不能被转发或作为缓存响应使用的头。其中包括`Proxy-Authenticate`、`Proxy-Connection`、`Transfer-Encoding`和`Upgrade`

#### HTTP/1.1 persistent connection

HTTP/1.1停止了对keep-alive连接的支持，使用persistent connection的改进型设计取代keep-alive。HTTP/1.1 持久连接在默认情况下是激活的。要在事务处理结束之后将连接关闭，HTTP/1.1应用程序必须向报文中显示地添加`Connection: close`。不发送`Connection: close`并不意味着服务器承诺永远将连接保持在打开状态，客户端和服务器仍然可以随时关闭空闲的连接。

#### 持久连接的限制和规则

- 发送了Connection: close请求首部之后，客户端就无法在那条连接上发送请求。

- 如果客户端不想在连接上发送其他请求了，就应该在最后一条请求中发送Connection: close

- 只有当连接上的所有消息都具有正确的，自定义的消息长度时，连接才能保持持久，即实体主体必须具有正确的Content-Length或者使用分块传输编码进行编码。

- HTTP/1.1的代理必须能够分别管理与客户端和服务器的持久连接——每个持久连接都只适用于一跳传输。

- HTTP/1.1的代理服务器不应该与HTTP/1.0客户端建立持久连接，除非它们了解客户端的处理能力。

- 尽管服务器不应该试图在传输过程中关闭连接，而且在关闭连接之前至少响应一条请求，不管Connection取什么值，HTTP/1.1设备都可以随时关闭连接。

- 除非重发会产生副作用，否则如果在客户端收到整条响应之前连接关闭了，客户端必须重新发起请求。

- 一个用户客户端对任何服务器或代理最多只能维护6条持久连接，以防服务器过载。

## 管道化连接

HTTP/1.1允许在持久连接上可使用管道请求。在响应到达之前，可以将多条请求放入队列。当第一条请求通过网络流向服务器时，第二条和第三条请求也可以开始发送了。在高时延网络条件下，这样做可以降低网络的环回时间，提高性能。

对管道化连接的几条限制

- 如果HTTP客户端无法确认连接是持久的，就不应该使用管道。

- 必须按照与请求相同的顺序回送HTTP响应。

- HTTP客户端必须做好连接会随时关闭的准备，还有准备好重复所有未完成的管道化请求。

- HTTP客户端不应该用管道化的方式发送会产生副作用的请求。
