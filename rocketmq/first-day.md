---
title: RocketMQ入门笔记(一)
date: 2018-12-19
updated: 2019-01-22
categories: RocketMQ
tags: 学习笔记
description: 这篇文章主要记录如何下载安装RocketMQ以及RocketMQ中涉及角色的简单介绍。
---

## 消息队列功能介绍

消息队列是一种进程间通信或同一进程的不同线程间的通信方式。消息队列可以提供应用解耦、流量消峰、消息分发等功能，成为互联网服务架构里标配的中间件。

### 应用解耦

通过将系统要处理的内容缓存到消息队列中，这样当某个子系统出现暂时不可用时，依赖这个系统的操作可以继续完成操作，只是这一个系统暂时不能处理操作的内容；待到系统可用时，可以从消息队列里获取未处理的操作内容。

### 流量消峰

当系统流量瞬间猛增时，如果没有一个缓冲机制，系统会出现崩溃现象。通过使用消息队列，把大量的请求暂存起来，分散到相对长的一段时间内处理，能大大提高系统的稳定性和用户体验。

## RocketMQ

RocketMQ是阿里开源的消息中间件，RocketMQ基于长轮询的拉取方式，解决了事务消息，顺序消息和海量堆积的问题。此外RocketMQ是采用Java语言进行开发的，比起Kafka的Scala语言和RabbitMQ的Erlang语言，更容易进行定制开发。

### 下载、安装和配置

Binary版是一些编译好的jar和辅助的shell脚本，可从官网下载http://rocketmq.apache.org/dowloading/releases/ 也可以下载源码自己编译https://github.com/apache/rocketmq 。

系统要求是64位的Linux、Unix或Max，Java版本要大于等于JDK1.8。如果自己编译的话，需要安装maven 3.2.x和Git。

下载完Binary版本后需要进行解压:

```bash
unzip rocketmq-all-4.3.2-bin-release.zip -d ./rocketmq-all-4.3.2-bin
```

解压后目录包含:

- LICENSE 、NOTICE、README.md 包括一些版权声明和功能说明信息；

- benchmark  包括允许benchmark程序的shell脚本

- bin  包含各种使用RocketMQ的shell脚本和cmd脚本

- conf  示例配置文件

- lib RocketMQ 各个模块的jar包，以及依赖jar包

#### 启动命令

先启动NameServer，再启动Broker。

```bash
# 启动NameServer
nohup sh $RocketMQ_HOME/bin/mqnamesrv &
# 日志查看
tail -f ~/logs/rocketmqlogs/namesrv.log

# 启动Broker
nohup sh $RocketMQ-HOME/bin/mqbroker [-n localhost:9876]|[-c 配置文件的路径]
# 日志查看
tail -f ~/logs/rocketmqlogs/broker.log
```

#### 关闭命令

```bash
sh $RocketMQ-HOME/bin/mqshutdown broker
sh $RocketMQ-HOME/bin/myshutdown namesrv
```

## RocketMQ角色简单介绍

### Name Server

Name Server是路由信息的提供者，Broker启动时向Name Server发送注册请求，Name Server接收Broker的请求注册路由信息，保存Broker列表。Producer/Consumer使用Topic查找相应的Broker列表。

### Broker

Broker是RocketMQ系统的主要组件。它接收从Producer发送的消息，存储这些消息并准备处理来自Consumer的拉取请求。还存储与消息相关的元数据，包括消费组，消耗进度offset和主题/队列信息。

### Producer

Producer将业务应用系统生成的消息发送给Broker。RocketMQ提供多种发送范例：synchronous, asynchronous and one-way

### Producer Group

具有相同角色的Producer组合在一起。如果原始生产者在事务之后崩溃，则Broker可以联系同一生产组的不同生产者实例以提交或回滚事务。

### Consumer

消费者从Broker那里获取消息并将其提供给应用程序。从用户应用角度看，提供了两种类型的消费者:

#### Push Consumer

Push Consumer封装了消息提取的过程，并注册MessageListener回调接口，获得消息后，唤醒MessageListener来消费。

#### Pull Consumer

Pull Consumer主动从Broker那里获取消息。获取消息的过程需要用户编写，首先通过打算消费的Topic拿到MessageQueue的集合，遍历MessageQueue集合，然后针对每个MessageQueue批量获取消息。

### Consumer Group

完全相同角色的Consumer组合在一起并命名的Consumer组。通过Consumer组可以非常容易在消息消费方面实现负载均衡和容错。Consumer组的Consumer实例必须具有完全相同的主体订阅。

### Topic

Topic是一个Producer传递消息和Consumer提取消息的类别。Topic与Producer和Consumer的关系非常松散。一个Topic可能有0个或者多个Producer向它发送消息;Producer可以发送不同Topic的消息。从Consumer的角度来看，Topic可以由0个或多个Consumer组订阅。Consumer组可以订阅一个或多个Topic，只有该组的实例保持其订阅一致即可。

### Message

消息是要传递的信息。消息必须有一个Topic，可以理解为发送邮件的目标地址。消息还可以具有Tag和额外的key-value。

### Tag

Tag相当于子Topic，为用户提供了额外的灵活性。来自同一业务模块的具有不同目的的消息可以具有相同的 Topic和不同的Tag。Tag有助于保持代码的清晰和连贯，同时也方便查询。
