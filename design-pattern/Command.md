---
title: 行为型模式——命令模式
date: 2018-10-26
categories: 设计模式
tags: 学习笔记
description: 
---
#### 一、什么是命令模式(What)
##### 1. 定义
将一个请求封装为一个对象，从而使你可用不同的请求对客户进行参数化，对请求排队或记录请求日志，以及支持可撤销的操作。

##### 命令模式的结构和说明
[示例图]
- Command：定义命令的接口，声明执行的方法。
- ConcreteCommand：命令接口实现对象；通常持有Receiver，并调用Receiver的功能来完成要执行的操作。
- Receiver: 接收者，真正执行命令的对象。任何类都可能成为接收者
- Invoker：要求命令对象执行请求，通常持有命令对象，可以持有很多的命令对象。
- Client：创建具体的命令对象，并设置命令对象的接收者。

```java
/**
 * 命令接口
 */
public interface Command {
    void execute();
}

/**
 * 具体的命令实现对象
 */
public class ConcreteCommand implements Command {
    /**
     * 持有相应的接收者对象
     */
    private Receiver receiver;
    // 示例：可以有自己的状态
    private String state;

    public ConcreteCommand(Receiver receiver) {
        this.receiver = receiver;
    }
    @Override
    public void execute() {
        // 转调接收者对象的方法，让接收者来真正的执行功能
        receiver.action();
    }
}

/**
 * 调用者
 */
public class Invoker {
    private Command command;

    public void setCommand(Command command) {
        this.command = command;
    }

    public void rumCommand() {
        command.execute();
    }
}

/**
 * 接收者对象
 */
public class Receiver {
    /**
     * 真正执行命令相应的操作
     */
    public void action() {

    }
}

public class Client {
    public void assemble() {
        // 创建接收者
        Receiver receiver = new Receiver();
        // 创建命令对象，设定它的接收者
        Command command = new ConcreteCommand(receiver);
        // 创建Invoker, 把命令对象设置进去
        Invoker invoker = new Invoker();
        invoker.setCommand(command);
        // 执行命令
        invoker.rumCommand();
    }
}
```

在命令模式中，会定一个命令的接口，用来约束所有的命令对象，然后提供具体的命令实现，每个命令实现对象是对客户端某个请求的封装，因此如果客户端有多个的请求，就会有多个命令实现对象。

命令模式的关键就是把请求封装成对象，并定义统一的执行操作的接口，整个命令模式都是围绕这个对象进行的，这个命令对象可以被存储、转发、记录、处理、撤销等。

命令模式的接收者可以是任意的类，一个接收者对象可以处理对个命令。在标准的命令模式里，命令对象是调用接收者来完成真正的操作。

如果不需要接收者，在命令对象里完成真正的操作，这种情况就成为智能命令。

使用命令模式的过程分成两个阶段，一个阶段是组装命令对象和接收者对象的过程；另一个是触发调用 Incoker，来让命令真正执行的过程。
[调用顺序示意图]
[调用顺序示意图]

#### 二、如何使用命令模式

#### 三、命令模式的优缺点

#### 四、什么时候选用命令模式

#### 五、相关模式