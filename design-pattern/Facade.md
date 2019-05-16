---
title: 结构型模式--外观模式
date: 2018-10-26
categories: 设计模式
tags: 学习笔记
description: 
---

#### 一、什么是外观模式

##### 定义

为子系统中的一组接口提供一个一致的界面，Facade模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。

> 这里的界面指的时从一个组件外部来看这个组件，能够看到什么，这就是这个组件的界面；比如，从一个类外部看这个类，那么这个类的public方法就是这个类的外观。
> 
> 这里的接口指的是外部和内部交互的一个通道，通常时指一些方法，可以是类的方法，也可以是interface的方法。

##### 应用外观模式的思路

外观模式就是通过引入一个外观类，在这个类里面定义客户端想要的简单的方法，然后在这些方法的实现里面，由外观类再去分别调用内部的多个模块来实现功能，从而让客户端变得简单。这样一来，客户端只需要和外观类交互就可以了。

##### 结构和说明

[示意图]

- Facade：定义子系统的多个模块对外的高层接口，通常需要调用内部多个模块，从而把客户的请求代理给适当的子系统对象。

- 模块：接受Facade对象的委派，真正实现功能，各个模块之间可能有交互。



##### 示例代码

由于外观模式的结构图过于抽象，因此把它稍微具体点。假设子系统内有三个模块AModule、BModule、CModule

```java
public interface AModuleApi {
    public void testA();
}

public class AModule implements AModuleApi {
    public void testA() {
        System.out.println("示意方法");
    }
}

public interface BModuleApi {
    public void testB();
}

public class BModule implements BModuleApi {
    public void testB() {
        System.out.println("示意方法");
    }
}

public interface CModuleApi {
    public void testC();
}

public class CModule implements CModuleApi {
    public void testC() {
        System.out.println("示意方法");
    }
}

public class Facade {
    public void test() {
        AModuleApi a = new AModule();
        a.testA();
        BModuleApi b = new BModule();
        b.testB();
        CModuleApi c = new CModule();
        c.testC();
    }
}

public class Client{
    public static void main(String[] args) {
        Facade facade = new Facade();
        facade.test();
    }
}
```

#### 二、外观模式的一些细节

##### 外观模式的目的

外观模式的目的不是给子系统添加新的功能接口，而是为了让外部减少与子系统内部多个模块的交互，松散耦合，从而让外部能够更简单地使用子系统。

##### Facade的实现

通常可以把外观类实现为单例，也可以直接把外观中的方法实现成为静态的方法，这样就可以不需要创建外观对的实例而直接使用。

也可以把Facade实现成interface，只是这样会增加系统的复杂程度。把Facade实现成接口，有个好处是可以选择性地暴露接口的方法，尽量减少模块对子系统外提供的接口方法。

##### Facade的方法实现

Facade的方法实现中，一般是负责把客户端的请求转发给子系统内部的各个模块进行处理，Facade的方法本身并不进行功能处理。

#### 三、外观模式的优缺点

- 外观模式松散了客户端与子系统的耦合关系，让子系统内部的模块能更容易扩展和维护。

- 外观模式让子系统更加易用，客户端不再需要了解子系统内部的实现，也不需要和众多子系统内部的模块进行交互，只需和外观类交互就可以了。

- 过多的或不太合理的Facade容易让人迷惑。

#### 四、何时选用外观模式

- 希望为一个复杂的子系统提供一个简单的接口时，可以考虑使用外观模式。

- 如果想让客户程序和抽象类的实现部分松散耦合，可以考虑使用外观模式。

#### 五、相关模式

- 外观模式和中介者模式

  二者相似，但有本质的区别。中介者模式主要用来封装多个对象之间的相互的交互，多用在系统内部的多个模块之间；而外观模式封装的是单向的交互，是从客户端访问系统的调用。在中介者模式的实现里，是需要实现具体的交互功能；而外观模式的实现里面，一般是组合调用或转调内部实现的功能。

#### 六、总结

- 外观模式封装了子系统外部和子系统内部多个模块的交互过程，从而简化了外部的调用。

- 外观模式体现了“最少知识原则”。在外观模式中，客户端只需要和外观类交互，不需要去关心子系统内部模块的变动情况。
