---
title: Go 语言中defer的一些细节
permalink: go-defer
toc: true
date: 2020-11-06 19:49:28
tags:
 - Golang
categories:
 - Go基础
---



defer语句是Go中一个非常有用的特性，可以将一个方法延迟到包裹该方法的方法返回时执行，在实际应用中，defer语句可以充当其他语言中try…catch…的角色，也可以用来处理关闭文件句柄等收尾操作。

### defer触发时机

> A "defer" statement invokes a function whose execution is deferred to the moment the surrounding function returns, either because the surrounding function executed a return statement, reached the end of its function body, or because the corresponding goroutine is panicking.

Go官方文档中对defer的执行时机做了阐述，分别是。

- 包裹defer的函数返回时
- 包裹defer的函数执行到末尾时
- 所在的goroutine发生panic时

### defer执行顺序

当一个方法中有多个defer时， defer会将要延迟执行的方法“压栈”，当defer被触发时，将所有“压栈”的方法“出栈”并执行。所以defer的执行顺序是LIFO的。

所以下面这段代码的输出不是1 2 3，而是3 2 1。

```go
func stackingDefers() {
    defer func() {
        fmt.Println("1")
    }()
    defer func() {
        fmt.Println("2")
    }()
    defer func() {
        fmt.Println("3")
    }()
}
```



### 坑1：defer在匿名返回值和命名返回值函数中的不同表现

先看下面两个方法执行的结果。



```go
// 第一种：
func returnValues() int {
    var result int
    defer func() {
        result++
        fmt.Println("defer")
    }()
    return result
}
// 0

// 第二种
func namedReturnValues() (result int) {
    defer func() {
        result++
        fmt.Println("defer")
    }()
    return result
}
// 1
```

上面的方法会输出0，下面的方法输出1。上面的方法使用了匿名返回值，下面的使用了命名返回值，除此之外其他的逻辑均相同，为什么输出的结果会有区别呢？

要搞清这个问题首先需要了解defer的执行逻辑，文档中说defer语句在方法返回“时”触发，也就是说return和defer是“同时”执行的。

第一种：以匿名返回值方法举例，过程如下。

- 将result赋值给返回值（可以理解成Go自动创建了一个返回值`retValue`，相当于执行`retValue = result`）
- 然后检查是否有defer，如果有则执行
- 返回刚才创建的返回值（retValue）

在这种情况下，defer中的修改是对`result`执行的，而不是`retValue`，所以defer之后返回的依然是`retValue`

第二种：以命名返回值方法举例，

* 由于返回值在方法定义时已经被定义，所以没有创建`retValue`的过程，`result`就是`retValue`，defer对于`result`的修改也会被直接返回。

### 坑2：在for循环中使用defer可能导致的性能问题

看下面的代码

```go
func deferInLoops() {
    for i := 0; i < 100; i++ {
        f, _ := os.Open("/etc/hosts")
        defer f.Close()
    }
}
```

defer在紧邻创建资源的语句后生命力，看上去逻辑没有什么问题。但是和直接调用相比，defer的执行存在着额外的开销，例如defer会对其后需要的参数进行内存拷贝，还需要对defer结构进行压栈出栈操作。所以在循环中定义defer可能导致大量的资源开销，在本例中，可以将`f.Close()`语句前的defer去掉，来减少大量defer导致的额外资源消耗。

### 坑3：判断执行没有err之后，再defer释放资源

一些获取资源的操作可能会返回err参数，我们可以选择忽略返回的err参数，但是如果要使用defer进行延迟释放的的话，需要在使用defer之前先判断是否存在err，如果资源没有获取成功，即没有必要也不应该再对资源执行释放操作。如果不判断获取资源是否成功就执行释放操作的话，还有可能导致释放方法执行错误。

正确写法如下。

```go
resp, err := http.Get(url)
// 先判断操作是否成功
if err != nil {
    return err
}
// 如果操作成功，再进行Close操作
defer resp.Body.Close()
```



### 坑4：调用os.Exit时defer不会被执行

当发生panic时，所在goroutine的所有defer会被执行，但是当调用`os.Exit()`方法退出程序时，defer并不会被执行。

```go
func deferExit() {
    defer func() {
        fmt.Println("defer")
    }()
    os.Exit(0)
}
```

上面的defer并不会输出。