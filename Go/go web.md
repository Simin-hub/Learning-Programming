# Go Web

[参考地址](https://learnku.com/docs/build-web-application-with-golang)

## 一、Web 基础

### web工作方式

**URL和DNS解析**

我们平时浏览网页的时候，会打开浏览器，输入网址后按下回车键，然后就会显示出你想要浏览的内容。在这个看似简单的用户行为背后，到底隐藏了些什么呢？

对于普通的上网过程，系统其实是这样做的：浏览器本身是一个客户端，当你输入 URL 的时候，首先浏览器会去请求 [**DNS 服务器(域名解析服务器)**](https://baike.baidu.com/item/%E5%9F%9F%E5%90%8D%E6%9C%8D%E5%8A%A1%E5%99%A8/9705133)，通过 DNS 获取相应的域名对应的 IP，然后通过 IP 地址找到 IP 对应的服务器后，要求建立 TCP 连接，等浏览器发送完 HTTP Request（请求）包后，服务器接收到请求包之后才开始处理请求包，服务器调用自身服务，返回 HTTP Response（响应）包；客户端收到来自服务器的响应后开始渲染这个 Response 包里的主体（body），等收到全部的内容随后断开与该服务器之间的 TCP 连接。

一个 Web 服务器也被称为 HTTP 服务器，它通过 HTTP 协议与客户端通信。这个客户端通常指的是 Web 浏览器 (其实手机端客户端内部也是浏览器实现的)。

Web 服务器的工作原理可以简单地归纳为：

- 客户机通过 TCP/IP 协议建立到服务器的 TCP 连接
- 客户端向服务器发送 HTTP 协议请求包，请求服务器里的资源文档(请求页面)
- 服务器向客户机发送 HTTP 协议应答包，如果请求的资源包含有动态语言的内容，那么服务器会调用动态语言的解释引擎负责处理 “动态内容”，并将处理得到的数据返回给客户端（响应页面）
- 客户机与服务器断开。由客户端解释 HTML 文档，在客户端屏幕上渲染图形结果

一个简单的 HTTP 事务就是这样实现的，看起来很复杂，原理其实是挺简单的。需要注意的是客户机与服务器之间的通信是非持久连接的，也就是当服务器发送了应答后就与客户机断开连接，等待下一次请求。

我们浏览网页都是通过 URL 访问的，那么 URL 到底是怎么样的呢？

**URL** (Uniform Resource Locator) 是 “统一资源定位符” 的英文缩写，用于描述一个网络上的资源，基本格式如下

```
scheme:	//host[:port#]/path/.../[?query-string][#anchor]
```

```
scheme         指定底层使用的协议(例如：http, https, ftp)
host           HTTP 服务器的 IP 地址或者域名
port#          HTTP 服务器的默认端口是 80，这种情况下端口号可以省略。如果使用了别的端口，必须指明，例如 http://www.cnblogs.com:8080/
path           访问资源的路径
query-string   发送给 http 服务器的数据
anchor         锚
```

**DNS** (Domain Name System) 是 “域名系统” 的英文缩写，是一种组织成域层次结构的计算机和网络服务命名系统，它用于 TCP/IP 网络，它从事将主机名或域名转换为实际 IP 地址的工作。DNS 就是这样的一位 “翻译官”，它的基本工作原理可用下图来表示。

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/3.1.dns_hierachy.png)

更详细的 DNS 解析的过程如下，这个过程有助于我们理解 DNS 的工作模式

在浏览器中输入 www.qq.com 域名，操作系统会先检查自己本地的 hosts 文件是否有这个网址映射关系，如果有，就先调用这个 IP 地址映射，完成域名解析。

如果 hosts 里没有这个域名的映射，则查找本地 DNS 解析器缓存，是否有这个网址映射关系，如果有，直接返回，完成域名解析。

如果 hosts 与本地 DNS 解析器缓存都没有相应的网址映射关系，首先会找 TCP/IP 参数中设置的首选 DNS 服务器，在此我们叫它本地 DNS 服务器，此服务器收到查询时，如果要查询的域名，包含在本地配置区域资源中，则返回解析结果给客户机，完成域名解析，此解析具有权威性。

如果要查询的域名，不由本地 DNS 服务器区域解析，但该服务器已缓存了此网址映射关系，则调用这个 IP 地址映射，完成域名解析，此解析不具有权威性。

如果本地 DNS 服务器本地区域文件与缓存解析都失效，则根据本地 DNS 服务器的设置（是否设置转发器）进行查询，如果未用转发模式，本地 DNS 就把请求发至 “根 DNS 服务器”，“根 DNS 服务器” 收到请求后会判断这个域名 (.com) 是谁来授权管理，并会返回一个负责该顶级域名服务器的一个 IP。本地 DNS 服务器收到 IP 信息后，将会联系负责 .com 域的这台服务器。这台负责 .com 域的服务器收到请求后，如果自己无法解析，它就会找一个管理 .com 域的下一级 DNS 服务器地址 (qq.com) 给本地 DNS 服务器。当本地 DNS 服务器收到这个地址后，就会找 qq.com 域服务器，重复上面的动作，进行查询，直至找到 www.qq.com 主机。

如果用的是转发模式，此 DNS 服务器就会把请求转发至上一级 DNS 服务器，由上一级服务器进行解析，上一级服务器如果不能解析，或找根 DNS 或把转请求转至上级，以此循环。不管是本地 DNS 服务器用是是转发，还是根提示，最后都是把结果返回给本地 DNS 服务器，由此 DNS 服务器再返回给客户机。

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/3.1.dns_inquery.png)

所谓 递归查询过程 就是 “查询的递交者” 更替，而 迭代查询过程 则是 “查询的递交者” 不变。

举个例子来说，你想知道某个一起上法律课的女孩的电话，并且你偷偷拍了她的照片，回到寝室告诉一个很仗义的哥们儿，这个哥们儿二话没说，拍着胸脯告诉你，甭急，我替你查 (此处完成了一次递归查询，即，问询者的角色更替)。然后他拿着照片问了学院大四学长，学长告诉他，这姑娘是 xx 系的；然后这哥们儿马不停蹄又问了 xx 系的办公室主任助理同学，助理同学说是 xx 系 yy 班的，然后很仗义的哥们儿去 xx 系 yy 班的班长那里取到了该女孩儿电话。(此处完成若干次迭代查询，即，问询者角色不变，但反复更替问询对象) 最后，他把号码交到了你手里。完成整个查询过程。

通过上面的步骤，我们最后获取的是 IP 地址，也就是浏览器最后发起请求的时候是基于 IP 来和服务器做信息交互的。

**HTTP 协议详解**

HTTP 协议是 Web 工作的核心，所以要了解清楚 Web 的工作方式就需要详细的了解清楚 HTTP 是怎么样工作的。

HTTP 是一种让 Web 服务器与浏览器 (客户端) 通过 Internet 发送与接收数据的协议，它建立在 TCP 协议之上，一般采用 TCP 的 80 端口。它是一个请求、响应协议 -- 客户端发出一个请求，服务器响应这个请求。在 HTTP 中，客户端总是通过建立一个连接与发送一个 HTTP 请求来发起一个事务。服务器不能主动去与客户端联系，也不能给客户端发出一个回调连接。客户端与服务器端都可以提前中断一个连接。例如，当浏览器下载一个文件时，你可以通过点击 “停止” 键来中断文件的下载，关闭与服务器的 HTTP 连接。

**HTTP 协议是无状态的，同一个客户端的这次请求和上次请求是没有对应关系，对 HTTP 服务器来说，它并不知道这两个请求是否来自同一个客户端。为了解决这个问题， Web 程序引入了 Cookie 机制来维护连接的可持续状态。**

HTTP 协议是建立在 TCP 协议之上的，因此 TCP 攻击一样会影响 HTTP 的通讯，例如比较常见的一些攻击：SYN Flood 是当前最流行的 DoS（拒绝服务攻击）与 DdoS（分布式拒绝服务攻击）的方式之一，这是一种利用 TCP 协议缺陷，发送大量伪造的 TCP 连接请求，从而使得被攻击方资源耗尽（CPU 满负荷或内存不足）的攻击方式。

[**HTTP 请求包（浏览器信息）**](https://zh.wikipedia.org/wiki/HTTP%E5%A4%B4%E5%AD%97%E6%AE%B5)

我们先来看看 Request 包的结构，Request 包分为 3 部分，第一部分叫 Request line（请求行）, 第二部分叫 Request header（请求头）, 第三部分是 body（主体）。header 和 body 之间有个空行，请求包的例子所示:

```
GET /domains/example/ HTTP/1.1      // 请求行: 请求方法 请求 URL HTTP 协议/协议版本
Host：www.iana.org               // 服务端的主机名
User-Agent：Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.4 (KHTML, like Gecko) Chrome/22.0.1229.94 Safari/537.4          // 浏览器信息
Accept：text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8  // 客户端能接收的 mine 能够接受的回应内容类型
Accept-Encoding：gzip,deflate,sdch       // 是否支持流压缩 	能够接受的编码方式列表
Accept-Charset：UTF-8,*;q=0.5        // 客户端字符编码集 	能够接受的字符集
// 空行,用于分割请求头和消息体
// 消息体,请求资源参数,例如 POST 传递的参数
```

HTTP 协议定义了很多与服务器交互的请求方法，最基本的有 4 种，分别是 GET, POST, PUT, DELETE。**一个 URL 地址用于描述一个网络上的资源，而 HTTP 中的 GET, POST, PUT, DELETE 就对应着对这个资源的查，增，改，删 4 个操作。**我们最常见的就是 GET 和 POST 了。GET 一般用于获取 / 查询资源信息，而 POST 一般用于更新资源信息。

通过 fiddler 抓包可以看到如下请求信息:

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/3.1.http.png)

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/3.1.httpPOST.png)

我们看看 GET 和 POST 的区别:

1. 我们可以看到 GET 请求消息体为空，POST 请求带有消息体。
2. **GET 提交的数据会放在 URL 之后，以 ? 分割 URL 和传输数据，参数之间以 & 相连**，如 EditPosts.aspx?name=test1&id=123456。**POST 方法是把提交的数据放在 HTTP 包的 body 中**。
3. GET 提交的数据大小有限制（因为浏览器对 URL 的长度有限制），而 POST 方法提交的数据没有限制。
4. GET 方式提交数据，会带来安全问题，比如一个登录页面，通过 GET 方式提交数据时，用户名和密码将出现在 URL 上，如果页面可以被缓存或者其他人可以访问这台机器，就可以从历史记录获得该用户的账号和密码。

**HTTP 响应包（服务器信息）**

我们再来看看 HTTP 的 response 包，他的结构如下：

```
HTTP/1.1 200 OK                     // 状态行
Server: nginx/1.0.8                 // 服务器使用的 WEB 软件名及版本
Date: Tue, 30 Oct 2012 04:14:25 GMT     // 发送时间
Content-Type: text/html             // 服务器发送信息的类型
Transfer-Encoding: chunked          // 表示发送 HTTP 包是分段发的
Connection: keep-alive              // 保持连接状态
Content-Length: 90                  // 主体内容长度
// 空行 用来分割消息头和主体
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"... // 消息体
```


Response 包中的**第一行叫做状态行**，由 HTTP 协议版本号， 状态码， 状态消息三部分组成。


状态码用来告诉 HTTP 客户端，HTTP 服务器是否产生了预期的 Response。HTTP/1.1 协议中定义了 5 类状态码， 状态码由三位数字组成，第一个数字定义了响应的类别

- 1XX 提示信息 - 表示请求已被成功接收，继续处理
- 2XX 成功 - 表示请求已被成功接收，理解，接受
- 3XX 重定向 - 要完成请求必须进行更进一步的处理
- 4XX 客户端错误 - 请求有语法错误或请求无法实现
- 5XX 服务器端错误 - 服务器未能实现合法的请求

我们看下面这个图展示了详细的返回信息，左边可以看到有很多的资源返回码，200 是常用的，表示正常信息，302 表示跳转。response header 里面展示了详细的信息。

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/3.1.response.png)

**HTTP 协议是无状态的和 Connection: keep-alive 的区别**

无状态是指协议对于事务处理没有记忆能力，服务器不知道客户端是什么状态。从另一方面讲，打开一个服务器上的网页和你之前打开这个服务器上的网页之间没有任何联系。

HTTP 是一个无状态的面向连接的协议，无状态不代表 HTTP 不能保持 TCP 连接，更不能代表 HTTP 使用的是 UDP 协议（面对无连接）。

从 HTTP/1.1 起，默认都开启了 Keep-Alive 保持连接特性，简单地说，当一个网页打开完成后，客户端和服务器之间用于传输 HTTP 数据的 TCP 连接不会关闭，如果客户端再次访问这个服务器上的网页，会继续使用这一条已经建立的 TCP 连接。

Keep-Alive 不会永久保持连接，它有一个保持时间，可以在不同服务器软件（如 Apache）中设置这个时间。

请求实例

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/3.1.web.png)

上面这张图我们可以了解到整个的通讯过程，同时细心的读者是否注意到了一点，一个 URL 请求但是左边栏里面为什么会有那么多的资源请求 (这些都是静态文件，go 对于静态文件有专门的处理方式)。

这个就是浏览器的一个功能，第一次请求 url，服务器端返回的是 html 页面，然后浏览器开始渲染 HTML：当解析到 HTML DOM 里面的图片连接，css 脚本和 js 脚本的链接，浏览器就会自动发起一个请求静态资源的 HTTP 请求，获取相对应的静态资源，然后浏览器就会渲染出来，最终将所有资源整合、渲染，完整展现在我们面前的屏幕上。

网页优化方面有一项措施是减少 HTTP 请求次数，就是把尽量多的 css 和 js 资源合并在一起，目的是尽量减少网页请求静态资源的次数，提高网页加载速度，同时减缓服务器的压力。

### Go 搭建一个 Web 服务器

Go 语言里面提供了一个完善的 net/http 包，通过 http 包可以很方便的就搭建起来一个可以运行的 Web 服务。同时使用这个包能很简单地对 Web 的路由，静态文件，模版，cookie 等数据进行设置和操作。

```

package main

import (
    "fmt"
    "net/http"
    "strings"
    "log"
)

func sayhelloName(w http.ResponseWriter, r *http.Request) {
    r.ParseForm()  // 解析参数，默认是不会解析的
    fmt.Println(r.Form)  // 这些信息是输出到服务器端的打印信息
    fmt.Println("path", r.URL.Path)
    fmt.Println("scheme", r.URL.Scheme)
    fmt.Println(r.Form["url_long"])
    for k, v := range r.Form {
        fmt.Println("key:", k)
        fmt.Println("val:", strings.Join(v, ""))
    }
    fmt.Fprintf(w, "Hello astaxie!") // 这个写入到 w 的是输出到客户端的
}

func main() {
    http.HandleFunc("/", sayhelloName) // 设置访问的路由
    err := http.ListenAndServe(":9090", nil) // 设置监听的端口
    if err != nil {
        log.Fatal("ListenAndServe: ", err)
    }
}
```

上面这个代码，我们 build 之后，然后执行 web.exe, 这个时候其实已经在 9090 端口监听 http 链接请求了。

在浏览器输入 http://localhost:9090

可以看到浏览器页面输出了 Hello astaxie!

可以换一个地址试试：http://localhost:9090/?url_long=111&url_long=222

看看浏览器输出的是什么，服务器输出的是什么？

在服务器端输出的信息如下：

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/202111281951334.png)

我们看到上面的代码，要编写一个 Web 服务器很简单，只要调用 http 包的两个函数就可以了。

如果你以前是 PHP 程序员，那你也许就会问，我们的 nginx、apache 服务器不需要吗？Go 就是不需要这些，因为他直接就监听 tcp 端口了，做了 nginx 做的事情，然后 sayhelloName 这个其实就是我们写的逻辑函数了，跟 php 里面的控制层（controller）函数类似。

如果你以前是 Python 程序员，那么你一定听说过 tornado，这个代码和他是不是很像，对，没错，Go 就是拥有类似 Python 这样动态语言的特性，写 Web 应用很方便。

如果你以前是 Ruby 程序员，会发现和 ROR 的 /script/server 启动有点类似。

我们看到 Go 通过简单的几行代码就已经运行起来一个 Web 服务了，而且这个 Web 服务内部有支持高并发的特性，我将会在接下来的两个小节里面详细的讲解一下 Go 是如何实现 Web 高并发的。

### Go 如何使得 Web 工作

前面小节介绍了如何通过 Go 搭建一个 Web 服务，我们可以看到简单应用一个 net/http 包就方便的搭建起来了。那么 Go 在底层到底是怎么做的呢？万变不离其宗，Go 的 Web 服务工作也离不开我们第一小节介绍的 Web 工作方式。

**web 工作方式的几个概念**
以下均是服务器端的几个概念

Request：用户请求的信息，用来解析用户的请求信息，包括 post、get、cookie、url 等信息

Response：服务器需要反馈给客户端的信息

Conn：用户的每次请求链接

Handler：处理请求和生成返回信息的处理逻辑

**分析 http 包运行机制**

如下图所示，是 Go 实现 Web 服务的工作模式的流程图

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/202111281952706.png)

1. 创建 Listen Socket, 监听指定的端口，等待客户端请求到来。
2. Listen Socket 接受客户端的请求，得到 Client Socket, 接下来通过 Client Socket 与客户端通信。
3. 处理客户端的请求，首先从 Client Socket 读取 HTTP 请求的协议头，如果是 POST 方法，还可能要读取客户端提交的数据，然后交给相应的 handler 处理请求，handler 处理完毕准备好客户端需要的数据，通过 Client Socket 写给客户端。

这整个的过程里面我们只要了解清楚下面三个问题，也就知道 Go 是如何让 Web 运行起来了

- 如何监听端口？
- 如何接收客户端请求？
- 如何分配 handler？

前面小节的代码里面我们可以看到，Go 是通过一个函数 ListenAndServe 来处理这些事情的，这个底层其实这样处理的：初始化一个 server 对象，然后调用了 net.Listen("tcp", addr)，也就是底层用 TCP 协议搭建了一个服务，然后监控我们设置的端口。

下面代码来自 Go 的 http 包的源码，通过下面的代码我们可以看到整个的 http 处理过程：

```
func (srv *Server) Serve(l net.Listener) error { // 表示Server类型有Serve方法
    defer l.Close()
    var tempDelay time.Duration // how long to sleep on accept failure
    for {
        rw, e := l.Accept()
        if e != nil {
            if ne, ok := e.(net.Error); ok && ne.Temporary() {
                if tempDelay == 0 {
                    tempDelay = 5 * time.Millisecond
                } else {
                    tempDelay *= 2
                }
                if max := 1 * time.Second; tempDelay > max {
                    tempDelay = max
                }
                log.Printf("http: Accept error: %v; retrying in %v", e, tempDelay)
                time.Sleep(tempDelay)
                continue
            }
            return e
        }
        tempDelay = 0
        c, err := srv.newConn(rw)
        if err != nil {
            continue
        }
        go c.serve()
    }
}
```


