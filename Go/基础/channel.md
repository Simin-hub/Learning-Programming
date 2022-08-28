# channel

[参考地址](https://segmentfault.com/a/1190000017958702)

[参考地址](https://www.topgoer.com/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/channel.html)

[地址](https://juejin.cn/post/6844904016254599176)

单纯地将函数并发执行是没有意义的。函数与函数间需要交换数据才能体现并发执行函数的意义。

虽然可以使用共享内存进行数据交换，但是共享内存在不同的goroutine中容易发生竞态问题。为了保证数据交换的正确性，必须使用互斥量对内存进行加锁，这种做法势必造成性能问题。

Go语言的并发模型是CSP（Communicating Sequential Processes），提倡通过通信共享内存而不是通过共享内存而实现通信。

如果说goroutine是Go程序并发的执行体，channel就是它们之间的连接。channel是可以让一个goroutine发送特定值到另一个goroutine的通信机制。

Go 语言中的通道（channel）是一种特殊的类型。通道像一个传送带或者队列，总是遵循先入先出（First In First Out）的规则，保证收发数据的顺序。每一个通道都是一个具体类型的导管，也就是声明channel的时候需要为其指定元素类型。

## channel类型

channel是一种类型，一种引用类型。声明通道类型的格式如下：

```go
var 变量 chan 元素类型
```

举几个例子：

```go
var ch1 chan int   // 声明一个传递整型的通道
var ch2 chan bool  // 声明一个传递布尔型的通道
var ch3 chan []int // 声明一个传递int切片的通道
```

Channel 分成了以下三种类型[^8^](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-channel/#fn:8)：

- 同步 Channel — 不需要缓冲区，发送方会直接将数据交给（Handoff）接收方；
- 异步 Channel — 基于环形缓存的传统生产者消费者模型；
- `chan struct{}` 类型的异步 Channel — `struct{}` 类型不占用内存空间，不需要实现缓冲区和直接发送（Handoff）的语义；

### 创建channel

通道是引用类型，通道类型的空值是nil。

```go
var ch chan int
fmt.Println(ch) // <nil>
```

声明的通道后需要使用make函数初始化之后才能使用。

创建channel的格式如下：

```
make(chan 元素类型, [缓冲大小])
```

channel的缓冲大小是可选的。

举几个例子：

```go
ch4 := make(chan int)
ch5 := make(chan bool)
ch6 := make(chan []int)
```

## channel的基本操作和注意事项

channel存在`3种状态`：

1. nil，未初始化的状态，只进行了声明，或者手动赋值为`nil`
2. active，正常的channel，可读或者可写
3. closed，已关闭，**千万不要误认为关闭channel后，channel的值是nil**

**通道有发送（send）、接收(receive）和关闭（close）三种操作。**

发送和接收都使用<-符号。

现在我们先使用以下语句定义一个通道：

```go
ch := make(chan int)
```

#### 发送

将一个值发送到通道中。

```go
ch <- 10 // 把10发送到ch中
```

#### 接收

从一个通道中接收值。

```go
x := <- ch // 从ch中接收值并赋值给变量x
<-ch       // 从ch中接收值，忽略结果
```

#### 关闭

我们通过调用内置的close函数来关闭通道。

```go
close(ch)
```

关于关闭通道需要注意的事情是，**只有在通知接收方goroutine所有的数据都发送完毕的时候才需要关闭通道**。通道是可以被垃圾回收机制回收的，它和关闭文件是不一样的，在结束操作之后关闭文件是必须要做的，但**关闭通道不是必须**的。

关闭后的通道有以下特点：

1. 对一个关闭的通道再发送值就会导致panic。

2. 对一个关闭的通道进行接收会一直获取值直到通道为空。

3. 对一个关闭的并且没有值的通道执行接收操作会得到对应类型的零值。

4. 关闭一个已经关闭的通道会导致panic。

| 操作      | nil的channel | 正常channel | 已关闭channel |
| --------- | ------------ | ----------- | ------------- |
| <- ch     | 阻塞         | 成功或阻塞  | 读到零值      |
| ch <-     | 阻塞         | 成功或阻塞  | panic         |
| close(ch) | panic        | 成功        | panic         |

对于nil通道的情况，也并非完全遵循上表，**有1个特殊场景**：当`nil`的通道在`select`的某个`case`中时，这个case会阻塞，但不会造成死锁.

### 无缓冲的通道

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/13.png)

无缓冲的通道又称为阻塞的通道。我们来看一下下面的代码：

```go
func main() {
    ch := make(chan int)
    ch <- 10
    fmt.Println("发送成功")
}
```

上面这段代码能够通过编译，但是执行的时候会出现以下错误：

```
fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan send]:
main.main()
.../src/github.com/pprof/studygo/day06/channel02/main.go:8 +0x54
```

为什么会出现deadlock错误呢？

因为我们使用ch := make(chan int)创建的是无缓冲的通道，无缓冲的通道只有在有人接收值的时候才能发送值。就像你住的小区没有快递柜和代收点，快递员给你打电话必须要把这个物品送到你的手中，简单来说就是无缓冲的通道必须有接收才能发送。

上面的代码会阻塞在ch <- 10这一行代码形成死锁，那如何解决这个问题呢？

一种方法是启用一个goroutine去接收值，例如：

```go
func recv(c chan int) {
    ret := <-c
    fmt.Println("接收成功", ret)
}
func main() {
    ch := make(chan int)
    go recv(ch) // 启用goroutine从通道接收值
    ch <- 10
    fmt.Println("发送成功")
}
```

无缓冲通道上的发送操作会阻塞，直到另一个goroutine在该通道上执行接收操作，这时值才能发送成功，两个goroutine将继续执行。相反，如果接收操作先执行，接收方的goroutine将阻塞，直到另一个goroutine在该通道上发送一个值。

**使用无缓冲通道进行通信将导致发送和接收的goroutine同步化**。因此，**无缓冲通道也被称为同步通道。**

### 有缓冲的通道

解决上面问题的方法还有一种就是使用有缓冲区的通道。

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/4.png)

我们可以在使用make函数初始化通道的时候为其指定通道的容量，例如：

```go
func main() {
    ch := make(chan int, 1) // 创建一个容量为1的有缓冲区通道
    ch <- 10
    fmt.Println("发送成功")
}
```

**只要通道的容量大于零，那么该通道就是有缓冲的通道**，通道的容量表示通道中能存放元素的数量。就像你小区的快递柜只有那么个多格子，格子满了就装不下了，就阻塞了，等到别人取走一个快递员就能往里面放一个。

我们可以使用内置的len函数获取通道内元素的数量，使用cap函数获取通道的容量，虽然我们很少会这么做。

### close()

可以通过内置的close()函数关闭channel（如果你的管道不往里存值或者取值的时候一定记得关闭管道）

```go
package main

import "fmt"

func main() {
    c := make(chan int)
    go func() {
        for i := 0; i < 5; i++ {
            c <- i
        }
        close(c)
    }()
    for {
        if data, ok := <-c; ok {
            fmt.Println(data)
        } else {
            break
        }
    }
    fmt.Println("main结束")
}
```

### 如何优雅的从通道循环取值

当通过通道发送有限的数据时，我们可以通过close函数关闭通道来告知从该通道接收值的goroutine停止等待。当通道被关闭时，往该通道发送值会引发panic，从该通道里接收的值一直都是类型零值。那如何判断一个通道是否被关闭了呢？

我们来看下面这个例子：

```go
// channel 练习
func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)
    // 开启goroutine将0~100的数发送到ch1中
    go func() {
        for i := 0; i < 100; i++ {
            ch1 <- i
        }
        close(ch1)
    }()
    // 开启goroutine从ch1中接收值，并将该值的平方发送到ch2中
    go func() {
        for {
            i, ok := <-ch1 // 通道关闭后再取值ok=false
            if !ok {
                break
            }
            ch2 <- i * i
        }
        close(ch2)
    }()
    // 在主goroutine中从ch2中接收值打印
    for i := range ch2 { // 通道关闭后会退出for range循环
        fmt.Println(i)
    }
}
```

从上面的例子中我们看到有两种方式在接收值的时候判断通道是否被关闭，我们通常使用的是for range的方式。

### 单向通道

有的时候我们会将通道作为参数在多个任务函数间传递，很多时候我们在不同的任务函数中使用通道都会对其进行限制，比如限制通道在函数中只能发送或只能接收。

Go语言中提供了单向通道来处理这种情况。例如，我们把上面的例子改造如下：

```go
func counter(out chan<- int) {
    for i := 0; i < 100; i++ {
        out <- i
    }
    close(out)
}

func squarer(out chan<- int, in <-chan int) {
    for i := range in {
        out <- i * i
    }
    close(out)
}
func printer(in <-chan int) {
    for i := range in {
        fmt.Println(i)
    }
}

func main() {
    ch1 := make(chan int)
    ch2 := make(chan int)
    go counter(ch1)
    go squarer(ch2, ch1)
    printer(ch2)
}
```

其中，

```
1.chan<- int是一个只能发送的通道，可以发送但是不能接收；
2.<-chan int是一个只能接收的通道，可以接收但是不能发送。
```

在函数传参及任何赋值操作中将双向通道转换为单向通道是可以的，但反过来是不可以的。

### 通道总结

channel常见的异常总结，如下图：

![通道总结](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/wss1.png)

注意:关闭已经关闭的channel也会引发panic。

![image-20220825164549552](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/image-20220825164549552.png)

### channel的遍历

[地址](https://www.jianshu.com/p/fe5dd2efed5d)

##### for range

`channel`支持 `for range` 的方式进行遍历：

```go
package main  

import "fmt"  

func main() {  
    ci := make(chan int, 5)  
    for i := 1; i <= 5; i++ {
        ci <- i
    }    
    close(ci)  

    for i := range ci {  
        fmt.Println(i)  
    }  
}  

```

值得注意的是，在遍历时，如果`channel` 没有关闭，那么会一直等待下去，出现 `deadlock` 的错误；如果在遍历时`channel`已经关闭，那么在遍历完数据后自动退出遍历。也就是说，`for range` 的遍历方式时阻塞型的遍历方式。意思就是只有当通道关闭且无数据才会结束循环。

[地址](https://segmentfault.com/a/1190000040399883)

1.在遍历时，如果 channel 没有关闭，则回出现 deadlock 的错误。

2.在遍历时，如果 channel 已经关闭，则会正常遍历数据，遍历完后，就会退出遍历。

3.对于nil channel，无论收发都会被阻塞。

4.写完 chan 之后一定要关闭close chan，否则主协程读的时候，会发生被阻塞。

5.已关闭的Channel(有缓冲的),如果继续读数据，得到的是零值(对于int，就是0), 如果没有关闭，读不到数据了。

##### for select

`select`可以处理非阻塞式消息发送、接收及多路选择。

`select`中有`case`代码块，用于`channel`发送或接收消息，**任意**一个`case`代码块准备好时，执行其对应内容；多个`case`代码块准备好时，**随机选择一个**`case`代码块并执行；**所有`case`代码块都没有准备好，则等待**；还可以有一个`default`代码块，**所有`case`代码块都没有准备好时执行`default`代码块**。



## channel的使用场景

把channel用在**数据流动的地方**：

1. 消息传递、消息过滤
2. 信号广播
3. 事件订阅与广播
4. 请求、响应转发
5. 任务分发
6. 结果汇总
7. 并发控制
8. 同步与异步
9. ...

## select

https://www.topgoer.com/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B/select.html

