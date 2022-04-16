# GC

[参考](https://qcrao91.gitbook.io/go/gc/gc)

[参考2](https://segmentfault.com/a/1190000022030353)

## 一、概念

### 1. 什么是GC

在早期经常遭到唾弃的就是在垃圾回收 (Garbage Collection，下称：GC) 机制中 STW (Stop-The-World) 的时间过长。那么这个时候，我们又会好奇一点，作为 STW 的起始，Go 语言中什么时候才会触发 GC 呢？当程序向操作系统申请的内存不再需要时，垃圾回收主动将其回收并供其他代码进行内存申请时候复用，或者将其归还给操作系统，这种针对内存级别资源的自动回收过程，即为垃圾回收。而负责垃圾回收的程序组件，即为垃圾回收器。

在计算机科学中，**垃圾回收 (GC) 是一种自动管理内存的机制**，垃圾回收器会去尝试回收程序不再使用的对象及其占用的内存。

最早 John McCarthy 在 1959 年左右发明了垃圾回收，以简化 Lisp 中的手动内存管理的机制 (来自 @wikipedia)。

通常，**垃圾回收器的执行过程被划分为两个半独立的组件**：

- 赋值器（Mutator）：这一名称本质上是在指代用户态的代码。因为对垃圾回收器而言，用户态的代码仅仅只是在修改对象之间的引用关系，也就是在对象图（对象之间引用关系的一个有向图）上进行操作。
- 回收器（Collector）：负责执行垃圾回收的代码。

### 2. **为什么要 GC**

手动管理内存挺麻烦，管错或者管漏内存也很糟糕，将会直接导致程序不稳定 (持续泄露) 甚至直接崩溃。

### 3. 根对象什么

**根对象在垃圾回收的术语中又叫做根集合**，它是垃圾回收器在标记过程时最先检查的对象，包括：

1. 全局变量：程序在编译期就能确定的那些存在于程序整个生命周期的变量。
2. 执行栈：每个 goroutine 都包含自己的执行栈，这些执行栈上包含栈上的变量及指向分配的堆内存区块的指针。
3. 寄存器：寄存器的值可能表示一个指针，参与计算的这些指针可能指向某些赋值器分配的堆内存区块。

### 4. 常见的GC实现方式有哪些？Go语言的GC使用的是什么？

所有的 GC 算法其存在形式可以归结为**追踪（Tracing）和引用计数（Reference Counting）**这两种形式的混合运用。

- 追踪式 GC

  从根对象出发，根据对象之间的引用信息，一步步推进直到扫描完毕整个堆并确定需要保留的对象，从而回收所有可回收的对象。Go、 Java、V8 对 JavaScript 的实现等均为追踪式 GC。

- 引用计数式 GC

  每个对象自身包含一个被引用的计数器，当计数器归零时自动得到回收。因为此方法缺陷较多，在追求高性能时通常不被应用。Python、Objective-C 等均为引用计数式 GC。

目前比较常见的 GC 实现方式包括：

- 追踪式，分为多种不同类型，例如：
  - 标记清扫：从根对象出发，将确定存活的对象进行标记，并清扫可以回收的对象。
  - 标记整理：为了解决内存碎片问题而提出，在标记过程中，将对象尽可能整理到一块连续的内存上。
  - 增量式：将标记与清扫的过程分批执行，每次执行很小的部分，从而增量的推进垃圾回收，达到近似实时、几乎无停顿的目的。
  - 增量整理：在增量式的基础上，增加对对象的整理过程。
  - 分代式：将对象根据存活时间的长短进行分类，存活时间小于某个值的为年轻代，存活时间大于某个值的为老年代，永远不会参与回收的对象为永久代。并根据分代假设（如果一个对象存活时间不长则倾向于被回收，如果一个对象已经存活很长时间则倾向于存活更长时间）对对象进行回收。
- 引用计数：根据对象自身的引用计数来回收，当引用计数归零时立即回收。

关于各类方法的详细介绍及其实现不在本文中详细讨论。**对于 Go 而言，Go 的 GC 目前使用的是无分代（对象没有代际之分）、不整理（回收过程中不对对象进行移动与整理）、并发（与用户代码并发执行）的三色标记清扫算法**。原因[1]在于：

1. 对象整理的优势是解决内存碎片问题以及“允许”使用顺序内存分配器。但 Go 运行时的分配算法基于 tcmalloc，基本上没有碎片问题。 并且顺序内存分配器在多线程的场景下并不适用。Go 使用的是基于 tcmalloc 的现代内存分配算法，对对象进行整理不会带来实质性的性能提升。
2. 分代 GC 依赖分代假设，即 GC 将主要的回收目标放在新创建的对象上（存活时间短，更倾向于被回收），而非频繁检查所有对象。但 Go 的编译器会通过**逃逸分析**将大部分新生对象存储在栈上（栈直接被回收），只有那些需要长期存在的对象才会被分配到需要进行垃圾回收的堆中。也就是说，分代 GC 回收的那些存活时间短的对象在 Go 中是直接被分配到栈上，当 goroutine 死亡后栈也会被直接回收，不需要 GC 的参与，进而分代假设并没有带来直接优势。并且 Go 的垃圾回收器与用户代码并发执行，使得 STW 的时间与对象的代际、对象的 size 没有关系。Go 团队更关注于如何更好地让 GC 与用户代码并发执行（使用适当的 CPU 来执行垃圾回收），而非减少停顿时间这一单一目标上。

### 3、GC 触发场景

**Go 语言中 GC 的流程是什么？**

当前版本的 Go 以 STW 为界限，可以将 GC 划分为五个阶段：

![image-20220408153248786](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/image-20220408153248786.png)

具体而言，各个阶段的触发函数分别为：

![img](https://433327134-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LjLtSYqqsBQODAJIhQ5%2F-Lxxz34HPuUqSYyGYrbb%2F-Lxxz3x6DZ0Z9lfqgQEU%2Fgc-process.png?generation=1578366687417405&alt=media)

GC 触发的场景主要分为两大类，分别是：

**系统触发**：运行时自行根据内置的条件，检查、发现到，则进行 GC 处理，维护整个应用程序的可用性。

**手动触发**：开发者在业务代码中自行调用 runtime.GC 方法来触发 GC 行为。

**3.1 系统触发**

在系统触发的场景中，Go 源码的 src/runtime/mgc.go 文件，明确标识了 **GC 系统触发的三种场景**，分别如下：

```go
const ( 
 gcTriggerHeap gcTriggerKind = iota 
 gcTriggerTime 
 gcTriggerCycle 
) 
```

gcTriggerHeap：**当所分配的堆大小达到阈值** (由控制器计算的触发堆的大小) 时，将会触发。

gcTriggerTime：**当距离上一个 GC 周期的时间超过一定时间时**，将会触发。- 时间周期以`runtime.forcegcperiod `变量为准，默认 2 分钟。

gcTriggerCycle：**如果没有开启 GC，则启动 GC**。

在手动触发的 runtime.GC 方法中涉及。

**3.2 手动触发**

在手动触发的场景下，Go 语言中仅有 runtime.GC 方法可以触发，也就没什么额外的分类的。

但我们要思考的是，一般我们**在什么业务场景**中，要涉及到手动干涉 GC，强制触发他呢？

需要手动强制触发的场景极其少见，可能会是在某些业务方法执行完后，因其**占用了过多的内存**，需要人为释放。又或是 **debug 程序所需**。

**3.3 基本流程**

在了解到 Go 语言会触发 GC 的场景后，我们进一步看看触发 GC 的流程代码是怎么样的，我们可以借助手动触发的 runtime.GC 方法来作为突破口。

核心代码如下：

```go
func GC() { 
 n := atomic.Load(&work.cycles) 
 gcWaitOnMark(n) 

 gcStart(gcTrigger{kind: gcTriggerCycle, n: n + 1}) 

 gcWaitOnMark(n + 1) 

 for atomic.Load(&work.cycles) == n+1 && sweepone() != ^uintptr(0) { 
  sweep.nbgsweep++ 
  Gosched() 
 } 

 for atomic.Load(&work.cycles) == n+1 && atomic.Load(&mheap_.sweepers) != 0 { 
  Gosched() 
 } 

 mp := acquirem() 
 cycle := atomic.Load(&work.cycles) 
 if cycle == n+1 || (gcphase == _GCmark && cycle == n+2) { 
  mProf_PostSweep() 
 } 
 releasem(mp) 
} 
```

在开始新的一轮 GC 周期前，需要调用 gcWaitOnMark 方法上一轮 GC 的标记结束 (含扫描终止、标记、或标记终止等)。

开始新的一轮 GC 周期，调用 gcStart 方法触发 GC 行为，开始扫描标记阶段。

需要调用 gcWaitOnMark 方法等待，直到当前 GC 周期的扫描、标记、标记终止完成。

需要调用 sweepone 方法，扫描未扫除的堆跨度，并持续扫除，保证清理完成。在等待扫除完毕前的阻塞时间，会调用 Gosched 让出。

在本轮 GC 已经基本完成后，会调用 mProf_PostSweep 方法。以此记录最后一次标记终止时的堆配置文件快照。

结束，释放 M。

**3.4 在哪触发**

看完 GC 的基本流程后，我们有了一个基本的了解。但可能又有小伙伴有疑惑了？

本文的标题是 “GC 什么时候会触发 GC”，虽然我们前面知道了触发的时机。但是….Go 是哪里实现的触发的机制，似乎在流程中完全没有看到？

### 4、监控线程

实质上在 Go 运行时 (runtime) 初始化时，会启动一个 goroutine，用于处理 GC 机制的相关事项。

代码如下：

```go
func init() { 
 go forcegchelper() 
} 

func forcegchelper() { 
 forcegc.g = getg() 
 lockInit(&forcegc.lock, lockRankForcegc) 
 for { 
  lock(&forcegc.lock) 
  if forcegc.idle != 0 { 
   throw("forcegc: phase error") 
  } 
  atomic.Store(&forcegc.idle, 1) 
  goparkunlock(&forcegc.lock, waitReasonForceGCIdle, traceEvGoBlock, 1) 
    // this goroutine is explicitly resumed by sysmon 
  if debug.gctrace > 0 { 
   println("GC forced") 
  } 

  gcStart(gcTrigger{kind: gcTriggerTime, now: nanotime()}) 
 } 
} 
```

在这段程序中，需要特别关注的是在 forcegchelper 方法中，会调用 goparkunlock 方法让该 goroutine 陷入休眠等待状态，以减少不必要的资源开销。

在休眠后，会由 sysmon 这一个系统监控线程来进行监控、唤醒等行为：

```
func sysmon() { 
 ... 
 for { 
  ... 
  // check if we need to force a GC 
  if t := (gcTrigger{kind: gcTriggerTime, now: now}); t.test() && atomic.Load(&forcegc.idle) != 0 { 
   lock(&forcegc.lock) 
   forcegc.idle = 0 
   var list gList 
   list.push(forcegc.g) 
   injectglist(&list) 
   unlock(&forcegc.lock) 
  } 
  if debug.schedtrace > 0 && lasttrace+int64(debug.schedtrace)*1000000 <= now { 
   lasttrace = now 
   schedtrace(debug.scheddetail > 0) 
  } 
  unlock(&sched.sysmonlock) 
 } 
} 
```

这段代码核心的行为就是不断地在 for 循环中，对 gcTriggerTime 和 now 变量进行比较，判断是否达到一定的时间 (默认为 2 分钟)。

若达到意味着满足条件，会将 forcegc.g 放到全局队列中接受新的一轮调度，再进行对上面 forcegchelper 的唤醒。

### **5、堆内存申请**

在了解定时触发的机制后，另外一个场景就是分配的堆空间的时候，那么我们要看的地方就非常明确了。

那就是运行时申请堆内存的 mallocgc 方法。核心代码如下：

```go
func mallocgc(size uintptr, typ *_type, needzero bool) unsafe.Pointer { 
 shouldhelpgc := false
 ... 
 if size <= maxSmallSize { 
  if noscan && size < maxTinySize { 
   ... 
   // Allocate a new maxTinySize block. 
   span = c.alloc[tinySpanClass] 
   v := nextFreeFast(span) 
   if v == 0 { 
    v, span, shouldhelpgc = c.nextFree(tinySpanClass) 
   } 
   ... 
   spc := makeSpanClass(sizeclass, noscan) 
   span = c.alloc[spc] 
   v := nextFreeFast(span) 
   if v == 0 { 
    v, span, shouldhelpgc = c.nextFree(spc) 
   } 
   ... 
  } 
 } else { 
  shouldhelpgc = true
  span = c.allocLarge(size, needzero, noscan) 
  ... 
 } 

 if shouldhelpgc { 
  if t := (gcTrigger{kind: gcTriggerHeap}); t.test() { 
   gcStart(t) 
  } 
 } 

 return x 
} 
```

小对象：如果申请小对象时，发现当前内存空间不存在空闲跨度时，将会需要调用 nextFree 方法获取新的可用的对象，可能会触发 GC 行为。

大对象：如果申请大于 32k 以上的大对象时，可能会触发 GC 行为。



## 二、GC算法

| 版本 |                     GC 算法                     |
| :--: | :---------------------------------------------: |
| v1.1 |               STW(stop the word)                |
| v1.3 |            Mark STW,Sweep (标记清除)            |
| v1.5 |                    三色标记                     |
| v1.8 | hybrid write barrier (三色标记基础上加入写屏障) |

### STW

`STW` 可以是 `Stop the World` 的缩写，也可以是 `Start the World` 的缩写。通常意义上指指代从 `Stop the World` 这一动作发生时到 `Start the World` 这一动作发生时这一段时间间隔，即万物静止。STW 在垃圾回收过程中为了保证实现的正确性、防止无止境的内存增长等问题而不可避免的需要停止赋值器进一步操作对象图的一段过程。

**在这个过程中整个用户代码被停止或者放缓执行**， `STW` 越长，对用户代码造成的影响（例如延迟）就越大，早期 Go 对垃圾回收器的实现中 `STW` 长达几百毫秒，对时间敏感的实时通信等应用程序会造成巨大的影响。

[stw工作过程](https://segmentfault.com/a/1190000022499402)

#### STW的步骤

第一步，抢占所有正在运行的`goroutines`

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/1460000022499414)

第二步，一旦 `goroutines`被抢占，正在运行的`goroutines`将在安全的地方暂停，然后所有的p[1]都将被标记为暂停，停止运行任何代码。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/1460000022499409)

第三步，然后，go调度器将M[2]与P分离,并且将M放到空闲列表里面。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/1460000022499415)

对于在每个M上运行的Goroutines，它们将在全局队列[3]>中等待：

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/1460000022499408)

