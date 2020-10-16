---
title: Go语言通过channel实现并发控制
permalink: GoPool
date: 2020-10-11 15:25:46
toc: true
tags:
 - Golang
categories:
 - Go基础
---

让Golang中并发控制像Python中那样优雅

<!--more-->

先来看一下Python中的优雅实现方式

```python
from concurrent.futures import ThreadPoolExecutor
import time

def thread_pool_executor_callback(worker):
    if worker.exception():
        logging.exception("worker %s got exception", worker)

def loooooong_task(i):
    print("task %s sleeping..." % i)
    time.sleep(10)
    print("task %s done..." % i)

with ThreadPoolExecutor(max_workers=2) as executor:
    for i in range(10):
        executor.submit(loooooong_task, i).add_done_callback(thread_pool_executor_callback)
```

在`ThreadPoolExecutor`中直接可以确定线程的数量，是不是很优雅。在go语言中如何实现如此优雅的协程控制呢。



使用go自带的WaitGroup**无法控制**最多使用多少个协程，如下的代码对每个URL请求，都要进行add操作，无法对协程的数量进行真正的控制。

```go
func main() {
	var wg sync.WaitGroup
	var urls = []string{
		"http://www.golang.org/",
		"http://www.google.com/",
		"http://www.somestupidname.com/",
	}
	for _, url := range urls {
		// Increment the WaitGroup counter.
		wg.Add(1)
		// Launch a goroutine to fetch the URL.
		go func(url string) {
			// Decrement the counter when the goroutine completes.
			defer wg.Done()
			// Fetch the URL.
			http.Get(url)
		}(url)
	}
	// Wait for all HTTP fetches to complete.
	wg.Wait()
}
```



通过channel可以简单地实现这个需求，上代码

```go
package gopool

/**
	用channel实现并发控制
*/

type GoPool struct {
	MaxLimit  int           // 最大的并发goroutine数量
	tokenChan chan struct{} // 创建MaxLimit数量的token缓冲chan，用来阻塞创建协程
}

type GoPoolOption func(gp *GoPool)

func WithLimitGoPool(limit int) GoPoolOption {
	return func(gp *GoPool) {
		gp.MaxLimit = limit
		gp.tokenChan = make(chan struct{}, limit)

		for i := 0; i < limit; i++ {
			gp.tokenChan <- struct{}{}
		}
	}
}

func NewGoPool(op ...GoPoolOption) *GoPool {
	gp := &GoPool{}
	for _, option := range op {
		option(gp)
	}
	return gp
}

func (gp *GoPool) Submit(fn func()) {
	// 每提交一个协程请求，从tokenChan获取一个token，
	// 如果没有可用token将会阻塞，不在创建协程
	token := <-gp.tokenChan
	go func() {
		fn()
		// 执行完一个fn()，归还token令牌
		gp.tokenChan <- token
	}()
}

// 等待所有令牌归还后，关闭chan
func (gp *GoPool) Wait() {
	for i := 0; i < gp.MaxLimit; i++ {
		<-gp.tokenChan
	}
	close(gp.tokenChan)
}

func (gp *GoPool) size() int {
	return len(gp.tokenChan)
}

// to use it
func main() {
	pool := NewGoPool(WithLimitGoPool(3))
	defer pool.Wait()

	for i := 0; i < 100; i++ {
		pool.Submit(func() {
			fmt.Println(i)
		})
	}
}


```

