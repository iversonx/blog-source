---
title: 访问者模式(Bridge)
date: 2018-10-26
categories: 设计模式
tags: 学习笔记
description: 
---
#### 一、什么是访问者模式
##### 定义
表示一个作用于某对象结构中的各元素的操作。它使你可以在不改变各元素的类的前提下定义作用域这些元素的新操作。

##### 应用访问者模式的思路
对于某个对象结构，不想改变类，又要添加新的功能，很明显就需要一种动态的方式，在运行期间把功能动态地添加到对象结构中去。

使用访问者模式解决这个问题的实现思路：

- 首先定义一个接口来代表要新加入的功能，为了通用，定一个通用的功能方法来代表新加入的功能。
- 在对象结构上添加一个方法，作为通用的功能方法，也就是可以代表被添加的功能，在这个方法中传入具体的实现新功能的对象。
- 在对象结构的具体实现对象中实现这个方法，回调传入具体的实现新功能的对象，就相当于调用到新功能上了。
- 接下来的步骤就是提供实现新功能的对象。
- 最后再提供一个能够循环访问整个对象结构的类，让这个类来提供符合客户端业务需求的方法，来满足客户端调用的需要。
- 这样一来，只有提供实现新功能的对象给对象结构，就可以为这些对象添加新的功能，由于在对象结构中定义的方法是 通用的功能方法，所以什么新功能都可以加入。

##### 结构说明
[示意图]

- Visitor：访问者接口，为所有的访问者对象声明一个visit方法，用来代表为对象结构添加的功能，理论上可以代表任意功能。
- ConcreteVisitor：具体的访问者实现对象，实现要真正被添加到对象结构中的功能。
- Element：抽象的元素对象，对象结构的顶层接口，定义接收访问的操作。
- ConcreteElement：具体元素对象，对象结构中具体的需i，也是被访问的对象，通常会回调访问者的真实功能，同时开发自身的数据供访问者使用。
- ObjectStructure：对象结构，通常包含多个被访问的对象，它可以遍历多个被访问对象，也可以让访问者访问它的元素。

##### 示例代码
```java
/**
 * 访问者接口，为所有的访问者对象声明visit方法，用来代表为对象接* * 口添加的功能，可以代表任意的功能
 */
public interface Visitor {
    void visitA(ConcreteElementA element);
    void visitB(ConcreteElementB element);
}

/**
 * 具体的访问者对象，实现要真正被添加到对象结构中的功能。
 */
public class ConcreteVisitor implements Visitor {
    @Override
    public void visitA(ConcreteElementA element) {
        element.opertionA();
    }

    @Override
    public void visitB(ConcreteElementB element) {
        element.opertionB();
    }
}

/**
 * 抽象的元素对象，对象结构的顶层接口，定义接受访问的操作。
 */
public abstract class Element {
    /**
     * 接受访问者的访问
     * @param visitor 访问者对象
     */
    public abstract void accept(Visitor visitor);
}

/**
 * 具体元素对象，对象结构中具体的对象，也是被访问的对象，
 * 通常会回调访问者的真实功能，同事开发自身的数据供访问者使用。
 */
public class ConcreteElementA extends Element{
    @Override
    public void accept(Visitor visitor) {
        visitor.visitA(this);
    }

    public void opertionA() {
        System.out.println("Opertion A");
    }
}

/**
 * 具体元素对象，对象结构中具体的对象，也是被访问的对象，
 * 通常会回调访问者的真实功能，同事开发自身的数据供访问者使用。
 */
public class ConcreteElementB extends Element{
    @Override
    public void accept(Visitor visitor) {
        visitor.visitB(this);
    }

    public void opertionB() {
        System.out.println("Opertion B");
    }
}

/**
 * 对象结构，通常包含多个被访问的对象，它可以遍历多个被访问的对象，也可以让访问者访问它的元素。
 * 可以是一个复合或一个集合
 */
public class ObjectStructure {
    private List<Element> list = new ArrayList<>();

    public void handleRequest(Visitor visitor) {
        for(Element element : list) {
            element.accept(visitor);
        }
    }

    public void addElement(Element element) {
        this.list.add(element);
    }
}

public class Client {
    public static void main(String[] args) {
        ObjectStructure os = new ObjectStructure();

        Element a = new ConcreteElementA();
        Element b = new ConcreteElementB();
        os.addElement(a);
        os.addElement(b);
        Visitor visitor = new ConcreteVisitor();
        os.handleRequest(visitor);
    }
}
```

##### 访问者的功能
访问者模式能给一系列对象透明地添加新功能，从而避免在维护期间对这一系列对象进行修改，而且还能变相实现复用访问者所具有的功能。

##### 两次分发
访问者模式能够实现在不改变对象结构的情况下，就可以给对象结构中的类增加功能，实现这个效果所使用的核心技术就是两次分发的技术。

在访问者模式中，当客户端调用ObjectStructure的时候，会遍历ObjectStructure中所有的元素，调用这些元素的accept方法，让这些元素来接受访问，这是请求的第一次分发；在具体的元素对象中实现accept方法的时候，会调用访问者的visit方法，等于请求被第二次分发了，请求被分发给访问者来进行处理，真正实现功能的正是访问者的visit方法。

两次分发技术使得客户端的请求不再被静态地绑定在元素对象上，这个时候真正执行什么样的功能同事取决于访问者类型和元素类型。

两次分发还有一个有点，就是可以在程序运行期间进行动态的功能组装和切换，只需在客户端调用时，组合使用不同的访问者对象实例即可。



#### 二、访问者模式的优缺点
##### 优点


#### 三、何时选用访问者模式
