# gorilla Web 

Gorilla is a web toolkit for the [Go programming language](http://golang.org/). Currently these packages are available:

- [gorilla/mux](https://www.gorillatoolkit.org/pkg/mux) is a powerful URL router and dispatcher.
- [gorilla/reverse](https://www.gorillatoolkit.org/pkg/reverse) produces reversible regular expressions for regexp-based muxes.
- [gorilla/rpc](https://www.gorillatoolkit.org/pkg/rpc) implements RPC over HTTP with codec for [JSON-RPC](https://www.gorillatoolkit.org/pkg/rpc/json).
- [gorilla/schema](https://www.gorillatoolkit.org/pkg/schema) converts form values to a struct.
- [gorilla/securecookie](https://www.gorillatoolkit.org/pkg/securecookie) encodes and decodes authenticated and optionally encrypted cookie values.
- [gorilla/sessions](https://www.gorillatoolkit.org/pkg/sessions) saves cookie and filesystem sessions and allows custom session backends.
- [gorilla/websocket](https://www.gorillatoolkit.org/pkg/websocket) implements the WebSocket protocol defined in [RFC 6455](http://tools.ietf.org/html/rfc6455).
- [gorilla/csrf](https://www.gorillatoolkit.org/pkg/csrf) provides Cross Site Request Forgery (CSRF) prevention middleware.
- [gorilla/handlers](https://www.gorillatoolkit.org/pkg/handlers) is a collection of useful handlers for Go's net/http package.

## gorilla/mux

[GitHub](https://github.com/gorilla/mux)

[参考地址](https://segmentfault.com/a/1190000040371581)

[`gorilla/mux`](https://link.segmentfault.com/?enc=xq%2B2vdd0aO6vAaDB5Q%2FF8Q%3D%3D.Lmstk3%2BToiOCsuXx9rTpS%2F7NbmxoZiiVhqpR5i%2FZuOI%3D)是 gorilla Web 开发工具包中的路由管理库。gorilla Web 开发包是 Go 语言中辅助开发 Web 服务器的工具包。它包括 Web 服务器开发的各个方面，有表单数据处理包[`gorilla/schema`](https://link.segmentfault.com/?enc=SR86s307SoOGm%2FMATBsaZQ%3D%3D.XKcE5viLivSRWput%2Fv9lIbNXK1JoFVcWr%2FXg8Q6ce%2BlltpYo%2FfYTyhWGVEvSANwN)，有 `websocket` 通信包[`gorilla/websocket`](https://link.segmentfault.com/?enc=87CZccV27eoLsmZFVVH7QA%3D%3D.1rXZDgWSxZBpIjK0ee3bjErRot4xhFb%2Fkejcy9DFsOzkKiQgd1wxiS9WyS7V51Vt)，有各种中间件的包[`gorilla/handlers`](https://link.segmentfault.com/?enc=85gGRi61HpNe2RtcizwHGA%3D%3D.MlmrOxK04GOqEi4nHI3dniaTRa1TT0HRskweUfJ9nLODu4HAJ%2FjaRMvMTc1%2FVaj8)，有 session 管理包[`gorilla/sessions`](https://link.segmentfault.com/?enc=bCe24N5JlaP56tEu7Uk2%2Bw%3D%3D.RXyr07eh%2Fnd243A9va1Kn76xM7NdHc3VfZ6fmyanXshnoXQJ2HNZsd3C4o68Qvn5)，有安全的 cookie 包[`gorilla/securecookie`](https://link.segmentfault.com/?enc=3UKGd28OxpcBsB936Wy88g%3D%3D.v%2FymIlp4RtUOT6jQObRBE8tj6SXWDUTUxUyv3sjqxH5rXDhuOpm2%2FihXlbybkEvs)。本文先介绍`gorilla/mux`（下文简称`mux`）

`mux`有以下优势：

- 实现了标准的`http.Handler`接口，所以可以与`net/http`标准库结合使用，非常轻量；
- 可以根据请求的主机名、路径、路径前缀、协议、HTTP 首部、查询字符串和 HTTP 方法匹配处理器，还可以自定义匹配逻辑；
- 可以在主机名、路径和请求参数中使用变量，还可以为之指定一个正则表达式；
- 可以传入参数给指定的处理器让其构造出完整的 URL；
- 支持路由分组，方便管理和维护。

### 快速使用

本文代码使用 Go Modules。

创建目录并初始化：

```shell
$ mkdir -p gorilla/mux && cd gorilla/mux
$ go mod init github.com/darjun/go-daily-lib/gorilla/mux
```

安装`gorilla/mux`库：

```vim
$ go get -u github.com/gorilla/gorilla/mux
```

我现在身边有几本 Go 语言的经典著作：

![img](https://segmentfault.com/img/remote/1460000040371583)

下面我们编写一个管理图书信息的 Web 服务。图书由 ISBN 唯一标识，ISBN 意为国际标准图书编号（International Standard Book Number）。

首先定义图书的结构：

```go
type Book struct {
  ISBN        string   `json:"isbn"`
  Name        string   `json:"name"`
  Authors     []string `json:"authors"`
  Press       string   `json:"press"`
  PublishedAt string   `json:"published_at"`
}

var (
  mapBooks map[string]*Book
  slcBooks []*Book
)
```

定义`init()`函数，从文件中加载数据：

```go
func init() {
  mapBooks = make(map[string]*Book)
  slcBooks = make([]*Book, 0, 1)

  data, err := ioutil.ReadFile("../data/books.json")
  if err != nil {
    log.Fatalf("failed to read book.json:%v", err)
  }

  err = json.Unmarshal(data, &slcBooks)
  if err != nil {
    log.Fatalf("failed to unmarshal books:%v", err)
  }

  for _, book := range slcBooks {
    mapBooks[book.ISBN] = book
  }
}
```

然后是两个处理函数，分别用于返回整个列表和某一本具体的图书：

```go
func BooksHandler(w http.ResponseWriter, r *http.Request) {
  enc := json.NewEncoder(w)
  enc.Encode(slcBooks)
}

func BookHandler(w http.ResponseWriter, r *http.Request) {
  book, ok := mapBooks[mux.Vars(r)["isbn"]]
  if !ok {
    http.NotFound(w, r)
    return
  }

  enc := json.NewEncoder(w)
  enc.Encode(book)
}
```

注册处理器：

```go
func main() {
  r := mux.NewRouter()
  r.HandleFunc("/", BooksHandler)
  r.HandleFunc("/books/{isbn}", BookHandler)
  http.Handle("/", r)
  log.Fatal(http.ListenAndServe(":8080", nil))
}
```

`mux`的使用与`net/http`非常类似。首先调用`mux.NewRouter()`创建一个类型为`*mux.Router`的路由对象，该路由对象注册处理器的方式与标准库的`*http.ServeMux`完全相同，即调用`HandleFunc()`方法注册类型为`func(http.ResponseWriter, *http.Request)`的处理函数，调用`Handle()`方法注册实现了`http.Handler`接口的处理器对象。上面注册了两个处理函数，一个是显示图书信息列表，一个显示具体某本书的信息。

注意到路径`/books/{isbn}`使用了变量，在`{}`中间指定变量名，它可以匹配路径中的特定部分。在处理函数中通过`mux.Vars(r)`获取请求`r`的路由变量，返回`map[string]string`，后续可以用变量名访问。如上面的`BookHandler`中对变量`isbn`的访问。

由于`*mux.Router`也实现了`http.Handler`接口，所以可以直接将它作为`http.Handle("/", r)`的处理器对象参数注册。这里注册的是根路径`/`，相当于把所有请求的处理都托管给了`*mux.Router`。

最后还是`http.ListenAndServe(":8080", nil)`开启一个 Web 服务器，等待接收请求。

运行，在浏览器中键入`localhost:8080`，显示书籍列表：

![img](https://segmentfault.com/img/remote/1460000040371584)

键入`localhost:8080/books/978-7-111-55842-2`，显示图书《Go 程序设计语言》的详细信息：

![img](https://segmentfault.com/img/remote/1460000040371585)

从上面的使用过程中可以看出，`mux`库非常轻量，能很好的与标准库`net/http`结合使用。

我们还可以使用正则表达式限定变量的模式。ISBN 有固定的模式，现在使用的模式大概是这样：`978-7-111-55842-2`（这就是《Go 程序设计语言》一书的 ISBN），即 3个数字-1个数字-3个数字-5个数字-1个数字，用正则表达式表示为`\d{3}-\d-\d{3}-\d{5}-\d`。在变量名后添加一个`:`分隔变量和正则表达式：

```go
r.HandleFunc("/books/{isbn:\\d{3}-\\d-\\d{3}-\\d{5}-\\d}", BookHandler)
```

### 灵活的匹配方式

`mux`提供了丰富的匹配请求的方式。相比之下，`net/http`只能指定具体的路径，稍显笨拙。

我们可以指定路由的域名或子域名：

```go
r.Host("github.io")
r.Host("{subdomain:[a-zA-Z0-9]+}.github.io")
```

上面的路由只接受域名`github.io`或其子域名的请求，例如我的博客地址`darjun.github.io`就是它的一个子域名。指定域名时可以使用正则表达式，上面第二行代码限制子域名的第一部分必须是若干个字母或数字。

指定路径前缀：

```go
// 只处理路径前缀为`/books/`的请求
r.PathPrefix("/books/")
```

指定请求的方法：

```go
// 只处理 GET/POST 请求
r.Methods("GET", "POST")
```

使用的协议（`HTTP/HTTPS`）：

```go
// 只处理 https 的请求
r.Schemes("https")
```

首部：

```go
// 只处理首部 X-Requested-With 的值为 XMLHTTPRequest 的请求
r.Headers("X-Requested-With", "XMLHTTPRequest")
```

查询参数（即 URL 中`?`后的部分）：

```go
// 只处理查询参数包含key=value的请求
r.Queries("key", "value")
```

最后我们可以组合这些条件：

```go
r.HandleFunc("/", HomeHandler)
 .Host("bookstore.com")
 .Methods("GET")
 .Schemes("http")
```

除此之外，`mux`还允许自定义匹配器。自定义的匹配器就是一个类型为`func(r *http.Request, rm *RouteMatch) bool`的函数，根据请求`r`中的信息判断是否能否匹配成功。`http.Request`结构中包含了非常多的信息：HTTP 方法、HTTP 版本号、URL、首部等。例如，如果我们要求只处理 HTTP/1.1 的请求可以这么写：

```go
r.MatchrFunc(func(r *http.Request, rm *RouteMatch) bool {
  return r.ProtoMajor == 1 && r.ProtoMinor == 1
})
```

**需要注意的是，`mux`会根据路由注册的顺序依次匹配。所以，通常是将特殊的路由放在前面，一般的路由放在后面**。如果反过来了，特殊的路由就不会被匹配到了：

```go
r.HandleFunc("/specific", specificHandler)
r.PathPrefix("/").Handler(catchAllHandler)
```

### 子路由

有时候对路由进行分组管理，能让程序模块更清晰，更易于维护。现在网站扩展业务，加入了电影相关信息。我们可以定义两个子路由分别管理：

```go
r := mux.NewRouter()
bs := r.PathPrefix("/books").Subrouter()
bs.HandleFunc("/", BooksHandler)
bs.HandleFunc("/{isbn}", BookHandler)

ms := r.PathPrefix("/movies").Subrouter()
ms.HandleFunc("/", MoviesHandler)
ms.HandleFunc("/{imdb}", MovieHandler)
```

子路由一般通过路径前缀来限定，`r.PathPrefix()`会返回一个`*mux.Route`对象，调用它的`Subrouter()`方法创建一个子路由对象`*mux.Router`，然后通过该对象的`HandleFunc/Handle`方法注册处理函数。

电影没有类似图书的 ISBN 国际统一标准，只有一个民间“准标准”：IMDB。我们采用豆瓣电影中的信息：

![img](https://segmentfault.com/img/remote/1460000040371586)

定义电影的结构：

```go
type Movie struct {
  IMDB        string `json:"imdb"`
  Name        string `json:"name"`
  PublishedAt string `json:"published_at"`
  Duration    uint32 `json:"duration"`
  Lang        string `json:"lang"`
}
```

加载：

```go
var (
  mapMovies map[string]*Movie
  slcMovies []*Movie
)

func init() {
  mapMovies = make(map[string]*Movie)
  slcMovies = make([]*Movie, 0, 1)

  data,  := ioutil.ReadFile("../../data/movies.json")
  json.Unmarshal(data, &slcMovies)
  for _, movie := range slcMovies {
    mapMovies[movie.IMDB] = movie
  }
}
```

使用子路由的方式，还可以将各个部分的路由分散到各自的模块去加载，在文件`book.go`中定义一个`InitBooksRouter()`方法负责注册图书相关的路由：

```go
func InitBooksRouter(r *mux.Router) {
  bs := r.PathPrefix("/books").Subrouter()
  bs.HandleFunc("/", BooksHandler)
  bs.HandleFunc("/{isbn}", BookHandler)
}
```

在文件`movie.go`中定义一个`InitMoviesRouter()`方法负责注册电影相关的路由：

```go
func InitMoviesRouter(r *mux.Router) {
  ms := r.PathPrefix("/movies").Subrouter()
  ms.HandleFunc("/", MoviesHandler)
  ms.HandleFunc("/{imdb}", MovieHandler)
}
```

在`main.go`的主函数中：

```go
func main() {
  r := mux.NewRouter()
  InitBooksRouter(r)
  InitMoviesRouter(r)

  http.Handle("/", r)
  log.Fatal(http.ListenAndServe(":8080", nil))
}
```

需要注意的是，子路由匹配是需要包含路径前缀的，也就是说`/books/`才能匹配`BooksHandler`。

### 构造路由 URL

我们可以为一个路由起一个名字，例如：

```go
r.HandleFunc("/books/{isbn}", BookHandler).Name("book")
```

上面的路由中有参数，我们可以传入参数值来构造一个完整的路径：

```go
fmt.Println(r.Get("book").URL("isbn", "978-7-111-55842-2"))
// /books/978-7-111-55842-2 <nil>
```

返回的是一个`*url.URL`对象，其路径部分为`/books/978-7-111-55842-2`。这同样适用于主机名和查询参数：

```go
r := mux.Router()
r.Host("{name}.github.io").
 Path("/books/{isbn}").
 HandlerFunc(BookHandler).
 Name("book")

url, err := r.Get("book").URL("name", "darjun", "isbn", "978-7-111-55842-2")
```

路径中所有的参数都需要指定，并且值需要满足指定的正则表达式（如果有的话）。运行输出：

```go
$ go run main.go
http://darjun.github.io/books/978-7-111-55842-2
```

可以调用`URLHost()`只生成主机名部分，`URLPath()`只生成路径部分。

### 中间件

`mux`定义了中间件类型`MiddlewareFunc`：

```go
type MiddlewareFunc func(http.Handler) http.Handler
```

所有满足该类型的函数都可以作为`mux`的中间件使用，通过调用路由对象`*mux.Router`的`Use()`方法应用中间件。如果看过我上一篇文章[《Go 每日一库之 net/http（基础和中间件）》](https://link.segmentfault.com/?enc=o1Qt9fEwh0Zdaog4u52A7w%3D%3D.c2NAHcfMLg0s52vQ3ZlEekttzXvFMud27Bbp5gn0bQkdeIoKDxG4SeUAQiVWP9G7hocKX%2F3DFif28uNsrUij5Q%3D%3D)应该对这种中间件不陌生了。编写中间件一般会将原处理器传入，中间件中会手动调用原处理函数，然后在前后增加通用处理逻辑：

```go
func loggingMiddleware(next http.Handler) http.Handler {
  return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    log.Println(r.RequestURI)
    next.ServeHTTP(w, r)
  })
}
```

我在上篇文章中写的 3 个中间件可以直接使用，这就是兼容`net/http`的好处：

```go
func main() {
  logger = log.New(os.Stdout, "[goweb]", log.Lshortfile|log.LstdFlags)

  r := mux.NewRouter()
  // 直接使用上一篇文章中定义的中间件
  r.Use(PanicRecover, WithLogger, Metric)
  InitBooksRouter(r)
  InitMoviesRouter(r)

  http.Handle("/", r)
  log.Fatal(http.ListenAndServe(":8080", nil))
}
```

如果不手动调用原处理函数，那么原处理函数就不会执行，这可以用来在校验不通过时直接返回错误。例如，网站需要登录才能访问，而 HTTP 是一个无状态的协议。所以发明了 Cookie 机制用于在客户端和服务器之间记录一些信息。

我们在登录成功之后生成一个键为`token`的 Cookie 表示已登录成功，我们可以编写一个中间件来出来这块逻辑，如果 Cookie 不存在或者非法，则重定向到登录界面：

```go
func login(w http.ResponseWriter, r *http.Request) {
  ptTemplate.ExecuteTemplate(w, "login.tpl", nil)
}

func doLogin(w http.ResponseWriter, r *http.Request) {
  r.ParseForm()
  username := r.Form.Get("username")
  password := r.Form.Get("password")
  if username != "darjun" || password != "handsome" {
    http.Redirect(w, r, "/login", http.StatusFound)
    return
  }

  token := fmt.Sprintf("username=%s&password=%s", username, password)
  data := base64.StdEncoding.EncodeToString([]byte(token))
  http.SetCookie(w, &http.Cookie{
    Name:     "token",
    Value:    data,
    Path:     "/",
    HttpOnly: true,
    Expires:  time.Now().Add(24 * time.Hour),
  })
  http.Redirect(w, r, "/", http.StatusFound)
}
```

上面为了记录登录状态，我将登录的用户名和密码组合成`username=xxx&password=xxx`形式的字符串，对这个字符串进行`base64`编码，然后设置到 Cookie 中。Cookie 有效期为 24 小时。同时为了安全只允许 HTTP 访问此 Cookie（JS 脚本不可访问）。**当然这种方式安全性很低，这里只是为了演示**。登录成功之后重定向到`/`。

为了展示登录界面，我创建了几个`template`模板文件，使用`html/template`解析：

登录展示页面：

```go
// login.tpl
<form action="/login" method="post">
  <label>Username:</label>
  <input name="username"><br>
  <label>Password:</label>
  <input name="password" type="password"><br>
  <button type="submit">登录</button>
</form>
```

主页面

```go
<ul>
  <li><a href="/books/">图书</a></li>
  <li><a href="/movies/">电影</a></li>
</ul>
```

同时也创建了图书和电影的页面：

```go
// movies.tpl
<ol>
  {{ range . }}
  <li>
    <p>书名: <a href="/movies/{{ .IMDB }}">{{ .Name }}</a></p>
    <p>上映日期: {{ .PublishedAt }}</p>
    <p>时长: {{ .Duration }}分</p>
    <p>语言: {{ .Lang }}</p>
  </li>
  {{ end }}
</ol>
// movie.tpl
<p>IMDB: {{ .IMDB }}</p>
<p>电影名: {{ .Name }}</p>
<p>上映日期: {{ .PublishedAt }}</p>
<p>时长: {{ .Duration }}分</p>
<p>语言: {{ .Lang }}</p>
```

图书页面类似。接下来要解析模板：

```go
var (
  ptTemplate *template.Template
)

func init() {
  var err error
  ptTemplate, err = template.New("").ParseGlob("./tpls/*.tpl")
  if err != nil {
    log.Fatalf("load templates failed:%v", err)
  }
}
```

访问对应的页面逻辑：

```go
func MoviesHandler(w http.ResponseWriter, r *http.Request) {
  ptTemplate.ExecuteTemplate(w, "movies.tpl", slcMovies)
}

func MovieHandler(w http.ResponseWriter, r *http.Request) {
  movie, ok := mapMovies[mux.Vars(r)["imdb"]]
  if !ok {
    http.NotFound(w, r)
    return
  }

  ptTemplate.ExecuteTemplate(w, "movie.tpl", movie)
}
```

执行对应的模板，传入电影列表或某个具体的电影信息即可。现在页面没有限制访问，我们来编写一个中间件限制只有登录用户才能访问，未登录用户访问时跳转到登录界面：

```go
func authenticateMiddleware(next http.Handler) http.Handler {
  return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    cookie, err := r.Cookie("token")
    if err != nil {
      // no cookie
      http.Redirect(w, r, "/login", http.StatusFound)
      return
    }

    data, _ := base64.StdEncoding.DecodeString(cookie.Value)
    values, _ := url.ParseQuery(string(data))
    if values.Get("username") != "dj" && values.Get("password") != "handsome" {
      // failed
      http.Redirect(w, r, "/login", http.StatusFound)
      return
    }

    next.ServeHTTP(w, r)
  })
}
```

再次强调，这里只是为了演示，这种验证方式安全性很低。

然后，我们让`books`和`movies`子路由应用中间件`authenticateMiddleware`（需要登录验证），而`login`子路由不用：

```go
func InitBooksRouter(r *mux.Router) {
  bs := r.PathPrefix("/books").Subrouter()
  // 这里
  bs.Use(authenticateMiddleware)
  bs.HandleFunc("/", BooksHandler)
  bs.HandleFunc("/{isbn}", BookHandler)
}

func InitMoviesRouter(r *mux.Router) {
  ms := r.PathPrefix("/movies").Subrouter()
  // 这里
  ms.Use(authenticateMiddleware)
  ms.HandleFunc("/", MoviesHandler)
  ms.HandleFunc("/{id}", MovieHandler)
}

func InitLoginRouter(r *mux.Router) {
  ls := r.PathPrefix("/login").Subrouter()
  ls.Methods("GET").HandlerFunc(login)
  ls.Methods("POST").HandlerFunc(doLogin)
}
```

运行程序（注意多文件程序运行方式）：

```go
$ go run .
```

访问`localhost:8080/movies/`，会重定向到`localhost:8080/login`。输入用户名`darjun`，密码`handsome`，登录成功显示主页面。后面的请求都不需要验证了，请随意点击点击吧😀

### 总结

本文介绍了轻量级的，功能强大的路由库`gorilla/mux`。它支持丰富的请求匹配方法，子路由能极大地方便我们管理路由。由于兼容标准库`net/http`，所以可以无缝集成到使用`net/http`的程序中，利用为`net/http`编写的中间件资源。下一篇我们介绍`gorilla/handlers`——一些常用的中间件。

大家如果发现好玩、好用的 Go 语言库，欢迎到 Go 每日一库 GitHub 上提交 issue😄

## gorilla/reverse

产生可逆的正则表达式muxes regexp的基础



## gorilla/handlers

### 简介

关于中间件，前面几篇文章已经介绍的很多了。这里就不赘述了。`handlers`库提供的中间件可用于标准库`net/http`和所有支持`http.Handler`接口的框架。由于`gorilla/mux`也支持`http.Handler`接口，所以也可以与`handlers`库结合使用。**这就是兼容标准的好处**。

### 项目初始化&安装

本文代码使用 Go Modules。

创建目录并初始化：

```
$ mkdir -p gorilla/handlers && cd gorilla/handlers
$ go mod init github.com/darjun/go-daily-lib/gorilla/handlers
```

安装`gorilla/handlers`库：

```
`$ go get -u github.com/gorilla/handlers `
```

下面依次介绍各个中间件和相应的源码。

### 日志

`handlers`提供了两个日志中间件：

- `LoggingHandler`：以 Apache 的`Common Log Format`日志格式记录 HTTP 请求日志；
- `CombinedLoggingHandler`：以 Apache的`Combined Log Format`日志格式记录 HTTP 请求日志，Apache 和 Nginx 默认都使用这种日志格式。

两种日志格式差别很小，`Common Log Format`格式如下：

| `1 ` | `%h %l %u %t "%r" %>s %b ` |
| ---- | -------------------------- |
|      |                            |

各个指示符含义如下：

- `%h`：客户端的 IP 地址或主机名；

- `%l`：`RFC 1413`定义的客户端标识，由客户端机器上的`identd`程序生成。如果不存在，则该字段为`-`；

- `%u`：已验证的用户名。如果不存在，该字段为`-`；

- ```
  %t
  ```

  ：时间，格式为

  ```
  day/month/year:hour:minute:second zone
  ```

  ，其中：

  - `day`： 2位数字；
  - `month`：月份缩写，3个字母，如`Jan`；
  - `year`：4位数字；
  - `hour`：2位数字；
  - `minute`：2位数字；
  - `second`：2位数字；
  - `zone`：`+`或`-`后跟4位数字；
  - 例如：`21/Jul/2021:06:27:33 +0800`

- `%r`：包含 HTTP 请求行信息，例`GET /index.html HTTP/1.1`；

- `%>s`：服务器发送给客户端的状态码，例如`200`；

- `%b`：响应长度（字节数）。

`Combined Log Format`格式如下：

| `1 ` | `%h %l %u %t "%r" %>s %b "%{Referer}i" "%{User-Agent}i" ` |
| ---- | --------------------------------------------------------- |
|      |                                                           |

可见相比`Common Log Format`只是多了：

- `%{Referer}i`：HTTP 首部中的`Referer`信息；
- `%{User-Agent}i`：HTTP 首部中的`User-Agent`信息。

对中间件，我们可以让它作用于全局，即全部处理器，也可以让它只对某些处理器生效。如果要对所有处理器生效，可以调用`Use()`方法。如果只需要作用于特定的处理器，在注册时用中间件将处理器包装一层：

| ` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 ` | `func index(w http.ResponseWriter, r *http.Request) {  fmt.Fprintln(w, "Hello World") } type greeting string func (g greeting) ServeHTTP(w http.ResponseWriter, r *http.Request) {  fmt.Fprintf(w, "Welcome, %s", g) } func main() {  r := mux.NewRouter()  r.Handle("/", handlers.LoggingHandler(os.Stdout, http.HandlerFunc(index)))  r.Handle("/greeting", handlers.CombinedLoggingHandler(os.Stdout, greeting("dj")))   http.Handle("/", r)  log.Fatal(http.ListenAndServe(":8080", nil)) } ` |
| ------------------------------------------------ | ------------------------------------------------------------ |
|                                                  |                                                              |

上面代码中`LoggingHandler`只作用于处理函数`index`，`CombinedLoggingHandler`只作用于处理器`greeting("dj")`。

运行代码，通过浏览器访问`localhost:8080`和`localhost:8080/greeting`：

| `1 2 ` | `::1 - - [21/Jul/2021:06:39:45 +0800] "GET / HTTP/1.1" 200 12 ::1 - - [21/Jul/2021:06:39:54 +0800] "GET /greeting HTTP/1.1" 200 11 "" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.164 Safari/537.36" ` |
| ------ | ------------------------------------------------------------ |
|        |                                                              |

对照前面分析的指示符，很容易看出各个部分。

由于`*mux.Router`的`Use()`方法接受类型为`MiddlewareFunc`的中间件：

| `1 ` | `type MiddlewareFunc func(http.Handler) http.Handler ` |
| ---- | ------------------------------------------------------ |
|      |                                                        |

而`handlers.LoggingHandler/CombinedLoggingHandler`并不满足，所以还需要包装一层才能传给`Use()`方法：

| ` 1 2 3 4 5 6 7 8 9 10 11 12 13 ` | `func Logging(handler http.Handler) http.Handler {  return handlers.CombinedLoggingHandler(os.Stdout, handler) } func main() {  r := mux.NewRouter()  r.Use(Logging)  r.HandleFunc("/", index)  r.Handle("/greeting/", greeting("dj"))   http.Handle("/", r)  log.Fatal(http.ListenAndServe(":8080", nil)) } ` |
| --------------------------------- | ------------------------------------------------------------ |
|                                   |                                                              |

另外`handlers`还提供了`CustomLoggingHandler`，我们可以利用它定义自己的日志中间件：

| `1 ` | `func CustomLoggingHandler(out io.Writer, h http.Handler, f LogFormatter) http.Handler ` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

最关键的`LogFormatter`类型定义：

| `1 2 3 4 5 6 7 8 9 ` | `type LogFormatterParams struct {  Request    *http.Request  URL        url.URL  TimeStamp  time.Time  StatusCode int  Size       int } type LogFormatter func(writer io.Writer, params LogFormatterParams) ` |
| -------------------- | ------------------------------------------------------------ |
|                      |                                                              |

我们实现一个简单的`LogFormatter`，记录时间 + 请求行 + 响应码：

| ` 1 2 3 4 5 6 7 8 9 10 11 12 13 ` | `func myLogFormatter(writer io.Writer, params handlers.LogFormatterParams) {  var buf bytes.Buffer  buf.WriteString(time.Now().Format("2006-01-02 15:04:05 -0700"))  buf.WriteString(fmt.Sprintf(` "%s %s %s" `, params.Request.Method, params.URL.Path, params.Request.Proto))  buf.WriteString(strconv.Itoa(params.StatusCode))  buf.WriteByte('\n')   writer.Write(buf.Bytes()) } func Logging(handler http.Handler) http.Handler {  return handlers.CustomLoggingHandler(os.Stdout, handler, myLogFormatter) } ` |
| --------------------------------- | ------------------------------------------------------------ |
|                                   |                                                              |

使用：

| `1 2 3 4 5 6 7 8 9 ` | `func main() {  r := mux.NewRouter()  r.Use(Logging)  r.HandleFunc("/", index)  r.Handle("/greeting/", greeting("dj"))   http.Handle("/", r)  log.Fatal(http.ListenAndServe(":8080", nil)) } ` |
| -------------------- | ------------------------------------------------------------ |
|                      |                                                              |

现在记录的日志是下面这种格式：

| `1 ` | `2021-07-21 07:03:18 +0800 "GET /greeting/ HTTP/1.1" 200 ` |
| ---- | ---------------------------------------------------------- |
|      |                                                            |

翻看源码，我们可以发现`LoggingHandler/CombinedLoggingHandler/CustomLoggingHandler`都是基于底层的`loggingHandler`实现的，不同的是`LoggingHandler`使用了预定义的`writeLog`作为`LogFormatter`，`CombinedLoggingHandler`使用了预定义的`writeCombinedLog`作为`LogFormatter`，而`CustomLoggingHandler`使用我们自己定义的`LogFormatter`：

| ` 1 2 3 4 5 6 7 8 9 10 11 ` | `func CombinedLoggingHandler(out io.Writer, h http.Handler) http.Handler {  return loggingHandler{out, h, writeCombinedLog} } func LoggingHandler(out io.Writer, h http.Handler) http.Handler {  return loggingHandler{out, h, writeLog} } func CustomLoggingHandler(out io.Writer, h http.Handler, f LogFormatter) http.Handler {  return loggingHandler{out, h, f} } ` |
| --------------------------- | ------------------------------------------------------------ |
|                             |                                                              |

预定义的`writeLog/writeCombinedLog`实现如下：

| ` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 ` | `func writeLog(writer io.Writer, params LogFormatterParams) {  buf := buildCommonLogLine(params.Request, params.URL, params.TimeStamp, params.StatusCode, params.Size)  buf = append(buf, '\n')  writer.Write(buf) } func writeCombinedLog(writer io.Writer, params LogFormatterParams) {  buf := buildCommonLogLine(params.Request, params.URL, params.TimeStamp, params.StatusCode, params.Size)  buf = append(buf, ` "`...)  buf = appendQuoted(buf, params.Request.Referer())  buf = append(buf, `" "`...)  buf = appendQuoted(buf, params.Request.UserAgent())  buf = append(buf, '"', '\n')  writer.Write(buf) } ` |
| --------------------------------------- | ------------------------------------------------------------ |
|                                         |                                                              |

它们都是基于`buildCommonLogLine`构造基本信息，`writeCombinedLog`还分别调用`http.Request.Referer()`和`http.Request.UserAgent`获取了`Referer`和`User-Agent`信息。

`loggingHandler`定义如下：

| `1 2 3 4 5 ` | `type loggingHandler struct {  writer    io.Writer  handler   http.Handler  formatter LogFormatter } ` |
| ------------ | ------------------------------------------------------------ |
|              |                                                              |

`loggingHandler`实现有一个比较巧妙的地方：为了记录响应码和响应大小，定义了一个类型`responseLogger`包装原来的`http.ResponseWriter`，在写入时记录信息：

| ` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 ` | `type responseLogger struct {  w      http.ResponseWriter  status int  size   int } func (l *responseLogger) Write(b []byte) (int, error) {  size, err := l.w.Write(b)  l.size += size  return size, err } func (l *responseLogger) WriteHeader(s int) {  l.w.WriteHeader(s)  l.status = s } func (l *responseLogger) Status() int {  return l.status } func (l *responseLogger) Size() int {  return l.size } ` |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

`loggingHandler`的关键方法`ServeHTTP()`：

| ` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 ` | `func (h loggingHandler) ServeHTTP(w http.ResponseWriter, req *http.Request) {  t := time.Now()  logger, w := makeLogger(w)  url := *req.URL   h.handler.ServeHTTP(w, req)  if req.MultipartForm != nil {    req.MultipartForm.RemoveAll()  }   params := LogFormatterParams{    Request:    req,    URL:        url,    TimeStamp:  t,    StatusCode: logger.Status(),    Size:       logger.Size(),  }   h.formatter(h.writer, params) } ` |
| ------------------------------------------------------ | ------------------------------------------------------------ |
|                                                        |                                                              |

构造`LogFormatterParams`对象，调用对应的`LogFormatter`函数。

### 压缩

如果客户端请求中有`Accept-Encoding`首部，服务器可以使用该首部指示的算法将响应压缩，以节省网络流量。`handlers.CompressHandler`中间件启用压缩功能。还有一个`CompressHandlerLevel`可以指定压缩级别。实际上`CompressHandler`就是使用`gzip.DefaultCompression`调用的`CompressHandlerLevel`：

| `1 2 3 ` | `func CompressHandler(h http.Handler) http.Handler {  return CompressHandlerLevel(h, gzip.DefaultCompression) } ` |
| -------- | ------------------------------------------------------------ |
|          |                                                              |

看代码：

| ` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 ` | `func index(w http.ResponseWriter, r *http.Request) {  fmt.Fprintln(w, "Hello World") } type greeting string func (g greeting) ServeHTTP(w http.ResponseWriter, r *http.Request) {  fmt.Fprintf(w, "Welcome, %s", g) } func main() {  r := mux.NewRouter()  r.Use(handlers.CompressHandler)  r.HandleFunc("/", index)  r.Handle("/greeting/", greeting("dj"))   http.Handle("/", r)  log.Fatal(http.ListenAndServe(":8080", nil)) } ` |
| --------------------------------------------------- | ------------------------------------------------------------ |
|                                                     |                                                              |

运行，请求`localhost:8080`，通过 Chrome 开发者工具的 Network 页签可以看到响应采用了 gzip 压缩：

![img](https://darjun.github.io/img/in-post/godailylib/handlers1.png#center)

忽略一些细节处理，`CompressHandlerLevel`函数代码如下：

| ` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 ` | `func CompressHandlerLevel(h http.Handler, level int) http.Handler {  const (    gzipEncoding  = "gzip"    flateEncoding = "deflate"  )   return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {    var encoding string    for _, curEnc := range strings.Split(r.Header.Get(acceptEncoding), ",") {      curEnc = strings.TrimSpace(curEnc)      if curEnc == gzipEncoding || curEnc == flateEncoding {        encoding = curEnc        break      }    }     if encoding == "" {      h.ServeHTTP(w, r)      return    }     if r.Header.Get("Upgrade") != "" {      h.ServeHTTP(w, r)      return    }     var encWriter io.WriteCloser    if encoding == gzipEncoding {      encWriter, _ = gzip.NewWriterLevel(w, level)    } else if encoding == flateEncoding {      encWriter, _ = flate.NewWriter(w, level)    }    defer encWriter.Close()     w.Header().Set("Content-Encoding", encoding)    r.Header.Del(acceptEncoding)     cw := &compressResponseWriter{      w:          w,      compressor: encWriter,    }     w = httpsnoop.Wrap(w, httpsnoop.Hooks{      Write: func(httpsnoop.WriteFunc) httpsnoop.WriteFunc {        return cw.Write      },      WriteHeader: func(httpsnoop.WriteHeaderFunc) httpsnoop.WriteHeaderFunc {        return cw.WriteHeader      },      Flush: func(httpsnoop.FlushFunc) httpsnoop.FlushFunc {        return cw.Flush      },      ReadFrom: func(rff httpsnoop.ReadFromFunc) httpsnoop.ReadFromFunc {        return cw.ReadFrom      },    })     h.ServeHTTP(w, r)  }) } ` |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

从请求`Accept-Encoding`首部中获取客户端指示的压缩算法。如果客户端未指定，或请求首部中有`Upgrade`，则不压缩。反之，则压缩。根据识别的压缩算法，创建对应`gzip`或`flate`的`io.Writer`实现对象。

与前面的日志中间件一样，为了压缩写入的内容，新增类型`compressResponseWriter`封装`http.ResponseWriter`，重写`Write()`方法，将写入的字节流传入前面创建的`io.Writer`实现压缩：

| ` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 ` | `type compressResponseWriter struct {  compressor io.Writer  w          http.ResponseWriter } func (cw *compressResponseWriter) Write(b []byte) (int, error) {  h := cw.w.Header()  if h.Get("Content-Type") == "" {    h.Set("Content-Type", http.DetectContentType(b))  }  h.Del("Content-Length")   return cw.compressor.Write(b) } ` |
| ------------------------------------ | ------------------------------------------------------------ |
|                                      |                                                              |

### 内容类型

我们可以通过`handler.ContentTypeHandler`指定请求的`Content-Type`必须在我们给出的类型中，只对`POST/PUT/PATCH`方法生效。例如我们限制登录请求必须通过`application/x-www-form-urlencoded`的形式发送：

| ` 1 2 3 4 5 6 7 8 9 10 ` | `func main() {  r := mux.NewRouter()  r.HandleFunc("/", index)  r.Methods("GET").Path("/login").HandlerFunc(login)  r.Methods("POST").Path("/login").    Handler(handlers.ContentTypeHandler(http.HandlerFunc(dologin), "application/x-www-form-urlencoded"))   http.Handle("/", r)  log.Fatal(http.ListenAndServe(":8080", nil)) } ` |
| ------------------------ | ------------------------------------------------------------ |
|                          |                                                              |

这样，只要请求`/login`的`Content-Type`不是`application/x-www-form-urlencoded`就会返回 415 错误。我们可以故意写错，再请求看看表现：

| `1 ` | `Unsupported content type "application/x-www-form-urlencoded"; expected one of ["application/x-www-from-urlencoded"] ` |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

`ContentTypeHandler`的实现非常简单：

| ` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 ` | `func ContentTypeHandler(h http.Handler, contentTypes ...string) http.Handler {  return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {    if !(r.Method == "PUT" || r.Method == "POST" || r.Method == "PATCH") {      h.ServeHTTP(w, r)      return    }     for _, ct := range contentTypes {      if isContentType(r.Header, ct) {        h.ServeHTTP(w, r)        return      }    }    http.Error(w, fmt.Sprintf("Unsupported content type %q; expected one of %q", r.Header.Get("Content-Type"), contentTypes), http.StatusUnsupportedMediaType)  }) } ` |
| ------------------------------------------ | ------------------------------------------------------------ |
|                                            |                                                              |

就是读取`Content-Type`首部，判断是否在我们指定的类型中。

### 方法分发器

在上面的例子中，我们注册路径`/login`的`GET`和`POST`方法处理采用`r.Methods("GET").Path("/login").HandlerFunc(login)`这种冗长的写法。`handlers.MethodHandler`可以简化这种写法：

| ` 1 2 3 4 5 6 7 8 9 10 11 ` | `func main() {  r := mux.NewRouter()  r.HandleFunc("/", index)  r.Handle("/login", handlers.MethodHandler{    "GET":  http.HandlerFunc(login),    "POST": http.HandlerFunc(dologin),  })   http.Handle("/", r)  log.Fatal(http.ListenAndServe(":8080", nil)) } ` |
| --------------------------- | ------------------------------------------------------------ |
|                             |                                                              |

`MethodHandler`底层是一个`map[string]http.Handler`类型，它的`ServeHTTP()`方法根据请求的 Method 调用不同的处理：

| ` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 ` | `type MethodHandler map[string]http.Handler func (h MethodHandler) ServeHTTP(w http.ResponseWriter, req *http.Request) {  if handler, ok := h[req.Method]; ok {    handler.ServeHTTP(w, req)  } else {    allow := []string{}    for k := range h {      allow = append(allow, k)    }    sort.Strings(allow)    w.Header().Set("Allow", strings.Join(allow, ", "))    if req.Method == "OPTIONS" {      w.WriteHeader(http.StatusOK)    } else {      http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)    }  } } ` |
| --------------------------------------------------- | ------------------------------------------------------------ |
|                                                     |                                                              |

方法如果未注册，则返回`405 Method Not Allowed`。有一个方法除外，`OPTIONS`。该方法通过`Allow`首部返回支持哪些方法。

### 重定向

`handlers.CanonicalHost`可以将请求重定向到指定的域名，同时指定重定向响应码。在同一个服务器对应多个域名时比较有用：

| ` 1 2 3 4 5 6 7 8 9 10 11 ` | `func index(w http.ResponseWriter, r *http.Request) {  fmt.Fprintln(w, "hello world") } func main() {  r := mux.NewRouter()  r.Use(handlers.CanonicalHost("http://www.gorillatoolkit.org", 302))  r.HandleFunc("/", index)  http.Handle("/", r)  log.Fatal(http.ListenAndServe(":8080", nil)) } ` |
| --------------------------- | ------------------------------------------------------------ |
|                             |                                                              |

上面将所有请求以 302 重定向到`http://www.gorillatoolkit.org`。

`CanonicalHost`的实现也很简单：

| `1 2 3 4 5 6 7 ` | `func CanonicalHost(domain string, code int) func(h http.Handler) http.Handler {  fn := func(h http.Handler) http.Handler {    return canonical{h, domain, code}  }   return fn } ` |
| ---------------- | ------------------------------------------------------------ |
|                  |                                                              |

关键类型`canonical`：

| `1 2 3 4 5 ` | `type canonical struct {  h      http.Handler  domain string  code   int } ` |
| ------------ | ------------------------------------------------------------ |
|              |                                                              |

核心方法：

| ` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 ` | `func (c canonical) ServeHTTP(w http.ResponseWriter, r *http.Request) {  dest, err := url.Parse(c.domain)  if err != nil {    c.h.ServeHTTP(w, r)    return  }   if dest.Scheme == "" || dest.Host == "" {    c.h.ServeHTTP(w, r)    return  }   if !strings.EqualFold(cleanHost(r.Host), dest.Host) {    dest := dest.Scheme + "://" + dest.Host + r.URL.Path    if r.URL.RawQuery != "" {      dest += "?" + r.URL.RawQuery    }    http.Redirect(w, r, dest, c.code)    return  }   c.h.ServeHTTP(w, r) } ` |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
|                                                              |                                                              |

由源码可知，域名不合法或未指定协议（Scheme）或域名（Host）的请求下不转发。

### Recovery

之前我们自己实现了`PanicRecover`中间件，避免请求处理时 panic。`handlers`提供了一个`RecoveryHandler`可以直接使用：

| ` 1 2 3 4 5 6 7 8 9 10 11 ` | `func PANIC(w http.ResponseWriter, r *http.Request) {  panic(errors.New("unexpected error")) } func main() {  r := mux.NewRouter()  r.Use(handlers.RecoveryHandler(handlers.PrintRecoveryStack(true)))  r.HandleFunc("/", PANIC)  http.Handle("/", r)  log.Fatal(http.ListenAndServe(":8080", nil)) } ` |
| --------------------------- | ------------------------------------------------------------ |
|                             |                                                              |

选项`PrintRecoveryStack`表示 panic 时输出堆栈信息。

`RecoveryHandler`的实现与之前我们自己编写的基本一样：

| ` 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 ` | `type recoveryHandler struct {  handler    http.Handler  logger     RecoveryHandlerLogger  printStack bool } func (h recoveryHandler) ServeHTTP(w http.ResponseWriter, req *http.Request) {  defer func() {    if err := recover(); err != nil {      w.WriteHeader(http.StatusInternalServerError)      h.log(err)    }  }()   h.handler.ServeHTTP(w, req) } ` |
| ------------------------------------------ | ------------------------------------------------------------ |
|                                            |                                                              |

### 总结

GitHub 上有很多开源的 Go Web 中间件实现，可以直接拿来使用，避免重复造轮子。`handlers`很轻量，容易与标准库`net/http`和 gorilla 路由库`mux`结合使用。