监控之后如何接收客户端的请求呢？上面代码执行监控端口之后，调用了 srv.Serve(net.Listener) 函数，这个函数就是处理接收客户端的请求信息。这个函数里面起了一个 for{}，首先通过 Listener 接收请求，其次创建一个 Conn，最后单独开了一个 goroutine，把这个请求的数据当做参数扔给这个 conn 去服务：go c.serve()。这个就是高并发体现了，用户的每一次请求都是在一个新的 goroutine 去服务，相互不影响。

那么如何具体分配到相应的函数来处理请求呢？conn 首先会解析 request:c.readRequest(), 然后获取相应的 handler:handler := c.server.Handler，也就是我们刚才在调用函数 ListenAndServe 时候的第二个参数，我们前面例子传递的是 nil，也就是为空，那么默认获取 handler = DefaultServeMux, 那么这个变量用来做什么的呢？对，**这个变量就是一个路由器，它用来匹配 url 跳转到其相应的 handle 函数**，那么这个我们有设置过吗？有，我们调用的代码里面第一句不是调用了 http.HandleFunc("/", sayhelloName) 嘛。**这个作用就是注册了请求 / 的路由规则**，当请求 uri 为 "/"，路由就会转到函数 sayhelloName，DefaultServeMux 会调用 ServeHTTP 方法，这个方法内部其实就是调用 sayhelloName 本身，最后通过写入 response 的信息反馈到客户端。

详细的整个流程如下图所示：

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/202111281952267.png)

### Go 的 http 包详解

Go 的 http 有两个核心功能：Conn、ServeMux

**Conn 的 goroutine**

与我们一般编写的 http 服务器不同，Go 为了实现高并发和高性能，使用了 goroutines 来处理 Conn 的读写事件，这样每个请求都能保持独立，相互不会阻塞，可以高效的响应网络事件。这是 Go 高效的保证。

Go 在等待客户端请求里面是这样写的：

```
c, err := srv.newConn(rw)
if err != nil {
    continue
}
go c.serve()
```


这里我们可以看到客户端的每次请求都会创建一个 Conn，这个 Conn 里面保存了该次请求的信息，然后再传递到对应的 handler，该 handler 中便可以读取到相应的 header 信息，这样保证了每个请求的独立性。

**ServeMux 的自定义**

我们前面小节讲述 conn.server 的时候，其实内部是调用了 http 包默认的路由器，通过路由器把本次请求的信息传递到了后端的处理函数。那么这个路由器是怎么实现的呢？

它的结构如下：

```
type ServeMux struct {
    mu sync.RWMutex   // 锁，由于请求涉及到并发处理，因此这里需要一个锁机制
    m  map[string]muxEntry  // 路由规则，一个 string 对应一个 mux 实体，这里的 string 就是注册的路由表达式
    hosts bool // 是否在任意的规则中带有 host 信息
}
```


下面看一下 muxEntry

```
type muxEntry struct {
    explicit bool   // 是否精确匹配
    h        Handler // 这个路由表达式对应哪个 handler
    pattern  string  // 匹配字符串
}
```

接着看一下 Handler 的定义

```
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)  // 路由实现器
}
```

**Handler 是一个接口**，但是前一小节中的 sayhelloName 函数并没有实现 ServeHTTP 这个接口，为什么能添加呢？原来在 http 包里面还定义了一个类型 HandlerFunc, 我们定义的函数 sayhelloName 就是这个 HandlerFunc 调用之后的结果，这个类型默认就实现了 ServeHTTP 这个接口，即我们调用了 HandlerFunc (f), 强制类型转换 f 成为 HandlerFunc 类型，这样 f 就拥有了 ServeHTTP 方法。

```
type HandlerFunc func(ResponseWriter, *Request)

// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```

路由器里面存储好了相应的路由规则之后，那么具体的请求又是怎么分发的呢？请看下面的代码，默认的路由器实现了 ServeHTTP：

```
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request) {
    if r.RequestURI == "*" {
        w.Header().Set("Connection", "close")
        w.WriteHeader(StatusBadRequest)
        return
    }
    h, _ := mux.Handler(r)
    h.ServeHTTP(w, r)
}
```


如上所示路由器接收到请求之后，如果是 * 那么关闭链接，不然调用 mux.Handler(r) 返回对应设置路由的处理 Handler，然后执行 h.ServeHTTP(w, r)

也就是调用对应路由的 handler 的 ServerHTTP 接口，那么 mux.Handler (r) 怎么处理的呢？

```
func (mux *ServeMux) Handler(r *Request) (h Handler, pattern string) {
    if r.Method != "CONNECT" {
        if p := cleanPath(r.URL.Path); p != r.URL.Path {
            _, pattern = mux.handler(r.Host, p)
            return RedirectHandler(p, StatusMovedPermanently), pattern
        }
    }   
    return mux.handler(r.Host, r.URL.Path)
}

func (mux *ServeMux) handler(host, path string) (h Handler, pattern string) {
    mux.mu.RLock()
    defer mux.mu.RUnlock()
    // Host-specific pattern takes precedence over generic ones
    if mux.hosts {
        h, pattern = mux.match(host + path)
    }
    if h == nil {
        h, pattern = mux.match(path)
    }
    if h == nil {
        h, pattern = NotFoundHandler(), ""
    }
    return
}
```


原来他是根据用户请求的 URL 和路由器里面存储的 map 去匹配的，当匹配到之后返回存储的 handler，调用这个 handler 的 ServeHTTP 接口就可以执行到相应的函数了。

通过上面这个介绍，我们了解了整个路由过程，Go 其实支持外部实现的路由器 ListenAndServe 的第二个参数就是用以配置外部路由器的，它是一个 Handler 接口，即外部路由器只要实现了 Handler 接口就可以，我们可以在自己实现的路由器的 ServeHTTP 里面实现自定义路由功能。

如下代码所示，我们自己实现了一个简易的路由器

```
package main

import (
    "fmt"
    "net/http"
)

type MyMux struct {
}

func (p *MyMux) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    if r.URL.Path == "/" {
        sayhelloName(w, r)
        return
    }
    http.NotFound(w, r)
    return
}

func sayhelloName(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello myroute!")
}

func main() {
    mux := &MyMux{}
    http.ListenAndServe(":9090", mux)
}
```

**Go 代码的执行流程**

通过对 http 包的分析之后，现在让我们来梳理一下整个的代码执行过程。

- 首先调用 Http.HandleFunc

  按顺序做了几件事：

  1 调用了 DefaultServeMux 的 HandleFunc

  2 调用了 DefaultServeMux 的 Handle

  3 往 DefaultServeMux 的 map [string] muxEntry 中增加对应的 handler 和路由规则

- 其次调用 http.ListenAndServe (":9090", nil)

  按顺序做了几件事情：

  1 实例化 Server

  2 调用 Server 的 ListenAndServe ()

  3 调用 net.Listen ("tcp", addr) 监听端口

  4 启动一个 for 循环，在循环体中 Accept 请求

  5 对每个请求实例化一个 Conn，并且开启一个 goroutine 为这个请求进行服务 go c.serve ()

  6 读取每个请求的内容 w, err := c.readRequest ()

  7 判断 handler 是否为空，如果没有设置 handler（这个例子就没有设置 handler），handler 就设置为 DefaultServeMux

  8 调用 handler 的 ServeHttp

  9 在这个例子中，下面就进入到 DefaultServeMux.ServeHttp

  10 根据 request 选择 handler，并且进入到这个 handler 的 ServeHTTP

  ```
  mux.handler(r).ServeHTTP(w, r)
  ```

  11 选择 handler：

  ​	A 判断是否有路由能满足这个 request（循环遍历 ServeMux 的 muxEntry）

  ​	B 如果有路由满足，调用这个路由 handler 的 ServeHTTP

  ​	C 如果没有路由满足，调用 NotFoundHandler 的 ServeHTTP

## 二、表单

表单是我们平常编写 Web 应用常用的工具，通过表单我们可以方便的让客户端和服务器进行数据的交互。对于以前开发过 Web 的用户来说表单都非常熟悉，但是对于 C/C++ 程序员来说，这可能是一个有些陌生的东西，那么什么是表单呢？

表单是一个包含表单元素的区域。表单元素是允许用户在表单中（比如：文本域、下拉列表、单选框、复选框等等）输入信息的元素。表单使用表单标签（\<form>）定义。

```
<form>
...
input 元素
...
</form>
```

Go 里面对于 form 处理已经有很方便的方法了，在 Request 里面的有专门的 form 处理，可以很方便的整合到 Web 开发里面来，

HTTP 协议是一种无状态的协议，那么如何才能辨别是否是同一个用户呢？同时又如何保证一个表单不出现多次递交的情况呢？

### 处理表单的输入

先来看一个表单递交的例子，我们有如下的表单内容，命名成文件 login.gtpl (放入当前新建项目的目录里面)

```
<html>

<head>
<title></title>
</head>

<body>

<form action="/login" method="post">
    用户名:<input type="text" name="username">
    密码:<input type="password" name="password">
    <input type="submit" value="登录">
</form>

</body>
</html>
```

上面递交表单到服务器的 /login，当用户输入信息点击登录之后，会跳转到服务器的路由 login 里面，我们首先要判断这个是什么方式传递过来，POST 还是 GET 呢？

http 包里面有一个很简单的方式就可以获取，我们在前面 web 的例子的基础上来看看怎么处理 login 页面的 form 数据

```
package main

import (
    "fmt"
    "html/template"
    "log"
    "net/http"
    "strings"
)

func sayhelloName(w http.ResponseWriter, r *http.Request) {
    r.ParseForm()       // 解析 url 传递的参数，对于 POST 则解析响应包的主体（request body）
    // 注意:如果没有调用 ParseForm 方法，下面无法获取表单的数据
    fmt.Println(r.Form) // 这些信息是输出到服务器端的打印信息
    fmt.Println("path", r.URL.Path)
    fmt.Println("scheme", r.URL.Scheme)
    fmt.Println(r.Form["url_long"])
    for k, v := range r.Form {
        fmt.Println("key:", k)
        fmt.Println("val:", strings.Join(v, ""))
    }
    fmt.Fprintf(w, "Hello astaxie!") // 这个写入到 w 的是输出到客户端的
}

func login(w http.ResponseWriter, r *http.Request) {
    fmt.Println("method:", r.Method) // 获取请求的方法
    if r.Method == "GET" {
        t, _ := template.ParseFiles("login.gtpl")
        log.Println(t.Execute(w, nil))
    } else {
        err := r.ParseForm()   // 解析 url 传递的参数，对于 POST 则解析响应包的主体（request body）
        if err != nil {
           // handle error http.Error() for example
          log.Fatal("ParseForm: ", err)
        }
        // 请求的是登录数据，那么执行登录的逻辑判断
        fmt.Println("username:", r.Form["username"])
        fmt.Println("password:", r.Form["password"])
    }
}

func main() {
    http.HandleFunc("/", sayhelloName)       // 设置访问的路由
    http.HandleFunc("/login", login)         // 设置访问的路由
    err := http.ListenAndServe(":9090", nil) // 设置监听的端口
    if err != nil {
        log.Fatal("ListenAndServe: ", err)
    }
}
```

通过上面的代码我们可以看出获取请求方法是通过 r.Method 来完成的，这是个字符串类型的变量，返回 GET, POST, PUT 等 method 信息。

login 函数中我们根据 r.Method 来判断是显示登录界面还是处理登录逻辑。当 GET 方式请求时显示登录界面，其他方式请求时则处理登录逻辑，如查询数据库、验证登录信息等。

当我们在浏览器里面打开 http://127.0.0.1:9090/login 的时候，出现如下界面

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/4.1.login.png)

如果你看到一个空页面，可能是你写的 login.gtpl 文件中有错误，请根据控制台中的日志进行修复。



我们输入用户名和密码之后发现在服务器端是不会打印出来任何输出的，为什么呢？默认情况下，Handler 里面是不会自动解析 form 的，必须显式的调用 r.ParseForm() 后，你才能对这个表单数据进行操作。我们修改一下代码，在 fmt.Println("username:", r.Form["username"]) 之前加一行 r.ParseForm(), 重新编译，再次测试输入递交，现在是不是在服务器端有输出你的输入的用户名和密码了。

r.Form 里面包含了所有请求的参数，比如 URL 中 query-string、POST 的数据、PUT 的数据，所以当你在 URL 中的 query-string 字段和 POST 冲突时，会保存成一个 slice，里面存储了多个值，Go 官方文档中说在接下来的版本里面将会把 POST、GET 这些数据分离开来。

现在我们修改一下 login.gtpl 里面 form 的 action 值 http://127.0.0.1:9090/login 修改为 http://127.0.0.1:9090/login?username=astaxie，再次测试，服务器的输出 username 是不是一个 slice。服务器端的输出如下：

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/4.1.slice.png)

图 4.2 服务器端打印接受到的信息

request.Form 是一个 url.Values 类型，里面存储的是对应的类似 key=value 的信息，下面展示了可以对 form 数据进行的一些操作:

```
v := url.Values{}
v.Set("name", "Ava")
v.Add("friend", "Jess")
v.Add("friend", "Sarah")
v.Add("friend", "Zoe")
// v.Encode() == "name=Ava&friend=Jess&friend=Sarah&friend=Zoe"
fmt.Println(v.Get("name"))
fmt.Println(v.Get("friend"))
fmt.Println(v["friend"])
```

Tips:
Request 本身也提供了 FormValue () 函数来获取用户提交的参数。如 r.Form ["username"] 也可写成 r.FormValue ("username")。调用 r.FormValue 时会自动调用 r.ParseForm，所以不必提前调用。r.FormValue 只会返回同名参数中的第一个，若参数不存在则返回空字符串。

### 预防跨站脚本

现在的网站包含大量的动态内容以提高用户体验，比过去要复杂得多。所谓动态内容，就是根据用户环境和需要，Web 应用程序能够输出相应的内容。动态站点会受到一种名为 “跨站脚本攻击”（Cross Site Scripting, 安全专家们通常将其缩写成 XSS）的威胁，而静态站点则完全不受其影响。

攻击者通常会在有漏洞的程序中插入 JavaScript、VBScript、 ActiveX 或 Flash 以欺骗用户。一旦得手，他们可以盗取用户帐户信息，修改用户设置，盗取 / 污染 cookie 和植入恶意广告等。

**对 XSS 最佳的防护应该结合以下两种方法：一是验证所有输入数据，有效检测攻击 (这个我们前面小节已经有过介绍); 另一个是对所有输出数据进行适当的处理，以防止任何已成功注入的脚本在浏览器端运行。**

那么 Go 里面是怎么做这个有效防护的呢？Go 的 html/template 里面带有下面几个函数可以帮你转义

```
func HTMLEscape (w io.Writer, b [] byte) // 把 b 进行转义之后写到 w
func HTMLEscapeString (s string) string // 转义 s 之后返回结果字符串
func HTMLEscaper (args ...interface {}) string // 支持多个参数一起转义，返回结果字符串
```

我们看 4.1 小节的例子

```
fmt.Println("username:", template.HTMLEscapeString(r.Form.Get("username"))) // 输出到服务器端
fmt.Println("password:", template.HTMLEscapeString(r.Form.Get("password")))
template.HTMLEscape(w, []byte(r.Form.Get("username"))) // 输出到客户端
```


如果我们输入的 username 是 `<script>alert()</script>`, 那么我们可以在浏览器上面看到输出如下所示：

