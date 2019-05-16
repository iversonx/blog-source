---
title: 行为型模式——迭代器模式
date: 2018-10-26
categories: 设计模式
tags: 学习笔记
description: 
---
#### 一、什么是迭代器模式
##### 1. 定义
提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示。

##### 2. 应用迭代器模式解决问题的思路
1. 分析出统一的访问方式，按照这个统一的访问方式定义出来的接口,即Iterator接口；
2. 给所有的聚合对象抽象出一个公共的父类，让它提供操作聚合对象的公共接口，这个抽象的公共父类在迭代器模式中对应的就是Aggregate对象。

##### 3. 迭代器模式的结构和说明
[示例图]
- Iterator：迭代器接口。定义访问和遍历元素的接口。
- ConcreteIterator：具体的迭代器实现对象。实现对聚合对象的遍历，并跟踪遍历时的当前位置。
- Aggregate：聚合对象，定义创建相应迭代器对象的接口。
- ConcreteAggregate：具体聚合对象。实现创建相应的迭代器对象。

##### 4. 示例代码
```java
public interface Iterator {
	public void first();
	public void next();
	public boolean isDone();
	public Object currentItem();
}

public class ConcreteIterator implements Iterator {
	private ConcreteAggregate aggregate;
	private int index = -1;
	public Concreteiterator(Concreteiterator aggregate) {
		this.aggregate = aggregate;
	}

	public void first() {
		index = 0;
	}

	public void next() {
		if(index < aggregate.size()) {
			index = index + 1;
		}
	}

	public boolean isDone() {
		if(index == this.aggregate.size()) {
			return true;
		}
		return false;		
	}

	public Object currentItem() {
		return this.aggregate.get(index);
	}
}

public interface Aggregate {
	public Iterator createIterator();
}

public class ConcreteAggregate implements Aggregate {
	// 示意，表示聚合对象具体的内容 
	private String[] ss = null;

	public ConcreteAggregate(String[] ss) {
		this.ss = ss;
	}

	public Iterator createIterator() {
		return new ConcreteIterator(this);
	}

	public Object get(int index) {
		Object object = null;
		if(index < ss.length) {
			object = null;
		}
		return object;
	}

	public int size() {
		reutn ss.length;
	}
}

public class Client { 
	public void someoperation() {
		String[] names = {"张三","李四","王五"};
		Aggregate aggregate = new ConcreteAggregate(names);
		Iterator it = aggregate.createIterator();
		it.first();
		while(!it.isDone()) {
			Object obj = it.currentItem();
			System.out.println(obj);
			it.next();
		}
	}
	
	public static void main(String[] args) {
		Client client = new Client();
		client.someOperation();
	}
}
```

#### 二、案例 - 整合数据
##### 背景
A公司收购了B公司，B公司都有自己的工资系统，现要整合到A公司已有的工资系统中。A公司工资系统内部采用List来记录工资列表；B公司使用数组来记录工资列表。幸运的是，两个系统描述工资的数据模型是差不多的。

现在要整合两个系统的数据，但因为种种原因，不能将List改成数组，或数组改成List。现在老板希望能够以一个统一的方式来查看所有的工资数据。



#### 三、迭代器模式的优缺点

#### 四、什么时候选用迭代器模式

#### 五、相关模式