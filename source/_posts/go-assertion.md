---
title: Go 断言的实用
date: 2020-08-01 08:05:56
toc: true
tags:
 - Golang
categories:
 - Go基础
---

golang的语言中提供了断言的功能。golang中的所有程序都实现了interface{}的接口，这意味着，所有的类型如string,int,int64甚至是自定义的struct类型都就此拥有了interface{}的接口，这种做法和java中的Object类型比较类似。那么在一个数据通过func funcName(interface{})的方式传进来的时候，也就意味着这个参数被自动的转为interface{}的类型，那么转回原有的类型就要用到断言了。





如以下的代码：

```go
func funcName(a interface{}) string {
     return string(a)
}
```

编译器将会返回：

`cannot convert a (type interface{}) to type string: need type assertion`

此时，意味着整个转化的过程需要类型断言。类型断言有以下几种形式：

1）直接断言使用

和类型转换不同，类型断言是将**接口类型**的值x，转换成类型T。类型断言的必要条件是x是接口类型,非接口类型的x不能做类型断言:

```go
value, ok := x.(T)
//x表示要断言的接口变量；
//T表示要断言的目标类型；
//value表示断言成功之后目标类型变量；
//ok表示断言的结果，是一个bool型变量，true表示断言成功，false表示失败，如果失败value的值为nil。

var a interface{}
fmt.Println("Where are you,Jonny?", a.(string))
```

但是如果断言失败一般会导致panic的发生。所以为了防止panic的发生，我们需要在断言前进行一定的判断

`value, ok := a.(string)`

如果断言失败，那么ok的值将会是false,但是如果断言成功ok的值将会是true,同时value将会得到所期待的正确的值。示例：

```go
value, ok := a.(string)
if !ok {
    fmt.Println("It's not ok for type string")
    return
}
fmt.Println("The value is ", value)
```

2）另外也可以配合switch语句进行判断：

type switch语句中可以有一个简写的变量声明，这种情况下，等价于这个变量声明在每个case clause隐式代码块的开始位置。如果case clause只列出了一个类型，则变量的类型就是这个类型，否则就是原始值的类型。

```go
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("unexpected type %T", t)       // %T prints whatever type t has
    break
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool
    break
case int, int32:
    fmt.Printf("integer %d\n", t)             // t has type int
    break
case *bool:
    fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
    break
case *int:
    fmt.Printf("pointer to integer %d\n", *t) // t has type *int
    break
}
```