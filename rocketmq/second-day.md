---
title: RocketMQ入门笔记(二)
date: 2018-12-20
updated: 2019-02-12
categories: RocketMQ
tags: 学习笔记
description: 这篇文章记录如何在Java中使用RocketMQ发送消息，接收消息。使用RocketMQ以三种方式发送消息:可靠的同步，可靠的异步和单向传输；使用RocketMQ接收消息。
---

## 在Java中使用RocketMQ

#### 1. 添加依赖

```bash
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-client</artifactId>
    <version>4.3.0</version>
</dependency>
```

#### 2. 可靠同步发送消息

同步发送是指消息发送方发出数据后，会在收到接收方发回响应之后才发下一个数据包的通讯方式。

可靠的同步传输用于广泛的场景，如重要的通知消息，短信通知，短信营销系统等。

```java
public class SyncProducer {
    public static void main(String[] args) throws Exception{
        DefaultMQProducer producer = new DefaultMQProducer("SyncGroup");
        // 指定NameServer地址
        producer.setNamesrvAddr("192.168.48.129:9876");
        producer.start();
        for(int i = 0; i < 100; i++) {
            User user;
            if(i % 2 == 0) {
                user = new User(i, "username" + i, "男", i);
            } else {
                user = new User(i, "username" + i, "女", i);
            }
            Message msg = new Message("TopicTest", "TagA", JSONObject.toJSONString(user).getBytes(RemotingHelper.DEFAULT_CHARSET));
            SendResult sendResult = producer.send(msg);
            System.out.printf("%s%n", sendResult);
        }
        producer.shutdown();
    }
}
```

#### 3. 可靠异步发送消息

异步发送是指发送方发出数据后，不等待接收方发回响应，接着发送下一个数据包的通讯方式。需要用户实现异步发送回调接口(SendCallback)。

常用于响应时间敏感的业务场景

```java
public static void main(String[] args) throws Exception{
   DefaultMQProducer producer = new DefaultMQProducer("AsyncGroup");

   producer.setNamesrvAddr("192.168.48.129:9876");

   producer.start();

   // producer.setRetryTimesWhenSendAsyncFailed(0);

   int messageCount = 100;

   final CountDownLatch countDownLatch = new CountDownLatch(messageCount);

   for(int i = 0; i < messageCount; i++) {

       User user;

       if(i % 2 == 0) {

           user = new User(i, "username" + i, "男", i);

       } else {

           user = new User(i, "username" + i, "女", i);

       }


       final int index = i;

       Message msg = new Message("TopicTest", "TagA",

               JSONObject.toJSONString(user).getBytes(RemotingHelper.DEFAULT_CHARSET));

       producer.send(msg, new SendCallback() {

           @Override

           public void onSuccess(SendResult sendResult) {

               System.out.printf("%-10d OK %s %n", index,

                       sendResult.getMsgId());

               countDownLatch.countDown();

           }


           @Override

           public void onException(Throwable e) {

               System.out.printf("%-10d wException %s %n", index, e);

               e.printStackTrace();

               countDownLatch.countDown();

           }

       });

   }


   countDownLatch.await();

   // 必须在消息都发送完成后才关闭，否则报错

   producer.shutdown();

}
```

#### 以单向模式发送消息

单向发送特点为发送方只负责发送消息，不等待服务器回应且没有回调函数触发，只发送 请求不等待响应。

单向传输用于需要中等可靠性的情况，例如日志收集。

```java
public static void main(String[] args) throws Exception{
        //Instantiate with a producer group name.
        DefaultMQProducer producer = new DefaultMQProducer("OneWayGroup");
        // Specify name server addresses.
        producer.setNamesrvAddr("192.168.48.129:9876");
        //Launch the instance.
        producer.start();
        for (int i = 0; i < 100; i++) {
            //Create a message instance, specifying topic, tag and message body.
            Message msg = new Message("TopicTest" /* Topic */,
                    "TagA" /* Tag */,
                    ("Hello RocketMQ " +
                            i).getBytes(RemotingHelper.DEFAULT_CHARSET) /* Message body */
            );
            //Call send message to deliver message to one of brokers.
            producer.sendOneway(msg);

        }
        //Shut down once the producer instance is not longer in use.
        producer.shutdown();
    }
```

#### 遇到的异常

```java
Exception in thread "main" org.apache.rocketmq.client.exception.MQClientException: No route info of this topic, TopicTest
```

我目前在三种情况(可能还有其他情况)下遇见这个异常:

1. broker没有正常启动

2. NamerServer地址设置错误

3. Topic没有创建
