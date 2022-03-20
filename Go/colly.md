# go colly 爬虫

## 一、[基本使用](https://segmentfault.com/a/1190000040275945)

### **安装**

```go
go get -u github.com/gocolly/colly
```

### **使用**

```go
package main

import (
  "fmt"

  "github.com/gocolly/colly/v2"
)

func main() {
  c := colly.NewCollector(
      colly.AllowedDomains("www.baidu.com" ),
  )

  c.OnRequest(func(r *colly.Request) {
      fmt.Println("Visiting", r.URL.String())
  })

  c.OnResponse(func(r *colly.Response) {
      fmt.Printf("Response %s: %d bytes\n", r.Request.URL, len(r.Body))
  })

  c.OnError(func(r *colly.Response, err error) {
      fmt.Printf("Error %s: %v\n", r.Request.URL, err)
  })
    
  c.OnHTML("a[href]", func(e *colly.HTMLElement) {
      link := e.Attr("href")
      fmt.Printf("Link found: %q -> %s\n", e.Text, link)
      c.Visit(e.Request.AbsoluteURL(link))
    })

  c.Visit("http://www.baidu.com/")
}
```

`colly`的使用比较简单：

首先，调用`colly.NewCollector()`创建一个类型为`*colly.Collector`的爬虫对象。由于每个网页都有很多指向其他网页的链接。如果不加限制的话，运行可能永远不会停止。所以上面通过传入一个选项`colly.AllowedDomains("www.baidu.com")`限制只爬取域名为`www.baidu.com`的网页。

调用`c.OnRequest()`方法注册请求回调，每次发送请求时执行该回调，这里只是简单打印请求的 URL。

调用`c.OnResponse()`方法注册响应回调，每次收到响应时执行该回调，这里也只是简单的打印 URL 和响应大小。

调用`c.OnError()`方法注册错误回调，执行请求发生错误时执行该回调，这里简单打印 URL 和错误信息。

然后我们调用`c.OnHTML`方法注册`HTML`回调，对每个有`href`属性的`a`元素执行回调函数。这里继续访问`href`指向的 URL。也就是说解析爬取到的网页，然后继续访问网页中指向其他页面的链接。

最后我们调用`c.Visit()`开始访问第一个页面。

```
$ go run main.go
Visiting http://www.baidu.com/
Response http://www.baidu.com/: 303317 bytes
Link found: "百度首页" -> /
Link found: "设置" -> javascript:;
Link found: "登录" -> https://passport.baidu.com/v2/?login&tpl=mn&u=http%3A%2F%2Fwww.baidu.com%2F&sms=5
Link found: "新闻" -> http://news.baidu.com
Link found: "hao123" -> https://www.hao123.com
Link found: "地图" -> http://map.baidu.com
Link found: "直播" -> https://live.baidu.com/
Link found: "视频" -> https://haokan.baidu.com/?sfrom=baidu-top
Link found: "贴吧" -> http://tieba.baidu.com
...
```

