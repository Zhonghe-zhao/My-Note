
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

原则： 如果违反 CSP 原则，**通过 Channel 间接共享内存**，仍需要锁：

```go
type Data struct{ X int }
func main() {
    ch := make(chan *Data)
    d := &Data{X: 1}
    go func() { ch <- d }() // 发送指针
    go func() { d.X++ }()   // 竞态！违背 CSP
}
```

channel 不是万能保险，传的是引用时就要小心。


社区回答！

~~~
There are cases where CSP produces better performance and cases where the performance is worse. On the one hand, for code that does a lot of small multithreaded operations (like incrementing integers etc), converting all operations to happen via channels is going to be much less performant because channels involve more work per operation. On the other hand, the fact that memory isn't concurrently shared means that you can write faster single threaded code because you don't need to worry about mutexes and barriers etc for objects received from channels.

~~~


[同步方法测试](https://go-benchmarks.com/synchronization-methods)

[一篇帖子的说明](https://www.jtolio.com/2016/03/go-channels-are-bad-and-you-should-feel-bad/)



*作者的吐槽点* ： “- **channel 适合某些场景**（如任务队列、事件通知、流水线模式）。
    
- **mutex 适合另一些场景**（如保护共享状态、简单临界区）。
    
- **"channel of channels" 确实可能增加复杂度**，但 Go 的 select + channel 机制也能提供强大的并发控制能力。”


“作者的观点是 **"不要为了用 channel 而用 channel"**，应该根据实际情况选择最合适的同步机制（mutex 或 channel），而不是盲目遵循 Go 的 "share memory by communicating" 哲学。”



[一篇论文](https://songlh.github.io/paper/go-study.pdf)


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

#### **关键是“封装” 和 “并发域最小化”**：

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



```go
The OS itself also avoids sharing memory, and of course there are "channels" for communicating: file descriptors, which can be devices, pipes, network connections, etc. Like in Unix. But it goes beyond what traditional Unix does, because many userspace applications are file servers that provide a file descriptor that can be mounted onto the file system, and provide more such file descriptors, etc. It's a very simple but powerful system.
```

 plan9操作系统避免了共享内存, 那内部是什么样的呢 进程之间如何通信呢
 

在Go SDK中 runtime包中含又 plan9的身影！

Plan 9 **避免传统的共享内存和多线程同步机制**，鼓励进程之间用“文件通信”（如命名管道、9P 协议）：

- 类似 Go 的 Channel，但用的是文件协议。
    
- 所有通信都可序列化，可远程传输。

> 难道操作文件就不是操作同一块内存了吗？


示例：

共享内存
```go
int *shared = mmap(...);  // 多个进程访问这个地址
*shared = 42;             // 谁都能读写这块内存
```

文件通信
```
echo 42 > /srv/somefile     # 写入
cat /srv/somefile           # 另一个程序读
```


虽然最终数据可能写入页缓存或磁盘（确实会进内存），但你**无法直接访问或共享那块内存**，只能通过：

- 系统调用（read/write）
    
- Channel 抽象（Go）
    
- 文件协议（Plan 9）

#### 疑问：

1. 那xv6系统的内存的关于 前面所说的CSP理论和共享内存，xv6是偏向与什么？ 或者说linux系统都沿用了文件通信？

> xv6 和 Linux 都以“共享内存 + 文件通信”为基础，核心机制是“共享内存 + 加锁”，而不是 CSP 模型。

2. 所以说文件通信和CSP之间的关系是什么？ 有联系吗 为什么前面你跟我说Plan 9 设计就是避免共享内存，强调文件通信 + 用户态协议 那既然文件通信是避免共享内存，那为什么不能说xv6等liunx操作系统不是CSP模型呢

> 文件通信是一种实现机制，CSP 是一种并发模型。Plan 9 把文件通信机制用于实现类似 CSP 的并发风格；而 Linux 虽然也有文件通信，但仍然基于共享内存 + 锁，不符合 CSP 的并发语义。


|特性|Linux/xv6|CSP (Plan 9 风格)|
|---|---|---|
|默认通信方式|共享内存 + 锁|Channel / 消息|
|文件通信|有，但不普遍用作并发通信|核心机制|
|并发语义|多线程共享状态|顺序过程 + 通信|
|并发控制|mutex、atomic、lock|通信即同步|
|数据一致性|程序员手动维护|通信机制保证|

Plan 9 并没有在商业或主流社区中广泛流行，但它对**现代操作系统的设计理念影响深远**。许多重要思想被吸收进 Linux、Go 语言、Docker 等系统中。




##  mkfifo is OS's channel

- `mkfifo` 是一个 Linux 系统调用（命令），用于**创建一个命名管道（FIFO）**。
    
- FIFO = First In, First Out，像文件一样存在于磁盘，但其实是一种特殊的**IPC（进程间通信）**手段。
    
- 创建后，多个进程可以通过读写这个“文件”来通信。


|Go 的 `chan`|OS 的 `mkfifo`|
|---|---|
|语言级通道，只在 Go 中使用|系统级别，多个进程/语言可用|
|内存中的结构，速度快|磁盘上的文件，效率低，但能跨进程|
|用于 goroutine 间通信|用于进程间通信（IPC）|
|类型安全、阻塞/非阻塞控制强|只能读字节流，无结构化信|


## 线程池



##  **批评 Go 语言在错误处理和一些特殊语法上的不一致性**


[另一篇](https://bravenewgeek.com/go-is-unapologetically-flawed-heres-why-we-use-it/)

~~~
There are other peculiar idiosyncrasies. Error handling is generally done by returning error values. This is fine, and I can certainly see the motivation coming from the abomination of C++ exceptions, but there are cases where Go doesn’t follow its own rule. For example, map lookups return two values: the value itself (or zero-value/nil if it doesn’t exist) and a boolean indicating if the key was in the map. Interestingly, we can choose to ignore the boolean value altogether—a syntax reserved for certain blessed types in the standard library. Type assertions and channel receives have equally curious behavior.
~~~


```go
file, err := os.Open("foo.txt")
if err != nil {
    return err
}
```

- 这是 Go 的核心设计哲学之一：**强制程序员显式处理错误**，避免像 C++ 异常（exceptions）那样隐式传播。

```go
value, exists := myMap["key"]  // 返回值和布尔值
value = myMap["key"]           // 可以忽略布尔值，直接取值
```

- 但 Go 允许 **直接忽略布尔值**，这种语法是 **标准库的 "特权"**（"blessed types"），普通函数无法实现类似行为。
    
- 这违背了 Go 的 "显式处理" 原则。


```go
str, ok := x.(string)  // 安全写法，返回 (value, bool)
str = x.(string)       // 如果失败，直接 panic（类似异常）
```


- 第一种形式（返回 `bool`）符合 Go 的错误处理风格。
    
- 第二种形式（直接 `panic`）却 **退回到了异常机制**，与 Go 的哲学矛盾。


```go
val, ok := <-ch  // 如果 channel 关闭，ok 为 false
val = <-ch       // 如果 channel 关闭，返回零值（无警告）
```


- 第一种形式可以检测 channel 是否关闭。
    
- 第二种形式 **静默接受零值**，可能导致隐蔽的 bug。



- ***对开发者的启示：***
    
    - ***需要警惕这些 "语法糖" 可能隐藏的问题。***
        
    - ***在关键代码中，始终使用完整形式（如 `val, ok := m[key]`）以避免 bug。***


## 理解异常和错误

1. 错误（`error`）——正常业务流程中的问题

本质：

- 是一种**值**（`error` 接口类型），代表函数运行时出现的问题。
    
- **你需要主动检查和处理**。


2. 异常（`panic`）——非正常流程，程序直接崩溃

本质：

- 是 Go 用来表示**程序出现严重问题时**的机制。
    
- 一旦 `panic` 被调用，当前函数就会停止执行，**逐层向上退出栈帧**，直到程序崩溃或被 `recover` 捕获

常用于：

- 数组越界
    
- nil 指针调用
    
- 程序员写错逻辑时提示开发者修复



>**错误（error）是业务可恢复的问题，异常（panic）是不可恢复、必须终止或特殊处理的问题。**  
在实际开发中，**90% 的问题都用 error 返回，不要滥用 panic**


Go 的理念之一是“通过交流分享记忆;不要通过共享内存来交流。这是标准库似乎经常打破的另一个规则。标准库中大约有 60 个通道，不包括测试。如果您浏览代码，您会发现互斥锁往往是首选，并且通常性能更好 — 稍后将对此进行详细介绍。


## sync/atomic

~~~
We want sync to be clearly documented and used when appropriate. We generally don’t want sync/atomic to be used at all…Experience has shown us again and again that very very few people are capable of writing correct code that uses atomic operations…If we had thought of internal packages when we added the sync/atomic package, perhaps we would have used that. Now we can’t remove the package because of the Go 1 guarantee.
~~~

1. **`sync` 包的定位**：
    
    - `sync` 包（如 `sync.Mutex`、`sync.WaitGroup`）是 **官方推荐** 的同步原语，应该被 **清晰地文档化** 并在合适的场景使用。
        
    - 这些高阶同步工具（如互斥锁、条件变量）已经封装了底层复杂性，普通开发者可以安全使用。
        
2. **`sync/atomic` 包的定位**：
    
    - `sync/atomic`（提供原子操作，如 `atomic.AddInt32`）**本应设计为内部包**（`internal`），因为它的正确使用需要极深的并发编程经验。
        
    - 绝大多数开发者 **无法写出正确的原子操作代码**（即使是有经验的程序员也容易犯错）。
        
    - 但由于历史原因（Go 1 兼容性承诺），现在无法移除或隐藏该包。


- ***Go 团队认为 原子操作是危险的，应该尽量避免使用，除非在极少数底层库（如运行时、标准库内部）中。***
    
- ***普通业务代码应优先使用 `sync` 包提供的更安全的抽象（如 `Mutex`），而非直接操作 `atomic`。***


*WHY？？？*


- 原子操作的正确性依赖于 **内存模型**（memory model）和 **CPU 指令顺序**（memory ordering）。
    
- 开发者需要理解 **可见性**（visibility）、**重排序**（reordering）、**ABA 问题** 等复杂概念，否则极易写出有 bug 的代码。


- 基于原子操作的代码通常难以阅读和调试（例如无锁数据结构）。
    
- 团队协作时，其他成员可能无法理解其背后的并发逻辑。


~~~
 Go 团队的历史决策反思
1. `internal` 包的缺失：

    - Go 早期没有 `internal` 包机制（限制某些包仅限标准库内部使用）。
        
    - 如果当时有，`sync/atomic` 可能会被标记为 `internal`，避免外部开发者误用。
        
2. Go 1 兼容性承诺：
    
    - Go 1 版本承诺不破坏向后兼容性，因此即使现在认识到 `atomic` 的问题，也无法移除或降级该包。
~~~


~~~
Clearly, channels are not particularly great for workload throughput, and you’re typically better off using a lock-free ring buffer or even a synchronized queue. Channels as a unit of composition tend to [fall short](https://gist.github.com/kachayev/21e7fe149bc5ae0bd878) as well. Instead, they are better suited as a coordination pattern, a mechanism for signaling and timing-related code. Ultimately, you must use channels judiciously if you are sensitive to performance.
~~~



### 原子操作和锁的区别

|特性|原子操作（`atomic`）|锁（`Mutex`）|
|---|---|---|
|**底层机制**|直接使用 CPU 原子指令（如 `CAS`、`LL/SC`）|基于操作系统调度（如 `futex`）|
|**粒度**|单变量级别（如 `int32`、`pointer`）|代码块级别（保护一段逻辑）|
|**是否阻塞**|非阻塞（硬件级原子操作，无上下文切换）|阻塞（竞争失败时，线程会休眠）|
|**适用场景**|简单变量操作（计数器、标志位）|复杂逻辑（需保护多个变量或代码段）|
|**性能**|极高（无锁，无线程切换）|较低（锁竞争时有上下文切换开销）|
|**正确性难度**|高（需理解内存模型，易写出 bug）|低（直接加锁，逻辑清晰）|



## 共享内存和CSP

共享内存去并发，和csp理论去并发 为什么会有很大的差别，为什么csp貌似是对并发更好的模型

**共享内存并发强调“状态共享”，而 CSP 并发强调“消息传递”**。  
**CSP 更容易构建正确、安全、可组合的并发程序。**

|方面|共享内存 (Shared Memory)|CSP（通信顺序进程）|
|---|---|---|
|本质|多个线程访问同一内存变量|多个进程通过通道通信|
|协作方式|读写共享变量 + 加锁|发送消息 + 阻塞等待|
|错误风险|数据竞争，死锁，难调试|更可控，天然同步|
|例子|C/C++ 中的线程 + mutex|Go 中的 goroutine + channel|


*共享内存的问题！*

- **可见性问题**：一个线程修改变量，另一个线程可能看不到（CPU 缓存、编译器优化等）
    
- **竞争条件（Race Condition）**：线程读写冲突导致不一致
    
- **加锁非常脆弱**：容易忘记加锁、锁顺序死锁、性能差、调试困难
    
- **状态耦合**：多个线程对共享数据的意图难以区分


*CSP 把“并发的核心问题”转化了：*

-  **从“如何共享变量”转为“如何传递消息”**
    
-  **通道通信是同步的，相当于自带锁机制**
    
- **goroutine 是轻量级的，天然适合大规模并发**


**CSP 不是性能最强的模型，但**：

- 对“人类开发者”更友好；
    
- 对复杂系统的“构建和演化”更稳定；
    
- 也是构建现代高并发系统时的**主流模型之一**。



***！！！ 你不是去“抢”变量，而是“请求”那个管理它的 goroutine 来操作它。***



