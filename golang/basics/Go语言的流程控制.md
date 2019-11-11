---
title: Go语言的流程控制
date: 2019-10-11
categories: Go
description: 
---

## if语句

Go语言中`if`语句和java语言中`if`语句不同之处在于，Go语言中`if`语句可以有一个子语句，用于初始化局部变量。

常规`if`语句

```go
if a < 20 {
    // if块
} else if a < 50 {
    // else if 块
} else {
    // else 块    
}
```

不同于java的`if`语句

```go
// 这里直接在if中声明和初始化变量a，变量a只能在if else 块中访问
if a := 10; a < 20 {
    
} else {
    
}
```

## switch语句

Go语言中`switch`语句根据初始化表达式得出值，然后根据`case`语句的条件，执行相应的代码块。

Go语言中`switch`语句没有`break`，当执行其中一个`case`语句，会自动跳出整个`switch`。

Go语言中`switch`语句也可以拥有初始化子语句。



方式1，和java的`switch`语句类似，

`switch`后面跟着变量，根据这个变量值决定执行哪个`case`语句

```go
grade := "B"
marks := 90

switch marks {

case 90:

	grade = "A"

case 80:

	grade = "B"

case 60, 70:

	grade = "C"

default:

	grade = "D"

}
```

方式2，类似于`if`语句，每一个case相当于`else if`可以进行逻辑运算：

```go
    grade := "E"
    marks := 90
	switch {

	case marks >= 90:
		grade = "A"
	case marks >= 80:
		grade = "B"
	case marks >= 70:
		grade = "C"
	case marks >= 60:
		grade = "D"
	default:
		grade = "E"
```

方式3，针对变量类型来决定执行哪个case，前提是该变量是`interface`

```go
var x interface{}
// 将x赋值为一个空字符串
x = ""
switch i := x.(type) {
	case nil:
		t.Logf("x的类型是%T", i)
	case int:
		t.Logf("x的类型是%T", i)
	case float64:
		t.Logf("x的类型是%T", i)
	case bool:
		t.Logf("x的类型是%T", i)
	case string:
		t.Logf("x的类型是%T", i)
	default:
		t.Logf("x的类型是%T", i)
	}
```