那么，一旦所有的`goroutines`都停止了，那么唯一活跃的`goroutines` （垃圾回收`goroutines`）将会安全的运行，并且在垃圾回收完成后，重新拉起所有的`goroutines`。具体情况，可以通过 go trace查看。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/1460000022499412)

#### System calls

STW时期可能会影响系统调用，因为系统调用可能会在stw时期返回，通过密集执行系统调用的程序来看看怎样处理这种情况，

```go
 func main() {  
  	var wg sync.WaitGroup  
  	wg.Add(10)   
	for i :\= 0; i < 10; i++ {  
         go func() {  
             http.Get(\`https://httpstat.us/200\`)  
             wg.Done()  
         }()  
     }   
     wg.Wait()  
 }
```

他的trace情况。

![img](https://segmentfault.com/img/remote/1460000022499411)

系统调用在STW时期返回，但是现在已经没有P在运行了。所以他会放到全局队列里面,等待STW结束后再运行。

#### Latencies

STW的第三步将所有的M与P分离。然而，go将等待调度程序运行、系统调用等自动停止。等待goroutine被抢占应该很快，但是在某些情况下会产生一些延迟，下面是一个极端的例子：

```go
 func main() {  
  var t int  
  for i :\= 0;i < 20 ;i++  {  
      go func() {  
          for i := 0;i < 1000000000 ;i++ {  
          	t++  
      	 }  
      }()  
  }  
   
  runtime.GC()  
 }
```

STW时长达到了2.6S

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/1460000022499413)

