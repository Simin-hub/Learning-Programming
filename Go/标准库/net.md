# net/http

## 简介

几乎所有的编程语言都以`Hello World`作为入门程序的示例，其中有一部分以编写一个 Web 服务器作为实战案例的开始。每种编程语言都有很多用于编写 Web 服务器的库，或以标准库，或通过第三方库的方式提供。Go 语言也不例外。本文及后续的文章就去探索 Go 语言中的各个Web 编程框架，它们的基本使用，阅读它们的源码，比较它们优缺点。让我们先从 Go 语言的标准库`net/http`开始。标准库`net/http`让编写 Web 服务器的工作变得非常简单。我们一起探索如何使用`net/http`库实现一些常见的功能或模块，了解这些对我们学习其他的库或框架将会很有帮助。

使用`net/http`编写一个简单的 Web 服务器非常简单：

```
package main

import (
  "fmt"
  "net/http"
)

func index(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintln(w, "Hello World")
}

func main() {
  http.HandleFunc("/", index)
  http.ListenAndServe(":8080", nil)
}
```

```golang
package main

import (
  "fmt"
  "net/http"
)

func index(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintln(w, "Hello World")
}

func main() {
  http.HandleFunc("/", index)
  http.ListenAndServe(":8080", nil)
}
```

首先，我们调用`http.HandleFunc("/", index)`注册路径处理函数，这里将路径`/`的处理函数设置为`index`。处理函数的类型必须是：



通过反射插入到正常逻辑的处理流程中，在 Go 语言中基本不这样做。

在 Go 中，**中间件是通过函数闭包来实现的**。Go 语言中的函数是第一类值，既可以作为参数传给其他函数，也可以作为返回值从其他函数返回。我们前面介绍了处理器/函数的使用和实现。那么可以利用闭包封装已有的处理函数。

首先，基于函数类型`func(http.Handler) http.Handler`定义一个中间件类型：

```
type Middleware func(http.Handler) http.Handler 
```

接下来我们来编写中间件，最简单的中间件就是在请求前后各输出一条日志：

| `1 2 3 4 5 6 7 8 9 ` | `func WithLogger(handler http.Handler) http.Handler {  return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {    logger.Printf("path:%s process start...\n", r.URL.Path)    defer func() {      logger.Printf("path:%s process end...\n", r.URL.Path)    }()    handler.ServeHTTP(w, r)  }) } ` |
| -------------------- | ------------------------------------------------------------ |
|                      |                                                              |

实现很简单，通过中间件封装原来的处理器对象，然后返回一个新的处理函数。在新的处理函数中，先输出开始处理的日志，然后用`defer`语句在函数结束后输出处理结束的日志。接着调用原处理器对象的`ServeHTTP()`方法执行原处理逻辑。

类似地，我们再来实现一个统计处理耗时的中间件：

| ` 1 2 3 4 5 6 7 8 9 10 ` | `func Metric(handler http.Handler) http.HandlerFunc {  return func (w http.ResponseWriter, r *http.Request) {    start := time.Now()    defer func() {      logger.Printf("path:%s elapsed:%fs\n", r.URL.Path, time.Since(start).Seconds())    }()    time.Sleep(1 * time.Second)    handler.ServeHTTP(w, r)  } } ` |
| ------------------------ | ------------------------------------------------------------ |
|                          |                                                              |

`Metric`中间件封装原处理器对象，开始执行前记录时间，执行完成后输出耗时。为了能方便看到结果，我在上面代码中添加了一个`time.Sleep()`调用。

最后，由于请求的处理逻辑都是由功能开发人员（而非库作者）自己编写的，所以为了 Web 服务器的稳定，我们需要捕获可能出现的 panic。`PanicRecover`中间件如下：

| ` 1 2 3 4 5 6 7 8 9 10 11 ` | `func PanicRecover(handler http.Handler) http.Handler {  return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {    defer func() {      if err := recover(); err != nil {        logger.Println(string(debug.Stack()))      }    }()     handler.ServeHTTP(w, r)  }) } ` |
| --------------------------- | ------------------------------------------------------------ |
|                             |                                                              |

调用`recover()`函数捕获 panic，输出堆栈信息，为了防止程序异常退出。实际上，在`conn.serve()`方法中也有`recover()`，程序一般不会异常退出。但是自定义的中间件可以添加我们自己的定制逻辑。

现在我们可以这样来注册处理函数：

| `1 2 ` | `mux.Handle("/", PanicRecover(WithLogger(Metric(http.HandlerFunc(index))))) mux.Handle("/greeting", PanicRecover(WithLogger(Metric(greeting("welcome, dj"))))) ` |
| ------ | ------------------------------------------------------------ |
|        |                                                              |

这种方式略显繁琐，我们可以编写一个帮助函数，它接受原始的处理器对象，和可变的多个中间件。对处理器对象应用这些中间件，返回新的处理器对象：

| `1 2 3 4 5 6 7 ` | `func applyMiddlewares(handler http.Handler, middlewares ...Middleware) http.Handler {  for i := len(middlewares)-1; i >= 0; i-- {    handler = middlewares[i](handler)  }   return handler } ` |
| ---------------- | ------------------------------------------------------------ |
|                  |                                                              |

注意应用顺序是**从右到左**的，即**右结合**，越靠近原处理器的越晚执行。

利用帮助函数，注册可以简化为：

