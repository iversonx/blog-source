---
title: 观察者模式
date: 2018-10-26
categories: 设计模式
tags: 学习笔记
description: 
---
#### 一、什么是观察者模式
##### 1. 定义
定义对象间的一种一对多的依赖关系。当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新。

##### 2. 结构和说明
[结构示例图]

- Subject：目标对象，通常具有如下功能
	- 一个目标可以被多个观察者观察
	- 目标提供对观察者注册和退订的维护
	- 当目标的状态发生变化时，目标负责通知所有注册的，有效的观察者。
- Observer：定义观察者的接口，提供目标通知时对应的更新方法，这个更新方法进行相应的业务处理，可以在这个方法里面回调目标对象，以获取目标对象的数据。
- ConcreteSubject：具体的目标实现对象，用来维护目标状态，当目标对象的状态发生改变时，通知所有注册的、有效的观察者，让观察者执行相应的处理。
- ConcreteObserver：观察者的具体实现对象，用来接收目标的通知，并进行相应的后续处理。

##### 3. 示例代码
```java
public interface Observer {
    void update(Subject subject);
}

/**
 * 目标对象
 */
public class Subject {
    private List<Observer> observers = new ArrayList<>();

    /**
     * 注册观察者对象
     * @param observer 观察者对象
     */
    public void attach(Observer observer) {
        observers.add(observer);
    }

    /**
     * 删除观察者对象
     * @param observer 观察者对象
     */
    public void detach(Observer observer) {
        observers.remove(observer);
    }

    protected void notifyObservers() {
        for(Observer observer : observers) {
            observer.update(this);
        }
    }
}

/**
 * 具体目标对象
 */
public class ConcreteSubject extends Subject {
    private String subjectState;
    public String getSubjectState() {
        return subjectState;
    }

    public void setSubjectState(String subjectState) {
        this.subjectState = subjectState;
        this.notifyObservers();
    }
}

public class ConcreteObserver implements Observer {
    private String observerState;
    @Override
    public void update(Subject subject) {
        observerState = ((ConcreteSubject)subject).getSubjectState();

    }
}
```

##### 4. 目标和观察者之间的关系
按照模式定义，目标和观察者之间是一对多的关系。

如果观察者只有一个，也是可以的，这样目标和观察者就是1对1的关系，所以处理一个对象的状态变化会影响到另一个对象时，也可以使用观察者模式。

一个观察者可以观察多个目标，如果观察多个目标且只定义一个update方法，就会很麻烦，需要在update方法里根据目标进行不同的后续操作。

当观察者观察多目标时，应该为不同的目标定义不同的回调方法。

##### 5. 实现说明
1. 具体的目标对象要能维护观察者的注册信息，最简单就是采用一个集合来保存观察者的注册信息。
2. 具体的目标对象需要维护引起通知的状态，一般情况下是目标自身的状态。
3. 观察者对象需要能接收目标的通知，能够接收目标传递的数据，或者是能够主动去获取目标的数据，并进行后续处理。
4. 命名建议: 目标接口的定义，建议在名称后面加Subject；观察者接口的定义，建议在名称后面跟Observer；观察者接口的更新方法，建议名称为update。

##### 6. java中的观察者模式
在Java中已经有观察者模式的部分实现。在java.util包里面有个类`Observable`，它实现了大部分我们需要的目标的功能；`Observer`接口，就是观察者接口，定义了update的方法。

#### 二、如何使用观察者模式
##### 案例一
模拟生活中订阅报纸的基本流程：
1. 订阅者想出版者订阅报纸，订阅者可以有多个；
2. 出版者出版新报纸时，把报纸发放给订阅者。

上面简单流程中，订阅者完成订阅后，最关心的是何时能收到新出的报纸。如果报纸出版的时间不固定，订阅者想要第一时间阅读到新报纸，只能守着邮箱了。

将上述问题抽象描述：当出版者类的状态发生改变时，多个订阅者类如何能得到通知，以及做出相应的改变。

###### 使用观察者模式实现案例
报纸对象相当于目标，订阅者相当于观察者。

```java
/**
 * 目标对象，作为被观察者
 */
public class Subject {
    private List<Observer> readers = new ArrayList<>();

    /**
     * 订阅报纸
     * @param reader 读者
     */
    public void attach(Observer reader) {
        readers.add(reader);
        System.out.println(((RealReader)reader).getName() + "订阅了报纸");
    }

    /**
     * 取消订阅
     * @param reader 读者
     */
    public void detach(Observer reader) {
        readers.remove(reader);
        System.out.println(((RealReader)reader).getName() + "订阅了报纸");
    }

    protected void notifyReaders() {
        for(Observer reader : readers) {
            reader.update(this);
        }
    }
}

/**
 * 报纸对象，具体的目标实现
 */
public class NewsPaper extends Subject {
    private String content;

    public String getContent() {
        return content;
    }

    public void releaseNewspaper(String content) {
        this.content = content;
		// 内容有了，通知所有的读者
        notifyReaders();
    }
}

/**
 * 观察者 - 报纸的读者
 */
public interface Observer {
    void update(Subject subject);
}

public class RealReader implements Observer {
    private String name;
    public RealReader(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    @Override
    public void update(Subject subject) {
        System.out.println(name + "收到了报纸,内容:" + ((NewsPaper)subject).getContent());
    }
}

public class NewpaperClient {
    public static void main(String[] args) {
        NewsPaper newsPaper = new NewsPaper();
        RealReader reader1 = new RealReader("张三");
        RealReader reader2 = new RealReader("李四");
        RealReader reader3 = new RealReader("王五");
        RealReader reader4 = new RealReader("赵柳");

        newsPaper.attach(reader1);
        newsPaper.attach(reader2);
        newsPaper.attach(reader3);
        newsPaper.attach(reader4);

        newsPaper.setContent("罗斯牛逼");
    }
}
```

#### 三、观察者模式的优缺点

##### 优点
- 实现了观察者和目标之间的抽象耦合  
原本目标对象在状态改变时，需要直接调用所有的观察者对象；抽象出观察者接口后，目标只是知道观察者接口，从而实现目标和具体观察者之间的解耦。

- 实现了动态联动  
观察者模式对观察者注册实行管理，那就可以在运行期间，动态地控制注册的观察者，来控制某个动作的联动范围，从而实现动态联动。

- 支持广播通信
由于目标发送通知给观察者是面向所有注册的观察者，所以每次通知的信息就要对所有注册的观察者进行广播。

##### 缺点
- 可能引起无谓的操作  
由于观察者模式每次都是广播通信，不管观察者需不需要，每个观察者都会被调用update方法，如果观察者不需要执行相应处理，那么这次操作就浪费了。

#### 四、什么时候选用观察者模式
1. 当一个抽象模型用两个方面，其中一个方面的操作依赖于另一个方面的状态变化，就可以使用观察者模式。这样就把抽象模型的这两个方面分离开了，使得它们可以独立地改变和复用。
2. 如果在更改一个对象时，需要同时改变其他对象，且不知道有多少对象需要被连带改变，这种情况可以选用观察者模式。
3. 当一个对象必须通知其他对象，但是希望这个对象和其他被通知的对象是松耦合的。就可以使用观察者模式。

#### 五、相关模式