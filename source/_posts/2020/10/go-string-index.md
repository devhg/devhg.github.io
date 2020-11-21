---
title: Go中下标访问字符串问题及string类型如何存储
permalink: go-string-index
toc: true
date: 2020-10-24 19:10:33
tags:
 - Golang
 - 编码
categories:
 - Go基础
---

在大多数语言中，字符串是可以通过下标访问的，但是在go语言中，有些时候却不能做到这样。

<!--more-->

### 分析

```go
func main() {
	s := "12as我"
	fmt.Printf("%c", s[4])
}
// æ
```

在解决这个问题之前，要先了解一下数组：数组是用于存储多个相同类型数据的集合。并且数组在申请内存的时候，是一次申请一块连续的内存。比如我们创建一个数组，里面存了这几个元素。

由于内存是连续的，元素的类型也是相同的，每一个元素所占用的储存空间也是固定的，比如Java中的char类型占用两个字节。数组的内存空间是平等划分的。



在可以用下标访问的语言中，字符串都是**按照字符编码**的。也就是将字符串` "abcd"` 赋给变量 a，本质上是创建了一个字符数组`char[]`用来存放字符串。每一个字符占用的空间相同。

但是go语言中，字符串是**按照字节编码**的。 26 个英文字母，数字等一些字符，在 go 语言的 string 里面就占用一个字节。而对于中文日文韩文就不一样了， go 语言内建只支持 utf8 编码，在 utf8 里面，有一部分汉字占用 3 个字节，一部分汉字占用 4 个字节。比如 `"1我"` 这个字符串，打印一下它的长度，发现打印了4。这是 string 占用 4 个字节，字符"1"占用一个字节，加上"我"之后占用 4 个字节，数字占用一个字节，我占用3个字节。这样应该能理解按字节编码的意思了。



```go
func main() {
	s := "1我"
	fmt.Println(len(s))
}
// 4
```



### 原因

为什么go语言要采用字节来编码呢？是为了节省空间。在utf8编码中，一些中文字符占用3个字节，有一些要占用4个字节，而英文字母只需要占用1个字节。如果采用按照字符编码的形式，一个中文算一个字符，一个英文字母也算一个字符，但是占用的内存相差很大，假设有一个超长字符串，里面有英文字符远多于中文字符，如果按字符来存储，每个字符要分配四个字节。每个字符分配四个字节是因为低于四个字节，有可能有些中文就不能正常存储了，在这种情况下，每存储一个英文字母，就要浪费三个字节的内存空间。

```go
func main() {
	s := "1我"
  fmt.Println(s[0], s[1])
	fmt.Println(s[0], s[1:])
}
// 49 230
// 49 我

//s[0] 是 49，等于字符1 的 ascii 码，s[1] 是 230，显然不会是汉字 "我" 的 utf8，事实它是 utf8 编码的第一字节的值。
```



### 解决

go语言提供了另一种方式`rune`类型来实现游标访问字符串

golang中还有一个byte数据类型与rune相似，它们都是用来表示字符类型的变量类型。它们的不同在于：

- byte 等同于int8，常用来处理ascii字符
- rune 等同于int32,常用来处理unicode或utf-8字符

```go
// byte is an alias for uint8 and is equivalent to uint8 in all ways. It is
// used, by convention, to distinguish byte values from 8-bit unsigned
// integer values.
type byte = uint8

// rune is an alias for int32 and is equivalent to int32 in all ways. It is
// used, by convention, to distinguish character values from integer values.
type rune = int32
```



