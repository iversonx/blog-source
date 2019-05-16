---
title: 创建型模式--原型模式
date: 2018-10-26
categories: 设计模式
tags: 学习笔记
description: 
---

#### 一、什么是原型模式

##### 1. 定义

用原型实例指定创建对象的种类，并通过拷贝这些原型创建新的对象。

##### 2. 应用构建者模式解决问题的思路

原型模式会要求对象实现一个可以“克隆”自身的接口，这样就可以通过拷贝或者是克隆一个实例对象本身来创建一个新的实例。通过原型实例创建新的对象，就不再需要关系这个实例本身的类型，也不需要关系它的具体实现，只要它实现了克隆自身的方法，就可以通过这个方法来获取新对象，而无须通过new来创建。

##### 3. 结构和说明

[示意图]

Prototype：声明一个克隆自身的接口，用来约束想要克隆自己的类，要求它们都要实现这里定义克隆的方法。

ConcretePrototype：实现Prototype接口，实现克隆自身的功能。

Client：使用原型的客户端，首先要获取到原型实例对象，然后通过原型实例克隆自身来创建新的对象实例。

##### 4. 示例代码

```java
public interface Prototype {
    Prototype clone();
}

public class ConcretePrototype implements Prototype {
    private String name;
    private int age;
    public Prototype clone() {
        ConcretePrototype p = new ConcretePrototype();
        p.setName(this.name);
        p.setAge(this.age);
        return p;
    }

    public String getName() {
        return this.name;
    }

    public void setName(String name) {
        this.name = name;
    }
    public int getAge() {
        return this.age;
    }
    public void setAge(intage) {
        this.age= age;
    }
}

public class Client {
    public static void main(String[] args) {
        // 原型实例
        Prototype p = new ConcretePrototype();
        // 通过原型克隆出来的实例
        Prototype c = p.clone();
    }
}
```

#### 二、原型模式的一些细节

##### 原型模式的功能

- 通过克隆来创建新的对象实例；

- 为克隆出来的新的对象实例复制原型实例属性的值。

##### Java中的克隆方法

在Java中已经提供了clone方法，定义在Object类中。使用Java中的克隆方法实现原型模式时，先让原型类实现`java.lang.Cloneable`接口，这个接口没有需要实现的方法，是一个标识接口；接着重写Object的clone方法。

```java
public class Prototype implements Cloneable {
    public Object clone() {
        Prototype type = null;
        try {
            obj = (Prototype)super.clone();
        } catch(Exception e) {
            e.printStackTrace();
        }
        return obj;
    }
}
```

##### 浅克隆和深克隆

- 浅克隆：只负责克隆按值传递的数据(如基本类型、String类型)

- 深克隆：除了浅克隆要克隆的值外，还负责克隆引用类型的数据，基本上就原型实例所有的属性数据都会被克隆出来。如果原型对象的属性是引用类型，则需要一直递归地克隆下去。

#### 三、原型模式的优缺点

- 对客户端隐藏具体的实现类型

  原型模式的客户端只知道原型接口的类型，并不知道具体的实现类型，从而减少了客户端对具体实现的依赖。

- 原型模式的缺点在于每个原型的子类都必须实现clone的操作，尤其在包含引用类的对象时，必须要能递归地让所有的相关对象都要正确地实现克隆。

#### 四、何时选用原型模式

- 如果需要根据某个对象来创建更多的相同类型的对象时，可以选用原型模式。

#### 五、总结

- 原型模式的本质是克隆生成对象，是创建对象型的模式。通过原型模式克隆的对象，其属性的值是否一定要和原型对象属性值完全一样，是没有强制规定的，不过大多数实现中，克隆出来的对象和原型对象的属性值是一样的。
