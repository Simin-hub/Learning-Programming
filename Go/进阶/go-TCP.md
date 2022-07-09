# TCP/IP

### 零拷贝

[参考](https://www.modb.pro/db/212924)、[参考](https://segmentfault.com/a/1190000040160235)

#### 1.数据拷贝



![img](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20211216_1f57929e-5e2f-11ec-92c9-fa163eb4f6be.png)

数据拷贝的模式，每一次拷贝我都打上了序号

1. DMA控制器将数据**从磁盘拷贝到内核缓冲区**，这是**第1次拷贝（DMA拷贝）**
2. CPU将数据**从内核缓冲区复制到应用程序缓冲区**，这是**第2次拷贝（CPU拷贝）（内核态=>用户态）**
3. CPU将数据**从应用程序缓冲区复制到Socket缓冲区**，这是**第3次拷贝（CPU拷贝）**
4. DMA控制器**将数据从Socket缓冲区拷贝到网卡**，这是**第4次拷贝（DMA拷贝）（用户态=>内核态）**

由上述信息可得：一共经历了四次拷贝，其中两次是CPU拷贝，两次是DMA拷贝;经历了两次的状态切换（我就拷贝个数据怎么这么麻烦）

#### 2.MMAP  

![img](https://oss-emcsprod-public.modb.pro/wechatSpider/modb_20211216_1f7b74ca-5e2f-11ec-92c9-fa163eb4f6be.png)

mmap主要是 `内存映射`

这项技术的出现，下面这段关于内存映射的介绍摘自网络

```
内存映射文件技术是操作系统提供的一种新的文件数据存取机制，利用内存映射文件技术，系统可以在内存空间中为文件保留一部分空间，并将文件映射到这块保留空间，一旦文件被映射后，操作系统将管理页映射缓冲以及高速缓冲等任务，而不需要调用分配、释放内存块和文件输入/输出的API函数，也不需要自己提供任何缓冲算法。
使用内存映射文件处理存储于磁盘上的文件时，将不必再对文件执行I/O 操作，这意味着在对文件进行处理时将不必再为文件申请并分配缓存，所有的文件缓存操作均由系统直接管理，由于取消了将文件数据加载到内存、数据从内存到文件的回写以及释放内存块等步骤，使得内存映射文件在处理大数据量的文件时能起到相当重要的作用。
```

通过mmap这项技术，我们可以实现了**避免将数据拷贝出内核空间**了，但是仍然存在一次CPU拷贝，CPU是非常珍贵的资源，并且这个mmap的模式除了CPU这次拷贝之外（其实在mmap出现的时候还是很棒的，不过我们现在有了新的认知了），还存在着另外一个问题，就是**可能出现碎片问题跟多进程下同时操作文件时可能产生引发coredump的signal**。

碎片问题主要是体现在，拷贝的时候，可能是小文件，如果是大文件就会大大降低这种碎片问题的出现。（碎片问题主要是我们查内存还有很多，但是申请大内存会有申请失败的情况出现，原理可以自行查看，主要是顺序分配内存与整块分配内存相关的）

当对文件进行了内存映射，然后调用 write() 系统调用，如果此时其他的进程截断了这个文件，那么 write() 系统调用将会被总线错误信号 SIGBUS 中断，因为此时正在执行的是一个错误的存储访问。该信号的默认行为是杀死进程和转储核心。（源于操作系统对于进程的内存保护机制）

**「当然上面所说存在的两个问题是可以通过其他方法解决的」**

①**「对SIGBUS捕捉处理」**

对SIGBUS 信号进行简单处理并返回，这样，write() 系统调用在它被中断之前就返回已经写入的字节数目，errno 会被设置成 success。（说白了就是让程序不会出现coredump）**「缺点」**：它不能反映出产生这个问题的根源所在，因为 BIGBUS 信号只是显示某进程发生了一些很严重的错误。

②**「文件租借锁」**

当进程尝试打开一个被租借锁保护的文件时，该进程会被阻塞，同时，在一定时间内拥有该文件租借锁的进程会收到一个信号。收到信号之后，拥有该文件租借锁的进程会首先更新文件，从而保证了文件内容的一致性，接着，该进程释放这个租借锁。如果拥有租借锁的进程在一定的时间间隔内没有完成工作，内核就会自动删除这个租借锁或者将该锁进行降级，从而允许被阻塞的进程继续工作。**「注意」**：文件租借锁需要在对文件进行内存映射之前设置。

#### 3.sendfile 

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/modb_20211216_1fadb14c-5e2f-11ec-92c9-fa163eb4f6be.png)

前面几次到Socket当中我们都是把完整的数据拷贝带Socket缓冲区当中，但是经过思考，反正数据是最终拷贝到网卡当中，也就是Socket缓冲区又是一个中间者而已，我们何不想个方法，能把数据直接拷贝到网卡，这个就是我们的sendfile了，我们**拷贝去socket缓冲区只拷贝了文件描述符跟数据长度，然后直接采用DMA收集拷贝内核缓冲区的数据到网卡**当中。

#### 4.只谈理论不实战就是耍流氓

以下是使用零拷贝从服务器当中拿文件进行响应返回

```
package main

import (
        "bytes"
        "flag"
        "fmt"
        "io"
        "io/ioutil"
        "log"
        "net"
        "os"
        "runtime"
        "sync"
        "syscall"
        "text/template"
)
const (
        DEFAULT_HOST      = "127.0.0.1"
        DEFAULT_PORT      = "8080"
        DEFAULT_MIME_TYPE = "text/plain"
)
var (
        src      *os.File
        size     int64
        headers  string
        offsetsz int     = 4096
        offset   []int64 = make([]int64, offsetsz, offsetsz)
        srcfd    int
        mutex    sync.Mutex
)
func main()  {
        var oerr error
        host, port, mimetype, procs, filename := parseArgs()
        fmt.Printf("服务器文件名为： %s 描述消息内容类型为： %s\n", filename, mimetype)
        fmt.Printf("监听的地址：端口 %s:%s\n", host, port)
        // 设置使用多少核，值得注意的是golang默认使用的是单核处理
        runtime.GOMAXPROCS(procs)
        log.Println("设置的核心数为： ", procs)
        src, oerr = os.Open(filename)
        if oerr != nil {
                log.Fatal("Error opening payload. ", oerr)
        }
        fileinfo, serr := src.Stat()
        if serr != nil {
                log.Fatal("Error Stat on payload")
        }
        size = fileinfo.Size()
        srcfd = int(src.Fd())
        log.Println("文件大小为：",size)
        log.Println("文件描述符为：",srcfd)
        tmpl, terr := template.New("headers").Parse(HEAD_TMPL)
        if terr != nil {
                log.Fatal("Error parsing HEAD_TMPL", terr)
        }
        tmplData := struct {
                Mime   string
                Length int64
        }{mimetype, size}
        headBuf := &bytes.Buffer{}
        terr = tmpl.Execute(headBuf, tmplData)
        if terr != nil {
                log.Fatal("Error executing header template", terr)
        }
        headers = headBuf.String()
        // 预热一下操作系统的页缓存
        _, _ = ioutil.ReadAll(src)
        _, _ = src.Seek(0, os.SEEK_SET)

        addr := host + ":" + port
        sock, lerr := net.Listen("tcp", addr)
        if lerr != nil {
                log.Fatal("Error listening on ", addr, ". ", lerr)
        }
        for {
                conn, aerr := sock.Accept()
                if aerr != nil {
                        log.Fatal("Error Accept. ", aerr)
                }
                go handle(conn)
        }
}
func handle(conn net.Conn) {
        log.Println("handle")
        var rerr, werr error
        var wrote int
        buf := make([]byte, 32*1024)
        wrote, werr = conn.Write([]byte(headers))
        if werr != nil {
                log.Fatal("Error writing headers", werr)
        }
        if wrote != len([]byte(headers)) {
                log.Fatal("Error: Wrote ", wrote, " headers bytes. Expected ", len([]byte(headers)))
        }
        outfile, ferr := conn.(*net.TCPConn).File()
        if ferr != nil {
                log.Fatal("Error getting conn fd", ferr)
        }
        outfd := int(outfile.Fd())
        if outfd >= offsetsz {
                growOffset(outfd)
        }
        currOffset := &offset[outfd]
        for *currOffset < size {
        // 零拷贝我是接收到了一个连接，可是我采用syscall.Sendfile()，我怎么转发这个端口？？？？
        // 思考1：我如果read出来，那么就会把数据读取了，那这样 网卡=》应用缓冲区
        // sendfile有四个参数：outfd int, infd int, offset *int64, count int
        //outfd是带读出内容的文件描述符、infd是待写入的内容的文件描述符、
        //offset是指定从文件流的哪个位置开始读（为空默认从头开始读）、count参数指定文件描述符in_fd和out_fd之间传输的字节数
        // in_fd必须是一个支持类似mmap函数的文件描述符（也就是必须指向真实文件）、out_fd是一个socket
                wrote, werr = syscall.Sendfile(outfd, srcfd, currOffset, int(size))
                if werr != nil {
                        log.Fatal("Sendfile error:", werr)
                }
        }
        offset[outfd] = 0
        werr = conn.(*net.TCPConn).CloseWrite()
        if werr != nil {
                log.Println("Error on CloseWrite", werr)
        }
        // Consume input
        for {
                _, rerr = conn.Read(buf)
                if rerr == io.EOF {
                        break
                } else if rerr != nil {
                        log.Println("Error consuming read input: ", rerr)
                        break
                }
        }
        werr = outfile.Close()
        if werr != nil {
                log.Println("Error on outfile Close", werr)
        }
        werr = conn.Close()
        if werr != nil {
                log.Println("Error on Close", werr)
        }
}
func growOffset(outfd int) {
        //  只允许一个协程来增长切片的偏移，否则会造成数据混乱
        mutex.Lock()
        // 加多一层校验，判断是否还需要这样去增长偏移（可能其他协程已经做完离开了）
        if outfd < offsetsz {
                mutex.Unlock()
                return
        }
        newSize := offsetsz * 2
        log.Println("Growing offset to:", newSize)
        newOff := make([]int64, newSize, newSize)
        copy(newOff, offset)
        offset = newOff
        offsetsz = newSize
        mutex.Unlock()
}
// 以下都是输入参数使用的，zero_copy.exe [options] [filename] 如果不想输入一些默认的ip与端口就直接输入一个文件名
func parseArgs() (host, port, mimetype string, procs int, filename string) {
        flag.Usage = Usage
        hostf := flag.String("h", DEFAULT_HOST, "Host or IP to listen on")
        portf := flag.String("p", DEFAULT_PORT, "Port to listen on")
        mimetypef := flag.String("m", DEFAULT_MIME_TYPE, "Mime type of file")
        procsf := flag.Int("c", 1, "Concurrent CPU cores to use.")
        if len(os.Args) < 2 {
                flag.Usage()
                os.Exit(1)
        }
        flag.Parse()
        return *hostf, *portf, *mimetypef, *procsf, flag.Arg(0)
}
func Usage() {
        _, _ = fmt.Fprintf(os.Stderr, "Usage of %s:\n", os.Args[0])
        _, _ = fmt.Fprintf(os.Stderr, "  %s [options] [filename]\n", os.Args[0])
        _, _ = fmt.Fprintf(os.Stderr, "Options:\n")
        flag.PrintDefaults()
}
```

## 5.零拷贝网关

以下是转发网络IO，零拷贝转发，有BUG的，等有缘人看看能不能先帮我修了，我暂时看不出问题。

```
package main

import (
   "flag"
   "fmt"
   "log"
   "net"
   "os"
   "runtime"
   "sync"
   "syscall"
)
var (
   src      *os.File
   size     int64
   headers  string
   offsetsz int     = 4096
   offset   []int64 = make([]int64, offsetsz, offsetsz)
   srcfd    int
   mutex    sync.Mutex
)
const (
   P_DEFAULT_HOST      = "127.0.0.1"
   P_DEFAULT_PORT      = "8080"
   P_DEFAULT_MIME_TYPE = "text/plain"
   P_HEAD_TMPL         = "HTTP/1.0 200 OK\r\nCache-Control: max-age=31536000\r\nExpires: Thu, 31 Dec 2037 23:55:55 GMT\r\nContent-Type: {{.Mime}}\r\nContent-Length: {{.Length}}\r\n\r\n"
)
func main() {
   host, port, mimetype, procs := parseArgs()
   runtime.GOMAXPROCS(procs)
   fmt.Printf("监听的地址：端口 %s:%s\n", host, port)
   log.Println("设置的核心数为： ", procs)
   fmt.Printf("描述消息内容类型为： %s\n", mimetype)
   addr := host + ":" + port
   sock, lerr := net.Listen("tcp", addr)
   if lerr != nil {
      log.Fatal("网关启动失败 ", addr, ". ", lerr)
   }
   for {
      conn, aerr := sock.Accept()
      if aerr != nil {
         log.Fatal("Error Accept. ", aerr)
      }
      // 另外起一个协程去处理这个事，这里推荐用携程池，任何用到协程的地方都要特别注意协程的数量
      go handle(conn)
   }
}
func handle(conn net.Conn) {
   log.Println("开始调用转发······")
   // 我需要转发到下面这个地址当中
   cli_conn, err := net.Dial("tcp", "127.0.0.1:10000")
   if err != nil {
      log.Println("connect error",err)
      return
   }
   defer cli_conn.Close()
   srcfile, ferr := conn.(*net.TCPConn).File()
   outfile, err := cli_conn.(*net.TCPConn).File()
   if ferr != nil {
      log.Fatal("TCP连接拿到的文件描述符错误：", ferr)
   }
   srcfd := int(srcfile.Fd())
   outfd := int(outfile.Fd())
   if srcfd >= offsetsz {
      growOffset(srcfd)
   }
   currOffset := &offset[srcfd]
   // 零拷贝我是接收到了一个连接，可是我采用syscall.Sendfile()，我怎么转发这个端口？？？？
   // 思考1：我如果read出来，那么就会把数据读取了，那这样 网卡=》应用缓冲区
   // sendfile有四个参数：outfd int, infd int, offset *int64, count int
   //outfd是带读出内容的文件描述符、infd是待写入的内容的文件描述符、
   //offset是指定从文件流的哪个位置开始读（为空默认从头开始读）、count参数指定文件描述符in_fd和out_fd之间传输的字节数
   // in_fd必须是一个支持类似mmap函数的文件描述符（也就是必须指向真实文件）、out_fd是一个socket
   for *currOffset < size {
      // 需要解决的一个问题就是从哪去得到这个需要发送的目标socket缓存区，由这个位置读取到网卡进一步转发，cli_conn怎么去拿outfd
      _, werr := syscall.Sendfile(outfd, srcfd, currOffset, int(size))
      if werr != nil {
         log.Fatal("系统调用Sendfile发送错误:", werr)
      }
   }
}
func growOffset(outfd int) {
   //  只允许一个协程来增长切片的偏移，否则会造成数据混乱
   mutex.Lock()
   // 加多一层校验，判断是否还需要这样去增长偏移（可能其他协程已经做完离开了）
   if outfd < offsetsz {
      mutex.Unlock()
      return
   }
   newSize := offsetsz * 2
   log.Println("Growing offset to:", newSize)
   newOff := make([]int64, newSize, newSize)
   copy(newOff, offset)
   offset = newOff
   offsetsz = newSize
   mutex.Unlock()
}
// 以下都是输入参数使用的，zero_copy.exe [options] 如果不想输入一些默认的ip与端口就直接输入一个文件名
func parseArgs() (host, port, mimetype string, procs int) {
   flag.Usage = Usage
   hostf := flag.String("h", P_DEFAULT_HOST, "Host or IP to listen on")
   portf := flag.String("p", P_DEFAULT_PORT, "Port to listen on")
   mimetypef := flag.String("m", P_DEFAULT_MIME_TYPE, "Mime type of file")
   procsf := flag.Int("c", 1, "Concurrent CPU cores to use.")
   flag.Parse()
   return *hostf, *portf, *mimetypef, *procsf
}
func Usage() {
   _, _ = fmt.Fprintf(os.Stderr, "Usage of %s:\n", os.Args[0])
   _, _ = fmt.Fprintf(os.Stderr, "  %s [options] \n", os.Args[0])
   _, _ = fmt.Fprintf(os.Stderr, "Options:\n")
   flag.PrintDefaults()
}
```

提供一个server端跟client端的代码

server.go

```
package main

import (
   "fmt"
   "net"
)
func main() {
   // 指定服务器通信协议ip地址和端口号
   listener, err := net.Listen("tcp", "127.0.0.1:10000") // 这里的listen并不是监听，而是创建一个用于监听的socket
   if err != nil {
      fmt.Println("net.Listen err:", err)
      return
   }
   defer listener.Close()

   fmt.Println("服务器等待客户端建立连接.......")
   // 阻塞监听客户端连接请求
   conn, err := listener.Accept() // 监听
   if err != nil {
      fmt.Println("accept err",err)
      return
   }

   buf := make([]byte,4096)
   n, err := conn.Read(buf) // n是字节数
   if err != nil{
      fmt.Println("conn Read err",err)
      return
   }
   fmt.Println("服务器收到的数据是：", string(buf[:n]))
   fmt.Println("服务器与客户端成功建立连接")
   defer conn.Close()
   // 读取客户端发送数据
}
```

client.go

```
package main

import (
   "fmt"
   "net"
)

func main() {
   conn, err := net.Dial("tcp", "127.0.0.1:8080")
   if err != nil {
      fmt.Println("net.Dail err", err)
      return
   }
   defer conn.Close()

   // 主动写数据给服务器
   conn.Write([]byte("来自客户端的连接..."))
}
```