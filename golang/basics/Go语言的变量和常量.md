---
title: Go语言的变量和常量
date: 2019-10-10
categories: Go
tags: 学习笔记
description: 
---

## 变量

变量是Go语言中表示抽象值和存储计算结果的一种概念。变量值通过变量名访问。

Go语言在声明变量时将类型放在名称后面，这是与其他大部分编程语言不同之处。

Go语言的变量名命名规则：只能由字母，数字，下划线组成，其中首字符不能为数字。

Go语言变量声明格式

```go
var name type = value
// var 关键字用于声明变量
// name 变量名
// type 变量类型
// 变量值，可以是表达式
```

### 变量零值

上面格式中的type和value可以省略一个。如果类型省略，变量类型由value决定；如果value省略，其初始值默认是零值：

- 数字对应的零值为0，

- bool对应的零值为false，

- string对应的零值为""，

- 接口和引用类型对应的零值为`nil`。

```go
// number的初始值为0
var number int
// str的初始值为空字符串
var str string
// boolean的初始值为false
var boolean bool
```

Go语言可以同时声明多个变量；忽略类型允许声明多个不同类型的变量。

```go
// 同时声明多个int类型变量
var i, j ,k int = 1,2,3
// 同时声明bool，float， string类型的变量
var b, f, s = true, 2.3, "four"

var (
    a, b int
    c bool
    str string
)
```

### 局部变量声明

在函数中，局部变量声明可以用来声明和初始化局部变量。格式为: `name := expression`。name的类型由expression的类型决定。

```go
number := 123
str := 'go语言'
boolean := false
// 与var一样，可以同时声明多个变量
i, j := 1, 2
```

`var`声明通常是为那些与初始化表达式类型不一致的局部变量保留的，或用于后面才对变量赋值以及变量初始值不重要的情况。而在局部变量的声明和初始化中主要使用短声明。

> 局部变量一旦声明，就必须被使用。否则编译不通过。

## 常量

Go语言中，常量是指编译期间就已知，且在执行过程中不可改变的变量。Go语言中使用`const`关键字来定义常量，可以定义的常量类型有数值，布尔值，字符串。格式为`const identifier [type] = value`

```go
const PI = 3.1415926536
const F int32 = 120
const IS_TRUE = true
const HELLO = "你好，GO语言"

const A, B, C = 1, 2, 3

const (
    MONDAY, TUESDAY, WEDNESDAY = 1, 2, 3
)

const (
 MONDAY = 1
 TUESDAY = 2
 WEDNESDAY = 3
)

const (
 MONDAY = iota + 1
 TUESDAY
 WEDNESDAY
)
```

## 命名规范

变量的命名规范一般遵循驼峰命名法：首个单词小写，每个独立单词的首字母大写，例如：setNum。

常量命名通常都是大写，多个单词之间用下划线分隔。