![](https://cdn.learnku.com/build-web-application-with-golang/images/4.3.escape.png)

图 4.3 Javascript 过滤之后的输出

Go 的 html/template 包默认帮你过滤了 html 标签，但是有时候你只想要输出这个 `<script>alert()</script> `看起来正常的信息，该怎么处理？请使用 text/template。请看下面的例子：

```
import "text/template"
...
t, err := template.New("foo").Parse(`{{define "T"}}Hello, {{.}}!{{end}}`)
err = t.ExecuteTemplate(out, "T", "<script>alert('you have been pwned')</script>")
```


输出

```
Hello, <script>alert('you have been pwned')</script>!
```

或者使用 template.HTML 类型

```
import "html/template"
...
t, err := template.New("foo").Parse(`{{define "T"}}Hello, {{.}}!{{end}}`)
err = t.ExecuteTemplate(out, "T", template.HTML("<script>alert('you have been pwned')</script>"))
```


输出

```
Hello, <script>alert('you have been pwned')</script>!
```

转换成 template.HTML 后，变量的内容也不会被转义

转义的例子：

```
import "html/template"
...
t, err := template.New("foo").Parse(`{{define "T"}}Hello, {{.}}!{{end}}`)
err = t.ExecuteTemplate(out, "T", "<script>alert('you have been pwned')</script>")
```


转义之后的输出：

```
Hello, <script>alert('you have been pwned')</script>!
```

### 验证表单的输入

开发 Web 的一个原则就是，不能信任用户输入的任何信息，所以验证和过滤用户的输入信息就变得非常重要，我们经常会在微博、新闻中听到某某网站被入侵了，存在什么漏洞，这些大多是因为网站对于用户输入的信息没有做严格的验证引起的，所以为了编写出安全可靠的 Web 程序，验证表单输入的意义重大。

我们平常编写 Web 应用主要有两方面的数据验证，**一个是在页面端的 js 验证** (目前在这方面有很多的插件库，比如 ValidationJS 插件)，**一个是在服务器端的验证**，我们这小节讲解的是如何在服务器端验证。

**必填字段**
你想要确保从一个表单元素中得到一个值，例如前面小节里面的用户名，我们如何处理呢？Go 有一个内置函数 len 可以获取字符串的长度，这样我们就可以通过 len 来获取数据的长度，例如：

```
if len(r.Form["username"][0])==0{
    // 为空的处理
}
```


r.Form 对不同类型的表单元素的留空有不同的处理， 对于空文本框、空文本区域以及文件上传，元素的值为空值，而如果是未选中的复选框和单选按钮，则根本不会在 r.Form 中产生相应条目，如果我们用上面例子中的方式去获取数据时程序就会报错。所以我们需要通过 r.Form.Get() 来获取值，因为如果字段不存在，通过该方式获取的是空值。但是通过 r.Form.Get() 只能获取单个的值，如果是 map 的值，必须通过上面的方式来获取。

**数字**

你想要确保一个表单输入框中获取的只能是数字，例如，你想通过表单获取某个人的具体年龄是 50 岁还是 10 岁，而不是像 “一把年纪了” 或 “年轻着呢” 这种描述

如果我们是判断正整数，那么我们先转化成 int 类型，然后进行处理

```
getint,err:=strconv.Atoi(r.Form.Get("age")) //strconv包提供了字符串与简单数据类型之间的类型转换功能。字符串转int：Atoi()
if err!=nil{
    // 数字转化出错了，那么可能就不是数字
}

// 接下来就可以判断这个数字的大小范围了
if getint >100 {
    // 太大了
}
还有一种方式就是正则匹配的方式

if m, _ := regexp.MatchString("^[0-9]+$", r.Form.Get("age")); !m {
    return false
}
```


对于性能要求很高的用户来说，这是一个老生常谈的问题了，他们认为应该尽量避免使用正则表达式，因为使用正则表达式的速度会比较慢。但是在目前机器性能那么强劲的情况下，对于这种简单的正则表达式效率和类型转换函数是没有什么差别的。如果你对正则表达式很熟悉，而且你在其它语言中也在使用它，那么在 Go 里面使用正则表达式将是一个便利的方式。

Go 实现的正则是 RE2，所有的字符都是 UTF-8 编码的。

**中文**

有时候我们想通过表单元素获取一个用户的中文名字，但是又为了保证获取的是正确的中文，我们需要进行验证，而不是用户随便的一些输入。对于中文我们目前有两种方式来验证，可以使用 unicode 包提供的 func Is(rangeTab *RangeTable, r rune) bool 来验证，也可以使用正则方式来验证，这里使用最简单的正则方式，如下代码所示

```
if m, _ := regexp.MatchString("^\\p{Han}+$", r.Form.Get("realname")); !m {
    return false
}
```

**英文**
我们期望通过表单元素获取一个英文值，例如我们想知道一个用户的英文名，应该是 astaxie，而不是 asta 谢。

我们可以很简单的通过正则验证数据：

```
if m, _ := regexp.MatchString("^[a-zA-Z]+$", r.Form.Get("engname")); !m {
    return false
}
```

**电子邮件地址**
你想知道用户输入的一个 Email 地址是否正确，通过如下这个方式可以验证：

```
if m, _ := regexp.MatchString(`^([\w\.\_]{2,10})@(\w{1,})\.([a-z]{2,4})$`, r.Form.Get("email")); !m {
    fmt.Println("no")
}else{
    fmt.Println("yes")
}
```

**手机号码**
你想要判断用户输入的手机号码是否正确，通过正则也可以验证：

```
if m, _ := regexp.MatchString(`^(1[3|4|5|8][0-9]\d{4,8})$`, r.Form.Get("mobile")); !m {
    return false
}
```

**下拉菜单**
如果我们想要判断表单里面 <select> 元素生成的下拉菜单中是否有被选中的项目。有些时候黑客可能会伪造这个下拉菜单不存在的值发送给你，那么如何判断这个值是否是我们预设的值呢？

我们的 select 可能是这样的一些元素

```
<select name="fruit">
<option value="apple">apple</option>
<option value="pear">pear</option>
<option value="banana">banana</option>
</select>
```

那么我们可以这样来验证

```
slice:=[]string{"apple","pear","banana"}

v := r.Form.Get("fruit")
for _, item := range slice {
    if item == v {
        return true
    }
}

return false
```

**单选按钮**

如果我们想要判断 radio 按钮是否有一个被选中了，我们页面的输出可能就是一个男、女性别的选择，但是也可能一个 15 岁大的无聊小孩，一手拿着 http 协议的书，另一只手通过 telnet 客户端向你的程序在发送请求呢，你设定的性别男值是 1，女是 2，他给你发送一个 3，你的程序会出现异常吗？因此我们也需要像下拉菜单的判断方式类似，判断我们获取的值是我们预设的值，而不是额外的值。

```
<input type="radio" name="gender" value="1">男
<input type="radio" name="gender" value="2">女
```


那我们也可以类似下拉菜单的做法一样

```
slice:=[]string{"1","2"}

for _, v := range slice {
    if v == r.Form.Get("gender") {
        return true
    }
}
return false
```

**复选框**
有一项选择兴趣的复选框，你想确定用户选中的和你提供给用户选择的是同一个类型的数据。

```
<input type="checkbox" name="interest" value="football">足球
<input type="checkbox" name="interest" value="basketball">篮球
<input type="checkbox" name="interest" value="tennis">网球
```


对于复选框我们的验证和单选有点不一样，因为接收到的数据是一个 slice

```
slice:=[]string{"football","basketball","tennis"}
a:=Slice_diff(r.Form["interest"],slice)
if a == nil{
    return true
}

return false
```

上面这个函数 Slice_diff 包含在我开源的一个库里面 (操作 slice 和 map 的库)，github.com/astaxie/beeku

**日期和时间**

你想确定用户填写的日期或时间是否有效。例如，用户在日程表中安排 8 月份的第 45 天开会，或者提供未来的某个时间作为生日。

Go 里面提供了一个 time 的处理包，我们可以把用户的输入年月日转化成相应的时间，然后进行逻辑判断

```
t := time.Date(2009, time.November, 10, 23, 0, 0, 0, time.Local)
fmt.Printf("Go launched at %s\n", t.Local())
```


获取 time 之后我们就可以进行很多时间函数的操作。具体的判断就根据自己的需求调整。

**身份证号码**
如果我们想验证表单输入的是否是身份证，通过正则也可以方便的验证，但是身份证有 15 位和 18 位，我们两个都需要验证

```
// 验证 15 位身份证，15 位的是全部数字
if m, _ := regexp.MatchString(`^(\d{15})$`, r.Form.Get("usercard")); !m {
    return false
}
```

// 验证 18 位身份证，18 位前 17 位为数字，最后一位是校验位，可能为数字或字符 X。

```
if m, _ := regexp.MatchString(`^(\d{17})([0-9]|X)$`, r.Form.Get("usercard")); !m {
    return false
}
```

上面列出了我们一些常用的服务器端的表单元素验证，希望通过这个引导入门，能够让你对 Go 的数据验证有所了解，特别是 Go 里面的正则处理。

### 防止多次递交表单

解决方案是在表单中添加一个带有唯一值的隐藏字段。在验证表单时，先检查带有该唯一值的表单是否已经递交过了。如果是，拒绝再次递交；如果不是，则处理表单进行逻辑处理。另外，如果是采用了 Ajax 模式递交表单的话，当表单递交后，通过 javascript 来禁用表单的递交按钮。

我继续拿 4.2 小节的例子优化：

```
<input type="checkbox" name="interest" value="football">足球
<input type="checkbox" name="interest" value="basketball">篮球
<input type="checkbox" name="interest" value="tennis">网球    
用户名:<input type="text" name="username">
密码:<input type="password" name="password">
<input type="hidden" name="token" value="{{.}}">
<input type="submit" value="登录">
```


我们在模版里面增加了一个隐藏字段 token，这个值我们通过 MD5 (时间戳) 来获取唯一值，然后我们把这个值存储到服务器端 (session 来控制，我们将在第六章讲解如何保存)，以方便表单提交时比对判定。

```
func login(w http.ResponseWriter, r *http.Request) {
    fmt.Println("method:", r.Method) // 获取请求的方法
    if r.Method == "GET" {
        crutime := time.Now().Unix()
        h := md5.New()
        io.WriteString(h, strconv.FormatInt(crutime, 10))
        token := fmt.Sprintf("%x", h.Sum(nil))
        t, _ := template.ParseFiles("login.gtpl")
    	t.Execute(w, token)
	} else {
        // 请求的是登录数据，那么执行登录的逻辑判断
        r.ParseForm()
        token := r.Form.Get("token")
        if token != "" {
            // 验证 token 的合法性
        } else {
            // 不存在 token 报错
        }
        fmt.Println("username length:", len(r.Form["username"][0]))
        fmt.Println("username:", template.HTMLEscapeString(r.Form.Get("username"))) // 输出到服务器端
        fmt.Println("password:", template.HTMLEscapeString(r.Form.Get("password")))
        template.HTMLEscape(w, []byte(r.Form.Get("username"))) // 输出到客户端
    }
}
```

上面的代码输出到页面的源码如下：

![](https://cdn.learnku.com/build-web-application-with-golang/images/4.4.token.png)

图 4.4 增加 token 之后在客户端输出的源码信息

我们看到 token 已经有输出值，你可以不断的刷新，可以看到这个值在不断的变化。这样就保证了每次显示 form 表单的时候都是唯一的，用户递交的表单保持了唯一性。

我们的解决方案可以防止非恶意的攻击，并能使恶意用户暂时不知所措，然后，它却不能排除所有的欺骗性的动机，对此类情况还需要更复杂的工作。

### 处理文件上传

要使表单能够上传文件，首先第一步就是要添加 form 的 enctype 属性，enctype 属性有如下三种情况:

```
application/x-www-form-urlencoded   表示在发送前编码所有字符（默认）
multipart/form-data   不对字符编码。在使用包含文件上传控件的表单时，必须使用该值。
text/plain    空格转换为 "+" 加号，但不对特殊字符编码。
```

所以，创建新的表单 html 文件，命名为 upload.gtpl, html 代码应该类似于:

```
<html>

<head>
    <title>上传文件</title>
</head>

<body>

<form enctype="multipart/form-data" action="/upload" method="post">
  <input type="file" name="uploadfile" />
  <input type="hidden" name="token" value="{{.}}"/>
  <input type="submit" value="upload" />
</form>

</body>
</html>
```

在服务器端，我们增加一个 handlerFunc:

    http.HandleFunc("/upload", upload)
    
    // 处理 /upload  逻辑
    func upload(w http.ResponseWriter, r *http.Request) {
        fmt.Println("method:", r.Method) // 获取请求的方法
        if r.Method == "GET" {
            crutime := time.Now().Unix()
            h := md5.New()
            io.WriteString(h, strconv.FormatInt(crutime, 10))
            token := fmt.Sprintf("%x", h.Sum(nil))
            t, _ := template.ParseFiles("upload.gtpl")
            t.Execute(w, token)
        } else {
            r.ParseMultipartForm(32 << 20)
            file, handler, err := r.FormFile("uploadfile")
            if err != nil {
                fmt.Println(err)
                return
            }
            defer file.Close()
            fmt.Fprintf(w, "%v", handler.Header)
            f, err := os.OpenFile("./test/"+handler.Filename, os.O_WRONLY|os.O_CREATE, 0666)  // 此处假设当前目录下已存在test目录
            if err != nil {
                fmt.Println(err)
                return
            }
            defer f.Close()
            io.Copy(f, file)
        }
    }
通过上面的代码可以看到，处理文件上传我们需要调用 r.ParseMultipartForm，里面的参数表示 maxMemory，调用 ParseMultipartForm 之后，上传的文件存储在 maxMemory 大小的内存里面，如果文件大小超过了 maxMemory，那么剩下的部分将存储在系统的临时文件中。我们可以通过 r.FormFile 获取上面的文件句柄，然后实例中使用了 io.Copy 来存储文件。

获取其他非文件字段信息的时候就不需要调用 r.ParseForm，因为在需要的时候 Go 自动会去调用。而且 ParseMultipartForm 调用一次之后，后面再次调用不会再有效果。

通过上面的实例我们可以看到我们上传文件主要三步处理：

1. 表单中增加 enctype="multipart/form-data"
2. 服务端调用 r.ParseMultipartForm, 把上传的文件存储在内存和临时文件中
3. 使用 r.FormFile 获取文件句柄，然后对文件进行存储等处理。

文件 handler 是 multipart.FileHeader, 里面存储了如下结构信息

```
type FileHeader struct {
    Filename string
    Header   textproto.MIMEHeader
    // contains filtered or unexported fields
}
```

我们通过上面的实例代码打印出来上传文件的信息如下

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/4.5.upload2.png)

图 4.5 打印文件上传后服务器端接受的信息

客户端上传文件
我们上面的例子演示了如何通过表单上传文件，然后在服务器端处理文件，其实 Go 支持模拟客户端表单功能支持文件上传，详细用法请看如下示例：

    package main
    
    import (
        "bytes"
        "fmt"
        "io"
        "io/ioutil"
        "mime/multipart"
        "net/http"
        "os"
    )
    
    func postFile(filename string, targetUrl string) error {
        bodyBuf := &bytes.Buffer{}
        bodyWriter := multipart.NewWriter(bodyBuf)
        // 关键的一步操作
        fileWriter, err := bodyWriter.CreateFormFile("uploadfile", filename)
        if err != nil {
            fmt.Println("error writing to buffer")
            return err
        }
    
        // 打开文件句柄操作
        fh, err := os.Open(filename)
        if err != nil {
            fmt.Println("error opening file")
            return err
        }
        defer fh.Close()
    
        // iocopy
        _, err = io.Copy(fileWriter, fh)
        if err != nil {
            return err
        }
    
        contentType := bodyWriter.FormDataContentType()
        bodyWriter.Close()
    
        resp, err := http.Post(targetUrl, contentType, bodyBuf)
        if err != nil {
            return err
        }
        defer resp.Body.Close()
        resp_body, err := ioutil.ReadAll(resp.Body)
        if err != nil {
            return err
        }
        fmt.Println(resp.Status)
        fmt.Println(string(resp_body))
        return nil
      	// sample usage
    func main() {
        target_url := "http://localhost:9090/upload"
        filename := "./astaxie.pdf"
        postFile(filename, target_url)
    } 
上面的例子详细展示了客户端如何向服务器上传一个文件的例子，客户端通过 multipart.Write 把文件的文本流写入一个缓存中，然后调用 http 的 Post 方法把缓存传到服务器。

如果你还有其他普通字段例如 username 之类的需要同时写入，那么可以调用 multipart 的 WriteField 方法写很多其他类似的字段。

## 三、数据库

### 接口

 Go 官方没有提供数据库驱动，而是为开发数据库驱动定义了一些标准接口，开发者可以根据定义的接口来开发相应的数据库驱动，这样做有一个好处，只要是按照标准接口开发的代码， 以后需要迁移数据库时，不需要任何修改。那么 Go 都定义了哪些标准接口呢

**sql.Register**

这个存在于 database/sql 的函数是用来注册数据库驱动的，当第三方开发者开发数据库驱动时，都会实现 init 函数，在 init 里面会调用这个 Register(name string, driver driver.Driver) 完成本驱动的注册。

我们来看一下 mymysql、sqlite3 的驱动里面都是怎么调用的：

```
// https://github.com/mattn/go-sqlite3 驱动
func init() {
    sql.Register("sqlite3", &SQLiteDriver{})
}

// https://github.com/mikespook/mymysql 驱动
// Driver automatically registered in database/sql
var d = Driver{proto: "tcp", raddr: "127.0.0.1:3306"}
func init() {
    Register("SET NAMES utf8")
    sql.Register("mymysql", &d)
}
```

我们看到第三方数据库驱动都是通过调用这个函数来注册自己的数据库驱动名称以及相应的 driver 实现。在 database/sql 内部通过一个 map 来存储用户定义的相应驱动。

```
var drivers = make(map[string]driver.Driver)

drivers[name] = driver
```

因此通过 database/sql 的注册函数可以同时注册多个数据库驱动，只要不重复。

在我们使用 database/sql 接口和第三方库的时候经常看到如下:

```
  import (
      "database/sql"
      _ "github.com/mattn/go-sqlite3"
  )
```

新手都会被这个 _ 所迷惑，其实这个就是 Go 设计的巧妙之处，我们在变量赋值的时候经常看到这个符号，它是用来忽略变量赋值的占位符，那么包引入用到这个符号也是相似的作用，这儿使用 _ 的意思是引入后面的包名而不直接使用这个包中定义的函数，变量等资源。

我们在 2.3 节流程和函数一节中介绍过 init 函数的初始化过程，包在引入的时候会自动调用包的 init 函数以完成对包的初始化。因此，我们引入上面的数据库驱动包之后会自动去调用 init 函数，然后在 init 函数里面注册这个数据库驱动，这样我们就可以在接下来的代码中直接使用这个数据库驱动了。

**driver.Driver**

Driver 是一个数据库驱动的接口，他定义了一个 method： Open (name string)，这个方法返回一个数据库的 Conn 接口。

```
type Driver interface {
    Open(name string) (Conn, error)
}
```


返回的 Conn 只能用来进行一次 goroutine 的操作，也就是说不能把这个 Conn 应用于 Go 的多个 goroutine 里面。如下代码会出现错误

```
...
go goroutineA (Conn)  // 执行查询操作
go goroutineB (Conn)  // 执行插入操作
...
```


上面这样的代码可能会使 Go 不知道某个操作究竟是由哪个 goroutine 发起的，从而导致数据混乱，比如可能会把 goroutineA 里面执行的查询操作的结果返回给 goroutineB 从而使 B 错误地把此结果当成自己执行的插入数据。

第三方驱动都会定义这个函数，它会解析 name 参数来获取相关数据库的连接信息，解析完成后，它将使用此信息来初始化一个 Conn 并返回它。

**driver.Conn**

Conn 是一个数据库连接的接口定义，他定义了一系列方法，这个 Conn 只能应用在一个 goroutine 里面，不能使用在多个 goroutine 里面，详情请参考上面的说明。

```
type Conn interface {
    Prepare(query string) (Stmt, error)
    Close() error
    Begin() (Tx, error)
}
```

Prepare 函数返回与当前连接相关的执行 Sql 语句的准备状态，可以进行查询、删除等操作。

Close 函数关闭当前的连接，执行释放连接拥有的资源等清理工作。因为驱动实现了 database/sql 里面建议的 conn pool，所以你不用再去实现缓存 conn 之类的，这样会容易引起问题。

Begin 函数返回一个代表事务处理的 Tx，通过它你可以进行查询，更新等操作，或者对事务进行回滚、递交。

**driver.Stmt**

Stmt 是一种准备好的状态，和 Conn 相关联，而且只能应用于一个 goroutine 中，不能应用于多个 goroutine。

```
type Stmt interface {
    Close() error
    NumInput() int
    Exec(args []Value) (Result, error)
    Query(args []Value) (Rows, error)
}
```


Close 函数关闭当前的链接状态，但是如果当前正在执行 query，query 还是有效返回 rows 数据。

NumInput 函数返回当前预留参数的个数，当返回 >=0 时数据库驱动就会智能检查调用者的参数。当数据库驱动包不知道预留参数的时候，返回 -1。

Exec 函数执行 Prepare 准备好的 sql，传入参数执行 update/insert 等操作，返回 Result 数据

Query 函数执行 Prepare 准备好的 sql，传入需要的参数执行 select 操作，返回 Rows 结果集

**driver.Tx**

事务处理一般就两个过程，递交或者回滚。数据库驱动里面也只需要实现这两个函数就可以

```
type Tx interface {
    Commit() error
    Rollback() error
}
```


这两个函数一个用来递交一个事务，一个用来回滚事务。

**driver.Execer**

这是一个 Conn 可选择实现的接口

```
type Execer interface {
    Exec(query string, args []Value) (Result, error)
}
```


如果这个接口没有定义，那么在调用 DB.Exec, 就会首先调用 Prepare 返回 Stmt，然后执行 Stmt 的 Exec，然后关闭 Stmt。

**driver.Result**

这个是执行 Update/Insert 等操作返回的结果接口定义

```
type Result interface {
    LastInsertId() (int64, error)
    RowsAffected() (int64, error)
}
```


LastInsertId 函数返回由数据库执行插入操作得到的自增 ID 号。

RowsAffected 函数返回 query 操作影响的数据条目数。

**driver.Rows**

Rows 是执行查询返回的结果集接口定义

```
type Rows interface {
    Columns() []string
    Close() error
    Next(dest []Value) error
}
```


Columns 函数返回查询数据库表的字段信息，这个返回的 slice 和 sql 查询的字段一一对应，而不是返回整个表的所有字段。

Close 函数用来关闭 Rows 迭代器。

Next 函数用来返回下一条数据，把数据赋值给 dest。dest 里面的元素必须是 driver.Value 的值除了 string，返回的数据里面所有的 string 都必须要转换成 [] byte。如果最后没数据了，Next 函数最后返回 io.EOF。

**driver.RowsAffected**

RowsAffected 其实就是一个 int64 的别名，但是他实现了 Result 接口，用来底层实现 Result 的表示方式

```
type RowsAffected int64

func (RowsAffected) LastInsertId() (int64, error)

func (v RowsAffected) RowsAffected() (int64, error)
```

**driver.Value**

Value 其实就是一个空接口，他可以容纳任何的数据

```
type Value interface{}
drive 的 Value 是驱动必须能够操作的 Value，Value 要么是 nil，要么是下面的任意一种


int64
float64
bool
[]byte
string   [*]除了Rows.Next 返回的不能是 string.
time.Time
driver.ValueConverter
ValueConverter 接口定义了如何把一个普通的值转化成 driver.Value 的接口


type ValueConverter interface {
    ConvertValue(v interface{}) (Value, error)
}
```


在开发的数据库驱动包里面实现这个接口的函数在很多地方会使用到，这个 ValueConverter 有很多好处：

转化 driver.value 到数据库表相应的字段，例如 int64 的数据如何转化成数据库表 uint16 字段
把数据库查询结果转化成 driver.Value 值
在 scan 函数里面如何把 driver.Value 值转化成用户定义的值

**driver.Valuer**

Valuer 接口定义了返回一个 driver.Value 的方式

```
type Valuer interface {
    Value() (Value, error)
}
```

很多类型都实现了这个 Value 方法，用来自身与 driver.Value 的转化。

通过上面的讲解，你应该对于驱动的开发有了一个基本的了解，一个驱动只要实现了这些接口就能完成增删查改等基本操作了，剩下的就是与相应的数据库进行数据交互等细节问题了，在此不再赘述。

**database**/sql

database/sql 在 database/sql/driver 提供的接口基础上定义了一些更高阶的方法，用以简化数据库操作，同时内部还建议性地实现一个 conn pool。

```
type DB struct {
    driver   driver.Driver
    dsn      string
    mu       sync.Mutex // protects freeConn and closed
    freeConn []driver.Conn
    closed   bool
}
```

我们可以看到 Open 函数返回的是 DB 对象，里面有一个 freeConn，它就是那个简易的连接池。它的实现相当简单或者说简陋，就是当执行 db.prepare -> db.prepareDC 的时候会 defer dc.releaseConn，然后调用 db.putConn，也就是把这个连接放入连接池，每次调用 db.conn 的时候会先判断 freeConn 的长度是否大于 0，大于 0 说明有可以复用的 conn，直接拿出来用就是了，如果不大于 0，则创建一个 conn，然后再返回之。

### 使用 MySQL 数据库

目前 Internet 上流行的网站构架方式是 LAMP，其中的 M 即 MySQL, 作为数据库，MySQL 以免费、开源、使用方便为优势成为了很多 Web 开发的后端数据库存储引擎。

**MySQL 驱动**
Go 中支持 MySQL 的驱动目前比较多，有如下几种，有些是支持 database/sql 标准，而有些是采用了自己的实现接口，常用的有如下几种:

```
github.com/go-sql-driver/mysql 支持 database/sql，全部采用 go 写。
github.com/ziutek/mymysql 支持 database/sql，也支持自定义的接口，全部采用 go 写。
github.com/Philio/GoMySQL 不支持 database/sql，自定义接口，全部采用 go 写。
```

接下来的例子我主要以第一个驱动为例 (我目前项目中也是采用它来驱动)，也推荐大家采用它，主要理由：

- 这个驱动比较新，维护的比较好
- 完全支持 database/sql 接口
- 支持 keepalive，保持长连接，虽然 星星 fork 的 mymysql 也支持 keepalive，但不是线程安全的，这个从底层就支持了 keepalive。

示例代码
接下来的几个小节里面我们都将采用同一个数据库表结构：数据库 test，用户表 userinfo，关联用户信息表 userdetail。

```
CREATE TABLE `userinfo` (
    `uid` INT(10) NOT NULL AUTO_INCREMENT,
    `username` VARCHAR(64) NULL DEFAULT NULL,
    `department` VARCHAR(64) NULL DEFAULT NULL,
    `created` DATE NULL DEFAULT NULL,
    PRIMARY KEY (`uid`)
);

CREATE TABLE `userdetail` (
    `uid` INT(10) NOT NULL DEFAULT '0',
    `intro` TEXT NULL,
    `profile` TEXT NULL,
    PRIMARY KEY (`uid`)
)
```

如下示例将示范如何使用 database/sql 接口对数据库表进行增删改查操作

```
package main

import (
    "database/sql"
    "fmt"
    // "time"

    _ "github.com/go-sql-driver/mysql"

)

func main() {
    db, err := sql.Open("mysql", "astaxie:astaxie@/test?charset=utf8")
    checkErr(err)

    // 插入数据
    stmt, err := db.Prepare("INSERT userinfo SET username=?,department=?,created=?")
    checkErr(err)
    
    res, err := stmt.Exec("astaxie", "研发部门", "2012-12-09")
    checkErr(err)
    
    id, err := res.LastInsertId()
    checkErr(err)
    
    fmt.Println(id)
    // 更新数据
    stmt, err = db.Prepare("update userinfo set username=? where uid=?")
    checkErr(err)
    
    res, err = stmt.Exec("astaxieupdate", id)
    checkErr(err)
    
    affect, err := res.RowsAffected()
    checkErr(err)
    
    fmt.Println(affect)
    
    // 查询数据
    rows, err := db.Query("SELECT * FROM userinfo")
    checkErr(err)
    
    for rows.Next() {
        var uid int
        var username string
        var department string
        var created string
        err = rows.Scan(&uid, &username, &department, &created)
        checkErr(err)
        fmt.Println(uid)
        fmt.Println(username)
        fmt.Println(department)
        fmt.Println(created)
    }
    
    // 删除数据
    stmt, err = db.Prepare("delete from userinfo where uid=?")
    checkErr(err)
    
    res, err = stmt.Exec(id)
    checkErr(err)
    
    affect, err = res.RowsAffected()
    checkErr(err)
    
    fmt.Println(affect)
    
    db.Close()

}

func checkErr(err error) {
    if err != nil {
        panic(err)
    }
}
```

通过上面的代码我们可以看出，Go 操作 Mysql 数据库是很方便的。

关键的几个函数：

sql.Open () 函数用来打开一个注册过的数据库驱动，go-sql-driver 中注册了 mysql 这个数据库驱动，第二个参数是 DSN (Data Source Name)，它是 go-sql-driver 定义的一些数据库链接和配置信息。它支持如下格式：

```
user@unix(/path/to/socket)/dbname?charset=utf8
user:password@tcp(localhost:5555)/dbname?charset=utf8
user:password@/dbname
user:password@tcp([de:ad:be:ef::ca:fe]:80)/dbname
```

db.Prepare () 函数用来返回准备要执行的 sql 操作，然后返回准备完毕的执行状态。

db.Query () 函数用来直接执行 Sql 返回 Rows 结果。

stmt.Exec () 函数用来执行 stmt 准备好的 SQL 语句

我们可以看到我们传入的参数都是 =? 对应的数据，这样做的方式可以一定程度上防止 SQL 注入。

### 使用 SQLite 数据库

SQLite 是一个开源的嵌入式关系数据库，实现自包容、零配置、支持事务的 SQL 数据库引擎。其特点是高度便携、使用方便、结构紧凑、高效、可靠。 与其他数据库管理系统不同，SQLite 的安装和运行非常简单，在大多数情况下，只要确保 SQLite 的二进制文件存在即可开始创建、连接和使用数据库。如果您正在寻找一个嵌入式数据库项目或解决方案，SQLite 是绝对值得考虑。SQLite 可以是说开源的 Access。

**驱动**
Go 支持 sqlite 的驱动也比较多，但是好多都是不支持 database/sql 接口的

```
github.com/mattn/go-sqlite3 支持 database/sql 接口，基于 cgo (关于 cgo 的知识请参看官方文档或者本书后面的章节) 写的
github.com/feyeleanor/gosqlite3 不支持 database/sql 接口，基于 cgo 写的
github.com/phf/go-sqlite3 不支持 database/sql 接口，基于 cgo 写的
```

目前支持 database/sql 的 SQLite 数据库驱动只有第一个，我目前也是采用它来开发项目的。采用标准接口有利于以后出现更好的驱动的时候做迁移。

实例代码
示例的数据库表结构如下所示，相应的建表 SQL：

```
CREATE TABLE `userinfo` (
    `uid` INTEGER PRIMARY KEY AUTOINCREMENT,
    `username` VARCHAR(64) NULL,
    `department` VARCHAR(64) NULL,
    `created` DATE NULL
);

CREATE TABLE `userdetail` (
    `uid` INT(10) NULL,
    `intro` TEXT NULL,
    `profile` TEXT NULL,
    PRIMARY KEY (`uid`)
);
```

看下面 Go 程序是如何操作数据库表数据：增删改查

```
package main

import (
    "database/sql"
    "fmt"
    "time"

    _ "github.com/mattn/go-sqlite3"

)

func main() {
    db, err := sql.Open("sqlite3", "./foo.db")
    checkErr(err)

    // 插入数据
    stmt, err := db.Prepare("INSERT INTO userinfo(username, department, created) values(?,?,?)")
    checkErr(err)
    
    res, err := stmt.Exec("astaxie", "研发部门", "2012-12-09")
    checkErr(err)
    
    id, err := res.LastInsertId()
    checkErr(err)
    
    fmt.Println(id)
    // 更新数据
    stmt, err = db.Prepare("update userinfo set username=? where uid=?")
    checkErr(err)
    
    res, err = stmt.Exec("astaxieupdate", id)
    checkErr(err)
    
    affect, err := res.RowsAffected()
    checkErr(err)
    
    fmt.Println(affect)
    
    // 查询数据
    rows, err := db.Query("SELECT * FROM userinfo")
    checkErr(err)
    
    for rows.Next() {
        var uid int
        var username string
        var department string
        var created time.Time
        err = rows.Scan(&uid, &username, &department, &created)
        checkErr(err)
        fmt.Println(uid)
        fmt.Println(username)
        fmt.Println(department)
        fmt.Println(created)
    }
    
    // 删除数据
    stmt, err = db.Prepare("delete from userinfo where uid=?")
    checkErr(err)
    
    res, err = stmt.Exec(id)
    checkErr(err)
    
    affect, err = res.RowsAffected()
    checkErr(err)
    
    fmt.Println(affect)
    
    db.Close()

}

func checkErr(err error) {
    if err != nil {
        panic(err)
    }
}
```

我们可以看到上面的代码和 MySQL 例子里面的代码几乎是一模一样的，唯一改变的就是导入的驱动改变了，然后调用 sql.Open 是采用了 SQLite 的方式打开。

sqlite 管理工具：sqliteadmin.orbmu2k.de/

可以方便的新建数据库管理。

### 使用 PostgreSQL 数据库

PostgreSQL 是一个自由的对象 - 关系数据库服务器 (数据库管理系统)，它在灵活的 BSD - 风格许可证下发行。它提供了相对其他开放源代码数据库系统 (比如 MySQL 和 Firebird)，和对专有系统比如 Oracle、Sybase、IBM 的 DB2 和 Microsoft SQL Server 的一种选择。

PostgreSQL 和 MySQL 比较，它更加庞大一点，因为它是用来替代 Oracle 而设计的。所以在企业应用中采用 PostgreSQL 是一个明智的选择。

MySQL 被 Oracle 收购之后正在逐步的封闭（自 MySQL 5.5.31 以后的所有版本将不再遵循 GPL 协议），鉴于此，将来我们也许会选择 PostgreSQL 而不是 MySQL 作为项目的后端数据库。

**驱动**

Go 实现的支持 PostgreSQL 的驱动也很多，因为国外很多人在开发中使用了这个数据库。

```
github.com/lib/pq 支持 database/sql 驱动，纯 Go 写的
github.com/jbarham/gopgsqldriver 支持 database/sql 驱动，纯 Go 写的
github.com/lxn/go-pgsql 支持 database/sql 驱动，纯 Go 写的
```

在下面的示例中我采用了第一个驱动，因为它目前使用的人最多，在 github 上也比较活跃。

实例代码

数据库建表语句：

```
CREATE TABLE userinfo
(
    uid serial NOT NULL,
    username character varying(100) NOT NULL,
    department character varying(500) NOT NULL,
    Created date,
    CONSTRAINT userinfo_pkey PRIMARY KEY (uid)
)
WITH (OIDS=FALSE);

CREATE TABLE userdetail
(
    uid integer,
    intro character varying(100),
    profile character varying(100)
)
WITH(OIDS=FALSE);
```

看下面这个 Go 如何操作数据库表数据：增删改查

```
package main

import (
    "database/sql"
    "fmt"

    _ "github.com/lib/pq"

)

func main() {
    db, err := sql.Open("postgres", "user=astaxie password=astaxie dbname=test sslmode=disable")
    checkErr(err)

    // 插入数据
    stmt, err := db.Prepare("INSERT INTO userinfo(username,department,created) VALUES($1,$2,$3) RETURNING uid")
    checkErr(err)
    
    res, err := stmt.Exec("astaxie", "研发部门", "2012-12-09")
    checkErr(err)
    
    // pg 不支持这个函数，因为他没有类似 MySQL 的自增 ID
    // id, err := res.LastInsertId()
    // checkErr(err)
    // fmt.Println(id)
    
    var lastInsertId int
    err = db.QueryRow("INSERT INTO userinfo(username,departname,created) VALUES($1,$2,$3) returning uid;", "astaxie", "研发部门", "2012-12-09").Scan(&lastInsertId)
    checkErr(err)
    fmt.Println("最后插入id =", lastInsertId)
    
    // 更新数据
    stmt, err = db.Prepare("update userinfo set username=$1 where uid=$2")
    checkErr(err)
    
    res, err = stmt.Exec("astaxieupdate", 1)
    checkErr(err)
    
    affect, err := res.RowsAffected()
    checkErr(err)
    
    fmt.Println(affect)
    
    // 查询数据
    rows, err := db.Query("SELECT * FROM userinfo")
    checkErr(err)
    
    for rows.Next() {
        var uid int
        var username string
        var department string
        var created string
        err = rows.Scan(&uid, &username, &department, &created)
        checkErr(err)
        fmt.Println(uid)
        fmt.Println(username)
        fmt.Println(department)
        fmt.Println(created)
    }
    
    // 删除数据
    stmt, err = db.Prepare("delete from userinfo where uid=$1")
    checkErr(err)
    
    res, err = stmt.Exec(1)
    checkErr(err)
    
    affect, err = res.RowsAffected()
    checkErr(err)
    
    fmt.Println(affect)
    
    db.Close()

}

func checkErr(err error) {
    if err != nil {
        panic(err)
    }
}
```

从上面的代码我们可以看到，**PostgreSQL 是通过 `$1`, `$2` 这种方式来指定要传递的参数，而不是 MySQL 中的 ?**，另外在 sql.Open 中的 dsn 信息的格式也与 MySQL 的驱动中的 dsn 格式不一样，所以在使用时请注意它们的差异。

还有 pg 不支持 LastInsertId 函数，因为 PostgreSQL 内部没有实现类似 MySQL 的自增 ID 返回，其他的代码几乎是一模一样

### 使用 Beego orm 库进行 ORM 开发

beego orm 是一个 Go 进行 ORM 操作的库，它采用了 Go style 方式对数据库进行操作，实现了 struct 到数据表记录的映射。beego orm 是一个十分轻量级的 Go ORM 框架，开发这个库的本意降低复杂的 ORM 学习曲线，尽可能在 ORM 的运行效率和功能之间寻求一个平衡，beego orm 是目前开源的 Go ORM 框架中实现比较完整的一个库，而且运行效率相当不错，功能也基本能满足需求。

beego orm 是支持 database/sql 标准接口的 ORM 库，所以理论上来说，只要数据库驱动支持 database/sql 接口就可以无缝的接入 beego orm。目前我测试过的驱动包括下面几个：

```
Mysql: github/go-mysql-driver/mysql

PostgreSQL: github.com/lib/pq

SQLite: github.com/mattn/go-sqlite3

Mysql: github.com/ziutek/mymysql/godrv
```

暂未支持数据库:

```
MsSql: github.com/denisenkom/go-mssqldb

MS ADODB: github.com/mattn/go-adodb

Oracle: github.com/mattn/go-oci8

ODBC: bitbucket.org/miquella/mgodbc
```

安装

beego orm 支持 go get 方式安装，是完全按照 Go Style 的方式来实现的。

```
go get github.com/astaxie/beego
```

如何初始化

首先你需要 import 相应的数据库驱动包、database/sql 标准接口包以及 beego orm 包，如下所示：

```
import (
    "database/sql"
    "github.com/astaxie/beego/orm"
    _ "github.com/go-sql-driver/mysql"
)

func init() {
    // 注册驱动
    orm.RegisterDriver("mysql", orm.DR_MySQL)
    // 设置默认数据库
    orm.RegisterDataBase("default", "mysql", "root:root@/my_db?charset=utf8", 30)
    // 注册定义的 model
    orm.RegisterModel(new(User))

    // 创建 table
    orm.RunSyncdb("default", false, true)

}
PostgreSQL 配置:

// 导入驱动
// _ "github.com/lib/pq"

// 注册驱动
orm.RegisterDriver("postgres", orm.DR_Postgres) 

// 设置默认数据库
// PostgresQL用户：postgres ，密码：zxxx ， 数据库名称：test ， 数据库别名：default
orm.RegisterDataBase("default", "postgres", "user=postgres password=zxxx dbname=test host=127.0.0.1 port=5432 sslmode=disable")
MySQL 配置:

// 导入驱动
// _ "github.com/go-sql-driver/mysql"

// 注册驱动
orm.RegisterDriver("mysql", orm.DR_MySQL)

// 设置默认数据库
//mysql用户：root ，密码：zxxx ， 数据库名称：test ， 数据库别名：default
 orm.RegisterDataBase("default", "mysql", "root:zxxx@/test?charset=utf8")
Sqlite 配置:

// 导入驱动
// _ "github.com/mattn/go-sqlite3"

// 注册驱动
orm.RegisterDriver("sqlite", orm.DR_Sqlite)

// 设置默认数据库
// 数据库存放位置：./datas/test.db ， 数据库别名：default
orm.RegisterDataBase("default", "sqlite3", "./datas/test.db")
导入必须的 package 之后，我们需要打开到数据库的链接，然后创建一个 beego orm 对象（以 MySQL 为例)，如下所示
beego orm:


func main() {
    o := orm.NewOrm()
}
简单示例:


package main

import (
    "fmt"
    "github.com/astaxie/beego/orm"
    _ "github.com/go-sql-driver/mysql" // 导入数据库驱动
)

// Model Struct
type User struct {
    Id   int
    Name string `orm:"size(100)"`
}

func init() {
    // 设置默认数据库
    orm.RegisterDataBase("default", "mysql", "root:root@/my_db?charset=utf8", 30)

    // 注册定义的 model
    orm.RegisterModel(new(User))

// RegisterModel 也可以同时注册多个 model
// orm.RegisterModel(new(User), new(Profile), new(Post))

    // 创建 table
    orm.RunSyncdb("default", false, true)

}

func main() {
    o := orm.NewOrm()

    user := User{Name: "slene"}
    
    // 插入表
    id, err := o.Insert(&user)
    fmt.Printf("ID: %d, ERR: %v\n", id, err)
    
    // 更新表
    user.Name = "astaxie"
    num, err := o.Update(&user)
    fmt.Printf("NUM: %d, ERR: %v\n", num, err)
    
    // 读取 one
    u := User{Id: user.Id}
    err = o.Read(&u)
    fmt.Printf("ERR: %v\n", err)
    
    // 删除表
    num, err = o.Delete(&u)
    fmt.Printf("NUM: %d, ERR: %v\n", num, err)

}
SetMaxIdleConns

根据数据库的别名，设置数据库的最大空闲连接


orm.SetMaxIdleConns("default", 30)
SetMaxOpenConns

根据数据库的别名，设置数据库的最大数据库连接 (go>= 1.2)


orm.SetMaxOpenConns("default", 30)
目前 beego orm 支持打印调试，你可以通过如下的代码实现调试


 orm.Debug = true
接下来我们的例子采用前面的数据库表 User，现在我们建立相应的 struct


type Userinfo struct {
    Uid         int `orm:"PK"` //如果表的主键不是id，那么需要加上pk注释，显式的说这个字段是主键
    Username    string
    Departname  string
    Created     time.Time
}

type User struct {
    Uid         int `orm:"PK"` //如果表的主键不是id，那么需要加上pk注释，显式的说这个字段是主键
    Name        string
    Profile     *Profile   `orm:"rel(one)"` // OneToOne relation
    Post        []*Post `orm:"reverse(many)"` // 设置一对多的反向关系
}

type Profile struct {
    Id          int
    Age         int16
    User        *User   `orm:"reverse(one)"` // 设置一对一反向关系(可选)
}

type Post struct {
    Id    int
    Title string
    User  *User  `orm:"rel(fk)"`    // 设置一对多关系
    Tags  []*Tag `orm:"rel(m2m)"`
}

type Tag struct {
    Id    int
    Name  string
    Posts []*Post `orm:"reverse(many)"`
}

func init() {
    // 需要在 init 中注册定义的 model
    orm.RegisterModel(new(Userinfo),new(User), new(Profile), new(Tag))
}
注意一点，beego orm 针对驼峰命名会自动帮你转化成下划线字段，例如你定义了 Struct 名字为 UserInfo，那么转化成底层实现的时候是 user_info，字段命名也遵循该规则。

插入数据
下面的代码演示了如何插入一条记录，可以看到我们操作的是 struct 对象，而不是原生的 sql 语句，最后通过调用 Insert 接口将数据保存到数据库。


o := orm.NewOrm()
var user User
user.Name = "zxxx"
user.Departname = "zxxx"

id, err := o.Insert(&user)
if err == nil {
    fmt.Println(id)
}
我们看到插入之后 user.Uid 就是插入成功之后的自增 ID。

同时插入多个对象: InsertMulti

类似 sql 语句


insert into table (name, age) values("slene", 28),("astaxie", 30),("unknown", 20)
第一个参数 bulk 为并列插入的数量，第二个为对象的 slice

返回值为成功插入的数量


users := []User{
    {Name: "slene"},
    {Name: "astaxie"},
    {Name: "unknown"},
    ...
}
successNums, err := o.InsertMulti(100, users)
bulk 为 1 时，将会顺序插入 slice 中的数据

更新数据
继续上面的例子来演示更新操作，现在 user 的主键已经有值了，此时调用 Insert 接口，beego orm 内部会自动调用 update 以进行数据的更新而非插入操作。

o := orm.NewOrm()
user := User{Uid: 1}
if o.Read(&user) == nil {
    user.Name = "MyName"
    if num, err := o.Update(&user); err == nil {
        fmt.Println(num)
    }
}
Update 默认更新所有的字段，可以更新指定的字段：


// 只更新 Name
o.Update(&user, "Name")
// 指定多个字段
// o.Update(&user, "Field1", "Field2", ...)
// Where: 用来设置条件，支持多个参数，第一个参数如果为整数，相当于调用了 Where ("主键 =?", 值)。

查询数据
beego orm 的查询接口比较灵活，具体使用请看下面的例子

例子 1，根据主键获取数据：


o := orm.NewOrm()
var user User

user := User{Id: 1}

err = o.Read(&user)

if err == orm.ErrNoRows {
    fmt.Println("查询不到")
} else if err == orm.ErrMissPK {
    fmt.Println("找不到主键")
} else {
    fmt.Println(user.Id, user.Name)
}
例子 2：


o := orm.NewOrm()
var user User

qs := o.QueryTable(user) // 返回 QuerySeter
qs.Filter("id", 1) // WHERE id = 1
qs.Filter("profile__age", 18) // WHERE profile.age = 18
例子 3，WHERE IN 查询条件：


qs.Filter("profile__age__in", 18, 20) 
// WHERE profile.age IN (18, 20)
例子 4，更加复杂的条件：


qs.Filter("profile__age__in", 18, 20).Exclude("profile__lt", 1000)
// WHERE profile.age IN (18, 20) AND NOT profile_id < 1000
可以通过如下接口获取多条数据，请看示例

例子 1，根据条件 age > 17，获取 20 位置开始的 10 条数据的数据


var allusers []User
qs.Filter("profile__age__gt", 17)
// WHERE profile.age > 17
例子 2，limit 默认从 10 开始，获取 10 条数据


qs.Limit(10, 20)
// LIMIT 10 OFFSET 20 注意跟 SQL 反过来的
删除数据
beedb 提供了丰富的删除数据接口，请看下面的例子

例子 1，删除单条数据


o := orm.NewOrm()
if num, err := o.Delete(&User{Id: 1}); err == nil {
    fmt.Println(num)
}
Delete 操作会对反向关系进行操作，此例中 Post 拥有一个到 User 的外键。删除 User 的时候。如果 on_delete 设置为默认的级联操作，将删除对应的 Post

关联查询
有些应用却需要用到连接查询，所以现在 beego orm 提供了一个简陋的实现方案：


type Post struct {
    Id    int    `orm:"auto"`
    Title string `orm:"size(100)"`
    User  *User  `orm:"rel(fk)"`
}

var posts []*Post
qs := o.QueryTable("post")
num, err := qs.Filter("User__Name", "slene").All(&posts)
上面代码中我们看到了一个 struct 关联查询

Group By 和 Having
针对有些应用需要用到 group by 的功能，beego orm 也提供了一个简陋的实现


qs.OrderBy("id", "-profile__age")
// ORDER BY id ASC, profile.age DESC

qs.OrderBy("-profile__age", "profile")
// ORDER BY profile.age DESC, profile_id ASC
上面的代码中出现了两个新接口函数

GroupBy: 用来指定进行 groupby 的字段

Having: 用来指定 having 执行的时候的条件

使用原生 sql
简单示例:
o := orm.NewOrm()
var r orm.RawSeter
r = o.Raw("UPDATE user SET name = ? WHERE name = ?", "testing", "slene")

复杂原生 sql 使用:
func (m *User) Query(name string) user []User {
    var o orm.Ormer
    var rs orm.RawSeter
    o = orm.NewOrm()
    rs = o.Raw("SELECT * FROM user "+
        "WHERE name=? AND uid>10 "+
        "ORDER BY uid DESC "+
        "LIMIT 100", name)
    // var user []User
    num, err := rs.QueryRows(&user)
    if err != nil {
        fmt.Println(err)
    } else {
        fmt.Println(num)
        // return user
    }
    return
}   
```

### NOSQL 数据库操作

NoSQL (Not Only SQL)，指的是非关系型的数据库。随着 Web 2.0 的兴起，传统的关系数据库在应付 Web 2.0 网站，特别是超大规模和高并发的 SNS 类型的 Web 2.0 纯动态网站已经显得力不从心，暴露了很多难以克服的问题，而非关系型的数据库则由于其本身的特点得到了非常迅速的发展。

而 Go 语言作为 21 世纪的 C 语言，对 NOSQL 的支持也是很好，目前流行的 NOSQL 主要有 **redis、mongoDB、Cassandra 和 Membase 等**。这些数据库都有高性能、高并发读写等特点，目前已经广泛应用于各种应用中。我接下来主要讲解一下 redis 和 mongoDB 的操作。

**redis**

redis 是一个 key-value 存储系统。和 Memcached 类似，它支持存储的 value 类型相对更多，包括 string (字符串)、list (链表)、set (集合 ) 和 zset (有序集合)。

目前应用 redis 最广泛的应该是新浪微博平台，其次还有 Facebook 收购的图片社交网站 instagram。以及其他一些有名的 互联网企业

Go 目前支持 redis 的驱动有如下

```
github.com/gomodule/redigo (推荐)
github.com/go-redis/redis
github.com/hoisie/redis
github.com/alphazero/Go-Redis
github.com/simonz05/godis
```

我以 redigo 驱动为例来演示如何进行数据的操作:

```
package main

import (
    "fmt"
    "os"
    "os/signal"
    "syscall"
    "time"

    "github.com/gomodule/redigo/redis"

)

var (
    Pool *redis.Pool
)

func init() {
    redisHost := ":6379"
    Pool = newPool(redisHost)
    close()
}

func newPool(server string) *redis.Pool {

    return &redis.Pool{
    
        MaxIdle:     3,
        IdleTimeout: 240 * time.Second,
    
        Dial: func() (redis.Conn, error) {
            c, err := redis.Dial("tcp", server)
            if err != nil {
                return nil, err
            }
            return c, err
        },
    
        TestOnBorrow: func(c redis.Conn, t time.Time) error {
            _, err := c.Do("PING")
            return err
        },
    }

}

func close() {
    c := make(chan os.Signal, 1)
    signal.Notify(c, os.Interrupt)
    signal.Notify(c, syscall.SIGTERM)
    signal.Notify(c, syscall.SIGKILL)
    go func() {
        <-c
        Pool.Close()
        os.Exit(0)
    }()
}

func Get(key string) ([]byte, error) {

    conn := Pool.Get()
    defer conn.Close()
    
    var data []byte
    data, err := redis.Bytes(conn.Do("GET", key))
    if err != nil {
        return data, fmt.Errorf("error get key %s: %v", key, err)
    }
    return data, err

}

func main() {
    test, err := Get("test")
    fmt.Println(test, err)
}
```

另外以前我 fork 了最后一个驱动，更新了一些 bug，目前应用在我自己的短域名服务项目中 (每天 200W 左右的 PV 值)

```
github.com/astaxie/goredis
```

接下来的以我自己 fork 的这个 redis 驱动为例来演示如何进行数据的操作

```
package main

import (
    "fmt"

    "github.com/astaxie/goredis"

)

func main() {
    var client goredis.Client
    // 设置端口为 redis 默认端口
    client.Addr = "127.0.0.1:6379"

    // 字符串操作
    client.Set("a", []byte("hello"))
    val, _ := client.Get("a")
    fmt.Println(string(val))
    client.Del("a")
    
    // list 操作
    vals := []string{"a", "b", "c", "d", "e"}
    for _, v := range vals {
        client.Rpush("l", []byte(v))
    }
    dbvals,_ := client.Lrange("l", 0, 4)
    for i, v := range dbvals {
        println(i,":",string(v))
    }
    client.Del("l")

}
```

我们可以看到操作 redis 非常的方便，而且我实际项目中应用下来性能也很高。client 的命令和 redis 的命令基本保持一致。所以和原生态操作 redis 非常类似。

**mongoDB**

MongoDB 是一个高性能，开源，无模式的文档型数据库，是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。他支持的数据结构非常松散，采用的是类似 json 的 bjson 格式来存储数据，因此可以存储比较复杂的数据类型。Mongo 最大的特点是他支持的查询语言非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对数据建立索引。

下图展示了 mysql 和 mongoDB 之间的对应关系，我们可以看出来非常的方便，但是 mongoDB 的性能非常好。

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/5.6.mongodb.png)

图 5.1 MongoDB 和 Mysql 的操作对比图

目前 Go 支持 mongoDB 最好的驱动就是 mgo，这个驱动目前最有可能成为官方的 pkg。

安装 mgo:

```
go get gopkg.in/mgo.v2
```

下面我将演示如何通过 Go 来操作 mongoDB：

```
package main

import (
    "fmt"
    "log"

    "gopkg.in/mgo.v2"
    "gopkg.in/mgo.v2/bson"

)

type Person struct {
    Name  string
    Phone string
}

func main() {
    session, err := mgo.Dial("server1.example.com,server2.example.com")
    if err != nil {
        panic(err)
    }
    defer session.Close()

    // Optional. Switch the session to a monotonic behavior.
    session.SetMode(mgo.Monotonic, true)
    
    c := session.DB("test").C("people")
    err = c.Insert(&Person{"Ale", "+55 53 8116 9639"},
        &Person{"Cla", "+55 53 8402 8510"})
    if err != nil {
        log.Fatal(err)
    }
    
    result := Person{}
    err = c.Find(bson.M{"name": "Ale"}).One(&result)
    if err != nil {
        log.Fatal(err)
    }
    
    fmt.Println("Phone:", result.Phone)

}
```

我们可以看出来 mgo 的操作方式和 beedb 的操作方式几乎类似，都是基于 struct 的操作方式，这个就是 Go Style。

### session 和 cookie

session 和 cookie 是网站浏览中较为常见的两个概念，也是比较难以辨析的两个概念，但它们在浏览需要认证的服务页面以及页面统计中却相当关键。我们先来了解一下 session 和 cookie 怎么来的？考虑这样一个问题：

如何抓取一个访问受限的网页？如新浪微博好友的主页，个人微博页面等。

显然，通过浏览器，我们可以手动输入用户名和密码来访问页面，而所谓的 “抓取”，其实就是使用程序来模拟完成同样的工作，因此我们需要了解 “登陆” 过程中到底发生了什么。

当用户来到微博登陆页面，输入用户名和密码之后点击 “登录” 后浏览器将认证信息 POST 给远端的服务器，服务器执行验证逻辑，如果验证通过，则浏览器会跳转到登录用户的微博首页，在登录成功后，服务器如何验证我们对其他受限制页面的访问呢？因为 HTTP 协议是无状态的，所以很显然服务器不可能知道我们已经在上一次的 HTTP 请求中通过了验证。当然，最简单的解决方案就是所有的请求里面都带上用户名和密码，这样虽然可行，但大大加重了服务器的负担（对于每个 request 都需要到数据库验证），也大大降低了用户体验 (每个页面都需要重新输入用户名密码，每个页面都带有登录表单)。既然直接在请求中带上用户名与密码不可行，那么就只有在服务器或客户端保存一些类似的可以代表身份的信息了，所以就有了 cookie 与 session。

**cookie，简而言之就是在本地计算机保存一些用户操作的历史信息（当然包括登录信息），并在用户再次访问该站点时浏览器通过 HTTP 协议将本地 cookie 内容发送给服务器，从而完成验证，或继续上一步操作。**

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/6.1.cookie2.png)

