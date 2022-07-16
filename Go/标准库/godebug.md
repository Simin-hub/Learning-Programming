# GODEBUG

## 前置知识

Go scheduler 的主要功能是**针对在处理器上运行的 OS 线程分发可运行的 Goroutine**，而我们一提到调度器，就离不开三个经常被提到的缩写，分别是：

- G：Goroutine，实际上我们每次调用 `go func` 就是生成了一个 G。
- P：处理器，一般为处理器的核数，可以通过 `GOMAXPROCS` 进行修改。
- M：OS 线程

这三者交互实际来源于 Go 的 M: N 调度模型，也就是 M 必须与 P 进行绑定，然后不断地在 M 上循环寻找可运行的 G 来执行相应的任务，如果想具体了解可以详细阅读 [《Go Runtime Scheduler》](https://link.segmentfault.com/?enc=1AUtNwx4eKgAMJTO05CsTw%3D%3D.yFCoCVIMvy%2FZhHtX%2F4bMg40I8iOiIxGoUxL3cQ5q6mOdGNjGhimKI659TcykTX7n24nJW1gT0VawF7kedGHdtw%3D%3D)，我们抽其中的工作流程图进行简单分析，如下:

![image](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/1460000020108091)

1. 当我们执行 `go func()` 时，实际上就是创建一个全新的 Goroutine，我们称它为 G。
2. 新创建的 G 会被放入 P 的本地队列（Local Queue）或全局队列（Global Queue）中，准备下一步的动作。
3. 唤醒或创建 M 以便执行 G。
4. 不断地进行事件循环
5. 寻找在可用状态下的 G 进行执行任务
6. 清除后，重新进入事件循环

而在描述中有提到全局和本地这两类队列，其实在功能上来讲都是用于存放正在等待运行的 G，但是不同点在于，本地队列有数量限制，不允许超过 256 个。并且在新建 G 时，会优先选择 P 的本地队列，如果本地队列满了，则将 P 的本地队列的一半的 G 移动到全局队列，这其实可以理解为调度资源的共享和再平衡。

另外我们可以看到图上有 steal 行为，这是用来做什么的呢，我们都知道当你创建新的 G 或者 G 变成可运行状态时，它会被推送加入到当前 P 的本地队列中。但其实当 P 执行 G 完毕后，它也会 “干活”，它会将其从本地队列中弹出 G，同时会检查当前本地队列是否为空，如果为空会随机的从其他 P 的本地队列中尝试窃取一半可运行的 G 到自己的名下。例子如下：

![image](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/1460000020108092)

在这个例子中，P2 在本地队列中找不到可以运行的 G，它会执行 `work-stealing` 调度算法，随机选择其它的处理器 P1，并从 P1 的本地队列中窃取了三个 G 到它自己的本地队列中去。至此，P1、P2 都拥有了可运行的 G，P1 多余的 G 也不会被浪费，调度资源将会更加平均的在多个处理器中流转。

## GODEBUG 调度跟踪

### GODEBUG

GODEBUG 变量可以控制运行时内的调试变量，参数以逗号分隔，格式为：`name=val`。本文着重点在调度器观察上，将会使用如下两个参数：

- schedtrace：设置 `schedtrace=X` 参数可以使运行时在每 X 毫秒发出一行调度器的摘要信息到标准 err 输出中。
- scheddetail：设置 `schedtrace=X` 和 `scheddetail=1` 可以使运行时在每 X 毫秒发出一次详细的多行信息，信息内容主要包括调度程序、处理器、OS 线程 和 Goroutine 的状态。

#### 演示代码

```css
func main() {
    wg := sync.WaitGroup{}
    wg.Add(10)
    for i := 0; i < 10; i++ {
        go func(wg *sync.WaitGroup) {
            var counter int
            for i := 0; i < 1e10; i++ {
                counter++
            }
            wg.Done()
        }(&wg)
    }

    wg.Wait()
}
```

#### schedtrace

```routeros
$ GODEBUG=schedtrace=1000 ./awesomeProject 
SCHED 0ms: gomaxprocs=4 idleprocs=1 threads=5 spinningthreads=1 idlethreads=0 runqueue=0 [0 0 0 0]
SCHED 1000ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=0 [1 2 2 1]
SCHED 2000ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=0 [1 2 2 1]
SCHED 3001ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=0 [1 2 2 1]
SCHED 4010ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=0 [1 2 2 1]
SCHED 5011ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=0 [1 2 2 1]
SCHED 6012ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=0 [1 2 2 1]
SCHED 7021ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=4 [0 1 1 0]
SCHED 8023ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=4 [0 1 1 0]
SCHED 9031ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=4 [0 1 1 0]
SCHED 10033ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=4 [0 1 1 0]
SCHED 11038ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=4 [0 1 1 0]
SCHED 12044ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=4 [0 1 1 0]
SCHED 13051ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=4 [0 1 1 0]
SCHED 14052ms: gomaxprocs=4 idleprocs=2 threads=5 
...
```

- sched：每一行都代表调度器的调试信息，后面提示的毫秒数表示启动到现在的运行时间，输出的时间间隔受 `schedtrace` 的值影响。
- gomaxprocs：当前的 CPU 核心数（GOMAXPROCS 的当前值）。
- idleprocs：空闲的处理器数量，后面的数字表示当前的空闲数量。
- threads：OS 线程数量，后面的数字表示当前正在运行的线程数量。
- spinningthreads：自旋状态的 OS 线程数量。
- idlethreads：空闲的线程数量。
- runqueue：全局队列中中的 Goroutine 数量，而后面的 [0 0 1 1] 则分别代表这 4 个 P 的本地队列正在运行的 Goroutine 数量。

在上面我们有提到 “自旋线程” 这个概念，如果你之前没有了解过相关概念，一听 “自旋” 肯定会比较懵，我们引用 《Head First of Golang Scheduler》 的内容来说明：

> 自旋线程的这个说法，是因为 Go Scheduler 的设计者在考虑了 “OS 的资源利用率” 以及 “频繁的线程抢占给 OS 带来的负载” 之后，提出了 “Spinning Thread” 的概念。也就是当 “自旋线程” 没有找到可供其调度执行的 Goroutine 时，并不会销毁该线程 ，而是采取 “自旋” 的操作保存了下来。虽然看起来这是浪费了一些资源，但是考虑一下 syscall 的情景就可以知道，比起 “自旋"，线程间频繁的抢占以及频繁的创建和销毁操作可能带来的危害会更大。

#### scheddetail

如果我们想要更详细的看到调度器的完整信息时，我们可以增加 `scheddetail` 参数，就能够更进一步的查看调度的细节逻辑，如下：

```routeros
$ GODEBUG=scheddetail=1,schedtrace=1000 ./awesomeProject
SCHED 1000ms: gomaxprocs=4 idleprocs=0 threads=5 spinningthreads=0 idlethreads=0 runqueue=0 gcwaiting=0 nmidlelocked=0 stopwait=0 sysmonwait=0
  P0: status=1 schedtick=2 syscalltick=0 m=3 runqsize=3 gfreecnt=0
  P1: status=1 schedtick=2 syscalltick=0 m=4 runqsize=1 gfreecnt=0
  P2: status=1 schedtick=2 syscalltick=0 m=0 runqsize=1 gfreecnt=0
  P3: status=1 schedtick=1 syscalltick=0 m=2 runqsize=1 gfreecnt=0
  M4: p=1 curg=18 mallocing=0 throwing=0 preemptoff= locks=0 dying=0 spinning=false blocked=false lockedg=-1
  M3: p=0 curg=22 mallocing=0 throwing=0 preemptoff= locks=0 dying=0 spinning=false blocked=false lockedg=-1
  M2: p=3 curg=24 mallocing=0 throwing=0 preemptoff= locks=0 dying=0 spinning=false blocked=false lockedg=-1
  M1: p=-1 curg=-1 mallocing=0 throwing=0 preemptoff= locks=1 dying=0 spinning=false blocked=false lockedg=-1
  M0: p=2 curg=26 mallocing=0 throwing=0 preemptoff= locks=0 dying=0 spinning=false blocked=false lockedg=-1
  G1: status=4(semacquire) m=-1 lockedm=-1
  G2: status=4(force gc (idle)) m=-1 lockedm=-1
  G3: status=4(GC sweep wait) m=-1 lockedm=-1
  G17: status=1() m=-1 lockedm=-1
  G18: status=2() m=4 lockedm=-1
  G19: status=1() m=-1 lockedm=-1
  G20: status=1() m=-1 lockedm=-1
  G21: status=1() m=-1 lockedm=-1
  G22: status=2() m=3 lockedm=-1
  G23: status=1() m=-1 lockedm=-1
  G24: status=2() m=2 lockedm=-1
  G25: status=1() m=-1 lockedm=-1
  G26: status=2() m=0 lockedm=-1
```

在这里我们抽取了 1000ms 时的调试信息来查看，信息量比较大，我们先从每一个字段开始了解。如下：

#### G

- status：G 的运行状态。
- m：隶属哪一个 M。
- lockedm：是否有锁定 M。

在第一点中我们有提到 G 的运行状态，这对于分析内部流转非常的有用，共涉及如下 9 种状态：

| 状态              | 值   | 含义                                                         |
| ----------------- | ---- | ------------------------------------------------------------ |
| _Gidle            | 0    | 刚刚被分配，还没有进行初始化。                               |
| _Grunnable        | 1    | 已经在运行队列中，还没有执行用户代码。                       |
| _Grunning         | 2    | 不在运行队列里中，已经可以执行用户代码，此时已经分配了 M 和 P。 |
| _Gsyscall         | 3    | 正在执行系统调用，此时分配了 M。                             |
| _Gwaiting         | 4    | 在运行时被阻止，没有执行用户代码，也不在运行队列中，此时它正在某处阻塞等待中。 |
| _Gmoribund_unused | 5    | 尚未使用，但是在 gdb 中进行了硬编码。                        |
| _Gdead            | 6    | 尚未使用，这个状态可能是刚退出或是刚被初始化，此时它并没有执行用户代码，有可能有也有可能没有分配堆栈。 |
| _Genqueue_unused  | 7    | 尚未使用。                                                   |
| _Gcopystack       | 8    | 正在复制堆栈，并没有执行用户代码，也不在运行队列中。         |

在理解了各类的状态的意思后，我们结合上述案例看看，如下：

```routeros
G1: status=4(semacquire) m=-1 lockedm=-1
G2: status=4(force gc (idle)) m=-1 lockedm=-1
G3: status=4(GC sweep wait) m=-1 lockedm=-1
G17: status=1() m=-1 lockedm=-1
G18: status=2() m=4 lockedm=-1
```

在这个片段中，G1 的运行状态为 `_Gwaiting`，并没有分配 M 和锁定。这时候你可能好奇在片段中括号里的是什么东西呢，其实是因为该 `status=4` 是表示 `Goroutine` 在**运行时时被阻止**，而阻止它的事件就是 `semacquire` 事件，是因为 `semacquire` 会检查信号量的情况，在合适的时机就调用 `goparkunlock` 函数，把当前 `Goroutine` 放进等待队列，并把它设为 `_Gwaiting` 状态。

那么在实际运行中还有什么原因会导致这种现象呢，我们一起看看，如下：

```awk
    waitReasonZero                                    // ""
    waitReasonGCAssistMarking                         // "GC assist marking"
    waitReasonIOWait                                  // "IO wait"
    waitReasonChanReceiveNilChan                      // "chan receive (nil chan)"
    waitReasonChanSendNilChan                         // "chan send (nil chan)"
    waitReasonDumpingHeap                             // "dumping heap"
    waitReasonGarbageCollection                       // "garbage collection"
    waitReasonGarbageCollectionScan                   // "garbage collection scan"
    waitReasonPanicWait                               // "panicwait"
    waitReasonSelect                                  // "select"
    waitReasonSelectNoCases                           // "select (no cases)"
    waitReasonGCAssistWait                            // "GC assist wait"
    waitReasonGCSweepWait                             // "GC sweep wait"
    waitReasonChanReceive                             // "chan receive"
    waitReasonChanSend                                // "chan send"
    waitReasonFinalizerWait                           // "finalizer wait"
    waitReasonForceGGIdle                             // "force gc (idle)"
    waitReasonSemacquire                              // "semacquire"
    waitReasonSleep                                   // "sleep"
    waitReasonSyncCondWait                            // "sync.Cond.Wait"
    waitReasonTimerGoroutineIdle                      // "timer goroutine (idle)"
    waitReasonTraceReaderBlocked                      // "trace reader (blocked)"
    waitReasonWaitForGCCycle                          // "wait for GC cycle"
    waitReasonGCWorkerIdle                            // "GC worker (idle)"
```

我们通过以上 `waitReason` 可以了解到 `Goroutine` 会被暂停运行的原因要素，也就是会出现在括号中的事件。

#### M

- p：隶属哪一个 P。
- curg：当前正在使用哪个 G。
- runqsize：运行队列中的 G 数量。
- gfreecnt：可用的G（状态为 Gdead）。
- mallocing：是否正在分配内存。
- throwing：是否抛出异常。
- preemptoff：不等于空字符串的话，保持 curg 在这个 m 上运行。

#### P

- status：P 的运行状态。
- schedtick：P 的调度次数。
- syscalltick：P 的系统调用次数。
- m：隶属哪一个 M。
- runqsize：运行队列中的 G 数量。
- gfreecnt：可用的G（状态为 Gdead）。

| 状态      | 值   | 含义                                                         |
| --------- | ---- | ------------------------------------------------------------ |
| _Pidle    | 0    | 刚刚被分配，还没有进行进行初始化。                           |
| _Prunning | 1    | 当 M 与 P 绑定调用 acquirep 时，P 的状态会改变为 _Prunning。 |
| _Psyscall | 2    | 正在执行系统调用。                                           |
| _Pgcstop  | 3    | 暂停运行，此时系统正在进行 GC，直至 GC 结束后才会转变到下一个状态阶段。 |
| _Pdead    | 4    | 废弃，不再使用。                                             |

### 总结

通过本文我们学习到了调度的一些基础知识，再通过神奇的 GODEBUG 掌握了观察调度器的方式方法，你想想，是不是可以和我上一篇文章的 `go tool trace` 来结合使用呢，在实际的使用中，类似的办法有很多，组合巧用是重点。

## GODEBUG for GC

### 6.5.1 GC 的基础知识

#### 6.5.1 什么是 GC

在计算机科学中，垃圾回收（GC）是一种自动管理内存的机制，垃圾回收器会去尝试回收程序不再使用的对象及其占用的内存。而最早 John McCarthy 在 1959 年左右发明了垃圾回收，以简化 Lisp 中的手动内存管理的机制。

#### 6.5.2 为什么要 GC

手动管理内存挺麻烦，管错或者管漏内存也很糟糕，将会直接导致程序不稳定（持续泄露）甚至直接崩溃。

6.5.3 GC 带来的问题

硬要说会带来什么问题的话，也就数大家最关注的 Stop The World（STW），STW 代指在执行某个垃圾回收算法的某个阶段时，需要将整个应用程序暂停去处理 GC 相关的工作事项。例如：

| 行为       | 会不会 STW | 为什么                                                       |
| ---------- | ---------- | ------------------------------------------------------------ |
| 标记开始   | 会         | 在准备开始标记时，需要对根对象进行扫描，此时会打开写屏障（Write Barrier） 和 辅助 GC（mutator assist），为标记做准备工作。 |
| 并发标记中 | 不会       | 标记阶段，主要目的是标记堆内存中仍在使用的值。               |
| 标记结束   | 会         | 在完成标记任务后，将重新扫描部分根对象，这时候会禁用写屏障（Write Barrier）和辅助 GC（mutator assist），而标记阶段和应用程序是并发运行的，所以在标记阶段可能会有新的对象产生，因此在重新扫描时需要进行 STW。 |

#### 6.5.4 Go 目前的 GC 情况

虽然 GC 会带来 STW，但是 Go 语言经过多个版本的优化和调整，目前 STW 时间已经在绝大部分场景下缩减为毫秒级，并且在每个新发布的 Go 版本中也持续不断在进行优化，在常规情况下已经不需要太过多的担忧。

同时在 Go1.5 起 Go Runtime 就已经从 C 和少量汇编，改为由 Go 和少量汇编实现，也就是基本实现 Go 自举，因此你感兴趣的话也可以去看看这块相关的运行时源码，由于本节不是重点介绍 GC，因此并不会具体的铺开来讲。

#### 6.5.5 如何调整 Go GC 频率

可以通过设置 GOGC 变量来调整初始垃圾收集器的目标百分比值，其对比的规则为当新分配的数值与上一次收集后剩余的实时数值的比例达到设置的目标百分比时，就会触发 GC。而 GOGC 的默认值为 GOGC=100，如果将其设置为 GOGC=off 可以完全禁用垃圾回收器。

GOGC 值的大小，与 GC 行为之间又有什么关系呢，简单来讲，GOGC 的值设置的越大，GC 的频率越低，但每次 GC 所触发到的堆内存也会更大。

而在程序运行时，我们可以通过调用下述方法来动态调整 GOGC 的值：

```go
// runtime/debug
debug.SetGCPercent
```

#### 6.5.6 主动触发 GC

在 Go 语言中，与 GC 相关的 API 极少，如果我们想要主动的触发 GC 行为，我们可以通过调用 `runtime.GC` 方法来达到这个目的。而相配套的内存相关的归还，我们也可以通过手动调用 `debug.FreeOSMemory` 方法来触发将内存归还给操作系统的行为。

### 6.5.2 GODEBUG

在 Go 语言中，GODEBUG 变量可以控制运行时内的调试变量，参数以逗号分隔，格式为：`name=val`。

本文着重点在 GODEBUG 的 GC 的观察上，因此主要涉及 gctrace 参数，我们可以通过设置 `gctrace=1` 后使得垃圾收集器向标准错误流发出 GC 运行信息，以此来观察程序的 GC 运行情况。

#### 6.5.2.1 示例代码

创建一个 main.go 文件，写入示例代码，如下：

```go
func main() {
    wg := sync.WaitGroup{}
    wg.Add(10)
    for i := 0; i < 10; i++ {
        go func(wg *sync.WaitGroup) {
            var counter int
            for i := 0; i < 1e10; i++ {
                counter++
            }
            wg.Done()
        }(&wg)
    }

    wg.Wait()
}
```

#### 6.5.2.2 gctrace

接下来我们在命令行设置 `GODEBUG=gctrace=1` 来运行 main.go 文件，为了便于展示和说明，我把 gctrace 的输出结果拆为了两个部分，分别是 GC 和 Scavenging 相关的调试信息。

6.5.2.2.1 垃圾回收（GC）信息

```shell
$ GODEBUG=gctrace=1 go run main.go    
gc 1 @0.032s 0%: 0.019+0.45+0.003 ms clock, 0.076+0.22/0.40/0.80+0.012 ms cpu, 4->4->0 MB, 5 MB goal, 4 P
gc 2 @0.046s 0%: 0.004+0.40+0.008 ms clock, 0.017+0.32/0.25/0.81+0.034 ms cpu, 4->4->0 MB, 5 MB goal, 4 P
gc 3 @0.063s 0%: 0.004+0.40+0.008 ms clock, 0.018+0.056/0.32/0.64+0.033 ms cpu, 4->4->0 MB, 5 MB goal, 4 P
gc 4 @0.080s 0%: 0.004+0.45+0.016 ms clock, 0.018+0.15/0.34/0.77+0.065 ms cpu, 4->4->1 MB, 5 MB goal, 4 P
...
```

我们看到其输出内容在关键字上都是类似的，其模板内容如下：

```shell
gc # @#s #%: #+#+# ms clock, #+#/#/#+# ms cpu, #->#-># MB, # MB goal, # P
```

- `gc#`：GC 执行次数的编号，每次叠加。
- `@#s`：自程序启动后到当前的具体秒数。
- `#%`：自程序启动以来在 GC 中花费的时间百分比。
- `#+...+#`：GC 的标记工作共使用的 CPU 时间占总 CPU 时间的百分比。
- `#->#-># MB`：分别表示 GC 启动时, GC 结束时, GC 活动时的堆大小.
- `#MB goal`：下一次触发 GC 的内存占用阈值。
- `#P`：当前使用的处理器 P 的数量。

#### 6.5.2.2.2 清除（Scavenging）信息

```shell
$ GODEBUG=gctrace=1 go run main.go    
...
scvg: 0 MB released
scvg: inuse: 3, idle: 60, sys: 63, released: 59, consumed: 4 (MB)
scvg: inuse: 1, idle: 61, sys: 63, released: 59, consumed: 4 (MB)
scvg: inuse: 2, idle: 61, sys: 63, released: 59, consumed: 3 (MB)
scvg: inuse: 2, idle: 61, sys: 63, released: 59, consumed: 3 (MB)
```

你可以看到上述输出内容一共分为两个部分，一个是摘要信息，另外一个是清除的调试信息。首先我们看到摘要信息的模板内容如下，如下：

```shell
scvg#: # MB released
```

该输出语句为 GC 将内存回收并释放到系统时所发出摘要信息，因此是在每一次 GC 释放都会输出该类信息。接下来我们看看清除信息的模板内容，如下：

```shell
scvg#: inuse: # idle: # sys: # released: # consumed: # (MB)
```

- `scvg# `：Scavenging 执行次数的编号，每次在清除时递增。
- `inuse: #`：正在占用的内存大小，单位均为 MB。
- `idle: # `：等待被清理的内存大小。
- `sys: #`：从系统映射的内存大小。
- `released: #`：已经释放的系统内存大小。
- `consumed: #`：已经从系统所申请分配的内存大小。

### 6.5.3 案例

我们抽取其中一个实际输出的 GC 调试信息来进行分析和说明（与案例的输出数值不一样，是正常的），如下：

```shell
gc 7 @0.140s 1%: 0.031+2.0+0.042 ms clock, 0.12+0.43/1.8/0.049+0.17 ms cpu, 4->4->1 MB, 5 MB goal, 4 P
```

- gc 7：第 7 次 GC。
- @0.140s：当前是程序启动后的 0.140s。
- 1%：程序启动后到现在共花费 1% 的时间在 GC 上。
- 0.031+2.0+0.042 ms clock：
  - 0.031：表示单个 P 在 mark 阶段的 STW 时间。
  - 2.0：表示所有 P 的 mark concurrent（并发标记）所使用的时间。
  - 0.042：表示单个 P 的 markTermination 阶段的 STW 时间。
- 0.12+0.43/1.8/0.049+0.17 ms cpu：
  - 0.12：表示整个进程在 mark 阶段 STW 停顿的时间。
  - 0.43/1.8/0.049：0.43 表示 mutator assist 占用的时间，1.8 表示 dedicated + fractional 占用的时间，0.049 表示 idle 占用的时间。
  - 0.17ms：0.17 表示整个进程在 markTermination 阶段 STW 时间。
- 4->4->1 MB：
  - 4：表示开始 mark 阶段前的 heap_live 大小。
  - 4：表示开始 markTermination 阶段前的 heap_live 大小。
  - 1：表示被标记对象的大小。
- 5 MB goal：表示下一次触发 GC 回收的阈值是 5 MB。
- 4 P：本次 GC 一共涉及多少个 P。

### 6.5.4 涉及术语

- mark：标记阶段。
- markTermination：标记结束阶段。
- mutator assist：辅助 GC，是指在 GC 过程中 mutator 线程会并发运行，而 mutator assist 机制会协助 GC 做一部分的工作。
- heap_live：在 Go 的内存管理中，span 是内存页的基本单元，每页大小为 8kb，同时 Go 会根据对象的大小不同而分配不同页数的 span，而 heap_live 就代表着所有 span 的总大小。
- dedicated / fractional / idle：在标记阶段会分为三种不同的 mark worker 模式，分别是 dedicated、fractional 和 idle，它们代表着不同的专注程度，其中 dedicated 模式最专注，是完整的 GC 回收行为，fractional 只会干部分的 GC 行为，idle 最轻松。这里你只需要了解它是不同专注程度的 mark worker 就好了，详细介绍我们可以等后续的文章。

### 6.5.5 小结

通过本章节我们掌握了使用 GODEBUG 查看应用程序垃圾回收（GC）和清除（Scavenging）信息运行情况的方法，只要用这种方法我们就可以观测不同情况下 GC、Scavenging 的情况了，甚至可以做出非常直观的对比图，在排查 GC 问题的时候会非常有帮助，不需要瞎猜。