没有函数调用的goroutine将不会被抢占，并且它的P在任务结束之前不会被释放。

### 标记清除算法 (mark and sweep)

主要包含两个步骤:

1. 找出不可达对象，然后做上标记
2. 回收标记好的对象

mark and sweep 算法在执行的时候，需要程序暂停，即 stop the world

标记清除算法存在的问题

- stop the world 程序暂停，即程序会出现卡顿
- 标记需要扫描整个堆 (heap)
- 清楚数据会产生 heap 碎片



### 三色标记法

理解**三色标记法**的关键是理解对象的**三色抽象**以及**波面（wavefront）推进**这两个概念。三色抽象只是一种描述追踪式回收器的方法，在实践中并没有实际含义，**它的重要作用在于从逻辑上严密推导标记清理这种垃圾回收方法的正确性**。也就是说，当我们谈及三色标记法时，通常指标记清扫的垃圾回收。

从垃圾回收器的视角来看，三色抽象规定了三种不同类型的对象，并用不同的颜色相称：

- 白色对象（可能死亡）：未被回收器访问到的对象。在回收开始阶段，所有对象均为白色，当回收结束后，白色对象均不可达。
- 灰色对象（波面）：已被回收器访问到的对象，但回收器需要对其中的一个或多个指针进行扫描，因为他们可能还指向白色对象。
- 黑色对象（确定存活）：已被回收器访问到的对象，其中所有字段都已被扫描，黑色对象中任何一个指针都不可能直接指向白色对象。

