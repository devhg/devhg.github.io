---
title: Go struct间的比较
permalink: go-strcut-cmp
toc: true
date: 2020-12-22 22:12:21
tags:
 - Golang
categories:
 - Go基础
---



<!--more-->

struct能不能比较？ 很显然这句话包含了两种情况：

- 同一个struct的两个实例能不能比较？
- 两个不同的struct的实例能不能比较？



在分析上面两个问题前，先跟大家梳理一下golang中，哪些数据类型是可比较的，哪些是不可比较的：

- 可比较：Integer，Floating-point，String，Boolean，Complex(复数型)，Pointer，Channel，Interface，Array
- 不可比较：Slice，Map，Function

下面就跟大家分别分析一下上面两种情况吧

## 问：同一个struct的两个实例能不能比较

首先，我们构造一个struct结构体来玩玩吧

```go
type S struct {
    Name    string
    Age     int
    Address *int
}
func main() {
    a := S{
        Name:    "aa",
        Age:     1,
        Address: new(int),
    }
    b := S{
        Name:    "aa",
        Age:     1,
        Address: new(int),
    }
   fmt.Println(a == b)
}

```

运行上面的代码发现会打印false。既然能正常打印输出，说明是可以个比较的，接下来让我们来个**「死亡两问」**



> 什么可以比较？

回到上面的划重点部分，在总结中我们可以知道，**golang中 Slice，Map，Function 这三种数据类型是不可以直接比较的**。我们再看看S结构体，该结构体并没有包含不可比较的成员变量，所以该结构体是可以直接比较的。



> 为什么打印输出false？

a 和 b 虽然是同一个struct 的两个实例，但是因为其中的指针变量 Address 的值不同，所以 a != b，如果a b 在初始化时把 Address 去掉（不给 Address 初始化），那么这时 a == b 为true, 因为ptr变量默认值是nil，又或者给 Address 成员变量赋上同一个指针变量的值，也是成立的。

如果给结构体S增加一个Slice类型的成员变量后又是什么情况呢？

```go
type S struct {
    Name    string
    Age     int
    Address *int
   Data    []int
}
func main() {
    a := S{
        Name:    "aa",
        Age:     1,
        Address: new(int),
      Data:    []int{1, 2, 3},
    }
    b := S{
        Name:    "aa",
        Age:     1,
        Address: new(int),
      Data:    []int{1, 2, 3},
    }
   fmt.Println(a == b)
}

```

这时候会打印输出什么呢？true？false？实际上运行上面的代码会报下面的错误：

```
# command-line-arguments
./test.go:37:16: invalid operation: a == b (struct containing []int cannot be compared)
```

a, b 虽然是同一个struct两个赋值相同的实例，因为结构体成员变量中带有了不能比较的成员（slice)，是不可以直接用 == 比较的，所以只要写 == 就报错

「总结」

同一个struct的两个实例可比较也不可比较，当结构不包含不可直接比较成员变量时可直接比较，否则不可直接比较


---


但在平时的实践过程中，当我们需要对含有不可直接比较的数据类型的结构体实例进行比较时，是不是就没法比较了呢？事实上并非如此，golang还是友好滴，我们可以借助 reflect.DeepEqual 函数 来对两个变量进行比较。所以上面代码我们可以这样写：

```go
type S struct {
    Name    string
    Age     int
    Address *int
   Data    []int
}
func main() {
    a := S{
        Name:    "aa",
        Age:     1,
        Address: new(int),
      Data:    []int{1, 2, 3},
    }
    b := S{
        Name:    "aa",
        Age:     1,
        Address: new(int),
      Data:    []int{1, 2, 3},
    }
   fmt.Println(reflect.DeepEqual(a, b))
}
// true
```

那么 reflect.DeepEqual 是如何对变量进行比较的呢？<br />



### reflect.DeepEqual

DeepEqual函数用来判断两个值是否深度一致。具体比较规则如下：

- 不同类型的值永远不会深度相等
- 当两个数组的元素对应深度相等时，两个数组深度相等
- 当两个相同结构体的所有字段对应深度相等的时候，两个结构体深度相等
- 当两个函数都为nil时，两个函数深度相等，其他情况不相等（相同函数也不相等）
- 当两个interface的真实值深度相等时，两个interface深度相等
- map的比较需要同时满足以下几个
  - 两个map都为nil或者都不为nil，并且长度要相等
  - 相同的map对象或者所有key要对应相同
  - map对应的value也要深度相等
- 指针，满足以下其一即是深度相等
  - 两个指针满足go的==操作符
  - 两个指针指向的值是深度相等的
- 切片，需要同时满足以下几点才是深度相等
  - 两个切片都为nil或者都不为nil，并且长度要相等
  - 两个切片底层数据指向的第一个位置要相同或者底层的元素要深度相等
  - 注意：空的切片跟nil切片是不深度相等的
- 其他类型的值（numbers, bools, strings, channels）如果满足go的==操作符，则是深度相等的。要注意不是所有的值都深度相等于自己，例如函数，以及嵌套包含这些值的结构体，数组等



## 问：两个不同的struct的实例能不能比较

**「结论」**：可以比较，也不可以比较



可通过强制转换来比较：

```go
type T2 struct {
    Name  string
    Age   int
    Arr   [2]bool
    ptr   *int
}
type T3 struct {
    Name  string
    Age   int
    Arr   [2]bool
    ptr   *int
}
func main() {
    var ss1 T2
    var ss2 T3
    // Cannot use 'ss2' (type T3) as type T2 in assignment
    //ss1 = ss2     // 不同结构体之间是不可以赋值的
    ss3 := T2(ss2)
    fmt.Println(ss3==ss1) // true
}

```

如果成员变量中含有不可比较成员变量，即使可以强制转换，也不可以比较

```go
type T2 struct {
    Name  string
    Age   int
    Arr   [2]bool
    ptr   *int
    map1  map[string]string
}
type T3 struct {
    Name  string
    Age   int
    Arr   [2]bool
    ptr   *int
    map1  map[string]string
}
func main() {
    var ss1 T2
    var ss2 T3
    
    ss3 := T2(ss2)
    fmt.Println(ss3==ss1)   // 含有不可比较成员变量
}

```

编译报错：

```
# command-line-arguments
./test.go:28:18: invalid operation: ss3 == ss1 (struct containing map[string]string cannot be compared)
```



## 问：struct可以作为map的key么

struct必须是可比较的，才能作为key，否则编译时报错

```go
type T1 struct {
    Name  string
    Age   int
    Arr   [2]bool
    ptr   *int
    slice []int
    map1  map[string]string
}
type T2 struct {
    Name string
    Age  int
    Arr  [2]bool
    ptr  *int
}
func main() {
    // n := make(map[T2]string, 0) // 无报错
    // fmt.Print(n)                // map[]
    m := make(map[T1]string, 0)
    fmt.Println(m) // invalid map key type T1
}
```



