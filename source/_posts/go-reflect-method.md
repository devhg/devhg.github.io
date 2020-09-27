---
title: Go语言反射进阶
date: 2020-09-27 15:37:36
toc: true
tags:
 - Golang
categories:
 - Go基础
---

Go 语言中反射的操作主要定义在标准库 [`reflect`](https://golang.org/pkg/reflect/) 中，在标准库中定义了两种类型来表现运行时的对象信息，分别是：[`reflect.Value`](https://golang.org/pkg/reflect/#Value)（反射对象的值）和 [`reflect.Type`](https://golang.org/pkg/reflect/#Type)（反射对象的类型），Go 语言中所有反射操作都是基于这两个类型进行的。

<!--more-->



## 一、反射对象 Value 和 Type

既然 Go 语言中所有反射操作都是基于 `Value` 和 `Type` 进行的，那么想要进行反射操作，首先就要获取到反射对象的这两个类型对象才可以。

`reflect` 包提供了两个函数：`reflect.ValueOf(x)` 和 `reflect.TypeOf(x)`，通过这两个函数就可以方便的获取到任意类型（用 `interface{}` 表示任意类型）的 `Value` 对象和 `Type` 对象。只有当X是指针的时候，才可以通过`reflect.Value`修改实际变量X的值，即：要修改反射类型的对象就一定要保证其值是**“addressable”**的。例如：

```go
u := User{"tom", 27, "beijing"}

v := reflect.ValueOf(u)
fmt.Println(v)

t := reflect.TypeOf(u)
fmt.Println(t)
```

知道 `Value` 对象后，也可以通过 `Value.Type()` 方法获取到 `Type` 对象。例如：

```go
t1 := v.Type()
fmt.Println(t == t1)
```

可以看到输出结果为 `true`。

通过 `Type` 类型对象也可以获取到 `Value` 类型对象，不过是零值的指针。例如：

```go
v1 := reflect.New(t)
fmt.Println(v1)
```

结果为：`&{ 0}`

## 二、反射对象的 Kind

`Kind` 表示反射对象的类型 `Type` 所代表的具体类型，零值表示无效的类型，具体有以下类型值：

```go
type Kind uint

const (
    Invalid Kind = iota
    Bool
    Int
    Int8
    Int16
    Int32
    Int64
    Uint
    Uint8
    Uint16
    Uint32
    Uint64
    Uintptr
    Float32
    Float64
    Complex64
    Complex128
    Array
    Chan
    Func
    Interface
    Map
    Ptr
    Slice
    String
    Struct
    UnsafePointer
)
```

可以通过 `Value.Kind()` 或者 `Type.Kind()` 函数获得。例如：

```go
// 获取 Kind 类型
k := t.Kind()
fmt.Println(k)
k1 := v.Kind()
fmt.Println(k1)
fmt.Println(k == k1) // true
fmt.Println()
```

可以看到两种方式获取的结果是一样的，都是 `struct`。

## 三、反射对象的字段

反射能够操作的字段和方法必须是可导出（**首字母大写**）的。

反射对象的字段值修改要通过调用 `Value` 类型的方法 `Elem()` 后返回的 `Value` 对象值来操作。

* `Elem()` 方法定义：`func (v Value) Elem() Value`，返回 `v` 包含的值或指针 `v` 指向的值，如果`v` 的 `Kind` 不是 `Interface` 或 `Ptr`，则会 panic。

* `reflect.Indirect()` 函数的如果参数是指针的 `Value`，则相当于调用了 `Elem()` 方法返回的值，否则返回 `Value` 自身值。
* 上面两个方法主要用于反射字段的操作，使用上面两个方法后无法反射到方法。反射最好传入对象指针类型

```go
// 修改反射对象的值
i := 20
fmt.Println("before i =", i)
e := reflect.Indirect(reflect.ValueOf(&i))
// 或者e := reflect.ValueOf(&i).Elem()
if e.CanSet() {
    e.SetInt(22)
}
fmt.Println("after i =", i)


// 反射字段操作
// elem := reflect.Indirect(reflect.ValueOf(&u))
elem := reflect.ValueOf(&u).Elem()
for i := 0; i < t.NumField(); i++ {
    // 反射获取字段的元信息，例如：名称、Tag 等
    ft := t.Field(i)
    fmt.Println("field name:", ft.Name)
    tag := ft.Tag
    fmt.Println("Tag:", tag) // 获取tag
    fmt.Println("Tag json:", tag.Get("json")) // 根据tag名字获取

    // 反射修改字段的值（）
    fv := elem.Field(i)
    if fv.CanSet() {
        if fv.Kind() == reflect.Int {
            fmt.Println("change age to 30")
            fv.SetInt(30)
        }
        if fv.Kind() == reflect.String {
            fmt.Println("change name to jerry")
            fv.SetString("jerry")
        }
    }
    fmt.Println()
}
fmt.Println("after user:", u)
```



## 四、反射对象的方法

可以通过 `Value` 的 `Method()` 方法或 `Type` 的 `Method()` 方法，两种形式获取对象方法信息进行反射调用，略有不同，示例如下：

```go
// 反射方法操作
for i := 0; i < v.NumMethod(); i++ {
    // method := t.Method(i) // 获取方法信息对象，方法 1(type反射)
    // mt := method.Type     // 获取方法信息 Type 对象，方法 1
    // fmt.Println("method name:", method.Name)
    
    
    m := v.Method(i) // 获取方法信息对象，方法 2 (value反射)
    mt := m.Type()   // 获取方法信息 Type 对象，方法 2

    in := []reflect.Value{}

    // 获取方法入参类型
    for j := 0; j < mt.NumIn(); j++ {
        fmt.Println("method in type:", mt.In(j))
        if mt.In(j).Kind() == reflect.String {
            in = append(in, reflect.ValueOf("welcome"))
        }
        // 方法 1 获取的方法信息对象会把方法的接受者也当着入参之一
        if mt.In(j).Name() == t.Name() {
            in = append(in, v)
        }
    }

    // 获取方法返回类型
    for j := 0; j < mt.NumOut(); j++ {
        fmt.Println("method out type:", mt.Out(j))
    }

    // 反射方法调用
    // out := method.Func.Call(in) // 方法 1 获取的 Method 对象反射调用方式
    out := m.Call(in) 						// 方法 2 获取的 Method 对象反射调用方式
    for _, o := range out {
        fmt.Println("out:", o)
    }
}
```



## 五、反射对象 Value 还原

通过 `reflect.ValueOf()` 可以把任意类型对象转换为 `Value` 类型对象，也可以通过 `Value` 类型的方法 `Interface()` 把 `Value` 类型对象还原为原始数据类型对象。

当执行`reflect.ValueOf(interface)`之后，就得到了一个类型为`relfect.Value`变量，可以通过它本身的`Interface()`方法获得接口变量的真实内容，然后可以通过类型判断进行转换，转换为原有真实类型。不过，我们可能是已知原有类型，也有可能是未知原有类型，因此，下面分两种情况进行说明。

已知原有类型

```go
// Value 转原始类型
if u1, ok := v.Interface().(User); ok {
    fmt.Println("after:", u1.Name, u1.Age)
}


func main() {
    var num float64 = 1.2345

    pointer := reflect.ValueOf(&num)
    value := reflect.ValueOf(num)

    // 可以理解为“强制转换”，但是需要注意的时候，转换的时候，如果转换的类型不完全符合，则直接panic
    // Golang 对类型要求非常严格，类型一定要完全符合
    // 如下两个，一个是*float64，一个是float64，如果弄混，则会panic
    convertPointer := pointer.Interface().(*float64)
    convertValue := value.Interface().(float64)

    fmt.Println(convertPointer)
    fmt.Println(convertValue)
}

运行结果：
0xc42000e238
1.2345
```

未知原有类型

```go
// 通过接口来获取任意参数，然后一一揭晓
func DoFiledAndMethod(input interface{}) {
	getType := reflect.TypeOf(input)
	fmt.Println("get Type is :", getType.Name())

	getValue := reflect.ValueOf(input)
	fmt.Println("get all Fields is:", getValue)

	// 获取方法字段
	// 1. 先获取interface的reflect.Type，然后通过NumField进行遍历
	// 2. 再通过reflect.Type的Field获取其Field
	// 3. 最后通过Field的Interface()得到对应的value
	for i := 0; i < getType.NumField(); i++ {
		field := getType.Field(i)
		value := getValue.Field(i).Interface()
		fmt.Printf("%s: %v = %v\n", field.Name, field.Type, value)
	}

	// 获取方法
	// 1. 先获取interface的reflect.Type，然后通过.NumMethod进行遍历
	for i := 0; i < getType.NumMethod(); i++ {
		m := getType.Method(i)
		fmt.Printf("%s: %v\n", m.Name, m.Type)
	}
}
```



### 参考文章

* [Golang的反射reflect深入理解和示例](https://juejin.im/post/6844903559335526407)
* [Golang 反射使用总结](https://ehlxr.me/2018/01/26/golang-reflect/)

