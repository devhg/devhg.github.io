---
title: Go语言的omitempty
permalink: go-omitempty
toc: true
date: 2020-12-07 19:04:39
tags:
 - Golang
categories:
 - Go基础
---

<!--more-->

### 使用

熟悉 Golang 的朋友对于 tag、json 和 struct 都不陌生。

```go
type Address struct {
	City   string `json:"city"`
	Street string `json:"street"`
	ZipCode string `json:"zip_code"`
}

func TestMarshal(t *testing.T) {
	data := `{
        "city": "Beijing",
        "street": "a"
    }`
	addr := &Address{}
	json.Unmarshal([]byte(data), addr)
	fmt.Printf("%#v\n", addr)

	addressBytes, _ := json.MarshalIndent(addr, "", "    ")
	fmt.Printf("%s\n", string(addressBytes))
}
/*
&omitempty.Address{City:"Beijing", Street:"a", ZipCode:""}
{
    "city": "Beijing",
    "street": "a",
    "zip_code": ""
}
*/
```

我们可以看到，多了一行 `"zip_code": "",` ，而这则信息在原本的 json 数据中是没有的，但我们更希望的是，在一个地址有 zip_code 号码的时候输出，不存在 zip_code 的时候就不输出，幸运的是，我们可以在 Golang 的结构体定义中添加 `omitempty` 关键字，来表示这条信息如果没有提供，在序列化成 json 的时候就不要包含其默认值。稍作修改，地址结构体就变成了

```go
type Address struct {
	City    string `json:"city"`
	Street  string `json:"street"`
	ZipCode string `json:"zip_code,omitempty"`
}

func TestMarshal(t *testing.T) {
	data := `{
        "city": "Beijing",
        "street": "a"
    }`
	addr := &Address{}
	json.Unmarshal([]byte(data), addr)
	fmt.Printf("%#v\n", addr)

	addressBytes, _ := json.MarshalIndent(addr, "", "    ")
	fmt.Printf("%s\n", string(addressBytes))
}
/*
&omitempty.Address{City:"Beijing", Street:"a", ZipCode:""}
{
    "city": "Beijing",
    "street": "a"
}
*/
```

成功解决。



<hr>

### 陷阱

带来方便的同时，使用 `omitempty` 也有些小陷阱。

#### 陷阱一：关键字无法忽略掉嵌套结构体

还是拿地址类型说事，这回我们想要往地址结构体中加一个新的结构体来表示经纬度，如果没有或缺乏相关的数据，可以忽略。新的 struct 定义如下所示

```go
type Address struct {
	City       string     `json:"city"`
	Street     string     `json:"street"`
	ZipCode    string     `json:"zip_code,omitempty"`
	Coordinate coordinate `json:"coordinate,omitempty"`
}

type coordinate struct {
	Lat float64 `json:"latitude"`
	Lng float64 `json:"longitude"`
}

func TestMarshal(t *testing.T) {
	data := `{
        "city": "Beijing",
        "street": "a"
    }`
	addr := &Address{}
	json.Unmarshal([]byte(data), addr)

	addressBytes, _ := json.MarshalIndent(addr, "", "    ")
	fmt.Printf("%s\n", string(addressBytes))
}
/*
{
    "city": "Beijing",
    "street": "a",
    "coordinate": {
        "latitude": 0,
        "longitude": 0
    }
}
*/
```

读入原来的地址数据，处理后序列化输出，我们就会发现**即使对新的结构体字段加上了 `omitempty` 关键字，输出的 json 还是带上了一个空的坐标信息。**

为了达到我们想要的效果，可以把**坐标结构体定义为指针类型**，这样 Golang 就能知道一个指针的“空值”是多少了，否则面对一个我们自定义的结构， Golang 是猜不出我们想要的空值的。于是有了如下的结构体定义：

```go
type Address struct {
	City       string      `json:"city"`
	Street     string      `json:"street"`
	ZipCode    string      `json:"zip_code,omitempty"`
	Coordinate *coordinate `json:"coordinate,omitempty"`
}

type coordinate struct {
	Lat float64 `json:"latitude"`
	Lng float64 `json:"longitude"`
}
// ... 同上 忽略
/*
{
    "city": "Beijing",
    "street": "a"
}
*/
```



#### 2. 陷阱二：想要传入零值

对于用 `omitempty` 定义的 字段 ，**如果给它赋的值恰好等于默认空值的话**，在转为 json 之后也不会输出这个 字段 。比如说上面定义的经纬度坐标结构体，如果我们将经纬度两个 字段 都加上 `omitempty`

```go
type coordinate struct {
	Lat float64 `json:"latitude,omitempty"`
	Lng float64 `json:"longitude,omitempty"`
}

func TestMarshal(t *testing.T) {
	data := `{
        "latitude": 1.0,
        "longitude": 0.0
    }`
	c := &coordinate{}
	json.Unmarshal([]byte(data), c)
	fmt.Printf("%#v\n", c)

	addressBytes, _ := json.MarshalIndent(c, "", "    ")
	fmt.Printf("%s\n", string(addressBytes))
}
/*
&omitempty.coordinate{Lat:1, Lng:0}
{
    "latitude": 1
}
*/
```

这个坐标的`longitude`消失不见了！

但我们的设想是，如果一个地点没有经纬度信息，则悬空，这没有问题，但对于“原点坐标”，我们在确切知道它的经纬度的情况下，（0.0, 0.0）仍然被忽略了。正确的写法也是将结构体内的定义改为指针

```go
type coordinate struct {
    Lat *float64 `json:"latitude,omitempty"`
    Lng *float64 `json:"longitude,omitempty"`
}
/*
&omitempty.coordinate{Lat:(*float64)(0xc0000a6288), Lng:(*float64)(0xc0000a6298)}
{
    "latitude": 1,
    "longitude": 0
}
*/
```

这样空值就从 `float64` 的 0.0 变为了指针类型的 `nil` ，我们就能看到正确的经纬度输出。





* [Golang 的 “omitempty” 关键字略解](https://old-panda.com/2019/12/11/golang-omitempty/)

