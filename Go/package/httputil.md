# net/http/httputil

```
import "net/http/httputil"
```

httputil包提供了HTTP公用函数，是对net/http包的更常见函数的补充。

## Index

- [Variables](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#pkg-variables)
- func DumpRequest(req *http.Request, body bool) (dump \[\]byte, err error)
- func DumpRequestOut(req *http.Request, body bool) ([\]byte, error)
- func DumpResponse(resp *http.Response, body bool) (dump [\]byte, err error)
- [func NewChunkedReader(r io.Reader) io.Reader](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#NewChunkedReader)
- [func NewChunkedWriter(w io.Writer) io.WriteCloser](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#NewChunkedWriter)
- [type ClientConn](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#ClientConn)
- [func NewClientConn(c net.Conn, r *bufio.Reader) *ClientConn](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#NewClientConn)
- [func NewProxyClientConn(c net.Conn, r *bufio.Reader) *ClientConn](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#NewProxyClientConn)
- [func (cc *ClientConn) Pending() int](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#ClientConn.Pending)
- [func (cc *ClientConn) Write(req *http.Request) (err error)](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#ClientConn.Write)
- [func (cc *ClientConn) Read(req *http.Request) (resp *http.Response, err error)](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#ClientConn.Read)
- [func (cc *ClientConn) Do(req *http.Request) (resp *http.Response, err error)](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#ClientConn.Do)
- [func (cc *ClientConn) Hijack() (c net.Conn, r *bufio.Reader)](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#ClientConn.Hijack)
- [func (cc *ClientConn) Close() error](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#ClientConn.Close)
- [type ServerConn](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#ServerConn)
- [func NewServerConn(c net.Conn, r *bufio.Reader) *ServerConn](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#NewServerConn)
- [func (sc *ServerConn) Pending() int](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#ServerConn.Pending)
- [func (sc *ServerConn) Read() (req *http.Request, err error)](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#ServerConn.Read)
- [func (sc *ServerConn) Write(req *http.Request, resp *http.Response) error](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#ServerConn.Write)
- [func (sc *ServerConn) Hijack() (c net.Conn, r *bufio.Reader)](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#ServerConn.Hijack)
- [func (sc *ServerConn) Close() error](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#ServerConn.Close)
- [type ReverseProxy](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#ReverseProxy)
- [func NewSingleHostReverseProxy(target *url.URL) *ReverseProxy](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#NewSingleHostReverseProxy)
- [func (p *ReverseProxy) ServeHTTP(rw http.ResponseWriter, req *http.Request)](https://wizardforcel.gitbooks.io/golang-stdlib-ref/content/92.html#ReverseProxy.ServeHTTP)

## Variables

```
var (
    ErrPersistEOF = &http.ProtocolError{ErrorString: "persistent connection closed"}
    ErrClosed     = &http.ProtocolError{ErrorString: "connection closed by user"}
    ErrPipeline   = &http.ProtocolError{ErrorString: "pipeline error"}
)
var ErrLineTooLong = errors.New("header line too long")
```

## func [DumpRequest](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/dump.go?name=release#176)

```
func DumpRequest(req *http.Request, body bool) (dump []byte, err error)
```

DumpRequest返回req的和被服务端接收到时一样的有线表示，可选地包括请求的主体，用于debug。本函数在语义上是无操作的，但为了转储请求主体，他会读取主体的数据到内存中，并将req.Body修改为指向内存中的拷贝。req的字段的使用细节请参见http.Request的文档。

## func [DumpRequestOut](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/dump.go?name=release#61)

```
func DumpRequestOut(req *http.Request, body bool) ([]byte, error)
```

DumpRequestOut类似DumpRequest，但会包括标准http.Transport类型添加的头域，如User-Agent。

## func [DumpResponse](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/dump.go?name=release#251)

```
func DumpResponse(resp *http.Response, body bool) (dump []byte, err error)
```

DumpResponse类似DumpRequest，但转储的是一个回复。

## func [NewChunkedReader](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/httputil.go?name=release#17)

```
func NewChunkedReader(r io.Reader) io.Reader
```

NewChunkedReader返回一个io.Reader。返回值的Read方法会将从r读取的采用HTTP "chunked"传输编码的数据翻译之后返回。当读取到最后的零长chunk时，返回值的Read会返回io.EOF。

NewChunkedReader在正常的应用中是不需要的，http包在读取回复主体时会自动将"chunked"编码进行解码。

## func [NewChunkedWriter](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/httputil.go?name=release#30)

```
func NewChunkedWriter(w io.Writer) io.WriteCloser
```

NewChunkedWriter返回一个io.Writer。返回值的Write方法会将写入的数据编码为HTTP "chunked"传输编码格式后再写入w。其Close方法会将最后的零长chunk写入w，标注数据流的结尾。

正常的应用中不需要NewChunkedWriter，http包会在处理器未设置Content-Length头时主动进行chunked编码。在处理器内部使用本函数会导致双重chunked或者有Content-Length头的chunked，两个都是错误的。

## type [ClientConn](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/persist.go?name=release#229)

```
type ClientConn struct {
    // 内含隐藏或非导出字段
}
```

ClientConn类型会在尊重HTTP keepalive逻辑的前提下，在下层的连接上发送请求和接收回复的头域。ClientConn类型支持通过Hijack方法劫持下层连接，取回对下层连接的控制权，按照调用者的预期处理该连接。

ClientConn是旧的、低层次的。应用程序应使用net/http包的Client类型和Transport类型代替它。

### func [NewClientConn](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/persist.go?name=release#247)

```
func NewClientConn(c net.Conn, r *bufio.Reader) *ClientConn
```

NewClientConn返回一个对c进行读写的ClientConn。如果r不是nil，它是从c读取时使用的缓冲。

### func [NewProxyClientConn](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/persist.go?name=release#264)

```
func NewProxyClientConn(c net.Conn, r *bufio.Reader) *ClientConn
```

NewProxyClientConn类似NewClientConn，但使用Request.WriteProxy方法将请求写入c。

### func (*ClientConn) [Pending](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/persist.go?name=release#351)

```
func (cc *ClientConn) Pending() int
```

Pending返回该连接上已发送但还未接收到回复的请求的数量。

### func (*ClientConn) [Write](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/persist.go?name=release#298)

```
func (cc *ClientConn) Write(req *http.Request) (err error)
```

Write写入一个请求。如果该连接已经在HTTP keepalive逻辑上关闭了（表示该连接不会再发送新的请求）返回ErrPersistEOF。如果req.Close设置为真，keepalive连接会在这次请求后逻辑上关闭，并通知远端的服务器。ErrUnexpectedEOF表示远端关闭了下层的TCP连接，这一般被视为优雅的（正常的）关闭。

### func (*ClientConn) [Read](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/persist.go?name=release#361)

```
func (cc *ClientConn) Read(req *http.Request) (resp *http.Response, err error)
```

Read读取下一个回复。合法的回复可能会和ErrPersistEOF一起返回，这表示远端要求该请求是最后一个被服务的请求。Read可以和Write同时调用，但不能和另一个Read同时调用。

### func (*ClientConn) [Do](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/persist.go?name=release#423)

```
func (cc *ClientConn) Do(req *http.Request) (resp *http.Response, err error)
```

Do是一个便利方法，它会写入一个请求，并读取一个回复。（能不能保证二者对应？不知道）

### func (*ClientConn) [Hijack](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/persist.go?name=release#274)

```
func (cc *ClientConn) Hijack() (c net.Conn, r *bufio.Reader)
```

Hijack拆开ClientConn返回下层的连接和读取侧的缓冲，其中可能有部分剩余的数据。Hijack可以在调用者自身或者其Read方法发出keepalive逻辑的终止信号之前调用。调用者不应在Write或Read执行过程中调用Hijack。

### func (*ClientConn) [Close](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/persist.go?name=release#285)

```
func (cc *ClientConn) Close() error
```

Close调用Hijack，然后关闭下层的连接。

## type [ServerConn](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/persist.go?name=release#36)

```
type ServerConn struct {
    // 内含隐藏或非导出字段
}
```

ServerConn类型在下层的连接上接收请求和发送回复，直到HTTP keepalive逻辑结束。ServerConn类型支持通过Hijack方法劫持下层连接，取回对下层连接的控制权，按照调用者的预期处理该连接。ServerConn支持管道内套，例如，请求可以不和回复的发送同步的读取（但仍按照相同的顺序）。

ServerConn是旧的、低层次的。应用程序应使用net/http包的Server类型代替它。

### func [NewServerConn](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/persist.go?name=release#53)

```
func NewServerConn(c net.Conn, r *bufio.Reader) *ServerConn
```

NewServerConn返回一个新的从c读写的ServerConn。如果r补位nil，它将作为从c读取时的缓冲。

### func (*ServerConn) [Pending](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/persist.go?name=release#163)

```
func (sc *ServerConn) Pending() int
```

Pending返回该连接上已接收到但还未回复的请求的数量。

### func (*ServerConn) [Read](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/persist.go?name=release#87)

```
func (sc *ServerConn) Read() (req *http.Request, err error)
```

Read读取下一个请求。如果它优雅的决定不会再有更多的请求（例如，在HTTP/1.0连接的第一个请求之后，或者HTTP/1.1的一个具有" Connection:close "头的请求之后），会返回ErrPersistEOF。

### func (*ServerConn) [Write](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/persist.go?name=release#172)

```
func (sc *ServerConn) Write(req *http.Request, resp *http.Response) error
```

Write写入req的回复resp。如要优雅的关闭该连接，可以设置resp.Close字段为真。Write方法应该尽量执行（以回复尽量多的请求），直到Write本身返回错误，而不应考虑读取侧返回的任何错误。

### func (*ServerConn) [Hijack](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/persist.go?name=release#64)

```
func (sc *ServerConn) Hijack() (c net.Conn, r *bufio.Reader)
```

Hijack拆开ServerConn返回下层的连接和读取侧的缓冲，其中可能有部分剩余的数据。Hijack可以在调用者自身或者其Read方法发出keepalive逻辑的终止信号之前调用。调用者不应在Write或Read执行过程中调用Hijack。

### func (*ServerConn) [Close](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/persist.go?name=release#75)

```
func (sc *ServerConn) Close() error
```

Close调用Hijack，然后关闭下层的连接。

## ==type [ReverseProxy](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/reverseproxy.go?name=release#27)==

```
type ReverseProxy struct {
    // Director必须是将请求修改为新的请求的函数。
    // 修改后的请求会使用Transport发送，得到的回复会不经修改的返回给客户端。
    Director func(*http.Request)
    // Transport用于执行代理请求。
    // 如果本字段为nil，会使用http.DefaultTransport。
    Transport http.RoundTripper
    // FlushInterval指定拷贝回复的主体时将数据刷新给客户端的时间间隔。
    // 如果本字段为零值，不会进行周期的刷新。（拷贝完回复主体后再刷新）
    FlushInterval time.Duration
}
```

ReverseProxy是一个HTTP处理器，它接收一个请求，发送给另一个服务端，将回复转发给客户端。

### func [NewSingleHostReverseProxy](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/reverseproxy.go?name=release#61)

```
func NewSingleHostReverseProxy(target *url.URL) *ReverseProxy
```

NewSingleHostReverseProxy返回一个新的ReverseProxy。返回值会将请求的URL重写为target参数提供的协议、主机和基路径。如果target参数的Path字段为"/base"，接收到的请求的URL.Path为"/dir"，修改后的请求的URL.Path将会是"/base/dir"。

### func (*ReverseProxy) [ServeHTTP](http://code.google.com/p/go/source/browse/src/pkg/net/http/httputil/reverseproxy.go?name=release#97)

```
func (p *ReverseProxy) ServeHTTP(rw http.ResponseWriter, req *http.Request)
```
