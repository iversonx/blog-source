---
title: MapperProxy是如何生成的
date: 2018-10-26
categories: mybatis
description: 
---

在使用Mybatis时，我们通常只要声明Mapper接口和Mapper映射文件，并将二者关联起来，就可以通过Mapper接口对数据库进行操作。



mybatis是通过使用动态代理来实现对Mapper接口的执行，涉及的对象有：

- MapperProxy：动态代理类；实现`InvocationHandler`接口，对Mapper接口进行动态代理

- MapperProxyFactory：MapperProxy工厂类，为Mapper接口的Class实例创建对应的MapperProxy代理对象。

- MapperRegistry：Mapper注册器，为每一个Mapper接口的Class实例都关联一个MapperProxyFactory对象。

- MapperMethod：记录Mapper方法的信息。在方法第一次被调用时创建实例，之后每次都从缓存里面获取。