图 6.1 cookie 的原理图

**session，简而言之就是在服务器上保存用户操作的历史信息。服务器使用 session id 来标识 session，session id 由服务器负责产生，保证随机性与唯一性，相当于一个随机密钥，避免在握手或传输中暴露用户真实密码。但该方式下，仍然需要将发送请求的客户端与 session 进行对应，所以可以借助 cookie 机制来获取客户端的标识（即 session id），也可以通过 GET 方式将 id 提交给服务器。**

![](C:\Users\jiutian\Desktop\6.1.session.png)

图 6.2 session 的原理图

**cookie**

**Cookie 是由浏览器维持的，存储在客户端的一小段文本信息，伴随着用户请求和页面在 Web 服务器和浏览器之间传递**。用户每次访问站点时，Web 应用程序都可以读取 cookie 包含的信息。浏览器设置里面有 cookie 隐私数据选项，打开它，可以看到很多已访问网站的 cookies，如下图所示：

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/6.1.cookie.png)

图 6.3 浏览器端保存的 cookie 信息

cookie 是有时间限制的，根据生命期不同分成两种：**会话 cookie** 和**持久 cookie**；

如果**不设置过期时间**，则表示这个 cookie 的生命周期为从创建到浏览器关闭为止，只要关闭浏览器窗口，cookie 就消失了。这种生命期为**浏览会话期的 cookie 被称为会话 cookie**。会话 cookie 一般不保存在硬盘上而是保存在内存里。

