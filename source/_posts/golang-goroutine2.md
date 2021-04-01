---
title: golang 并发基础（二）
tags: golang
categories: 并发
copyright: true
date: 2019-08-25 22:49:36
---


# golang 并发基础（二）

上一篇简单介绍了并发与并行，goroutine实际工作的原理示意图，现在我们来看下golang是怎么处理并发中竞争状态的。

竞争状态：如果两个或多个goroutine在没有同步的情况下对同一个资源进行读写操作，就处于相互竞争的状态，称为竞争状态。对一个共享资源的操作必须是**原子化**的，即同一时刻只能由一个goroutine对共享资源进行读和写操作。
<!--more-->
## 同步操作方式--锁住共享资源

golang提供atomic和sync包，两个包里的函数提供了很好的解决方案。

### 原子函数 atomic 包

我们可以看一下atomic提供的原子函数，它提供底层加锁的方式来同步访问整形变量和指针，示例代码:

```golang
var (
	counter int64

	wg sync.WaitGroup
)

func main() {
	wg.Add(2)

	go incCounter(1)
	go incCounter(2)

	wg.Wait()

	fmt.Println("Final Counter:", counter)
}

func incCounter(id int) {
	defer wg.Done()

	for count := 0; count < 2; count++ {
		atomic.AddInt64(&counter, 1)

		runtime.Gosched()
	}
}
```

### 互斥锁 mutex

使用互斥锁也是一种同步访问共享资源的方式，互斥的概念就是AB只有一个可以访问，要不就是A，要不就是B。相当于一张门，只能一个人进入，第一个进入的人就把门锁了，其它人都不可以进来，只有等这个人把锁打开了，其它人才能重新竞争锁。

看一下代码：

```golang

var (
	counter int

	wg sync.WaitGroup

	mutex sync.Mutex
)

func main() {
	wg.Add(2)

	go incCounter(1)
	go incCounter(2)

	wg.Wait()
	fmt.Printf("Final Counter: %d\n", counter)
}

func incCounter(id int) {
	defer wg.Done()

	for count := 0; count < 2; count++ {
		mutex.Lock() //锁住
		{
			value := counter

			runtime.Gosched()

			value++

			counter = value
		}
		mutex.Unlock()//释放锁
	}
}

```

### 通道 channel

在goroutine之间还可以通过通道来发送和接受需要共享的资源，在goroutine之间做同步。当一个资源需要被goroutine共享时，通道在goroutine之间架起了一个管道，并提供了确保同步交换数据的机制。声明通道，需要指定被共享的数据类型。go中需要使用内置函数make来创建一个通道。

通道的类型由两种：无缓冲通道与有缓冲通道。定义的方式如下：

```golang

//无缓冲
unbuffered := make(chan int)

// 有缓冲 
buffered := make(chan string, 10)

```

通道的赋值和取值如下:

```golang

//定义
buffered := make(chan string, 10)

//赋值
buffered <- "ok, set channel value"

//取值
value := <- buffered

```

无缓冲通道就像“接力跑步赛”，数据就是那个“接力棒”，通道就是两个运动员之间的跑道。如果运动员A拿到接力棒，不将接力棒传到运动员B的手中，那么B就无法开始跑（阻塞），而A因为B没有拿到接力棒，A也无法去做其它的事情。所以A其实也是阻塞在了传递的这个过程中。可以看到传递“接力棒”和接受“接力棒”这两个过程其实是个同步的，两个都无法独立存在。

有缓存通道就像是吃转转火锅，通道就是传送带，数据就是传送带上的每碟食物，goroutine就是厨师和顾客。缓冲数量就是传送带可以放的碟子数量。所以一个厨师(goroutine)将餐碟放到传送带上，传送带本身就只能存放固定数量的餐碟，也就是缓冲带。如果缓冲带满了，那么厨师只能等待着，等到某个餐碟空了，再放食物。而顾客也是从传送带上取餐碟，只要传送带上面有餐碟，就会吃么。如果顾客拿到手中的餐碟没吃完，传送带上还有空位，那么厨师可以继续放，如果没有空位了，就只能等顾客，吃完再从传送带上取出餐碟，这样才能继续。

两者的区别：无缓冲通道保证接受和发送的goroutine会在同一时间进行数据交换；有缓冲通道就不提供这种保证。

要注意通道的关闭操作，当通道关闭后，goroutine依旧可以从通道接收数据，但是不能再往里面发送数据了。从通道获取数据的时候会返回一个ok标志，如果值为false，那说明通道已经关闭了。

看一下示例。

无缓冲通道：

