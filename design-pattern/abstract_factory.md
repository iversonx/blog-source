---
title: 创建型模式--抽象工厂模式
date: 2018-10-26
categories: 设计模式
tags: 学习笔记
description: 提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。
---

#### 一、什么是抽象工厂模式

##### 1. 定义

提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。

##### 2. 应用抽象工厂模式的思路

首先定义一个抽象工厂，在里面定义创建一系列相关或相互依赖的对象的抽象方法，然后由具体的抽象工厂子类来提供者一系列对象的创建。这样一来可以为同一个抽象工厂提供很多不同的实现，那么创建的这一系列对象也就不一样了。

抽象工厂在这里起到了约束的作用，并提供所有子类的一个统一外观，来让客户端使用。

##### 3. 结构和说明

[示意图]

- Abstract Factory：抽象工厂，定义创建一系列产品对象的操作接口。

- Concrete Factory：具体工厂，实现抽象工厂定义的方法，具体实现一系列对象的创建。

- Abstract Product：定义一类产品对象的接口。

- Concrete Product：具体产品对象。

- Client：客户端，主要使用抽象工厂来获取一系列所需要的产品对象，然后面向这些产品对象的接口编程。

##### 4. 示例代码

```java
public interface AbstractProductA {
}

public interface AbstractProductB {
}

public class ProductA1 implements AbstractProductA {
}

public class ProductA2 implements AbstractProductA {
}


public class ProductB1 implements AbstractProductB {
}

public class ProductB2 implements AbstractProductB {
}

public interface AbstractFactory {
    AbstractProductA createProductA();
    AbstractProductB createProductB();
}

public class ConcreteFactory1 implements AbstractFactory{
    @Override
    public AbstractProductA createProductA() {
        return new ProductA1();
    }

    @Override
    public AbstractProductB createProductB() {
        return new ProductB1();
    }
}

public class ConcreteFactory2  implements AbstractFactory{
    @Override
    public AbstractProductA createProductA() {
        return new ProductA2();
    }

    @Override
    public AbstractProductB createProductB() {
        return new ProductB2();
    }
}
```

#### 二、抽象工厂模式的一些细节

##### 1. 抽象工厂模式的功能

抽象工厂模式的功能是为一系列相关对象或相互依赖的对象创建一个接口。

##### 2. 实现成接口

通常情况下我们把AbstractFactory实现成接口，而不是抽象类。

##### 3. 使用了工厂方法

AbstractFactory定义的创建产品的方法可以看成是工厂方法，而这些工厂方法的具体实现就延迟到了具体的工厂里面。

#### 三、抽象工厂模式的优缺点

##### 优点

- 分离接口和实现

  客户端使用抽象工厂来创建需要的对象，而客户端根本不知道具体的实现是谁，客户端只是面向产品的接口编程，从而分离了接口和实现。

- 使得切换产品簇变得容易

  客户端选用不同的工厂实现，就相当于是在切换不同的产品簇。

##### 缺点

- 不容易添加新的产品

  如果需要给产品簇添加一个新的产品，就需要修改抽象工厂，这样会导致修改所有的工厂实现类。

- 容易造成类层次复杂

  在使用抽象工厂模式时，如果需要选择的层次过多，那么会造成整个类层次变的复杂。

#### 四、何时选用抽象工厂模式

- 如果希望一个系统独立于它的产品的创建、组合和表示时，可以选择抽象工厂模式。

- 如果一个系统要由多个产品系列中的一个来配置的时候，可以使用抽象工厂模式。

- 如果要强调一系列相关产品的接口，以便联合使用这些产品接口时，可以使用抽象工厂模式。

#### 五、总结

工厂方法是选择单个产品的实现，虽然一个类里可以有多个工厂方法，但这些方法之间是没有联系的。抽象工厂的本质就是为一个产品簇选择实现，定义在抽象工厂里面的方法通常是有联系的，它们都是产品的某一个部分或是相互依赖的。

#### 六、相关模式

- 抽象工厂模式和工厂方法模式

  这两个模式既有区别，又有联系，可以组合使用。区别是工厂方法模式是针对单个产品对象的创建，而抽象工厂模式注重产品簇对象的创建。如果把抽象工厂创建的产品簇简化到只有一个产品，那么抽象工厂和工厂方法是差不多的。在抽象工厂的实现中，还可以使用工厂方法来提供抽象工厂的具体实现，也就是说它们可以组合使用。
