---
title: 深入理解Go的slice及其扩容规则
permalink: go-slice
toc: true
date: 2020-11-14 19:46:32
tags:
 - Golang
categories:
 - Go底层原理
---

在 Go 中，Slice（切片）是抽象在 Array（数组）之上的特殊类型。为了更好地了解 Slice，第一步需要先对 Array 进行理解。深刻了解 Slice 与 Array 之间的区别后，就能更好的对其底层实现更好地理解。

<!--more-->

先通过源码查看一下go的slice底层

```go
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}
```

`slice`类型包括三个字段

- array：指向所引用的数组指针（`unsafe.Pointer` 可以表示任何可寻址的值的指针）
- len：长度，当前引用切片的元素个数
- cap：容量，当前引用切片的容量（底层数组的元素总数）



关于切片底层还是移步这个视频吧，实在不想去画图长篇论述，这个up已经讲得很好了。我这篇文章只是想总结一下切片扩容。

<iframe width="700px" height="500px" src="//player.bilibili.com/player.html?aid=413046944&bvid=BV1CV411d7W8&cid=205107628&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>



## slice扩容

### 1. 预估扩容后容量

```go
ints := []int{1, 2} 			// 扩容前 oldCap = 2
ints = append(ints, 3,4,5) // newCap = ?  5?
```

预估扩容规则

* 如果旧的容量乘以2  小于  所需最小容量，那么容量`newCap`就等于所需最小容量
* 如果不符合上面的情况，也就是说旧的容量乘以2  大于等于  所需最小容量。接下来判断长度`len`。如果旧的长度小于等于1024，那么直接在旧的容量基础上翻倍扩容；如果旧的长度大于等于1024，先在旧的基础上扩1.25倍，循环往复扩充。



源代码

```go
func growslice(et *_type, old slice, cap int) slice {
  //...
  newcap := old.cap
  doublecap := newcap + newcap
  if cap > doublecap {
    newcap = cap
  } else {
    if old.len < 1024 {
      newcap = doublecap
    } else {
      // Check 0 < newcap to detect overflow
      // and prevent an infinite loop.
      for 0 < newcap && newcap < cap {
        newcap += newcap / 4
      }
      // Set newcap to the requested cap when
      // the newcap calculation overflowed.
      if newcap <= 0 {
        newcap = cap
      }
    }
  }
  //...
}
```



### 2. 匹配到合适的内存规格

`所需内存 = 预估容量 * 元素类型大小`

比如预估容量是3个，int类型，那么所需内存就是24，但实际分配的时候并不是直接向系统索要24个字节的内存。而是想go本身的内存管理模块申请。

Go语言的内存管理模块将内存分成了大大小小67个级别的span，**其中0级代表特殊的大对象，其大小是不固定的**。当具体的对象需要分配内存时，并不是直接分配span，而是分配不同级别的span中的元素。因此span的级别也不是以每个span大小为依据，而是以span中元素的大小为依据。

```go
	 span等级   元素大小   span大小  对象个数
    1          8        8192     1024       
    2         16        8192      512        
    3         32        8192      256          
    4         48        8192      170       
    5         64        8192      128  
		...
    65      28672       57344      2   
    66      32768       32768      1
```

申请内存时，内存管理模块会帮我们选足够大且最接近的内存规格，所以上面3个int类型需要24个字节的内存，那么实际分配的是第3个span等级，32个字节大小的内存span。

32个字节的内存span 能存储 大小为8个字节的int类型 共四个。所以真实的扩容容量为 4 而不是预估容量3。



扩容及内存分配部分源码

```go
func growslice(et *_type, old slice, cap int) slice {
    ...
    var overflow bool
    var lenmem, newlenmem, capmem uintptr
    const ptrSize = unsafe.Sizeof((*byte)(nil))
    switch et.size {
    case 1:
        lenmem = uintptr(old.len)
        newlenmem = uintptr(cap)
        capmem = roundupsize(uintptr(newcap))
        overflow = uintptr(newcap) > _MaxMem
        newcap = int(capmem)
        ...
    }

    if cap < old.cap || overflow || capmem > _MaxMem {
        panic(errorString("growslice: cap out of range"))
    }

    var p unsafe.Pointer
    if et.kind&kindNoPointers != 0 {
        p = mallocgc(capmem, nil, false)
        memmove(p, old.array, lenmem)
        memclrNoHeapPointers(add(p, newlenmem), capmem-newlenmem)
    } else {
        p = mallocgc(capmem, et, true)
        if !writeBarrier.enabled {
            memmove(p, old.array, lenmem)
        } else {
            for i := uintptr(0); i < lenmem; i += et.size {
                typedmemmove(et, add(p, i), add(old.array, i))
            }
        }
    }
    ...
}
```



