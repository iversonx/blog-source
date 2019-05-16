---
title: 行为型模式——组合模式
date: 2018-10-26
categories: 设计模式
tags: 学习笔记
description: 
---

## 一、什么是组合模式

### 定义

将对象组合成树型结构以表示"部分-整体"的层次结构。使客户端可以使用统一的方式处理部分对象和整体对象。

### 应用组合模式解决问题的思路

组合模式通过引入一个抽象的组件对象，作为组合对象和叶子对象的父对象，这样就把组合对象和叶子对象统一起来，用户使用的时候，始终是在操作组件对象，而不再去区分是组合对象还是叶子对象。

### 2. 结构和说明

[结构示例图]

- Component：抽象的组件对象，定义Composite和Leaf的共同点，让客户端可以通过这个接口访问和管理整个对象结构。
- Composite：组合对象，通常会存储子组件。
- Leaf：叶子节点对象，定义和实现叶子对象的行为，不再包含其他的字节点对象。
- Client：客户端，通过组件接口来操作组合结构里面的组件对象。

##### 3. 示例代码

```java
public abstract class Component {
    /**
     * 示意方法，子组件对象可能有的功能方法
     */
    public abstract void someOperation();

    public void addChild(Component child) {
        throw new UnsupportedOperationException("对象不支持这个功能");
    }

    public void removeChild(Component child) {
        throw new UnsupportedOperationException("对象不支持这个功能");
    }

    public Component getChildren(int index) {
        throw new UnsupportedOperationException("对象不支持这个功能");
    }
}

public class Composite extends Component {
    private List<Component> childComponents = null;
    @Override
    public void someOperation() {
        if(childComponents != null) {
            for(Component c : childComponents) {
                c.someOperation();
            }
        }
    }

    public void addChild(Component child) {
        if(childComponents == null) {
            childComponents = new ArrayList<>();
        }

        childComponents.add(child);
    }

    public void removeChild(Component child) {
        if(childComponents != null) {
            childComponents.remove(child);
        }
    }

    @Override
    public Component getChildren(int index) {
        if(childComponents != null) {
            if(index >= 0 && index < childComponents.size()) {
                return childComponents.get(index);
            }
        }
        return null;
    }
}

public class Leaf extends Component {
    @Override
    public void someOperation() {

    }
}

public class Client {

    public static void main(String[] args) {
        Composite root = new Composite();
        Composite c1 = new Composite();
        Composite c2 = new Composite();

        Component leaf = new Leaf();
        Component leaf2 = new Leaf();
        Component leaf3 = new Leaf();
        Component leaf4 = new Leaf();

        root.addChild(c1);
        root.addChild(c2);

        c1.addChild(leaf);
        c1.addChild(leaf2);
        c2.addChild(leaf3);
        c2.addChild(leaf4);

        root.someOperation();
    }
}
```

#### 二、组合模式讲解

##### 1. 组合模式的目的

组合模式的目的是：让客户端不再区分操作的是组合对象还是叶子对象，而是以一个统一的方式来操作。其关键之处是设计一个抽象类的组件类，让它可以代表组合对象和叶子对象。

##### 2. 对橡树

组合模式会组合出树型结构，这个树型结构所使用的多个组件对象，就形成了对象树。这意味着，所有可以使用对象树来描述或操作的功能，都可以考虑使用组合模式。

##### 3. 最大化Component定义

在Component的定义中，为了统一组合对象和叶子对象的操作，需要定义的方法这两种对象对外方法的和。这种做法与类设计原则相冲突的，类设计有这样的原则：父类应该只定义对子类有意义的操作。

在组合模式中，Component中有些方法对于叶子对象是没有意义的；为了解决这个问题，常见做法是在Component中为某些子对象没有意义的方法提供默认的实现，或是默认抛出不支持该功能的异常。这样，子对象需要这个功能时，就要重写它。

##### 4. 安全性和透明性

安全性是指：从客户使用组合模式上看是否更安全。如果是安全的，就不会有发生误操作的可能，能访问的方法都是被支持的功能。

透明性是指：从客户使用上，是否需要区分是组合对象还是叶子对象。如果是透明的，就不用区分，对客户而言，都是组件对象，具体类型对客户是透明。

###### 透明性的实现

在Component中声明管理子组件的操作，并为这些方法提供默认的实现。透明性的实现是以安全性为代价的，因为在Component中定义的一些方法，对于叶子对象是没有意义的。而客户不知道这些区别，因此客户可能会对叶子对象调用这些方法，这样的操作是不安全的。

###### 安全性的实现

把管理子组件的操作定义在Composite中，就不会发生不安全的操作了。但是这样就牺牲了透明性，客户端需要区分对象的类型。

###### 两种实现方式的选择

组合模式更看中透明性，因为其功能是让用户使用统一的方式操作叶子对象和组合对象。

#### 三、组合模式的优缺点

###### 优点

1. 定义了包含基本对象和组合对象的类层次结构
2. 统一了组合对象和叶子对象：
3. 简化了客户端调用：客户端不需要再区分对象类型。
4. 更易于扩展：由于客户端统一面对Component来操作，因此新定义的Composite或Leaf子类能够很容易地与已有的结构一起工作，而客户端不需要进行改变。

###### 缺点

1. 很难限制组合中的组件类型

#### 四、什么时候选用组合模式

1. 如果想表示对象的部分-整体层次结构，可以使用组合模式，把整体和部分的操作统一起来，使层次结构实现更简单，从外部来使用这个层次结构也容易。
2. 如果希望统一地使用组合结构中的所有对象，可以使用组合模式。
3. 所有树型结构都可以使用组合模式。

#### 五、相关模式

- 组合模式和装饰模式
- 组合模式和享元模式
- 组合模式和迭代器模式
- 组合模式和访问者模式
- 组合模式和职责链模式
- 组合模式和命令模式
