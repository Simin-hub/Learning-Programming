# trace

[参考](https://golang2.eddycjy.com/posts/ch6/03-trace/)、[参考](https://studygolang.com/articles/12639?fr=sidebar)、[参考](https://burxtx.github.io/2018/05/11/%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BAGo%E8%AF%AD%E8%A8%80%E6%89%A7%E8%A1%8C%E8%B7%9F%E8%B8%AA/)

## 介绍

原理是：**监听 Go 运行时的一些特定的事件**，如：

1. goroutine的创建、开始和结束。
2. 阻塞/解锁goroutine的一些事件（系统调用，channel，锁）
3. 网络I/O相关事件
4. 系统调用
5. 垃圾回收

追踪器会原原本本地收集这些信息，不做任何聚合或者抽样操作。对于负载高的应用来说，就可能会生成一个比较大的文件，该文件后面可以通过 `go tool trace` 命令来进行解析。

在引入执行trace程序之前，已经有了pprof内存和CPU分析器，那么为什么它还会被添加到官方的工具链中呢？虽然CPU分析器做了一件很好的工作，告诉你什么函数占用了最多的CPU时间，但它**并不能帮助你确定是什么阻止了goroutine运行，或者在可用的OS线程上如何调度goroutines**。这正是跟踪器真正起作用的地方。trace[设计文档](https://docs.google.com/document/u/1/d/1FP5apqzBgr7ahCCgFO-yoVhk4YZrNIDNf9RybngBc14/pub)很好地解释了跟踪程序背后的动机以及它是如何被设计和工作的。

有时候单单使用 pprof 还不一定足够完整观查并解决问题，因为在真实的程序中还包含许多的隐藏动作，例如 Goroutine 在执行时会做哪些操作？执行/阻塞了多长时间？在什么时候阻止？在哪里被阻止的？谁又锁/解锁了它们？GC 是怎么影响到 Goroutine 的执行的？这些东西用 pprof 是很难分析出来的，但如果你又想知道上述的答案的话，你可以用本章节的主角 `go tool trace` 来打开新世界的大门。

### trace 和 pprof 的区别

**trace 和 pprof 的区别在于两者关注的维度不同，后者更关注代码栈层面，而 trace 更关注于 latency（延迟）**。

比如说一个请求在客户端观察从发送到完成经过了 5s，做 profile 可能发现这个请求的 CPU 时间只有 2s，那剩下的 3s 就不是很清楚了， profile 更侧重的是我们代码执行了多久，至于其他的，例如：网络 IO，系统调用，Goroutine 调度，GC 时间等，很难反映出来。这是就应该使用 trace 了。

## 1 初步了解

```go
import (
	"os"
	"runtime/trace"
)

func main() {
	trace.Start(os.Stderr)
	defer trace.Stop()

	ch := make(chan string)
	go func() {
		ch <- "Go 语言编程之旅"
	}()

	<-ch
}
```

生成跟踪文件：

```shell
$ go run main.go 2> trace.out
```

启动可视化界面：

```shell
$ go tool trace trace.out
2019/06/22 16:14:52 Parsing trace...
2019/06/22 16:14:52 Splitting trace...
2019/06/22 16:14:52 Opening browser. Trace viewer is listening on http://127.0.0.1:57321
```

查看可视化界面：

![image](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/trace_view.jpg)

- View trace：查看跟踪
- Goroutine analysis：Goroutine 分析
- Network blocking profile：网络阻塞概况
- Synchronization blocking profile：同步阻塞概况
- Syscall blocking profile：系统调用阻塞概况
- Scheduler latency profile：调度延迟概况
- User defined tasks：用户自定义任务
- User defined regions：用户自定义区域
- Minimum mutator utilization：最低 Mutator 利用率

### 1.1 调度延迟概况

在刚开始查看问题时，除非是很明显的现象，否则不应该一开始就陷入细节，因此我们一般先查看 “Scheduler latency profile”，我们能通过 Graph 看到整体的调用开销情况，如下：

![image](https://golang2.eddycjy.com/images/ch6/trace_scheduler_latency_profile.png)

演示程序比较简单，因此这里就两块，一个是 `trace` 本身，另外一个是 `channel` 的收发。

### 1.2 Goroutine 分析

第二步看 “Goroutine analysis”，我们能通过这个功能看到整个运行过程中，每个函数块有多少个有 Goroutine 在跑，并且观察每个的 Goroutine 的运行开销都花费在哪个阶段。如下：

![image](https://golang2.eddycjy.com/images/ch6/trace_goroutine_analysis.png)

通过上图我们可以看到共有 3 个 goroutine，分别是 `runtime.main`、`runtime/trace.Start.func1`、`main.main.func1`，那么它都做了些什么事呢，接下来我们可以通过点击具体细项去观察。如下：

![image](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/trace_goroutine_analysis_main.jpg)

同时也可以看到当前 Goroutine 在整个调用耗时中的占比，以及 GC 清扫和 GC 暂停等待的一些开销。如果你觉得还不够，可以把图表下载下来分析，相当于把整个 Goroutine 运行时掰开来看了，这块能够很好的帮助我们**对 Goroutine 运行阶段做一个的剖析，可以得知到底慢哪，然后再决定下一步的排查方向**。如下：

| 名称                  | 含义         | 耗时   |
| --------------------- | ------------ | ------ |
| Execution Time        | 执行时间     | 3140ns |
| Network Wait Time     | 网络等待时间 | 0ns    |
| Sync Block Time       | 同步阻塞时间 | 0ns    |
| Blocking Syscall Time | 调用阻塞时间 | 0ns    |
| Scheduler Wait Time   | 调度等待时间 | 14ns   |
| GC Sweeping           | GC 清扫      | 0ns    |
| GC Pause              | GC 暂停      | 0ns    |

### 1.3 查看跟踪

在对当前程序的 Goroutine 运行分布有了初步了解后，我们再通过 “查看跟踪” 看看之间的关联性，如下：

![image](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/trace_view_trace.png)

这个跟踪图粗略一看，相信有的小伙伴会比较懵逼，我们可以依据注解一块块查看，如下：

1. 时间线：显示执行的时间单元，根据时间维度的不同可以调整区间，具体可执行 `shift` + `?` 查看帮助手册。
2. 堆：显示执行期间的内存分配和释放情况。
3. 协程：显示在执行期间的每个 Goroutine 运行阶段有多少个协程在运行，其包含 GC 等待（GCWaiting）、可运行（Runnable）、运行中（Running）这三种状态。
4. OS 线程：显示在执行期间有多少个线程在运行，其包含正在调用 Syscall（InSyscall）、运行中（Running）这两种状态。
5. 虚拟处理器：每个虚拟处理器显示一行，虚拟处理器的数量一般默认为系统内核数。
6. 协程和事件：显示在每个虚拟处理器上有什么 Goroutine 正在运行，而连线行为代表事件关联。

![image](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/trace_view_trace_event.jpg)

点击具体的 Goroutine 行为后可以看到其相关联的详细信息，这块很简单，大家实际操作一下就懂了。文字解释如下：

- Start：开始时间
- Wall Duration：持续时间
- Self Time：执行时间
- Start Stack Trace：开始时的堆栈信息
- End Stack Trace：结束时的堆栈信息
- Incoming flow：输入流
- Outgoing flow：输出流
- Preceding events：之前的事件
- Following events：之后的事件
- All connected：所有连接的事件

### 1.4 查看事件

我们可以通过点击 View Options-Flow events、Following events 等方式，查看我们应用运行中的事件流情况。如下：

![image](https://golang2.eddycjy.com/images/ch6/trace_view_events-1.png)

通过分析图上的事件流，我们可得知这程序从 `G1 runtime.main` 开始运行，在运行时创建了 2 个 Goroutine，先是创建 `G18 runtime/trace.Start.func1`，然后再是 `G19 main.main.func1` 。而同时我们可以通过其 Goroutine Name 去了解它的调用类型，如：`runtime/trace.Start.func1` 就是程序中在 `main.main` 调用了 `runtime/trace.Start` 方法，然后该方法又利用协程创建了一个闭包 `func1` 去进行调用。

![image](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/trace_view_events-2.png)

在这里我们结合开头的代码去看的话，很明显就是 `ch` 的输入输出的过程了。

## 2 实战演练

突然生产环境突然出现了问题，机智的你早已埋好 `_ "net/http/pprof"` 这个神奇的工具，你通过特定的方式执行了如下命令：

```shell
$ curl http://127.0.0.1:6060/debug/pprof/trace\?seconds\=20 > trace.out
$ go tool trace trace.out
```

### 2.1 查看跟踪

你很快的看到了熟悉的 List 界面，然后不信邪点开了 View trace 界面，如下：

![image](https://golang2.eddycjy.com/images/ch6/trace_drill-1.jpg)

完全看懵的你，稳住，对着合适的区域执行快捷键 `W` 不断地放大时间线，如下：

![image](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/trace_drill-2.jpg)

经过初步排查，你发现上述绝大部分的 G 竟然都和 `google.golang.org/grpc.(*Server).Serve.func` 有关，关联的一大串也是 `Serve` 所触发的相关动作。

![image](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/trace_drill-3.jpg)

这时候有经验的你心里已经有了初步结论，你可以继续追踪 View trace 深入进去，不过我建议先鸟瞰全貌，因此我们再往下看 “Network blocking profile” 和 “Syscall blocking profile” 所提供的信息，如下：

### 2.2 网络阻塞概况

![image](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/trace_block_profile-1.jpg)

### 2.3 系统调用阻塞概况

![image](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/trace_block_profile-2.jpg)

通过对以上三项的跟踪分析，加上这个泄露，这个阻塞的耗时，这个涉及的内部方法名，很明显就是哪位又忘记关闭客户端连接了，这时候我们就可以接下进行下一步的排查和修改了。

## 3 小结

通过本文我们习得了 `go tool trace` 的武林秘籍，它能够跟踪捕获各种执行中的事件，例如 Goroutine 的创建/阻塞/解除阻塞，Syscall 的进入/退出/阻止，GC 事件，Heap 的大小改变，Processor 启动/停止等等。

希望你能够用好 Go 的两大杀器 pprof + trace 组合，此乃排查好搭档，谁用谁清楚，即使他并不万能。