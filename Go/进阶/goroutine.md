# Goroutine

[参考地址](https://www.topgoer.com/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/goroutine.html)

## 同步、异步、阻塞和非阻塞

在研究I/O多路复用时，《UNIX网络编程》中，归纳总结了五种I/O模型，包括同步异步I/O：

- 阻塞 I/O (Blocking I/O)
- 非阻塞 I/O (Nonblocking I/O)
- I/O 多路复用 (I/O multiplexing)
- 信号驱动 I/O (Signal driven I/O)
- 异步 I/O (Asynchronous I/O)
  因此，在研究I/O多路复用之前，先去明确下之前一直模糊的同步异步阻塞与非阻塞概念。

### 0. 消息通知

当一个同步调用发出后，调用者要一直等待返回消息（结果）通知后，才进行后续逻辑的处理。当一个异步调用发出后，调用者无法立刻得到返回信息，需要被调用者在完成后，通过状态、通知或回调来通知调用者。

### 1. 同步异步

#### 1.1 判断标准

- 所谓**同步就是一个任务的完成需要依赖另外一个任务时，只有等待被依赖的任务完成后**，依赖的任务才能算完成，这是一种可靠的任务序列。
- 所谓**异步是不需要等待被依赖的任务完成，只是通知被依赖的任务要完成什么工作**，依赖的任务也立即执行，只要自己完成了整个任务就算完成了。

> **同步异步关注的是消息通信机制，同步的消息通信是调用者主动等待调用结果的返回，异步是指调用者不等待被调用者的处理结果，等到被调用者有了结果之后再通过状态，通知或者回调来启动调用者获取结果的后续逻辑操作。**

### 2. 阻塞与非阻塞

#### 2.1 判断标准

**阻塞调用是指调用结果返回之前，发起调用的进程会被挂起**。非阻塞调用是指，即使调用结果没有返回，调用进程也可以处理其他消息。

> 阻塞与非阻塞关注的是，程序在等待调用结果时的状态（是否被挂起）

#### 2.2 同步与阻塞的区别

- **同步调用时，调用进程可能还是可以激活的，也可以处理其他消息**，只不过根据定义，这个任务还没做完，他要继续主动去确认一些状态来判断这次调用任务是否完成，因此如果调用进程同时去做了一些事情（调用另外一个函数（进程）），要么他就需要在这两者的逻辑处理中互相切换。

> 因此，
>
> - 如果这个线程在**等待当前函数返回时**，**仍在执行其他消息处理**，那这种情况就叫做**同步非阻塞**；
> - 如果这个线程在等待当前函数返回时，**没有执行其他消息处理**，而是处于挂起等待状态，那这种情况就叫做**同步阻塞**

- 对于阻塞调用来说，调用进程会被挂起等待被调用进程（函数）返回

#### 2.3 阻塞与非阻塞性能对比

非阻塞（当其不能立刻得到结果之前，被调用函数（进程）不会阻塞当前进程），因此非阻塞可以提高CPU的使用效率，但是也增加了时间成本：线程切换时间增加，因此做设计的时候需要做balance。

### 3. 同步/异步 & 阻塞/非阻塞

通过Lucifer下载软件来举例调用方式，如下

#### 3.1 同步阻塞

进程A同步阻塞式调用进程B，A被挂起，等待B的返回结果继续进行后续逻辑。

> Lucifer启动下载后，盯着下载进度条不做其他任何事（挂起不做其他事情，阻塞）直到下载完成（等待结果，同步）。

#### 3.2 同步非阻塞

进程A调用进程B，A此时可以接收处理其他消息，但是关于调用B的后续逻辑，需要等待B的返回结果。

> Lucifer启动下载后，打开steam玩Ark了（可以做其他事情）时不时回来切屏出来看下下载进度（调用者等待调用结果，同步）

#### 3.3 异步阻塞

进程A调用进程B，B完成后回去通知A（体现异步），但是A被挂起，等待B的处理结果直到B有返回。

> Lucifer启动下载后，盯着进度条不做其他任何事（挂起不做其他任何事，阻塞），当下载完成之后下载软件会发出“叮”的一声并弹窗告知Lucifer已下载完成（被调用方完成任务后主动通知）。

#### 3.4 异步非阻塞

效率更高，A调用B， A调用完成后继续处理后续逻辑，当B处理完成任务后会将处理结果通过通知或回调的方式告知A。

> Lucifer启动下载后，打开steam去玩Ark了（调用发起者发起调用后可以去做其他事），当下载完成之后下载软件会发出“叮”的一声并弹窗高速Lucifer下载已完成（被调用方完成任务后主动通知）。

## 并发介绍

#### 进程和线程

A. **进程是程序在操作系统中的一次执行过程**，**系统进行资源分配和调度的一个独立单位**。

B. **线程是进程的一个执行实体,是CPU调度和分派的基本单位,**它是比进程更小的能**独立运行的基本单位**。

C. 一个进程可以创建和撤销多个线程;同一个进程中的多个线程之间可以**并发执行**。

#### 并发和并行

```
A. 多线程程序在一个核的cpu上运行，就是并发。
B. 多线程程序在多个核的cpu上运行，就是并行。
```

#### 并发

![并发](https://www.topgoer.com/static/7.1/1.png)

#### 并行

![并行](https://www.topgoer.com/static/7.1/2.png)

#### 协程和线程

协程：独立的栈空间，共享堆空间，调度由用户自己控制，本质上有点类似于用户级线程，这些用户级线程的调度也是自己实现的。

线程：**一个线程上可以跑多个协程**，协程是轻量级的线程。

#### goroutine 只是由官方实现的超级"线程池"。

每个实力`4~5KB`的栈内存占用和由于实现机制而大幅减少的创建和销毁开销是go高并发的根本原因。

#### 并发不是并行：

**并发主要由切换时间片来实现"同时"运行，并行则是直接利用多核实现多线程的运行**，go可以设置使用核数，以发挥多核计算机的能力。

**goroutine 奉行通过通信来共享内存，而不是共享内存来通信。**



## goroutine

goroutine的概念类似于线程，但 goroutine是由Go的运行时（runtime）调度和管理的。Go程序会智能地将 goroutine 中的任务合理地分配给每个CPU。Go语言之所以被称为现代化的编程语言，就是因为它在语言层面已经内置了调度和上下文切换的机制。

在Go语言编程中你不需要去自己写进程、线程、协程，你的技能包里只有一个技能–goroutine，当你需要让某个任务并发执行的时候，你只需要把这个任务包装成一个函数，开启一个goroutine去执行这个函数就可以了，就是这么简单粗暴。

### 与线程的关系

可以从三个角度区别：内存消耗、创建与销毀、切换。

- **内存占用**

创建一个 goroutine 的**栈内存消耗为 2 KB**，实际运行过程中，如果栈空间不够用，会自动进行扩容。创建一个 thread 则需要**消耗 1 MB 栈内存**，而且还需要一个被称为 “a guard page” 的区域用于和其他 thread 的栈空间进行隔离。

对于一个用 Go 构建的 HTTP Server 而言，对到来的每个请求，创建一个 goroutine 用来处理是非常轻松的一件事。而如果用一个使用线程作为并发原语的语言构建的服务，例如 Java 来说，每个请求对应一个线程则太浪费资源了，很快就会出 OOM 错误（Out Of Memory Error）。

- **创建和销毀**

**Thread 创建和销毀都会有巨大的消耗，因为要和操作系统打交道，是内核级的，通常解决的办法就是线程池**。而 goroutine 因为是由 Go runtime 负责管理的，**创建和销毁的消耗非常小，是用户级**。

- **切换**

**当 threads 切换时，需要保存各种寄存器，以便将来恢复**：

> 16 general purpose registers, PC (Program Counter), SP (Stack Pointer), segment registers, 16 XMM registers, FP coprocessor state, 16 AVX registers, all MSRs etc.

而 goroutines 切换只需保存**三个寄存器：Program Counter, Stack Pointer and BP**。

一般而言，线程切换会消耗 1000-1500 纳秒，一个纳秒平均可以执行 12-18 条指令。所以由于线程切换，执行指令的条数会减少 12000-18000。

Goroutine 的切换约为 200 ns，相当于 2400-3600 条指令。

因此，goroutines 切换成本比 threads 要小得多。

#### 可增长的栈

**OS线程（操作系统线程）一般都有固定的栈内存（通常为2MB）**,**一个[goroutine的栈](http://www.huamo.online/2019/06/25/%E6%B7%B1%E5%85%A5%E7%A0%94%E7%A9%B6goroutine%E6%A0%88/)在其生命周期开始时只有很小的栈（典型情况下2KB），goroutine的栈不是固定的，他可以按需增大和缩小，goroutine的栈大小限制可以达到1GB**，虽然极少会用到这个大。所以在Go语言中一次创建十万左右的goroutine也是可以的。

固定了栈的大小导致了两个问题：一是对于很多只需要很小的栈空间的线程来说是一个巨大的浪费，二是对于少数需要巨大栈空间的线程来说又面临栈溢出的风险。针对这两个问题的解决方案是：要么降低固定的栈大小，提升空间的利用率；要么增大栈的大小以允许更深的函数递归调用，但这两者是没法同时兼得的。

### 使用goroutine

Go语言中使用goroutine非常简单，只需要在调用函数的时候在前面加上go关键字，就可以为一个函数创建一个goroutine。

一个goroutine必定对应一个函数，可以创建多个goroutine去执行相同的函数。

#### 启动单个goroutine

启动goroutine的方式非常简单，只需要在调用的函数（普通函数和匿名函数）前面加上一个go关键字。

举个例子如下：

```go
func hello() {
    fmt.Println("Hello Goroutine!")
}
func main() {
    hello()
    fmt.Println("main goroutine done!")
}
```

这个示例中hello函数和下面的语句是串行的，执行的结果是打印完Hello Goroutine!后打印main goroutine done!。

接下来我们在调用hello函数前面加上关键字go，也就是启动一个goroutine去执行hello这个函数。

```go
func main() {
    go hello() // 启动另外一个goroutine去执行hello函数
    fmt.Println("main goroutine done!")
}
```

这一次的执行结果只打印了main goroutine done!，并没有打印Hello Goroutine!。为什么呢？

**Go 协程的主要性质**。

- **启动一个新的协程时，协程的调用会立即返回。与函数不同，程序控制不会去等待 Go 协程执行完毕。在调用 Go 协程之后，程序控制会立即返回到代码的下一行，忽略该协程的任何返回值。**
- **如果希望运行其他 Go 协程，Go 主协程必须继续运行着。如果 Go 主协程终止，则程序终止，于是其他 Go 协程也不会继续运行。**

**在程序启动时，Go程序就会为main()函数创建一个默认的goroutine**。

**当main()函数返回的时候该goroutine就结束了，所有在main()函数中启动的goroutine会一同结束**，main函数所在的goroutine就像是权利的游戏中的夜王，其他的goroutine都是异鬼，夜王一死它转化的那些异鬼也就全部GG了。

所以我们要想办法让main函数等一等hello函数，最简单粗暴的方式就是time.Sleep了。

```go
func main() {
    go hello() // 启动另外一个goroutine去执行hello函数
    fmt.Println("main goroutine done!")
    time.Sleep(time.Second)
}
```

执行上面的代码你会发现，这一次先打印main goroutine done!，然后紧接着打印Hello Goroutine!。

首先为什么会先打印main goroutine done!是因为我们在创建新的goroutine的时候需要花费一些时间，而此时main函数所在的goroutine是继续执行的。

#### 启动多个goroutine

在Go语言中实现并发就是这样简单，我们还可以启动多个goroutine。让我们再来一个例子： （这里使用了sync.WaitGroup来实现goroutine的同步）

```go
var wg sync.WaitGroup

func hello(i int) {
    defer wg.Done() // goroutine结束就登记-1
    fmt.Println("Hello Goroutine!", i)
}
func main() {

    for i := 0; i < 10; i++ {
        wg.Add(1) // 启动一个goroutine就登记+1
        go hello(i)
    }
    wg.Wait() // 等待所有登记的goroutine都结束
}
```

多次执行上面的代码，会发现每次打印的数字的顺序都不一致。这是因为10个goroutine是并发执行的，而goroutine的调度是随机的。

#### 注意

- 如果主协程退出了，其他任务还执行吗（运行下面的代码测试一下吧）

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // 合起来写
    go func() {
        i := 0
        for {
            i++
            fmt.Printf("new goroutine: i = %d\n", i)
            time.Sleep(time.Second)
        }
    }()
    i := 0
    for {
        i++
        fmt.Printf("main goroutine: i = %d\n", i)
        time.Sleep(time.Second)
        if i == 2 {
            break
        }
    }
}
```

主协程退出了，其他任务退出

#### goroutine调度

GPM是Go语言运行时（runtime）层面的实现，是go语言自己实现的一套调度系统。区别于操作系统调度OS线程。

- G很好理解，就是个goroutine的，里面除了存放本goroutine信息外 还有与所在P的绑定等信息。
- P管理着一组goroutine队列，P里面会存储当前goroutine运行的上下文环境（函数指针，堆栈地址及地址边界），P会对自己管理的goroutine队列做一些调度（比如把占用CPU时间较长的goroutine暂停、运行后续的goroutine等等）当自己的队列消费完了就去全局队列里取，如果全局队列里也消费完了会去其他P的队列里抢任务。
- M（machine）是Go运行时（runtime）**对操作系统内核线程的虚拟**， M与内核线程一般是一一映射的关系， 一个groutine最终是要放到M上执行的；

P与M一般也是一一对应的。他们关系是： P管理着一组G挂载在M上运行。当一个G长久阻塞在一个M上时，runtime会新建一个M，阻塞G所在的P会把其他的G 挂载在新建的M上。当旧的G阻塞完成或者认为其已经死掉时 回收旧的M。

P的个数是通过runtime.GOMAXPROCS设定（最大256），Go1.5版本之后**默认为物理线程数**。 在并发量大的时候会增加一些P和M，但不会太多，切换太频繁的话得不偿失。

单从线程调度讲，Go语言相比起其他语言的优势在于OS线程是由OS内核来调度的，goroutine则是由Go运行时（runtime）自己的调度器调度的，这个调度器使用一个称为m:n调度的技术（复用/调度m个goroutine到n个OS线程）。 其一大特点是goroutine的调度是在用户态下完成的， 不涉及内核态与用户态之间的频繁切换，包括内存的分配与释放，都是在用户态维护着一块大的内存池， 不直接调用系统的malloc函数（除非内存池需要改变），成本比调度OS线程低很多。 另一方面充分利用了多核的硬件资源，近似的把若干goroutine均分在物理线程上， 再加上本身goroutine的超轻量，以上种种保证了go调度方面的性能。

## CSP

[参考地址](https://zhuanlan.zhihu.com/p/313763247)

### CSP是什么

CSP（Communicating Sequential Process，**通信顺序进程**）。是一种并发编程模型，是一个很强大的并发数据模型，是上个世纪七十年代提出的，**用于描述两个独立的并发实体通过共享的通讯 channel(管道)进行通信的并发模型**。相对于Actor模型，CSP中channel是第一类对象，它不关注发送消息的实体，而关注与发送消息时使用的channel。

Golang，其实只用到了 CSP 的很小一部分，即理论中的 Process/Channel（对应到语言中的 goroutine/channel）：**这两个并发原语之间没有从属关系， Process 可以订阅任意个 Channel，Channel 也并不关心是哪个 Process 在利用它进行通信；Process 围绕 Channel 进行读写，形成一套有序阻塞和可预测的并发模型**。

Go语言的CSP模型是由协程Goroutine与通道Channel实现：

- **Go协程goroutine: 是一种轻量线程，它不是操作系统的线程，而是将一个操作系统线程分段使用，通过调度器实现协作式调度。是一种绿色线程，微线程，它与Coroutine协程也有区别，能够在发现堵塞后启动新的微线程。**
- **通道channel: 类似Unix的Pipe，用于协程之间通讯和同步。协程之间虽然解耦，但是它们和Channel有着耦合。**

### Channel

[底层实现](https://github.com/Simin-hub/Golang-Learning-and-Interview/blob/main/Go/%E8%BF%9B%E9%98%B6/array%E3%80%81slice%E3%80%81map%E3%80%81channel.md#channel)

Goroutine 和 channel 是 Go 语言并发编程的两大基石。Goroutine 用于执行并发任务，channel 用于 goroutine 之间的同步、通信。

Channel 在 gouroutine 间架起了一条管道，在管道里传输数据，实现 gouroutine 间的通信；由于它是线程安全的，所以用起来非常方便；channel 还提供 “先进先出” 的特性；它还能影响 goroutine 的阻塞和唤醒。

相信大家一定见过一句话：

> **Do not communicate by sharing memory; instead, share memory by communicating.**

**不要通过共享内存来通信，而要通过通信来实现内存共享。**

这就是 Go 的并发哲学，它依赖 CSP 模型，基于 channel 实现。

**channel 实现 CSP**

Channel 是 Go 语言中一个非常重要的类型，是 Go 里的第一对象。通过 channel，Go 实现了通过通信来实现内存共享。Channel 是在多个 goroutine 之间传递数据和同步的重要手段。

使用原子函数、读写锁可以保证资源的共享访问安全，但使用 channel 更优雅。

channel 字面意义是 “通道”，类似于 Linux 中的管道。声明 channel 的语法如下：

```text
chan T // 声明一个双向通道
chan<- T // 声明一个只能用于发送的通道
<-chan T // 声明一个只能用于接收的通道COPY
```

单向通道的声明，用 `<-` 来表示，它指明通道的方向。你只要明白，代码的书写顺序是从左到右就马上能掌握通道的方向是怎样的。

因为 channel 是一个引用类型，所以在它被初始化之前，它的值是 nil，channel 使用 make 函数进行初始化。可以向它传递一个 int 值，代表 channel 缓冲区的大小（容量），构造出来的是一个缓冲型的 channel；不传或传 0 的，构造的就是一个非缓冲型的 channel。

两者有一些差别：非缓冲型 channel 无法缓冲元素，对它的操作一定顺序是 “发送 -> 接收 -> 发送 -> 接收 -> ……”，如果想连续向一个非缓冲 chan 发送 2 个元素，并且没有接收的话，第一次一定会被阻塞；对于缓冲型 channel 的操作，则要 “宽松” 一些，毕竟是带了 “缓冲” 光环。

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/channelpic.jpg)

对 chan 的发送和接收操作都会在编译期间转换成为底层的发送接收函数。

Channel 分为两种：**带缓冲、不带缓冲**。对不带缓冲的 channel 进行的操作实际上可以看作 “**同步模式**”，带缓冲的则称为 “**异步模式**”。

同步模式下，发送方和接收方要同步就绪，只有在两者都 ready 的情况下，数据才能在两者间传输（后面会看到，实际上就是内存拷贝）。否则，任意一方先行进行发送或接收操作，都会被挂起，等待另一方的出现才能被唤醒。

异步模式下，在缓冲槽可用的情况下（有剩余容量），发送和接收操作都可以顺利进行。否则，操作的一方（如写入）同样会被挂起，直到出现相反操作（如接收）才会被唤醒。

**小结一下**：同步模式下，必须要使发送方和接收方配对，操作才会成功，否则会被阻塞；异步模式下，缓冲槽要有剩余容量，操作才会成功，否则也会被阻塞。

简单来说，CSP 模型由并发执行的实体（线程或者进程或者协程）所组成，实体之间通过发送消息进行通信，这里发送消息时使用的就是通道，或者叫 channel。

CSP 模型的关键是关注 channel，而不关注发送消息的实体。Go 语言实现了 CSP 部分理论，goroutine 对应 CSP 中并发执行的实体，channel 也就对应着 CSP 中的 channel。

## Goroutine 调度器

[参考地址](https://learnku.com/articles/41728)

[参考地址2](https://zboya.github.io/post/go_scheduler/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)

- runtime
  `runtime` 也叫运行时，对golang程序很重要，runtime包含了调度、内存管理、垃圾回收、内部数据结构、定时器和各种系统调用的封装等。可以说golang的强大都归功于runtime的实现。
- scheduler
  `scheduler` 有调度器、日程安排的意识，这里是指调度器，它的工作是将准备好运行的goroutine分散到工作线程中执行。
- TLS(thread local storage)
  `TLS`全称是Thread Local Storage，代表每个线程中的本地数据。**写入TLS中的数据不会干扰到其余线程中的值**。Go的协程实现非常依赖于TLS机制，会用于获取系统线程中当前的G和G所属于的M实例。 Go操作TLS会使用系统原生的接口，以Linux X64为例，go在新建M时候会调用 arch_prctl 这个syscall来设置FS寄存器的值为M.tls的地址，运行中每个M的FS寄存器都会指向它们对应的M实例的tls， linux内核调度线程时FS寄存器会跟着线程一起切换，这样go代码只需要访问FS寄存器就可以获取到线程本地的数据。
- spinning
  `spinning` 表示自旋，字面的意思是自己围绕自己转。在程序里一般指一直重复某块代码。
- systemstack、mcall或asmcgocall
  每个M启动都有一个叫g0的系统堆栈，runtime通常使用`systemstack`、`mcall`或`asmcgocall`临时**切换到系统堆栈**，以**执行必须不被抢占的任务、不得增加用户堆栈的任务或切换用户goroutines**。在系统堆栈上运行的代码隐式不可抢占，垃圾收集器不扫描系统堆栈。在系统堆栈上运行时，不会使用当前用户堆栈执行。

### Golang “调度器” 的由来？

**(1) 单进程时代不需要调度器**

我们知道，一切的软件都是跑在操作系统上，真正用来干活 (计算) 的是 CPU。早期的操作系统每个程序就是一个进程，直到一个程序运行完，才能进行下一个进程，就是 “单进程时代”

一切的程序只能串行发生。

<img src="https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/06IoYRyruP.png" style="zoom:50%;" />


早期的单进程操作系统，面临 2 个问题：

1. 单一的执行流程，计算机只能一个任务一个任务处理。

2. 进程阻塞所带来的 CPU 时间浪费。

那么能不能有多个进程来宏观一起来执行多个任务呢？

后来操作系统就具有了最早的并发能力：多进程并发，当一个进程阻塞的时候，切换到另外等待执行的进程，这样就能尽量把 CPU 利用起来，CPU 就不浪费了。

**(2) 多进程 / 线程时代有了调度器需求**

<img src="https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/sLve6TagD3.png" style="zoom:50%;" />


在多进程 / 多线程的操作系统中，就解决了阻塞的问题，因为一个进程阻塞 cpu 可以立刻切换到其他进程中去执行，而且调度 cpu 的算法可以保证在运行的进程都可以被分配到 cpu 的运行时间片。这样从宏观来看，似乎多个进程是在同时被运行。

但新的问题就又出现了，进程拥有太多的资源，进程的创建、切换、销毁，都会占用很长的时间，CPU 虽然利用起来了，但如果进程过多，CPU 有很大的一部分都被用来进行进程调度了。

**怎么才能提高 CPU 的利用率呢？**

但是对于 Linux 操作系统来讲，cpu 对进程的态度和线程的态度是一样的。

<img src="https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/dNWd00AiEZ.png" style="zoom:50%;" />


很明显，CPU 调度切换的是进程和线程。尽管线程看起来很美好，但实际上多线程开发设计会变得更加复杂，要考虑很多同步竞争等问题，如锁、竞争冲突等。

**(3) 协程来提高 CPU 利用率**

多进程、多线程已经提高了系统的并发能力，但是在当今互联网高并发场景下，**为每个任务都创建一个线程是不现实的**，因为会消耗大量的内存 (进程虚拟内存会占用 4GB [32 位操作系统], 而线程也要大约 4MB)。

大量的进程 / 线程出现了新的问题

- 高内存占用

- 调度的高消耗 CPU

好了，然后工程师们就发现，其实**一个线程分为 “内核态 “线程和” 用户态 “线程**。

一个 “用户态线程” 必须要绑定一个 “内核态线程”，但是 CPU 并不知道有 “用户态线程” 的存在，它只知道它运行的是一个 “内核态线程”(Linux 的 PCB 进程控制块)。

<img src="https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/TfStmYsfyF.png" style="zoom:50%;" />

这样，我们再去细化去分类一下，内核线程依然叫 “线程 (thread)”，用户线程叫 “协程 (co-routine)”.

<img src="https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/vgzlKzvOUL.png" style="zoom:50%;" />

 看到这里，我们就要开脑洞了，既然一个协程 (co-routine) 可以绑定一个线程 (thread)，那么能不能多个协程 (co-routine) 绑定一个或者多个线程 (thread) 上呢。

 之后，我们就看到了有 3 种协程和线程的映射关系：

#### **N:1 关系**

N 个协程绑定 1 个线程，优点就是协程在用户态线程即完成切换，不会陷入到内核态，这种切换非常的轻量快速。但也有很大的缺点，1 个进程的所有协程都绑定在 1 个线程上

缺点：

- 某个程序用不了硬件的多核加速能力
- 一旦某协程阻塞，造成线程阻塞，本进程的其他协程都无法执行了，根本就没有并发的能力了。

<img src="https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/kfPbThcyRU.png" style="zoom:50%;" />

#### **1:1 关系**

1 个协程绑定 1 个线程，这种最容易实现。协程的调度都由 CPU 完成了，不存在 N:1 缺点，

缺点：

协程的创建、删除和切换的代价都由 CPU 完成，有点略显昂贵了。

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/EhNQY2VCpk.png)

#### M:N 关系

M 个协程绑定 1 个线程，是 N:1 和 1:1 类型的结合，克服了以上 2 种模型的缺点，但实现起来最为复杂。

<img src="https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/ANDQLx3g9U.png" style="zoom:50%;" />

 协程跟线程是有区别的，**线程由 CPU 调度是抢占式的，协程由用户态调度是协作式的**，一个协程让出 CPU 后，才执行下一个协程。



**(4) Go 语言的协程 goroutine**

Go 为了提供更容易使用的并发方法，使用了 goroutine 和 channel。goroutine 来自协程的概念，让一组可复用的函数运行在一组线程之上，即使有协程阻塞，该线程的其他协程也可以被 runtime 调度，转移到其他可运行的线程上。最关键的是，程序员看不到这些底层的细节，这就降低了编程的难度，提供了更容易的并发。

Go 中，协程被称为 goroutine，它非常轻量，一个 goroutine 只占几 KB，并且这几 KB 就足够 goroutine 运行完，这就能在有限的内存空间内支持大量 goroutine，支持了更多的并发。虽然一个 goroutine 的栈只占几 KB，但实际是可伸缩的，如果需要更多内容，runtime 会自动为 goroutine 分配。

Goroutine 特点：

- 占用内存更小（几 kb）

- 调度更灵活 (runtime 调度)

**(5) 被废弃的 goroutine 调度器**

Go 目前使用的调度器是 2012 年重新设计的，因为之前的调度器性能存在问题，所以使用 4 年就被废弃了，那么我们先来分析一下被废弃的调度器是如何运作的？

大部分文章都是会用 **G 来表示 Goroutine，用 M 来表示线程**，那么我们也会用这种表达的对应关系。

<img src="https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/Af6GQ3GSlb.png" style="zoom:50%;" />

下面我们来看看被废弃的 golang 调度器是如何实现的？

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/uWk9pzdREk.png)

**M 想要执行、放回 G 都必须访问全局 G 队列，并且 M 有多个，即多线程访问同一资源需要加锁进行保证互斥 / 同步，所以全局 G 队列是有互斥锁进行保护的。**

老调度器有几个缺点：

- 创建、销毁、调度 G 都需要每个 M 获取锁，这就形成了激烈的锁竞争。
- M 转移 G 会造成延迟和额外的系统负载。比如当 G 中包含创建新协程的时候，M 创建了 G’，为了继续执行 G，需要把 G’交给 M’执行，也造成了很差的局部性，因为 G’和 G 是相关的，最好放在 M 上执行，而不是其他 M’。
- 系统调用 (CPU 在 M 之间的切换) 导致频繁的线程阻塞和取消阻塞操作增加了系统开销。

### Goroutine 调度器的 GMP 模型的设计思想

面对之前调度器的问题，Go 设计了新的调度器。

在新调度器中，除了 M (thread) 和 G (goroutine)，又引进了 P (Processor)。

<img src="https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/zaZ4nQYcZe.png" style="zoom:50%;" />

**Processor，它包含了运行 goroutine 的资源，如果线程想运行 goroutine，必须先获取 P，P 中还包含了可运行的 G 队列。**

- P表示**逻辑processor**，**代表线程M的执行的上下文**。
- P的最大作用是其拥有的**各种G对象队列、链表、cache和状态**。
- P的数量也代表了golang的执行并发度，即有多少goroutine可以同时运行

这里的p虽然表示逻辑处理器，但P并不执行任何代码，对G来说，P相当于CPU核，G只有绑定到P才能被调度。 对M来说，P提供了相关的执行环境(Context)，如内存分配状态(mcache)，任务队列(G)等

**M（machine）**

- **M代表着真正的执行计算资源**，可以认为它就是os thread（系统线程）。
- M是真正调度系统的执行者，每个M就像一个勤劳的工作者，总是从各种队列中找到可运行的G，而且这样M的可以同时存在多个。
- M在绑定有效的P后，进入调度循环，而且M并不保留G状态，这是G可以跨M调度的基础。

#### (1) GMP 模型

在 Go 中，线程是运行 goroutine 的实体，调度器的功能是把可运行的 goroutine 分配到工作线程上。

<img src="https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/Ugu3C2WSpM.jpeg"  />

**全局队列（Global Queue）**：存放等待运行的 G。

**P 的本地队列**：同全局队列类似，存放的也是等待运行的 G，**存的数量有限，不超过 256 个**。新建 G 时，G 优先加入到 P 的本地队列，如果队列满了，则会把**本地队列中一半**的 G 移动到全局队列。

**P 列表**：所有的 P 都在程序启动时创建，并保存在数组中，**最多有 GOMAXPROCS(可配置) 个**。

**M**：线程想运行任务就得获取 P，从 P 的本地队列获取 G，P 队列为空时，M 也会尝试从全局队列拿一批 G 放到 P 的本地队列，或从其他 P 的本地队列偷一半放到自己 P 的本地队列。M 运行 G，G 执行之后，M 会从 P 获取下一个 G，不断重复下去。

**Goroutine 调度器和 OS 调度器是通过 M 结合起来的，每个 M 都代表了 1 个内核线程，OS 调度器负责把内核线程分配到 CPU 的核上执行。**

##### 有关 P 和 M 的个数问题

**1、P 的数量：**

P的个数是通过runtime.GOMAXPROCS设定（最大256），Go1.5版本之后**默认为物理线程数**。启动时环境变量 `GOMAXPROCS `或者是由 runtime 的方法 `GOMAXPROCS() `决定。这意味着在程序执行的**任意时刻都只有 `GOMAXPROCS` 个 goroutine 在同时运行**。

**2、M 的数量:**

- go 语言本身的限制：**go 程序启动时，会设置 M 的最大数量，默认 10000**. 但是内核很难支持这么多的线程数，所以这个限制可以忽略。

- runtime/debug 中的 SetMaxThreads 函数，设置 M 的最大数量
- 一个 M 阻塞了，会创建新的 M。
- M 与 P 的数量没有绝对关系，一个 M 阻塞，P 就会去创建或者切换另一个 M，所以，即使 P 的默认数量是 1，也有可能会创建很多个 M 出来。

##### P 和 M 何时会被创建

1、P 何时创建：在确定了 P 的最大数量 n 后，运行时系统会根据这个数量创建 n 个 P。

2、M 何时创建：**没有足够的 M 来关联 P 并运行其中的可运行的 G**。比如所有的 M 此时都阻塞住了，而 P 中还有很多就绪任务，就会去寻找空闲的 M，而没有空闲的，就会去创建新的 M。

#### (2) 调度器的设计策略

**复用线程：避免频繁的创建、销毁线程，而是对线程的复用。**

1）**work stealing 机制**

 当本线程无可运行的 G 时，尝试从其他线程绑定的 P 偷取 G，而不是销毁线程。

2）**hand off 机制**

 当本线程因为 G 进行系统调用阻塞时，线程释放绑定的 P，把 P 转移给其他空闲的线程执行。

利用并行：GOMAXPROCS 设置 P 的数量，最多有 GOMAXPROCS 个线程分布在多个 CPU 上同时运行。GOMAXPROCS 也限制了并发的程度，比如 **GOMAXPROCS = 核数/2**，则**最多利用了一半的 CPU 核进行并行**。

抢占：在 coroutine 中要等待一个协程主动让出 CPU 才执行下一个协程，在 Go 中，**一个 goroutine 最多占用 CPU 10ms**，防止其他 goroutine 被饿死，这就是 goroutine 不同于 coroutine 的一个地方。

全局 G 队列：在新的调度器中依然有全局 G 队列，但功能已经被弱化了，当 M 执行 work stealing 从其他 P 偷不到 G 时，它可以从全局 G 队列获取 G。

#### **(3) go func () 调度流程**

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/a4vWtvRWGQ.jpeg)

从上图我们可以分析出几个结论：

 1、**我们通过 go func () 来创建一个 goroutine**；

 2、有两个存储 G 的队列，一个是局部调度器 P 的本地队列、一个是全局 G 队列。新创建的 G 会先保存在 P 的本地队列中，如果 P 的本地队列已经满了就会保存在全局的队列中；

 3、G 只能运行在 M 中，一个 M 必须持有一个 P，M 与 P 是 1：1 的关系。M 会从 P 的本地队列弹出一个可执行状态的 G 来执行，如果 P 的本地队列为空，就会想其他的 MP 组合偷取一个可执行的 G 来执行；

 4、一个 M 调度 G 执行的过程是一个循环机制；

 5、当 M 执行某一个 G 时候如果发生了 syscall 或则其余阻塞操作，M 会阻塞，如果当前有一些 G 在执行，runtime 会把这个线程 M 从 P 中摘除 (detach)，然后再创建一个新的操作系统的线程 (如果有空闲的线程可用就复用空闲线程) 来服务于这个 P；

 6、当 M 系统调用结束时候，这个 G 会尝试获取一个空闲的 P 执行，并放入到这个 P 的本地队列。如果获取不到 P，那么这个线程 M 变成休眠状态， 加入到空闲线程中，然后这个 G 会被放入全局队列中。

#### (4) 调度器的生命周期

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/j37FX8nek9.png)

**特殊的 M0 和 G0**

**M0**

**M0 是启动程序后的编号为 0 的主线程**，这个 M 对应的实例会在全局变量 runtime.m0 中，不需要在 heap 上分配，**M0 负责执行初始化操作和启动第一个 G**， 在之后 M0 就和其他的 M 一样了。

**G0**

**G0 是每次启动一个 M 都会第一个创建的 goroutine**，**G0 仅用于负责调度的 G**，G0 不指向任何可执行的函数，**每个 M 都会有一个自己的 G0**。在调度或系统调用时会使用 G0 的栈空间，全局变量的 G0 是 M0 的 G0。

我们来跟踪一段代码

```go
package main

import "fmt"

func main() {
    fmt.Println("Hello world")
}
```

接下来我们来针对上面的代码对调度器里面的结构做一个分析。

也会经历如上图所示的过程：

1. runtime 创建最初的线程 m0 和 goroutine g0，并把 2 者关联。

2. **调度器初始化：初始化 m0、栈、垃圾回收，以及创建和初始化由 GOMAXPROCS 个 P 构成的 P 列表。**

3. 示例代码中的 main 函数是 main.main，runtime 中也有 1 个 main 函数 ——runtime.main，代码经过编译后，runtime.main 会调用 main.main，程序启动时会为 runtime.main 创建 goroutine，称它为 main goroutine 吧，然后把 main goroutine 加入到 P 的本地队列。

4. 启动 m0，m0 已经绑定了 P，会从 P 的本地队列获取 G，获取到 main goroutine。

5. G 拥有栈，M 根据 G 中的栈信息和调度信息设置运行环境

6. M 运行 G

7. G 退出，再次回到 M 获取可运行的 G，这样重复下去，直到 main.main 退出，runtime.main 执行 Defer 和 Panic 处理，或调用 runtime.exit 退出程序。

8. 调度器的生命周期几乎占满了一个 Go 程序的一生，runtime.main 的 goroutine 执行之前都是为调度器做准备工作，**runtime.main 的 goroutine 运行，才是调度器的真正开始，直到 runtime.main 结束而结束**。


#### (5) 可视化 GMP 编程

有 2 种方式可以查看一个程序的 GMP 的数据。

方式 1：**go tool trace**

trace 记录了运行时的信息，能提供可视化的 Web 页面。

简单测试代码：main 函数创建 trace，trace 会运行在单独的 goroutine 中，然后 main 打印”Hello World” 退出。

trace.go

```go
package main

import (
    "os"
    "fmt"
    "runtime/trace"
)

func main() {

    //创建trace文件
    f, err := os.Create("trace.out")
    if err != nil {
        panic(err)
    }

    defer f.Close()

    //启动trace goroutine
    err = trace.Start(f)
    if err != nil {
        panic(err)
    }
    defer trace.Stop()

    //main
    fmt.Println("Hello World")
}
```

运行程序

```
$ go run trace.go 
Hello World
```

会得到一个 trace.out 文件，然后我们可以用一个工具打开，来分析这个文件。

```
$ go tool trace trace.out 
2020/02/23 10:44:11 Parsing trace...
2020/02/23 10:44:11 Splitting trace...
2020/02/23 10:44:11 Opening browser. Trace viewer is listening on http://127.0.0.1:33479
```

我们可以通过浏览器打开 http://127.0.0.1:33479 网址，点击 view trace 能够看见可视化的调度流程。

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/Xr9qi3emlx.png)

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/vYyO9YJmam.png)

G 信息

点击 Goroutines 那一行可视化的数据条，我们会看到一些详细的信息。

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/QLm0KK1hhd.png)

  一共有两个G在程序中，一个是特殊的G0，是每个M必须有的一个初始化的G，这个我们不必讨论。
其中 G1 应该就是 main goroutine (执行 main 函数的协程)，在一段时间内处于可运行和运行的状态。

M 信息

点击 Threads 那一行可视化的数据条，我们会看到一些详细的信息。

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/mYYA4V17yF.png)

一共有两个 M 在程序中，一个是特殊的 M0，用于初始化使用，这个我们不必讨论。

P 信息

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/QbWwbth8uN.png)


G1 中调用了 main.main，创建了 trace goroutine g18。G1 运行在 P1 上，G18 运行在 P0 上。

这里有两个 P，我们知道，一个 P 必须绑定一个 M 才能调度 G。

我们在来看看上面的 M 信息。

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/5kS6NfpQAI.png)

我们会发现，确实 G18 在 P0 上被运行的时候，确实在 Threads 行多了一个 M 的数据，点击查看如下：

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/EN1OESafVZ.png)

多了一个 M2 应该就是 P0 为了执行 G18 而动态创建的 M2.

**方式 2：Debug trace**

```
package main

import (
    "fmt"
    "time"
)

func main() {
    for i := 0; i < 5; i++ {
        time.Sleep(time.Second)
        fmt.Println("Hello World")
    }
}
```

编译

```
$ go build trace2.go
```

通过 Debug 方式运行

```
$ GODEBUG=schedtrace=1000 ./trace2 
SCHED 0ms: gomaxprocs=2 idleprocs=0 threads=4 spinningthreads=1 idlethreads=1 runqueue=0 [0 0]
Hello World
SCHED 1003ms: gomaxprocs=2 idleprocs=2 threads=4 spinningthreads=0 idlethreads=2 runqueue=0 [0 0]
Hello World
SCHED 2014ms: gomaxprocs=2 idleprocs=2 threads=4 spinningthreads=0 idlethreads=2 runqueue=0 [0 0]
Hello World
SCHED 3015ms: gomaxprocs=2 idleprocs=2 threads=4 spinningthreads=0 idlethreads=2 runqueue=0 [0 0]
Hello World
SCHED 4023ms: gomaxprocs=2 idleprocs=2 threads=4 spinningthreads=0 idlethreads=2 runqueue=0 [0 0]
Hello World
```

- SCHED：调试信息输出标志字符串，代表本行是 goroutine 调度器的输出；

- 0ms：即从程序启动到输出这行日志的时间；
- gomaxprocs: P 的数量，本例有 2 个 P, 因为**默认的 P 的属性是和 cpu 核心数量默认一致**，当然也可以通过 GOMAXPROCS 来设置；
- idleprocs: 处于 idle 状态的 P 的数量；通过 gomaxprocs 和 idleprocs 的差值，我们就可知道执行 go 代码的 P 的数量；
- threads: os threads/M 的数量，包含 scheduler 使用的 m 数量，加上 runtime 自用的类似 sysmon 这样的 thread 的数量；
- spinningthreads: 处于自旋状态的 os thread 数量；
- idlethread: 处于 idle 状态的 os thread 的数量；
- runqueue=0： Scheduler 全局队列中 G 的数量；
- [0 0]: 分别为 2 个 P 的 local queue 中的 G 的数量。

### Go 调度器调度场景过程全解析

**(1) 场景 1**

P 拥有 G1，M1 获取 P 后开始运行 G1，G1 使用 go func() 创建了 G2，为了局部性 G2 优先加入到 P1 的本地队列。

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/Pm8LOYcsWQ.png)

(2) 场景 2

G1 运行完成后 (函数：`goexit`)，M 上运行的 goroutine 切换为 G0，G0 负责调度时协程的切换（函数：`schedule`）。从 P 的本地队列取 G2，从 G0 切换到 G2，并开始运行 G2 (函数：execute)。实现了线程 M1 的复用。

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/JWDtmKG3rK.png)

(3) 场景 3

假设每个 P 的本地队列只能存 3 个 G。G2 要创建了 6 个 G，前 3 个 G（G3, G4, G5）已经加入 p1 的本地队列，p1 本地队列满了。

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/UpjRxzIBd3.png)

(4) 场景 4

G2 在创建 G7 的时候，发现 P1 的本地队列已满，需要执行负载均衡 (把 P1 中本地队列中前一半的 G，还有新创建 G 转移到全局队列)

（实现中并不一定是新的 G，如果 G 是 G2 之后就执行的，会被保存在本地队列，利用某个老的 G 替换新 G 加入全局队列）

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/chqTgsiuWi.png)

这些 G 被转移到全局队列时，会被打乱顺序。所以 G3,G4,G7 被转移到全局队列。

(5) 场景 5

G2 创建 G8 时，P1 的本地队列未满，所以 G8 会被加入到 P1 的本地队列。

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/nukEY92G6D.png)

G8 加入到 P1 点本地队列的原因还是因为 P1 此时在与 M1 绑定，而 G2 此时是 M1 在执行。所以 G2 创建的新的 G 会优先放置到自己的 M 绑定的 P 上。

(6) 场景 6

规定：**在创建 G 时，运行的 G 会尝试唤醒其他空闲的 P 和 M 组合去执行。**

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/2FWNXSuHfX.png)

假定 G2 唤醒了 M2，M2 绑定了 P2，并运行 G0，但 P2 本地队列没有 G，**M2 此时为自旋线程（**没有 G 但为运行状态的线程，不断寻找 G）。

(7) 场景 7

M2 尝试从全局队列 (简称 “GQ”) 取一批 G 放到 P2 的本地队列（函数：findrunnable()）。M2 从全局队列取的 G 数量符合下面的公式：

$$n = min(len(GQ)/GOMAXPROCS + 1, len(GQ/2))$$

至少从全局队列取 1 个 g，但每次不要从全局队列移动太多的 g 到 p 本地队列，给其他 p 留点。**这是从全局队列到 P 本地队列的负载均衡。**

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/0fn8DGqI8N.jpeg)

假定我们场景中一共有 4 个 P（GOMAXPROCS 设置为 4，那么我们允许最多就能用 4 个 P 来供 M 使用）。所以 M2 只从能从全局队列取 1 个 G（即 G3）移动 P2 本地队列，然后完成从 G0 到 G3 的切换，运行 G3。

(8) 场景 8

假设 G2 一直在 M1 上运行，经过 2 轮后，M2 已经把 G7、G4 从全局队列获取到了 P2 的本地队列并完成运行，全局队列和 P2 的本地队列都空了，如场景 8 图的左半部分。

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/qn1NRMLqnp.png)

**全局队列已经没有 G，那 m 就要执行 work stealing (偷取)**：从其他有 G 的 P 哪里偷取一半 G 过来，放到自己的 P 本地队列。P2 从 P1 的本地队列尾部取一半的 G，本例中一半则只有 1 个 G8，放到 P2 的本地队列并执行。

(9) 场景 9

G1 本地队列 G5、G6 已经被其他 M 偷走并运行完成，当前 M1 和 M2 分别在运行 G2 和 G8，M3 和 M4 没有 goroutine 可以运行，M3 和 M4 处于自旋状态，它们不断寻找 goroutine。

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/1DjlseEGTT.png)

**为什么要让 m3 和 m4 自旋，自旋本质是在运行，线程在运行却没有执行 G**，就变成了浪费 CPU. 为什么不销毁现场，来节约 CPU 资源。因为创建和销毁 CPU 也会浪费时间，我们希望**当有新 goroutine 创建时，立刻能有 M 运行它，如果销毁再新建就增加了时延，降低了效率**。当然也考虑了过多的自旋线程是浪费 CPU，所以**系统中最多有 GOMAXPROCS 个自旋的线程 (**当前例子中的 GOMAXPROCS=4，所以一共 4 个 P)，多余的没事做线程会让他们休眠。

(10) 场景 10

假定当前除了 M3 和 M4 为自旋线程，还有 M5 和 M6 为空闲的线程 (没有得到 P 的绑定，注意我们这里最多就只能够存在 4 个 P，所以 **P 的数量应该永远是 M>=P**, 大部分都是 M 在抢占需要运行的 P)，G8 创建了 G9，G8 进行了**阻塞的系统调用**，**M2 和 P2 立即解绑，P2 会执行以下判断：如果 P2 本地队列有 G、全局队列有 G 或有空闲的 M，P2 都会立马唤醒 1 个 M 和它绑定，否则 P2 则会加入到空闲 P 列表，等待 M 来获取可用的 p**。本场景中，P2 本地队列有 G9，可以和其他空闲的线程 M5 绑定。

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/k3HKE9U21M.png)

(11) 场景 11

G8 创建了 G9，假如 G8 进行了非阻塞系统调用。

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/zBvpl8ENSb.png) 


 **M2 和 P2会解绑，但 M2 会记住 P2，然后 G8 和 M2 进入系统调用状态**。当 G8 和 M2 退出系统调用时，会尝试获取 P2，如果无法获取，则获取空闲的 P，如果依然没有，G8 会被记为可运行状态，并加入到全局队列，M2 因为没有 P 的绑定而变成休眠状态 (长时间休眠等待 GC 回收销毁)。

### 调度时机

在四种情形下，goroutine 可能会发生调度，但也并不一定会发生，只是说 Go scheduler 有机会进行调度。

|      情形       |                             说明                             |
| :-------------: | :----------------------------------------------------------: |
| 使用关键字 `go` |      go 创建一个新的 goroutine，Go scheduler 会考虑调度      |
|       GC        | 由于进行 GC 的 goroutine 也需要在 M 上运行，因此肯定会发生调度。当然，Go scheduler 还会做很多其他的调度，例如调度不涉及堆访问的 goroutine 来运行。**GC 不管栈上的内存，只会回收堆上的内存** |
|    系统调用     | 当 goroutine 进行系统调用时，会阻塞 M，所以它会被调度走，同时一个新的 goroutine 会被调度上来 |
|  内存同步访问   | atomic，mutex，channel 操作等会使 goroutine 阻塞，因此会被调度走。等条件满足后（例如其他 goroutine 解锁了）还会被调度上来继续运行 |

### 小结

总结，Go 调度器很轻量也很简单，足以撑起 goroutine 的调度工作，并且让 Go 具有了原生（强大）并发的能力。Go 调度本质是把大量的 goroutine 分配到少量线程上去执行，并利用多核并行，实现更强大的并发。

## 常见的并发模式

[参考地址](https://www.bookstack.cn/read/advanced-go-programming-book/ch1-basic-ch1-06-goroutine.md)

### 生产者消费者模型

并发编程中最常见的例子就是生产者消费者模式，**该模式主要通过平衡生产线程和消费线程的工作能力来提高程序的整体处理数据的速度**。简单地说，就是生产者生产一些数据，然后放到成果队列中，同时消费者从成果队列中来取这些数据。这样就让生产消费变成了异步的两个过程。当成果队列中没有数据时，消费者就进入饥饿的等待中；而当成果队列中数据已满时，生产者则面临因产品挤压导致CPU被剥夺的下岗问题。

Go语言实现生产者消费者并发很简单：

```go
// 生产者: 生成 factor 整数倍的序列
func Producer(factor int, out chan<- int) {
    for i := 0; ; i++ {
        out <- i*factor
    }
}
// 消费者
func Consumer(in <-chan int) {
    for v := range in {
        fmt.Println(v)
    }
}
func main() {
    ch := make(chan int, 64) // 成果队列
    go Producer(3, ch) // 生成 3 的倍数的序列
    go Producer(5, ch) // 生成 5 的倍数的序列
    go Consumer(ch)    // 消费 生成的队列
    // 运行一定时间后退出
    time.Sleep(5 * time.Second)
}
```

### 发布订阅模型

发布订阅（publish-and-subscribe）模型通常被简写为pub/sub模型。在这个模型中，**消息生产者成为发布者（publisher），而消息消费者则成为订阅者（subscriber）**，**生产者和消费者是M:N的关系**。在传统生产者和消费者模型中，是将消息发送到一个队列中，而发布订阅模型则是将消息发布给一个主题。

为此，我们构建了一个名为`pubsub`的发布订阅模型支持包：

```go
// Package pubsub implements a simple multi-topic pub-sub library.
package pubsub
import (
    "sync"
    "time"
)
type (
    subscriber chan interface{}         // 订阅者为一个管道
    topicFunc  func(v interface{}) bool // 主题为一个过滤器
)
// 发布者对象
type Publisher struct {
    m           sync.RWMutex             // 读写锁
    buffer      int                      // 订阅队列的缓存大小
    timeout     time.Duration            // 发布超时时间
    subscribers map[subscriber]topicFunc // 订阅者信息
}
// 构建一个发布者对象, 可以设置发布超时时间和缓存队列的长度
func NewPublisher(publishTimeout time.Duration, buffer int) *Publisher {
    return &Publisher{
        buffer:      buffer,
        timeout:     publishTimeout,
        subscribers: make(map[subscriber]topicFunc),
    }
}
// 添加一个新的订阅者，订阅全部主题
func (p *Publisher) Subscribe() chan interface{} {
    return p.SubscribeTopic(nil)
}
// 添加一个新的订阅者，订阅过滤器筛选后的主题
func (p *Publisher) SubscribeTopic(topic topicFunc) chan interface{} {
    ch := make(chan interface{}, p.buffer)
    p.m.Lock()
    p.subscribers[ch] = topic
    p.m.Unlock()
    return ch
}
// 退出订阅
func (p *Publisher) Evict(sub chan interface{}) {
    p.m.Lock()
    defer p.m.Unlock()
    delete(p.subscribers, sub)
    close(sub)
}
// 发布一个主题
func (p *Publisher) Publish(v interface{}) {
    p.m.RLock()
    defer p.m.RUnlock()
    var wg sync.WaitGroup
    for sub, topic := range p.subscribers {
        wg.Add(1)
        go p.sendTopic(sub, topic, v, &wg)
    }
    wg.Wait()
}
// 关闭发布者对象，同时关闭所有的订阅者管道。
func (p *Publisher) Close() {
    p.m.Lock()
    defer p.m.Unlock()
    for sub := range p.subscribers {
        delete(p.subscribers, sub)
        close(sub)
    }
}
// 发送主题，可以容忍一定的超时
func (p *Publisher) sendTopic(sub subscriber, topic topicFunc, v interface{}, wg *sync.WaitGroup) {
    defer wg.Done()
    if topic != nil && !topic(v) {
        return
    }
    select {
    case sub <- v:
    case <-time.After(p.timeout):
    }
}
```

下面的例子中，有两个订阅者分别订阅了全部主题和含有”golang”的主题：

```go
import "path/to/pubsub"
func main() {
    p := pubsub.NewPublisher(100*time.Millisecond, 10)
    defer p.Close()
    all := p.Subscribe()
    golang := p.SubscribeTopic(func(v interface{}) bool {
        if s, ok := v.(string); ok {
            return strings.Contains(s, "golang")
        }
        return false
    })
    p.Publish("hello,  world!")
    p.Publish("hello, golang!")
    go func() {
        for  msg := range all {
            fmt.Println("all:", msg)
        }
    } ()
    go func() {
        for  msg := range golang {
            fmt.Println("golang:", msg)
        }
    } ()
    // 运行一定时间后退出
    time.Sleep(3 * time.Second)
}
```

在发布订阅模型中，**每条消息都会传送给多个订阅者。发布者通常不会知道、也不关心哪一个订阅者正在接收主题消息。**订阅者和发布者可以在运行时动态添加，是一种松散的耦合关系，这使得系统的复杂性可以随时间的推移而增长。在现实生活中，像天气预报之类的应用就可以应用这个并发模式。

## 控制并发数

### 利用 channel 的缓存区

可以利用信道 channel 的缓冲区大小来实现：

```
// main_chan.go
func main() {
	var wg sync.WaitGroup
	ch := make(chan struct{}, 3)
	for i := 0; i < 10; i++ {
		ch <- struct{}{}
		wg.Add(1)
		go func(i int) {
			defer wg.Done()
			log.Println(i)
			time.Sleep(time.Second)
			<-ch
		}(i)
	}
	wg.Wait()
}
```

- `make(chan struct{}, 3)` 创建缓冲区大小为 3 的 channel，在没有被接收的情况下，至多发送 3 个消息则被阻塞。
- 开启协程前，调用 `ch <- struct{}{}`，若缓存区满，则阻塞。
- 协程任务结束，调用 `<-ch` 释放缓冲区。
- `sync.WaitGroup` 并不是必须的，例如 http 服务，每个请求天然是并发的，此时使用 channel 控制并发处理的任务数量，就不需要 `sync.WaitGroup`。

运行结果如下：

```
$ go run main_chan.go
2020/12/21 00:48:28 2
2020/12/21 00:48:28 0
2020/12/21 00:48:28 1
2020/12/21 00:48:29 3
2020/12/21 00:48:29 4
2020/12/21 00:48:29 5
2020/12/21 00:48:30 6
2020/12/21 00:48:30 7
2020/12/21 00:48:30 8
2020/12/21 00:48:31 9
```

从日志中可以很容易看到，每秒钟只并发执行了 3 个任务，达到了协程并发控制的目的。

### 利用第三方库

目前有很多第三方库实现了协程池，可以很方便地用来控制协程的并发数量，比较受欢迎的有：

- [Jeffail/tunny](https://github.com/Jeffail/tunny)
- [panjf2000/ants](https://github.com/panjf2000/ants)

以 `tunny` 举例：

```
package main

import (
	"log"
	"time"

	"github.com/Jeffail/tunny"
)

func main() {
	pool := tunny.NewFunc(3, func(i interface{}) interface{} {
		log.Println(i)
		time.Sleep(time.Second)
		return nil
	})
	defer pool.Close()

	for i := 0; i < 10; i++ {
		go pool.Process(i)
	}
	time.Sleep(time.Second * 4)
}
```

- `tunny.NewFunc(3, f)` 第一个参数是协程池的大小(poolSize)，第二个参数是协程运行的函数(worker)。
- `pool.Process(i)` 将参数 i 传递给协程池定义好的 worker 处理。
- `pool.Close()` 关闭协程池。

运行结果如下：

```
$ go run main_tunny.go
2020/12/21 01:00:21 6
2020/12/21 01:00:21 1
2020/12/21 01:00:21 3
2020/12/21 01:00:22 8
2020/12/21 01:00:22 4
2020/12/21 01:00:22 7
2020/12/21 01:00:23 5
2020/12/21 01:00:23 2
2020/12/21 01:00:23 0
2020/12/21 01:00:24 9
```

在Go语言自带的godoc程序实现中有一个`vfs`的包对应虚拟的文件系统，在`vfs`包下面有一个`gatefs`的子包，`gatefs`子包的目的就是为了控制访问该虚拟文件系统的最大并发数。`gatefs`包的应用很简单：

```go
import (
    "golang.org/x/tools/godoc/vfs"
    "golang.org/x/tools/godoc/vfs/gatefs"
)
func main() {
    fs := gatefs.New(vfs.OS("/path"), make(chan bool, 8))
    // ...
}
```

其中`vfs.OS("/path")`基于本地文件系统构造一个虚拟的文件系统，然后`gatefs.New`基于现有的虚拟文件系统构造一个并发受控的虚拟文件系统。**并发数控制的原理就是通过带缓存管道的发送和接收规则来实现最大并发阻塞**：

```go
var limit = make(chan int, 3)
func main() {
    for _, w := range work {
        go func() {
            limit <- 1
            w()
            <-limit
        }()
    }
    select{}
}
```

不过`gatefs`对此做一个抽象类型`gate`，增加了`enter`和`leave`方法分别对应并发代码的进入和离开。当超出并发数目限制的时候，`enter`方法会阻塞直到并发数降下来为止。

```go
type gate chan bool
func (g gate) enter() { g <- true }
func (g gate) leave() { <-g }
```

`gatefs`包装的新的虚拟文件系统就是将需要控制并发的方法增加了`enter`和`leave`调用而已：

```go
type gatefs struct {
    fs vfs.FileSystem
    gate
}
func (fs gatefs) Lstat(p string) (os.FileInfo, error) {
    fs.enter()
    defer fs.leave()
    return fs.fs.Lstat(p)
}
```

我们不仅可以控制最大的并发数目，而且可以通过带缓存Channel的使用量和最大容量比例来判断程序运行的并发率。当管道为空的时候可以认为是空闲状态，当管道满了时任务是繁忙状态，这对于后台一些低级任务的运行是有参考价值的。

### 调整系统资源的上限

#### ulimit

有些场景下，即使我们有效地限制了协程的并发数量，但是仍旧出现了某一类资源不足的问题，例如：

- too many open files
- out of memory
- …

例如分布式编译加速工具，需要解析 gcc 命令以及依赖的源文件和头文件，有些编译命令依赖的头文件可能有上百个，那这个时候即使我们将协程的并发数限制到 1000，也可能会超过进程运行时并发打开的文件句柄数量，但是分布式编译工具，仅将依赖的源文件和头文件分发到远端机器执行，并不会消耗本机的内存和 CPU 资源，因此 1000 个并发并不高，这种情况下，降低并发数会影响编译加速的效率，那能不能增加进程能同时打开的文件句柄数量呢？

操作系统通常会限制同时打开文件数量、栈空间大小等，`ulimit -a` 可以看到系统当前的设置：

```
$ ulimit -a
-t: cpu time (seconds)              unlimited
-f: file size (blocks)              unlimited
-d: data seg size (kbytes)          unlimited
-s: stack size (kbytes)             8192
-c: core file size (blocks)         0
-v: address space (kbytes)          unlimited
-l: locked-in-memory size (kbytes)  unlimited
-u: processes                       1418
-n: file descriptors                12800
```

我们可以使用 `ulimit -n 999999`，将同时打开的文件句柄数量调整为 999999 来解决这个问题，其他的参数也可以按需调整。

#### 虚拟内存(virtual memory)

虚拟内存是一项非常常见的技术了，即在内存不足时，将磁盘映射为内存使用，比如 linux 下的交换分区(swap space)。

在 linux 上创建并使用交换分区是一件非常简单的事情：

```
sudo fallocate -l 20G /mnt/.swapfile # 创建 20G 空文件
sudo mkswap /mnt/.swapfile    # 转换为交换分区文件
sudo chmod 600 /mnt/.swapfile # 修改权限为 600
sudo swapon /mnt/.swapfile    # 激活交换分区
free -m # 查看当前内存使用情况(包括交换分区)
```

关闭交换分区也非常简单：

```
sudo swapoff /mnt/.swapfile
rm -rf /mnt/.swapfile
```

磁盘的 I/O 读写性能和内存条相差是非常大的，例如 DDR3 的内存条读写速率很容易达到 20GB/s，但是 SSD 固态硬盘的读写性能通常只能达到 0.5GB/s，相差 40倍之多。因此，使用虚拟内存技术将硬盘映射为内存使用，显然会对性能产生一定的影响。如果应用程序只是在较短的时间内需要较大的内存，那么虚拟内存能够有效避免 `out of memory` 的问题。如果应用程序长期高频度读写大量内存，那么虚拟内存对性能的影响就比较明显了。

## 赢者为王

采用并发编程的动机有很多：并发编程可以简化问题，比如一类问题对应一个处理线程会更简单；并发编程还可以提升性能，在一个多核CPU上开2个线程一般会比开1个线程快一些。其实对于提升性能而言，程序并不是简单地运行速度快就表示用户体验好的；很多时候程序能快速响应用户请求才是最重要的，当没有用户请求需要处理的时候才合适处理一些低优先级的后台任务。

假设我们想快速地搜索“golang”相关的主题，我们可能会同时打开Bing、Google或百度等多个检索引擎。当某个搜索最先返回结果后，就可以关闭其它搜索页面了。因为受网络环境和搜索引擎算法的影响，某些搜索引擎可能很快返回搜索结果，某些搜索引擎也可能等到他们公司倒闭也没有完成搜索。我们可以采用类似的策略来编写这个程序：

```go
func main() {    
    ch := make(chan string, 32)    
    go func() {       
        ch <- searchByBing("golang")    
    }()    
    go func() {        
    	ch <- searchByGoogle("golang")    
    }()    
    go func() {        
    	ch <- searchByBaidu("golang")    
    }()    
fmt.Println(<-ch)}
```

首先，我们创建了一个带缓存的管道，管道的缓存数目要足够大，保证不会因为缓存的容量引起不必要的阻塞。然后我们开启了多个后台线程，分别向不同的搜索引擎提交搜索请求。当任意一个搜索引擎最先有结果之后，都会马上将结果发到管道中（因为管道带了足够的缓存，这个过程不会阻塞）。但是最终我们只从管道取第一个结果，也就是最先返回的结果。

通过适当开启一些冗余的线程，尝试用不同途径去解决同样的问题，最终以赢者为王的方式提升了程序的相应性能。

## 并发的安全退出

### select

有时候我们需要通知goroutine停止它正在干的事情，特别是当它工作在错误的方向上的时候。Go语言并没有提供在一个直接终止Goroutine的方法，由于这样会导致goroutine之间的共享变量处在未定义的状态上。但是如果我们想要退出两个或者任意多个Goroutine怎么办呢？

Go语言中不同Goroutine之间主要依靠管道进行通信和同步。要同时处理多个管道的发送或接收操作，我们需要使用`select`关键字（这个关键字和网络编程中的`select`函数的行为类似）。**当`select`有多个分支时，会随机选择一个可用的管道分支，如果没有可用的管道分支则选择`default`分支，否则会一直保存阻塞状态**。

基于`select`实现的管道的超时判断：

```go
select {
case v := <-in:
    fmt.Println(v)
case <-time.After(time.Second):
    return // 超时
}
```

通过`select`的`default`分支实现非阻塞的管道发送或接收操作：

```go
func main() {
    // do some thins
    select{}
}
```

当有多个管道均可操作时，`select`会随机选择一个管道。基于该特性我们可以用`select`实现一个生成随机数序列的程序：

```
func main() {
    ch := make(chan int)
    go func() {
        for {
            select {
            case ch <- 0:
            case ch <- 1:
            }
        }
    }()
    for v := range ch {
        fmt.Println(v)
    }
}
```

我们通过`select`和`default`分支可以很容易实现一个Goroutine的退出控制:

```
func worker(cannel chan bool) {
    for {
        select {
        default:
            fmt.Println("hello")
            // 正常工作
        case <-cannel:
            // 退出
        }
    }
}
func main() {
    cannel := make(chan bool)
    go worker(cannel)
    time.Sleep(time.Second)
    cannel <- true
}
```

### close

但是管道的发送操作和接收操作是一一对应的，如果要停止多个Goroutine那么可能需要创建同样数量的管道，这个代价太大了。其实我们可以通过`close`关闭一个管道来实现广播的效果，所有从关闭管道接收的操作均会收到一个零值和一个可选的失败标志。

```
func worker(cannel chan bool) {
    for {
        select {
        default:
            fmt.Println("hello")
            // 正常工作
        case <-cannel:
            // 退出
        }
    }
}
func main() {
    cancel := make(chan bool)
    for i := 0; i < 10; i++ {
        go worker(cancel)
    }
    time.Sleep(time.Second)
    close(cancel)
}
```

我们通过`close`来关闭`cancel`管道向多个Goroutine广播退出的指令。不过这个程序依然不够稳健：当每个Goroutine收到退出指令退出时一般会进行一定的清理工作，但是退出的清理工作并不能保证被完成，因为`main`线程并没有等待各个工作Goroutine退出工作完成的机制。我们可以结合`sync.WaitGroup`来改进:

```
func worker(wg *sync.WaitGroup, cannel chan bool) {
    defer wg.Done()
    for {
        select {
        default:
            fmt.Println("hello")
        case <-cannel:
            return
        }
    }
}
func main() {
    cancel := make(chan bool)
    var wg sync.WaitGroup
    for i := 0; i < 10; i++ {
        wg.Add(1)
        go worker(&wg, cancel)
    }
    time.Sleep(time.Second)
    close(cancel)
    wg.Wait()
}
```

现在每个工作者并发体的创建、运行、暂停和退出都是在`main`函数的安全控制之下了。

### context包

在Go1.7发布时，标准库增加了一个`context`包，**用来简化对于处理单个请求的多个Goroutine之间与请求域的数据、超时和退出等操作**，官方有博文对此做了专门介绍。我们可以用`context`包来重新实现前面的线程安全退出或超时的控制:

```go
func worker(ctx context.Context, wg *sync.WaitGroup) error {    
    defer wg.Done()    
    for {        
        select {        
        default:            
        	fmt.Println("hello")        
        case <-ctx.Done():            
        	return ctx.Err()        
    	}    
    }
}
func main() {    
    ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)    
    var wg sync.WaitGroup    
    for i := 0; i < 10; i++ {       
        wg.Add(1)        
        go worker(ctx, &wg)    
    }    
    time.Sleep(time.Second)    
    cancel()    
    wg.Wait()
}
```

当并发体超时或`main`主动停止工作者Goroutine时，每个工作者都可以安全退出。

Go语言是带内存自动回收特性的，因此内存一般不会泄漏。当`main`函数不再使用管道时后台Goroutine有泄漏的风险。我们可以通过`context`包来避免这个问题，下面是改进的素数筛实现：

```
// 返回生成自然数序列的管道: 2, 3, 4, ...
func GenerateNatural(ctx context.Context) chan int {
    ch := make(chan int)
    go func() {
        for i := 2; ; i++ {
            select {
            case <- ctx.Done():
                return
            case ch <- i:
            }
        }
    }()
    return ch
}
// 管道过滤器: 删除能被素数整除的数
func PrimeFilter(ctx context.Context, in <-chan int, prime int) chan int {
    out := make(chan int)
    go func() {
        for {
            if i := <-in; i%prime != 0 {
                select {
                case <- ctx.Done():
                    return
                case out <- i:
                }
            }
        }
    }()
    return out
}
func main() {
    // 通过 Context 控制后台Goroutine状态
    ctx, cancel := context.WithCancel(context.Background())
    ch := GenerateNatural(ctx) // 自然数序列: 2, 3, 4, ...
    for i := 0; i < 100; i++ {
        prime := <-ch // 新出现的素数
        fmt.Printf("%v: %v\n", i+1, prime)
        ch = PrimeFilter(ctx, ch, prime) // 基于新素数构造的过滤器
    }
    cancel()
}
```

当main函数完成工作前，通过调用`cancel()`来通知后台Goroutine退出，这样就避免了Goroutine的泄漏。

并发是一个非常大的主题，我们这里只是展示几个非常基础的并发编程的例子。官方文档也有很多关于并发编程的讨论，国内也有专门讨论Go语言并发编程的书籍。读者可以根据自己的需求查阅相关的文献。

## 协程栈

[参考](https://segmentfault.com/a/1190000019570427)

在1.4版本之前go的协程栈管理使用[分段栈](https://link.segmentfault.com/?enc=zGkSQfB0kyvvTw3f%2FiHL9w%3D%3D.4%2FbUxtYQaekP4cSF0VzsCgXsDkxul0iLlB6r1ZlrQDXcX01ZfwpYc1%2Bxxh8b0pr%2B)机制实现。实现方式：**当检测到函数需要更多栈时，分配一块新栈，旧栈和新栈使用指针连接起来，函数返回就释放**。 这样的机制存在2个问题：

- 多次循环调用同一个函数会出现“hot split”问题，例子：[stacksplit.go](https://link.segmentfault.com/?enc=okmdG59zUtY5Ckw0btU%2B8g%3D%3D.uM3%2FQfL%2Fh9RWnms37wwgFwHSD00x4ab%2FWdqZKtY34I8%2FCMhd9UPwAxJ5WIWmzrU%2BD6qCqhCIsxTCfuO%2FKwwA1wPrmqfjuV9Z13BTYGhxLzuQgr1Er19ZyMReOwNOhifV%2FUY4oDl%2FnJU32%2FoQQxOZgA%3D%3D)
- 每次分配和释放都要额外消耗

为了解决这2个问题，官方使用：**连续栈**。**连续栈的实现方式：当检测到需要更多栈时，分配一块比原来大一倍的栈，把旧栈数据copy到新栈，释放旧栈。**

### 连续栈

> 栈的扩容和缩容代码量很大，所以精简了很大一部分。在看连续栈的源码前我们不妨思考一下下面的问题：

- 扩容和缩容的触发条件是什么？
- 扩容和缩容的大小如何计算出来？
- 扩容和缩容这个过程做了什么？对性能是否有影响？

### [栈扩容](https://link.segmentfault.com/?enc=e0xuM5OUi4cZs5%2FJlgzsNA%3D%3D.Vsv3jx1Vc5UcF0qTe9NX3kzNEtNQjEaMgAmI%2FgoFeBq8r2cKuXMkoAmCg%2BSlhrXXfOL%2BwpW8muLuOlmxbJDEIO%2BOWMRMnqOVDxzT3%2BYCdd0%3D)

```go
func newstack() {
    thisg := getg()
    ......
    gp := thisg.m.curg
    ......
    // Allocate a bigger segment and move the stack.
    oldsize := gp.stack.hi - gp.stack.lo
    newsize := oldsize * 2 // 比原来大一倍
    ......
    // The goroutine must be executing in order to call newstack,
    // so it must be Grunning (or Gscanrunning).
    casgstatus(gp, _Grunning, _Gcopystack) //修改协程状态

    // The concurrent GC will not scan the stack while we are doing the copy since
    // the gp is in a Gcopystack status.
    copystack(gp, newsize, true) //在下面会讲到
    ......
    casgstatus(gp, _Gcopystack, _Grunning)
    gogo(&gp.sched)
}
```

每一个函数执行都要占用栈空间，用于保存变量，参数等。运行在协程里的函数自然是占用运行它的协程栈。但协程的栈是有限的，如果**发现不够用，会调用`stackalloc`分配一块新的栈，大小比原来大一倍**。

### [栈缩容](https://link.segmentfault.com/?enc=GyVIakqU9PsrteMqE9DToQ%3D%3D.CxP1bufEuXdcM7i3k%2Fegol3EU593XROGfr9oRYzIObXb6ExdnUPfLib4nA60umfX5csJxF1fZwbcdgo5gnPsQpwRiWw1daeu%2FVfm2GlfU4I%3D)

```go
func shrinkstack(gp *g) {
    gstatus := readgstatus(gp)
    ......
    oldsize := gp.stack.hi - gp.stack.lo
    newsize := oldsize / 2 // 比原来小1倍
    // Don't shrink the allocation below the minimum-sized stack
    // allocation.
    if newsize < _FixedStack {
        return
    }
    // Compute how much of the stack is currently in use and only
    // shrink the stack if gp is using less than a quarter of its
    // current stack. The currently used stack includes everything
    // down to the SP plus the stack guard space that ensures
    // there's room for nosplit functions.
    avail := gp.stack.hi - gp.stack.lo
    //当已使用的栈占不到总栈的1/4 进行缩容
    if used := gp.stack.hi - gp.sched.sp + _StackLimit; used >= avail/4 {
        return
    }

    copystack(gp, newsize, false) //在下面会讲到
}
```

**栈的缩容主要是发生在GC期间**。一个协程变成常驻状态，繁忙时需要占用很大的内存，但空闲时占用很少，这样会浪费很多内存，为了避免浪费Go在GC时对协程的栈进行了缩容，**缩容也是分配一块新的内存替换原来的，大小只有原来的1/2**。

### 扩容和缩容这个过程做了什么？

```go
func copystack(gp *g, newsize uintptr, sync bool) {
    ......
    old := gp.stack
    ......
    used := old.hi - gp.sched.sp

    // allocate new stack
    new := stackalloc(uint32(newsize))
    ......
    // Compute adjustment.
    var adjinfo adjustinfo
    adjinfo.old = old
    adjinfo.delta = new.hi - old.hi //用于旧栈指针的调整

    //后面有机会和 select / chan 一起分析
    // Adjust sudogs, synchronizing with channel ops if necessary.
    ncopy := used
    if sync {
        adjustsudogs(gp, &adjinfo)
    } else {
        ......
        adjinfo.sghi = findsghi(gp, old)

        // Synchronize with channel ops and copy the part of
        // the stack they may interact with.
        ncopy -= syncadjustsudogs(gp, used, &adjinfo)
    }
    //把旧栈数据复制到新栈
    // Copy the stack (or the rest of it) to the new location
    memmove(unsafe.Pointer(new.hi-ncopy), unsafe.Pointer(old.hi-ncopy), ncopy)

    // Adjust remaining structures that have pointers into stacks.
    // We have to do most of these before we traceback the new
    // stack because gentraceback uses them.
    adjustctxt(gp, &adjinfo)
    adjustdefers(gp, &adjinfo)
    adjustpanics(gp, &adjinfo)
    ......
    // Swap out old stack for new one
    gp.stack = new
    gp.stackguard0 = new.lo + _StackGuard // NOTE: might clobber a preempt request
    gp.sched.sp = new.hi - used
    gp.stktopsp += adjinfo.delta
    // Adjust pointers in the new stack.
    gentraceback(^uintptr(0), ^uintptr(0), 0, gp, 0, nil, 0x7fffffff, adjustframe, noescape(unsafe.Pointer(&adjinfo)), 0)
    ......
    //释放旧栈
    stackfree(old)
}
```

在扩容和缩容这个过程中，做了很多调整。从连续栈的实现方式上我们了解到，**不管是扩容还是缩容，都重新申请一块新栈，然后把旧栈的数据复制到新栈**。协程占用的物理内存完全被替换了，而Go在运行时会把指针保存到内存里面，例如：`gp.sched.ctxt` ，`gp._defer` ，`gp._panic`，包括函数里的指针。这部分指针值会被转换成整数型`uintptr`，然后 `+ delta`进行调整。

```go
func adjustpointer(adjinfo *adjustinfo, vpp unsafe.Pointer) {
    pp := (*uintptr)(vpp)
    p := *pp
    ......
    //如果这个整数型数字在旧栈的范围，就调整
    if adjinfo.old.lo <= p && p < adjinfo.old.hi {
        *pp = p + adjinfo.delta
        ......
    }
}
```

### Frame调整

如果只是想了解栈的扩缩容，上面就够了。这部分深入到细节，没兴趣可以跳过。在了解Frame调整前，先了解下 [Stack Frame](https://link.segmentfault.com/?enc=oGx%2F8G34QHoNz5Vgd%2Fjk7g%3D%3D.v2jzBAGCb%2Bhw72X5pdBm01v5otvvtgQj%2B1Dusc8hsXqqzEUTw3WdhXf8KcdY2VQ7SpTQW9fa%2Fw2lzRxXPHlTmRg53sDgqcy6clLIZvipRHU%3D)。Stack Frame ：函数运行时占用的内存空间，是栈上的数据集合，它包括：

- Local variables
- Saved copies of registers modified by subprograms that could need restoration
- Argument parameters
- Return address

#### `FP`，`SP`，`PC` ，`LR`

- FP: Frame Pointer

  – Points to the bottom of the argument list

- SP: Stack Pointer

  – Points to the top of the space allocated for local variables

- PC: Program Counter

- LR：Caller's Program Counter

#### [Stack frame layout](https://link.segmentfault.com/?enc=OvI8cIf9hjQH1PgBC%2FjrKw%3D%3D.TewNLBjwYaw6RMyqZDj860PElZHD1hZ1dc6WZcGU%2FYyGMDvOFyvqfwhZCNs9vhtfvHh85QSoWrfMImOxS9VJU11b5pWDelxZgJ8sYy0blAA03dte3j1DCc73v7nXOuK6)

```go
// (x86)  
// +------------------+  
// | args from caller |  
// +------------------+ <- frame->argp  
// |  return address  |  
// +------------------+  
// |  caller's BP (*) | (*) if framepointer_enabled && varp < sp  
// +------------------+ <- frame->varp  
// |     locals       |  
// +------------------+  
// |  args to callee  |  
// +------------------+ <- frame->sp
```

在Go里针对X86和ARM的[Stack frame layout](https://link.segmentfault.com/?enc=54LYECseNlJ2AC%2Bg28KQVQ%3D%3D.X%2BcQzSeem2U27E%2By232pDSJ3Vdm0jUIcTb82%2B1RFh24IvUKK7bpLB4Fx98Fj%2BkkUN7PzAMMSl07T6%2FqfdItkfLPYEFT%2B3Ymfke%2BSz7ZHD6LMFvKfjurv6gUg2km5pLVE)会不一样，这里只对X86进行分析。

> 为了直观看到Frame调整的结果，我们看下面的例子：

```go
func bb(a *int, aa *int) {
    var v1 int
    println("v1 before morestack", uintptr(unsafe.Pointer(&v1)))

    cc(0)

    println("a after morestack", uintptr(unsafe.Pointer(a)))
    println("aa after morestack", uintptr(unsafe.Pointer(aa)))
    println("v1 after morestack", uintptr(unsafe.Pointer(&v1)))
}

// for morestack
func cc(i int){
    i++
    if i >= 30 {
        println("morestack done")
    }else{
        cc(i)
    }
}

func main()  {
    wg := sync.WaitGroup{}
    wg.Add(1)
    go func() {
        var a, aa int
        a = 1000
        aa = 1000

        println("a before morestack", uintptr(unsafe.Pointer(&a)))
        println("aa before morestack", uintptr(unsafe.Pointer(&aa)))

        bb(&a, &aa)
        wg.Done()
    }()
    wg.Wait()
}
```

结果：

```go
a before morestack 824633925560
aa before morestack 824633925552
v1 before morestack 824633925504
morestack done
a after morestack 824634142648
aa after morestack 824634142640
v1 after morestack 824634142592
```

从结果看出bb的参数a，aa和变量v1地址在经过扩容后发生了变化，这个变化是怎么实现的呢？我们主要围绕下面3个问题进行分析：

1. 如何确认函数Frame的位置
2. 如何找到函数参数，变量的指针
3. 如何确认父函数的Frame

#### 从[gentraceback](https://link.segmentfault.com/?enc=51GqctJaEwuajiZKQWGGtQ%3D%3D.6Vvf3dJ1FyB5LnylKnjhskVq7FWB4WJL374uiD5yQzA06ZJssv309nsOAhcLzB7QU4EobahREY%2BhyJEMq65LUQqBu%2FRugn8Wz%2B3tMN0XSASIsvdD%2FxC7tsStg8878sCh)开始

```go
func gentraceback(pc0, sp0, lr0 uintptr, gp *g, skip int, pcbuf *uintptr, max int, callback func(*stkframe, unsafe.Pointer) bool, v unsafe.Pointer, flags uint) int {
    ......
    g := getg()
    ......
    if pc0 == ^uintptr(0) && sp0 == ^uintptr(0) { // Signal to fetch saved values from gp.
        if gp.syscallsp != 0 {
            ......
        } else {
            //运行位置
            pc0 = gp.sched.pc
            sp0 = gp.sched.sp
            ......
        }
    }
    nprint := 0
    var frame stkframe
    frame.pc = pc0
    frame.sp = sp0
    ......
    f := findfunc(frame.pc)
    ......
    frame.fn = f

    n := 0
    for n < max {
        ......
        f = frame.fn
        if f.pcsp == 0 {
            // No frame information, must be external function, like race support.
            // See golang.org/issue/13568.
            break
        }
        ......
        if frame.fp == 0 {
            sp := frame.sp
            ......
            //计算FP
            frame.fp = sp + uintptr(funcspdelta(f, frame.pc, &cache))
            if !usesLR {
                // On x86, call instruction pushes return PC before entering new function.
                frame.fp += sys.RegSize
            }
        }
        var flr funcInfo
        if topofstack(f, gp.m != nil && gp == gp.m.g0) {
            ......
        } else if usesLR && f.funcID == funcID_jmpdefer {
            ......
        } else {
            var lrPtr uintptr
            if usesLR {
                ......
            } else {
                if frame.lr == 0 {
                    //获取调用函数的PC值
                    lrPtr = frame.fp - sys.RegSize
                    frame.lr = uintptr(*(*sys.Uintreg)(unsafe.Pointer(lrPtr)))
                }
            }
            flr = findfunc(frame.lr)
            ......
        }

        frame.varp = frame.fp
        if !usesLR {
            // On x86, call instruction pushes return PC before entering new function.
            frame.varp -= sys.RegSize
        }
        ......
        if framepointer_enabled && GOARCH == "amd64" && frame.varp > frame.sp {
            frame.varp -= sys.RegSize
        }
        ......
        if callback != nil || printing {
            frame.argp = frame.fp + sys.MinFrameSize
            ......
        }
        ......
        //当前为调整frame
        if callback != nil {
            if !callback((*stkframe)(noescape(unsafe.Pointer(&frame))), v) {
                return n
            }
        }
        ......
        n++
    skipped:
        ......
    //确认父Frame
        // Unwind to next frame.
        frame.fn = flr
        frame.pc = frame.lr
        frame.lr = 0
        frame.sp = frame.fp
        frame.fp = 0
        frame.argmap = nil
        ......
    }
    ......
    return n
}
```

> [gentraceback](https://link.segmentfault.com/?enc=F74VJuFMNscrXUFs0LqL7Q%3D%3D.Rz6%2FpDlhTp%2FrkZfqJb7py%2FBpPhraA2ta%2F8dhafGUQb7jCq8zQdTnvuxxraY1s26vvecFyr4A0%2Fmv1c8yolR1VrsuMBwiL746QcA%2FXqY6S2FSy0Hf%2B3LJKnHbli%2F36g6H)代码量很大，这里根据Frame调整传的参数和我们将要探索部分进行了精简。精简后还是很长，不用担心，我们一层一层剥开这个函数。

- 确认当前位置

  > 当发生扩缩容时，Go的runtime已经把PC保存到`gp.sched.pc`，SP保存到`gp.sched.sp`。

- 找出函数信息

  > 函数的参数、变量个数，frame size，file line等信息，编译通过后被保存进执行文件，执行时被加载进内存，这部分数据可以通过PC获取出来：[findfunc](https://link.segmentfault.com/?enc=NXrRTTQwXf9RJEfbB6zL5A%3D%3D.BRlSXhk%2F58%2FEZr5oC0HfqnH2jk%2FuFVsCqJyoUYHZaqfh0CVRQ80Tr2L9owRv879DZOpBLRcw0K6dG0%2Fw13NSzyXBodtQ%2FGl3I6%2B3bvu7Tof4crFFkSOTg3o48uwoEhgg) -> [findmoduledatap](https://link.segmentfault.com/?enc=U%2FgcpJCEq95vPYtRElGHJg%3D%3D.2euHB62QU7LBsHp%2BMi%2BxPvRPJLK%2BOvsa2qr8zOlT1RF0UNE%2FHemGFaGEFy2bUoMa25eNrPNvo8oAdi9k%2FiGmRlNCz3J3MzqARDRMRBzz9SAAD5AqH94qCrTw3TFozDXt)

  ```go
  func findmoduledatap(pc uintptr) *moduledata {
         for datap := &firstmoduledata; datap != nil; datap = datap.next {
             if datap.minpc <= pc && pc < datap.maxpc {
                 return datap
             }
         }
         return nil
  }
  ```

- 计算FP

```go
frame.fp = sp + uintptr(funcspdelta(f, frame.pc, &cache))
```

> SP我们可以理解为函数的顶端，FP是函数的底部，有了SP，缺函数长度（frame size）。其实我们可以根据pcsp获取，因为它已经被映射进了内存，详情请看[Go 1.2 Runtime Symbol Information](https://link.segmentfault.com/?enc=s7ZUNsmlzYk86R4W5t1ZxA%3D%3D.ifwM75yEa7dJUiYG02i4AjJoLW%2F2psU3paX%2FkMNfVVgFZc6hcTTjTkSNXJIelD228nDGlwjg1oHrGpqrDeSwKw0X4nuzfJtwfyFl2vNRCURAebF%2FZR%2FidU%2F4bNxNj0y2)。知道了FP和SP，我们就可以知道函数在协程栈的具体位置。

- 获取父函数PC指令(LR)

  ```go
  lrPtr = frame.fp - sys.RegSize
  frame.lr = uintptr(*(*sys.Uintreg)(unsafe.Pointer(lrPtr)))
  ```

  > 父函数的PC指令放在了stack frame图的`return address`位置，我们可以直接拿出来，根据这个指令我们获得父函数的信息。

- 确认父函数Frame

```go
frame.fn = flr
frame.pc = frame.lr
frame.lr = 0
frame.sp = frame.fp
frame.fp = 0
frame.argmap = nil
```

> 从stack frame图可以看到子函数的FP等于父函数SP。知道了父函数的SP和PC，重复上面的步骤就可以找出函数所在整条调用链，我们平时看到panic出现的调用链就是这样出来的。

#### 以[adjustframe](https://link.segmentfault.com/?enc=lvDGbP276%2Fgpo70wGoj5Hw%3D%3D.QHEhjT%2BJjX6Qj1XLJNUHYSrP2GxXa9fKX%2F38OksJSLZpS7YhGKEF1p0FkYTuxSfF%2FXkHJvScLwvK5RL05STL%2Bs41mQpIEis1x%2BtcBWuc%2F6gd%2FGRWapsuEb%2FrAncZeUE%2F)结束

```go
func adjustframe(frame *stkframe, arg unsafe.Pointer) bool {
    adjinfo := (*adjustinfo)(arg)
    ......
    f := frame.fn
    ......
    locals, args := getStackMap(frame, &adjinfo.cache, true)
    // Adjust local variables if stack frame has been allocated.
    if locals.n > 0 {
        size := uintptr(locals.n) * sys.PtrSize
        adjustpointers(unsafe.Pointer(frame.varp-size), &locals, adjinfo, f)
    }

    // Adjust saved base pointer if there is one.
    if sys.ArchFamily == sys.AMD64 && frame.argp-frame.varp == 2*sys.RegSize {
        ......
        adjustpointer(adjinfo, unsafe.Pointer(frame.varp))
    }
    // Adjust arguments.
    if args.n > 0 {
        ......
        adjustpointers(unsafe.Pointer(frame.argp), &args, adjinfo, f)
    }
    return true
}
```

> 通过[gentraceback](https://link.segmentfault.com/?enc=oxaZFrUBr05r7VrM5CELtw%3D%3D.Sb89sacXxirf%2FNUcworO0n%2B%2Be2gNDizea4cdIQS3Ftzo1Dn4PhZUGkx16576E1rylgP9lgn35xg07b2BlfnVGIYRjod%2B4U1tJ8V8TpWP3JxZKxMwQBmVZEDMMQGHtlxK)获取frame在协程栈的准确位置，结合 [Stack frame layout](https://link.segmentfault.com/?enc=uPzYGRTVjvbsKw9AV6XftA%3D%3D.0GN0MJY2Wl%2FVvEH9W%2FSj48um4QcV548VPabiy6BPNkEJz3xeIVqCFIYfg%2BQNPMB17s3mFPykcH0v4ePirWI3%2Fh1Kw9wS3ENWiva2iKwfJIvGhFI%2Bi0Muu8stz9wyxnE%2B)，我们就可以知道函数参数`argp`和变量`varp`地址。在64位系统，每个指针占用8个字节。以8做为步长，就可得出函数参数和变量里的指针并进行调整。

来到这里协程栈的源码分析已经完成，通过上面我们了解到连续栈具体实现方式，收获不少，接下来看看连续栈缺点和收益。

#### 连续栈的缺点

连续栈虽然解决了分段栈的2个问题，但这种实现方式也会带来其他问题：

- **更多的虚拟内存碎片**。尤其是你需要更大的栈时，分配一块连续的内存空间会变得更困难
- **指针会被限制放入栈**。在go里面不允许二个协程的指针相互指向。这会增加实现的复杂性。

#### 收益

这部分数据来自[Contiguous stacks](https://link.segmentfault.com/?enc=%2F3PW0C07acVJxUn6mRv%2B2g%3D%3D.Jq2f4ocI0k27qLwu6Sd9HyLep9km9aQpjiiBPcUd80nx%2FUO3ItHv%2Fxx8uckhBPtVrF3r3PadB4LBZcSP%2FiE%2BnyJsmXOwQCJ0WlUcV7k62EAYpDaXNj%2BfDDVoVBbrzpuC)。

- 栈增长1倍快了10%，增长50%只快了2%，增长25%慢了20%
- `Hot split`性能问题。

```apache
segmented stacks:

no split: 1.25925147s
with split: 5.372118558s   <- 出发了 hot split 问题
both split: 1.293200571s

contiguous stacks:

no split: 1.261624848s
with split: 1.262939769s
both split: 1.29008309s
```

## 相关问题

### 强制 kill goroutine 可能吗

答案是不能**，goroutine 只能自己退出，而不能被其他 goroutine 强制关闭或杀死**。

> goroutine 被设计为不可以从外部无条件地结束掉，只能通过 channel 来与它通信。也就是说，每一个 goroutine 都需要承担自己退出的责任。(A goroutine cannot be programmatically killed. It can only commit a cooperative suicide.)

关于这个问题，Github 上也有讨论：

> [question: is it possible to a goroutine immediately stop another goroutine?](https://github.com/golang/go/issues/32610)

摘抄其中几个比较有意思的观点如下：

- 杀死一个 goroutine 设计上会有很多挑战，当前所拥有的资源如何处理？堆栈如何处理？defer 语句需要执行么？
- 如果允许 defer 语句执行，那么 defer 语句可能阻塞 goroutine 退出，这种情况下怎么办呢？

**当父协程是main协程时，父协程退出，父协程下的所有子协程也会跟着退出；当父协程不是main协程时，父协程退出，父协程下的所有子协程并不会跟着退出（子协程直到自己的所有逻辑执行完或者是main协程结束才结束）**

### 一些建议

因为 goroutine 不能被强制 kill，在超时或其他类似的场景下，为了 goroutine 尽可能正常退出，建议如下：

- **尽量使用非阻塞 I/O**（非阻塞 I/O 常用来实现高性能的网络库），阻塞 I/O 很可能导致 goroutine 在某个调用一直等待，而无法正确结束。
- 业务逻辑总是考虑退出机制，避免死循环。
- 任务分段执行，超时后即时退出，避免 goroutine 无用的执行过多，浪费资源。

### 通道关闭原则

#### channel 的三种状态和三种操作结果

| 操作     | 空值(nil) | 非空已关闭 | 非空未关闭                             |
| :------- | :-------- | :--------- | :------------------------------------- |
| 关闭     | panic     | panic      | 成功关闭                               |
| 发送数据 | 永久阻塞  | panic      | 阻塞（同步通道）或成功发送（缓存通道） |
| 接收数据 | 永久阻塞  | 永不阻塞   | 阻塞（同步通道）或者成功接收           |



> 通道关闭原则
>
> 一个常用的使用Go通道的原则是**不要在数据接收方或者在有多个发送者的情况下关闭通道**。换句话说，我们只应该**让一个通道唯一的发送者关闭此通道**。

在 [如何优雅地关闭通道 - go101](https://gfw.go101.org/article/channel-closing.html) 这篇文章中，作者介绍了常见的几种关闭 channel 的方法：

#### 粗鲁的方式（非常不推荐）

如果 channel 已经被关闭，再次关闭会产生 panic，这时通过 recover 使程序恢复正常。

```
func SafeClose(ch chan T) (justClosed bool) {
	defer func() {
		if recover() != nil {
			// 一个函数的返回结果可以在defer调用中修改。
			justClosed = false
		}
	}()

	// 假设ch != nil。
	close(ch)   // 如果 ch 已关闭，将 panic
	return true // <=> justClosed = true; return
}
```

####  礼貌的方式

使用 sync.Once 或互斥锁(sync.Mutex)确保 channel 只被关闭一次。

```
type MyChannel struct {
	C    chan T
	once sync.Once
}

func NewMyChannel() *MyChannel {
	return &MyChannel{C: make(chan T)}
}

func (mc *MyChannel) SafeClose() {
	mc.once.Do(func() {
		close(mc.C)
	})
}
```

#### 优雅的方式

- 情形一：M个接收者和一个发送者，发送者**通过关闭用来传输数据的通道**来传递发送结束信号。
- 情形二：一个接收者和N个发送者，此唯一接收者**通过关闭一个额外的信号通道来通知发送者不要再发送数据了**。
- 情形三：M个接收者和N个发送者，它们中的任何协程都可以**让一个中间调解协程帮忙发出停止数据传送**的信号。
- 情形四：“M个接收者和一个发送者”情形的一个变种：用来传输数据的通道的关闭请求由第三方发出
- 情形五：“N个发送者”的一个变种：用来传输数据的通道必须被关闭以通知各个接收者数据发送已经结束了

### 什么是Goroutine泄漏以及原因、解决办法

[参考](https://segmentfault.com/a/1190000040161853)

#### Goroutine 泄露

了解清楚大家爱问的原因后，我们开始对 Goroutine 泄露的 N 种方法进行研究，希望通过前人留下的 “坑”，了解其原理和避开这些问题。

泄露的原因大多集中在：

- Goroutine 内正在进行 channel/mutex 等读写操作，但由于逻辑问题，某些情况下会**被一直阻塞**。
- Goroutine 内的业务逻辑进入死循环，资源一直无法释放。
- Goroutine 内的业务逻辑进入长时间等待，有不断新增的 Goroutine 进入等待。

#### channel 使用不当

Goroutine+Channel 是最经典的组合，因此不少泄露都出现于此。

最经典的就是上面提到的 channel 进行读写操作时的逻辑问题。

##### 发送不接收

第一个例子：

```go
func main() {
    for i := 0; i < 4; i++ {
        queryAll()
        fmt.Printf("goroutines: %d\n", runtime.NumGoroutine())
    }
}

func queryAll() int {
    ch := make(chan int)
    for i := 0; i < 3; i++ {
        go func() { ch <- query() }()
        }
    return <-ch
}

func query() int {
    n := rand.Intn(100)
    time.Sleep(time.Duration(n) * time.Millisecond)
    return n
}
```

输出结果：

```apache
goroutines: 3
goroutines: 5
goroutines: 7
goroutines: 9
```

在这个例子中，我们调用了多次 `queryAll` 方法，并在 `for` 循环中利用 Goroutine 调用了 `query` 方法。其重点在于调用 `query` 方法后的结果会写入 `ch` 变量中，接收成功后再返回 `ch` 变量。

最后可看到输出的 goroutines 数量是在不断增加的，每次多 2 个。也就是每调用一次，都会泄露 Goroutine。

原因在于 channel 均已经发送了（每次发送 3 个），但是在接收端并没有接收完全（只返回 1 个 ch），所诱发的 Goroutine 泄露。

##### 接收不发送

第二个例子：

```go
func main() {
    defer func() {
        fmt.Println("goroutines: ", runtime.NumGoroutine())
    }()

    var ch chan struct{}
    go func() {
        ch <- struct{}{}
    }()
    
    time.Sleep(time.Second)
}
```

输出结果：

```apache
goroutines:  2
```

在这个例子中，与 “发送不接收” 两者是相对的，channel 接收了值，但是不发送的话，同样会造成阻塞。

但在实际业务场景中，一般更复杂。基本是一大堆业务逻辑里，有一个 channel 的读写操作出现了问题，自然就阻塞了。

##### nil channel

第三个例子：

```go
func main() {
    defer func() {
        fmt.Println("goroutines: ", runtime.NumGoroutine())
    }()

    var ch chan int
    go func() {
        <-ch
    }()
    
    time.Sleep(time.Second)
}
```

输出结果：

```apache
goroutines:  2
```

在这个例子中，可以得知 channel 如果忘记初始化，那么无论你是读，还是写操作，都会造成阻塞。

正常的初始化姿势是：

```go
    ch := make(chan int)
    go func() {
        <-ch
    }()
    ch <- 0
    time.Sleep(time.Second)
```

调用 `make` 函数进行初始化。

#### 奇怪的慢等待

第四个例子：

```go
func main() {
    for {
        go func() {
            _, err := http.Get("https://www.xxx.com/")
            if err != nil {
                fmt.Printf("http.Get err: %v\n", err)
            }
            // do something...
    }()

    time.Sleep(time.Second * 1)
    fmt.Println("goroutines: ", runtime.NumGoroutine())
    }
}
```

输出结果：

```avrasm
goroutines:  5
goroutines:  9
goroutines:  13
goroutines:  17
goroutines:  21
goroutines:  25
...
```

在这个例子中，展示了一个 Go 语言中经典的事故场景。也就是一般我们会在应用程序中去调用第三方服务的接口。

**但是第三方接口，有时候会很慢，久久不返回响应结果**。恰好，Go 语言中默认的 `http.Client` 是没有设置超时时间的。

因此就会导致一直阻塞，一直阻塞就一直爽，Goroutine 自然也就持续暴涨，不断泄露，最终占满资源，导致事故。

在 Go 工程中，我们一般建议至少对 `http.Client` 设置超时时间：

```go
    httpClient := http.Client{
        Timeout: time.Second * 15,
    }
```

并且要做限流、熔断等措施，以防突发流量造成依赖崩塌，依然吃 P0。

#### 互斥锁忘记解锁

第五个例子：

```go
func main() {
    total := 0
    defer func() {
        time.Sleep(time.Second)
        fmt.Println("total: ", total)
        fmt.Println("goroutines: ", runtime.NumGoroutine())
    }()

    var mutex sync.Mutex
    for i := 0; i < 10; i++ {
        go func() {
            mutex.Lock()
            total += 1
        }()
    }
}
```

输出结果：

```apache
total:  1
goroutines:  10
```

在这个例子中，第一个互斥锁 `sync.Mutex` 加锁了，但是他可能在处理业务逻辑，又或是忘记 `Unlock` 了。

因此导致后面的所有 `sync.Mutex` 想加锁，却因未释放又都阻塞住了。一般在 Go 工程中，我们建议如下写法：

```go
    var mutex sync.Mutex
    for i := 0; i < 10; i++ {
        go func() {
            mutex.Lock()
            defer mutex.Unlock()
            total += 1
    }()
    }
```

#### 同步锁使用不当

第六个例子：

```go
func handle(v int) {
    var wg sync.WaitGroup
    wg.Add(5)
    for i := 0; i < v; i++ {
        fmt.Println("脑子进煎鱼了")
        wg.Done()
    }
    wg.Wait()
}

func main() {
    defer func() {
        fmt.Println("goroutines: ", runtime.NumGoroutine())
    }()

    go handle(3)
    time.Sleep(time.Second)
}
```

在这个例子中，我们调用了同步编排 `sync.WaitGroup`，模拟了一遍我们会从外部传入循环遍历的控制变量。

但由于 `wg.Add` 的数量与 `wg.Done` 数量并不匹配，因此在调用 `wg.Wait` 方法后一直阻塞等待。

在 Go 工程中使用，我们会建议如下写法：

```go
    var wg sync.WaitGroup
    for i := 0; i < v; i++ {
        wg.Add(1)
        defer wg.Done()
        fmt.Println("脑子进煎鱼了")
    }
    wg.Wait()
```

#### 排查方法

我们可以调用 `runtime.NumGoroutine` 方法来获取 Goroutine 的运行数量，进行前后一比较，就能知道有没有泄露了。

但在业务服务的运行场景中，Goroutine 内导致的泄露，大多数处于生产、测试环境，因此更多的是使用 PProf：

```go
import (
    "net/http"
     _ "net/http/pprof"
)

http.ListenAndServe("localhost:6060", nil))
```

只要我们调用 `http://localhost:6060/debug/pprof/goroutine?debug=1`，PProf 会返回所有带有堆栈跟踪的 Goroutine 列表。

也可以利用 PProf 的其他特性进行综合查看和分析，这块参考我之前写的《Go 大杀器之性能剖析 PProf》，基本是全村最全的教程了。

#### 解决办法

[参考](http://xueyuan.coder55.com/read/go-senior-learn/go-question-14.13)

可以通过 context 包来避免内存泄漏。

### Channel 可能会引发 goroutine 泄漏

**泄漏的原因是 goroutine 操作 channel 后，处于发送或接收阻塞状态，而 channel 处于满或空的状态，一直得不到改变。同时，垃圾回收器也不会回收此类资源，进而导致 gouroutine 会一直处于等待队列中，不见天日**。

另外，程序运行过程中，对于一个 channel，如果没有任何 goroutine 引用了，gc 会对其进行回收操作，不会引起内存泄漏。

### 会诱发 Goroutine 挂起的 27 个原因

[参考](https://developer.51cto.com/article/681462.html)

#### 第一部分

| 标识                      | 含义              |
| :------------------------ | :---------------- |
| waitReasonZero            | 无                |
| waitReasonGCAssistMarking | GC assist marking |
| waitReasonIOWait          | IO wait           |

- waitReasonZero：无正式解释，从使用情况来看。主要在 sleep 和 lock 的 2 个场景中使用。
- waitReasonGCAssistMarking：GC 辅助标记阶段会使得阻塞等待。
- waitReasonIOWait：IO 阻塞等待时，例如：网络请求等。



#### 第二部分

| 标识                         | 含义                    |
| :--------------------------- | :---------------------- |
| waitReasonChanReceiveNilChan | chan receive (nil chan) |
| waitReasonChanSendNilChan    | chan send (nil chan)    |

- waitReasonChanReceiveNilChan：对未初始化的 channel 进行读操作。
- waitReasonChanSendNilChan：对未初始化的 channel 进行写操作。

#### 第三部分

| 标识                            | 含义                    |
| :------------------------------ | :---------------------- |
| waitReasonDumpingHeap           | dumping heap            |
| waitReasonGarbageCollection     | garbage collection      |
| waitReasonGarbageCollectionScan | garbage collection scan |

- waitReasonDumpingHeap：对 Go Heap 堆 dump 时，这个的使用场景仅在 runtime.debug 时，也就是常见的 pprof 这一类采集时阻塞。
- waitReasonGarbageCollection：在垃圾回收时，主要场景是 GC 标记终止(GC Mark Termination)阶段时触发。
- waitReasonGarbageCollectionScan：在垃圾回收扫描时，主要场景是 GC 标记(GC Mark)扫描 Root 阶段时触发。

#### 第四部分

| 标识                    | 含义              |
| :---------------------- | :---------------- |
| waitReasonPanicWait     | panicwait         |
| waitReasonSelect        | select            |
| waitReasonSelectNoCases | select (no cases) |

- waitReasonPanicWait：在 main goroutine 发生 panic 时，会触发。
- waitReasonSelect：在调用关键字 select 时会触发。
- waitReasonSelectNoCases：在调用关键字 select 时，若一个 case 都没有，会直接触发。

#### 第五部分

| 标识                     | 含义             |
| :----------------------- | :--------------- |
| waitReasonGCAssistWait   | GC assist wait   |
| waitReasonGCSweepWait    | GC sweep wait    |
| waitReasonGCScavengeWait | GC scavenge wait |

- waitReasonGCAssistWait：GC 辅助标记阶段中的结束行为，会触发。
- waitReasonGCSweepWait：GC 清扫阶段中的结束行为，会触发。
- waitReasonGCScavengeWait：GC scavenge 阶段的结束行为，会触发。GC Scavenge 主要是新空间的垃圾回收，是一种经常运行、快速的 GC，负责从新空间中清理较小的对象。

#### 第六部分

| 标识                    | 含义           |
| :---------------------- | :------------- |
| waitReasonChanReceive   | chan receive   |
| waitReasonChanSend      | chan send      |
| waitReasonFinalizerWait | finalizer wait |

- waitReasonChanReceive：在 channel 进行读操作，会触发。
- waitReasonChanSend：在 channel 进行写操作，会触发。
- waitReasonFinalizerWait：在 finalizer 结束的阶段，会触发。在 Go 程序中，可以通过调用 runtime.SetFinalizer 函数来为一个对象设置一个终结者函数。这个行为对应着结束阶段造成的回收。

#### 第七部分

| 标识                  | 含义            |
| :-------------------- | :-------------- |
| waitReasonForceGCIdle | force gc (idle) |
| waitReasonSemacquire  | semacquire      |
| waitReasonSleep       | sleep           |

- waitReasonForceGCIdle：强制 GC(空闲时间)结束时，会触发。
- waitReasonSemacquire：信号量处理结束时，会触发。
- waitReasonSleep：经典的 sleep 行为，会触发。

#### 第八部分

| 标识                         | 含义                   |
| :--------------------------- | :--------------------- |
| waitReasonSyncCondWait       | sync.Cond.Wait         |
| waitReasonTimerGoroutineIdle | timer goroutine (idle) |
| waitReasonTraceReaderBlocked | trace reader (blocked) |

- waitReasonSyncCondWait：结合 sync.Cond 用法能知道，是在调用 sync.Wait 方法时所触发。
- waitReasonTimerGoroutineIdle：与 Timer 相关，在没有定时器需要执行任务时，会触发。
- waitReasonTraceReaderBlocked：与 Trace 相关，ReadTrace会返回二进制跟踪数据，将会阻塞直到数据可用。

#### 第九部分

| 标识                     | 含义              |
| :----------------------- | :---------------- |
| waitReasonWaitForGCCycle | wait for GC cycle |
| waitReasonGCWorkerIdle   | GC worker (idle)  |
| waitReasonPreempted      | preempted         |
| waitReasonDebugCall      | debug call        |

- waitReasonWaitForGCCycle：等待 GC 周期，会休眠造成阻塞。
- waitReasonGCWorkerIdle：GC Worker 空闲时，会休眠造成阻塞。
- waitReasonPreempted：发生循环调用抢占时，会会休眠等待调度。
- waitReasonDebugCall：调用 GODEBUG 时，会触发。

#### 总结

今天这篇文章是对开头 runtime.gopark 函数的详解文章的一个补充，我们能够对此了解到其诱发的因素。

主要场景为：

- 通道(Channel)。
- 垃圾回收(GC)。
- 休眠(Sleep)。
- 锁等待(Lock)。
- 抢占(Preempted)。
- IO 阻塞(IO Wait)
- 其他，例如：panic、finalizer、select 等。

我们可以根据这些特性，去拆解可能会造成阻塞的原因。其实也就没必要记了，他们会导致阻塞肯定是由于存在影响控制流的因素，才会导致 gopark 的调用。

### 跳出 for select循环

[参考](https://blog.csdn.net/bravezhe/article/details/81674591)

使用break lable 和 goto lable 都能跳出for循环；不同之处在于：break标签只能用于for循环，且标签位于for循环前面，goto是指跳转到指定标签处。



