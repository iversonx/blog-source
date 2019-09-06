---
title: golang中的数组
date: 2019-06-28
categories: http
tags: 学习笔记
description: 
---

## 主要内容

- 使用加载和保存方法创建数据结构

- 使用`net/http`包构建Web应用程序

- 使用`html/template`包处理HTML模版

- 使用`regexp`包验证用户输入

- 使用闭包

## 准备工作

> 前提：已经搭建好Go开发环境

1. 创建目录

```bash
mkdir gowiki
cd gowiki
```

2. 创建一个go文件作为程序入口，你可以在喜欢的编辑器中打开这个文件

```go
package main

import (
    "fmt"
    "io/ioutil"
)
```

## 定义数据结构

定义一个名为Page的`struct`，其中两个字段代表标题和正文。

```go
type Page struct {
    Title string
    // 表示一个字节分片
    Body []byte
}
```

Body元素是`[]byte`，而不是`string`，因为`[]byte`是后面使用的io库所期望的类型。

### 保存页面

Page结构描述了页面数据将如何存储在内存中。为Page添加一个save方法，进行持久化

```go
func (p *Page) save() error {
    filename := p.Ttitle + ".txt"
    return ioutil.WriteFile(filename, p.Body, 0600)
}
```

save方法将Page的Body保存到一个文本文件，其文件名为Title。返回错误值，是为了让应用程序处理写入文件时发生的错误。

0600是一个八进制整数，在Unix中表示读写权限，这里意思是为当前用户创建具有读写权限权限的文件。

### 加载页面

```go
func loadPage(title string) (*Page, error) {
    filename := title + 'txt'
    body, _ := ioutil.ReadFile(filename)
    return &Page{Title: title, Body: body}, nil
}
```

到此，简单的数据结构就定义好了，其能够保存到文件并从文件中加载。现在在`main`函数中测试上面代码

```go
func main() {
	p1 := &Page{
		Title: "index",
		Body:  []byte("This is a index Page."),
	}
	err := p1.save()
	if err != nil {
		os.Exit(-1)
	}

	p2, _ := loadPage("index")
	fmt.Println(string(p2.Body))
}
```

## 简单使用net/http包

```go
package main

import (
	"fmt"
	"net/http"
)

func main() {
	// 告诉http包处理所有对web根目录(/)的请求
	http.HandleFunc("/", handler)
	// 指定在所有接口上监听8080端口
	// ListenAndServe只有在发生意外错误时返回，其返回结果是一个error
	_ = http.ListenAndServe(":8080", nil)
}

/*
handler的类型为http.HandleFunc, 因为handler使用http.ResponseWriter和http.Request作为它的参数

http.ResponseWriter值组装HTTP服务响应，通过写入，将数据发送到HTTP客户端

http.Request表示客户端HTTP请求的数据结构

request.URL.Path是请求URL的路径组件
 */
func handler(writer http.ResponseWriter, request *http.Request) {
	_, _ = fmt.Fprintf(writer, "Hi there, I love %s!", request.URL.Path[1:])
}
```

## 使用net/http包提供页面

创建一个允许用户查看页面的处理程序viewHandler，处理前缀为"/view/"的URL。

```go
func viewHandler(w http.ResponseWriter, r *http.Request) {
	title:= r.URL.Path[len("/view/"):]
	// 使用_来忽略loadPage的错误返回值，通常被认为是不好的做法，
	// 这里是为了简单

	p, _ :=loadPage(title)
	_, _ = fmt.Fprintf(w, "<h1>%s</h1><div>%s</div>", p.Title, p.Body)
}
```

在main函数中使用viewHandler来初始化http以处理path/view下的任何请求。

```go
func main() {
	http.HandleFunc("/view/", viewHandler)
	_ = http.ListenAndServe(":8080", nil)
}
```

创建一些数据文件，然后通过浏览器访问。例如创建一个index.txt放入你的工作目录，并保存"This is index page"字符串到index.txt；然后运行程序后，访问127.0.0.1:8080/view/index。

## 编辑页面

现在开始提供编辑保存页面功能。先创建两个新的handler: `editHadler`用于显示编辑表单，`saveHadler`用于保存通过表单输入的数据。

```go
func editHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/edit/"):]
    p, err := loadPage(title)
    if err != nil {
        p = &Page{Title: title}
    }
    fmt.Fprintf(w, "<h1>Editing %s</h1>"+
        "<form action=\"/save/%s\" method=\"POST\">"+
        "<textarea name=\"body\">%s</textarea><br>"+
        "<input type=\"submit\" value=\"Save\">"+
        "</form>",
        p.Title, p.Title, p.Body)
}

func saveHandler(w http.ResponseWriter, r *http.Request) {
	title := r.URL.Path[len("/save/"):]
	body := r.FormValue("body")
	p := &Page{Title: title, Body: []byte(body)}
	_ = p.save()
	http.Redirect(w, r, "/view/"+title, http.StatusFound)
}

// 在main函数中添加这两个handler
func main() {
	http.HandleFunc("/view/", viewHandler)
	http.HandleFunc("/edit/", editHandler)
	http.HandleFunc("/save/", saveHandler)
	_ = http.ListenAndServe(":8080", nil)
}
```

运行程序，访问127.0.0.1:8080/edit/detail，编辑内容，点击保存；然后我们输入的内就保存到detail.txt文件中了。

