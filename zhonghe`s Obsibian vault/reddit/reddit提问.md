
~~~
hello everyone! Recently, while learning the concurrency model of Go language, I have been very interested in its idea of "Do not communicate by sharing memory" (instant, share memory by communication).The channel mechanism of Go replaces explicit locks with data transfer between goroutines, making concurrent programming safer and simpler. This makes me think: can similar ideas be used in operating system design? For example, replacing traditional IPC mechanisms such as shared memory and semaphore with channels?I would like to discuss the following points with everyone:The inter process/thread communication (IPC) of the operating system currently relies on shared memory, message queues, pipelines, and so on. What are the advantages and challenges of using a mechanism similar to Go channel?Will performance become a bottleneck (such as system call overhead)?Realistic case:Have any existing operating systems or research projects attempted this design? (For example, microkernel, Unikernel, or certain academic systems?)? ）Do you think the abstraction of channels is feasible at the OS level?  
大家好！最近，我在学习 Go 语言的并发模型时，对它的 “Do not communicate by sharing memory” （instant， share memory by communication） 的思想非常感兴趣。Go 的通道机制用 goroutine 之间的数据传输取代了显式锁，使并发编程更安全、更简单。这让我思考：类似的思路可以用在作系统设计中吗？例如，用通道替换共享内存和信号量等传统 IPC 机制？我想和大家讨论以下几点：作系统的进程间 / 线程通信（IPC）目前依赖于共享内存、消息队列、管道等。使用类似于 Go 频道的机制有哪些优势和挑战？性能会不会成为瓶颈（如系统调用开销）？现实案例：是否有任何现有的作系统或研究项目尝试过这种设计？（例如，微内核、Unikernel 或某些学术系统？）您认为通道的抽象在 OS 级别是否可行？
~~~


[很开心这么多人回复问题！](https://www.reddit.com/r/golang/comments/1krtd88/could_gos_share_memory_by_communicating/)


![[回答1.png]]



![[回答2.png]]



### Understanding CSP

**CSP（Communicating Sequential Processes）** 是一种并发模型，全称是 **通信顺序进程**，由计算机科学家 **Tony Hoare** 在 1978 年提出。


>CSP 的核心思想是：进程之间不共享内存，而是通过通信（消息传递）来协作


[CSP_blog](https://www.cnblogs.com/papering/p/9479496.html)


```go
func worker(ch chan int) {
    val := <-ch         // 接收数据
    fmt.Println(val)
}

func main() {
    ch := make(chan int)
    go worker(ch)
    ch <- 42             // 发送数据
}

