---
title: 装饰模式(Decorator)
date: 2018-10-26
categories: 设计模式
tags: 学习笔记
description: 
---
#### 一、什么是装饰模式

##### 1. 定义
动态地给一个对象添加一些额外的职责。就增加功能来说，装饰模式比生成子类更为灵活。

在装饰模式的实现中，为了能够实现和原来使用被装饰对象的代码无缝结合，是通过定义一个抽象类，让这个类实现与被装饰对象相同的接口，然后在具体实现类中，转调被装饰的对象，在转调的前后添加新的功能，这就实现了给被装饰对象增加功能。

##### 2. 结构说明
[结构示意图]

- Component：组件对象的接口，可以给这些对象动态地添加职责。
- ConcreteComponent：具体的组件对象，实现组件对象接口，通常就是被装饰器装饰的原始对象，也就是可以给这个对象添加职责。
- Decorator：所有装饰器的抽象父类，需要定一个与组件接口一致的接口，并持有一个Component对象，其实就是持有一个被装饰的对象。
- ConcreteDecorator：实际的装饰器对象，实现具体要向被装饰对象添加的功能。

##### 3. 示例代码
```java
/**
 * 组件对象的接口，可以给这些对象动态地添加职责
 */
public interface Component {
    void operation();
}

public class ConcreteComponent implements Component {
    @Override
    public void operation() {
        System.out.println("ConcreteComponent");
    }
}

public abstract class Decorator implements Component{
    protected Component component;

    public Decorator(Component component) {
        this.component = component;
    }

    public void operation() {
        System.out.println("转发前做附加动作");
        this.component.operation();
        System.out.println("转发后做附加动作");
    }

}

public class ConcreteDecoratorA extends Decorator {
    public ConcreteDecoratorA(Component component) {
        super(component);
    }

    private String addedState;

    public String getAddedState() {
        return addedState;
    }

    public void setAddedState(String addedState) {
        this.addedState = addedState;
    }

    @Override
    public void operation() {
        System.out.println(ConcreteDecoratorA.class.getName() + " Begin...");
        //super.operation();
        component.operation();
        System.out.println(ConcreteDecoratorA.class.getName() + " End...");
    }
}

public class Client {
    public static void main(String[] args) {
        Component component = new ConcreteComponent();
        Decorator decorator = new ConcreteDecoratorA(component);
        decorator.operation();
    }
}
```

装饰模式能够实现动态地为对象添加功能，是从一个对象外部来给对象增加功能，

##### 4. 装饰器
装饰器实现了对被装饰对象的某些装饰功能，可以在装饰器中调用被装饰对象的功能。

装饰器不仅可以给被装饰器对象增加功能，还可以根据需要选择是否调用被装饰对象的功能，如果不调用被装饰对象的功能，那就变成完全重新实现了，相当于动态修改了被装饰对象的功能。

##### 5. 装饰器和组件的关系
装饰器是用来装饰组件的，装饰器一定要实现和组件类一致的接口，保证它们是同一个类型，并具有同一个外观，这样组合完成的装饰才能递归调用下去。

##### 6. Java中的装饰模式应用
###### 1. Java中典型的装饰模式应用——IO流
```java
public class IOTest {
	public static void main(String[] args) throws Exception {
		DataInputStream in = null;
		try {
			in = new DataInputStream(new BufferedInputStream(new FileInputStream("IOTest.txt")));
		} finally {
			in.close();
		}
	}
}
```
在上面代码，最里层是一个FileInputStream对象，然后把它传递给一个BufferedinputStream对象，经过BufferedInputStream处理，再把处理后的对象传递给DataInputStream对象进行处理，这个过程其实就是装饰器的组装过程，FileInputStream对象相当于原始的被装饰的对象，而BufferedinputStream对象和DataInputStream对象则相当于装饰器。


#### 二、装饰模式的优缺点
##### 优点
- 从为对象添加功能的角度来看，装饰模式比继承更灵活。装饰模式采用把功能分离到每个装饰器当中，然后通过对象组合的方式，在运行时动态地组合功能，每个被装饰的对象最终有哪些功能，是由运行期动态组合的功能来决定的。
- 装饰模式把一系列复杂的功能分散到每个装饰器中，使实现装饰器变得简单，这样有利于装饰器功能的复用。可以给一个对象增加多个同样的装饰器，也可以把一个装饰器用来装饰不同的对象，从而实现复用装饰器的功能。

##### 缺点
- 会产生很多细粒度对象：装饰模式把一系列复杂的功能分散到每个装饰器中，一个装饰器只实现一个功能，这样会产生很多细粒度的对象，功能越复杂，需要的细粒度对象越多。


#### 三、何时选用装饰模式
- 如果需要在不影响其他对象的情况下，以动态、透明的方式给对象添加职责，可以使用装饰模式。
- 如果不适合使用子类进行扩展的时候，可以考虑使用装饰模式。