## 使用html/template包

上面的示例不足的是，页面的布局是编写在go源码中的，这意味着每次更改页面布局，都要重新修改go源码。

`html/template`包是Go标准库中一部分，可以使用这个包，将html保存在一个单独的文件中，这样就可以在不修改底层go代码的情况下更改页面布局。



首先将`html/template`包添加到导入列表中。

```go
import (
	"html/template"
	"io/ioutil"
	"net/http"
)
```

模版view.html

```html
<h1>{{.Title}}</h1>

<p>[<a href="/edit/{{.Title}}">edit</a>]</p>

<div>{{printf "%s" .Body}}</div>
```

模版edit.html

```html
<h1>Editing {{.Title}}</h1>
 <form action="/save/{{.Title}}" method="POST">
 <div><textarea name="body" rows="20" cols="80">{{printf "%s" .Body}}</textarea></div>
 <div><input type="submit" value="Save"></div>
 </form>
```

修改editHandler以使用模版

```go
func editHandler(w http.ResponseWriter, r *http.Request) {
    title := r.URL.Path[len("/edit/"):]
    p, err := loadPage(title)
    if err != nil {
        p = &Page{Title: title}
    }
    t, _ := template.ParseFiles("edit.html")
    t.Execute(w, p)
}
```

template.ParseFiles函数将读取edit.html的内容并返回*temple.Template。

t.Execute函数执行模版，将生成的HTML写入http.ResponseWriter。

模版中的.Title和.Body标识符指的是p.Title和p.Body。

`html/template`包有助于保证模版操作只生成安全且正确的HTML。例如，它会自动转义大于(>)符号，替换为& gt;以确保用户数据不会破坏表单HTML。

## 处理不存在的页面

当访问一个不存在的页面/view/list，会看到包含HTML的页面。这是因为忽略了loadPage的错误返回值，并继续尝试填写没有数据的模版。

现在对loadPage的错误进行处理，当页面不存在是，重定向到编辑页面。

```go
func viewHandler(w http.ResponseWriter, r *http.Request) {
	title:= r.URL.Path[len("/view/"):]
	p, err :=loadPage(title)
	if err != nil {
		http.Redirect(w, r, "/edit/" + title, http.StatusFound)
		return
	}
	renderTemplate(w, "template/view", p)
}
```

## 缓存模版

每次呈现页面时,renderTemplate都会调用ParseFiles，这样的代码是低效率的。

更好的方法是爱程序初始化时调用ParseFiles一次，将所有模版解析为单个*Template，然后可以使用ExecuteTemplate方法呈现特定模版。

```go
// 定义一个全局变量，预先加载模版
var templates = template.Must(
	template.ParseFiles("./template/edit.html","./template/view.html"))

// 然后修改renderTemplate函数
func renderTemplate(w http.ResponseWriter, tmpl string, p *Page) {
	err := templates.ExecuteTemplate(w, tmpl + ".html", p)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
	}
	/* err = t.Execute(w, p)

	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
	}*/
}
```

这样就必须每次都加载解析模版，只要渲染模版就可以了。

如果修改了模版，就需要重新启动程序。

## 闭包

捕获每个处理程序中的错误条件会引入大量重复的代码。Go的字面函数提供了一种强大的抽象功能可以将每个处理程序包装在执行验证和错误检查的函数中。



首先在每个处理程序的函数定义增加一个string参数用来接收title字符串。

```go
func viewHandler(w http.ResponseWriter, r *http.Request, title string)

func editHandler(w http.ResponseWriter, r *http.Request, title string)

func saveHandler(w http.ResponseWriter, r *http.Request, title string)
```

定义一个将上述函数作为参数的行数，并返回一个http.HandlerFunc类型的函数

```go
func makeHandler(fn func(w http.ResponseWriter, r *http.Request, title string)) http.HandlerFunc {
    // 返回的函数被称为闭包，因为它包含在其外部定义的值。

	return func(writer http.ResponseWriter, request *http.Request) {
		m := validPath.FindStringSubmatch(request.URL.Path)
		if m == nil {
			http.NotFound(writer, request)
			return
		}
		fn(writer, request, m[2])
	}
}
```

修改main函数

```go
func main() {
	http.HandleFunc("/view/", makeHandler(viewHandler))
	http.HandleFunc("/edit/", makeHandler(editHandler))
	http.HandleFunc("/save/", makeHandler(saveHandler))
	_ = http.ListenAndServe(":8080", nil)
}
```

修改handler函数

```go
func viewHandler(w http.ResponseWriter, r *http.Request, title string) {
    p, err := loadPage(title)
    if err != nil {
        http.Redirect(w, r, "/edit/"+title, http.StatusFound)
        return
    }
    renderTemplate(w, "view", p)
}

func editHandler(w http.ResponseWriter, r *http.Request, title string) {
    p, err := loadPage(title)
    if err != nil {
        p = &Page{Title: title}
    }
    renderTemplate(w, "edit", p)
}

func saveHandler(w http.ResponseWriter, r *http.Request, title string) {
    body := r.FormValue("body")
    p := &Page{Title: title, Body: []byte(body)}
    err := p.save()
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    http.Redirect(w, r, "/view/"+title, http.StatusFound)
}
```