如果**设置了过期时间 (setMaxAge (606024))**，**浏览器就会把 cookie 保存到硬盘上**，关闭后再次打开浏览器，这些 cookie 依然有效直到超过设定的过期时间。存储在硬盘上的 cookie 可以在不同的浏览器进程间共享，比如两个 IE 窗口。而对于保存在内存的 cookie，不同的浏览器有不同的处理方式。
　　

**Go 设置 cookie**

Go 语言中通过 net/http 包中的 SetCookie 来设置：

```
http.SetCookie(w ResponseWriter, cookie *Cookie)
```


w 表示需要写入的 response，cookie 是一个 struct，让我们来看一下 cookie 对象是怎么样的

```
type Cookie struct {
    Name       string
    Value      string
    Path       string
    Domain     string
    Expires    time.Time
    RawExpires string

// MaxAge=0 means no 'Max-Age' attribute specified.
// MaxAge<0 means delete cookie now, equivalently 'Max-Age: 0'
// MaxAge>0 means Max-Age attribute present and given in seconds
    MaxAge   int
    Secure   bool
    HttpOnly bool
    Raw      string
    Unparsed []string // Raw text of unparsed attribute-value pairs
}
```

我们来看一个例子，如何设置 cookie

```
expiration := time.Now()
expiration = expiration.AddDate(1, 0, 0)
cookie := http.Cookie{Name: "username", Value: "astaxie", Expires: expiration}
http.SetCookie(w, &cookie)　　
```

**Go 读取 cookie**

上面的例子演示了如何设置 cookie 数据，我们这里来演示一下如何读取 cookie

```
cookie, _ := r.Cookie("username")
fmt.Fprint(w, cookie)
```

还有另外一种读取方式

```
for _, cookie := range r.Cookies() {
    fmt.Fprint(w, cookie.Name)
}
```


可以看到通过 request 获取 cookie 非常方便。

**session**

session，中文经常翻译为会话，**其本来的含义是指有始有终的一系列动作 / 消息**，比如打电话是从拿起电话拨号到挂断电话这中间的一系列过程可以称之为一个 session。然而当 session 一词与网络协议相关联时，它又往往隐含了 “面向连接” 和 / 或 “保持状态” 这样两个含义。

session 在 Web 开发环境下的语义又有了新的扩展，它的含义是指一类用来在客户端与服务器端之间保持状态的解决方案。有时候 Session 也用来指这种解决方案的存储结构。

session 机制是一种服务器端的机制，服务器使用一种类似于散列表的结构 (也可能就是使用散列表) 来保存信息。

**但程序需要为某个客户端的请求创建一个 session 的时候，服务器首先检查这个客户端的请求里是否包含了一个 session 标识－称为 session id**，如果已经包含一个 session id 则说明以前已经为此客户创建过 session，服务器就按照 session id 把这个 session 检索出来使用 (如果检索不到，可能会新建一个，这种情况可能出现在服务端已经删除了该用户对应的 session 对象，但用户人为地在请求的 URL 后面附加上一个 JSESSION 的参数)。如果客户请求不包含 session id，则为此客户创建一个 session 并且同时生成一个与此 session 相关联的 session id，这个 session id 将在本次响应中返回给客户端保存。