| `1 2 3 4 5 6 7 ` | `middlewares := []Middleware{  PanicRecover,  WithLogger,  Metric, } mux.Handle("/", applyMiddlewares(http.HandlerFunc(index), middlewares...)) mux.Handle("/greeting", applyMiddlewares(greeting("welcome, dj"), middlewares...)) ` |
| ---------------- | ------------------------------------------------------------ |
|                  |                                                              |

上面每次注册处理逻辑都需要调用一次`applyMiddlewares()`函数，还是略显繁琐。我们可以这样来优化，封装一个自己的`ServeMux`结构，然后定义一个方法`Use()`将中间件保存下来，重写`Handle/HandleFunc`将传入的`http.HandlerFunc/http.Handler`处理器包装中间件之后再传给底层的`ServeMux.Handle()`方法：

| ` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 ` | `type MyMux struct {  *http.ServeMux  middlewares []Middleware } func NewMyMux() *MyMux {  return &MyMux{    ServeMux: http.NewServeMux(),  } } func (m *MyMux) Use(middlewares ...Middleware) {  m.middlewares = append(m.middlewares, middlewares...) } func (m *MyMux) Handle(pattern string, handler http.Handler) {  handler = applyMiddlewares(handler, m.middlewares...)  m.ServeMux.Handle(pattern, handler) } func (m *MyMux) HandleFunc(pattern string, handler http.HandlerFunc) {  newHandler := applyMiddlewares(handler, m.middlewares...)  m.ServeMux.Handle(pattern, newHandler) } ` |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

注册时只需要创建`MyMux`对象，调用其`Use()`方法传入要应用的中间件即可：

| `1 2 3 4 5 6 7 8 9 ` | `middlewares := []Middleware{  PanicRecover,  WithLogger,  Metric, } mux := NewMyMux() mux.Use(middlewares...) mux.HandleFunc("/", index) mux.Handle("/greeting", greeting("welcome, dj")) ` |
| -------------------- | ------------------------------------------------------------ |
|                      |                                                              |

这种方式简单易用，但是也有它的问题，最大的问题是必须先设置好中间件，然后才能调用`Handle/HandleFunc`注册，后添加的中间件不会对之前注册的处理器/函数生效。

为了解决这个问题，我们可以改写`ServeHTTP`方法，在确定了处理器之后再应用中间件。这样后续添加的中间件也能生效。很多第三方库都是采用这种方式。`http.ServeMux`默认的`ServeHTTP()`方法如下：

| ` 1 2 3 4 5 6 7 8 9 10 11 ` | `func (m *ServeMux) ServeHTTP(w http.ResponseWriter, r *http.Request) {  if r.RequestURI == "*" {    if r.ProtoAtLeast(1, 1) {      w.Header().Set("Connection", "close")    }    w.WriteHeader(http.StatusBadRequest)    return  }  h, _ := m.Handler(r)  h.ServeHTTP(w, r) } ` |
| --------------------------- | ------------------------------------------------------------ |
|                             |                                                              |

改造这个方法定义`MyMux`类型的`ServeHTTP()`方法也很简单，只需要在`m.Handler(r)`获取处理器之后，应用当前的中间件即可：

| `1 2 3 4 5 6 7 ` | `func (m *MyMux) ServeHTTP(w http.ResponseWriter, r *http.Request) {  // ...  h, _ := m.Handler(r)  // 只需要加这一行即可  h = applyMiddlewares(h, m.middlewares...)  h.ServeHTTP(w, r) } ` |
| ---------------- | ------------------------------------------------------------ |
|                  |                                                              |

后面我们分析其他 Web 框架的源码时会发现，很多都是类似的做法。为了测试宕机恢复，编写一个会触发 panic 的处理函数：

| `1 2 3 4 5 ` | `func panics(w http.ResponseWriter, r *http.Request) {  panic("not implemented") } mux.HandleFunc("/panic", panics) ` |
| ------------ | ------------------------------------------------------------ |
|              |                                                              |

运行，在浏览器中请求`localhost:8080/`和`localhost:8080/greeting`，最后请求`localhost:8080/panic`触发 panic：

![img](https://darjun.github.io/img/in-post/godailylib/nethttp3.png#center)![img](https://darjun.github.io/img/in-post/godailylib/nethttp4.png#center)

## 思考题

思考题：

这其实就是看阅读代码是不是仔细，最长前缀的排序列表在`ServeMux.Handle()`方法中生成：

| `1 2 3 4 5 ` | `func (mux *ServeMux) Handle(pattern string, handler Handler) {  if pattern[len(pattern)-1] == '/' {    mux.es = appendSorted(mux.es, e)  } } ` |
| ------------ | ------------------------------------------------------------ |
|              |                                                              |

这里明显有个限制条件，即注册路径最后必须以`/`结尾才会触发。所以`localhost:8080/greeting/a/b/c`和`localhost:8080/a/b/c`都只会匹配`/`路径。如果想要让`localhost:8080/greeting/a/b/c`匹配路径`/greeting`，注册路径需要改为`/greeting/`：

| `1 ` | `http.Handle("/greeting/", greeting("Welcome to go web frameworks")) ` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

这时请求路径`/greeting`会自动重定向（301）到`/greeting/`。

## 总结

本文介绍了使用标准库`net/http`创建 Web 服务器的基本流程，一步步分析源码。然后介绍了如何使用中间件简化通用的处理逻辑。学习并理解了`net/http`库的内容对于学习其他的 Go Web 框架非常有帮助。第三方的 Go Web 框架大多也是基于`net/http`实现自己的`ServeMux`对象而已。

[详解golang net之transport](https://www.cnblogs.com/charlieroro/p/11409153.html)