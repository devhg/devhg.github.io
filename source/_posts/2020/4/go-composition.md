---
title: Go 组合取代继承
permalink: go-composition
date: 2020-04-07 21:26:54
toc: true
tags:
 - Golang
categories:
 - Go基础
---



Go 不支持继承，但它支持组合（Composition）。组合一般定义为“合并在一起”，即将几个结构体嵌套起来构成大的结构体类型。汽车就是一个关于组合的例子：一辆汽车由车轮、引擎和其他各种部件组合在一起。

<!--more-->

组合的典型例子就是博客博文。每一个博客的博文都有标题、内容、发表时间和作者信息。使用组合可以很好地表示它们。网站结构体中 放有 文章结构体，文章结构体中 放有 作者信息结构体。下面用代码实现。

### 通过嵌套结构体进行组合

首先定义一个作者信息author结构体和一个获取作者信息的方法`fullName()`方法，其中 `author` 作为接收者类型，该方法返回了作者的全名。

```go
type author struct {
	firstName string
	lastName  string
	//....
}

// 获取作者的姓名
func (a author) fullName() string {
	return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}

```

下面定义一个 博文信息post结构体，以及他的一个方法。它有一个嵌套的匿名字段 `author`。该字段指定 `author` 组成了 `post` 结构体。现在 `post` 可以访问 `author` 结构体的**所有字段和方法**。我们同样给 `post` 结构体添加了 `details()` 方法，用于打印标题、内容和作者的信息。

关键知识点： 一旦结构体A内嵌套了一个结构体B，Go 可以使我们访问嵌套的B的所有字段和方法，好像这些字段属于外部结构体一样。所以下面的 `p.author.fullName()` 可以替换为 `p.fullName()`。

```go
type post struct {
	title   string
permalink: go-composition
	content string
	author  // 嵌套author结构体 **这里会自动组合author的所有方法和字段
}

func (p post) details() {
	fmt.Println("title: ", p.title)
permalink: go-composition
	fmt.Println("content: ", p.content)
	// fmt.Println("author: ", p.author.fullName())
	fmt.Println("author: ", p.fullName()) // 自动组合author的所有方法和字段
}


```

整合运行结果

```go
package main

import (
	"fmt"
)

type author struct {
	firstName string
	lastName  string
	//....
}

// 获取作者的姓名
func (a author) fullName() string {
	return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}

type post struct {
	title   string
permalink: go-composition
	content string
	author  // 嵌套author结构体 **这里会自动组合author的所有方法和字段
}

func (p post) details() {
	fmt.Println("title: ", p.title)
permalink: go-composition
	fmt.Println("content: ", p.content)
	//fmt.Println("author: ", p.author.fullName())
	fmt.Println("author: ", p.fullName()) // 自动组合author的所有方法和字段
}

func main() {
	author1 := author{
		firstName: "gh",
		lastName:  "z",
	}

	post1 := post{
		title:   "1",
permalink: go-composition
		content: "1111",
		author:  author1,
	}
	post1.details()
}
```

```shell
title:  1
permalink: go-composition
content:  1111
author:  gh z

Process finished with exit code 0
```



### 结构体切片的嵌套

结合上面代码，我们加入一个website结构体，一个website结构体中包含多个post结构体，所以采用一个切片来存储。

**注意：**结构体不能嵌套一个匿名切片。我们需要一个字段名。所以我们来修复这个错误，让编译器顺利通过。

```go
package main

import (
	"fmt"
)

type author struct {
	firstName string
	lastName  string
	//....
}

// 获取作者的姓名
func (a author) fullName() string {
	return fmt.Sprintf("%s %s", a.firstName, a.lastName)
}

type post struct {
	title   string
permalink: go-composition
	content string
	author  // 嵌套author结构体 **这里会自动组合author的所有方法和字段
}

func (p post) details() {
	fmt.Println("title: ", p.title)
permalink: go-composition
	fmt.Println("content: ", p.content)
	//fmt.Println("author: ", p.author.fullName())
	fmt.Println("author: ", p.fullName()) // 自动组合author的所有方法和字段
}

// 网站
type website struct {
  //[]post 匿名切片报错
	posts []post
}
// 查看遍历文章
func (w website) view() {
	fmt.Println("view website")
	for _, p := range w.posts {
		p.details()
		fmt.Println()
	}
}

func main() {
	author1 := author{
		firstName: "gh",
		lastName:  "z",
	}

	post1 := post{
		title:   "1",
permalink: go-composition
		content: "1111",
		author:  author1,
	}
	post2 := post{
		title:   "2",
permalink: go-composition
		content: "2222",
		author:  author1,
	}

	w := website{posts: []post{post1, post2}}
	w.view()
}

```

```
view website
title:  1
permalink: go-composition
content:  1111
author:  gh z

title:  2
permalink: go-composition
content:  2222
author:  gh z


Process finished with exit code 0
```

在主函数中，我们创建了一个作者 `author1`，以及两篇博文 `post1`、`post2` 。我们最后通过嵌套两篇博文构成的切片，创建了网站 `w`，并在下一行调用website结构体的 view方法显示内容。





#### 参考文章

* [Go系列教程](https://studygolang.com/articles/12680)

