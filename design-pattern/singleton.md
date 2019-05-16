---
title: 创建型模式 -- 单例模式
date: 2018-10-25
categories: 设计模式
tags: 学习笔记
description: 
---

#### 一、什么是单例模式

##### 定义

保证一个类仅有一个实例，并提供一个访问它的全局访问点。

##### 应用单例模式的思路

要想一个类只有一个实例，那么就要把创建实例的权限收回，只有类自身可以创建自己的实例，然后由这个类提供外部可以访问这个类实例的方法。

##### 结构说明

[示意图]

- Singleton：单例类，负责创建Singleton类自己的唯一实例，并提供一个公共的方法让外部来访问这个类的唯一实例。

##### 示例代码

```java
/**
 * 懒汉式单例
 */
public class Singleton {
    private static Singleton instance;
    private Singleton() {
    }

    public static synchronized Singleton getInstance() {
        if(instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

/**
 * 饿汉式单例
 */
public class Singleton {
    private static Singleton instance = new Singleton();
    private Singleton() {
    }

    public static synchronized Singleton getInstance() {
        return instance;
    }
}
```

- 饿汉式指的是在类加载的时候就创建对象实例。
- 懒汉式指的是在第一次使用的时候才创建对象实例。

##### 单例的范围

目前Java里面实现的单例是一个虚拟机的范围。装载类的功能是虚拟机的，一个虚拟机在通过自己的ClassLoader装载饿汉式实现的单例类时就会创建一个类实例。如果一个虚拟机有多个ClassLoader，并且都装载单例类时，也是产生多个实例。

##### 单例模式的实现方式

###### 懒汉式

由于示例代码中已经展示了懒汉式和饿汉式，这里就只说明步骤

1. 私有化构造方法
2. 定义存储实例的属性
3. 把这个属性定义成静态的
4. 提供获取实例的静态方法
5. 在方法里实现控制实例的创建

懒汉式是利用时间换空间，每次获取实例都会进行判断是否需要创建实例；如果一直没有人使用的话，就不会创建实例，则节约内存空间。

不加synchronized的懒汉式是线程不安全的。

###### 饿汉式

和懒汉式相比，区别在于如何实现获取实例的方法。饿汉式在声明属性的时候，就创建实例，在静态方法中直接返回属性。

饿汉式是采用了空间换时间。在类装载的时候就会创建类实例，不管是否使用，然后每次调用时就不需要判断，节省了时间。

饿汉式是线程安全的，因为虚拟机保证只会装载一次，在装载类的时候不会发生并发。

###### 使用静态内部类实现单例模式

```java
public class Singleton {
    private static class SingletonHolder {
        private static Singleton instance = new Singleton();
    }

    private Singleton() {
    }

    public static Singleton getInstance() {
        return SingletonHolder.instance;
    }
}
```

当getInstance方法第一次被调用时，它第一次读取SingletonHolder.instance，导致SingletonHolder类得到初始化；而这个类在装载并被初始化时，会初始化它的静态域，从而创建Singleton的实例，由于是静态域，因此只会在虚拟机加载类的时候初始化一次，并有虚拟机保证它的线程安全性。

这种方式的优势在于，getInstance方法并没有被同步，并且只是执行一个域的访问，因此延迟初始化并没有增加任何访问成本。

###### 使用枚举实现单例模式

单元素的枚举类型已经成为实现单例的最佳方法。

```java
public enum Singleton {
    INSTANCE;
    public void operation() {
        // 其他操作    
    }
}
```

使用枚举来实现单例会根据简洁，而且无偿地提供了序列化的机制，并由JVM从根本上提供保障，绝对防止多次实例化，是更简洁，高效，安全的实现方式。

#### 二、何时选用单例模式

当需要控制一个类的实例只能有一个，而且客户只能从一个全局访问点访问它时，可以使用单例模式。