```golang
// This sample program demonstrates how to use an unbuffered
// channel to simulate a relay race between four goroutines.
package main

import (
	"fmt"
	"sync"
	"time"
)

// wg is used to wait for the program to finish.
var wg sync.WaitGroup

// main is the entry point for all Go programs.
func main() {
	// Create an unbuffered channel.
	baton := make(chan int)

	// Add a count of one for the last runner.
	wg.Add(1)

	// First runner to his mark.
	go Runner(baton)

	// Start the race.
	baton <- 1

	// Wait for the race to finish.
	wg.Wait()
}

// Runner simulates a person running in the relay race.
func Runner(baton chan int) {
	var newRunner int

	// Wait to receive the baton.
	runner := <-baton

	// Start running around the track.
	fmt.Printf("Runner %d Running With Baton\n", runner)

	// New runner to the line.
	if runner != 2 {
		newRunner = runner + 1
		fmt.Printf("Runner %d To The Line\n", newRunner)
		go Runner(baton)
	}

	// Running around the track.
	time.Sleep(100 * time.Millisecond)

	// Is the race over.
	if runner == 2 {
		fmt.Printf("Runner %d Finished, Race Over\n", runner)
		wg.Done()
		return
	}

	// Exchange the baton for the next runner.
	fmt.Printf("Runner %d Exchange With Runner %d\n",
		runner,
		newRunner)

	baton <- newRunner
}
```

有缓冲通道：

```golang
// This sample program demonstrates how to use a buffered
// channel to work on multiple tasks with a predefined number
// of goroutines.
package main

import (
	"fmt"
	"math/rand"
	"sync"
	"time"
)

const (
	numberGoroutines = 4  // Number of goroutines to use.
	taskLoad         = 10 // Amount of work to process.
)

// wg is used to wait for the program to finish.
var wg sync.WaitGroup

// init is called to initialize the package by the
// Go runtime prior to any other code being executed.
func init() {
	// Seed the random number generator.
	rand.Seed(time.Now().Unix())
}

// main is the entry point for all Go programs.
func main() {
	// Create a buffered channel to manage the task load.
	tasks := make(chan string, taskLoad)

	// Launch goroutines to handle the work.
	wg.Add(numberGoroutines)
	for gr := 1; gr <= numberGoroutines; gr++ {
		go worker(tasks, gr)
	}

	// Add a bunch of work to get done.
	for post := 1; post <= taskLoad; post++ {
		tasks <- fmt.Sprintf("Task : %d", post)
	}

	// Close the channel so the goroutines will quit
	// when all the work is done.
	close(tasks)

	// Wait for all the work to get done.
	wg.Wait()
}

// worker is launched as a goroutine to process work from
// the buffered channel.
func worker(tasks chan string, worker int) {
	// Report that we just returned.
	defer wg.Done()

	for {
		// Wait for work to be assigned.
		task, ok := <-tasks
		if !ok {
			// This means the channel is empty and closed.
			fmt.Printf("Worker: %d : Shutting Down\n", worker)
			return
		}

		// Display we are starting the work.
		fmt.Printf("Worker: %d : Started %s\n", worker, task)

		// Randomly wait to simulate work time.
		sleep := rand.Int63n(100)
		time.Sleep(time.Duration(sleep) * time.Millisecond)

		// Display we finished the work.
		fmt.Printf("Worker: %d : Completed %s\n", worker, task)
	}
}

```


## select 

提到goroutine的channel，通常都会与select一起使用。官网的说明：[select](https://golang.google.cn/ref/spec#Select_statements)。简单的说一下select的用法：
1：select 的case一定是一个chan的表达式
2：select监听的是当前运行的goroutine，如果当前没有运行的goroutine会直接抛出panic；
3：如果没有default语句，而多个case都可以执行，那么随机取一个；
4：如果有default语句，且case多个可以执行，执行default

```golang

package main

import (
	"fmt"
	"testing"
	"time"
)

func Chann(ch chan int, stopCh chan bool) {
	var i = 10
	for j := 0; j < 10; j++ {
	//for {
		time.Sleep(time.Second)
		//ch <- i
		//fmt.Println(i)
		fmt.Println(i)
	}

	stopCh <- true
	fmt.Println("ok")
}

func TestChan(t *testing.T) {
	ch := make(chan int)
	c := 0
	stopCh := make(chan bool)

	go Chann(ch, stopCh)

	for {
		select {
		case ch <- 10:
			fmt.Println("ok")
		case <-ch:
			fmt.Println("Recvice c1", c)
			fmt.Println("channel")
		case <-ch:
			fmt.Println("Receive s", ch)
			//case _ = <-stopCh:
			//	fmt.Println("stop")
			//	goto end
			//default:
			//	fmt.Println("default")
		}

	}
	//end:
}

```