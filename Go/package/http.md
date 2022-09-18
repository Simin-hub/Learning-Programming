# net/http

[参考地址](https://studygolang.com/pkgdoc)

[参考地址2](https://segmentfault.com/a/1190000021653550)

[参考地址3](https://darjun.github.io/2021/07/13/in-post/godailylib/nethttp/)

[示例](https://studygolang.com/articles/9467)

## HTTP 服务处理流程

基于HTTP构建的服务标准模型包括两个端，客户端(`Client`)和服务端(`Server`)。HTTP 请求从客户端发出，服务端接受到请求后进行处理然后将响应返回给客户端。所以http服务器的工作就在于如何接受来自客户端的请求，并向客户端返回响应。

典型的 HTTP 服务的处理流程如下图所示：

![HTTP处理流程](https://segmentfault.com/img/remote/1460000021653553)

服务器在接收到请求时，首先会进入路由(`router`)，也成为服务复用器（`Multiplexe`），**路由的工作在于请求找到对应的处理器(`handler`)**，处理器对接收到的请求进行相应处理后构建响应并返回给客户端。Go实现的`http server`同样遵循这样的处理流程。

```
import "net/http"
```

**http包提供了HTTP客户端和服务端的实现。**

Get、Head、Post和PostForm函数发出HTTP/ HTTPS请求。

```
resp, err := http.Get("http://example.com/")
...
resp, err := http.Post("http://example.com/upload", "image/jpeg", &buf)
...
resp, err := http.PostForm("http://example.com/form",
	url.Values{"key": {"Value"}, "id": {"123"}})
```

**程序在使用完回复后必须关闭回复的主体**。

```
resp, err := http.Get("http://example.com/")
if err != nil {
	// handle error
}
defer resp.Body.Close()
body, err := ioutil.ReadAll(resp.Body)
// ...
```

要**管理HTTP客户端的头域、重定向策略和其他设置，创建一个Client**：

```
client := &http.Client{
	CheckRedirect: redirectPolicyFunc,
}
resp, err := client.Get("http://example.com")
// ...
req, err := http.NewRequest("GET", "http://example.com", nil)
// ...
req.Header.Add("If-None-Match", `W/"wyzzy"`)
resp, err := client.Do(req)
// ...
```

要**管理代理、TLS配置、keep-alive、压缩和其他设置，创建一个Transport**：

```
tr := &http.Transport{
	TLSClientConfig:    &tls.Config{RootCAs: pool},
	DisableCompression: true,
}
client := &http.Client{Transport: tr}
resp, err := client.Get("https://example.com")
```

**Client和Transport类型都可以安全的被多个go程同时使用**。出于效率考虑，应该**一次建立、尽量重用**。

L**istenAndServe使用指定的监听地址和处理器启动一个HTTP服务端**。处理器参数通常是nil，这表示采用包变量DefaultServeMux作为处理器。Handle和HandleFunc函数可以向DefaultServeMux添加处理器。

```
http.Handle("/foo", fooHandler)
http.HandleFunc("/bar", func(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello, %q", html.EscapeString(r.URL.Path))
})
log.Fatal(http.ListenAndServe(":8080", nil))
```

要**管理服务端的行为，可以创建一个自定义的Server**：

```go
s := &http.Server{
	Addr:           ":8080",
	Handler:        myHandler,
	ReadTimeout:    10 * time.Second,
	WriteTimeout:   10 * time.Second,
	MaxHeaderBytes: 1 << 20,
}
log.Fatal(s.ListenAndServe())
```

```
Constants
Variables
type ProtocolError
func (err *ProtocolError) Error() string
func CanonicalHeaderKey(s string) string
func DetectContentType(data []byte) string
func ParseHTTPVersion(vers string) (major, minor int, ok bool)
func ParseTime(text string) (t time.Time, err error)
func StatusText(code int) string
type ConnState
func (c ConnState) String() string
type Header
func (h Header) Get(key string) string
func (h Header) Set(key, value string)
func (h Header) Add(key, value string)
func (h Header) Del(key string)
func (h Header) Write(w io.Writer) error
func (h Header) WriteSubset(w io.Writer, exclude map[string]bool) error
type Cookie
func (c *Cookie) String() string
type CookieJar
type Request
func NewRequest(method, urlStr string, body io.Reader) (*Request, error)
func ReadRequest(b *bufio.Reader) (req *Request, err error)
func (r *Request) ProtoAtLeast(major, minor int) bool
func (r *Request) UserAgent() string
func (r *Request) Referer() string
func (r *Request) AddCookie(c *Cookie)
func (r *Request) SetBasicAuth(username, password string)
func (r *Request) Write(w io.Writer) error
func (r *Request) WriteProxy(w io.Writer) error
func (r *Request) Cookies() []*Cookie
func (r *Request) Cookie(name string) (*Cookie, error)
func (r *Request) ParseForm() error
func (r *Request) ParseMultipartForm(maxMemory int64) error
func (r *Request) FormValue(key string) string
func (r *Request) PostFormValue(key string) string
func (r *Request) FormFile(key string) (multipart.File, *multipart.FileHeader, error)
func (r *Request) MultipartReader() (*multipart.Reader, error)
type Response
func ReadResponse(r *bufio.Reader, req *Request) (*Response, error)
func (r *Response) ProtoAtLeast(major, minor int) bool
func (r *Response) Cookies() []*Cookie
func (r *Response) Location() (*url.URL, error)
func (r *Response) Write(w io.Writer) error
type ResponseWriter
type Flusher
type CloseNotifier
type Hijacker
type RoundTripper
type Transport
func (t *Transport) RegisterProtocol(scheme string, rt RoundTripper)
func (t *Transport) RoundTrip(req *Request) (resp *Response, err error)
func (t *Transport) CloseIdleConnections()
func (t *Transport) CancelRequest(req *Request)
type Client
func (c *Client) Do(req *Request) (resp *Response, err error)
func (c *Client) Head(url string) (resp *Response, err error)
func (c *Client) Get(url string) (resp *Response, err error)
func (c *Client) Post(url string, bodyType string, body io.Reader) (resp *Response, err error)
func (c *Client) PostForm(url string, data url.Values) (resp *Response, err error)
type Handler
func NotFoundHandler() Handler
func RedirectHandler(url string, code int) Handler
func TimeoutHandler(h Handler, dt time.Duration, msg string) Handler
func StripPrefix(prefix string, h Handler) Handler
type HandlerFunc
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request)
type ServeMux
func NewServeMux() *ServeMux
func (mux *ServeMux) Handle(pattern string, handler Handler)
func (mux *ServeMux) HandleFunc(pattern string, handler func(ResponseWriter, *Request))
func (mux *ServeMux) Handler(r *Request) (h Handler, pattern string)
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request)
type Server
func (s *Server) SetKeepAlivesEnabled(v bool)
func (srv *Server) Serve(l net.Listener) error
func (srv *Server) ListenAndServe() error
func (srv *Server) ListenAndServeTLS(certFile, keyFile string) error
type File
type FileSystem
type Dir
func (d Dir) Open(name string) (File, error)
func NewFileTransport(fs FileSystem) RoundTripper
func FileServer(root FileSystem) Handler
func SetCookie(w ResponseWriter, cookie *Cookie)
func Redirect(w ResponseWriter, r *Request, urlStr string, code int)
func NotFound(w ResponseWriter, r *Request)
func Error(w ResponseWriter, error string, code int)
func ServeContent(w ResponseWriter, req *Request, name string, modtime time.Time, content io.ReadSeeker)
func ServeFile(w ResponseWriter, r *Request, name string)
func MaxBytesReader(w ResponseWriter, r io.ReadCloser, n int64) io.ReadCloser
func ProxyURL(fixedURL *url.URL) func(*Request) (*url.URL, error)
func ProxyFromEnvironment(req *Request) (*url.URL, error)
func Head(url string) (resp *Response, err error)
func Get(url string) (resp *Response, err error)
func Post(url string, bodyType string, body io.Reader) (resp *Response, err error)
func PostForm(url string, data url.Values) (resp *Response, err error)
func Handle(pattern string, handler Handler)
func HandleFunc(pattern string, handler func(ResponseWriter, *Request))
func Serve(l net.Listener, handler Handler) error
func ListenAndServe(addr string, handler Handler) error
func ListenAndServeTLS(addr string, certFile string, keyFile string, handler Handler) error
```

## Request

### type [Request](https://github.com/golang/go/blob/master/src/net/http/request.go?name=release#76)

```go
type Request struct {
    // Method指定HTTP方法（GET、POST、PUT等）。对客户端，""代表GET。
    Method string
    // URL在服务端表示被请求的URI，在客户端表示要访问的URL。
    //
    // 在服务端，URL字段是解析请求行的URI（保存在RequestURI字段）得到的，
    // 对大多数请求来说，除了Path和RawQuery之外的字段都是空字符串。
    // （参见RFC 2616, Section 5.1.2）
    //
    // 在客户端，URL的Host字段指定了要连接的服务器，
    // 而Request的Host字段（可选地）指定要发送的HTTP请求的Host头的值。
    URL *url.URL
    // 接收到的请求的协议版本。本包生产的Request总是使用HTTP/1.1
    Proto      string // "HTTP/1.0"
    ProtoMajor int    // 1
    ProtoMinor int    // 0
    // Header字段用来表示HTTP请求的头域。如果头域（多行键值对格式）为：
    //	accept-encoding: gzip, deflate
    //	Accept-Language: en-us
    //	Connection: keep-alive
    // 则：
    //	Header = map[string][]string{
    //		"Accept-Encoding": {"gzip, deflate"},
    //		"Accept-Language": {"en-us"},
    //		"Connection": {"keep-alive"},
    //	}
    // HTTP规定头域的键名（头名）是大小写敏感的，请求的解析器通过规范化头域的键名来实现这点。
    // 在客户端的请求，可能会被自动添加或重写Header中的特定的头，参见Request.Write方法。
    Header Header
    // Body是请求的主体。
    //
    // 在客户端，如果Body是nil表示该请求没有主体买入GET请求。
    // Client的Transport字段会负责调用Body的Close方法。
    //
    // 在服务端，Body字段总是非nil的；但在没有主体时，读取Body会立刻返回EOF。
    // Server会关闭请求的主体，ServeHTTP处理器不需要关闭Body字段。
    Body io.ReadCloser
    // ContentLength记录相关内容的长度。
    // 如果为-1，表示长度未知，如果>=0，表示可以从Body字段读取ContentLength字节数据。
    // 在客户端，如果Body非nil而该字段为0，表示不知道Body的长度。
    ContentLength int64
    // TransferEncoding按从最外到最里的顺序列出传输编码，空切片表示"identity"编码。
    // 本字段一般会被忽略。当发送或接受请求时，会自动添加或移除"chunked"传输编码。
    TransferEncoding []string
    // Close在服务端指定是否在回复请求后关闭连接，在客户端指定是否在发送请求后关闭连接。
    Close bool
    // 在服务端，Host指定URL会在其上寻找资源的主机。
    // 根据RFC 2616，该值可以是Host头的值，或者URL自身提供的主机名。
    // Host的格式可以是"host:port"。
    //
    // 在客户端，请求的Host字段（可选地）用来重写请求的Host头。
    // 如过该字段为""，Request.Write方法会使用URL字段的Host。
    Host string
    // Form是解析好的表单数据，包括URL字段的query参数和POST或PUT的表单数据。
    // 本字段只有在调用ParseForm后才有效。在客户端，会忽略请求中的本字段而使用Body替代。
    Form url.Values
    // PostForm是解析好的POST或PUT的表单数据。
    // 本字段只有在调用ParseForm后才有效。在客户端，会忽略请求中的本字段而使用Body替代。
    PostForm url.Values
    // MultipartForm是解析好的多部件表单，包括上传的文件。
    // 本字段只有在调用ParseMultipartForm后才有效。
    // 在客户端，会忽略请求中的本字段而使用Body替代。
    MultipartForm *multipart.Form
    // Trailer指定了会在请求主体之后发送的额外的头域。
    //
    // 在服务端，Trailer字段必须初始化为只有trailer键，所有键都对应nil值。
    // （客户端会声明哪些trailer会发送）
    // 在处理器从Body读取时，不能使用本字段。
    // 在从Body的读取返回EOF后，Trailer字段会被更新完毕并包含非nil的值。
    // （如果客户端发送了这些键值对），此时才可以访问本字段。
    //
    // 在客户端，Trail必须初始化为一个包含将要发送的键值对的映射。（值可以是nil或其终值）
    // ContentLength字段必须是0或-1，以启用"chunked"传输编码发送请求。
    // 在开始发送请求后，Trailer可以在读取请求主体期间被修改，
    // 一旦请求主体返回EOF，调用者就不可再修改Trailer。
    //
    // 很少有HTTP客户端、服务端或代理支持HTTP trailer。
    Trailer Header
    // RemoteAddr允许HTTP服务器和其他软件记录该请求的来源地址，一般用于日志。
    // 本字段不是ReadRequest函数填写的，也没有定义格式。
    // 本包的HTTP服务器会在调用处理器之前设置RemoteAddr为"IP:port"格式的地址。
    // 客户端会忽略请求中的RemoteAddr字段。
    RemoteAddr string
    // RequestURI是被客户端发送到服务端的请求的请求行中未修改的请求URI
    // （参见RFC 2616, Section 5.1）
    // 一般应使用URI字段，在客户端设置请求的本字段会导致错误。
    RequestURI string
    // TLS字段允许HTTP服务器和其他软件记录接收到该请求的TLS连接的信息
    // 本字段不是ReadRequest函数填写的。
    // 对启用了TLS的连接，本包的HTTP服务器会在调用处理器之前设置TLS字段，否则将设TLS为nil。
    // 客户端会忽略请求中的TLS字段。
    TLS *tls.ConnectionState
}
```

Request类型代表一个服务端接受到的或者客户端发送出去的HTTP请求。

Request各字段的意义和用途在服务端和客户端是不同的。除了字段本身上方文档，还可参见Request.Write方法和RoundTripper接口的文档。

## Response

### type [Response](https://github.com/golang/go/blob/master/src/net/http/response.go?name=release#29)

```
type Response struct {
    Status     string // 例如"200 OK"
    StatusCode int    // 例如200
    Proto      string // 例如"HTTP/1.0"
    ProtoMajor int    // 例如1
    ProtoMinor int    // 例如0
    // Header保管头域的键值对。
    // 如果回复中有多个头的键相同，Header中保存为该键对应用逗号分隔串联起来的这些头的值
    // （参见RFC 2616 Section 4.2）
    // 被本结构体中的其他字段复制保管的头（如ContentLength）会从Header中删掉。
    //
    // Header中的键都是规范化的，参见CanonicalHeaderKey函数
    Header Header
    // Body代表回复的主体。
    // Client类型和Transport类型会保证Body字段总是非nil的，即使回复没有主体或主体长度为0。
    // 关闭主体是调用者的责任。
    // 如果服务端采用"chunked"传输编码发送的回复，Body字段会自动进行解码。
    Body io.ReadCloser
    // ContentLength记录相关内容的长度。
    // 其值为-1表示长度未知（采用chunked传输编码）
    // 除非对应的Request.Method是"HEAD"，其值>=0表示可以从Body读取的字节数
    ContentLength int64
    // TransferEncoding按从最外到最里的顺序列出传输编码，空切片表示"identity"编码。
    TransferEncoding []string
    // Close记录头域是否指定应在读取完主体后关闭连接。（即Connection头）
    // 该值是给客户端的建议，Response.Write方法的ReadResponse函数都不会关闭连接。
    Close bool
    // Trailer字段保存和头域相同格式的trailer键值对，和Header字段相同类型
    Trailer Header
    // Request是用来获取此回复的请求
    // Request的Body字段是nil（因为已经被用掉了）
    // 这个字段是被Client类型发出请求并获得回复后填充的
    Request *Request
    // TLS包含接收到该回复的TLS连接的信息。 对未加密的回复，本字段为nil。
    // 返回的指针是被（同一TLS连接接收到的）回复共享的，不应被修改。
    TLS *tls.ConnectionState
}
```

Response代表一个HTTP请求的回复。

## Flusher

### type [Flusher](https://github.com/golang/go/blob/master/src/net/http/server.go?name=release#79)

```
type Flusher interface {
    // Flush将缓冲中的所有数据发送到客户端
    Flush()
}
```

HTTP处理器ResponseWriter接口参数的下层如果实现了Flusher接口，可以让HTTP处理器将缓冲中的数据发送到客户端。

注意：即使ResponseWriter接口的下层支持Flush方法，如果客户端是通过HTTP代理连接的，缓冲中的数据也可能直到回复完毕才被传输到客户端。

## Transport

### type [Transport](https://github.com/golang/go/blob/master/src/net/http/transport.go?name=release#49)

```
type Transport struct {
    // Proxy指定一个对给定请求返回代理的函数。
    // 如果该函数返回了非nil的错误值，请求的执行就会中断并返回该错误。
    // 如果Proxy为nil或返回nil的*URL置，将不使用代理。
    Proxy func(*Request) (*url.URL, error)
    // Dial指定创建TCP连接的拨号函数。如果Dial为nil，会使用net.Dial。
    Dial func(network, addr string) (net.Conn, error)
    // TLSClientConfig指定用于tls.Client的TLS配置信息。
    // 如果该字段为nil，会使用默认的配置信息。
    TLSClientConfig *tls.Config
    // TLSHandshakeTimeout指定等待TLS握手完成的最长时间。零值表示不设置超时。
    TLSHandshakeTimeout time.Duration
    // 如果DisableKeepAlives为真，会禁止不同HTTP请求之间TCP连接的重用。
    DisableKeepAlives bool
    // 如果DisableCompression为真，会禁止Transport在请求中没有Accept-Encoding头时，
    // 主动添加"Accept-Encoding: gzip"头，以获取压缩数据。
    // 如果Transport自己请求gzip并得到了压缩后的回复，它会主动解压缩回复的主体。
    // 但如果用户显式的请求gzip压缩数据，Transport是不会主动解压缩的。
    DisableCompression bool
    // 如果MaxIdleConnsPerHost!=0，会控制每个主机下的最大闲置连接。
    // 如果MaxIdleConnsPerHost==0，会使用DefaultMaxIdleConnsPerHost。
    MaxIdleConnsPerHost int
    // ResponseHeaderTimeout指定在发送完请求（包括其可能的主体）之后，
    // 等待接收服务端的回复的头域的最大时间。零值表示不设置超时。
    // 该时间不包括获取回复主体的时间。
    ResponseHeaderTimeout time.Duration
    // 内含隐藏或非导出字段
}
```

Transport类型实现了RoundTripper接口，支持http、https和http/https代理。Transport类型可以缓存连接以在未来重用。

```
var DefaultTransport RoundTripper = &Transport{
    Proxy: ProxyFromEnvironment,
    Dial: (&net.Dialer{
        Timeout:   30 * time.Second,
        KeepAlive: 30 * time.Second,
    }).Dial,
    TLSHandshakeTimeout: 10 * time.Second,
}
```

DefaultTransport是被包变量DefaultClient使用的默认RoundTripper接口。它会根据需要创建网络连接，并缓存以便在之后的请求中重用这些连接。它使用环境变量$HTTP_PROXY和$NO_PROXY（或$http_proxy和$no_proxy）指定的HTTP代理。

## Handler

### type [Handler](https://github.com/golang/go/blob/master/src/net/http/server.go?name=release#45)

```
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

实现了Handler接口的对象可以注册到HTTP服务端，为特定的路径及其子树提供服务。

ServeHTTP应该将回复的头域和数据写入ResponseWriter接口然后返回。返回标志着该请求已经结束，HTTP服务端可以转移向该连接上的下一个请求。

### type [HandlerFunc](https://github.com/golang/go/blob/master/src/net/http/server.go?name=release#1231)

```
type HandlerFunc func(ResponseWriter, *Request)
```

HandlerFunc type是一个适配器，通过类型转换让我们可以将普通的函数作为HTTP处理器使用。如果f是一个具有适当签名的函数，HandlerFunc(f)通过调用f实现了Handler接口。

#### func (HandlerFunc) [ServeHTTP](https://github.com/golang/go/blob/master/src/net/http/server.go?name=release#1234)

```
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request)
```

ServeHTTP方法会调用f(w, r)

## Server

### type [Server](https://github.com/golang/go/blob/master/src/net/http/server.go?name=release#1581)

```
type Server struct {
    Addr           string        // 监听的TCP地址，如果为空字符串会使用":http"
    Handler        Handler       // 调用的处理器，如为nil会调用http.DefaultServeMux
    ReadTimeout    time.Duration // 请求的读取操作在超时前的最大持续时间
    WriteTimeout   time.Duration // 回复的写入操作在超时前的最大持续时间
    MaxHeaderBytes int           // 请求的头域最大长度，如为0则用DefaultMaxHeaderBytes
    TLSConfig      *tls.Config   // 可选的TLS配置，用于ListenAndServeTLS方法
    // TLSNextProto（可选地）指定一个函数来在一个NPN型协议升级出现时接管TLS连接的所有权。
    // 映射的键为商谈的协议名；映射的值为函数，该函数的Handler参数应处理HTTP请求，
    // 并且初始化Handler.ServeHTTP的*Request参数的TLS和RemoteAddr字段（如果未设置）。
    // 连接在函数返回时会自动关闭。
    TLSNextProto map[string]func(*Server, *tls.Conn, Handler)
    // ConnState字段指定一个可选的回调函数，该函数会在一个与客户端的连接改变状态时被调用。
    // 参见ConnState类型和相关常数获取细节。
    ConnState func(net.Conn, ConnState)
    // ErrorLog指定一个可选的日志记录器，用于记录接收连接时的错误和处理器不正常的行为。
    // 如果本字段为nil，日志会通过log包的标准日志记录器写入os.Stderr。
    ErrorLog *log.Logger
    // 内含隐藏或非导出字段
}
```

Server类型定义了运行HTTP服务端的参数。Server的零值是合法的配置。

#### func (*Server) [Serve](https://github.com/golang/go/blob/master/src/net/http/server.go?name=release#1694)

```
func (srv *Server) Serve(l net.Listener) error
```

Serve会接手监听器l收到的每一个连接，并为每一个连接创建一个新的服务go程。该go程会读取请求，然后调用srv.Handler回复请求。

#### func (*Server) [ListenAndServe](https://github.com/golang/go/blob/master/src/net/http/server.go?name=release#1679)

```
func (srv *Server) ListenAndServe() error
```

ListenAndServe监听srv.Addr指定的TCP地址，并且会调用Serve方法接收到的连接。如果srv.Addr为空字符串，会使用":http"。

#### func (*Server) [ListenAndServeTLS](https://github.com/golang/go/blob/master/src/net/http/server.go?name=release#1823)

```
func (srv *Server) ListenAndServeTLS(certFile, keyFile string) error
```

ListenAndServeTLS监听srv.Addr确定的TCP地址，并且会调用Serve方法处理接收到的连接。必须提供证书文件和对应的私钥文件。如果证书是由权威机构签发的，certFile参数必须是顺序串联的服务端证书和CA证书。如果srv.Addr为空字符串，会使用":https"。

