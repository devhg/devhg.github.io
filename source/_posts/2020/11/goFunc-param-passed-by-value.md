---
title: Go语言参数传递是传值还是传引用
permalink: goFunc-param-passed-by-value
toc: true
date: 2020-11-12 09:52:31
tags:
 - Golang
categories:
 - Go基础
---

对于了解一门语言来说，会关心我们在函数调用的时候，参数到底是传的值，还是引用？其实对于传值和传引用，是一个比较常见的话题，我们必须非常清楚。对于我们做Go语言开发的来说，也必须知道到底是什么传递。

<!--more-->

## 什么是传值（值传递）

传值的意思是：函数传递的总是原来这个东西的一个副本（拷贝）。比如我们传递一个`int`类型的参数，传递的其实是这个参数的一个副本；传递一个指针类型的参数，其实传递的是这个该指针的一份拷贝，而不是这个指针。

对于`int float`等这类基础类型我们可以很好的理解，它们就是一个拷贝。但是指针呢？我们觉得可以通过它修改原来的值，怎么会是一个拷贝呢？下面我们看个例子。

```go
func modify(ptr *int) {
	fmt.Printf("modify指针存储的地址：%p\n", ptr)
	fmt.Printf("形参指针的地址：%p\n", &ptr)
	*ptr = 233
}

func main() {
	i := 10
	iPtr := &i
	fmt.Printf("main指针存储的地址：%p\n", iPtr)
	fmt.Printf("实参指针的地址：%p\n", &iPtr)
	modify(iPtr)
	fmt.Printf("int值被修改了，新值为：%d\n", i)
}
/*
main指针存储的地址：0xc0000180a8
实参指针的地址：0xc00000e028
modify指针存储的地址：0xc0000180a8
形参指针的地址：0xc00000e038
int值被修改了，新值为：233
*/
```

首先我们要知道，任何存放在内存里的东西都有自己的地址，指针也不例外，它虽然指向别的数据，但是也有存放该指针的内存。所以通过输出我们可以看到，传递指针参数是传的一个指针值的拷贝，实参形参虽然是不同的指针，但他们两个都存储了相同的地址值，即变量的`i`的地址。

<img src="https://cdn.jsdelivr.net/gh/QXQZX/CDN@1.0.4/images/go/goFunc-param-passed-by-value.png" alt="1" style="zoom:50%;" />



通过上面的图，可以更好的理解。 首先我们看到，我们声明了一个变量`i`,值为`10`,它的内存存放地址是`0xc0000180a8`,通过这个内存地址，我们可以找到变量`i`。

指针`iPtr`也是一个指针类型的变量，它存放了`i`的地址，这个指针本身内存地址是`0xc00000e028`。 在我们传递指针变量`iPtr`给`modify`函数的时候，是该指针变量的拷贝，所以新拷贝的指针变量`ptr`，它的内存地址已经变了，是新的`0xc00000e038`。

虽然形参和实参的地址不同，但我们都可以称之为指针的指针，他们存储了同一个地址，即变量`i`的地址，这也就是为什么我们可以修改变量`i`的值的原因。

<hr>

## 什么是传引用(引用传递)

Go语言是没有引用传递的，这里我不能使用Go举例子，但是可以通过说明描述。

以上面的例子为例，如果在`modify`函数里打印出来的形参和实参的内存地址是一样的，即`&iPtr == &ptr`，那么就是引用传递。



### map类型

了解清楚了传值和传引用，但是对于Map类型来说，可能觉得还是迷惑，一我们可以通过方法修改它的内容，二它没有明显的指针。

```go
func modify(m map[int]string) {
	fmt.Printf("modify中map的内存地址：%p\n", &m)
	m[1] = "李四"
}

func main() {
	m := make(map[int]string)
	m[1] = "张三"
	fmt.Printf("main中map的内存地址是：%p\n", &m)
	modify(m)
	fmt.Printf("map值被修改了，新值为：%s\n", m[1])
}
/*
main中map的内存地址是：0xc0000ae018
modify中map的内存地址：0xc0000ae028
map值被修改了，新值为：李四
*/
```

