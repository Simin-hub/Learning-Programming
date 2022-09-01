# 上下文 Context

[参考地址](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-context/)

**上下文 [`context.Context`](https://draveness.me/golang/tree/context.Context) Go 语言中用来设置截止日期、同步信号，传递请求相关值的结构体**。上下文与 Goroutine 有比较密切的关系，是 Go 语言中独特的设计，在其他编程语言中我们很少见到类似的概念。

[`context.Context`](https://draveness.me/golang/tree/context.Context) 是 Go 语言在 1.7 版本中引入标准库的接口[1](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-context/#fn:1)，该接口定义了四个需要实现的方法，其中包括：

1. `Deadline` — 返回 [`context.Context`](https://draveness.me/golang/tree/context.Context) 被取消的时间，也就是完成工作的截止日期；如果没有设定期限，将返回`ok == false`。
2. `Done` — 返回一个 Channel，这个 Channel 会在当前工作完成或者上下文被取消后关闭，多次调用 `Done` 方法会返回同一个 Channel；
3. `Err`— 返回`context.Context`结束的原因，它只会在`Done`方法对应的 Channel 关闭时返回非空的值；
   1. 如果 [`context.Context`](https://draveness.me/golang/tree/context.Context) 被取消，会返回 `Canceled` 错误；
   2. 如果 [`context.Context`](https://draveness.me/golang/tree/context.Context) 超时，会返回 `DeadlineExceeded` 错误；
4. `Value` — 从 [`context.Context`](https://draveness.me/golang/tree/context.Context) 中获取键对应的值，对于同一个上下文来说，多次调用 `Value` 并传入相同的 `Key` 会返回相同的结果，该方法可以用来传递请求特定的数据；

```
type Context interface {
	Deadline() (deadline time.Time, ok bool)
	Done() <-chan struct{}
	Err() error
	Value(key interface{}) interface{}
}
```

[`context`](https://github.com/golang/go/tree/master/src/context) 包中提供的 [`context.Background`](https://draveness.me/golang/tree/context.Background)、[`context.TODO`](https://draveness.me/golang/tree/context.TODO)、[`context.WithDeadline`](https://draveness.me/golang/tree/context.WithDeadline) 和 [`context.WithValue`](https://draveness.me/golang/tree/context.WithValue) 函数会返回实现该接口的私有结构体.

## 设计原理

**在 Goroutine 构成的树形结构中对信号进行同步以减少计算资源的浪费是 [`context.Context`](https://draveness.me/golang/tree/context.Context) 的最大作用**。Go 服务的每一个请求都是通过单独的 Goroutine 处理的[2](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-context/#fn:2)，HTTP/RPC 请求的处理器会启动新的 Goroutine 访问数据库和其他服务。

如下图所示，我们可能会创建多个 Goroutine 来处理一次请求，而 [`context.Context`](https://draveness.me/golang/tree/context.Context) 的作用是在不同 Goroutine 之间同步请求特定数据、取消信号以及处理请求的截止日期。

[![golang-context-usage](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/golang-context-usage.png)](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/golang-context-usage.png)

如上图所示，当最上层的 Goroutine 因为某些原因执行失败时，下层的 Goroutine 由于没有接收到这个信号所以会继续工作；但是当我们正确地使用 [`context.Context`](https://draveness.me/golang/tree/context.Context) 时，就可以在下层及时停掉无用的工作以减少额外资源的消耗：

[![golang-with-context](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/golang-with-context.png)](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/golang-with-context.png)

我们可以通过一个代码片段了解 [`context.Context`](https://draveness.me/golang/tree/context.Context) 是如何对信号进行同步的。在这段代码中，我们创建了一个过期时间为 1s 的上下文，并向上下文传入 `handle` 函数，该方法会使用 500ms 的时间处理传入的请求：

```
func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
	defer cancel()

	go handle(ctx, 500*time.Millisecond)
	select {
	case <-ctx.Done():
		fmt.Println("main", ctx.Err())
	}
}

func handle(ctx context.Context, duration time.Duration) {
	select {
	case <-ctx.Done():
		fmt.Println("handle", ctx.Err())
	case <-time.After(duration):
		fmt.Println("process request with", duration)
	}
}
```

因为过期时间大于处理时间，所以我们有足够的时间处理该请求，运行上述代码会打印出下面的内容：

```
$ go run context.go
process request with 500ms
main context deadline exceeded
```

`handle` 函数没有进入超时的 `select` 分支，但是 `main` 函数的 `select` 却会等待 [`context.Context`](https://draveness.me/golang/tree/context.Context) 超时并打印出 `main context deadline exceeded`。

如果我们将处理请求时间增加至 1500ms，整个程序都会因为上下文的过期而被中止，：

```
$ go run context.go
main context deadline exceeded
handle context deadline exceeded
```

相信这两个例子能够帮助各位读者理解 [`context.Context`](https://draveness.me/golang/tree/context.Context) 的使用方法和设计原理 — **多个 Goroutine 同时订阅 `ctx.Done()` 管道中的消息，一旦接收到取消信号就立刻停止当前正在执行的工作。**

## 默认上下文

[`context`](https://github.com/golang/go/tree/master/src/context) 包中最常用的方法还是 [`context.Background`](https://draveness.me/golang/tree/context.Background)、[`context.TODO`](https://draveness.me/golang/tree/context.TODO)，这两个方法都会返回预先初始化好的私有变量 `background` 和 `todo`，它们会在同一个 Go 程序中被复用：

```
func Background() Context {
	return background
}

func TODO() Context {
	return todo
}
```

这两个私有变量都是通过 `new(emptyCtx)` 语句初始化的，它们是指向私有结构体[`context.emptyCtx`](https://draveness.me/golang/tree/context.emptyCtx) 的指针，这是最简单、最常用的上下文类型：

```
type emptyCtx int

func (*emptyCtx) Deadline() (deadline time.Time, ok bool) {
	return
}

func (*emptyCtx) Done() <-chan struct{} {
	return nil
}

func (*emptyCtx) Err() error {
	return nil
}

func (*emptyCtx) Value(key interface{}) interface{} {
	return nil
}
```

从上述代码中，我们不难发现 [`context.emptyCtx`](https://draveness.me/golang/tree/context.emptyCtx) 通过空方法实现了 [`context.Context`](https://draveness.me/golang/tree/context.Context) 接口中的所有方法，它没有任何功能。

[![golang-context-hierarchy](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/golang-context-hierarchy.png)](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/golang-context-hierarchy.png)

从源代码来看，[`context.Background`](https://draveness.me/golang/tree/context.Background) 和 [`context.TODO`](https://draveness.me/golang/tree/context.TODO) 也**只是互为别名，没有太大的差别**，只是在使用和语义上稍有不同：

- [`context.Background`](https://draveness.me/golang/tree/context.Background) 是上下文的默认值，所有其他的上下文都应该从它衍生出来；
- [`context.TODO`](https://draveness.me/golang/tree/context.TODO) 应该**仅在不确定应该使用哪种上下文时使用**；

`Background`和`TODO`只是用于不同场景下： `Background`通常被**用于主函数、初始化以及测试中，作为一个顶层的`context`**，也就是说一般我们创建的`context`都是基于`Background`；而`TODO`是**在不确定使用什么`context`的时候才会使用**。

在多数情况下，如果当前函数没有上下文作为入参，我们都会使用 [`context.Background`](https://draveness.me/golang/tree/context.Background) 作为起始的上下文向下传递。

## 取消信号

[`context.WithCancel`](https://draveness.me/golang/tree/context.WithCancel) 函数能够从 [`context.Context`](https://draveness.me/golang/tree/context.Context) 中衍生出一个新的子上下文并返回用于取消该上下文的函数。一旦我们执行返回的取消函数，当前上下文以及它的子上下文都会被取消，所有的 Goroutine 都会同步收到这一取消信号。

<img src="https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/2020-01-20-15795072700927-golang-parent-cancel-context.png" alt="golang-parent-cancel-context" style="zoom: 67%;" />

我们直接从 [`context.WithCancel`](https://draveness.me/golang/tree/context.WithCancel) 函数的实现来看它到底做了什么：

```go
func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
	c := newCancelCtx(parent)
	propagateCancel(parent, &c)
	return &c, func() { c.cancel(true, Canceled) }
}
```

- [`context.newCancelCtx`](https://draveness.me/golang/tree/context.newCancelCtx) 将传入的上下文包装成私有结构体 [`context.cancelCtx`](https://draveness.me/golang/tree/context.cancelCtx)；
- [`context.propagateCancel`](https://draveness.me/golang/tree/context.propagateCancel) 会构建父子上下文之间的关联，当父上下文被取消时，子上下文也会被取消：

```go
func propagateCancel(parent Context, child canceler) {
	done := parent.Done()
	if done == nil {
		return // 父上下文不会触发取消信号
	}
	select {
	case <-done:
		child.cancel(false, parent.Err()) // 父上下文已经被取消
		return
	default:
	}

	if p, ok := parentCancelCtx(parent); ok {
		p.mu.Lock()
		if p.err != nil {
			child.cancel(false, p.err)
		} else {
			p.children[child] = struct{}{}
		}
		p.mu.Unlock()
	} else {
		go func() {
			select {
			case <-parent.Done():
				child.cancel(false, parent.Err())
			case <-child.Done():
			}
		}()
	}
}
```

上述函数总共与父上下文相关的三种不同的情况：

1. 当 `parent.Done() == nil`，也就是 `parent` 不会触发取消事件时，当前函数会直接返回；
2. 当`child`的继承链包含可以取消的上下文时，会判断`parent`是否已经触发了取消信号；
   - 如果已经被取消，`child` 会立刻被取消；
   - 如果没有被取消，`child` 会被加入 `parent` 的 `children` 列表中，等待 `parent` 释放取消信号；
3. 当父上下文是开发者自定义的类型、实现了`context.Context`接口并在`Done()`方法中返回了非空的管道时；
   1. 运行一个新的 Goroutine 同时监听 `parent.Done()` 和 `child.Done()` 两个 Channel；
   2. 在 `parent.Done()` 关闭时调用 `child.cancel` 取消子上下文；

[`context.propagateCancel`](https://draveness.me/golang/tree/context.propagateCancel) 的作用是在 `parent` 和 `child` 之间同步取消和结束的信号，保证在 `parent` 被取消时，`child` 也会收到对应的信号，不会出现状态不一致的情况。

[`context.cancelCtx`](https://draveness.me/golang/tree/context.cancelCtx) 实现的几个接口方法也没有太多值得分析的地方，该结构体最重要的方法是 [`context.cancelCtx.cancel`](https://draveness.me/golang/tree/context.cancelCtx.cancel)，该方法会关闭上下文中的 Channel 并向所有的子上下文同步取消信号：

```go
func (c *cancelCtx) cancel(removeFromParent bool, err error) {
	c.mu.Lock()
	if c.err != nil {
		c.mu.Unlock()
		return
	}
	c.err = err
	if c.done == nil {
		c.done = closedchan
	} else {
		close(c.done)
	}
	for child := range c.children {
		child.cancel(false, err)
	}
	c.children = nil
	c.mu.Unlock()

	if removeFromParent {
		removeChild(c.Context, c)
	}
}
```

除了 [`context.WithCancel`](https://draveness.me/golang/tree/context.WithCancel) 之外，[`context`](https://github.com/golang/go/tree/master/src/context) 包中的另外两个函数 [`context.WithDeadline`](https://draveness.me/golang/tree/context.WithDeadline) 和 [`context.WithTimeout`](https://draveness.me/golang/tree/context.WithTimeout) 也都能创建可以被取消的计时器上下文 [`context.timerCtx`](https://draveness.me/golang/tree/context.timerCtx)：

```go
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
	return WithDeadline(parent, time.Now().Add(timeout))
}

func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) {
	if cur, ok := parent.Deadline(); ok && cur.Before(d) {
		return WithCancel(parent)
	}
	c := &timerCtx{
		cancelCtx: newCancelCtx(parent),
		deadline:  d,
	}
	propagateCancel(parent, c)
	dur := time.Until(d)
	if dur <= 0 {
		c.cancel(true, DeadlineExceeded) // 已经过了截止日期
		return c, func() { c.cancel(false, Canceled) }
	}
	c.mu.Lock()
	defer c.mu.Unlock()
	if c.err == nil {
		c.timer = time.AfterFunc(dur, func() {
			c.cancel(true, DeadlineExceeded)
		})
	}
	return c, func() { c.cancel(true, Canceled) }
}
```

[`context.WithDeadline`](https://draveness.me/golang/tree/context.WithDeadline) 在创建 [`context.timerCtx`](https://draveness.me/golang/tree/context.timerCtx) 的过程中判断了父上下文的截止日期与当前日期，并通过 [`time.AfterFunc`](https://draveness.me/golang/tree/time.AfterFunc) 创建定时器，当时间超过了截止日期后会调用 [`context.timerCtx.cancel`](https://draveness.me/golang/tree/context.timerCtx.cancel) 同步取消信号。

[`context.timerCtx`](https://draveness.me/golang/tree/context.timerCtx) 内部不仅通过嵌入 [`context.cancelCtx`](https://draveness.me/golang/tree/context.cancelCtx) 结构体继承了相关的变量和方法，还通过持有的定时器 `timer` 和截止时间 `deadline` 实现了定时取消的功能：

```go
type timerCtx struct {
	cancelCtx
	timer *time.Timer // Under cancelCtx.mu.

	deadline time.Time
}

func (c *timerCtx) Deadline() (deadline time.Time, ok bool) {
	return c.deadline, true
}

func (c *timerCtx) cancel(removeFromParent bool, err error) {
	c.cancelCtx.cancel(false, err)
	if removeFromParent {
		removeChild(c.cancelCtx.Context, c)
	}
	c.mu.Lock()
	if c.timer != nil {
		c.timer.Stop()
		c.timer = nil
	}
	c.mu.Unlock()
}
```

[`context.timerCtx.cancel`](https://draveness.me/golang/tree/context.timerCtx.cancel) 方法不仅调用了 [`context.cancelCtx.cancel`](https://draveness.me/golang/tree/context.cancelCtx.cancel)，还会停止持有的定时器减少不必要的资源浪费。

## 传值方法

在最后我们需要了解如何使用上下文传值，[`context`](https://github.com/golang/go/tree/master/src/context) 包中的 [`context.WithValue`](https://draveness.me/golang/tree/context.WithValue) 能从父上下文中创建一个子上下文，传值的子上下文使用 [`context.valueCtx`](https://draveness.me/golang/tree/context.valueCtx) 类型：

```go
func WithValue(parent Context, key, val interface{}) Context {
	if key == nil {
		panic("nil key")
	}
	if !reflectlite.TypeOf(key).Comparable() {
		panic("key is not comparable")
	}
	return &valueCtx{parent, key, val}
}
```

[`context.valueCtx`](https://draveness.me/golang/tree/context.valueCtx) 结构体会将除了 `Value` 之外的 `Err`、`Deadline` 等方法代理到父上下文中，它只会响应 [`context.valueCtx.Value`](https://draveness.me/golang/tree/context.valueCtx.Value) 方法，该方法的实现也很简单：

```go
type valueCtx struct {
	Context
	key, val interface{}
}

func (c *valueCtx) Value(key interface{}) interface{} {
	if c.key == key {
		return c.val
	}
	return c.Context.Value(key)
}
```

如果 [`context.valueCtx`](https://draveness.me/golang/tree/context.valueCtx) 中存储的键值对与 [`context.valueCtx.Value`](https://draveness.me/golang/tree/context.valueCtx.Value) 方法中传入的参数不匹配，就会从父上下文中查找该键对应的值直到某个父上下文中返回 `nil` 或者查找到对应的值。

## context使用规范

最后，Context虽然是神器，但开发者使用也要遵循基本法，以下是一些Context使用的规范：

- Do not store Contexts inside a struct type; instead, pass a Context explicitly to each function that needs it. The Context should be the first parameter, typically named ctx；**不要把Context存在一个结构体当中，显式地传入函数**。**Context变量需要作为第一个参数使用，一般命名为ctx**；

- Do not pass a nil Context, even if a function permits it. Pass context.TODO if you are unsure about which Context to use；**即使方法允许，也不要传入一个nil的Context，如果你不确定你要用什么Context的时候传一个context.TODO**；
- Use context Values only for request-scoped data that transits processes and APIs, not for passing optional parameters to functions；**使用context的Value相关方法只应该用于在程序和接口中传递的和请求相关的元数据，不要用它来传递一些可选的参数**；
- The same Context may be passed to functions running in different goroutines; Contexts are safe for simultaneous use by multiple goroutines；**同样的Context可以用来传递到不同的goroutine中，Context在多个goroutine中是安全的**；

## 源码分析

### emptyCtx

`emptyCtx`是一个`int`类型的变量，但实现了`context`的接口。`emptyCtx`没有超时时间，不能取消，也不能存储任何额外信息，所以`emptyCtx`用来作为`context`树的根节点。

```go
// An emptyCtx is never canceled, has no values, and has no deadline. It is not
// struct{}, since vars of this type must have distinct addresses.
type emptyCtx int

func (*emptyCtx) Deadline() (deadline time.Time, ok bool) {
    return
}

func (*emptyCtx) Done() <-chan struct{} {
    return nil
}

func (*emptyCtx) Err() error {
    return nil
}

func (*emptyCtx) Value(key interface{}) interface{} {
    return nil
}

func (e *emptyCtx) String() string {
    switch e {
    case background:
        return "context.Background"
    case todo:
        return "context.TODO"
    }
    return "unknown empty Context"
}

var (
    background = new(emptyCtx)
    todo       = new(emptyCtx)
)

func Background() Context {
    return background
}

func TODO() Context {
    return todo
}
```

但我们一般不会直接使用`emptyCtx`，而是使用由`emptyCtx`实例化的两个变量，分别可以通过调用`Background`和`TODO`方法得到，但这两个`context`在实现上是一样的。那么`Background`和`TODO`方法得到的`context`有什么区别呢？可以看一下官方的解释：

```go
// Background returns a non-nil, empty Context. It is never canceled, has no
// values, and has no deadline. It is typically used by the main function,
// initialization, and tests, and as the top-level Context for incoming
// requests.

// TODO returns a non-nil, empty Context. Code should use context.TODO when
// it's unclear which Context to use or it is not yet available (because the
// surrounding function has not yet been extended to accept a Context
// parameter).
```

`Background`和`TODO`只是用于不同场景下： `Background`通常被用于主函数、初始化以及测试中，作为一个顶层的`context`，也就是说一般我们创建的`context`都是基于`Background`；而`TODO`是在不确定使用什么`context`的时候才会使用。

下面将介绍两种不同功能的基础`context`类型：`valueCtx`和`cancelCtx`。

### valueCtx

### valueCtx结构体

```go
type valueCtx struct {
    Context
    key, val interface{}
}

func (c *valueCtx) Value(key interface{}) interface{} {
    if c.key == key {
        return c.val
    }
    return c.Context.Value(key)
}
```

`valueCtx`利用一个`Context`类型的变量来表示父节点`context`，所以当前`context`继承了父`context`的所有信息；`valueCtx`类型还携带一组键值对，也就是说这种`context`可以携带额外的信息。`valueCtx`实现了`Value`方法，用以在`context`链路上获取`key`对应的值，如果当前`context`上不存在需要的`key`,会沿着`context`链向上寻找`key`对应的值，直到根节点。

### WithValue

`WithValue`用以向`context`添加键值对：

```text
func WithValue(parent Context, key, val interface{}) Context {
    if key == nil {
        panic("nil key")
    }
    if !reflect.TypeOf(key).Comparable() {
        panic("key is not comparable")
    }
    return &valueCtx{parent, key, val}
}
```

这里添加键值对不是在原`context`结构体上直接添加，而是以此`context`作为父节点，重新创建一个新的`valueCtx`子节点，将键值对添加在子节点上，由此形成一条`context`链。获取`value`的过程就是在这条`context`链上由尾部上前搜寻：



![img](https://pic4.zhimg.com/v2-6e74cf6f7a4f1701d262c4c0939df52f_r.jpg)



### cancelCtx

### cancelCtx结构体

```go
type cancelCtx struct {
    Context

    mu       sync.Mutex            // protects following fields
    done     chan struct{}         // created lazily, closed by first cancel call
    children map[canceler]struct{} // set to nil by the first cancel call
    err      error                 // set to non-nil by the first cancel call
}

type canceler interface {
    cancel(removeFromParent bool, err error)
    Done() <-chan struct{}
}
```

跟`valueCtx`类似，`cancelCtx`中也有一个`context`变量作为父节点；变量`done`表示一个`channel`，用来表示传递关闭信号；`children`表示一个`map`，存储了当前`context`节点下的子节点；`err`用于存储错误信息表示任务结束的原因。

再来看一下`cancelCtx`实现的方法：

```go
func (c *cancelCtx) Done() <-chan struct{} {
    c.mu.Lock()
    if c.done == nil {
        c.done = make(chan struct{})
    }
    d := c.done
    c.mu.Unlock()
    return d
}

func (c *cancelCtx) Err() error {
    c.mu.Lock()
    err := c.err
    c.mu.Unlock()
    return err
}

func (c *cancelCtx) cancel(removeFromParent bool, err error) {
    if err == nil {
        panic("context: internal error: missing cancel error")
    }
    c.mu.Lock()
    if c.err != nil {
        c.mu.Unlock()
        return // already canceled
    }
    // 设置取消原因
    c.err = err
    设置一个关闭的channel或者将done channel关闭，用以发送关闭信号
    if c.done == nil {
        c.done = closedchan
    } else {
        close(c.done)
    }
    // 将子节点context依次取消
    for child := range c.children {
        // NOTE: acquiring the child's lock while holding parent's lock.
        child.cancel(false, err)
    }
    c.children = nil
    c.mu.Unlock()

    if removeFromParent {
        // 将当前context节点从父节点上移除
        removeChild(c.Context, c)
    }
}
```

可以发现`cancelCtx`类型变量其实也是`canceler`类型，因为`cancelCtx`实现了`canceler`接口。 `Done`方法和`Err`方法没必要说了，`cancelCtx`类型的`context`在调用`cancel`方法时会设置取消原因，将`done channel`设置为一个关闭`channel`或者关闭`channel`，然后将子节点`context`依次取消，如果有需要还会将当前节点从父节点上移除。

### WithCancel

`WithCancel`函数用来创建一个可取消的`context`，即`cancelCtx`类型的`context`。`WithCancel`返回一个`context`和一个`CancelFunc`，调用`CancelFunc`即可触发`cancel`操作。直接看源码：

```go
type CancelFunc func()

func WithCancel(parent Context) (ctx Context, cancel CancelFunc) {
    c := newCancelCtx(parent)
    propagateCancel(parent, &c)
    return &c, func() { c.cancel(true, Canceled) }
}

// newCancelCtx returns an initialized cancelCtx.
func newCancelCtx(parent Context) cancelCtx {
    // 将parent作为父节点context生成一个新的子节点
    return cancelCtx{Context: parent}
}

func propagateCancel(parent Context, child canceler) {
    if parent.Done() == nil {
        // parent.Done()返回nil表明父节点以上的路径上没有可取消的context
        return // parent is never canceled
    }
    // 获取最近的类型为cancelCtx的祖先节点
    if p, ok := parentCancelCtx(parent); ok {
        p.mu.Lock()
        if p.err != nil {
            // parent has already been canceled
            child.cancel(false, p.err)
        } else {
            if p.children == nil {
                p.children = make(map[canceler]struct{})
            }
            // 将当前子节点加入最近cancelCtx祖先节点的children中
            p.children[child] = struct{}{}
        }
        p.mu.Unlock()
    } else {
        go func() {
            select {
            case <-parent.Done():
                child.cancel(false, parent.Err())
            case <-child.Done():
            }
        }()
    }
}

func parentCancelCtx(parent Context) (*cancelCtx, bool) {
    for {
        switch c := parent.(type) {
        case *cancelCtx:
            return c, true
        case *timerCtx:
            return &c.cancelCtx, true
        case *valueCtx:
            parent = c.Context
        default:
            return nil, false
        }
    }
}
```

之前说到`cancelCtx`取消时，会将后代节点中所有的`cancelCtx`都取消，`propagateCancel`即用来建立当前节点与祖先节点这个取消关联逻辑。

1. 如果`parent.Done()`返回`nil`，表明父节点以上的路径上没有可取消的`context`，不需要处理；
2. 如果在`context`链上找到到`cancelCtx`类型的祖先节点，则判断这个祖先节点是否已经取消，如果已经取消就取消当前节点；否则将当前节点加入到祖先节点的`children`列表。
3. 否则开启一个协程，监听`parent.Done()`和`child.Done()`，一旦`parent.Done()`返回的`channel`关闭，即`context`链中某个祖先节点`context`被取消，则将当前`context`也取消。

这里或许有个疑问，为什么是祖先节点而不是父节点？这是因为当前`context`链可能是这样的：



![img](https://pic3.zhimg.com/v2-dcdc25c62efcb902015058e1bebb8cde_r.jpg)



当前`cancelCtx`的父节点`context`并不是一个可取消的`context`，也就没法记录`children`。

### timerCtx

`timerCtx`是一种基于`cancelCtx`的`context`类型，从字面上就能看出，这是一种可以定时取消的`context`。

```go
type timerCtx struct {
    cancelCtx
    timer *time.Timer // Under cancelCtx.mu.

    deadline time.Time
}

func (c *timerCtx) Deadline() (deadline time.Time, ok bool) {
    return c.deadline, true
}

func (c *timerCtx) cancel(removeFromParent bool, err error) {
    将内部的cancelCtx取消
    c.cancelCtx.cancel(false, err)
    if removeFromParent {
        // Remove this timerCtx from its parent cancelCtx's children.
        removeChild(c.cancelCtx.Context, c)
    }
    c.mu.Lock()
    if c.timer != nil {
        取消计时器
        c.timer.Stop()
        c.timer = nil
    }
    c.mu.Unlock()
}
```

`timerCtx`内部使用`cancelCtx`实现取消，另外使用定时器`timer`和过期时间`deadline`实现定时取消的功能。`timerCtx`在调用`cancel`方法，会先将内部的`cancelCtx`取消，如果需要则将自己从`cancelCtx`祖先节点上移除，最后取消计时器。

### WithDeadline

`WithDeadline`返回一个基于`parent`的可取消的`context`，并且其过期时间`deadline`不晚于所设置时间`d`。

```go
func WithDeadline(parent Context, d time.Time) (Context, CancelFunc) {
    if cur, ok := parent.Deadline(); ok && cur.Before(d) {
        // The current deadline is already sooner than the new one.
        return WithCancel(parent)
    }
    c := &timerCtx{
        cancelCtx: newCancelCtx(parent),
        deadline:  d,
    }
    // 建立新建context与可取消context祖先节点的取消关联关系
    propagateCancel(parent, c)
    dur := time.Until(d)
    if dur <= 0 {
        c.cancel(true, DeadlineExceeded) // deadline has already passed
        return c, func() { c.cancel(false, Canceled) }
    }
    c.mu.Lock()
    defer c.mu.Unlock()
    if c.err == nil {
        c.timer = time.AfterFunc(dur, func() {
            c.cancel(true, DeadlineExceeded)
        })
    }
    return c, func() { c.cancel(true, Canceled) }
}
```

1. 如果父节点`parent`有过期时间并且过期时间早于给定时间`d`，那么新建的子节点`context`无需设置过期时间，使用`WithCancel`创建一个可取消的`context`即可；
2. 否则，就要利用`parent`和过期时间`d`创建一个定时取消的`timerCtx`，并建立新建`context`与可取消`context`祖先节点的取消关联关系，接下来判断当前时间距离过期时间`d`的时长`dur`：
3. 如果`dur`小于0，即当前已经过了过期时间，则直接取消新建的`timerCtx`，原因为`DeadlineExceeded`；
4. 否则，为新建的`timerCtx`设置定时器，一旦到达过期时间即取消当前`timerCtx`。

### WithTimeout

与`WithDeadline`类似，`WithTimeout`也是创建一个定时取消的`context`，只不过`WithDeadline`是接收一个过期时间点，而`WithTimeout`接收一个相对当前时间的过期时长`timeout`:

```go
func WithTimeout(parent Context, timeout time.Duration) (Context, CancelFunc) {
    return WithDeadline(parent, time.Now().Add(timeout))
}
```

## 示例

首先使用`context`实现文章开头`done channel`的例子来示范一下如何更优雅实现协程间取消信号的同步：

```go
func main() {
    messages := make(chan int, 10)

    // producer
    for i := 0; i < 10; i++ {
        messages <- i
    }

    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)

    // consumer
    go func(ctx context.Context) {
        ticker := time.NewTicker(1 * time.Second)
        for _ = range ticker.C {
            select {
            case <-ctx.Done():
                fmt.Println("child process interrupt...")
                return
            default:
                fmt.Printf("send message: %d\n", <-messages)
            }
        }
    }(ctx)

    defer close(messages)
    defer cancel()

    select {
    case <-ctx.Done():
        time.Sleep(1 * time.Second)
        fmt.Println("main process exit!")
    }
}
```

这个例子中，只要让子线程监听主线程传入的`ctx`，一旦`ctx.Done()`返回空`channel`，子线程即可取消执行任务。但这个例子还无法展现`context`的传递取消信息的强大优势。

阅读过`net/http`包源码的朋友可能注意到在实现`http server`时就用到了`context`, 下面简单分析一下。

1、首先`Server`在开启服务时会创建一个`valueCtx`,存储了`server`的相关信息，之后每建立一条连接就会开启一个协程，并携带此`valueCtx`。

```go
func (srv *Server) Serve(l net.Listener) error {

    ...

    var tempDelay time.Duration     // how long to sleep on accept failure
    baseCtx := context.Background() // base is always background, per Issue 16220
    ctx := context.WithValue(baseCtx, ServerContextKey, srv)
    for {
        rw, e := l.Accept()

        ...

        tempDelay = 0
        c := srv.newConn(rw)
        c.setState(c.rwc, StateNew) // before Serve can return
        go c.serve(ctx)
    }
}
```

2、建立连接之后会基于传入的`context`创建一个`valueCtx`用于存储本地地址信息，之后在此基础上又创建了一个`cancelCtx`，然后开始从当前连接中读取网络请求，每当读取到一个请求则会将该`cancelCtx`传入，用以传递取消信号。一旦连接断开，即可发送取消信号，取消所有进行中的网络请求。

```go
func (c *conn) serve(ctx context.Context) {
    c.remoteAddr = c.rwc.RemoteAddr().String()
    ctx = context.WithValue(ctx, LocalAddrContextKey, c.rwc.LocalAddr())
    ...

    ctx, cancelCtx := context.WithCancel(ctx)
    c.cancelCtx = cancelCtx
    defer cancelCtx()

    ...

    for {
        w, err := c.readRequest(ctx)

        ...

        serverHandler{c.server}.ServeHTTP(w, w.req)

        ...
    }
}
```

3、读取到请求之后，会再次基于传入的`context`创建新的`cancelCtx`,并设置到当前请求对象`req`上，同时生成的`response`对象中`cancelCtx`保存了当前`context`取消方法。

```go
func (c *conn) readRequest(ctx context.Context) (w *response, err error) {

    ...

    req, err := readRequest(c.bufr, keepHostHeader)

    ...

    ctx, cancelCtx := context.WithCancel(ctx)
    req.ctx = ctx

    ...

    w = &response{
        conn:          c,
        cancelCtx:     cancelCtx,
        req:           req,
        reqBody:       req.Body,
        handlerHeader: make(Header),
        contentLength: -1,
        closeNotifyCh: make(chan bool, 1),

        // We populate these ahead of time so we're not
        // reading from req.Header after their Handler starts
        // and maybe mutates it (Issue 14940)
        wants10KeepAlive: req.wantsHttp10KeepAlive(),
        wantsClose:       req.wantsClose(),
    }

    ...
    return w, nil
}
```

这样处理的目的主要有以下几点：

- 一旦请求超时，即可中断当前请求；
- 在处理构建`response`过程中如果发生错误，可直接调用`response`对象的`cancelCtx`方法结束当前请求；
- 在处理构建`response`完成之后，调用`response`对象的`cancelCtx`方法结束当前请求。

在整个`server`处理流程中，使用了一条`context`链贯穿`Server`、`Connection`、`Request`，不仅将上游的信息共享给下游任务，同时实现了上游可发送取消信号取消所有下游任务，而下游任务自行取消不会影响上游任务。

## 小结 

Go 语言中的 [`context.Context`](https://draveness.me/golang/tree/context.Context) 的主要作用还是在多个 Goroutine 组成的树中同步取消信号以减少对资源的消耗和占用，虽然它也有传值的功能，但是这个功能我们还是很少用到。

在真正使用传值的功能时我们也应该非常谨慎，使用 [`context.Context`](https://draveness.me/golang/tree/context.Context) 传递请求的所有参数一种非常差的设计，比较常见的使用场景是传递请求对应用户的认证令牌以及用于进行分布式追踪的请求 ID。



## 延伸阅读 

- [Package context · Golang](https://golang.org/pkg/context/)
- [Go Concurrency Patterns: Context](https://blog.golang.org/context)
- [Using context cancellation in Go](https://www.sohamkamani.com/blog/golang/2018-06-17-golang-using-context-cancellation/)