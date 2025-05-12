
### **可变参数用法**

1. **只能有一个可变参数**，且必须是最后一个参数：

```go


    func foo(a int, b ...string) {}  // 正确
    func bar(a ...int, b string) {}   // 错误：可变参数必须在最后
    
```


```go
2. **可变参数的类型必须相同**：
    func baz(a ...int) {}    // 正确
    func qux(a ...any) {}    // 正确（Go 1.18+ 泛型）
    func quux(a ...int, b ...string) {}  // 错误：只能有一个可变参数
```



 `context.WithoutCancel` 解析

`ctx, cancel := context.WithoutCancel(context.Background())` 是 Go 1.21 版本引入的一个新 Context 函数，它的作用是**创建一个不受父 Context 取消影响的新 Context**。


---

### 对于这段代码我的问题

```go
package main  
  
import (  
    "fmt"  
    "sync"    "time")  
  
func producer(ch chan<- int) {  
    for i := 1; i < 11; i++ {  
       ch <- i  
       time.Sleep(time.Second)  
    }  
    close(ch)  
  
}  
  
func consumer(id int, ch <-chan int, wg *sync.WaitGroup) {  
    defer wg.Done()  
    fmt.Println("val:", <-ch)  
}  
  
func main() {  
    ch := make(chan int)  
    var wg sync.WaitGroup  
    wg.Add(2)  
    for i := 1; i <= 2; i++ {  
       go consumer(i, ch, &wg)  
    }  
    producer(ch)  
    wg.Wait()  
}
```

***为什么生产者也要用go启动，这是我的疑惑 虽然生成的数字的时候会阻塞 单go consumer(ch)终究会启动然后区消费数据呀***




#### 回答：

如果我们没有启动生产者的 `go` 协程，那么它会在主线程中**阻塞**，直到生产者完成所有的数据发送，然后消费者才开始执行。这会破坏并发的效果。


由于生产者和消费者没有并发执行，**消费者会等到生产者完全发送完数据**才能开始工作。这样的顺序执行会出现问题，特别是当生产者执行完毕且数据都发送到通道时，消费者开始消费数据。


```go
package main  
  
import (  
    "fmt"  
    "sync"    "sync/atomic")  
  
func producer(id int, ch chan<- int, wg *sync.WaitGroup, total *int32) {  
    defer wg.Done()  
    for {  
       current := atomic.LoadInt32(total)  
       if current >= 100 {  
          return  
       }  
       newVal := atomic.AddInt32(total, 1)  
       if newVal > 100 {  
          return  
       }  
       fmt.Println("生产者:", id, "生产:", newVal)  
       ch <- int(newVal)  
  
    }  
  
    //close(ch) 不能让每个生产者都能关闭通道 而是要统一关闭  
  
}  
  
func consumer(id int, ch <-chan int, wg *sync.WaitGroup) {  
    defer wg.Done()  
    for val := range ch {  
       fmt.Println("消费者: ", id, "消费:", val)  
    }  
}  
  
func main() {  
    var counter int32  
    ch := make(chan int)  
    var wgPro sync.WaitGroup  
    var wgCon sync.WaitGroup  
  
    wgPro.Add(3)  
    // 28-31行的数据顺序也会导致不同的结果 对于没有 go 启动的生产者来说，生产者执行的是一个阻塞操作，它会依次执行以下步骤  
    for a := 1; a <= 3; a++ {  
       go producer(a, ch, &wgPro, &counter)  
    }  
  
    //统一处理通道关闭  
    go func() {  
       wgPro.Wait()  
       close(ch)  
    }()  
  
    wgCon.Add(5)  
    for i := 1; i <= 5; i++ {  
       go consumer(i, ch, &wgCon)  
    }  
  
    wgCon.Wait() //等待所有消费者处理完  
}
```



---

接下来 带超时、上下文取消的版本，强化 Go 并发控制？