两个内存地址是不一样的，所以这又是一个值传递（值的拷贝），那么为什么我们可以修改Map的内容呢？



先不急，我们先看一个自己实现的`struct`。

```go
func modify(p Person) {
	fmt.Printf("modify中Person的内存地址：%p\n", &p)
	p.Name = "李四"
}
func main() {
	p := Person{"张三"}
	fmt.Printf("main中Person的内存地址是：%p\n", &p)
	modify(p)
	fmt.Println(p)
}
/*
main中Person的内存地址是：0xc000010200
modify中Person的内存地址：0xc000010210
{张三}
*/
```



我们发现，我们自己定义的`Person`类型，在函数传参的时候也是值传递，但是它的值`Name`字段并没有被修改，我们想改成`李四`，发现最后的结果还是`张三`。

这也就是说，`map`类型和我们自己定义的`struct`类型是不一样的。我们尝试把`modify`函数的接收参数改为`Person`的指针。

```go
type Person struct {
	Name string
}
func modify(p *Person) {
	fmt.Printf("modify中Person的内存地址：%p\n", &p)
	p.Name = "李四"
}
func main() {
	p := Person{"张三"}
	fmt.Printf("main中Person的内存地址是：%p\n", &p)
	modify(&p)
	fmt.Println(p)
}
/*
main中Person的内存地址是：0xc000010200
modify中Person的内存地址：0xc00000e030
{李四}
*/
```

我们发现，这次被修改了。我们这里内存地址的不再解释，因为我们上面`int`类型的例子已经证明了指针类型的参数也是值传递的。 指针类型可以修改，非指针类型不行，那么我们可以大胆的猜测，我们使用`make`函数创建的`map`是不是一个指针类型呢？看一下源代码:

```go
// makemap implements Go map creation for make(map[k]v, hint).
// If the compiler has determined that the map or the first bucket
// can be created on the stack, h and/or bucket may be non-nil.
// If h != nil, the map can be created directly in h.
// If h.buckets != nil, bucket pointed to can be used as the first bucket.
func makemap(t *maptype, hint int, h *hmap) *hmap {
    //...
}
```

通过查看`src/runtime/map.go`源代码发现，的确和我们猜测的一样，`make`函数返回的是一个`hmap`类型的指针`*hmap`。也就是说`map==*hmap`。 现在看`func modify(p map[][])`这样的函数，其实就类似于`func modify(p *hmap)`，但我们不能这样去写。这和我们前面**什么是值传递**里举的`func modify(ip *int)`的例子一样，可以参考分析。

所以在这里，Go语言通过`make`函数，字面量的包装，为我们省去了指针的操作，让我们可以更容易的使用map。这里的`map`可以理解为引用类型，但是记住引用类型不是传引用。



### chan类型

`chan`类型本质上和`map`类型是一样的，这里不做过多的介绍，参考下源代码:

```go
func makechan(t *chantype, size int) *hchan {
    //...
}
```

`chan`也是一个引用类型，和`map`相差无几，`make`返回的是一个`*hchan`。



<hr>

### slice类型

`slice`和`map`、`chan`都不太一样的，一样的是，它也可以在函数中修改对应的内容。

```go
func modify(nums []int) {
	fmt.Printf("%p\n", nums)     // num 中真实存储的地址
	fmt.Printf("%p\n", &nums[0]) // 地址nums==&nums[0]==a== &a[0]

	fmt.Printf("%p\n", &nums)    // num 的地址
	nums[0] = 2333
}

func main() {
	a := []int{6, 6, 6}
	fmt.Printf("%p\n", &a)    // slice 的地址
	fmt.Printf("%p\n", a)     // slice 中真实存储的地址
	fmt.Printf("%p\n", &a[0]) // 地址a == &a[0]

	modify(a)
	fmt.Println(a)
}
/*
0xc0000a6020
0xc0000b8000
0xc0000b8000
0xc0000b8000
0xc0000b8000
0xc0000a6060
[2333 6 6]
*/
```