```go
func main() {
	s := "1我"
	fmt.Println(s[0], s[1])
	fmt.Println(s[0], s[1:])

	// 解决
	fmt.Println("real len=", utf8.RuneCount([]byte(s)))
	fmt.Println("s[0]=", Index(s, 0))
	fmt.Println("s[1]=", Index(s, 1))
 
  // 或者
	runes := []rune(s)
  fmt.Println(len(runes))
	fmt.Printf("s[0]=%c\n", runes[0])
	fmt.Printf("s[1]=%c\n", runes[1])
  
  // 或者
  //s := "l我"
	//for _, v := range s {
	//	fmt.Printf("%T ", v)
	//	fmt.Printf("%c\n", v)
	//}
  /*
  int32 l
	int32 我
	*/
}
// 下标访问字符串
func Index(s string, index uint) string {
	runes := bytes.Runes([]byte(s))
	for i, rune := range runes {
		if i == int(index) {
			return string(rune)
		}
	}
	return ""
}
/*
49 230
49 我
real len= 2
s[0]= 1
s[1]= 我
2
s[0]=1
s[1]=我
*/
```



### UTF-8 和 Unicode 有何区别？

Unicode 与 ASCII 类似，都是一种字符集。

字符集为每个字符分配一个唯一的 ID，我们使用到的所有字符在 Unicode 字符集中都有一个唯一的 ID，例如上面例子中的 a 在 Unicode 与 ASCII 中的编码都是 97。汉字“你”在 Unicode 中的编码为 20320，在不同国家的字符集中，字符所对应的 ID 也会不同。而无论任何情况下，Unicode 中的字符的 ID 都是不会变化的。

UTF-8 是编码规则，将 Unicode 中字符的 ID 以某种方式进行编码，UTF-8 的是一种**变长编码规则**，从 1 到 4 个字节不等。编码规则如下：

- 0xxxxxx 表示文字符号 0～127，兼容 ASCII 字符集。
- 从 128 到 0x10ffff 表示其他字符。

```
Unicode符号范围     |        UTF-8编码方式
(十六进制)          |            （二进制）
----------------------+---------------------------------------------
0000 0000-0000 007F | 0xxxxxxx
0000 0080-0000 07FF | 110xxxxx 10xxxxxx
0000 0800-0000 FFFF | 1110xxxx 10xxxxxx 10xxxxxx
0001 0000-0010 FFFF | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx
```

跟据上表，解读 UTF-8 编码非常简单。如果一个字节的第一位是`0`，则这个字节单独就是一个字符；如果第一位是`1`，则连续有多少个`1`，就表示当前字符占用多少个字节。

下面，还是以汉字`严`为例，演示如何实现 UTF-8 编码。

`严`的 Unicode 是`4E25`（`100111000100101`），根据上表，可以发现`4E25`处在第三行的范围内（`0000 0800 - 0000 FFFF`），因此`严`的 UTF-8 编码需要三个字节，即格式是`1110xxxx 10xxxxxx 10xxxxxx`。然后，从`严`的最后一个二进制位开始，依次从后向前填入格式中的`x`，多出的位补`0`。这样就得到了，`严`的 UTF-8 编码是`11100100 10111000 10100101`，转换成十六进制就是`E4B8A5`。 (转自[阮一峰 - 字符编码笔记：ASCII，Unicode 和 UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html))



根据这个规则，拉丁文语系的字符编码一般情况下每个字符占用一个字节，而中文每个字符占用 3 个字节。

广义的 Unicode 指的是一个标准，它定义了字符集及编码规则，即 Unicode 字符集和 UTF-8、UTF-16 编码等。



### golang string如何存储

<img src="https://i.loli.net/2020/11/14/RI8arN36BMYCovi.jpg" alt="gostring.png" style="zoom:50%;" />

在c语言中，字符串以一个`\0`结束一个字符串。而在go语言中不是这样的。

上源码

```go
type stringStruct struct {
	str unsafe.Pointer // 指向一个 [len]byte 的数组
	len int						 // 字节数组长度
}
```

如上就是go语言的string结构体，string类型是一个不可变类型，那么任何对string的修改都会新生成一个string的实例，如果是考虑效率的场景就要好好考虑一下如何修改了。

只能用下标访问一些特殊的字符；不能直接修改字符串；字符串转字节切片会重新分配一块内存。

```go
s1 := "111我"
s2 := s1[1:]
// s2 和 s1指向同一块内存
```



推荐文章

* [阮一峰 - 字符编码笔记：ASCII，Unicode 和 UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)