```

这个例子就是标准的 CSP：

- 两个顺序进程（`main` 和 `worker`）
    
- 通道 `ch` 是它们的通信桥梁
    
- 没有共享内存，只靠通道通信
- 

![[CSP示例.png]]


- **CSP** 是一种并发编程理论，强调进程通过**事件（Event）**通信（而非共享内存）。
    
- **进程（Process）**：代表一个独立的行为实体（如售货机或顾客），通过**事件序列**描述行为。
    
- **同步（Synchronization）**：多个进程在特定事件上必须“同步执行”（如顾客投币和售货机接收硬币是同一个 `coin` 事件）。

#### **1. 售货机进程**
**行为**：
    
    1. 等待 `coin` 事件（投币）。
        
    2. 执行 `choc` 事件（发放巧克力）。
        
    3. 终止（`STOP`）。
        
- **意义**：售货机**必须先收钱再给货**。


#### **2. 顾客进程**

Person = (coin \rightarrow STOP) \square (card \rightarrow STOP)

- **行为**：
    
    - `□` 表示**外部选择**（顾客可以选 `coin` 或 `card` 事件，但不会同时发生）。
        
    - 选择后终止（`STOP`）。
        
- **意义**：顾客有**两种支付方式**（硬币或刷卡），但每次只能选一种。

#### **3. 如果售货机只同步 `coin`**

VendingMachine \,|\, [coin] \,|\, Person

- **结果**：

    (coin \rightarrow choc \rightarrow STOP) \square (card \rightarrow STOP)
    
    - 顾客选 `coin`：正常走售货机流程。
        
    - 顾客选 `card`：售货机不响应，顾客直接终止（`STOP`）。

#### **4. 隐藏事件**

**结果**：外部只能看到 `choc` 或直接终止，表现为**非确定性选择**（`⊓`）：

(choc \rightarrow STOP) \sqcap STOP

- 可能发放巧克力后停止，也可能直接停止（因为看不到顾客的支付选择）



- ***同步约束：售货机和顾客必须就支付方式达成一致（如只支持现金时，刷卡会失败）。***
    
- ***非确定性：隐藏内部事件后，系统行为对外部观察者变得不可预测（如同实际场景中，路人看不到顾客是否投币，只能看到巧克力是否出来）。***

CSP 的 `coin → choc → STOP` 类似 Go 中通过 channel 同步的两个 goroutine：

- **同步失败**：如果 `vendingMachine` 只监听 `coin`，`card` 会阻塞。



### 操作系统为什么没用 CSP：

 1. **性能优先：共享内存更快**

操作系统关注 **高性能调度与资源访问**，**共享内存 + 同步机制（如锁、信号量）**可以做到：

- 最低延迟（不需要拷贝）
    
- 高吞吐（直接访问同一块内存）
    

➡ 相比之下，**CSP 的通信（尤其跨进程）要复制数据、上下文切换，代价更大**。

---

 2. **OS 中的“进程”不是 CSP 的“轻量进程”**

CSP 模型适合的是 **轻量并发实体（如 goroutine）**，操作系统的：

- 线程：重量级，切换成本高
    
- 进程：拥有独立内存空间，天然隔离
    

操作系统级别的“进程”之间**不共享内存**，但通信靠 IPC（管道、socket、共享内存）实现——和 CSP 很像，但远不如 goroutine 高效。


### IPC 进程间通信

~~~
IPC（Inter-Process Communication，进程间通信）是一种机制，允许操作系统中不同进程之间交换数据或信号。由于每个进程拥有独立的内存空间，它们无法直接访问彼此的数据，因此需要通过IPC来实现协作和资源共享。
~~~


### CSP的优劣势

社区回答！

~~~
There are cases where CSP produces better performance and cases where the performance is worse. On the one hand, for code that does a lot of small multithreaded operations (like incrementing integers etc), converting all operations to happen via channels is going to be much less performant because channels involve more work per operation. On the other hand, the fact that memory isn't concurrently shared means that you can write faster single threaded code because you don't need to worry about mutexes and barriers etc for objects received from channels.

~~~


#### 一、CSP 的劣势场景（性能差）：

```go
// 用 mutex 的方式做 ++ 操作
mu.Lock()
counter++
mu.Unlock()

// 用 channel 的方式做 ++ 操作
counterChan <- 1
```

- `chan` 本质是有同步开销的（排队、调度、阻塞），一个简单的 `++` 操作搞成 channel 会变得很重。
    
- 所以在高频、低开销场景，**channel 性能不如原子操作 / 锁**。



#### CSP 的优势场景（性能好）：

“但另一方面，因为内存不是共享的，单线程处理收到的 channel 数据，不需要加锁，写起来更快、更简单。”

**内存不是共享的”表述不严谨**：

- channel 的底层本质仍然是共享内存；
    
- 它只是**封装了共享细节，让你不直接共享**；
    
- 所以说准确表述应是：“你**不需要直接访问共享内存**”

```go
// 一个 goroutine 独占处理任务队列
go func() {
  for task := range taskChan {
    handle(task) // 不需要担心并发访问
  }
}()

```



#### 困惑

channel的内部也就是底层实现 不还是使用的内存共享+锁机制实现的吗，这里面真的会有性能差别吗？


#### 核心回答！

`你用不用锁，和系统/库替你加不加锁，是两回事。`


Go 的 `channel` 底层确实是用**共享内存 + 加锁**实现的：

- `chan` 的发送和接收涉及锁（`mutex`）、等待队列（`sudog`）、调度协程切换等。
    
- 所以说 channel 本身也有“同步开销”。

 那为什么说 “你写的代码没锁” 性能反而更好？

#### ✅ **关键是“封装” 和 “并发域最小化”**：

> channel 把“并发边界”控制在接口级别，而不是让你每个字段自己去加锁。

**你写的处理函数**只处理 `<-chan` 接收的数据，不访问共享变量：

- 它是**串行**的，不涉及任何并发操作；
    
- 所以**你可以不用管加锁、原子操作、同步屏障**；
    
- 你的代码天然是“线程安全”的，这会**减少脑力负担 + 提高可维护性 + 避免 bug**。


**channel 是“通信安全”，不是“零成本”。**  
真正的性能差异不在“是否用了内存共享”，而是**谁来负责并发同步的复杂性**


*数据被携带着传输”这句，**精准命中了 CSP 的核心思想**，也正是 Go 在并发编程中推荐的思维方式。*

> ***不要通过共享内存来通信，而应该通过通信来共享内存。**（Go 编程语言官方理念）*

### **Plan 9 from Bell Labs**


在Go SDK中 runtime包中含又 plan9的身影！


## 线程池