这样三种不变性所定义的回收过程其实是一个**波面**不断前进的过程，这个波面同时也是黑色对象和白色对象的边界，灰色对象就是这个波面。

当垃圾回收开始时，只有白色对象。随着标记过程开始进行时，灰色对象开始出现（着色），这时候波面便开始扩大。当一个对象的所有子节点均完成扫描时，会被着色为黑色。当整个堆遍历完成时，只剩下黑色和白色对象，这时的黑色对象为可达对象，即存活；而白色对象为不可达对象，即死亡。这个过程可以视为以灰色对象为波面，将黑色对象和白色对象分离，使波面不断向前推进，直到所有可达的灰色对象都变为黑色对象为止的过程。如下图所示：

![img](https://433327134-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LjLtSYqqsBQODAJIhQ5%2F-Lxxz34HPuUqSYyGYrbb%2F-Lxxz3wrkAVKZ93IQ_MI%2Fgc-blueprint.png?generation=1578366688391319&alt=media)

三色标记法全貌

图中展示了根对象、可达对象、不可达对象，黑、灰、白对象以及波面之间的关系。

三色标记清除算法背后的首要原则就是它把堆中的对象根据它们的颜色分到不同集合里面，颜色是根据算法进行标记的

- 黑色集合 指针指向白色集合。

- 白色集合中的对象允许有指针指向黑色集合，白色集合中的对象就是垃圾回收的候选对象。
- 灰色集合可能会有指针指向白色集合里的对象。

**三色标记过程**

1. 首先：程序创建的对象都标记为白色。

   ![img](https://cdn.learnku.com/uploads/images/202006/18/21196/QgfqbyCXcy.png!large)

2. gc 开始：扫描所有可到达的对象，标记为灰色

![img](https://cdn.learnku.com/uploads/images/202006/18/21196/xofJzpKOMF.png!large)

3.  从灰色对象中找到其引用对象标记为灰色，把灰色对象本身标记为黑色

![img](https://cdn.learnku.com/uploads/images/202006/18/21196/m5xOllBECd.png!large)

4. 监视对象中的内存修改，并持续上一步的操作，直到灰色标记的对象不存在

![img](https://cdn.learnku.com/uploads/images/202006/18/21196/dFhjL88XES.png!large)

5.  此时，gc 回收白色对象。

   ![img](https://cdn.learnku.com/uploads/images/202006/18/21196/6T3l9s38Z4.png!large)

6. 最后，将所有黑色对象变为白色，并重复以上所有过程。

![img](https://cdn.learnku.com/uploads/images/202006/18/21196/Aasw7TXGSY.png!large)



### 并发标记清除法的难点

**在没有用户态代码并发修改`三色抽象`的情况下，回收可以正常结束。但是并发回收的根本问题在于，用户态代码在回收过程中会并发地更新对象图，从而造成赋值器和回收器可能对对象图的结构产生不同的认知。**这时以一个固定的三色波面作为回收过程前进的边界则不再合理。

![image-20220408151647827](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/image-20220408151647827.png)

回收器：由于所有子节点均已标记，回收器也不会重新扫描已经被标记为黑色的对象，此时 A 被着色为黑色，`scan(A)` 什么也不会发生，进而 B 在此次回收过程中永远不会被标记为黑色，进而错误地被回收。

- 初始状态：假设某个黑色对象 C 指向某个灰色对象 A ，而 A 指向白色对象 B；
- `C.ref3 = C.ref2.ref1`：赋值器并发地将黑色对象 C 指向（ref3）了白色对象 B；
- `A.ref1 = nil`：移除灰色对象 A 对白色对象 B 的引用（ref2）；
- 最终状态：在继续扫描的过程中，白色对象 B 永远不会被标记为黑色对象了（回收器不会重新扫描黑色对象），进而对象 B 被错误地回收。

![img](https://433327134-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LjLtSYqqsBQODAJIhQ5%2F-LxPbWTGWIkowf_Q2b_P%2F-LxPbXDIxqboY8-HxHTa%2Fgc-mutator.png?generation=1577773325275884&alt=media)

总而言之，**并发标记清除中面临的一个根本问题就是如何保证标记与清除过程的正确性**。



### 什么是写屏障、混合写屏障，如何实现？

#### 快速理解

process 新生成对象的时候，GC 该如何操作呢？不会乱吗？

我们看如下图，在此状态下：process 程序又新生成了一个对象，我们设想会变成这样：

![img](https://cdn.learnku.com/uploads/images/202006/18/21196/mbi7k8n6A9.png!large)

但是这样显然是不对的，因为按照三色标记法的步骤，这样新生成的对象 A 最后会被清除掉，这样会影响程序逻辑。
Golang 为了解决这个问题，引入了写屏障这个机制。

**写屏障：该屏障之前的写操作和之后的写操作相比，先被系统其它组件感知。**

通俗的讲：就是在 gc 跑的过程中，可以监控对象的内存修改，并对对象进行重新标记。(实际上也是超短暂的 stw，然后对对象进行标记)

每次堆中的指针被修改写屏障都会去执行。如果堆中对象的指针被修改，就意味着那个对象现在是可触达的，写屏障会把它标记为灰色并把它放到灰色集合中。

**修改器 运行写屏障，从而保证黑色集合中没有任何元素的指针去指向白色集合中的元素。**

写屏障直观作用有两个:

1. process 新生成的内存对象会被直接标记成灰色

2. 位于黑色集合中的内存对象引用了一个白色集合中的对象，写屏障会将白色集合的这个对象标记为灰色

gc 一旦开始，无论是创建对象还是对象的引用改变，都会先变为灰色。



#### 深入解析

要讲清楚写屏障，就需要理解三色标记清除算法中的**强弱不变性**以及**赋值器的颜色**，理解他们需要一定的抽象思维。写屏障是一个在并发垃圾回收器中才会出现的概念，垃圾回收器的正确性体现在：**不应出现对象的丢失，也不应错误的回收还不需要回收的对象。**

可以证明，当以下两个条件同时满足时会破坏垃圾回收器的正确性：

- **条件 1**: 赋值器修改对象图，导致某一黑色对象引用白色对象；
- **条件 2**: 从灰色对象出发，到达白色对象的、未经访问过的路径被赋值器破坏。

只要能够避免其中任何一个条件，则不会出现对象丢失的情况，因为：

- 如果条件 1 被避免，则所有白色对象均被灰色对象引用，没有白色对象会被遗漏；
- 如果条件 2 被避免，即便白色对象的指针被写入到黑色对象中，但从灰色对象出发，总存在一条没有访问过的路径，从而找到到达白色对象的路径，白色对象最终不会被遗漏。

我们不妨将三色不变性所定义的波面根据这两个条件进行削弱：

- 当满足原有的三色不变性定义（或上面的两个条件都不满足时）的情况称为**强三色不变性（strong tricolor invariant）**
- 当赋值器令黑色对象引用白色对象时（满足条件 1 时）的情况称为**弱三色不变性（weak tricolor invariant）**

当赋值器进一步破坏灰色对象到达白色对象的路径时（进一步满足条件 2 时），即打破弱三色不变性， 也就破坏了回收器的正确性；或者说，在破坏强弱三色不变性时必须引入额外的辅助操作。 弱三色不变形的好处在于：**只要存在未访问的能够到达白色对象的路径，就可以将黑色对象指向白色对象。**

如果我们考虑并发的用户态代码，回收器不允许同时停止所有赋值器，就是涉及了存在的多个不同状态的赋值器。为了对概念加以明确，还需要换一个角度，把回收器视为对象，把赋值器视为影响回收器这一对象的实际行为（即影响 GC 周期的长短），从而引入赋值器的颜色：

- 黑色赋值器：已经由回收器扫描过，不会再次对其进行扫描。
- 灰色赋值器：尚未被回收器扫描过，或尽管已经扫描过但仍需要重新扫描。

赋值器的颜色对回收周期的结束产生影响：

- 如果某种并发回收器允许灰色赋值器的存在，则必须在回收结束之前重新扫描对象图。
- 如果重新扫描过程中发现了新的灰色或白色对象，回收器还需要对新发现的对象进行追踪，但是在新追踪的过程中，赋值器仍然可能在其根中插入新的非黑色的引用，如此往复，直到重新扫描过程中没有发现新的白色或灰色对象。

于是，在允许灰色赋值器存在的算法，最坏的情况下，回收器只能将所有赋值器线程停止才能完成其跟对象的完整扫描，也就是我们所说的 STW。

为了确保强弱三色不变性的并发指针更新操作，需要通过赋值器屏障技术来保证指针的读写操作一致。因此我们所说的 Go 中的写屏障、混合写屏障，其实是指赋值器的写屏障，赋值器的写屏障作为一种同步机制，使赋值器在进行指针写操作时，能够“通知”回收器，进而不会破坏弱三色不变性。

有两种非常经典的写屏障：**Dijkstra 插入屏障和 Yuasa 删除屏障。**

灰色赋值器的 Dijkstra 插入屏障的基本思想是避免满足条件 1：

```
// 灰色赋值器 Dijkstra 插入屏障
func DijkstraWritePointer(slot *unsafe.Pointer, ptr unsafe.Pointer) {
    shade(ptr)
    *slot = ptr
}
```

为了防止黑色对象指向白色对象，应该假设 `*slot` 可能会变为黑色，为了确保 `ptr` 不会在被赋值到 `*slot` 前变为白色，`shade(ptr)` 会先将指针 `ptr` 标记为灰色，进而避免了条件 1。如图所示：

![img](https://433327134-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LjLtSYqqsBQODAJIhQ5%2F-Lxxz34HPuUqSYyGYrbb%2F-LxWIclhYCSYqaN6skbo%2Fgc-wb-dijkstra.png?generation=1578366683649015&alt=media)

Dijkstra 插入屏障的好处在于可以立刻开始并发标记。但存在两个缺点：

1. 由于 Dijkstra 插入屏障的“保守”，在一次回收过程中可能会残留一部分对象没有回收成功，只有在下一个回收过程中才会被回收；
2. 在标记阶段中，每次进行指针赋值操作时，都需要引入写屏障，这无疑会增加大量性能开销；为了避免造成性能问题，Go 团队在最终实现时，没有为所有栈上的指针写操作，启用写屏障，而是当发生栈上的写操作时，将栈标记为灰色，但此举产生了灰色赋值器，将会需要标记终止阶段 STW 时对这些栈进行重新扫描。

另一种比较经典的写屏障是黑色赋值器的 Yuasa 删除屏障。其基本思想是避免满足条件 2：

```
// 黑色赋值器 Yuasa 屏障
func YuasaWritePointer(slot *unsafe.Pointer, ptr unsafe.Pointer) {
    shade(*slot)
    *slot = ptr
}
```

为了防止丢失从灰色对象到白色对象的路径，应该假设 `*slot` 可能会变为黑色，为了确保 `ptr` 不会在被赋值到 `*slot` 前变为白色，`shade(*slot)` 会先将 `*slot` 标记为灰色，进而该写操作总是创造了一条灰色到灰色或者灰色到白色对象的路径，进而避免了条件 2。

Yuasa 删除屏障的优势则在于不需要标记结束阶段的重新扫描，结束时候能够准确的回收所有需要回收的白色对象。缺陷是 Yuasa 删除屏障会拦截写操作，进而导致波面的退后，产生“冗余”的扫描：

![img](https://433327134-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LjLtSYqqsBQODAJIhQ5%2F-Lxxz34HPuUqSYyGYrbb%2F-LxWIcljJBnHL1q8dFqC%2Fgc-wb-yuasa.png?generation=1578366685654095&alt=media)

Go 在 1.8 的时候为了简化 GC 的流程，同时减少标记终止阶段的重扫成本，将 Dijkstra 插入屏障和 Yuasa 删除屏障进行混合，形成混合写屏障。该屏障提出时的基本思想是：**对正在被覆盖的对象进行着色，且如果当前栈未扫描完成，则同样对指针进行着色。**

但在最终实现时原提案[4]中对 `ptr` 的着色还额外包含对执行栈的着色检查，但由于时间有限，并未完整实现过，所以混合写屏障在目前的实现伪代码是：

```
// 混合写屏障
func HybridWritePointerSimple(slot *unsafe.Pointer, ptr unsafe.Pointer) {
    shade(*slot)
    shade(ptr)
    *slot = ptr
}
```

在这个实现中，如果无条件对引用双方进行着色，自然结合了 Dijkstra 和 Yuasa 写屏障的优势，但缺点也非常明显，因为着色成本是双倍的，而且编译器需要插入的代码也成倍增加，随之带来的结果就是编译后的二进制文件大小也进一步增加。为了针对写屏障的性能进行优化，Go 1.10 前后，Go 团队随后实现了批量写屏障机制。其基本想法是将需要着色的指针统一写入一个缓存，每当缓存满时统一对缓存中的所有 `ptr` 指针进行着色。

#### [屏障机制](https://segmentfault.com/a/1190000022030353)

#### [混合写屏障(hybrid write barrier)机制](https://segmentfault.com/a/1190000022030353)

插入写屏障和删除写屏障的短板：

- 插入写屏障：结束时需要STW来重新扫描栈，标记栈上引用的白色对象的存活；
- 删除写屏障：回收精度低，GC开始时STW扫描堆栈来记录初始快照，这个过程会保护开始时刻的所有存活对象。

Go V1.8版本引入了混合写屏障机制（hybrid write barrier），避免了对栈re-scan的过程，极大的减少了STW的时间。结合了两者的优点。

##### (1) 混合写屏障规则

`具体操作`:

1、GC开始**将栈上的对象全部扫描并标记为黑色**(之后不再进行第二次重复扫描，无需STW)，

2、GC期间，**任何在栈上创建的新对象，均为黑色**。

3、被删除的对象标记为灰色。

4、被添加的对象标记为灰色。

`满足`: 变形的**弱三色不变式**.



## GC 的优化问题

### GC 关注的指标有哪些？

**Go 的 GC 被设计为成比例触发、大部分工作与赋值器并发、不分代、无内存移动且会主动向操作系统归还申请的内存**。因此最主要关注的、能够影响赋值器的性能指标有：

- CPU 利用率：回收算法会在多大程度上拖慢程序？有时候，这个是通过回收占用的 CPU 时间与其它 CPU 时间的百分比来描述的。
- GC 停顿时间：回收器会造成多长时间的停顿？目前的 GC 中需要考虑 STW 和 Mark Assist 两个部分可能造成的停顿。
- GC 停顿频率：回收器造成的停顿频率是怎样的？目前的 GC 中需要考虑 STW 和 Mark Assist 两个部分可能造成的停顿。
- GC 可扩展性：当堆内存变大时，垃圾回收器的性能如何？但大部分的程序可能并不一定关心这个问题。

### Go 的 GC 如何调优？

[参考](https://qcrao91.gitbook.io/go/gc/gc#14.-go-de-gc-ru-he-tiao-you)

Go 的 GC 被设计为极致简洁，与较为成熟的 Java GC 的数十个可控参数相比，严格意义上来讲，Go 可供用户调整的参数只有 GOGC 环境变量。**当我们谈论 GC 调优时，通常是指减少用户代码对 GC 产生的压力，这一方面包含了减少用户代码分配内存的数量（即对程序的代码行为进行调优），另一方面包含了最小化 Go 的 GC 对 CPU 的使用率（即调整 GOGC）**。

GC 的调优是在特定场景下产生的，并非所有程序都需要针对 GC 进行调优。**只有那些对执行延迟非常敏感、 当 GC 的开销成为程序性能瓶颈的程序，才需要针对 GC 进行性能调优**，几乎不存在于实际开发中 99% 的情况。 除此之外，Go 的 GC 也仍然有一定的可改进的空间，也有部分 GC 造成的问题，目前仍属于 Open Problem。

总的来说，我们可以在现在的开发中处理的有以下几种情况：

1. 对停顿敏感：GC 过程中产生的长时间停顿、或由于需要执行 GC 而没有执行用户代码，导致需要立即执行的用户代码执行滞后。
2. 对资源消耗敏感：对于频繁分配内存的应用而言，频繁分配内存增加 GC 的工作量，原本可以充分利用 CPU 的应用不得不频繁地执行垃圾回收，影响用户代码对 CPU 的利用率，进而影响用户代码的执行效率。

从这两点来看，所谓 GC 调优的核心思想也就是充分的围绕上面的两点来展开：**优化内存的申请速度，尽可能的少申请内存，复用已申请的内存**。或者简单来说，不外乎这三个关键字：**控制、减少、复用**。

我们将通过三个实际例子介绍如何定位 GC 的存在的问题，并一步一步进行性能调优。当然，在实际情况中问题远比这些例子要复杂，这里也只是讨论调优的核心思想，更多的时候也只能具体问题具体分析。

**合理化内存分配的速度、提高赋值器的 CPU 利用率**

**降低并复用已经申请的内存**

**调整 GOGC**



### Go 的垃圾回收器有哪些相关的 API？其作用分别是什么？

在 Go 中存在数量极少的与 GC 相关的 API，它们是

- `runtime.GC`：手动触发 GC
- `runtime.ReadMemStats`：读取内存相关的统计信息，其中包含部分 GC 相关的统计信息
- `debug.FreeOSMemory`：手动将内存归还给操作系统
- `debug.ReadGCStats`：读取关于 GC 的相关统计信息
- `debug.SetGCPercent`：设置 GOGC 调步变量
- `debug.SetMaxHeap`（尚未发布[10]）：设置 Go 程序堆的上限值



## 相关问题

### 有了GC，为什么还是会有内存泄漏

在一个具有 GC 的语言中，我们常说的内存泄漏，用严谨的话来说应该是：**预期的能很快被释放的内存由于附着在了长期存活的内存上、或生命期意外地被延长，导致预计能够立即回收的内存而长时间得不到回收**。

在 Go 中，由于 goroutine 的存在，所谓的内存泄漏除了附着在长期对象上之外，还存在多种不同的形式。

#### 形式1：预期能被快速释放的内存因被根对象引用而没有得到迅速释放

当有一个全局对象时，可能不经意间将某个变量附着在其上，且忽略的将其进行释放，则该内存永远不会得到释放。例如：

```
var cache = map[interface{}]interface{}{}

func keepalloc() {
    for i := 0; i < 10000; i++ {
        m := make([]byte, 1<<10)
        cache[i] = m
    }
}
```

#### 形式2：goroutine 泄漏

Goroutine 作为一种逻辑上理解的轻量级线程，需要维护执行用户代码的上下文信息。在运行过程中也需要消耗一定的内存来保存这类信息，而这些内存在目前版本的 Go 中是不会被释放的。因此，**如果一个程序持续不断地产生新的 goroutine、且不结束已经创建的 goroutine 并复用这部分内存，就会造成内存泄漏的现象**，例如：

```
func keepalloc2() {
    for i := 0; i < 100000; i++ {
        go func() {
            select {}
        }()
    }
}
```

**验证**

我们可以通过如下形式来调用上述两个函数：

```
package main

import (
    "os"
    "runtime/trace"
)

func main() {
    f, _ := os.Create("trace.out")
    defer f.Close()
    trace.Start(f)
    defer trace.Stop()
    keepalloc()
    keepalloc2()
}
```

会看到程序中生成了 `trace.out` 文件，我们可以使用 `go tool trace trace.out` 命令得到下图：

![img](https://433327134-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-LjLtSYqqsBQODAJIhQ5%2F-LxPbWTGWIkowf_Q2b_P%2F-LxPbXDFz2lrTOOX6GN3%2Fgc-leak1.png?generation=1577773322335909&alt=media)

可以看到，途中的 Heap 在持续增长，没有内存被回收，产生了内存泄漏的现象。

值得一提的是，这种形式的 goroutine 泄漏还可能由 channel 泄漏导致。而 channel 的泄漏本质上与 goroutine 泄漏存在直接联系。Channel 作为一种同步原语，会连接两个不同的 goroutine，如果一个 goroutine 尝试向一个没有接收方的无缓冲 channel 发送消息，则该 goroutine 会被永久的休眠，整个 goroutine 及其执行栈都得不到释放，例如：

```
var ch = make(chan struct{})

func keepalloc3() {
    for i := 0; i < 100000; i++ {
        // 没有接收方，goroutine 会一直阻塞
        go func() { ch <- struct{}{} }()
    }
}
```

### 如果内存分配速度超过了标记清除的速度怎么办？

目前的 Go 实现中，当 GC 触发后，会首先进入并发标记的阶段。并发标记会设置一个标志，并在 mallocgc 调用时进行检查。当存在新的内存分配时，会暂停分配内存过快的那些 goroutine，并将其转去执行一些辅助标记（Mark Assist）的工作，从而达到放缓继续分配、辅助 GC 的标记工作的目的。

编译器会分析用户代码，并在需要分配内存的位置，将申请内存的操作翻译为 `mallocgc` 调用，而 `mallocgc` 的实现决定了标记辅助的实现，其伪代码思路如下：

```
func mallocgc(t typ.Type, size uint64) {
    if enableMarkAssist {
        // 进行标记辅助，此时用户代码没有得到执行
        (...)
    }
    // 执行内存分配
    (...)
}
```