1、获取老 Slice 长度和计算假定扩容后的新 Slice 元素长度、容量大小以及指针地址（用于后续操作内存的一系列操作）

2、确定新 Slice 容量大于老 Sice，并且新容量内存小于指定的最大内存、没有溢出。否则抛出异常

3、若元素类型为 `kindNoPointers`，也就是**非指针**类型。则在老 Slice 后继续扩容

- 第一步：根据先前计算的 `capmem`，在老 Slice cap 后继续申请内存空间，其后用于扩容
- 第二步：将 old.array 上的 n 个 bytes（根据 lenmem）拷贝到新的内存空间上
- 第三步：新内存空间（p）加上新 Slice cap 的容量地址。最终得到完整的新 Slice cap 内存地址 `add(p, newlenmem)` （ptr）
- 第四步：从 ptr 开始重新初始化 n 个 bytes（capmem-newlenmem）

注：那么问题来了，为什么要重新初始化这块内存呢？这是因为 ptr 是未初始化的内存（例如：可重用的内存，一般用于新的内存分配），其可能包含 “垃圾”。因此在这里应当进行 “清理”。便于后面实际使用（扩容）

4、不满足 3 的情况下，重新申请并初始化一块内存给新 Slice 用于存储 Array

5、检测当前是否正在执行 GC，也就是当前是否启用 Write Barrier（写屏障），若**启用**则通过 `typedmemmove` 方法，利用指针运算**循环拷贝**。否则通过 `memmove` 方法采取**整体拷贝**的方式将 lenmem 个字节从 old.array 拷贝到 ptr，以此达到更高的效率

注：一般会在 GC 标记阶段启用 Write Barrier，并且 Write Barrier 只针对指针启用。那么在第 5 点中，你就不难理解为什么会有两种截然不同的处理方式了



这里需要注意的是，扩容时的内存管理的选择项，如下：

- 翻新扩展：当前元素为 `kindNoPointers`，将在老 Slice cap 的地址后继续申请空间用于扩容
- 举家搬迁：重新申请一块内存地址，整体迁移并扩容



### 3. 陷阱

#### 1. 同根

Slice 切片指向所引用的 Array。因此在 Slice 上的变更。会直接修改到原始 Array 上（两者所引用的是同一个）



#### 2. 时过境迁

随着 Slice 不断 append，内在的元素越来越多，终于触发了扩容。往 Slice append 元素时，若满足扩容策略，也就是假设插入后，原本数组的容量就超过最大值了，这时候内部就会重新申请一块内存空间，将原本的元素**拷贝**一份到新的内存空间上。此时其与原本的数组就没有任何关联关系了，**再进行修改值也不会变动到原始数组**。这是需要注意的



#### 3. 复制

复制不会触发扩容，所以我们必须准备好cap够用的dst slice。

copy 函数支持在不同长度的 Slice 之间进行复制，若出现长度不一致，在复制时会按照最少的 Slice 元素个数进行复制

那么在源码中是如何完成复制这一个行为的呢？我们来一起看看源码的实现，如下：

```go
func slicecopy(to, fm slice, width uintptr) int {
    if fm.len == 0 || to.len == 0 {
        return 0
    }

    n := fm.len
    if to.len < n {
        n = to.len
    }

    if width == 0 {
        return n
    }
    ...
    size := uintptr(n) * width
    if size == 1 {
        *(*byte)(to.array) = *(*byte)(fm.array) // known to be a byte pointer
    } else {
        memmove(to.array, fm.array, size)
    }
    return n
}
```

- 若源 Slice 或目标 Slice 存在长度为 0 的情况，则直接返回 0（因为压根不需要执行复制行为）
- 通过对比两个 Slice，获取最小的 Slice 长度。便于后续操作
- 若 Slice 只有一个元素，则直接利用指针的特性进行转换
- 若 Slice 大于一个元素，则从 `fm.array` 复制 `size` 个字节到 `to.array` 的地址处（会覆盖原有的值）



#### 4. 奇特的初始化

```go
var nums []int	// nil len=0 cap=0
nums := []int{} 	// [] len=0 cap=0
renums := make([]int, 0) // [] len=0 cap=0
```





<hr>

- [深入理解 Go Slice](https://segmentfault.com/a/1190000017341615)

