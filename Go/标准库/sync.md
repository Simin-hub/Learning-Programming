# sync

[官网地址](https://pkg.go.dev/sync#section-documentation)

[参考地址1](https://www.topgoer.com/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/sync.html)

[参考地址2](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-sync-primitives/#621-%E5%9F%BA%E6%9C%AC%E5%8E%9F%E8%AF%AD)

- [type Cond](https://pkg.go.dev/sync#Cond)
- - [func NewCond(l Locker) *Cond](https://pkg.go.dev/sync#NewCond)
- - [func (c *Cond) Broadcast()](https://pkg.go.dev/sync#Cond.Broadcast)
  - [func (c *Cond) Signal()](https://pkg.go.dev/sync#Cond.Signal)
  - [func (c *Cond) Wait()](https://pkg.go.dev/sync#Cond.Wait)
- [type Locker](https://pkg.go.dev/sync#Locker)
- [type Map](https://pkg.go.dev/sync#Map)
- - [func (m *Map) Delete(key any)](https://pkg.go.dev/sync#Map.Delete)
  - [func (m *Map) Load(key any) (value any, ok bool)](https://pkg.go.dev/sync#Map.Load)
  - [func (m *Map) LoadAndDelete(key any) (value any, loaded bool)](https://pkg.go.dev/sync#Map.LoadAndDelete)
  - [func (m *Map) LoadOrStore(key, value any) (actual any, loaded bool)](https://pkg.go.dev/sync#Map.LoadOrStore)
  - [func (m *Map) Range(f func(key, value any) bool)](https://pkg.go.dev/sync#Map.Range)
  - [func (m *Map) Store(key, value any)](https://pkg.go.dev/sync#Map.Store)
- [type Mutex](https://pkg.go.dev/sync#Mutex)
- - [func (m *Mutex) Lock()](https://pkg.go.dev/sync#Mutex.Lock)
  - [func (m *Mutex) TryLock() bool](https://pkg.go.dev/sync#Mutex.TryLock)
  - [func (m *Mutex) Unlock()](https://pkg.go.dev/sync#Mutex.Unlock)
- [type Once](https://pkg.go.dev/sync#Once)
- - [func (o *Once) Do(f func())](https://pkg.go.dev/sync#Once.Do)
- [type Pool](https://pkg.go.dev/sync#Pool)
- - [func (p *Pool) Get() any](https://pkg.go.dev/sync#Pool.Get)
  - [func (p *Pool) Put(x any)](https://pkg.go.dev/sync#Pool.Put)
- [type RWMutex](https://pkg.go.dev/sync#RWMutex)
- - [func (rw *RWMutex) Lock()](https://pkg.go.dev/sync#RWMutex.Lock)
  - [func (rw *RWMutex) RLock()](https://pkg.go.dev/sync#RWMutex.RLock)
  - [func (rw *RWMutex) RLocker() Locker](https://pkg.go.dev/sync#RWMutex.RLocker)
  - [func (rw *RWMutex) RUnlock()](https://pkg.go.dev/sync#RWMutex.RUnlock)
  - [func (rw *RWMutex) TryLock() bool](https://pkg.go.dev/sync#RWMutex.TryLock)
  - [func (rw *RWMutex) TryRLock() bool](https://pkg.go.dev/sync#RWMutex.TryRLock)
  - [func (rw *RWMutex) Unlock()](https://pkg.go.dev/sync#RWMutex.Unlock)
- [type WaitGroup](https://pkg.go.dev/sync#WaitGroup)
- - [func (wg *WaitGroup) Add(delta int)](https://pkg.go.dev/sync#WaitGroup.Add)
  - [func (wg *WaitGroup) Done()](https://pkg.go.dev/sync#WaitGroup.Done)
  - [func (wg *WaitGroup) Wait()](https://pkg.go.dev/sync#WaitGroup.Wait)

## type Cond

一句话总结：`sync.Cond` 条件变量用来协调想要访问共享资源的那些 goroutine，**当共享资源的状态发生变化的时候，它可以用来通知被互斥锁阻塞的 goroutine**。

`sync.Cond` 基于互斥锁/读写锁，它和互斥锁的区别是什么呢？

互斥锁 `sync.Mutex` 通常用来**保护临界区和共享资源**，条件变量 `sync.Cond` 用来**协调想要访问共享资源的 goroutine**。

`sync.Cond` 经常用在**多个 goroutine 等待，一个 goroutine 通知（事件发生）的场景**。如果是一个通知，一个等待，使用互斥锁或 channel 就能搞定了。

我们想象一个非常简单的场景：

有一个协程在异步地接收数据，剩下的多个协程必须等待这个协程接收完数据，才能读取到正确的数据。在这种情况下，如果单纯使用 chan 或互斥锁，那么只能有一个协程可以等待，并读取到数据，没办法通知其他的协程也读取数据。

这个时候，就需要有个全局的变量来标志第一个协程数据是否接受完毕，剩下的协程，反复检查该变量的值，直到满足要求。或者创建多个 channel，每个协程阻塞在一个 channel 上，由接收数据的协程在数据接收完毕后，逐个通知。总之，需要额外的复杂度来完成这件事。

Go 语言在标准库 sync 中内置一个 `sync.Cond` 用来解决这类问题。

###  sync.Cond 的四个方法

sync.Cond 的定义如下：

```
// Each Cond has an associated Locker L (often a *Mutex or *RWMutex),
// which must be held when changing the condition and
// when calling the Wait method.
//
// A Cond must not be copied after first use.
type Cond struct {
        noCopy noCopy

        // L is held while observing or changing the condition
        L Locker

        notify  notifyList
        checker copyChecker
}
```

每个 Cond 实例都会关联一个锁 L（互斥锁 *Mutex，或读写锁 *RWMutex），当修改条件或者调用 Wait 方法时，必须加锁。

和 sync.Cond 相关的有如下几个方法：

####  NewCond 创建实例

```
func NewCond(l Locker) *Cond
```

NewCond 创建 Cond 实例时，需要关联一个锁。

#### Broadcast 广播唤醒所有

```
// Broadcast wakes all goroutines waiting on c.
//
// It is allowed but not required for the caller to hold c.L
// during the call.
func (c *Cond) Broadcast()
```

Broadcast 唤醒所有等待条件变量 c 的 goroutine，无需锁保护。

#### Signal 唤醒一个协程

```
// Signal wakes one goroutine waiting on c, if there is any.
//
// It is allowed but not required for the caller to hold c.L
// during the call.
func (c *Cond) Signal()
```

Signal 只唤醒任意 1 个等待条件变量 c 的 goroutine，无需锁保护。

#### Wait 等待

```
// Wait atomically unlocks c.L and suspends execution
// of the calling goroutine. After later resuming execution,
// Wait locks c.L before returning. Unlike in other systems,
// Wait cannot return unless awoken by Broadcast or Signal.
//
// Because c.L is not locked when Wait first resumes, the caller
// typically cannot assume that the condition is true when
// Wait returns. Instead, the caller should Wait in a loop:
//
//    c.L.Lock()
//    for !condition() {
//        c.Wait()
//    }
//    ... make use of condition ...
//    c.L.Unlock()
//
func (c *Cond) Wait()
```

**调用 Wait 会自动释放锁 c.L，并挂起调用者所在的 goroutine，因此当前协程会阻塞在 Wait 方法调用的地方。如果其他协程调用了 Signal 或 Broadcast 唤醒了该协程，那么 Wait 方法在结束阻塞时，会重新给 c.L 加锁，并且继续执行 Wait 后面的代码**。

对条件的检查，使用了 `for !condition()` 而非 `if`，是因为当前协程被唤醒时，条件不一定符合要求，需要再次 Wait 等待下次被唤醒。为了保险起见，使用 `for` 能够确保条件符合要求后，再执行后续的代码。

```
c.L.Lock()
for !condition() {
    c.Wait()
}
... make use of condition ...
c.L.Unlock()
```

###  使用示例

接下来我们实现一个简单的例子，三个协程调用 `Wait()` 等待，另一个协程调用 `Broadcast()` 唤醒所有等待的协程。

```
var done = false

func read(name string, c *sync.Cond) {
	c.L.Lock()
	for !done {
		c.Wait()
	}
	log.Println(name, "starts reading")
	c.L.Unlock()
}

func write(name string, c *sync.Cond) {
	log.Println(name, "starts writing")
	time.Sleep(time.Second)
	c.L.Lock()
	done = true
	c.L.Unlock()
	log.Println(name, "wakes all")
	c.Broadcast()
}

func main() {
	cond := sync.NewCond(&sync.Mutex{})

	go read("reader1", cond)
	go read("reader2", cond)
	go read("reader3", cond)
	write("writer", cond)

	time.Sleep(time.Second * 3)
}
```

- `done` 即互斥锁需要保护的条件变量。
- `read()` 调用 `Wait()` 等待通知，直到 done 为 true。
- `write()` 接收数据，接收完成后，将 done 置为 true，调用 `Broadcast()` 通知所有等待的协程。
- `write()` 中的暂停了 1s，一方面是模拟耗时，另一方面是确保前面的 3 个 read 协程都执行到 `Wait()`，处于等待状态。main 函数最后暂停了 3s，确保所有操作执行完毕。

运行结果如下：

```
$ go run main.go
2021/01/14 23:18:20 writer starts writing
2021/01/14 23:18:21 writer wakes all
2021/01/14 23:18:21 reader2 starts reading
2021/01/14 23:18:21 reader3 starts reading
2021/01/14 23:18:21 reader1 starts reading
```

writer 接收数据花费了 1s，同步通知所有等待的协程。

> 更多关于 sync.Cond 的讨论可参考 [How to correctly use sync.Cond? - StackOverflow](https://stackoverflow.com/questions/36857167/how-to-correctly-use-sync-cond)

## type Locker

```
type Locker interface {
	Lock()
	Unlock()
}
```

Locker 表示可以锁定和解锁的对象。

## type Map

[参考地址](https://juejin.cn/post/6844903895227957262)

```go
type Map struct {
	// contains filtered or unexported fields
}
```

map在并发情况虚啊，只读是线程安全的，同时写线程不安全，所以为了**并发安全 & 高效**

### sync.Map源码实现

先白话文说下大概逻辑。让下文看的更快。（大概只有是这样流程就好） **写：直写。 读：先读read，没有再读dirty。**

![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/7/23/16c1d7f700587ced~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)



从“基础结构 + 增删改查”的思路来详细过一遍源码。

#### 基础结构

sync.Map的核心数据结构:

```go
type Map struct {
	mu Mutex
	read atomic.Value // readOnly
	dirty map[interface{}]*entry
	misses int
}
```

| 说明   | 类型                   | 作用                                                         |
| ------ | ---------------------- | ------------------------------------------------------------ |
| mu     | Mutex                  | 加锁作用。保护后文的dirty字段                                |
| read   | atomic.Value           | 存读的数据。因为是atomic.Value类型，只读，所以并发是安全的。实际存的是readOnly的数据结构。 |
| misses | int                    | 计数作用。每次从read中读失败，则计数+1。                     |
| dirty  | map[interface{}]*entry | 包含最新写入的数据。当misses计数达到一定值，将其赋值给read。 |

这里有必要简单描述一下，大概的逻辑，

readOnly的数据结构：

```
type readOnly struct {
    m  map[interface{}]*entry
    amended bool 
}
复制代码
```

| 说明    | 类型                   | 作用                                                   |
| ------- | ---------------------- | ------------------------------------------------------ |
| m       | map[interface{}]*entry | 单纯的map结构                                          |
| amended | bool                   | Map.dirty的数据和这里的 m 中的数据不一样的时候，为true |

entry的数据结构：

```go
type entry struct {
    //可见value是个指针类型，虽然read和dirty存在冗余情况（amended=false），但是由于是指针类型，存储的空间应该不是问题
    p unsafe.Pointer // *interface{}
}
```

这个结构体主要是想说明。虽然前文read和dirty存在冗余的情况，但是由于value都是指针类型，其实存储的空间其实没增加多少。

#### 查询

```go
func (m *Map) Load(key interface{}) (value interface{}, ok bool) {
    // 因read只读，线程安全，优先读取
    read, _ := m.read.Load().(readOnly)
    e, ok := read.m[key]
    
    // 如果read没有，并且dirty有新数据，那么去dirty中查找
    if !ok && read.amended {
        m.mu.Lock()
        // 双重检查（原因是前文的if判断和加锁非原子的，害怕这中间发生故事）
        read, _ = m.read.Load().(readOnly)
        e, ok = read.m[key]
        
        // 如果read中还是不存在，并且dirty中有新数据
        if !ok && read.amended {
            e, ok = m.dirty[key]
            // m计数+1
            m.missLocked()
        }
        
        m.mu.Unlock()
    }
    
    if !ok {
        return nil, false
    }
    return e.load()
}

func (m *Map) missLocked() {
    m.misses++
    if m.misses < len(m.dirty) {
        return
    }
    
    // 将dirty置给read，因为穿透概率太大了(原子操作，耗时很小)
    m.read.Store(readOnly{m: m.dirty})
    m.dirty = nil
    m.misses = 0
}
```

流程图：

![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/7/23/16c1d7f70079a2b2~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

这边有几个点需要强调一下：



> 如何设置阀值？

这里采用**miss计数和dirty长度**的比较，来进行阀值的设定。

> 为什么dirty可以直接换到read？

因为**写操作只会操作dirty**，所以保证了dirty是最新的，并且**数据集是肯定包含read的**。 （可能有同学疑问，dirty不是下一步就置为nil了，为何还包含？后文会有解释。）

> 为什么dirty置为nil？

我不确定这个原因。猜测：一方面是当read完全等于dirty的时候，读的话read没有就是没有了，即使穿透也是一样的结果，所以存的没啥用。另一方是当存的时候，如果元素比较多，影响插入效率。

####  删

```
func (m *Map) Delete(key interface{}) {
    // 读出read，断言为readOnly类型
    read, _ := m.read.Load().(readOnly)
    e, ok := read.m[key]
    // 如果read中没有，并且dirty中有新元素，那么就去dirty中去找。这里用到了amended，当read与dirty不同时为true，说明dirty中有read没有的数据。
    
    if !ok && read.amended {
        m.mu.Lock()
        // 再检查一次，因为前文的判断和锁不是原子操作，防止期间发生了变化。
        read, _ = m.read.0Load().(readOnly)
        e, ok = read.m[key]
        
        if !ok && read.amended {
            // 直接删除
            delete(m.dirty, key)
        }
        m.mu.Unlock()
    }
    
    if ok {
    // 如果read中存=-99在该key，则将该value 赋值nil（采用标记的方式删除！）
        e.delete()
    }
}

func (e *entry) delete() (hadValue bool) {
    for {
    	// 再次再一把数据的指针
        p := atomic.LoadPointer(&e.p)
        if p == nil || p == expunged {
            return false
        }
        
        // 原子操作
        if atomic.CompareAndSwapPointer(&e.p, p, nil) {
            return true
        }
    }
}
```

流程图：

![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/7/23/16c1d7f700d4b21d~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)



这边有几个点需要强调一下：

> 1.为什么dirty是直接删除，而read是标记删除？

read的作用是在dirty前头优先度，遇到相同元素的时候为了不穿透到dirty，所以采用标记的方式。 同时正是因为这样的机制+amended的标记，可以保证read找不到&&amended=false的时候，dirty中肯定找不到

> 2.为什么dirty是可以直接删除，而没有先进行读取存在后删除？

删除成本低。读一次需要寻找，删除也需要寻找，无需重复操作。

> 3.如何进行标记的？

将值置为nil。（这个很关键）

#### 增(改)

```
func (m *Map) Store(key, value interface{}) {
    // 如果m.read存在这个key，并且没有被标记删除，则尝试更新。
    read, _ := m.read.Load().(readOnly)
    if e, ok := read.m[key]; ok && e.tryStore(&value) {
        return
    }
    
    // 如果read不存在或者已经被标记删除
    m.mu.Lock()
    read, _ = m.read.Load().(readOnly)
   
    if e, ok := read.m[key]; ok { // read 存在该key
    // 如果entry被标记expunge，则表明dirty没有key，可添加入dirty，并更新entry。
        if e.unexpungeLocked() { 
            // 加入dirty中，这儿是指针
            m.dirty[key] = e
        }
        // 更新value值
        e.storeLocked(&value) 
        
    } else if e, ok := m.dirty[key]; ok { // dirty 存在该key，更新
        e.storeLocked(&value)
        
    } else { // read 和 dirty都没有
        // 如果read与dirty相同，则触发一次dirty刷新（因为当read重置的时候，dirty已置为nil了）
        if !read.amended { 
            // 将read中未删除的数据加入到dirty中
            m.dirtyLocked() 
            // amended标记为read与dirty不相同，因为后面即将加入新数据。
            m.read.Store(readOnly{m: read.m, amended: true})
        }
        m.dirty[key] = newEntry(value) 
    }
    m.mu.Unlock()
}

// 将read中未删除的数据加入到dirty中
func (m *Map) dirtyLocked() {
    if m.dirty != nil {
        return
    }
    
    read, _ := m.read.Load().(readOnly)
    m.dirty = make(map[interface{}]*entry, len(read.m))
    
    // 遍历read。
    for k, e := range read.m {
        // 通过此次操作，dirty中的元素都是未被删除的，可见标记为expunged的元素不在dirty中！！！
        if !e.tryExpungeLocked() {
            m.dirty[k] = e
        }
    }
}

// 判断entry是否被标记删除，并且将标记为nil的entry更新标记为expunge
func (e *entry) tryExpungeLocked() (isExpunged bool) {
    p := atomic.LoadPointer(&e.p)
    
    for p == nil {
        // 将已经删除标记为nil的数据标记为expunged
        if atomic.CompareAndSwapPointer(&e.p, nil, expunged) {
            return true
        }
        p = atomic.LoadPointer(&e.p)
    }
    return p == expunged
}

// 对entry尝试更新 （原子cas操作）
func (e *entry) tryStore(i *interface{}) bool {
    p := atomic.LoadPointer(&e.p)
    if p == expunged {
        return false
    }
    for {
        if atomic.CompareAndSwapPointer(&e.p, p, unsafe.Pointer(i)) {
            return true
        }
        p = atomic.LoadPointer(&e.p)
        if p == expunged {
            return false
        }
    }
}

// read里 将标记为expunge的更新为nil
func (e *entry) unexpungeLocked() (wasExpunged bool) {
    return atomic.CompareAndSwapPointer(&e.p, expunged, nil)
}

// 更新entry
func (e *entry) storeLocked(i *interface{}) {
    atomic.StorePointer(&e.p, unsafe.Pointer(i))
}
```

流程图：

![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/7/23/16c1d7f700ee3129~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

这边有几个点需要强调一下：



> 1. read中的标记为已删除的区别？

标记为nil，说明是正常的delete操作，此时dirty中不一定存在 a. dirty赋值给read后，此时dirty不存在 b. dirty初始化后，肯定存在

标记为expunged，说明是在dirty初始化的时候操作的，此时dirty中肯定不存在。

> 1. 可能存在性能问题？

初始化dirty的时候，虽然都是指针赋值，但read如果较大的话，可能会有些影响。

### sync.Map的优缺点？

先说结论，后来证明。

> 优点：是官方出的，是亲儿子；通过读写分离，降低锁时间来提高效率； 缺点：不适用于大量写的场景，这样会导致read map读不到数据而进一步加锁读取，同时dirty map也会一直晋升为read map，整体性能较差。 适用场景：大量读，少量写

这里主要证明一下，为什么适合**大量读，少量写**。 代码的大概思路：通过比较单纯的map和sync.Map，在并发安全的情况下，只写和读写的效率

```
var s sync.RWMutex
var w sync.WaitGroup
func main() {
	mapTest()
	syncMapTest()
}
func mapTest() {
	m := map[int]int {1:1}
	startTime := time.Now().Nanosecond()
	w.Add(1)
	go writeMap(m)
	w.Add(1)
	go writeMap(m)
	//w.Add(1)
	//go readMap(m)

	w.Wait()
	endTime := time.Now().Nanosecond()
	timeDiff := endTime-startTime
	fmt.Println("map:",timeDiff)
}

func writeMap (m map[int]int) {
	defer w.Done()
	i := 0
	for i < 10000 {
		// 加锁
		s.Lock()
		m[1]=1
		// 解锁
		s.Unlock()
		i++
	}
}

func readMap (m map[int]int) {
	defer w.Done()
	i := 0
	for i < 10000 {
		s.RLock()
		_ = m[1]
		s.RUnlock()
		i++
	}
}

func syncMapTest() {
	m := sync.Map{}
	m.Store(1,1)
	startTime := time.Now().Nanosecond()
	w.Add(1)
	go writeSyncMap(m)
	w.Add(1)
	go writeSyncMap(m)
	//w.Add(1)
	//go readSyncMap(m)

	w.Wait()
	endTime := time.Now().Nanosecond()
	timeDiff := endTime-startTime
	fmt.Println("sync.Map:",timeDiff)
}

func writeSyncMap (m sync.Map) {
	defer w.Done()
	i := 0
	for i < 10000 {
		m.Store(1,1)
		i++
	}
}

func readSyncMap (m sync.Map) {
	defer w.Done()
	i := 0
	for i < 10000 {
		m.Load(1)
		i++
	}
}
复制代码
```

| 情况 | 结果                                   |
| ---- | -------------------------------------- |
| 只写 | map: 1,022,000    sync.Map: 2,164,000  |
| 读写 | map: 8,696,000     sync.Map: 2,047,000 |

会发现大量写的场景下，由于sync.Map里头操作更多其实，所以效率没有单纯的map+metux高。

### Delete

```
func (m *Map) Delete(key any)
```

Delete 删除键的值

### Load 

```
func (m *Map) Load(key any) (value any, ok bool)
```

Load 返回存储在映射中的键的值，如果不存在值，则返回 nil。 ok 结果表明是否在映射中找到值。

### LoadAndDelete

```go
func (m *Map) LoadAndDelete(key any) (value any, loaded bool)
```

删除键的值，如果有则返回前一个值。加载的结果报告key是否存在。

### LoadOrStore

```go
func (m *Map) LoadOrStore(key, value any) (actual any, loaded bool)
```

LoadOrStore 返回键的现有值（如果存在）。否则，它存储并返回给定的值。如果值已加载，则加载结果为 true，如果已存储，则为 false。

### [Range](https://www.cnblogs.com/qcrao-2018/p/12833787.html#range)

```
func (m *Map) Range(f func(key, value any) bool)
```

Range 的参数是一个函数：

```golang
f func(key, value interface{}) bool
```

由使用者提供实现，Range 将遍历调用时刻 map 中的所有 k-v 对，将它们传给 f 函数，如果 f 返回 false，将停止遍历。

### Store

```
func (m *Map) Store(key, value any)
```

存储设置键的值

Go 语言的 [`sync.Mutex`](https://draveness.me/golang/tree/sync.Mutex) 由两个字段 `state` 和 `sema` 组成。其中 `state` 表示当前互斥锁的状态，而 `sema` 是用于控制锁状态的信号量。

## type Mutex 

```
type Mutex struct {
	state int32
	sema  uint32
}
```

上述两个加起来只占 8 字节空间的结构体表示了 Go 语言中的互斥锁。

#### 状态

互斥锁的状态比较复杂，如下图所示，最低三位分别表示 `mutexLocked`、`mutexWoken` 和 `mutexStarving`，剩下的位置用来表示当前有多少个 Goroutine 在等待互斥锁的释放：

![golang-mutex-state](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/2020-01-23-15797104328010-golang-mutex-state.png)

在默认情况下，互斥锁的所有状态位都是 0，`int32` 中的不同位分别表示了不同的状态：

- `mutexLocked` — 表示互斥锁的锁定状态；
- `mutexWoken` — 表示从正常模式被从唤醒；
- `mutexStarving` — 当前的互斥锁进入饥饿状态；
- `waitersCount` — 当前互斥锁上等待的 Goroutine 个数；

#### 正常模式和饥饿模式

[`sync.Mutex`](https://draveness.me/golang/tree/sync.Mutex) 有两种模式 — 正常模式和饥饿模式。我们需要在这里先了解正常模式和饥饿模式都是什么以及它们有什么样的关系。

在**正常模式下，锁的等待者会按照先进先出的顺序获取锁**。但是刚被唤起的 Goroutine 与新创建的 Goroutine 竞争时，大概率会获取不到锁，为了减少这种情况的出现，**一旦 Goroutine 超过 1ms 没有获取到锁，它就会将当前互斥锁切换饥饿模式**，防止部分 Goroutine 被『饿死』。

![golang-mutex-mode](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/2020-01-23-15797104328020-golang-mutex-mode.png)

饥饿模式是在 Go 语言在 1.9 中通过提交 [sync: make Mutex more fair](https://github.com/golang/go/commit/0556e26273f704db73df9e7c4c3d2e8434dec7be) 引入的优化[1](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-sync-primitives/#fn:1)，引入的目的是保证互斥锁的公平性。

在**饥饿模式中，互斥锁会直接交给等待队列最前面的 Goroutine**。新的 Goroutine 在该状态下不能获取锁、也不会进入自旋状态，它们只会在队列的末尾等待。如果一个 Goroutine 获得了互斥锁并且它在队列的末尾或者它等待的时间少于 1ms，那么当前的互斥锁就会切换回正常模式。

与饥饿模式相比，正常模式下的互斥锁能够提供更好地性能，饥饿模式的能避免 Goroutine 由于陷入等待无法获取锁而造成的高尾延时。

#### 加锁和解锁 

互斥锁的加锁和解锁过程，它们分别使用 [`sync.Mutex.Lock`](https://draveness.me/golang/tree/sync.Mutex.Lock) 和 [`sync.Mutex.Unlock`](https://draveness.me/golang/tree/sync.Mutex.Unlock) 方法。

互斥锁的加锁是靠 [`sync.Mutex.Lock`](https://draveness.me/golang/tree/sync.Mutex.Lock) 完成的，最新的 Go 语言源代码中已经将 [`sync.Mutex.Lock`](https://draveness.me/golang/tree/sync.Mutex.Lock) 方法进行了简化，方法的主干只保留最常见、简单的情况 — 当锁的状态是 0 时，将 `mutexLocked` 位置成 1：

```go
func (m *Mutex) Lock() {
	if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
		return
	}
	m.lockSlow()
}
```

如果**互斥锁的状态不是 0 时就会调用 [`sync.Mutex.lockSlow`](https://draveness.me/golang/tree/sync.Mutex.lockSlow) 尝试通过自旋（Spinnig）等方式等待锁的释放**，该方法的主体是一个非常大 for 循环，这里将它分成几个部分介绍获取锁的过程：

1. 判断当前 Goroutine 能否进入自旋；
2. 通过自旋等待互斥锁的释放；
3. 计算互斥锁的最新状态；
4. 更新互斥锁的状态并获取锁；

我们先来介绍互斥锁是如何判断当前 Goroutine 能否进入自旋等互斥锁的释放：

```go
func (m *Mutex) lockSlow() {
	var waitStartTime int64
	starving := false
	awoke := false
	iter := 0
	old := m.state
	for {
		if old&(mutexLocked|mutexStarving) == mutexLocked && runtime_canSpin(iter) {
			if !awoke && old&mutexWoken == 0 && old>>mutexWaiterShift != 0 &&
				atomic.CompareAndSwapInt32(&m.state, old, old|mutexWoken) {
				awoke = true
			}
			runtime_doSpin()
			iter++
			old = m.state
			continue
		}
```

**自旋是一种多线程同步机制**，当前的进程在进入自旋的过程中会一直保持 CPU 的占用，持续检查某个条件是否为真。在多核的 CPU 上，**自旋可以避免 Goroutine 的切换**，使用恰当会对性能带来很大的增益，但是使用的不恰当就会拖慢整个程序，所以 Goroutine 进入自旋的条件非常苛刻：

1. **互斥锁只有在普通模式才能进入自旋**；
2. `runtime.sync_runtime_canSpin`需要返回true
   1. 运行在多 CPU 的机器上；
   2. 当前 Goroutine 为了获取该锁进入自旋的次数小于四次；
   3. 当前机器上至少存在一个正在运行的处理器 P 并且处理的运行队列为空；

一旦当前 Goroutine 能够进入自旋就会调用[`runtime.sync_runtime_doSpin`](https://draveness.me/golang/tree/runtime.sync_runtime_doSpin) 和 [`runtime.procyield`](https://draveness.me/golang/tree/runtime.procyield) 并执行 30 次的 `PAUSE` 指令，该指令只会占用 CPU 并消耗 CPU 时间：

```go
func sync_runtime_doSpin() {
	procyield(active_spin_cnt)
}

TEXT runtime·procyield(SB),NOSPLIT,$0-0
	MOVL	cycles+0(FP), AX
again:
	PAUSE
	SUBL	$1, AX
	JNZ	again
	RET
```

处理了自旋相关的特殊逻辑之后，互斥锁会根据上下文计算当前互斥锁最新的状态。几个不同的条件分别会更新 `state` 字段中存储的不同信息 — `mutexLocked`、`mutexStarving`、`mutexWoken` 和 `mutexWaiterShift`：

```go
    new := old
    if old&mutexStarving == 0 {
        new |= mutexLocked
    }
    if old&(mutexLocked|mutexStarving) != 0 {
        new += 1 << mutexWaiterShift
    }
    if starving && old&mutexLocked != 0 {
        new |= mutexStarving
    }
    if awoke {
        new &^= mutexWoken
    }
```

计算了新的互斥锁状态之后，会使用 CAS 函数 [`sync/atomic.CompareAndSwapInt32`](https://draveness.me/golang/tree/sync/atomic.CompareAndSwapInt32) 更新状态：

```go
		if atomic.CompareAndSwapInt32(&m.state, old, new) {
			if old&(mutexLocked|mutexStarving) == 0 {
				break // 通过 CAS 函数获取了锁
			}
			...
			runtime_SemacquireMutex(&m.sema, queueLifo, 1)
			starving = starving || runtime_nanotime()-waitStartTime > starvationThresholdNs
			old = m.state
			if old&mutexStarving != 0 {
				delta := int32(mutexLocked - 1<<mutexWaiterShift)
				if !starving || old>>mutexWaiterShift == 1 {
					delta -= mutexStarving
				}
				atomic.AddInt32(&m.state, delta)
				break
			}
			awoke = true
			iter = 0
		} else {
			old = m.state
		}
	}
}
```

如果没有通过 CAS 获得锁，会调用 [`runtime.sync_runtime_SemacquireMutex`](https://draveness.me/golang/tree/runtime.sync_runtime_SemacquireMutex) 通过信号量保证资源不会被两个 Goroutine 获取。[`runtime.sync_runtime_SemacquireMutex`](https://draveness.me/golang/tree/runtime.sync_runtime_SemacquireMutex) 会在方法中不断尝试获取锁并陷入休眠等待信号量的释放，一旦当前 Goroutine 可以获取信号量，它就会立刻返回，[`sync.Mutex.Lock`](https://draveness.me/golang/tree/sync.Mutex.Lock) 的剩余代码也会继续执行。

- 在正常模式下，这段代码会设置唤醒和饥饿标记、重置迭代次数并重新执行获取锁的循环；
- 在饥饿模式下，当前 Goroutine 会获得互斥锁，如果等待队列中只存在当前 Goroutine，互斥锁还会从饥饿模式中退出；

互斥锁的解锁过程 [`sync.Mutex.Unlock`](https://draveness.me/golang/tree/sync.Mutex.Unlock) 与加锁过程相比就很简单，该过程会先使用 [`sync/atomic.AddInt32`](https://draveness.me/golang/tree/sync/atomic.AddInt32) 函数快速解锁，这时会发生下面的两种情况：

- 如果该函数返回的新状态等于 0，当前 Goroutine 就成功解锁了互斥锁；
- 如果该函数返回的新状态不等于 0，这段代码会调用 [`sync.Mutex.unlockSlow`](https://draveness.me/golang/tree/sync.Mutex.unlockSlow) 开始慢速解锁：

```go
func (m *Mutex) Unlock() {
	new := atomic.AddInt32(&m.state, -mutexLocked)
	if new != 0 {
		m.unlockSlow(new)
	}
}
```

[`sync.Mutex.unlockSlow`](https://draveness.me/golang/tree/sync.Mutex.unlockSlow) 会先校验锁状态的合法性 — 如果当前互斥锁已经被解锁过了会直接抛出异常 “sync: unlock of unlocked mutex” 中止当前程序。

在正常情况下会根据当前互斥锁的状态，分别处理正常模式和饥饿模式下的互斥锁：

```go
func (m *Mutex) unlockSlow(new int32) {
	if (new+mutexLocked)&mutexLocked == 0 {
		throw("sync: unlock of unlocked mutex")
	}
	if new&mutexStarving == 0 { // 正常模式
		old := new
		for {
			if old>>mutexWaiterShift == 0 || old&(mutexLocked|mutexWoken|mutexStarving) != 0 {
				return
			}
			new = (old - 1<<mutexWaiterShift) | mutexWoken
			if atomic.CompareAndSwapInt32(&m.state, old, new) {
				runtime_Semrelease(&m.sema, false, 1)
				return
			}
			old = m.state
		}
	} else { // 饥饿模式
		runtime_Semrelease(&m.sema, true, 1)
	}
}
```

- 在正常模式下，上述代码会使用如下所示的处理过程：
  - 如果互斥锁不存在等待者或者互斥锁的 `mutexLocked`、`mutexStarving`、`mutexWoken` 状态不都为 0，那么当前方法可以直接返回，不需要唤醒其他等待者；
  - 如果互斥锁存在等待者，会通过 [`sync.runtime_Semrelease`](https://draveness.me/golang/tree/sync.runtime_Semrelease) 唤醒等待者并移交锁的所有权；
- 在饥饿模式下，上述代码会直接调用 [`sync.runtime_Semrelease`](https://draveness.me/golang/tree/sync.runtime_Semrelease) 将当前锁交给下一个正在尝试获取锁的等待者，等待者被唤醒后会得到锁，在这时互斥锁还不会退出饥饿状态；

#### 小结

我们已经从多个方面分析了互斥锁 [`sync.Mutex`](https://draveness.me/golang/tree/sync.Mutex) 的实现原理，这里我们从加锁和解锁两个方面总结注意事项。

互斥锁的加锁过程比较复杂，它涉及自旋、信号量以及调度等概念：

- 如果互斥锁处于初始化状态，会通过置位 `mutexLocked` 加锁；
- 如果互斥锁处于 `mutexLocked` 状态并且在普通模式下工作，会进入自旋，执行 30 次 `PAUSE` 指令消耗 CPU 时间等待锁的释放；
- 如果当前 Goroutine 等待锁的时间超过了 1ms，互斥锁就会切换到饥饿模式；
- 互斥锁在正常情况下会通过 [`runtime.sync_runtime_SemacquireMutex`](https://draveness.me/golang/tree/runtime.sync_runtime_SemacquireMutex) 将尝试获取锁的 Goroutine 切换至休眠状态，等待锁的持有者唤醒；
- 如果当前 Goroutine 是互斥锁上的最后一个等待的协程或者等待的时间小于 1ms，那么它会将互斥锁切换回正常模式；

互斥锁的解锁过程与之相比就比较简单，其代码行数不多、逻辑清晰，也比较容易理解：

- 当互斥锁已经被解锁时，调用 [`sync.Mutex.Unlock`](https://draveness.me/golang/tree/sync.Mutex.Unlock) 会直接抛出异常；
- 当互斥锁处于饥饿模式时，将锁的所有权交给队列中的下一个等待者，等待者会负责设置 `mutexLocked` 标志位；
- 当互斥锁处于普通模式时，如果没有 Goroutine 等待锁的释放或者已经有被唤醒的 Goroutine 获得了锁，会直接返回；在其他情况下会通过 [`sync.runtime_Semrelease`](https://draveness.me/golang/tree/sync.runtime_Semrelease) 唤醒对应的 Goroutine；

## type RWMutex

读写互斥锁 [`sync.RWMutex`](https://draveness.me/golang/tree/sync.RWMutex) 是细粒度的互斥锁，它**不限制资源的并发读，但是读写、写写操作无法并行执行**。

|      |  读  |  写  |
| :--: | :--: | :--: |
|  读  |  Y   |  N   |
|  写  |  N   |  N   |

常见服务的资源读写比例会非常高，因为大多数的读请求之间不会相互影响，所以我们可以分离读写操作，以此来提高服务的性能。

#### 结构体

[`sync.RWMutex`](https://draveness.me/golang/tree/sync.RWMutex) 中总共包含以下 5 个字段：

```go
type RWMutex struct {
	w           Mutex
	writerSem   uint32 // 写等待读
	readerSem   uint32 // 读等待写
	readerCount int32 // 当前正在执行的读操作数量
	readerWait  int32 // 写操作被阻塞时等待的读操作个数
}
```

- `w` — 复用互斥锁提供的能力；
- `writerSem` 和 `readerSem` — 分别用于**写等待读**和**读等待写**：
- `readerCount` 存储了当前正在执行的读操作数量；
- `readerWait` 表示当**写操作被阻塞时等待的读操作个数**；

我们会依次分析获取写锁和读锁的实现原理，其中：

- 写操作使用 [`sync.RWMutex.Lock`](https://draveness.me/golang/tree/sync.RWMutex.Lock) 和 [`sync.RWMutex.Unlock`](https://draveness.me/golang/tree/sync.RWMutex.Unlock) 方法；
- 读操作使用 [`sync.RWMutex.RLock`](https://draveness.me/golang/tree/sync.RWMutex.RLock) 和 [`sync.RWMutex.RUnlock`](https://draveness.me/golang/tree/sync.RWMutex.RUnlock) 方法；

#### 写锁

当资源的使用者想要获取写锁时，需要调用 [`sync.RWMutex.Lock`](https://draveness.me/golang/tree/sync.RWMutex.Lock) 方法：

```go
func (rw *RWMutex) Lock() {
	rw.w.Lock()
	r := atomic.AddInt32(&rw.readerCount, -rwmutexMaxReaders) + rwmutexMaxReaders
	if r != 0 && atomic.AddInt32(&rw.readerWait, r) != 0 {
		runtime_SemacquireMutex(&rw.writerSem, false, 0)
	}
}
```

1. 调用结构体持有的`sync.Mutex`结构体的`sync.Mutex.Lock`阻塞后续的写操作；
   - 因为互斥锁已经被获取，其他 Goroutine 在获取写锁时会进入自旋或者休眠；
2. 调用 [`sync/atomic.AddInt32`](https://draveness.me/golang/tree/sync/atomic.AddInt32) 函数阻塞后续的读操作：
3. 如果仍然有其他 Goroutine 持有互斥锁的读锁，该 Goroutine 会调用 [`runtime.sync_runtime_SemacquireMutex`](https://draveness.me/golang/tree/runtime.sync_runtime_SemacquireMutex) 进入休眠状态等待所有读锁所有者执行结束后释放 `writerSem` 信号量将当前协程唤醒；

写锁的释放会调用 [`sync.RWMutex.Unlock`](https://draveness.me/golang/tree/sync.RWMutex.Unlock)：

```go
func (rw *RWMutex) Unlock() {
	r := atomic.AddInt32(&rw.readerCount, rwmutexMaxReaders)
	if r >= rwmutexMaxReaders {
		throw("sync: Unlock of unlocked RWMutex")
	}
	for i := 0; i < int(r); i++ {
		runtime_Semrelease(&rw.readerSem, false, 0)
	}
	rw.w.Unlock()
}
```

与加锁的过程正好相反，写锁的释放分以下几个执行：

1. 调用 [`sync/atomic.AddInt32`](https://draveness.me/golang/tree/sync/atomic.AddInt32) 函数将 `readerCount` 变回正数，释放读锁；
2. 通过 for 循环释放所有因为获取读锁而陷入等待的 Goroutine：
3. 调用 [`sync.Mutex.Unlock`](https://draveness.me/golang/tree/sync.Mutex.Unlock) 释放写锁；

获取写锁时会先阻塞写锁的获取，后阻塞读锁的获取，这种策略能够保证读操作不会被连续的写操作『饿死』。

#### 读锁

读锁的加锁方法 [`sync.RWMutex.RLock`](https://draveness.me/golang/tree/sync.RWMutex.RLock) 很简单，该方法会通过 [`sync/atomic.AddInt32`](https://draveness.me/golang/tree/sync/atomic.AddInt32) 将 `readerCount` 加一：

```go
func (rw *RWMutex) RLock() {
	if atomic.AddInt32(&rw.readerCount, 1) < 0 {
		runtime_SemacquireMutex(&rw.readerSem, false, 0)
	}
}
```

1. 如果该方法返回负数 — 其他 Goroutine 获得了写锁，当前 Goroutine 就会调用 [`runtime.sync_runtime_SemacquireMutex`](https://draveness.me/golang/tree/runtime.sync_runtime_SemacquireMutex) 陷入休眠等待锁的释放；
2. 如果该方法的结果为非负数 — 没有 Goroutine 获得写锁，当前方法会成功返回；

当 Goroutine 想要释放读锁时，会调用如下所示的 [`sync.RWMutex.RUnlock`](https://draveness.me/golang/tree/sync.RWMutex.RUnlock) 方法：

```go
func (rw *RWMutex) RUnlock() {
	if r := atomic.AddInt32(&rw.readerCount, -1); r < 0 {
		rw.rUnlockSlow(r)
	}
}
```

该方法会先减少正在读资源的 `readerCount` 整数，根据 [`sync/atomic.AddInt32`](https://draveness.me/golang/tree/sync/atomic.AddInt32) 的返回值不同会分别进行处理：

- 如果返回值大于等于零 — 读锁直接解锁成功；
- 如果返回值小于零 — 有一个正在执行的写操作，在这时会调用[`sync.RWMutex.rUnlockSlow`](https://draveness.me/golang/tree/sync.RWMutex.rUnlockSlow) 方法；

```go
func (rw *RWMutex) rUnlockSlow(r int32) {
	if r+1 == 0 || r+1 == -rwmutexMaxReaders {
		throw("sync: RUnlock of unlocked RWMutex")
	}
	if atomic.AddInt32(&rw.readerWait, -1) == 0 {
		runtime_Semrelease(&rw.writerSem, false, 1)
	}
}
```

[`sync.RWMutex.rUnlockSlow`](https://draveness.me/golang/tree/sync.RWMutex.rUnlockSlow) 会减少获取锁的写操作等待的读操作数 `readerWait` 并在所有读操作都被释放之后触发写操作的信号量 `writerSem`，该信号量被触发时，调度器就会唤醒尝试获取写锁的 Goroutine。

#### 小结

虽然读写互斥锁 [`sync.RWMutex`](https://draveness.me/golang/tree/sync.RWMutex) 提供的功能比较复杂，但是因为它建立在 [`sync.Mutex`](https://draveness.me/golang/tree/sync.Mutex) 上，所以实现会简单很多。我们总结一下读锁和写锁的关系：

- 调用`sync.RWMutex.Lock`尝试获取写锁时；
  - 每次 [`sync.RWMutex.RUnlock`](https://draveness.me/golang/tree/sync.RWMutex.RUnlock) 都会将 `readerCount` 其减一，当它归零时该 Goroutine 会获得写锁；
  - 将 `readerCount` 减少 `rwmutexMaxReaders` 个数以阻塞后续的读操作；
- 调用 [`sync.RWMutex.Unlock`](https://draveness.me/golang/tree/sync.RWMutex.Unlock) 释放写锁时，会先通知所有的读操作，然后才会释放持有的互斥锁；

读写互斥锁在互斥锁之上提供了额外的更细粒度的控制，能够在读操作远远多于写操作时提升性能。

## type Once

Go 语言标准库中 [`sync.Once`](https://draveness.me/golang/tree/sync.Once) 可以保证在 Go 程序运行期间的某段代码只会执行一次。在运行如下所示的代码时，我们会看到如下所示的运行结果：

```go
func main() {
    o := &sync.Once{}
    for i := 0; i < 10; i++ {
        o.Do(func() {
            fmt.Println("only once")
        })
    }
}

$ go run main.go
only once
```

#### 结构体

每一个 [`sync.Once`](https://draveness.me/golang/tree/sync.Once) 结构体中都只包含一个用于标识代码块是否执行过的 `done` 以及一个互斥锁 [`sync.Mutex`](https://draveness.me/golang/tree/sync.Mutex)：

```go
type Once struct {
	done uint32
	m    Mutex
}
```

#### done 为什么是第一个字段

字段 `done` 的注释也非常值得一看：

```go
type Once struct {
    // done indicates whether the action has been performed.
    // It is first in the struct because it is used in the hot path.
    // The hot path is inlined at every call site.
    // Placing done first allows more compact instructions on some architectures (amd64/x86),
    // and fewer instructions (to calculate offset) on other architectures.
    done uint32
    m    Mutex
}
```

其中解释了为什么将 done 置为 Once 的第一个字段：done 在热路径中，**done 放在第一个字段，能够减少 CPU 指令**，也就是说，这样做能够提升性能。

简单解释下这句话：

1. 热路径(hot path)是程序非常频繁执行的一系列指令，sync.Once 绝大部分场景都会访问 `o.done`，在热路径上是比较好理解的，如果 hot path 编译后的机器码指令更少，更直接，必然是能够提升性能的。
2. 为什么放在第一个字段就能够减少指令呢？因为**结构体第一个字段的地址和结构体的指针是相同的**，如果是第一个字段，直接对结构体的指针解引用即可。如果是其他的字段，除了结构体指针外，还需要计算与第一个值的偏移(calculate offset)。在机器码中，偏移量是随指令传递的附加值，CPU 需要做一次偏移值与指针的加法运算，才能获取要访问的值的地址。因为，访问第一个字段的机器代码更紧凑，速度更快。

#### 接口

[`sync.Once.Do`](https://draveness.me/golang/tree/sync.Once.Do) 是 [`sync.Once`](https://draveness.me/golang/tree/sync.Once) 结构体对外唯一暴露的方法，该方法会接收一个入参为空的函数：

- 如果传入的函数已经执行过，会直接返回；
- 如果传入的函数没有执行过，会调用 [`sync.Once.doSlow`](https://draveness.me/golang/tree/sync.Once.doSlow) 执行传入的函数：

```go
func (o *Once) Do(f func()) {
	if atomic.LoadUint32(&o.done) == 0 {
		o.doSlow(f)
	}
}

func (o *Once) doSlow(f func()) {
	o.m.Lock()
	defer o.m.Unlock()
	if o.done == 0 {
		defer atomic.StoreUint32(&o.done, 1)
		f()
	}
}
```

1. 为当前 Goroutine 获取互斥锁；
2. 执行传入的无入参函数；
3. 运行延迟函数调用，将成员变量 `done` 更新成 1；

[`sync.Once`](https://draveness.me/golang/tree/sync.Once) 会通过成员变量 `done` 确保函数不会执行第二次。

#### 小结

作为用于保证函数执行次数的 [`sync.Once`](https://draveness.me/golang/tree/sync.Once) 结构体，它使用互斥锁和 [`sync/atomic`](https://draveness.me/golang/tree/sync/atomic) 包提供的方法实现了某个函数在程序运行期间只能执行一次的语义。在使用该结构体时，我们也需要注意以下的问题：

- [`sync.Once.Do`](https://draveness.me/golang/tree/sync.Once.Do) 方法中传入的函数只会被执行一次，哪怕函数中发生了 `panic`；
- 两次调用 [`sync.Once.Do`](https://draveness.me/golang/tree/sync.Once.Do) 方法传入不同的函数只会执行第一次调传入的函数；

## type Pool

[参考地址](https://geektutu.com/post/hpg-sync-pool.html)

## sync.Pool 的使用场景

> 一句话总结：保存和复用临时对象，减少内存分配，降低 GC 压力。

举个简单的例子：

```
type Student struct {
	Name   string
	Age    int32
	Remark [1024]byte
}

var buf, _ = json.Marshal(Student{Name: "Geektutu", Age: 25})

func unmarsh() {
	stu := &Student{}
	json.Unmarshal(buf, stu)
}
```

json 的反序列化在文本解析和网络通信过程中非常常见，当程序并发度非常高的情况下，短时间内需要创建大量的临时对象。而这些对象是都是分配在堆上的，会给 GC 造成很大压力，严重影响程序的性能。

> 参考：[垃圾回收(GC)的工作原理](https://geektutu.com/post/qa-golang-2.html#Q5-简述-Go-语言GC-垃圾回收-的工作原理)

Go 语言从 1.3 版本开始提供了对象重用的机制，即 sync.Pool。**sync.Pool 是可伸缩的，同时也是并发安全的，其大小仅受限于内存的大小**。sync.Pool 用于存储那些被分配了但是没有被使用，而未来可能会使用的值。这样就可以不用再次经过内存分配，可直接复用已有对象，减轻 GC 的压力，从而提升系统的性能。

sync.Pool 的大小是可伸缩的，高负载时会动态扩容，存放在池中的对象如果不活跃了会被自动清理。

###  如何使用

sync.Pool 的使用方式非常简单：

### 声明对象池

只需要实现 New 函数即可。对象池中没有对象时，将会调用 New 函数创建。

```
var studentPool = sync.Pool{
    New: func() interface{} { 
        return new(Student) 
    },
}
```

###  Get & Put

```
stu := studentPool.Get().(*Student)
json.Unmarshal(buf, stu)
studentPool.Put(stu)
```

- `Get()` 用于从对象池中获取对象，因为返回值是 `interface{}`，因此需要类型转换。
- `Put()` 则是在对象使用完毕后，返回对象池。

### 性能测试

####  struct 反序列化

```
func BenchmarkUnmarshal(b *testing.B) {
	for n := 0; n < b.N; n++ {
		stu := &Student{}
		json.Unmarshal(buf, stu)
	}
}

func BenchmarkUnmarshalWithPool(b *testing.B) {
	for n := 0; n < b.N; n++ {
		stu := studentPool.Get().(*Student)
		json.Unmarshal(buf, stu)
		studentPool.Put(stu)
	}
}
```

测试结果如下：

```
$ go test -bench . -benchmem
goos: darwin
goarch: amd64
pkg: example/hpg-sync-pool
BenchmarkUnmarshal-8           1993   559768 ns/op   5096 B/op 7 allocs/op
BenchmarkUnmarshalWithPool-8   1976   550223 ns/op    234 B/op 6 allocs/op
PASS
ok      example/hpg-sync-pool   2.334s
```

在这个例子中，因为 Student 结构体内存占用较小，内存分配几乎不耗时间。而标准库 json 反序列化时利用了反射，效率是比较低的，占据了大部分时间，因此两种方式最终的执行时间几乎没什么变化。但是内存占用差了一个数量级，使用了 `sync.Pool` 后，内存占用仅为未使用的 234/5096 = 1/22，对 GC 的影响就很大了。

##### 3.2 bytes.Buffer

```
var bufferPool = sync.Pool{
	New: func() interface{} {
		return &bytes.Buffer{}
	},
}

var data = make([]byte, 10000)

func BenchmarkBufferWithPool(b *testing.B) {
	for n := 0; n < b.N; n++ {
		buf := bufferPool.Get().(*bytes.Buffer)
		buf.Write(data)
		buf.Reset()
		bufferPool.Put(buf)
	}
}

func BenchmarkBuffer(b *testing.B) {
	for n := 0; n < b.N; n++ {
		var buf bytes.Buffer
		buf.Write(data)
	}
}
```

测试结果如下：

```
BenchmarkBufferWithPool-8    8778160    133 ns/op       0 B/op   0 allocs/op
BenchmarkBuffer-8             906572   1299 ns/op   10240 B/op   1 allocs/op
```

这个例子创建了一个 `bytes.Buffer` 对象池，而且每次只执行一个简单的 `Write` 操作，存粹的内存搬运工，耗时几乎可以忽略。而内存分配和回收的耗时占比较多，因此对程序整体的性能影响更大。

### 在标准库中的应用

#### fmt.Printf

Go 语言标准库也大量使用了 `sync.Pool`，例如 `fmt` 和 `encoding/json`。

以下是 `fmt.Printf` 的源代码(go/src/fmt/print.go)：

```
// go 1.13.6

// pp is used to store a printer's state and is reused with sync.Pool to avoid allocations.
type pp struct {
    buf buffer
    ...
}

var ppFree = sync.Pool{
	New: func() interface{} { return new(pp) },
}

// newPrinter allocates a new pp struct or grabs a cached one.
func newPrinter() *pp {
	p := ppFree.Get().(*pp)
	p.panicking = false
	p.erroring = false
	p.wrapErrs = false
	p.fmt.init(&p.buf)
	return p
}

// free saves used pp structs in ppFree; avoids an allocation per invocation.
func (p *pp) free() {
	if cap(p.buf) > 64<<10 {
		return
	}

	p.buf = p.buf[:0]
	p.arg = nil
	p.value = reflect.Value{}
	p.wrappedErr = nil
	ppFree.Put(p)
}

func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error) {
	p := newPrinter()
	p.doPrintf(format, a)
	n, err = w.Write(p.buf)
	p.free()
	return
}

// Printf formats according to a format specifier and writes to standard output.
// It returns the number of bytes written and any write error encountered.
func Printf(format string, a ...interface{}) (n int, err error) {
	return Fprintf(os.Stdout, format, a...)
}
```

`fmt.Printf` 的调用是非常频繁的，利用 `sync.Pool` 复用 pp 对象能够极大地提升性能，减少内存占用，同时降低 GC 压力。

> 这个例子来源于：[深度解密 Go 语言之 sync.Pool](https://www.cnblogs.com/qcrao-2018/p/12736031.html)

## type WaitGroup

WaitGroup是Golang应用开发过程中经常使用的并发控制技术。

WaitGroup，可理解为Wait-Goroutine-Group，即等待一组goroutine结束。比如某个goroutine需要等待其他几个goroutine全部完成，那么使用WaitGroup可以轻松实现。

下面程序展示了一个goroutine等待另外两个goroutine结束的例子：

```
package mainimport (    "fmt"    "time"    "sync")func main() {    var wg sync.WaitGroup    wg.Add(2) //设置计数器，数值即为goroutine的个数    go func() {        //Do some work        time.Sleep(1*time.Second)        fmt.Println("Goroutine 1 finished!")        wg.Done() //goroutine执行结束后将计数器减1    }()    go func() {        //Do some work        time.Sleep(2*time.Second)        fmt.Println("Goroutine 2 finished!")        wg.Done() //goroutine执行结束后将计数器减1    }()    wg.Wait() //主goroutine阻塞等待计数器变为0    fmt.Printf("All Goroutine finished!")}
```

简单的说，上面程序中wg内部维护了一个计数器：

1. 启动goroutine前将计数器通过Add(2)将计数器设置为待启动的goroutine个数。
2. 启动goroutine后，使用Wait()方法阻塞自己，等待计数器变为0。
3. 每个goroutine执行结束通过Done()方法将计数器减1。
4. 计数器变为0后，阻塞的goroutine被唤醒。

其实WaitGroup也可以实现一组goroutine等待另一组goroutine，这有点像玩杂技，很容出错，如果不了解其实现原理更是如此。实际上，WaitGroup的实现源码非常简单。

### 基础知识

#### 信号量

信号量是Unix系统提供的一种保护共享资源的机制，用于防止多个线程同时访问某个资源。

可简单理解为信号量为一个数值：

- 当信号量>0时，表示资源可用，获取信号量时系统自动将信号量减1；
- 当信号量==0时，表示资源暂不可用，获取信号量时，当前线程会进入睡眠，当信号量为正时被唤醒；

由于WaitGroup实现中也使用了信号量，在此做个简单介绍。

####  数据结构

源码包中`src/sync/waitgroup.go:WaitGroup`定义了其数据结构：

```
type WaitGroup struct {    
	state1 [3]uint32
}
```

state1是个长度为3的数组，其中包含了state和一个信号量，而state实际上是两个计数器：

- counter： 当前还未执行结束的goroutine计数器
- waiter count: 等待goroutine-group结束的goroutine数量，即有多少个等候者
- semaphore: 信号量

考虑到字节是否对齐，三者出现的位置不同，为简单起见，依照字节已对齐情况下，三者在内存中的位置如下所示：

![5.2 WaitGroup - 图1](https://static.sitestack.cn/projects/GoExpertProgramming/chapter05/images/wg-01-layout.png)

WaitGroup对外提供三个接口：

- Add(delta int): 将delta值加到counter中
- Wait()： waiter递增1，并阻塞等待信号量semaphore
- Done()： counter递减1，按照waiter数值释放相应次数信号量

下面分别介绍这三个函数的实现细节。

#### Add(delta int)

Add()做了两件事，一是把delta值累加到counter中，因为delta可以为负值，也就是说counter有可能变成0或负值，所以第二件事就是当counter值变为0时，根据waiter数值释放等量的信号量，把等待的goroutine全部唤醒，如果counter变为负值，则panic.

Add()伪代码如下：

```
func (wg *WaitGroup) Add(delta int) {
    statep, semap := wg.state() //获取state和semaphore地址指针
    state := atomic.AddUint64(statep, uint64(delta)<<32) //把delta左移32位累加到state，即累加到counter中
    v := int32(state >> 32) //获取counter值
    w := uint32(state)      //获取waiter值
    if v < 0 {              //经过累加后counter值变为负值，panic
        panic("sync: negative WaitGroup counter")
    }
    //经过累加后，此时，counter >= 0
    //如果counter为正，说明不需要释放信号量，直接退出
    //如果waiter为零，说明没有等待者，也不需要释放信号量，直接退出
    if v > 0 || w == 0 {
        return
    }
    //此时，counter一定等于0，而waiter一定大于0（内部维护waiter，不会出现小于0的情况），
    //先把counter置为0，再释放waiter个数的信号量
    *statep = 0
    for ; w != 0; w-- {
        runtime_Semrelease(semap, false) //释放信号量，执行一次释放一个，唤醒一个等待者
    }
}
```

#### 3.3 Wait()

Wait()方法也做了两件事，一是累加waiter, 二是阻塞等待信号量

```
func (wg *WaitGroup) Wait() {
    statep, semap := wg.state() //获取state和semaphore地址指针
    for {
        state := atomic.LoadUint64(statep) //获取state值
        v := int32(state >> 32)            //获取counter值
        w := uint32(state)                 //获取waiter值
        if v == 0 {                        //如果counter值为0，说明所有goroutine都退出了，不需要待待，直接返回
            return
        }
        // 使用CAS（比较交换算法）累加waiter，累加可能会失败，失败后通过for loop下次重试
        if atomic.CompareAndSwapUint64(statep, state, state+1) {
            runtime_Semacquire(semap) //累加成功后，等待信号量唤醒自己
            return
        }
    }
}
```

这里用到了CAS算法保证有多个goroutine同时执行Wait()时也能正确累加waiter。

#### 3.4 Done()

Done()只做一件事，即把counter减1，我们知道Add()可以接受负值，所以Done实际上只是调用了Add(-1)。

源码如下：

```
func (wg *WaitGroup) Done() {    wg.Add(-1)}
```

Done()的执行逻辑就转到了Add()，实际上也正是最后一个完成的goroutine把等待者唤醒的。

### 4 编程Tips

- Add()操作必须早于Wait(), 否则会panic
- Add()设置的值必须与实际等待的goroutine个数一致，否则会panic