session 机制本身并不复杂，然而其实现和配置上的灵活性却使得具体情况复杂多变。这也要求我们不能把仅仅某一次的经验或者某一个浏览器，服务器的经验当作普遍适用的。

**小结**

如上文所述，session 和 cookie 的目的相同，都是为了克服 http 协议无状态的缺陷，但完成的方法不同。session 通过 cookie，在客户端保存 session id，而将用户的其他会话消息保存在服务端的 session 对象中，与此相对的，cookie 需要将所有信息都保存在客户端。因此 cookie 存在着一定的安全隐患，例如本地 cookie 中保存的用户名密码被破译，或 cookie 被其他网站收集（例如：1. appA 主动设置域 B cookie，让域 B cookie 获取；2. XSS，在 appA 上通过 javascript 获取 document.cookie，并传递给自己的 appB）。

### Go 如何使用 session

**session 创建过程**

session 的基本原理是由服务器为每个会话维护一份信息数据，客户端和服务端依靠一个全局唯一的标识来访问这份数据，以达到交互的目的。当用户访问 Web 应用时，服务端程序会随需要创建 session，这个过程可以概括为三个步骤：

1. 生成全局唯一标识符（sessionid）；
2. 开辟数据存储空间。一般会在内存中创建相应的数据结构，但这种情况下，系统一旦掉电，所有的会话数据就会丢失，如果是电子商务类网站，这将造成严重的后果。所以为了解决这类问题，你可以将会话数据写到文件里或存储在数据库中，当然这样会增加 I/O 开销，但是它可以实现某种程度的 session 持久化，也更有利于 session 的共享；
3. 将 session 的全局唯一标示符发送给客户端。

以上三个步骤中，最关键的是如何发送这个 session 的唯一标识这一步上。考虑到 HTTP 协议的定义，数据无非可以放到请求行、头域或 Body 里，所以一般来说会有两种常用的方式：cookie 和 URL 重写。

1. Cookie
   服务端通过设置 Set-cookie 头就可以将 session 的标识符传送到客户端，而客户端此后的每一次请求都会带上这个标识符，另外一般包含 session 信息的 cookie 会将失效时间设置为 0 (会话 cookie)，即浏览器进程有效时间。至于浏览器怎么处理这个 0，每个浏览器都有自己的方案，但差别都不会太大 (一般体现在新建浏览器窗口的时候)；
2. URL 重写
   所谓 URL 重写，就是在返回给用户的页面里的所有的 URL 后面追加 session 标识符，这样用户在收到响应之后，无论点击响应页面里的哪个链接或提交表单，都会自动带上 session 标识符，从而就实现了会话的保持。虽然这种做法比较麻烦，但是，如果客户端禁用了 cookie 的话，此种方案将会是首选。

**Go 实现 session 管理**

我们知道 session 管理涉及到如下几个因素

- 全局 session 管理器
- 保证 sessionid 的全局唯一性
- 为每个客户关联一个 session
- session 的存储 (可以存储到内存、文件、数据库等)
- session 过期处理

接下来我将讲解一下我关于 session 管理的整个设计思路以及相应的 go 代码示例：

**Session 管理器**

定义一个全局的 session 管理器

```
type Manager struct {
    cookieName  string     // private cookiename
    lock        sync.Mutex // protects session
    provider    Provider
    maxLifeTime int64
}

func NewManager(provideName, cookieName string, maxLifeTime int64) (*Manager, error) {
    provider, ok := provides[provideName]
    if !ok {
        return nil, fmt.Errorf("session: unknown provide %q (forgotten import?)", provideName)
    }
    return &Manager{provider: provider, cookieName: cookieName, maxLifeTime: maxLifeTime}, nil
}
```

Go 实现整个的流程应该也是这样的，在 main 包中创建一个全局的 session 管理器

```
var globalSessions *session.Manager
// 然后在 init 函数中初始化
func init() {
    globalSessions, _ = NewManager("memory", "gosessionid", 3600)
}
```


我们知道 session 是保存在服务器端的数据，它可以以任何的方式存储，比如存储在内存、数据库或者文件中。因此我们抽象出一个 Provider 接口，用以表征 session 管理器底层存储结构。

```
type Provider interface {
    SessionInit(sid string) (Session, error)
    SessionRead(sid string) (Session, error)
    SessionDestroy(sid string) error
    SessionGC(maxLifeTime int64)
}
```

SessionInit 函数实现 Session 的初始化，操作成功则返回此新的 Session 变量

- SessionRead 函数返回 sid 所代表的 Session 变量，如果不存在，那么将以 sid 为参数调用 SessionInit 函数创建并返回一个新的 Session 变量
- SessionDestroy 函数用来销毁 sid 对应的 Session 变量
- SessionGC 根据 maxLifeTime 来删除过期的数据

那么 Session 接口需要实现什么样的功能呢？有过 Web 开发经验的读者知道，对 Session 的处理基本就设置值、读取值、删除值以及获取当前 sessionID 这四个操作，所以我们的 Session 接口也就实现这四个操作。

```
type Session interface {
    Set(key, value interface{}) error // set session value
    Get(key interface{}) interface{}  // get session value
    Delete(key interface{}) error     // delete session value
    SessionID() string                // back current sessionID
}
```


以上设计思路来源于 database/sql/driver，先定义好接口，然后具体的存储 session 的结构实现相应的接口并注册后，相应功能这样就可以使用了，以下是用来随需注册存储 session 的结构的 Register 函数的实现。

```
var provides = make(map[string]Provider)

// Register makes a session provide available by the provided name.
// If Register is called twice with the same name or if driver is nil,
// it panics.
func Register(name string, provider Provider) {
    if provider == nil {
        panic("session: Register provider is nil")
    }
    if _, dup := provides[name]; dup {
        panic("session: Register called twice for provider " + name)
    }
    provides[name] = provider
}
```

**全局唯一的 Session ID**

Session ID 是用来识别访问 Web 应用的每一个用户，因此必须保证它是全局唯一的（GUID），下面代码展示了如何满足这一需求：

```
func (manager *Manager) sessionId() string {
    b := make([]byte, 32)
    if _, err := rand.Read(b); err != nil {
        return ""
    }
    return base64.URLEncoding.EncodeToString(b)
}
```

**session 创建**

我们需要为每个来访用户分配或获取与他相关连的 Session，以便后面根据 Session 信息来验证操作。SessionStart 这个函数就是用来检测是否已经有某个 Session 与当前来访用户发生了关联，如果没有则创建之。

```
func (manager *Manager) SessionStart(w http.ResponseWriter, r *http.Request) (session Session) {
    manager.lock.Lock()
    defer manager.lock.Unlock()
    cookie, err := r.Cookie(manager.cookieName)
    if err != nil || cookie.Value == "" {
        sid := manager.sessionId()
        session, _ = manager.provider.SessionInit(sid)
        cookie := http.Cookie{Name: manager.cookieName, Value: url.QueryEscape(sid), Path: "/", HttpOnly: true, MaxAge: int(manager.maxLifeTime)}
        http.SetCookie(w, &cookie)
    } else {
        sid, _ := url.QueryUnescape(cookie.Value)
        session, _ = manager.provider.SessionRead(sid)
    }
    return
}
```


我们用前面 login 操作来演示 session 的运用：

```
func login(w http.ResponseWriter, r *http.Request) {
    sess := globalSessions.SessionStart(w, r)
    r.ParseForm()
    if r.Method == "GET" {
        t, _ := template.ParseFiles("login.gtpl")
        w.Header().Set("Content-Type", "text/html")
        t.Execute(w, sess.Get("username"))
    } else {
        sess.Set("username", r.Form["username"])
        http.Redirect(w, r, "/", 302)
    }
}
```

**操作值：设置、读取和删除**

SessionStart 函数返回的是一个满足 Session 接口的变量，那么我们该如何用他来对 session 数据进行操作呢？

上面的例子中的代码 session.Get("uid") 已经展示了基本的读取数据的操作，现在我们再来看一下详细的操作:

```
func count(w http.ResponseWriter, r *http.Request) {
    sess := globalSessions.SessionStart(w, r)
    createtime := sess.Get("createtime")
    if createtime == nil {
        sess.Set("createtime", time.Now().Unix())
    } else if (createtime.(int64) + 360) < (time.Now().Unix()) {
        globalSessions.SessionDestroy(w, r)
        sess = globalSessions.SessionStart(w, r)
    }
    ct := sess.Get("countnum")
    if ct == nil {
        sess.Set("countnum", 1)
    } else {
        sess.Set("countnum", (ct.(int) + 1))
    }
    t, _ := template.ParseFiles("count.gtpl")
    w.Header().Set("Content-Type", "text/html")
    t.Execute(w, sess.Get("countnum"))
}

```


通过上面的例子可以看到，Session 的操作和操作 key/value 数据库类似: Set、Get、Delete 等操作

因为 Session 有过期的概念，所以我们定义了 GC 操作，当访问过期时间满足 GC 的触发条件后将会引起 GC，但是当我们进行了任意一个 session 操作，都会对 Session 实体进行更新，都会触发对最后访问时间的修改，这样当 GC 的时候就不会误删除还在使用的 Session 实体。

**session 重置**

我们知道，Web 应用中有用户退出这个操作，那么当用户退出应用的时候，我们需要对该用户的 session 数据进行销毁操作，上面的代码已经演示了如何使用 session 重置操作，下面这个函数就是实现了这个功能：

```
// Destroy sessionid
func (manager *Manager) SessionDestroy(w http.ResponseWriter, r *http.Request){
    cookie, err := r.Cookie(manager.cookieName)
    if err != nil || cookie.Value == "" {
        return
    } else {
        manager.lock.Lock()
        defer manager.lock.Unlock()
        manager.provider.SessionDestroy(cookie.Value)
        expiration := time.Now()
        cookie := http.Cookie{Name: manager.cookieName, Path: "/", HttpOnly: true, Expires: expiration, MaxAge: -1}
        http.SetCookie(w, &cookie)
    }
}
```

**session 销毁**

我们来看一下 Session 管理器如何来管理销毁，只要我们在 Main 启动的时候启动：

```
func init() {
    go globalSessions.GC()
}

func (manager *Manager) GC() {
    manager.lock.Lock()
    defer manager.lock.Unlock()
    manager.provider.SessionGC(manager.maxLifeTime)
    time.AfterFunc(time.Duration(manager.maxLifeTime), func() { manager.GC() })
}
```

我们可以看到 GC 充分利用了 time 包中的定时器功能，当超时 maxLifeTime 之后调用 GC 函数，这样就可以保证 maxLifeTime 时间内的 session 都是可用的，类似的方案也可以用于统计在线用户数之类的。

### session 存储



```

package memory

import (
    "container/list"
    "github.com/astaxie/session"
    "sync"
    "time"
)

var pder = &Provider{list: list.New()}

type SessionStore struct {
    sid          string                      // session id唯一标示
    timeAccessed time.Time                   // 最后访问时间
    value        map[interface{}]interface{} // session里面存储的值
}

func (st *SessionStore) Set(key, value interface{}) error {
    st.value[key] = value
    pder.SessionUpdate(st.sid)
    return nil
}

func (st *SessionStore) Get(key interface{}) interface{} {
    pder.SessionUpdate(st.sid)
    if v, ok := st.value[key]; ok {
        return v
    } else {
        return nil
    }
}

func (st *SessionStore) Delete(key interface{}) error {
    delete(st.value, key)
    pder.SessionUpdate(st.sid)
    return nil
}

func (st *SessionStore) SessionID() string {
    return st.sid
}

type Provider struct {
    lock     sync.Mutex               // 用来锁
    sessions map[string]*list.Element // 用来存储在内存
    list     *list.List               // 用来做 gc
}

func (pder *Provider) SessionInit(sid string) (session.Session, error) {
    pder.lock.Lock()
    defer pder.lock.Unlock()
    v := make(map[interface{}]interface{}, 0)
    newsess := &SessionStore{sid: sid, timeAccessed: time.Now(), value: v}
    element := pder.list.PushFront(newsess)
    pder.sessions[sid] = element
    return newsess, nil
}

func (pder *Provider) SessionRead(sid string) (session.Session, error) {
    if element, ok := pder.sessions[sid]; ok {
        return element.Value.(*SessionStore), nil
    } else {
        sess, err := pder.SessionInit(sid)
        return sess, err
    }
    return nil, nil
}

func (pder *Provider) SessionDestroy(sid string) error {
    if element, ok := pder.sessions[sid]; ok {
        delete(pder.sessions, sid)
        pder.list.Remove(element)
        return nil
    }
    return nil
}

func (pder *Provider) SessionGC(maxlifetime int64) {
    pder.lock.Lock()
    defer pder.lock.Unlock()

    for {
        element := pder.list.Back()
        if element == nil {
            break
        }
        if (element.Value.(*SessionStore).timeAccessed.Unix() + maxlifetime) < time.Now().Unix() {
            pder.list.Remove(element)
            delete(pder.sessions, element.Value.(*SessionStore).sid)
        } else {
            break
        }
    }
}

func (pder *Provider) SessionUpdate(sid string) error {
    pder.lock.Lock()
    defer pder.lock.Unlock()
    if element, ok := pder.sessions[sid]; ok {
        element.Value.(*SessionStore).timeAccessed = time.Now()
        pder.list.MoveToFront(element)
        return nil
    }
    return nil
}

func init() {
    pder.sessions = make(map[string]*list.Element, 0)
    session.Register("memory", pder)
}
```

上面这个代码实现了一个内存存储的 session 机制。通过 init 函数注册到 session 管理器中。这样就可以方便的调用了。我们如何来调用该引擎呢？请看下面的代码

```
import (
    "github.com/astaxie/session"
    _ "github.com/astaxie/session/providers/memory"
)
```

当 import 的时候已经执行了 memory 函数里面的 init 函数，这样就已经注册到 session 管理器中，我们就可以使用了，通过如下方式就可以初始化一个 session 管理器：

```

var globalSessions *session.Manager

// 然后在 init 函数中初始化
func init() {
    globalSessions, _ = session.NewManager("memory", "gosessionid", 3600)
    go globalSessions.GC()
}
```

### 预防 session 劫持

session 劫持是一种广泛存在的比较严重的安全威胁，在 session 技术中，客户端和服务端通过 session 的标识符来维护会话， 但这个标识符很容易就能被嗅探到，从而被其他人利用。它是中间人攻击的一种类型。

本节将通过一个实例来演示会话劫持，希望通过这个实例，能让读者更好地理解 session 的本质。

**session 劫持过程**

我们写了如下的代码来展示一个 count 计数器：

```

func count(w http.ResponseWriter, r *http.Request) {
    sess := globalSessions.SessionStart(w, r)
    ct := sess.Get("countnum")
    if ct == nil {
        sess.Set("countnum", 1)
    } else {
        sess.Set("countnum", (ct.(int) + 1))
    }
    t, _ := template.ParseFiles("count.gtpl")
    w.Header().Set("Content-Type", "text/html")
    t.Execute(w, sess.Get("countnum"))
}
```

count.gtpl 的代码如下所示：

```
Hi. Now count:{{.}}
```

然后我们在浏览器里面刷新可以看到如下内容：

## 四、文本处理

Web 开发中对于文本处理是非常重要的一部分，我们往往需要对输出或者输入的内容进行处理，这里的文本包括字符串、数字、Json、XML 等等。Go 语言作为一门高性能的语言，对这些文本的处理都有官方的标准库来支持。而且在你使用中你会发现 Go 标准库的一些设计相当的巧妙，而且对于使用者来说也很方便就能处理这些文本。

### XML 处理

XML 作为一种数据交换和信息传递的格式已经十分普及。而随着 Web 服务日益广泛的应用，现在 XML 在日常的开发工作中也扮演了愈发重要的角色。

```
<?xml version="1.0" encoding="utf-8"?>
<servers version="1">
    <server>
        <serverName>Shanghai_VPN</serverName>
        <serverIP>127.0.0.1</serverIP>
    </server>
    <server>
        <serverName>Beijing_VPN</serverName>
        <serverIP>127.0.0.2</serverIP>
    </server>
</servers>
```

**解析 XML**

如何解析如上这个 XML 文件呢？ 我们可以通过 xml 包的 `Unmarshal` 函数来达到我们的目的

```
func Unmarshal(data []byte, v interface{}) error
```

data 接收的是 XML 数据流，v 是需要输出的结构，定义为 interface，也就是可以把 XML 转换为任意的格式。我们这里主要介绍 struct 的转换，**因为 struct 和 XML 都有类似树结构的特征**。

```

package main

import (
    "encoding/xml"
    "fmt"
    "io/ioutil"
    "os"
)

type Recurlyservers struct {
    XMLName     xml.Name `xml:"servers"`
    Version     string   `xml:"version,attr"`
    Svs         []server `xml:"server"`
    Description string   `xml:",innerxml"`
}

type server struct {
    XMLName    xml.Name `xml:"server"`
    ServerName string   `xml:"serverName"`
    ServerIP   string   `xml:"serverIP"`
}

func main() {
    file, err := os.Open("servers.xml") // For read access.     
    if err != nil {
        fmt.Printf("error: %v", err)
        return
    }
    defer file.Close()
    data, err := ioutil.ReadAll(file)
    if err != nil {
        fmt.Printf("error: %v", err)
        return
    }
    v := Recurlyservers{}
    err = xml.Unmarshal(data, &v)
    if err != nil {
        fmt.Printf("error: %v", err)
        return
    }

    fmt.Println(v)
}
```

XML 本质上是一种树形的数据格式，而我们可以定义与之匹配的 go 语言的 struct 类型，然后通过 xml.Unmarshal 来将 xml 中的数据解析成对应的 struct 对象。如上例子输出如下数据

```

{{ servers} 1 [{{ server} Shanghai_VPN 127.0.0.1} {{ server} Beijing_VPN 127.0.0.2}]
<server>
    <serverName>Shanghai_VPN</serverName>
    <serverIP>127.0.0.1</serverIP>
</server>
<server>
    <serverName>Beijing_VPN</serverName>
    <serverIP>127.0.0.2</serverIP>
</server>
}
```

上面的例子中，将 xml 文件解析成对应的 struct 对象是通过 xml.Unmarshal 来完成的，这个过程是如何实现的？可以看到我们的 struct 定义后面多了一些类似于 xml:"serverName" 这样的内容，这个是 struct 的一个特性，它们被称为 struct tag，它们是用来辅助反射的。我们来看一下 Unmarshal 的定义：

```

func Unmarshal(data []byte, v interface{}) error
```

我们看到函数定义了两个参数，**第一个是 XML 数据流，第二个是存储的对应类型，目前支持 struct、slice 和 string**，XML 包内部**采用了反射来进行数据的映射**，所以 v 里面的字段必须是导出的。Unmarshal 解析的时候 XML 元素和字段怎么对应起来的呢？这是有一个优先级读取流程的，**首先会读取 struct tag，如果没有，那么就会对应字段名**。必须注意一点的是解析的时候 tag、字段名、XML 元素都是大小写敏感的，所以必须一一对应字段。

Go 语言的反射机制，可以利用这些 tag 信息来将来自 XML 文件中的数据反射成对应的 struct 对象，

