
异步编程是一种编程范式，它的核心思想是**避免阻塞**，让程序在等待某些操作（如 I/O、网络请求、数据库查询）完成时，能够继续执行其他任务，从而提高程序的并发性和性能。异步编程通常通过**回调函数**、**Promise**、**Future** 或 **协程** 来实现。

在 Go 语言中，异步编程主要通过 **Goroutine** 和 **Channel** 来实现。Go 的并发模型天然支持异步编程，且语法简洁，易于理解。


### 并发与并行最大区别： 

多个任务是否互相抢占资源

异步编程： 让程序并发运行的一种手段

channel gorutine context 


chan作用: 负责gorutine之间的通信

只读<-chan  只写chan<-   双向chan

### 有无缓冲channel： 

无缓冲 ：Channel 的发送和接收操作是**阻塞的**

只有当发送方和接收方都准备好时，数据才会传递。这种特性使得无缓冲 Channel 非常适合用于**任务同步**和**协程间通信**。

1. **任务同步**：确保两个 Goroutine 在某个时刻同步执行。
    
2. **协程间通信**：实现 Goroutine 之间的数据传递和协调。
    
3. **替代锁**：通过 Channel 实现更优雅的并发控制。


```go

package main

import (
	"fmt"
	"time"
)

func worker(done chan bool) {
	fmt.Println("Worker: 开始工作")
	time.Sleep(2 * time.Second) // 模拟耗时任务
	fmt.Println("Worker: 工作完成")
	done <- true // 发送完成信号
}

func main() {
	done := make(chan bool) // 无缓冲 Channel
	go worker(done)

	// 等待 worker 完成
	<-done
	fmt.Println("Main: 收到完成信号，程序结束")
}
```




### 异步编程：

```go
package main

import (
	"fmt"
	"time"
)

func asyncTask(name string, delay time.Duration, done chan bool) {
	fmt.Println("Task", name, "开始执行")
	time.Sleep(delay) // 模拟耗时任务
	fmt.Println("Task", name, "执行完成")
	done <- true // 发送完成信号
}

func main() {
	done := make(chan bool)

	// 启动异步任务
	go asyncTask("A", 2*time.Second, done)
	go asyncTask("B", 1*time.Second, done)

	// 等待任务完成
	<-done
	<-done
	fmt.Println("所有任务完成")
}
```


也就是 子goruntine 写入内容： 主goroutine 可以读取内容

### 疑问：使用go关键字就是异步吗？


不，`go` 关键字本身并不表示异步。它表示并发执行，即启动一个新的 goroutine。一个 goroutine 是一个轻量级线程，可以与其他 goroutine 并发执行。虽然通常情况下它会异步执行，但并不意味着它总是异步的，因为执行顺序仍然可能受调度器的控制，取决于系统资源和调度策略。


### **异步与并发的区别**

- **异步**指的是“我发起了一个操作，不关心它什么时候完成，我继续执行我的程序，等它完成再处理结果。”
- **并发**则是指“多个任务同时进行，任务之间可以相互交替执行”，并发操作并不一定是异步的，它只是让多个任务在同一时间片内交替执行。


### 辅助chan

close（关闭） range（读取）



```go
package main

import "fmt"

func main() {
    messagech := make(chan int, 100)

    for i := 0; i < 10; i++ {
        messagech <- i
    }

    // 关闭通道
    close(messagech)

    // 使用 range 进行读取
    for item := range messagech {
        fmt.Println("message number:", item)
    }
	//for循环

	/*
	for {
		time.sleep(time.Second)
		item, ok := <- messagech
		if !=ok {
		break
	}
	fmt.Printf("item:,%d\n, ok: , %%v", item,ok)	
	}
	*/
}
```



我误会了，以为关闭通道的位置，导致无法读取，其实是：

1. **关闭通道**：`close(messagech)` 关闭了通道 `messagech`。关闭通道后，不能再向通道发送数据，但仍然可以从中读取数据，直到通道中没有剩余的数据为止。



### waitgroup  实际控制

time.sleep 很低效 性能消耗

sync.WaitGroup

wg ：= &sync.Waitgroup

常见方法:

add done wait

wg.Add(1) 

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func doSome(number int) {
	time.Sleep(time.Millisecond * 100)
	fmt.Printf("result %d\n", number)
}

func asyncDoSome(number int, wg *sync.WaitGroup) {
	defer wg.Done() // 确保在函数退出时调用 wg.Done()
	time.Sleep(time.Millisecond * 100)
	fmt.Printf("result %d\n", number)
}

func main() {
	start := time.Now()
	wg := &sync.WaitGroup{}

	// 启动 10 个 goroutine
	for i := 0; i < 10; i++ {
		wg.Add(1) // 每启动一个 goroutine，WaitGroup 计数器加 1
		//go asyncDoSome(i, wg) // 传入 i 作为参数，而不是固定值 13

	//匿名写法
	go func(number int,wg *sync.WaitGroup) {
		time.Sleep(time.Millisecond * 100)
		fmt.Printf("result: %d\n",number)
		wg.Done()
	}(13,wg)

	}

	wg.Wait() // 等待所有 goroutine 完成

	fmt.Println("use time:", time.Since(start))
	fmt.Println("end game")
}
```


疑惑： 为什么时间这么短呢，每一次调用函数不都睡眠100ms吗调用了10次，应该更长才对吧

回答：

- **串行执行**：
    
    - 如果代码是串行执行的（即每次调用函数都等待前一次调用完成），那么总时间会是每次调用时间的累加。
        
    - 例如，每次调用 `asyncDoSome` 休眠 100ms，调用 10 次，总时间就是 `10 * 100ms = 1000ms`。
        
- **并发执行**：
    
    - 在并发执行的情况下，多个 goroutine 是同时运行的，而不是一个接一个地运行。
        
    - 在你的代码中，10 个 goroutine 是同时启动的，它们会并行地休眠 100ms，然后几乎同时完成。
        
    - 因此，总时间主要由最慢的 goroutine 决定，而不是所有 goroutine 的时间累加。


### Select 流程控制


quitCh chan struct { } //0 byte


```go
for{
	select {
	case <- s.messageCh:
		s.handleMessage(message)
	case <- s.quitCh:
		fmt.Println("quiting server")
	}
}
```


 **1. 无限循环设计**

- `for { ... }` 没有终止条件，意味着循环会一直运行，直到遇到 `return` 或外部中断。
    
- **目的**：持续监听消息通道 (`messageCh`) 和退出通道 (`quitCh`)，实现服务器的长期运行。
    

 **2. Select 多路复用**

- `select` 会阻塞，直到其中一个 `case` 被触发：
    
    - **消息处理** (`case message := <-s.messageCh`)：当 `messageCh` 中有消息时，调用 `handleMessage` 处理。
        
    - **退出信号** (`case <-s.quitCh`)：当 `quitCh` 被关闭或收到信号时，打印退出信息并 `return`，结束循环。



### Mutex和Atomic

解决： DATA RACE 数据竞争！



使用指令检测： `go run --race main.go

做data race 检查 

Mutex

1. mu sync.Mutex持锁

```go
c.Mutex.Lock()
defer c.Mutex.Unlock
业务代码
```

解决资源竞争

1. 原子操作

atomic


### 异步模式
