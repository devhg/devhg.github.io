---
title: Go 基础之反射
permalink: go-reflect
date: 2020-06-14 13:02:09
toc: true
tags:
 - Golang
categories:
 - Go基础
---

众所周知，反射是框架设计的灵魂。反射在很多语言中都有其妙用。在计算机科学领域，反射是指一类应用，它们能够**自描述**和**自控制**。本文将对于Golang的反射的笔记。

<!--more-->

### 反射的用途

Golang提供了一种机制，在编译时不知道类型的情况下，可更新变量、运行时查看值、调用方法以及直接对他们的布局进行操作的机制，称为反射。

### 为什么用反射

目的就是增加程序的灵活性，避免将程序写死在代码里。借助反射透视一个未知的类型。

> [为何需要反射?](http://shouce.jb51.net/gopl-zh/ch12/ch12-01.html)



### 使用反射

reflect提供了两种类型来进行访问接口变量的内容

|First Header  | Second Header |
| ------------- | :-----------: |
| reflect.ValueOf() | 获取输入参数接口中的数据的值，如果为空则返回**0** <- 注意是0 |
| reflect.TypeOf() | 动态获取输入参数接口中的值的类型，如果为空则返回**nil** <- 注意是nil |



#### 简单的反射

```go
func main() {
	var name string = "我是反射"
	//TypeOf会返回目标数据的类型，比如int/float/struct/指针等
	reflectType := reflect.TypeOf(name)

	//valueOf返回目标数据的的值，比如上文的"我是反射"
	reflectValue := reflect.ValueOf(name)

	fmt.Println("type: ", reflectType)
	fmt.Println("value: ", reflectValue)
}
//输出：
//type:  string
//value:  我是反射
```

#### 结构体的反射

```go
type User struct {
	Id   int
	Name string
}

func (u User) Hello() {
	fmt.Println("hello")
}

func TestReflect(t interface{}) {
	objT := reflect.TypeOf(t)
	objV := reflect.ValueOf(t)

	//获取去这个类型的名称
	fmt.Println("这个类型的名称是:", objT.Name())
  
	//通过.NumField()来获取结构体里的字段数量
	for i := 0; i < objT.NumField(); i++ {
		//从0开始获取结构体所包含的key
		key := objT.Field(i)
		//从0开始通过interface方法来获取key所对应的值
		value := objV.Field(i).Interface()

		fmt.Printf("第%d个字段是：%s:%v = %v \n", i+1, key.Name, key.Type, value)
	}
	//通过.NumMethod()来获取结构体里的方法数量  这里只能获取 值接收器的方法。
	for i := 0; i < objT.NumMethod(); i++ {
		m := objT.Method(i)
		fmt.Printf("第%d个方法是：%s:%v\n", i+1, m.Name, m.Type)
	}

}
func main() {
	u := User{
		Id:   1,
		Name: "反射",
	}
	TestReflect(u)
}
```

输出：

```
这个类型的名称是: User
第1个字段是：Id:int = 1 
第2个字段是：Name:string = 反射 
第1个方法是：Hello:func(main.User)
```

#### 匿名或嵌入字段的反射

```go
type User struct {
	Student //匿名字段
}

type Student struct {
	Id   int
	Name string
}

func main() {
	u := User{Student{
		Id:   1,
		Name: "反射",
	}}

	t := reflect.TypeOf(u)
	//这里需要加一个#号，可以把struct的详情都给打印出来
	//会发现有Anonymous:true，说明是匿名字段
	fmt.Printf("%#v\n", t.Field(0))

	fmt.Printf("%#v\n", t.FieldByIndex([]int{0, 1}))

	//获取匿名字段的值的详情
	v := reflect.ValueOf(u)
	fmt.Printf("%#v\n", v.Field(0))
}
```

输出：

```
reflect.StructField{Name:"Student", PkgPath:"", Type:(*reflect.rtype)(0x10bb640), Tag:"", Offset:0x0, Index:[]int{0}, Anonymous:true}
reflect.StructField{Name:"Name", PkgPath:"", Type:(*reflect.rtype)(0x10ae120), Tag:"", Offset:0x8, Index:[]int{1}, Anonymous:false}
main.Student{Id:1, Name:"反射"}
```

#### 用反射判断传入的类型

```go
type Student struct {
	Id   int
	Name string
}

func main() {
	s := Student{Id: 1, Name: "反射"}
	t := reflect.TypeOf(s)

	//通过.Kind()来判断对比的值是否是struct类型
	if k := t.Kind(); k == reflect.Struct {
		fmt.Println("bingo")
	}

	num := 1;
	numType := reflect.TypeOf(num)
	if k := numType.Kind(); k == reflect.Int {
		fmt.Println("bingo")
	}
}
/*输出：
bingo
bingo
*/
```



#### 通过反射修改内容

```go
type User struct {
	Student //匿名字段
}

type Student struct {
	Id   int
	Name string
}

func main() {
	u := &User{Student{
		Id:   1,
		Name: "反射",
	}}

	v := reflect.ValueOf(u)
	//修改值必须是指针类型
	if v.Kind() != reflect.Ptr {
		fmt.Println("不是指针类型，无法进行修改操作")
		return
	}
	//获取指针所指向的元素
	v = v.Elem()

	fmt.Printf("%#v\n", *u)
	name := v.FieldByName("Name")
	if name.Kind() == reflect.String {
		name.SetString("小学生")
	}
	fmt.Printf("%#v\n", *u)

	//如果是整型的话
	test := 888
	testV := reflect.ValueOf(&test)
	testV.Elem().SetInt(666)
	fmt.Println(test)
}

```

输出：

```
main.User{Student:main.Student{Id:1, Name:"反射"}}
main.User{Student:main.Student{Id:1, Name:"小学生"}}
666
```



#### 通过反射调用方法

```go
type Student struct {
	Id   int
	Name string
}

func (s Student) Hello(msg string) {
	fmt.Println("hello, ", msg)
}

func main() {
	u := Student{
		Id:   1,
		Name: "反射",
	}
	v := reflect.ValueOf(u)

	//获取方法控制权 返回v的名为name的方法的已绑定（到v的持有值的）状态的函数形式的Value封装
	method := v.MethodByName("Hello")
	//拼凑参数
	args := []reflect.Value{reflect.ValueOf("反射")}
	//调用函数
	method.Call(args)
}
```

输出：

```
hello,  反射
```

 

### 小结

上述详细说明了Golang的反射reflect的各种功能和用法，都附带有相应的示例，相信能够在工程应用中进行相应实践，总结一下就是：

- 反射可以大大提高程序的灵活性，使得interface{}有更大的发挥余地
  - 反射必须结合interface才玩得转
  - 变量的type要是concrete type的（也就是interface变量）才有反射一说
- 反射可以将“接口类型变量”转换为“反射类型对象”
  - 反射使用 TypeOf 和 ValueOf 函数从接口中获取目标对象信息
- 反射可以将“反射类型对象”转换为“接口类型变量
  - reflect.value.Interface().(已知的类型)
  - 遍历reflect.Type的Field获取其Field
- 反射可以修改反射类型对象，但是其值必须是“addressable”
  - 想要利用反射修改对象状态，前提是 interface.data 是 settable,即 pointer-interface
- 通过反射可以“动态”调用方法
- 因为Golang本身不支持模板，因此在以往需要使用模板的场景下往往就需要使用反射(reflect)来实现

Golang的反射很慢，这个和它的API设计有关。Golang reflect慢主要有两个原因

1. 涉及到内存分配以及后续的GC；
2. reflect实现里面有大量的枚举，也就是for循环，比如类型之类的。

### 参考文章

* [Go语言圣经](http://shouce.jb51.net/gopl-zh/ch12/ch12.html)
* [标准库文档](https://studygolang.com/pkgdoc)

