---
title: Go 基础之import包与包内init方法的执行时机
date: 2020-06-17 22:32:57
toc: true
tags:
 - Golang
categories:
 - Go基础
---

Go语言导包的几种方式

<!--more-->

### 第一种方式相对路径

```go
import "./module"   //当前文件同一目录的module目录，此方式没什么用并且容易出错
```

### 第二种方式绝对路径

```go
import "LearnGo/init"    //加载gopath/src/LearnGo/init模块
//如果使用go mod的话，加载gopath/pkg/mod/LearnGo/init模块
```

下面几种特殊的操作



1. 点操作

```go
import . "fmt"
```

这个点操作的含义就是这个包导入之后在你调用这个包的函数时，你可以省略前缀的包名，也就是前面你调用的`fmt.Println("hello world")`可以省略的写成`Println("hello world")`

2. 别名操作

```go
import f "fmt"
```

别名操作的话调用包函数时前缀变成了我们的前缀，即`f.Println("hello world")。`

3. _(占位符) 操作

```go
import _ "fmt"
```

_ 操作其实是引入该包，而不直接使用包里面的函数，而是**调用了该包里面的init函数**，要理解这个问题，需要看下面这个图，理解包是怎么按照顺序加载的。



### import后的操作

程序的初始化和执行都起始于main包。如果main包还导入了其它的包，那么就会在编译时将它们依次导入。有时一个包会被多个包同时导入，那么它 **只会被导入一次**（例如很多包可能都会用到fmt包，但它只会被导入一次，因为没有必要导入多次）。当一个包被导入时，如果该包还导入了其它的包，那么会先 将其它包导入进来，然后再对这些包中的包级常量和变量进行初始化，接着执行init函数（如果有的话），依次类推。等所有被导入的包都加载完毕了，就会开 始对main包中的包级常量和变量进行初始化，然后执行main包中的init函数（如果存在的话），最后执行main函数。此外需了解别名操作方式导入包也会执行init函数。

![go-import](https://i.loli.net/2020/06/17/PuN1pLm7YK4sedv.jpg)

<hr>

### 参考文章

* 
* 