`colly`爬取到页面之后，会使用[goquery](https://link.segmentfault.com/?enc=P1JZV6MDYvKKa%2BWn2IQXdw%3D%3D.4FX3gBz2JNo8Z24%2BhA9VEuqshjoO8DfCfigE15ybbWI%2FKICwMhXStM%2Bx02SULUXIOIrSDewEAHTi9ZOol4Ss%2BQ%3D%3D)解析这个页面。然后查找注册的 HTML 回调对应元素选择器（element-selector），将`goquery.Selection`封装成一个`colly.HTMLElement`执行回调。

`colly.HTMLElement`其实就是对`goquery.Selection`的简单封装：

```
type HTMLElement struct {
  Name string 	// Name is the name of the tag
  Text string	// Text is the content of the tag
  attributes []html.Attribute // 该元素的属性
  Request *Request	//Request is the request object of the element's HTML document
  Response *Response	// Response is the Response object of the element's HTML document
  DOM *goquery.Selection	// DOM is the goquery parsed DOM object of the page. DOM is relative to the current HTMLElement
  Index int	// Index stores the position of the current element within all the elements matched by an OnHTML callback
}
```

并提供了简单易用的方法：

- `Attr(k string)`：返回当前元素的属性，上面示例中我们使用`e.Attr("href")`获取了`href`属性；

- `ChildAttr(goquerySelector, attrName string)`：返回`goquerySelector`选择的第一个子元素的`attrName`属性；

- `ChildAttrs(goquerySelector, attrName string)`：返回`goquerySelector`选择的所有子元素的`attrName`属性，以`[]string`返回；

- `ChildText(goquerySelector string)`：拼接`goquerySelector`选择的子元素的文本内容并返回；

- `ChildTexts(goquerySelector string)`：返回`goquerySelector`选择的子元素的文本内容组成的切片，以`[]string`返回。

- `ForEach(goquerySelector string, callback func(int, *HTMLElement))`：对每个`goquerySelector`选择的子元素执行回调`callback`；

- `Unmarshal(v interface{})`：通过给结构体字段指定` goquerySelector` 格式的 tag，可以将一个 `HTMLElement` 对象 `Unmarshal `到一个结构体实例中。

  Unmarshal函数的用法。

```
type Hot struct {
  Rank   string `selector:"a > div.index_1Ew5p"`
  Name   string `selector:"div.content_1YWBm > a.title_dIF3B"`
  Author string `selector:"div.content_1YWBm > div.intro_1l0wp:nth-child(2)"`
  Type   string `selector:"div.content_1YWBm > div.intro_1l0wp:nth-child(3)"`
  Desc   string `selector:"div.desc_3CTjT"`
}
```

tag 中是 CSS 选择器语法，添加这个是为了可以直接调用`HTMLElement.Unmarshal()`方法填充`Hot`对象。

然后创建`Collector`对象：

```go
c := colly.NewCollector()
```

注册回调：

```go
c.OnHTML("div.category-wrap_iQLoo", func(e *colly.HTMLElement) {
  hot := &Hot{}

  err := e.Unmarshal(hot)
  if err != nil {
    fmt.Println("error:", err)
    return
  }

  hots = append(hots, hot)
})

c.OnRequest(func(r *colly.Request) {
  fmt.Println("Requesting:", r.URL)
})

c.OnResponse(func(r *colly.Response) {
  fmt.Println("Response:", len(r.Body))
})
```

`OnHTML`对每个条目执行`Unmarshal`生成`Hot`对象。

`OnRequest/OnResponse`只是简单输出调试信息。

然后，调用`c.Visit()`访问网址：

### **限速**

有时候并发请求太多，网站会限制访问。这时就需要使用`LimitRule`了。说白了，`LimitRule`就是限制访问速度和并发量的：

```go
type LimitRule struct {
  DomainRegexp string
  DomainGlob string
  Delay time.Duration
  RandomDelay time.Duration
  Parallelism    int
}
```

常用的就`Delay/RandomDelay/Parallism`这几个，分别表示请求与请求之间的延迟，随机延迟，和并发数。另外**必须**指定对哪些域名施行限制，通过`DomainRegexp`或`DomainGlob`设置，如果这两个字段都未设置`Limit()`方法会返回错误。

```go
err := c.Limit(&colly.LimitRule{
  DomainRegexp: `unsplash\.com`,
  RandomDelay:  500 * time.Millisecond,
  Parallelism:  12,
})
if err != nil {
  log.Fatal(err)
}
```

我们设置针对`unsplash.com`这个域名，请求与请求之间的随机最大延迟 500ms，最多同时并发 12 个请求。

**设置超时**

有时候网速较慢，`colly`中使用的`http.Client`有默认超时机制，我们可以通过`colly.WithTransport()`选项改写：

```go
c.WithTransport(&http.Transport{
  Proxy: http.ProxyFromEnvironment,
  DialContext: (&net.Dialer{
    Timeout:   30 * time.Second,
    KeepAlive: 30 * time.Second,
  }).DialContext,
  MaxIdleConns:          100,
  IdleConnTimeout:       90 * time.Second,
  TLSHandshakeTimeout:   10 * time.Second,
  ExpectContinueTimeout: 1 * time.Second,
})
```

### **扩展**

`colly`在子包`extension`中提供了一些扩展特性，最最常用的就是随机 User-Agent 了。通常网站会通过 User-Agent 识别请求是否是浏览器发出的，爬虫一般会设置这个 Header 把自己伪装成浏览器。使用也比较简单：

```go
import "github.com/gocolly/colly/v2/extensions"

func main() {
  c := colly.NewCollector()
  extensions.RandomUserAgent(c)
}
```

随机 User-Agent 实现也很简单，就是从一些预先定义好的 User-Agent 数组中随机一个设置到 Header 中：

```go
func RandomUserAgent(c *colly.Collector) {
  c.OnRequest(func(r *colly.Request) {
    r.Headers.Set("User-Agent", uaGens[rand.Intn(len(uaGens))]())
  })
}
```

实现自己的扩展也不难，例如我们每次请求时需要设置一个特定的 Header，扩展可以这么写：

```go
func MyHeader(c *colly.Collector) {
  c.OnRequest(func(r *colly.Request) {
    r.Headers.Set("My-Header", "dj")
  })
}
```

用`Collector`对象调用`MyHeader()`函数即可：

```go
MyHeader(c)
```

### **[官方示例](https://blog.csdn.net/qq_27818541/category_10717854.html)**

### [示例](https://www.jianshu.com/p/cbe0f6aae5bf)



## 二、Colly源码解析

[**框架**](https://blog.csdn.net/breaksoftware/article/details/84564875)

**[结合例子](https://blog.csdn.net/breaksoftware/article/details/84582416)**

**[go爬虫框架colly源码以及软件架构分析](https://cloud.tencent.com/developer/article/1429378)**

### **架构特点**

了解爬虫的都知道一个爬虫请求的生命周期

>  

1. 构建请求
2. 发送请求
3. 获取文档或数据
4. 解析文档或清洗数据
5. 数据处理或持久化

scrapy的设计理念是将上面的每一个步骤抽离出来，然后做出组件的形式， 最后通过调度组成流水线的工作形式。 我们看一下scrapy的架构图， 这里只是简单的介绍下， 后面有时间，我深入介绍scrapy

![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/SCRAPY.png)

如图，`downloader`负责请求获取页面，`spiders`中写具体解析文档的逻辑，`item PipeLine`数据最后处理， 中间有一些[中间件](https://cloud.tencent.com/product/tdmq?from=10680)，可以一些功能的装饰。比如，代理，请求频率等。

我们介绍一下colly的架构特点 colly的逻辑更像是面向过程编程的， colly的逻辑就是按上面生命周期的顺序管道处理， 只是在不同阶段，加上回调函数进行过滤的时候进行处理。

<img src="https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/COLLY.png" style="zoom: 25%;" />

下面也按照这个逻辑进行介绍

### 源码分析

先给一个?

```go
package main

import (
    "fmt"

    "github.com/gocolly/colly"
)

func main() {
    // Instantiate default collector
    c := colly.NewCollector(
        // Visit only domains: hackerspaces.org, wiki.hackerspaces.org
        colly.AllowedDomains("hackerspaces.org", "wiki.hackerspaces.org"),
    )

    // On every a element which has href attribute call callback
    c.OnHTML("a[href]", func(e *colly.HTMLElement) {
        link := e.Attr("href")
        // Print link
        fmt.Printf("Link found: %q -> %s\n", e.Text, link)
        // Visit link found on page
        // Only those links are visited which are in AllowedDomains
        c.Visit(e.Request.AbsoluteURL(link))
    })

    // Before making a request print "Visiting ..."
    c.OnRequest(func(r *colly.Request) {
        fmt.Println("Visiting", r.URL.String())
    })

    // Start scraping on https://hackerspaces.org
    c.Visit("https://hackerspaces.org/")
}
```

这是官方给的示例， 可以看到`colly.NewCollector`创建一个`收集器`， colly的所有处理逻辑都是以`Collector`为核心进行操作的。

我们看一下 `Collector`结构体的定义

```go
// Collector provides the scraper instance for a scraping job
type Collector struct {
    // UserAgent is the User-Agent string used by HTTP requests
    UserAgent string
    // MaxDepth limits the recursion depth of visited URLs.
    // Set it to 0 for infinite recursion (default).
    MaxDepth int
    // AllowedDomains is a domain whitelist.
    // Leave it blank to allow any domains to be visited
    AllowedDomains []string
    // DisallowedDomains is a domain blacklist.
    DisallowedDomains []string
    // DisallowedURLFilters is a list of regular expressions which restricts
    // visiting URLs. If any of the rules matches to a URL the
    // request will be stopped. DisallowedURLFilters will
    // be evaluated before URLFilters
    // Leave it blank to allow any URLs to be visited
    DisallowedURLFilters []*regexp.Regexp
    // URLFilters is a list of regular expressions which restricts
    // visiting URLs. If any of the rules matches to a URL the
    // request won't be stopped. DisallowedURLFilters will
    // be evaluated before URLFilters

    // Leave it blank to allow any URLs to be visited
    URLFilters []*regexp.Regexp

    // AllowURLRevisit allows multiple downloads of the same URL
    AllowURLRevisit bool
    // MaxBodySize is the limit of the retrieved response body in bytes.
    // 0 means unlimited.
    // The default value for MaxBodySize is 10MB (10 * 1024 * 1024 bytes).
    MaxBodySize int
    // CacheDir specifies a location where GET requests are cached as files.
    // When it's not defined, caching is disabled.
    CacheDir string
    // IgnoreRobotsTxt allows the Collector to ignore any restrictions set by
    // the target host's robots.txt file.  See http://www.robotstxt.org/ for more
    // information.
    IgnoreRobotsTxt bool
    // Async turns on asynchronous network communication. Use Collector.Wait() to
    // be sure all requests have been finished.
    Async bool
    // ParseHTTPErrorResponse allows parsing HTTP responses with non 2xx status codes.
    // By default, Colly parses only successful HTTP responses. Set ParseHTTPErrorResponse
    // to true to enable it.
    ParseHTTPErrorResponse bool
    // ID is the unique identifier of a collector
    ID uint32
    // DetectCharset can enable character encoding detection for non-utf8 response bodies
    // without explicit charset declaration. This feature uses https://github.com/saintfish/chardet
    DetectCharset bool
    // RedirectHandler allows control on how a redirect will be managed
    RedirectHandler func(req *http.Request, via []*http.Request) error
    // CheckHead performs a HEAD request before every GET to pre-validate the response
    CheckHead         bool
    store             storage.Storage
    debugger          debug.Debugger
    robotsMap         map[string]*robotstxt.RobotsData
    htmlCallbacks     []*htmlCallbackContainer
    xmlCallbacks      []*xmlCallbackContainer
    requestCallbacks  []RequestCallback
    responseCallbacks []ResponseCallback
    errorCallbacks    []ErrorCallback
    scrapedCallbacks  []ScrapedCallback
    requestCount      uint32
    responseCount     uint32
    backend           *httpBackend
    wg                *sync.WaitGroup
    lock              *sync.RWMutex
}
```

Collector中绝大部分成员均有对应的方法，而且它们的名称（函数名和成员名）也一致。但是其中只有3个方法——ParseHTTPErrorResponse、AllowURLRevisit和IgnoreRobotsTxt比较特殊，因为它们没有参数。如果被调用，则对应的Collector成员会被设置为true

```go
// AllowURLRevisit instructs the Collector to allow multiple downloads of the same URL
func AllowURLRevisit() func(*Collector) {
	return func(c *Collector) {
		c.AllowURLRevisit = true
	}
}
```

再回到NewCollector函数，其最后一个逻辑是调用parseSettingsFromEnv方法。从名称我们可以看出它是用于解析环境变量的。将它放在最后是可以理解的，因为后面执行的逻辑可以覆盖前面的逻辑。这样我们可以让环境变量对应的设置生效。

```Go
func (c *Collector) parseSettingsFromEnv() {
	for _, e := range os.Environ() {
		if !strings.HasPrefix(e, "COLLY_") {
			continue
		}

		pair := strings.SplitN(e[6:], "=", 2)
        
		if f, ok := envMap[pair[0]]; ok {
			f(c, pair[1])
		} else {
			log.Println("Unknown environment variable:", pair[0])
		}
	}
}
```

​    它从os.Environ()中获取系统环境变量，然后遍历它们。对于以COLLY_开头的变量，找到其在envMap中的对应方法，并调用之以覆盖之前设置的Collector成员变量值。envMap是一个<string,func>的映射，它是包内全局的。

```Go
var envMap = map[string]func(*Collector, string){

	"ALLOWED_DOMAINS": func(c *Collector, val string) {
		c.AllowedDomains = strings.Split(val, ",")
	},
	"CACHE_DIR": func(c *Collector, val string) {
		c.CacheDir = val
	},

……
```

​    初始化完Collector，我们就可以让其发送请求。目前Colly公开了5个方法，其中3个是和Post相关的：Post、PostRaw和PostMultipart。一个Get请求方法：Visit。以及一个用户可以高度定制的方法：Request。这些方法底层都调用了scrape方法。比如Visit的实现是

```Go
func (c *Collector) Visit(URL string) error {
	return c.scrape(URL, "GET", 1, nil, nil, nil, true)
}
```

### scrape

​    scrape方法是需要我们展开分析的。因为它是Colly库中两个最重要的方法之一

```go
// Visit starts Collector's collecting job by creating a
// request to the URL specified in parameter.
// Visit also calls the previously provided callbacks
func (c *Collector) Visit(URL string) error {
    if c.CheckHead {
        if check := c.scrape(URL, "HEAD", 1, nil, nil, nil, true); check != nil {
            return check
        }
    }
    return c.scrape(URL, "GET", 1, nil, nil, nil, true)
}

// 首先requestCheck方法检测一些和递归深度以及URL相关的信息
func (c *Collector) requestCheck(u, method string, depth int, checkRevisit bool) error {
	if u == "" {
		return ErrMissingURL
	}
    //  Collector的MaxDepth默认设置为0，即不用比较深度。如果它被设置值，则递归深度不可以超过它。
	if c.MaxDepth > 0 && c.MaxDepth < depth {
		return ErrMaxDepth
	}

func (c *Collector) scrape(u, method string, depth int, requestData io.Reader, ctx *Context, hdr http.Header, checkRevisit bool) error {
    // 检查请求是否合法
    if err := c.requestCheck(u, method, depth, checkRevisit); err != nil {
        return err
    }
    // 解析url，
    parsedURL, err := url.Parse(u)
    if err != nil {
        return err
    }
    if parsedURL.Scheme == "" {
        parsedURL.Scheme = "http"
    }
    
    //  然后针对host进行精确匹配（在requestCheck中，是对URL使用正则进行匹配）。先检测host是否在被禁止的列表中，然后检测其是否在准入的列表中。
    if !c.isDomainAllowed(parsedURL.Hostname()) {
        return ErrForbiddenDomain
    }
    // robots协议
    if method != "HEAD" && !c.IgnoreRobotsTxt {
        if err = c.checkRobots(parsedURL); err != nil {
            return err
        }
    }
     // headers
    if hdr == nil {
        hdr = http.Header{"User-Agent": []string{c.UserAgent}}
    }
    rc, ok := requestData.(io.ReadCloser)
    if !ok && requestData != nil {
        rc = ioutil.NopCloser(requestData)
    }
    // The Go HTTP API ignores "Host" in the headers, preferring the client
    // to use the Host field on Request.
    host := parsedURL.Host
    if hostHeader := hdr.Get("Host"); hostHeader != "" {
        host = hostHeader
    }
    // 构造http.Request
    req := &http.Request{
        Method:     method,
        URL:        parsedURL,
        Proto:      "HTTP/1.1",
        ProtoMajor: 1,
        ProtoMinor: 1,
        Header:     hdr,
        Body:       rc,
        Host:       host,
    }
    // 请求的数据（requestData）转换成io.ReadCloser接口数据
    setRequestBody(req, requestData)
    u = parsedURL.String()
    c.wg.Add(1)
    // 异步方式
    if c.Async {
        go c.fetch(u, method, depth, requestData, ctx, hdr, req)
        return nil
    }
    return c.fetch(u, method, depth, requestData, ctx, hdr, req)
}
```

上面很大篇幅都是检查， 现在还在 `request`的阶段， 还没有response，看`c.fetch`

fetch就是colly的核心内容

每次调用fetch方法都会构建一个全新Request结构。

```go
func (c *Collector) fetch(u, method string, depth int, requestData io.Reader, ctx *Context, hdr http.Header, req *http.Request) error {
    defer c.wg.Done()
    if ctx == nil {
        ctx = NewContext()
    }
    request := &Request{
        URL:       req.URL,
        Headers:   &req.Header,
        Ctx:       ctx,
        Depth:     depth,
        Method:    method,
        Body:      requestData,
        collector: c, // 这里将Collector放到request中，这个可以对请求继续处理
        ID:        atomic.AddUint32(&c.requestCount, 1),
    }
    // 回调函数处理 request
    c.handleOnRequest(request)

    if request.abort {
        return nil
    }

    if method == "POST" && req.Header.Get("Content-Type") == "" {
        req.Header.Add("Content-Type", "application/x-www-form-urlencoded")
    }

    if req.Header.Get("Accept") == "" {
        req.Header.Set("Accept", "*/*")
    }

    origURL := req.URL
    // 这里是 去请求网络， 是调用了 `http.Client.Do`方法请求的
    response, err := c.backend.Cache(req, c.MaxBodySize, c.CacheDir)
    if proxyURL, ok := req.Context().Value(ProxyURLKey).(string); ok {
        request.ProxyURL = proxyURL
    }
    // 回调函数，处理error
    if err := c.handleOnError(response, err, request, ctx); err != nil {
        return err
    }
    if req.URL != origURL {
        request.URL = req.URL
        request.Headers = &req.Header
    }
    atomic.AddUint32(&c.responseCount, 1)
    response.Ctx = ctx
    response.Request = request

    err = response.fixCharset(c.DetectCharset, request.ResponseCharacterEncoding)
    if err != nil {
        return err
    }
    // 回调函数 处理Response
    c.handleOnResponse(response)
    
    // 回调函数 HTML
    err = c.handleOnHTML(response)
    if err != nil {
        c.handleOnError(response, err, request, ctx)
    }
    // 回调函数XML
    err = c.handleOnXML(response)
    if err != nil {
        c.handleOnError(response, err, request, ctx)
    }
    // 回调函数 Scraped
    c.handleOnScraped(response)

    return err
}
```

### 回调函数

这里介绍按生命周期的顺序来介绍

#### 1. OnRequest

```javascript
// OnRequest registers a function. Function will be executed on every
// request made by the Collector
// 这里是注册回调函数到 requestCallbacks
func (c *Collector) OnRequest(f RequestCallback) {
   c.lock.Lock()
   if c.requestCallbacks == nil {
       c.requestCallbacks = make([]RequestCallback, 0, 4)
   }
   c.requestCallbacks = append(c.requestCallbacks, f)
   c.lock.Unlock()
}


// 在fetch中调用最早调用的
func (c *Collector) handleOnRequest(r *Request) {
   if c.debugger != nil {
       c.debugger.Event(createEvent("request", r.ID, c.ID, map[string]string{
           "url": r.URL.String(),
       }))
   }
   for _, f := range c.requestCallbacks {
       f(r)
   }
}
```

#### 2. OnResponse & handleOnResponse

```javascript
// OnResponse registers a function. Function will be executed on every response
func (c *Collector) OnResponse(f ResponseCallback) {
    c.lock.Lock()
    if c.responseCallbacks == nil {
        c.responseCallbacks = make([]ResponseCallback, 0, 4)
    }
    c.responseCallbacks = append(c.responseCallbacks, f)
    c.lock.Unlock()
}


func (c *Collector) handleOnResponse(r *Response) {
    if c.debugger != nil {
        c.debugger.Event(createEvent("response", r.Request.ID, c.ID, map[string]string{
            "url":    r.Request.URL.String(),
            "status": http.StatusText(r.StatusCode),
        }))
    }
    for _, f := range c.responseCallbacks {
        f(r)
    }
}
```

#### 3. OnHTML & handleOnHTML

```javascript
// OnHTML registers a function. Function will be executed on every HTML
// element matched by the GoQuery Selector parameter.
// GoQuery Selector is a selector used by https://github.com/PuerkitoBio/goquery
func (c *Collector) OnHTML(goquerySelector string, f HTMLCallback) {
    c.lock.Lock()
    if c.htmlCallbacks == nil {
        c.htmlCallbacks = make([]*htmlCallbackContainer, 0, 4)
    }
    c.htmlCallbacks = append(c.htmlCallbacks, &htmlCallbackContainer{
        Selector: goquerySelector,
        Function: f,
    })
    c.lock.Unlock()
}

// 这个解析html的逻辑比较多一些
func (c *Collector) handleOnHTML(resp *Response) error {
    if len(c.htmlCallbacks) == 0 || !strings.Contains(strings.ToLower(resp.Headers.Get("Content-Type")), "html") {
        return nil
    }
    doc, err := goquery.NewDocumentFromReader(bytes.NewBuffer(resp.Body))
    if err != nil {
        return err
    }
    if href, found := doc.Find("base[href]").Attr("href"); found {
        resp.Request.baseURL, _ = url.Parse(href)
    }
    for _, cc := range c.htmlCallbacks {
        i := 0
        doc.Find(cc.Selector).Each(func(_ int, s *goquery.Selection) {
            for _, n := range s.Nodes {
                e := NewHTMLElementFromSelectionNode(resp, s, n, i)
                i++
                if c.debugger != nil {
                    c.debugger.Event(createEvent("html", resp.Request.ID, c.ID, map[string]string{
                        "selector": cc.Selector,
                        "url":      resp.Request.URL.String(),
                    }))
                }
                cc.Function(e)
            }
        })
    }
    return nil
}
```

#### 4. OnXML & handleOnXML

```javascript
// OnXML registers a function. Function will be executed on every XML
// element matched by the xpath Query parameter.
// xpath Query is used by https://github.com/antchfx/xmlquery
func (c *Collector) OnXML(xpathQuery string, f XMLCallback) {
    c.lock.Lock()
    if c.xmlCallbacks == nil {
        c.xmlCallbacks = make([]*xmlCallbackContainer, 0, 4)
    }
    c.xmlCallbacks = append(c.xmlCallbacks, &xmlCallbackContainer{
        Query:    xpathQuery,
        Function: f,
    })
    c.lock.Unlock()
}



func (c *Collector) handleOnXML(resp *Response) error {
    if len(c.xmlCallbacks) == 0 {
        return nil
    }
    contentType := strings.ToLower(resp.Headers.Get("Content-Type"))
    isXMLFile := strings.HasSuffix(strings.ToLower(resp.Request.URL.Path), ".xml") || strings.HasSuffix(strings.ToLower(resp.Request.URL.Path), ".xml.gz")
    if !strings.Contains(contentType, "html") && (!strings.Contains(contentType, "xml") && !isXMLFile) {
        return nil
    }

    if strings.Contains(contentType, "html") {
        doc, err := htmlquery.Parse(bytes.NewBuffer(resp.Body))
        if err != nil {
            return err
        }
        if e := htmlquery.FindOne(doc, "//base"); e != nil {
            for _, a := range e.Attr {
                if a.Key == "href" {
                    resp.Request.baseURL, _ = url.Parse(a.Val)
                    break
                }
            }
        }

        for _, cc := range c.xmlCallbacks {
            for _, n := range htmlquery.Find(doc, cc.Query) {
                e := NewXMLElementFromHTMLNode(resp, n)
                if c.debugger != nil {
                    c.debugger.Event(createEvent("xml", resp.Request.ID, c.ID, map[string]string{
                        "selector": cc.Query,
                        "url":      resp.Request.URL.String(),
                    }))
                }
                cc.Function(e)
            }
        }
    } else if strings.Contains(contentType, "xml") || isXMLFile {
        doc, err := xmlquery.Parse(bytes.NewBuffer(resp.Body))
        if err != nil {
            return err
        }

        for _, cc := range c.xmlCallbacks {
            xmlquery.FindEach(doc, cc.Query, func(i int, n *xmlquery.Node) {
                e := NewXMLElementFromXMLNode(resp, n)
                if c.debugger != nil {
                    c.debugger.Event(createEvent("xml", resp.Request.ID, c.ID, map[string]string{
                        "selector": cc.Query,
                        "url":      resp.Request.URL.String(),
                    }))
                }
                cc.Function(e)
            })
        }
    }
    return nil
}
```

#### 5. OnError & handleOnError

这个会多次调用， 如果 `err != nil情况下调用比较多`， 爬虫异常的情况下，会调用

```javascript
// OnError registers a function. Function will be executed if an error
// occurs during the HTTP request.
func (c *Collector) OnError(f ErrorCallback) {
    c.lock.Lock()
    if c.errorCallbacks == nil {
        c.errorCallbacks = make([]ErrorCallback, 0, 4)
    }
    c.errorCallbacks = append(c.errorCallbacks, f)
    c.lock.Unlock()
}


func (c *Collector) handleOnError(response *Response, err error, request *Request, ctx *Context) error {
    if err == nil && (c.ParseHTTPErrorResponse || response.StatusCode < 203) {
        return nil
    }
    if err == nil && response.StatusCode >= 203 {
        err = errors.New(http.StatusText(response.StatusCode))
    }
    if response == nil {
        response = &Response{
            Request: request,
            Ctx:     ctx,
        }
    }
    if c.debugger != nil {
        c.debugger.Event(createEvent("error", request.ID, c.ID, map[string]string{
            "url":    request.URL.String(),
            "status": http.StatusText(response.StatusCode),
        }))
    }
    if response.Request == nil {
        response.Request = request
    }
    if response.Ctx == nil {
        response.Ctx = request.Ctx
    }
    for _, f := range c.errorCallbacks {
        f(response, err)
    }
    return err
}
```

#### 6. OnScraped & handleOnScraped

最后一步的回调函数处理

```javascript
// OnScraped registers a function. Function will be executed after
// OnHTML, as a final part of the scraping.
func (c *Collector) OnScraped(f ScrapedCallback) {
    c.lock.Lock()
    if c.scrapedCallbacks == nil {
        c.scrapedCallbacks = make([]ScrapedCallback, 0, 4)
    }
    c.scrapedCallbacks = append(c.scrapedCallbacks, f)
    c.lock.Unlock()
}

func (c *Collector) handleOnScraped(r *Response) {
    if c.debugger != nil {
        c.debugger.Event(createEvent("scraped", r.Request.ID, c.ID, map[string]string{
            "url": r.Request.URL.String(),
        }))
    }
    for _, f := range c.scrapedCallbacks {
        f(r)
    }
}
```

注册回调函数的method还有几个没有列出来，感兴趣的，自己看一下，

```javascript
    // On every a element which has href attribute call callback
    c.OnHTML("a[href]", func(e *colly.HTMLElement) {
        link := e.Attr("href")
        // Print link
        fmt.Printf("Link found: %q -> %s\n", e.Text, link)
        // Visit link found on page
        // Only those links are visited which are in AllowedDomains
        c.Visit(e.Request.AbsoluteURL(link))
    })

    // Before making a request print "Visiting ..."
    c.OnRequest(func(r *colly.Request) {
        fmt.Println("Visiting", r.URL.String())
    })
```

一般文档解析放在html, xml 中



![](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/colly.png)

### 页面跳转爬取

1. 在`html`,`xml`，解析出来以后，构建新的请求，我们看一下，相同页面

```javascript
   // On every a element which has href attribute call callback
   c.OnHTML("a[href]", func(e *colly.HTMLElement) {
       // If attribute class is this long string return from callback
       // As this a is irrelevant
       if e.Attr("class") == "Button_1qxkboh-o_O-primary_cv02ee-o_O-md_28awn8-o_O-primaryLink_109aggg" {
           return
       }
       link := e.Attr("href")
       // If link start with browse or includes either signup or login return from callback
       if !strings.HasPrefix(link, "/browse") || strings.Index(link, "=signup") > -1 || strings.Index(link, "=login") > -1 {
           return
       }
       // start scaping the page under the link found
       e.Request.Visit(link)
   })
```

上面是 HTML的回调函数，解析页面，获取了`url`,使用 `e.Request.Visit(link)`, 其实就是 `e.Request.collector.Visit(link)`

```javascript
func (c *Collector) fetch(u, method string, depth int, requestData io.Reader, ctx *Context, hdr http.Header, req *http.Request) error {
    defer c.wg.Done()
    if ctx == nil {
        ctx = NewContext()
    }
    request := &Request{
        URL:       req.URL,
        Headers:   &req.Header,
        Ctx:       ctx,
        Depth:     depth,
        Method:    method,
        Body:      requestData,
        collector: c, // 这个上面有介绍
        ID:        atomic.AddUint32(&c.requestCount, 1),
    }
    ....
    }}


// Visit continues Collector's collecting job by creating a
// request and preserves the Context of the previous request.
// 访问也会调用之前提供的回调
func (r *Request) Visit(URL string) error {
    return r.collector.scrape(r.AbsoluteURL(URL), "GET", r.Depth+1, nil, r.Ctx, nil, true)
}
```

这种方法在实际开发中经常会用到。

2.子页面的处理逻辑

 colly中主要是以`Collector`为中心， 然后各种回调函数进行处理，子页面需要不同的回调函数，所以就需要新的 `Collector`

```go
// Instantiate default collector
    c := colly.NewCollector(
        // Visit only domains: coursera.org, www.coursera.org
        colly.AllowedDomains("coursera.org", "www.coursera.org"),

        // Cache responses to prevent multiple download of pages
        // even if the collector is restarted
        colly.CacheDir("./coursera_cache"),
    )

    // Create another collector to scrape course details
    detailCollector := c.Clone()

    // Before making a request print "Visiting ..."
    c.OnRequest(func(r *colly.Request) {
        log.Println("visiting", r.URL.String())
    })

    // On every a HTML element which has name attribute call callback
    c.OnHTML(`a[name]`, func(e *colly.HTMLElement) {
        // Activate detailCollector if the link contains "coursera.org/learn"
        courseURL := e.Request.AbsoluteURL(e.Attr("href"))
        if strings.Index(courseURL, "coursera.org/learn") != -1 {
           // 子页面或其他页面
            detailCollector.Visit(courseURL)
        }
    })
```

### 三、crawlab

https://docs.crawlab.cn/zh/Installation/Docker.html



https://www.sofineday.com/gocolly.html#_2-%E5%AF%B9%E6%8E%A5ip%E4%BB%A3%E7%90%86%E6%9C%8D%E5%8A%A1

### 四、搭建IP代理服务

[示例](https://www.sofineday.com/gocolly.html)

### 五。分布式爬取

