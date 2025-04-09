
```go
package main

import (

    "fmt"

    "os"

    "runtime/trace"

)


func main() {

    f, err := os.Create("trace.out")

    if err != nil {

        panic(err)

    }


    defer f.Close()

    err = trace.Start(f)

    if err != nil {

        panic(err)

    }

    defer trace.Stop()

  

    fmt.Println("Hello, world!")

}
```

~~~

go run go_tool_trace.go
Hello, world!

go tool trace trace.out
2025/03/30 21:03:58 Preparing trace for viewer...
2025/03/30 21:03:58 Splitting trace for viewer...
2025/03/30 21:03:58 Opening browser. Trace viewer is listening on http://127.0.0.1:63561

~~~

###  trace1


|            |           |                     |     |
| ---------- | --------- | ------------------- | --- |
| Goroutines | GCWaiting | 0.43050400000000005 | 0   |
| Goroutines | Runnable  | 0.43050400000000005 | 0   |
| Goroutines | Running   | 0.43050400000000005 | 2   |
为什么是两个核心在运行？


#### trace2

```go
package main

  

import "time"

  

func main() {

    for i := 0; i < 5; i++ {

        time.Sleep(1 * time.Second)

        println("Hello, World")

    }

  

}
```


~~~
$env:GODEBUG="schedtrace=1000"
PS E:\Go_Advanced\chaper1\cmd\trace2> go run .\deBug_trace.go
~~~


---

## 闭包

 #### **实现状态封装**

闭包可以隐藏变量，仅通过函数暴露操作

**闭包（Closure）** 是指一个函数值（function value）引用了其函数体之外的变量。闭包允许函数访问并操作这些外部变量，即使这些变量原本的作用域已经失效。闭包的核心作用是 **捕获和保持状态**，它在 Go 中常用于实现以下功能：


*1. 操作外部变量*

```go
func outer() func() int {
    count := 0          // 外部变量
    return func() int { // 闭包函数
        count++         // 操作外部变量
        return count
    }
}
```


*2. 闭包可以隐藏变量，仅通过函数暴露操作 *

调用：

```go
func main() {
    c := newCounter()
    fmt.Println(c()) // 1
    fmt.Println(c()) // 2
}
```

*3.   延迟执行（Deferred Execution）

```c
func main() {
    msg := "hello"
    defer func() {
        fmt.Println(msg) // 闭包捕获了 msg
    }()
    msg = "world"
}
// 输出: world（defer 执行时捕获的是最终值）
```

## 断言

##### 类型断言

```go
// 使用类型断言获取更多错误信息
		if urlErr, ok := err.(*URLFetchError); ok {
			fmt.Printf("Error fetching URL: %v\n", urlErr)
		} else {
			fmt.Printf("Unknown error: %v\n", err)
		}
		return
```


---

检查 `err` 是否是自定义的 `*URLFetchError` 类型

类似功能 errors.As

---


示例用法：

```go
value, ok := interfaceValue.(ConcreteType)
```


~~~
- `interfaceValue`：一个接口类型的值（比如 `error` 接口）
    
- `ConcreteType`：你想断言的具体类型（比如 `*URLFetchError`）
    
- `ok`：布尔值，表示断言是否成功
    
- `value`：如果成功，是对应类型的值；否则是该类型的零值
~~~


## select


```go
func main() {

	c1 := make(chan string)
	c2 := make(chan string)

	go func() {
		time.Sleep(1 * time.Second)
		c1 <- "one"
	}()
	go func() {
		time.Sleep(2 * time.Second)
		c2 <- "two"
	}()
	//select 允许您等待多个通道作
	for i := 0; i < 3; i++ {
		select {
		case msg1 := <-c1:
			fmt.Println("received", msg1)
		case msg2 := <-c2:
			fmt.Println("received", msg2)
		default:
			time.Sleep(10000 * time.Millisecond) // 避免 CPU 忙等待
			fmt.Println("no message received")
		}

	}
}

```

#### 问题：

~~~
为什么我这么设置 最先打印的还是 go run main.go
no message received
received one
received two 这其中是什么造成的
~~~

`default` 分支设置了 `time.Sleep(10000 * time.Millisecond)`（10秒），但 **最先打印的仍然是 `"no message received"`**，这看起来似乎不合理，因为 `c1` 应该在 **1秒后** 就发送数据，而 `default` 要等 **10秒** 才继续执行。


1. **`select` 的执行是立即的**：
    
    - 当 `select` 执行时，它会 **立即检查所有 `case`**，如果没有 `case` 就绪（即 `c1` 和 `c2` 都没有数据），就会进入 `default`。
        
    - **`default` 不会等待 `case` 就绪**，而是直接执行。

#### 回答：

我忘记了  这并不是并行执行的，每个for只会执行一次， 因为select是立即执行的，因为 前两个 go需
要准备时间 所以无论如何都是 default先执行！



```go
func main() {

    messages := make(chan string)

    signals := make(chan bool)

  

    select {

    case msg := <-messages:

        fmt.Println("received message", msg)

    default:

        fmt.Println("no message received")

    }

  

    msg := "hi"

    select {

    case messages <- msg:

        fmt.Println("sent message", msg)

    default:

        fmt.Println("no message sent")

    }

  

    select {

    case msg := <-messages:

        fmt.Println("received message", msg)

    case sig := <-signals:

        fmt.Println("received signal", sig)

    default:

        fmt.Println("no activity")

    }

  

    signals <- true

    select {

    case msg := <-messages:

        fmt.Println("received message", msg)

    case sig := <-signals:

        fmt.Println("received signal", sig)

    default:

        fmt.Println("no activity")

    }

}
```


#### 问题：

为什么会造成死锁   ****为什么要开辟goroutine 无缓冲通道才可以使用呢？ 为什么一个缓冲就可以在主goroutine中接收呢***

1. **`signals` 是无缓冲通道**，`signals <- true` **会阻塞**，直到某个 goroutine 从 `signals` 接收数据。
    
2. 但你的代码 **只有一个 goroutine（主 goroutine）**，而 `select` 的执行是 **非阻塞的**（因为有 `default`），所以：
    
    - `case sig := <-signals` **不会执行**（因为 `select` 优先执行 `default`）。
        
    - `signals <- true` **永远阻塞**，导致 **死锁**。

#### 核心
. `ch <- 42` 会阻塞主 goroutine，等待接收者。
    
. 但接收代码 `<-ch` 在同一个 goroutine 中，永远无法执行。
    
. Go 检测到所有 goroutine 都阻塞，直接报错。


**子 goroutine** 中执行，不会阻塞主 goroutine。    - **无缓冲通道的本质是“同步队列”**，必须有两个 goroutine 配合（一个发，一个收）。

缓冲通道（`make(chan T, N)`）的**核心特性**是 **异步存储**：  

**发送 只在缓冲区满时阻塞**。 

**- 接收操作 (`<-ch`) **只在缓冲区空时阻塞**。