解析 XML 到 struct 的时候遵循如下的规则：

- 如果 struct 的**一个字段是 string 或者 [] byte 类型且它的 tag 含有 ",innerxml"，Unmarshal 将会将此字段所对应的元素内所有内嵌的原始 xml 累加到此字段上**，如上面例子 Description 定义。最后的输出是


```
	<server>
        <serverName>Shanghai_VPN</serverName>
        <serverIP>127.0.0.1</serverIP>
    </server>
    <server>
        <serverName>Beijing_VPN</serverName>
        <serverIP>127.0.0.2</serverIP>
    </server>
```

- 如果 struct 中有**一个叫做 XMLName，且类型为 xml.Name 字段，那么在解析的时候就会保存这个 element 的名字到该字段**，如上面例子中的 servers。
- 如果某个 struct 字段的 **tag 定义中含有 XML 结构中 element 的名称，那么解析的时候就会把相应的 element 值赋值给该字段**，如上 servername 和 serverip 定义。
- 如果某个 struct 字段的 **tag 定义了中含有 ",attr"，那么解析的时候就会将该结构所对应的 element 的与字段同名的属性的值赋值给该字段**，如上 version 定义。
- 如果某个 struct 字段的 **tag 定义 型如 "a>b>c", 则解析的时候，会将 xml 结构 a 下面的 b 下面的 c 元素的值赋值给该字段**。
- 如果某个 struct 字段的 **tag 定义了 "-", 那么不会为该字段解析匹配任何 xml 数据**。
- 如果 struct 字段后面的 **tag 定义了 ",any"，如果他的子元素在不满足其他的规则的时候就会匹配到这个字段**。
- 如果某个 XML 元素**包含一条或者多条注释，那么这些注释将被累加到第一个 tag 含有 ",comments" 的字段上，这个字段的类型可能是 [] byte 或 string, 如果没有这样的字段存在，那么注释将会被抛弃**。

上面详细讲述了如何定义 struct 的 tag。 只要设置对了 tag，那么 XML 解析就如上面示例般简单，tag 和 XML 的 element 是一一对应的关系，如上所示，我们还可以通过 slice 来表示多个同级元素。

注意： 为了正确解析，go 语言的 xml 包要求 struct 定义中的所有字段必须是可导出的（即首字母大写）

**输出 XML**

假若我们不是要解析如上所示的 XML 文件，而是生成它，那么在 go 语言中又该如何实现呢？ xml 包中提供了 Marshal 和 MarshalIndent 两个函数，来满足我们的需求。这两个函数主要的区别是第二个函数会增加前缀和缩进，函数的定义如下所示：

```
func Marshal(v interface{}) ([]byte, error)
func MarshalIndent(v interface{}, prefix, indent string) ([]byte, error)
```


两个函数第一个参数是用来生成 XML 的结构定义类型数据，都是返回生成的 XML 数据流。

下面我们来看一下如何输出如上的 XML：

```
package main

import (
    "encoding/xml"
    "fmt"
    "os"
)

type Servers struct {
    XMLName xml.Name `xml:"servers"`
    Version string   `xml:"version,attr"`
    Svs     []server `xml:"server"`
}

type server struct {
    ServerName string `xml:"serverName"`
    ServerIP   string `xml:"serverIP"`
}

func main() {
    v := &Servers{Version: "1"}
    v.Svs = append(v.Svs, server{"Shanghai_VPN", "127.0.0.1"})
    v.Svs = append(v.Svs, server{"Beijing_VPN", "127.0.0.2"})
    output, err := xml.MarshalIndent(v, "  ", "    ")
    if err != nil {
        fmt.Printf("error: %v\n", err)
    }
    os.Stdout.Write([]byte(xml.Header))

    os.Stdout.Write(output)

}
```

上面的代码输出如下信息：

```
<?xml version="1.0" encoding="UTF-8"?>
<servers version="1">
<server>
    <serverName>Shanghai_VPN</serverName>
    <serverIP>127.0.0.1</serverIP>
</server>
<server>
    <serverName>Beijing_VPN</serverName>
    <serverIP>127.0.0.2</serverIP>
</server>
</servers>
```


和我们之前定义的文件的格式一模一样，之所以会有 os.Stdout.Write([]byte(xml.Header)) 这句代码的出现，是因为 xml.MarshalIndent 或者 xml.Marshal 输出的信息都是不带 XML 头的，为了生成正确的 xml 文件，我们使用了 xml 包预定义的 Header 变量。

我们看到 Marshal 函数接收的参数 v 是 interface {} 类型的，即它可以接受任意类型的参数，那么 xml 包，根据什么规则来生成相应的 XML 文件呢？

- 如果 v 是 array 或者 slice，那么输出每一个元素，类似 value
- 如果 v 是指针，那么会 Marshal 指针指向的内容，如果指针为空，什么都不输出
- 如果 v 是 interface，那么就处理 interface 所包含的数据
- 如果 v 是其他数据类型，就会输出这个数据类型所拥有的字段信息

生成的 XML 文件中的 element 的名字又是根据什么决定的呢？元素名按照如下优先级从 struct 中获取：

- 如果 v 是 struct，XMLName 的 tag 中定义的名称
- 类型为 xml.Name 的名叫 XMLName 的字段的值
- 通过 struct 中字段的 tag 来获取
- 通过 struct 的字段名用来获取
- marshall 的类型名称

我们应如何设置 struct 中字段的 tag 信息以控制最终 xml 文件的生成呢？

- XMLName 不会被输出
- tag 中含有 "-" 的字段不会输出
- tag 中含有 "name,attr"，会以 name 作为属性名，字段值作为值输出为这个 XML 元素的属性，如上 version 字段所描述
- tag 中含有 ",attr"，会以这个 struct 的字段名作为属性名输出为 XML 元素的属性，类似上一条，只是这个 name 默认是字段名了。
- tag 中含有 ",chardata"，输出为 xml 的 character data 而非 element。
- tag 中含有 ",innerxml"，将会被原样输出，而不会进行常规的编码过程
- tag 中含有 ",comment"，将被当作 xml 注释来输出，而不会进行常规的编码过程，字段值中不能含有 "--" 字符串
- tag 中含有 "omitempty", 如果该字段的值为空值那么该字段就不会被输出到 XML，空值包括：false、0、nil 指针或 nil 接口，任何长度为 0 的 array, slice, map 或者 string
- tag 中含有 "a>b>c"，那么就会循环输出三个元素 a 包含 b，b 包含 c，例如如下代码就会输出

    FirstName string   `xml:"name>first"`
    LastName  string   `xml:"name>last"`
    
    <name>
    <first>Asta</first>
    <last>Xie</last>
    </name>

### JSON 处理



### 正则处理



### 模板处理



### 文件操作



### 字符串处理



## 五、Web服务

Web 服务可以让你在 HTTP 协议的基础上通过 XML 或者 JSON 来交换信息。如果你想知道上海的天气预报、中国石油的股价或者淘宝商家的一个商品信息，你可以编写一段简短的代码，通过抓取这些信息然后通过标准的接口开放出来，就如同你调用一个本地函数并返回一个值。

Web 服务背后的关键在于平台的无关性，你可以运行你的服务在 Linux 系统，可以与其他 Windows 的 asp.net 程序交互，同样的，也可以通过同一个接口和运行在 FreeBSD 上面的 JSP 无障碍地通信。

目前主流的有如下几种 Web 服务：REST、SOAP。

REST 请求是很直观的，因为 REST 是基于 HTTP 协议的一个补充，他的每一次请求都是一个 HTTP 请求，然后根据不同的 method 来处理不同的逻辑，很多 Web 开发者都熟悉 HTTP 协议，所以学习 REST 是一件比较容易的事情。所以我们在 8.3 小节将详细的讲解如何在 Go 语言中来实现 REST 方式。

SOAP 是 W3C 在跨网络信息传递和远程计算机函数调用方面的一个标准。但是 SOAP 非常复杂，其完整的规范篇幅很长，而且内容仍然在增加。Go 语言是以简单著称，所以我们不会介绍 SOAP 这样复杂的东西。而 Go 语言提供了一种天生性能很不错，开发起来很方便的 RPC 机制

### Socket 编程

**什么是 Socket？**

Socket 起源于 Unix，而 **Unix 基本哲学之一就是 “一切皆文件”，都可以用 “打开 open –> 读写 write/read –> 关闭 close” 模式来操作**。Socket 就是该模式的一个实现，网络的 Socket 数据传输是一种特殊的 I/O，Socket 也是一种文件描述符。Socket 也具有一个类似于打开文件的函数调用：Socket ()，该函数返回一个整型的 Socket 描述符，随后的连接建立、数据传输等操作都是通过该 Socket 实现的。

常用的 Socket 类型有两种**：流式 Socket（SOCK_STREAM）**和**数据报式 Socket（SOCK_DGRAM）**。**流式是一种面向连接的 Socket，针对于面向连接的 TCP 服务应用；数据报式 Socket 是一种无连接的 Socket，对应于无连接的 UDP 服务应用**。

**Socket 如何通信**

网络中的进程之间如何通过 Socket 通信呢？首要解决的问题是如何唯一标识一个进程，否则通信无从谈起！在本地可以通过进程 PID 来唯一标识一个进程，但是在网络中这是行不通的。其实 TCP/IP 协议族已经帮我们解决了这个问题，**网络层的 “ip 地址”** 可以唯一标识网络中的主机，而**传输层的 “协议 + 端口” 可以唯一标识主机中的应用程序（进程）**。这样**利用三元组（ip 地址，协议，端口）就可以标识网络的进程**了，网络中需要互相通信的进程，就可以利用这个标志在他们之间进行交互。请看下面这个 TCP/IP 协议结构图

使用 TCP/IP 协议的应用程序通常采用应用编程接口：UNIX BSD 的套接字（socket）和 UNIX System V 的 TLI（已经被淘汰），来实现网络进程之间的通信。就目前而言，几乎所有的应用程序都是采用 socket，而现在又是网络时代，网络中进程通信是无处不在，这就是为什么说 “一切皆 Socket”。

**Socket 基础知识**

Socket 有两种：TCP Socket 和 UDP Socket，TCP 和 UDP 是协议，而要确定一个进程的需要三元组，需要 IP 地址和端口。

**IPv4 地址**

目前的全球因特网所采用的协议族是 TCP/IP 协议。IP 是 TCP/IP 协议中网络层的协议，是 TCP/IP 协议族的核心协议。目前主要采用的 IP 协议的版本号是 4 (简称为 IPv4)，发展至今已经使用了 30 多年。

IPv4 的地址位数为 32 位，也就是最多有 2 的 32 次方的网络设备可以联到 Internet 上。近十年来由于互联网的蓬勃发展，IP 位址的需求量愈来愈大，使得 IP 位址的发放愈趋紧张，前一段时间，据报道 IPV4 的地址已经发放完毕，我们公司目前很多服务器的 IP 都是一个宝贵的资源。

地址格式类似这样：127.0.0.1 172.122.121.111

**IPv6 地址**

IPv6 是下一版本的互联网协议，也可以说是下一代互联网的协议，它是为了解决 IPv4 在实施过程中遇到的各种问题而被提出的，IPv6 采用 128 位地址长度，几乎可以不受限制地提供地址。按保守方法估算 IPv6 实际可分配的地址，整个地球的每平方米面积上仍可分配 1000 多个地址。在 IPv6 的设计过程中除了一劳永逸地解决了地址短缺问题以外，还考虑了在 IPv4 中解决不好的其它问题，主要有端到端 IP 连接、服务质量（QoS）、安全性、多播、移动性、即插即用等。

**Go 支持的 IP 类型**

在 Go 的 `net` 包中定义了很多类型、函数和方法用来网络编程，其中 IP 的定义如下：

```
type IP []byte
```

在 `net` 包中有很多函数来操作 IP，但是其中比较有用的也就几个，其中 `ParseIP(s string) IP` 函数会把一个 IPv4 或者 IPv6 的地址转化成 IP 类型，请看下面的例子

```
package main
import (
    "net"
    "os"
    "fmt"
)
func main() {
    if len(os.Args) != 2 {
        fmt.Fprintf(os.Stderr, "Usage: %s ip-addr\n", os.Args[0])
        os.Exit(1)
    }
    name := os.Args[1]
    addr := net.ParseIP(name)
    if addr == nil {
        fmt.Println("Invalid address")
    } else {
        fmt.Println("The address is ", addr.String())
    }
    os.Exit(0)
}
```

**TCP Socket**

当我们知道如何通过网络端口访问一个服务时，那么我们能够做什么呢？作为客户端来说，我们可以通过向远端某台机器的的某个网络端口发送一个请求，然后得到在机器的此端口上监听的服务反馈的信息。作为服务端，我们需要把服务绑定到某个指定端口，并且在此端口上监听，当有客户端来访问时能够读取信息并且写入反馈信息。

在 Go 语言的 net 包中有一个类型 TCPConn，这个类型可以用来作为客户端和服务器端交互的通道，他有两个主要的函数：

```
func (c *TCPConn) Write(b []byte) (int, error)
func (c *TCPConn) Read(b []byte) (int, error)
```

`TCPConn` 可以用在客户端和服务器端来读写数据。

还有我们需要知道一个 `TCPAddr` 类型，他表示一个 TCP 的地址信息，他的定义如下：

```
type TCPAddr struct {
    IP IP
    Port int
    Zone string // IPv6 scoped addressing zone
}
```

在 Go 语言中通过 `ResolveTCPAddr` 获取一个 `TCPAddr`

```
func ResolveTCPAddr(net, addr string) (*TCPAddr, os.Error)
```

- net 参数是 “tcp4”、”tcp6”、”tcp” 中的任意一个，分别表示 TCP (IPv4-only), TCP (IPv6-only) 或者 TCP (IPv4, IPv6 的任意一个)。
- addr 表示域名或者 IP 地址，例如”www.google.com:80" 或者 “127.0.0.1:22”。

**TCP client**

Go 语言中通过 net 包中的 DialTCP 函数来建立一个 TCP 连接，并返回一个 TCPConn 类型的对象，当连接建立时服务器端也创建一个同类型的对象，此时客户端和服务器段通过各自拥有的 TCPConn 对象来进行数据交换。一般而言，客户端通过 TCPConn 对象将请求信息发送到服务器端，读取服务器端响应的信息。服务器端读取并解析来自客户端的请求，并返回应答信息，这个连接只有当任一端关闭了连接之后才失效，不然这连接可以一直在使用。建立连接的函数定义如下：

```
func DialTCP(network string, laddr, raddr *TCPAddr) (*TCPConn, error)
```

- net 参数是 “tcp4”、”tcp6”、”tcp” 中的任意一个，分别表示 TCP (IPv4-only)、TCP (IPv6-only) 或者 TCP (IPv4, IPv6 的任意一个)
- laddr 表示本机地址，一般设置为 nil
- raddr 表示远程的服务地址

接下来我们写一个简单的例子，模拟一个基于 HTTP 协议的客户端请求去连接一个 Web 服务端。我们要写一个简单的 http 请求头，格式类似如下：

```
"HEAD / HTTP/1.0\r\n\r\n"
```

从服务端接收到的响应信息格式可能如下：

```
HTTP/1.0 200 OK
ETag: "-9985996"
Last-Modified: Thu, 25 Mar 2010 17:51:10 GMT
Content-Length: 18074
Connection: close
Date: Sat, 28 Aug 2010 00:43:48 GMT
Server: lighttpd/1.4.23
```

我们的客户端代码如下所示：

```

package main

import (
    "fmt"
    "io/ioutil"
    "net"
    "os"
)

func main() {
    if len(os.Args) != 2 {
        fmt.Fprintf(os.Stderr, "Usage: %s host:port ", os.Args[0])
        os.Exit(1)
    }
    service := os.Args[1]
    tcpAddr, err := net.ResolveTCPAddr("tcp4", service)
    checkError(err)
    conn, err := net.DialTCP("tcp", nil, tcpAddr)
    checkError(err)
    _, err = conn.Write([]byte("HEAD / HTTP/1.0\r\n\r\n"))
    checkError(err)
    result, err := ioutil.ReadAll(conn)
    checkError(err)
    fmt.Println(string(result))
    os.Exit(0)
}
func checkError(err error) {
    if err != nil {
        fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
        os.Exit(1)
    }
}
```

通过上面的代码我们可以看出：首先程序将用户的输入作为参数 service 传入 net.ResolveTCPAddr 获取一个 tcpAddr, 然后把 tcpAddr 传入 DialTCP 后创建了一个 TCP 连接 conn，通过 conn 来发送请求信息，最后通过 ioutil.ReadAll 从 conn 中读取全部的文本，也就是服务端响应反馈的信息。

**TCP server**

上面我们编写了一个 **TCP 的客户端程序**，也可以通过 net 包来创建一个**服务器端程序**，在服务器端我们需要绑定服务到指定的非激活端口，并监听此端口，当有客户端请求到达的时候可以接收到来自客户端连接的请求。net 包中有相应功能的函数，函数定义如下：

```
func ListenTCP(network string, laddr *TCPAddr) (*TCPListener, error)
func (l *TCPListener) Accept() (Conn, error)
```

参数说明同 DialTCP 的参数一样。下面我们实现一个简单的时间同步服务，监听 7777 端口

```

package main

import (
    "fmt"
    "net"
    "os"
    "time"
)

func main() {
    service := ":7777"
    tcpAddr, err := net.ResolveTCPAddr("tcp4", service)
    checkError(err)
    listener, err := net.ListenTCP("tcp", tcpAddr)
    checkError(err)
    for {
        conn, err := listener.Accept()
        if err != nil {
            continue
        }
        daytime := time.Now().String()
        conn.Write([]byte(daytime)) // don't care about return value
        conn.Close()                // we're finished with this client
    }
}
func checkError(err error) {
    if err != nil {
        fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
        os.Exit(1)
    }
}
```

上面的服务跑起来之后，它将会一直在那里等待，直到有新的客户端请求到达。当有新的客户端请求到达并同意接受 Accept 该请求的时候他会反馈当前的时间信息。值得注意的是，在代码中 for 循环里，当有错误发生时，直接 continue 而不是退出，是因为在服务器端跑代码的时候，当有错误发生的情况下最好是由服务端记录错误，然后当前连接的客户端直接报错而退出，从而不会影响到当前服务端运行的整个服务。

上面的代码有个缺点，执行的时候是单任务的，不能同时接收多个请求，那么该如何改造以使它支持多并发呢？Go 里面有一个 goroutine 机制，请看下面改造后的代码

```

package main

import (
    "fmt"
    "net"
    "os"
    "time"
)

func main() {
    service := ":1200"
    tcpAddr, err := net.ResolveTCPAddr("tcp4", service)
    checkError(err)
    listener, err := net.ListenTCP("tcp", tcpAddr)
    checkError(err)
    for {
        conn, err := listener.Accept()
        if err != nil {
            continue
        }
        go handleClient(conn)
    }
}

func handleClient(conn net.Conn) {
    defer conn.Close()
    daytime := time.Now().String()
    conn.Write([]byte(daytime)) // don't care about return value
    // we're finished with this client
}
func checkError(err error) {
    if err != nil {
        fmt.Fprintf(os.Stderr, "Fatal error: %s", err.Error())
        os.Exit(1)
    }
}
```