运行打印结果，发现slice的确是被修改了，而且我们这里打印`slice`的内存地址是可以直接通过`%p`打印的，不用使用`&`取地址符转换。并且地址关系是`nums==&nums[0]==a== &a[0]`

这就可以证明`make`的slice也是一个指针了吗？不一定，也可能`fmt.Printf`把`slice`特殊处理了。



```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
//...
func makeslice(et *_type, len, cap int) unsafe.Pointer {
	//...
}
```

通过查看`src/runtime/slice.go`源代码发现，对于`chan`、`map`、`slice`等被当成指针处理，通过`value.Pointer()`获取对应的值的指针。

```go
// If v's Kind is Slice, the returned pointer is to the first
// element of the slice. If the slice is nil the returned value
// is 0.  If the slice is empty but non-nil the return value is non-zero.
func (v Value) Pointer() uintptr {
	// TODO: deprecate
	k := v.kind()
	switch k {
	//...
	case Slice:
		return (*SliceHeader)(v.ptr).Data
	}
}
```

很明显了，当是`slice`类型的时候，返回是`slice`这个结构体里，字段Data第一个元素的地址。



所以我们通过`%p`打印的`slice`变量的地址其实就是内部存储数组元素的地址，`slice`是一种结构体+元素指针的混合类型，通过元素`array`的指针，可以达到修改`slice`里存储元素的目的。

所以修改类型的内容的办法有很多种，类型本身作为指针可以，类型里有指针类型的字段也可以。

单纯的从`slice`这个结构体看，我们可以通过`modify`修改存储元素的内容，**但是永远修改不了`len`和`cap`**，因为他们只是一个拷贝，不是指针，如果要修改，那就要传递`*slice`作为参数才可以。

<hr>

下面通过这个`Person`和`slice`对比，以便于更好理解。

```go
type Person struct {
	Name string
	Age  *int
}

func (p Person) String() string {
	return "姓名为：" + p.Name + ",年龄为：" + strconv.Itoa(*p.Age)
}

func modify(p Person) {
	p.Name = "李四" // 无法修改
	*p.Age = 2333
}

func main() {
	i := 19
	p := Person{Name: "张三", Age: &i}
	fmt.Println(p)
	modify(p)
	fmt.Println(p)
}
/*
姓名为：张三,年龄为：19
姓名为：张三,年龄为：2333
*/
```

`Person`的`Name`字段就类似于`slice`的`len`或者`cap`字段，`Age`类似于`slice`的`array`字段。在传参为非指针类型的情况下，可以修改`Age`字段，`Name`字段无法被修改。这就是`slice`可以修改值，而不可以更改容量和长度问题的原因所在。要修改`Name`字段，就要把传参改为指针，伪代码比如：

```go
modify(&p)
func modify(p *Person){
	p.name = "李四" // 可以修改
	*p.age = 2333
}
/*
姓名为：张三,年龄为：19
姓名为：李四,年龄为：2333
*/
```

这样`name`和`age`字段双双都被修改了。

所以**slice**能够通过函数传参后，修改对应的数组值，是因为 slice 内部保存了引用数组的指针，并不是因为引用传递。

<hr>

## 小结

最终我们可以确认的是Go语言中所有的传参都是值传递（传值），都是一个副本。但是类型引用有引用类型，他们是：**slice**、**map**、**channel**。

因为拷贝的内容有时候是非引用类型（`int`、`float`、`string`、`struct`等这些），这样在函数中就无法修改原内容数据；有的是引用类型（`pointer`、`map`、`slice`、`chan`等这些），这样就可以修改原内容数据。

是否可以修改原内容数据，和传值、传引用没有必然的关系。在C++中，传引用肯定是可以修改原内容数据的，在Go语言里，虽然只有传值，但是我们也可以修改原内容数据，因为值类型的某个字段是引用类型。

这里也要记住，引用类型和传引用是两个概念。

再记住，Go里只有传值（值传递）。