**控制 TCP 连接**

TCP 有很多连接控制函数

**UDP Socket**

Go 语言包中处理 UDP Socket 和 TCP Socket 不同的地方就是在服务器端处理多个客户端请求数据包的方式不同，**UDP 缺少了对客户端连接请求的 Accept 函数**。**其他基本几乎一模一样**，只有 TCP 换成了 UDP 而已。UDP 的几个主要函数如下所示：

```
func ResolveUDPAddr(net, addr string) (*UDPAddr, os.Error)
func DialUDP(net string, laddr, raddr *UDPAddr) (c *UDPConn, err os.Error)
func ListenUDP(net string, laddr *UDPAddr) (c *UDPConn, err os.Error)
func (c *UDPConn) ReadFromUDP(b []byte) (n int, addr *UDPAddr, err os.Error)
func (c *UDPConn) WriteToUDP(b []byte, addr *UDPAddr) (n int, err os.Error)
```

一个 UDP 的客户端代码如下所示，我们可以看到不同的就是 TCP 换成了 UDP 而已：

```

package main

import (
    "fmt"
    "net"
    "os"
)

func main() {
    if len(os.Args) != 2 {
        fmt.Fprintf(os.Stderr, "Usage: %s host:port", os.Args[0])
        os.Exit(1)
    }
    service := os.Args[1]
    udpAddr, err := net.ResolveUDPAddr("udp4", service)
    checkError(err)
    conn, err := net.DialUDP("udp", nil, udpAddr)
    checkError(err)
    _, err = conn.Write([]byte("anything"))
    checkError(err)
    var buf [512]byte
    n, err := conn.Read(buf[0:])
    checkError(err)
    fmt.Println(string(buf[0:n]))
    os.Exit(0)
}
func checkError(err error) {
    if err != nil {
        fmt.Fprintf(os.Stderr, "Fatal error ", err.Error())
        os.Exit(1)
    }
}
```

我们来看一下 UDP 服务器端如何来处理：

```

package main

import (
    "fmt"
    "net"
    "os"
    "time"
)

func main() {
    service := ":1200"
    udpAddr, err := net.ResolveUDPAddr("udp4", service)
    checkError(err)
    conn, err := net.ListenUDP("udp", udpAddr)
    checkError(err)

    handleClient(conn)
}
func handleClient(conn *net.UDPConn) {
    var buf [512]byte
    _, addr, err := conn.ReadFromUDP(buf[0:])
    if err != nil {
        return
    }
    daytime := time.Now().String()
    conn.WriteToUDP([]byte(daytime), addr)
}
func checkError(err error) {
    if err != nil {
        fmt.Fprintf(os.Stderr, "Fatal error ", err.Error())
        os.Exit(1)
    }
}
```

### WebSocket



### REST

RESTful，是目前最为流行的一种互联网软件架构。因为它结构清晰、符合标准、易于理解、扩展方便，所以正得到越来越多网站的采用。本小节我们将来学习它到底是一种什么样的架构？以及在 Go 里面如何来实现它。

**REST (REpresentational State Transfer)** 这个概念，首次出现是在 2000 年 Roy Thomas Fielding（他 是 HTTP 规范的主要编写者之一）的博士论文中，它指的是一组架构约束条件和原则。满足这些约束条件和原则的应用程序或设计就是 RESTful 的。

要理解什么是 REST，我们需要理解下面几个概念:

- 资源（Resources）
  REST 是 "表现层状态转化"，其实它省略了主语。"表现层" 其实指的是 "资源" 的 "表现层"。

  那么什么是资源呢？就是我们平常上网访问的一张图片、一个文档、一个视频等。这些资源我们通过 URI 来定位，也就是一个 URI 表示一个资源。

- 表现层（Representation）

  资源是做一个具体的实体信息，他可以有多种的展现方式。而把实体展现出来就是表现层，例如一个 txt 文本信息，他可以输出成 html、json、xml 等格式，一个图片他可以 jpg、png 等方式展现，这个就是表现层的意思。

  URI 确定一个资源，但是如何确定它的具体表现形式呢？应该在 HTTP 请求的头信息中用 Accept 和 Content-Type 字段指定，这两个字段才是对 "表现层" 的描述。

- 状态转化（State Transfer）

  访问一个网站，就代表了客户端和服务器的一个互动过程。在这个过程中，肯定涉及到数据和状态的变化。而 HTTP 协议是无状态的，那么这些状态肯定保存在服务器端，所以如果客户端想要通知服务器端改变数据和状态的变化，肯定要通过某种方式来通知它。

  客户端能通知服务器端的手段，只能是 HTTP 协议。具体来说，就是 HTTP 协议里面，四个表示操作方式的动词：GET、POST、PUT、DELETE。它们分别对应四种基本操作：**GET 用来获取资源，POST 用来新建资源（也可以用于更新资源），PUT 用来更新资源，DELETE 用来删除资源。**

综合上面的解释，我们总结一下什么是 RESTful 架构：

（1）每一个 URI 代表一种资源；

（2）客户端和服务器之间，传递这种资源的某种表现层；

（3）客户端通过四个 HTTP 动词，对服务器端资源进行操作，实现 "表现层状态转化"。

Web 应用要满足 REST 最重要的原则是：**客户端和服务器之间的交互在请求之间是无状态的，即从客户端到服务器的每个请求都必须包含理解请求所必需的信息。如果服务器在请求之间的任何时间点重启，客户端不会得到通知。**此外此请求可以由任何可用服务器回答，这十分适合云计算之类的环境。因为是无状态的，所以客户端可以缓存数据以改进性能。

另一个重要的 REST 原则是系统分层，这表示组件无法了解除了与它直接交互的层次以外的组件。通过将系统知识限制在单个层，可以限制整个系统的复杂性，从而促进了底层的独立性。

### RPC

RPC 就是想实现函数调用模式的网络化。客户端就像调用本地函数一样，然后客户端把这些参数打包之后通过网络传递到服务端，服务端解包到处理过程中执行，然后执行的结果反馈给客户端。

RPC（Remote Procedure Call Protocol）—— 远程过程调用协议，是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。它假定某些传输协议的存在，如 TCP 或 UDP，以便为通信程序之间携带信息数据。通过它可以使函数调用模式网络化。在 OSI 网络通信模型中，RPC 跨越了传输层和应用层。RPC 使得开发包括网络分布式多程序在内的应用程序更加容易。

**RPC 工作原理**

![RPC 工作流程图](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/8.4.rpc.png)

运行时，一次客户机对服务器的 RPC 调用，其内部操作大致有如下十步：

1. 调用客户端句柄；执行传送参数
2. 调用本地系统内核发送网络消息
3. 消息传送到远程主机
4. 服务器句柄得到消息并取得参数
5. 执行远程过程
6. 执行的过程将结果返回服务器句柄
7. 服务器句柄返回结果，调用远程系统内核
8. 消息传回本地主机
9. 客户句柄由内核接收消息
10. 客户接收句柄返回的数据

**Go RPC**

Go 标准包中已经提供了对 RPC 的支持，而且支持三个级别的 RPC：TCP、HTTP、JSONRPC。但 Go 的 RPC 包是独一无二的 RPC，它和传统的 RPC 系统不同，它只支持 Go 开发的服务器与客户端之间的交互，因为在内部，它们采用了 Gob 来编码。

Go RPC 的函数只有符合下面的条件才能被远程访问，不然会被忽略，详细的要求如下：

- 函数必须是导出的 (首字母大写)
- 必须有两个导出类型的参数，
- 第一个参数是接收的参数，第二个参数是返回给客户端的参数，第二个参数必须是指针类型的
- 函数还要有一个返回值 error

举个例子，正确的 RPC 函数格式如下：

```
func (t *T) MethodName(argType T1, replyType *T2) error
```

T、T1 和 T2 类型必须能被 encoding/gob 包编解码。

任何的 RPC 都需要通过网络来传递数据，Go RPC 可以利用 HTTP 和 TCP 来传递数据，利用 HTTP 的好处是可以直接复用 net/http 里面的一些函数。详细的例子请看下面的实现

**HTTP RPC**

http 的服务端代码实现如下：

```

package main

import (
    "errors"
    "fmt"
    "net/http"
    "net/rpc"
)

type Args struct {
    A, B int
}

type Quotient struct {
    Quo, Rem int
}

type Arith int

func (t *Arith) Multiply(args *Args, reply *int) error {
    *reply = args.A * args.B
    return nil
}

func (t *Arith) Divide(args *Args, quo *Quotient) error {
    if args.B == 0 {
        return errors.New("divide by zero")
    }
    quo.Quo = args.A / args.B
    quo.Rem = args.A % args.B
    return nil
}

func main() {

    arith := new(Arith)
    rpc.Register(arith)
    rpc.HandleHTTP()

    err := http.ListenAndServe(":1234", nil)
    if err != nil {
        fmt.Println(err.Error())
    }
}
```

通过上面的例子可以看到，我们注册了一个 Arith 的 RPC 服务，然后通过 rpc.HandleHTTP 函数把该服务注册到了 HTTP 协议上，然后我们就可以利用 http 的方式来传递数据了。

请看下面的客户端代码：

```

package main

import (
    "fmt"
    "log"
    "net/rpc"
    "os"
)

type Args struct {
    A, B int
}

type Quotient struct {
    Quo, Rem int
}

func main() {
    if len(os.Args) != 2 {
        fmt.Println("Usage: ", os.Args[0], "server")
        os.Exit(1)
    }
    serverAddress := os.Args[1]

    client, err := rpc.DialHTTP("tcp", serverAddress+":1234")
    if err != nil {
        log.Fatal("dialing:", err)
    }
    // Synchronous call
    args := Args{17, 8}
    var reply int
    err = client.Call("Arith.Multiply", args, &reply)
    if err != nil {
        log.Fatal("arith error:", err)
    }
    fmt.Printf("Arith: %d*%d=%d\n", args.A, args.B, reply)

    var quot Quotient
    err = client.Call("Arith.Divide", args, &quot)
    if err != nil {
        log.Fatal("arith error:", err)
    }
    fmt.Printf("Arith: %d/%d=%d remainder %d\n", args.A, args.B, quot.Quo, quot.Rem)

}
```

我们把上面的服务端和客户端的代码分别编译，然后先把服务端开启，然后开启客户端，输入代码，就会输出如下信息：

```

$ ./http_c localhost
Arith: 17*8=136
Arith: 17/8=2 remainder 1
```

通过上面的调用可以看到参数和返回值是我们定义的 struct 类型，在服务端我们把它们当做调用函数的参数的类型，在客户端作为 client.Call 的第 2，3 两个参数的类型。客户端最重要的就是这个 Call 函数，它有 3 个参数，第 1 个要调用的函数的名字，第 2 个是要传递的参数，第 3 个要返回的参数 (注意是指针类型)，通过上面的代码例子我们可以发现，使用 Go 的 RPC 实现相当的简单，方便。

**TCP RPC**

上面我们实现了基于 HTTP 协议的 RPC，接下来我们要实现基于 TCP 协议的 RPC，服务端的实现代码如下所示：

```

package main

import (
    "errors"
    "fmt"
    "net"
    "net/rpc"
    "os"
)

type Args struct {
    A, B int
}

type Quotient struct {
    Quo, Rem int
}

type Arith int

func (t *Arith) Multiply(args *Args, reply *int) error {
    *reply = args.A * args.B
    return nil
}

func (t *Arith) Divide(args *Args, quo *Quotient) error {
    if args.B == 0 {
        return errors.New("divide by zero")
    }
    quo.Quo = args.A / args.B
    quo.Rem = args.A % args.B
    return nil
}

func main() {

    arith := new(Arith)
    rpc.Register(arith)

    tcpAddr, err := net.ResolveTCPAddr("tcp", ":1234")
    checkError(err)

    listener, err := net.ListenTCP("tcp", tcpAddr)
    checkError(err)

    for {
        conn, err := listener.Accept()
        if err != nil {
            continue
        }
        rpc.ServeConn(conn)
    }

}

func checkError(err error) {
    if err != nil {
        fmt.Println("Fatal error ", err.Error())
        os.Exit(1)
    }
}
```

上面这个代码和 http 的服务器相比，不同在于：在此处我们采用了 TCP 协议，然后需要自己控制连接，当有客户端连接上来后，我们需要把这个连接交给 rpc 来处理。

如果你留心了，你会发现这它是一个阻塞型的单用户的程序，如果想要实现多并发，那么可以使用 goroutine 来实现，我们前面在 socket 小节的时候已经介绍过如何处理 goroutine。
下面展现了 TCP 实现的 RPC 客户端：

```

package main

import (
    "fmt"
    "log"
    "net/rpc"
    "os"
)

type Args struct {
    A, B int
}

type Quotient struct {
    Quo, Rem int
}

func main() {
    if len(os.Args) != 2 {
        fmt.Println("Usage: ", os.Args[0], "server:port")
        os.Exit(1)
    }
    service := os.Args[1]

    client, err := rpc.Dial("tcp", service)
    if err != nil {
        log.Fatal("dialing:", err)
    }
    // Synchronous call
    args := Args{17, 8}
    var reply int
    err = client.Call("Arith.Multiply", args, &reply)
    if err != nil {
        log.Fatal("arith error:", err)
    }
    fmt.Printf("Arith: %d*%d=%d\n", args.A, args.B, reply)

    var quot Quotient
    err = client.Call("Arith.Divide", args, &quot)
    if err != nil {
        log.Fatal("arith error:", err)
    }
    fmt.Printf("Arith: %d/%d=%d remainder %d\n", args.A, args.B, quot.Quo, quot.Rem)

}
```

**JSON RPC**

JSON RPC 是数据编码采用了 JSON，而不是 gob 编码，其他和上面介绍的 RPC 概念一模一样，下面我们来演示一下，如何使用 Go 提供的 json-rpc 标准包，请看服务端代码的实现：

```

package main

import (
    "errors"
    "fmt"
    "net"
    "net/rpc"
    "net/rpc/jsonrpc"
    "os"
)

type Args struct {
    A, B int
}

type Quotient struct {
    Quo, Rem int
}

type Arith int

func (t *Arith) Multiply(args *Args, reply *int) error {
    *reply = args.A * args.B
    return nil
}

func (t *Arith) Divide(args *Args, quo *Quotient) error {
    if args.B == 0 {
        return errors.New("divide by zero")
    }
    quo.Quo = args.A / args.B
    quo.Rem = args.A % args.B
    return nil
}

func main() {

    arith := new(Arith)
    rpc.Register(arith)

    tcpAddr, err := net.ResolveTCPAddr("tcp", ":1234")
    checkError(err)

    listener, err := net.ListenTCP("tcp", tcpAddr)
    checkError(err)

    for {
        conn, err := listener.Accept()
        if err != nil {
            continue
        }
        jsonrpc.ServeConn(conn)
    }

}

func checkError(err error) {
    if err != nil {
        fmt.Println("Fatal error ", err.Error())
        os.Exit(1)
    }
}
```

通过示例我们可以看出 json-rpc 是基于 TCP 协议实现的，目前它还不支持 HTTP 方式。

请看客户端的实现代码：

```

package main

import (
    "fmt"
    "log"
    "net/rpc/jsonrpc"
    "os"
)

type Args struct {
    A, B int
}

type Quotient struct {
    Quo, Rem int
}

func main() {
    if len(os.Args) != 2 {
        fmt.Println("Usage: ", os.Args[0], "server:port")
        log.Fatal(1)
    }
    service := os.Args[1]

    client, err := jsonrpc.Dial("tcp", service)
    if err != nil {
        log.Fatal("dialing:", err)
    }
    // Synchronous call
    args := Args{17, 8}
    var reply int
    err = client.Call("Arith.Multiply", args, &reply)
    if err != nil {
        log.Fatal("arith error:", err)
    }
    fmt.Printf("Arith: %d*%d=%d\n", args.A, args.B, reply)

    var quot Quotient
    err = client.Call("Arith.Divide", args, &quot)
    if err != nil {
        log.Fatal("arith error:", err)
    }
    fmt.Printf("Arith: %d/%d=%d remainder %d\n", args.A, args.B, quot.Quo, quot.Rem)

}
```

## 六、安全与加密

###  预防CSRF攻击



### 确保输入过滤



### 避免 XSS 攻击



###  避免 SQL 注入



### 存储密码



### 加密和解密数据



## 七、国际化和本地化



## 八、错误处理



## 九、部署与维护

### 应用日志



### 网站错误处理



### 应用部署



### 备份和恢复



## 十、如何设计一个 Web 框架

### 项目规划

**gopath 以及项目设置**

假设指定 gopath 是文件系统的普通目录名，当然我们可以随便设置一个目录名，然后将其路径存入 GOPATH。前面介绍过 GOPATH 可以是多个目录：在 window 系统设置环境变量；在 Linux/MacOS 系统只要输入终端命令 export gopath=/home/astaxie/gopath，但是必须保证 gopath 这个代码目录下面有三个目录 pkg、bin、src。新建项目的源码放在 src 目录下面

**应用程序流程图**

博客系统是基于模型 - 视图 - 控制器这一设计模式的。MVC 是一种将应用程序的逻辑层和表现层进行分离的结构方式。在实践中，由于表现层从 Go 中分离了出来，所以它允许你的网页中只包含很少的脚本。

- **模型 (Model) 代表数据结构**。通常来说，模型类将包含取出、插入、更新数据库资料等这些功能。
- **视图 (View) 是展示给用户的信息的结构及样式**。一个视图通常是一个网页，但是在 Go 中，一个视图也可以是一个页面片段，如页头、页尾。它还可以是一个 RSS 页面，或其它类型的 “页面”，Go 实现的 template 包已经很好的实现了 View 层中的部分功能。
- **控制器 (Controller) 是模型、视图以及其他任何处理 HTTP 请求所必须的资源之间的中介**，并生成网页。
  下图显示了项目设计中框架的数据流是如何贯穿整个系统:

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/13.1.flow.png)

图 13.3 框架的数据流

1. main.go 作为应用入口，初始化一些运行博客所需要的基本资源，配置信息，监听端口。
2. 路由功能检查 HTTP 请求，根据 URL 以及 method 来确定谁 (控制层) 来处理请求的转发资源。
3. 如果缓存文件存在，它将绕过通常的流程执行，被直接发送给浏览器。
4. 安全检测：应用程序控制器调用之前，HTTP 请求和任一用户提交的数据将被过滤。
5. 控制器装载模型、核心库、辅助函数，以及任何处理特定请求所需的其它资源，控制器主要负责处理业务逻辑。
6. 输出视图层中渲染好的即将发送到 Web 浏览器中的内容。如果开启缓存，视图首先被缓存，将用于以后的常规请求。

**目录结构**

根据上面的应用程序流程设计，博客的目录结构设计如下：

```
|——main.go         入口文件
|——conf            配置文件和处理模块
|——controllers     控制器入口
|——models          数据库处理模块
|——utils           辅助函数库
|——static          静态文件目录
|——views           视图库
```

**框架设计**

为了实现博客的快速搭建，打算基于上面的流程设计开发一个最小化的框架，框架包括路由功能、支持 REST 的控制器、自动化的模板渲染，日志系统、配置管理等。

### 自定义路由器设计



### controller 设计



### 日志和配置设计
