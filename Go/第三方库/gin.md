# Gin

[参考地址](https://geektutu.com/post/quick-go-gin.html)、[参考地址](https://heary.cn/posts/%E4%BB%8E%E6%BA%90%E7%A0%81%E7%90%86%E8%A7%A3Gin%E6%A1%86%E6%9E%B6%E5%8E%9F%E7%90%86/)、[参考地址](https://gin-gonic.com/zh-cn/docs/introduction/)

## Gin 简介

Gin 是使用 Go/golang 语言实现的 HTTP Web 框架。接口简洁，性能极高。截止 *1.4.0* 版本，包含测试代码，仅14K，其中测试代码 9K 左右，也就是说框架源码仅 5K 左右。

**Gin 特性**

- **快速**：路由不使用反射，基于Radix树，内存占用少。基于 [Radix 树](https://toutiao.io/posts/o9anj0/preview)的路由，小内存占用。没有反射。可预测的 API 性能。
- **中间件**：HTTP请求，可先经过一系列中间件处理，例如：Logger，Authorization，GZIP等。这个特性和 NodeJs 的 `Koa` 框架很像。中间件机制也极大地提高了框架的可扩展性。
- **异常处理**：服务始终可用，不会宕机。Gin 可以捕获 panic，并恢复。而且有极为便利的机制处理HTTP请求过程中发生的错误。
- **JSON**：Gin可以**解析并验证请求的JSON**。这个特性对`Restful API`的开发尤其有用。
- **路由分组**：例如将需要授权和不需要授权的API分组，不同版本的API分组。而且分组可嵌套，且性能不受影响。
- **错误管理**：Gin **提供了一种方便的方法来收集 HTTP 请求期间发生的所有错误**。最终，中间件可以将它们写入日志文件，数据库并通过网络发送。
- **渲染内置**：原生支持JSON，XML和HTML的渲染。
- **可扩展性**：新建一个中间件非常简单

## 快速入门

下载安装

```
go get -u github.com/gin-gonic/gin
```



在一个空文件夹里新建文件`main.go`。

```
// geektutu.com
// main.go
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()
	r.GET("/", func(c *gin.Context) {
		c.String(200, "Hello, Geektutu")
	})
	r.Run() // listen and serve on 0.0.0.0:8080
}
```

1. 首先，我们使用了`gin.Default()`**生成了一个实例**，这个实例即 WSGI 应用程序。
2. 接下来，我们使用`r.Get("/", ...)`**声明了一个路由**，告诉 Gin 什么样的URL 能触发传入的函数，这个函数返回我们想要显示在用户浏览器中的信息。
3. 最后用 `r.Run()`函数来**让应用运行**在本地服务器上，默认监听端口是 _8080_，可以传入参数设置端口，例如`r.Run(":9999")`即运行在 _9999_端口。

- 运行

```
$ go run main.go
[GIN-debug] GET    /                         --> main.main.func1 (3 handlers)
[GIN-debug] Environment variable PORT is undefined. Using port :8080 by default
[GIN-debug] Listening and serving HTTP on :8080
```

- 浏览器访问 `http://localhost:8080`

![Hello Gin](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/hello_gin.jpg)

### 路由(Route)

路由方法有 **GET, POST, PUT, PATCH, DELETE** 和 **OPTIONS**，还有**Any**，可匹配以上任意类型的请求。

### 获取参数

#### 无参数

```
// 无参数
r.GET("/", func(c *gin.Context) {
	c.String(http.StatusOK, "Who are you?")
})
$ curl http://localhost:9999/
Who are you?
curl`参数可参考`https://man.linuxde.net/curl
```

#### 解析路径参数

有时候我们需要动态的路由，如 `/user/:name`，通过调用不同的 url 来传入不同的 name。`/user/:name/*role`，`*` 代表可选。

```
// 匹配 /user/geektutu
r.GET("/user/:name/*action", func(c *gin.Context) {
    name := c.Param("name")
    action := c.Param("action")
    //截取/
    action = strings.Trim(action, "/")
    c.String(http.StatusOK, name+" is "+action)
})
$ curl http://localhost:9999/user/geektutu
Hello geektutu
```

`c.Param(...)` 函数解析动态路由中

#### 获取Query参数

```
// 匹配users?name=xxx&role=xxx，role可选
r.GET("/users", func(c *gin.Context) {
	name := c.Query("name")
	role := c.DefaultQuery("role", "teacher")
	c.String(http.StatusOK, "%s is a %s", name, role)
})
$ curl "http://localhost:9999/users?name=Tom&role=student"
Tom is a student
```

`c.Query()` 获取由Get方法传输的参数，`c.DefaultQuery(...,...)` 为参数设置一个默认值，`query` 获取到的是在URL中

#### 获取POST参数

- 表单传输为post请求，http常见的传输格式为四种：
  - application/json
  - application/x-www-form-urlencoded
  - application/xml
  - multipart/form-data
- 表单参数可以通过PostForm()方法获取，该方法默认解析的是x-www-form-urlencoded或from-data格式的参数

```
// POST
r.POST("/form", func(c *gin.Context) {
	username := c.PostForm("username")
	password := c.DefaultPostForm("password", "000000") // 可设置默认值

	c.JSON(http.StatusOK, gin.H{
		"username": username,
		"password": password,
	})
})
$ curl http://localhost:9999/form  -X POST -d 'username=geektutu&password=1234'
{"password":"1234","username":"geektutu"}
```

典型的如 `POST` 提交的数据，无论是 `multipart/form-data`格式还是`application/x-www-form-urlencoded`格式，都可以使用 `c.PostForm`获取到参数。该方法始终返回一个 `string` 类型的数据。

#### Query和POST混合参数

```
// GET 和 POST 混合
r.POST("/posts", func(c *gin.Context) {
	id := c.Query("id")
	page := c.DefaultQuery("page", "0")
	username := c.PostForm("username")
	password := c.DefaultPostForm("username", "000000") // 可设置默认值

	c.JSON(http.StatusOK, gin.H{
		"id":       id,
		"page":     page,
		"username": username,
		"password": password,
	})
})
$ curl "http://localhost:9999/posts?id=9876&page=7"  -X POST -d 'username=geektutu&password=1234'
{"id":"9876","page":"7","password":"1234","username":"geektutu"}
```

#### Map参数(字典参数)

```
r.POST("/post", func(c *gin.Context) {
	ids := c.QueryMap("ids")
	names := c.PostFormMap("names")

	c.JSON(http.StatusOK, gin.H{
		"ids":   ids,
		"names": names,
	})
})
$ curl -g "http://localhost:9999/post?ids[Jack]=001&ids[Tom]=002" -X POST -d 'names[a]=Sam&names[b]=David'
{"ids":{"Jack":"001","Tom":"002"},"names":{"a":"Sam","b":"David"}}
```

### 重定向(Redirect)

```
r.GET("/redirect", func(c *gin.Context) {
    c.Redirect(http.StatusMovedPermanently, "/index")
})

r.GET("/goindex", func(c *gin.Context) {
	c.Request.URL.Path = "/"
	r.HandleContext(c)
})
$ curl -i http://localhost:9999/redirect
HTTP/1.1 301 Moved Permanently
Content-Type: text/html; charset=utf-8
Location: /
Date: Thu, 08 Aug 2019 17:22:14 GMT
Content-Length: 36

<a href="/">Moved Permanently</a>.

$ curl "http://localhost:9999/goindex"
Who are you?
```

两种重定向方式

### 分组路由(Grouping Routes)

如果有一组路由，前缀都是`/api/v1`开头，是否每个路由都需要加上`/api/v1`这个前缀呢？答案是不需要，分组路由可以解决这个问题。利用分组路由还可以更好地实现权限控制，例如将需要登录鉴权的路由放到同一分组中去，简化权限控制。

```
// group routes 分组路由
defaultHandler := func(c *gin.Context) {
	c.JSON(http.StatusOK, gin.H{
		"path": c.FullPath(),
	})
}
// {} 是书写规范
// group: v1
v1 := r.Group("/v1")
{
	v1.GET("/posts", defaultHandler)
	v1.GET("/series", defaultHandler)
}
// group: v2
v2 := r.Group("/v2")
{
	v2.GET("/posts", defaultHandler)
	v2.GET("/series", defaultHandler)
}
$ curl http://localhost:9999/v1/posts
{"path":"/v1/posts"}
$ curl http://localhost:9999/v2/posts
{"path":"/v2/posts"}
```

### 路由拆分与注册

#### 基本的路由注册

下面最基础的gin路由注册方式，适用于路由条目比较少的简单项目或者项目demo。

```go
package main

import (
    "net/http"

    "github.com/gin-gonic/gin"
)

func helloHandler(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{
        "message": "Hello www.topgoer.com!",
    })
}

func main() {
    r := gin.Default()
    r.GET("/topgoer", helloHandler)
    if err := r.Run(); err != nil {
        fmt.Println("startup service failed, err:%v\n", err)
    }
}
```

#### 路由拆分成单独文件或包

当项目的规模增大后就不太适合继续在项目的main.go文件中去实现路由注册相关逻辑了，我们会倾向于把路由部分的代码都拆分出来，形成一个单独的文件或包：

我们在routers.go文件中定义并注册路由信息：

```go
package main

import (
    "net/http"

    "github.com/gin-gonic/gin"
)

func helloHandler(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{
        "message": "Hello www.topgoer.com!",
    })
}

func setupRouter() *gin.Engine {
    r := gin.Default()
    r.GET("/topgoer", helloHandler)
    return r
}
```

此时main.go中调用上面定义好的setupRouter函数：

```go
func main() {
    r := setupRouter()
    if err := r.Run(); err != nil {
        fmt.Println("startup service failed, err:%v\n", err)
    }
}
```

此时的目录结构：

```go
gin_demo
├── go.mod
├── go.sum
├── main.go
└── routers.go
```

把路由部分的代码单独拆分成包的话也是可以的，拆分后的目录结构如下：

```
gin_demo
├── go.mod
├── go.sum
├── main.go
└── routers
    └── routers.go
```

routers/routers.go需要注意此时setupRouter需要改成首字母大写：

```go
package routers

import (
    "net/http"

    "github.com/gin-gonic/gin"
)

func helloHandler(c *gin.Context) {
    c.JSON(http.StatusOK, gin.H{
        "message": "Hello www.topgoer.com",
    })
}

// SetupRouter 配置路由信息
func SetupRouter() *gin.Engine {
    r := gin.Default()
    r.GET("/topgoer", helloHandler)
    return r
}
```

main.go文件内容如下：

```go
package main

import (
    "fmt"
    "gin_demo/routers"
)

func main() {
    r := routers.SetupRouter()
    if err := r.Run(); err != nil {
        fmt.Println("startup service failed, err:%v\n", err)
    }
}
```

#### 路由拆分成多个文件

当我们的业务规模继续膨胀，单独的一个routers文件或包已经满足不了我们的需求了，

```go
func SetupRouter() *gin.Engine {
    r := gin.Default()
    r.GET("/topgoer", helloHandler)
    r.GET("/xx1", xxHandler1)
    ...
    r.GET("/xx30", xxHandler30)
    return r
}
```

因为我们把所有的路由注册都写在一个SetupRouter函数中的话就会太复杂了。

我们可以分开定义多个路由文件，例如：

```
gin_demo
├── go.mod
├── go.sum
├── main.go
└── routers
    ├── blog.go
    └── shop.go
```

routers/shop.go中添加一个LoadShop的函数，将shop相关的路由注册到指定的路由器：

```
func LoadShop(e *gin.Engine)  {
    e.GET("/hello", helloHandler)
    e.GET("/goods", goodsHandler)
    e.GET("/checkout", checkoutHandler)
    ...
}
```

routers/blog.go中添加一个LoadBlog的函数，将blog相关的路由注册到指定的路由器：

```go
func LoadBlog(e *gin.Engine) {
    e.GET("/post", postHandler)
    e.GET("/comment", commentHandler)
    ...
}
```

在main函数中实现最终的注册逻辑如下：

```go
func main() {
    r := gin.Default()
    routers.LoadBlog(r)
    routers.LoadShop(r)
    if err := r.Run(); err != nil {
        fmt.Println("startup service failed, err:%v\n", err)
    }
}
```

#### 路由拆分到不同的APP

有时候项目规模实在太大，那么我们就更倾向于把业务拆分的更详细一些，例如把不同的业务代码拆分成不同的APP。

因此我们在项目目录下单独定义一个app目录，用来存放我们不同业务线的代码文件，这样就很容易进行横向扩展。大致目录结构如下：

```
gin_demo
├── app
│   ├── blog
│   │   ├── handler.go
│   │   └── router.go
│   └── shop
│       ├── handler.go
│       └── router.go
├── go.mod
├── go.sum
├── main.go
└── routers
    └── routers.go
```

其中app/blog/router.go用来定义post相关路由信息，具体内容如下：

```go
func Routers(e *gin.Engine) {
    e.GET("/post", postHandler)
    e.GET("/comment", commentHandler)
}
```

app/shop/router.go用来定义shop相关路由信息，具体内容如下：

```go
func Routers(e *gin.Engine) {
    e.GET("/goods", goodsHandler)
    e.GET("/checkout", checkoutHandler)
}
```

routers/routers.go中根据需要定义Include函数用来注册子app中定义的路由，Init函数用来进行路由的初始化操作：

```go
type Option func(*gin.Engine)

var options = []Option{}

// 注册app的路由配置
func Include(opts ...Option) {
    options = append(options, opts...)
}

// 初始化
func Init() *gin.Engine {
    r := gin.New()
    for _, opt := range options {
        opt(r)
    }
    return r
}
```

main.go中按如下方式先注册子app中的路由，然后再进行路由的初始化：

```go
func main() {
    // 加载多个APP的路由配置
    routers.Include(shop.Routers, blog.Routers)
    // 初始化路由
    r := routers.Init()
    if err := r.Run(); err != nil {
        fmt.Println("startup service failed, err:%v\n", err)
    }
}
```

### 上传文件

post 传输文件时使用`Content-Type:multipart/form-data`

Gin 对接受用户上传的文件做了友好的处理，在 Handler 中可以很简单的实现文件的接收。

要注意的是，上传文件的大小有限制，通常是 `<32MB`，你可以使用 `router.MaxMultipartMemory`更改它。

#### 单个文件

```
r.POST("/upload1", func(c *gin.Context) {
	file, _ := c.FormFile("file")
	// c.SaveUploadedFile(file, dst)
	c.String(http.StatusOK, "%s uploaded!", file.Filename)
})
```

`c.FormFile()` 接收文件，解析post 传输的消息体中为file字段的内容  .

#### 多个文件

```
r.POST("/upload2", func(c *gin.Context) {
	// Multipart form
	form, _ := c.MultipartForm()
	files := form.File["upload[]"]

	for _, file := range files {
		log.Println(file.Filename)
		// c.SaveUploadedFile(file, dst)
	}
	c.String(http.StatusOK, "%d files uploaded!", len(files))
})
```

```
# 单一文件上传
$ curl -X POST http://localhost:8080/upload1 \
  -F "file=@/Users/appleboy/test.zip" \
  -H "Content-Type: multipart/form-data"

# 多文件上传
$ curl -X POST http://localhost:8080/upload2 \
  -F "upload[]=@/Users/appleboy/test1.zip" \
  -F "upload[]=@/Users/appleboy/test2.zip" \
  -H "Content-Type: multipart/form-data"
```

### 其他格式的数据

一些复杂的场景下，如用户直接 `POST`一段 `json`字符串到应用中，我们需要获取原始数据，这时需要用 `c.GetRawData`来获取原始字节。

```go
// 省略的代码 ...

func main() {
    router := gin.Default()

    router.POST("/post", func(c *gin.Context) {
        // 获取原始字节
        d, err := c.GetRawData()
        if err!=nil {
            log.Fatalln(err)
        }
        log.Println(string(d))
        c.String(200, "ok")
    })
    router.Run(":8080")
}
```

`curl` 请求示例：

```bash
$ curl -v -X POST \
  http://localhost:8080/post \
  -H 'content-type: application/json' \
  -d '{ "user": "manu" }'
```

### HTML模板(Template)

```
type student struct {
	Name string
	Age  int8
}

r.LoadHTMLGlob("templates/*")

stu1 := &student{Name: "Geektutu", Age: 20}
stu2 := &student{Name: "Jack", Age: 22}
r.GET("/arr", func(c *gin.Context) {
	c.HTML(http.StatusOK, "arr.tmpl", gin.H{
		"title":  "Gin",
		"stuArr": [2]*student{stu1, stu2},
	})
})
<!-- templates/arr.tmpl -->
<html>
<body>
    <p>hello, {{.title}}</p>
    {{range $index, $ele := .stuArr }}
    <p>{{ $index }}: {{ $ele.Name }} is {{ $ele.Age }} years old</p>
    {{ end }}
</body>
</html>
$ curl http://localhost:9999/arr

<html>
<body>
    <p>hello, Gin</p>
    <p>0: Geektutu is 20 years old</p>
    <p>1: Jack is 22 years old</p>
</body>
</html>
```

- Gin默认使用模板Go语言标准库的模板`text/template`和`html/template`，语法与标准库一致，支持各种复杂场景的渲染。
- 参考官方文档[text/template](https://golang.org/pkg/text/template/)，[html/template](https://golang.org/pkg/html/template/)

### 中间件(Middleware)

```
// 作用于全局
r.Use(gin.Logger())
r.Use(gin.Recovery())

// 作用于单个路由
r.GET("/benchmark", MyBenchLogger(), benchEndpoint)

// 作用于某个组
authorized := r.Group("/")
authorized.Use(AuthRequired())
{
	authorized.POST("/login", loginEndpoint)
	authorized.POST("/submit", submitEndpoint)
}
```

如何自定义中间件呢？

```
func Logger() gin.HandlerFunc {
	return func(c *gin.Context) {
		t := time.Now()
		// 给Context实例设置一个值
		c.Set("geektutu", "1111")
		// 请求前
		c.Next()
		// 请求后
		latency := time.Since(t)
		log.Print(latency)
	}
}
```

### 数据绑定

Gin 提供了非常方便的数据绑定功能，可以将用户传来的参数自动跟我们定义的结构体绑定在一起。

#### 绑定 Url 查询参数（Only Bind Query String）

使用 `c.ShouldBindQuery`方法，可以自动绑定 Url 查询参数到 `struct`.

```go
package main

import (
    "log"
    "github.com/gin-gonic/gin"
)

// 定义一个 Person 结构体，用来绑定 url query
type Person struct {
    Name    string `form:"name"` // 使用成员变量标签定义对应的参数名
    Address string `form:"address"`
}

func main() {
    route := gin.Default()
    route.Any("/testing", startPage)
    route.Run(":8085")
}

func startPage(c *gin.Context) {
    var person Person
    // 将 url 查询参数和person绑定在一起
    if c.ShouldBindQuery(&person) == nil {
        log.Println("====== Only Bind By Query String ======")
        log.Println(person.Name)
        log.Println(person.Address)
    }
    c.String(200, "Success")
}
```

#### 绑定url查询参数和POST参数

使用 `c.ShouldBind`方法，可以将参数自动绑定到 `struct`.该方法是**会检查 Url 查询字符串和 POST 的数据**，而且会根据 `content-type`类型，优先匹配`JSON`或者 `XML`,之后才是 `Form`. 有关详情查阅 [这里](https://github.com/gin-gonic/gin/blob/master/binding/binding.go#L48)

```go
package main

import "log"
import "github.com/gin-gonic/gin"
import "time"

// 定义一个 Person 结构体，用来绑定数据
type Person struct {
    Name     string    `form:"name"`
    Address  string    `form:"address"`
    Birthday time.Time `form:"birthday" time_format:"2006-01-02" time_utc:"1"`
}

func main() {
    route := gin.Default()
    route.GET("/testing", startPage)
    route.Run(":8085")
}

func startPage(c *gin.Context) {
    var person Person
    // 绑定到 person
    if c.ShouldBind(&person) == nil {
        log.Println(person.Name)
        log.Println(person.Address)
        log.Println(person.Birthday)
    }

    c.String(200, "Success")
}
```

#### 其他数据绑定

Gin 提供的数据绑定功能很强大，建议你仔细查阅官方文档，了解 `gin.Context`的 `Bind*`系列方法。这里就不再一一详述。

### 数据验证

永远不要相信用户任何的输入。**对于获取的外来数据，我们要做的第一步就是校验和转换**。对于这类基础并且常用的功能， Gin 很贴心的帮我们提供了数据校验的方法，省去我们重复判断的烦恼。

Gin 的数据验证是和数据绑定结合在一起的。只需要在数据绑定的结构体成员变量的标签添加`binding`规则即可。详细的规则查阅 [这里](https://godoc.org/gopkg.in/go-playground/validator.v8#hdr-Baked_In_Validators_and_Tags)。

来看官方给出的一段代码：

```go
// 省略的代码 ...

// 定义的 Login 结构体
// 该 struct 可以绑定在 Form 和 JSON 中
// binding:"required" 意思是必要参数。如果未提供，Bind 会返回 error
type Login struct {
    User     string `form:"user" json:"user" binding:"required"`
    Password string `form:"password" json:"password" binding:"required"`
}

func main() {
    router := gin.Default()

    // POST 到这个路由一段 JSON, 如 ({"user": "manu", "password": "123"})
    router.POST("/loginJSON", func(c *gin.Context) {
        var json Login
        // 验证数据并绑定
        if err := c.ShouldBindJSON(&json); err == nil {
            if json.User == "manu" && json.Password == "123" {
                c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
            } else {
                c.JSON(http.StatusUnauthorized, gin.H{"status": "unauthorized"})
            }
        } else {
            c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        }
    })

    // POST 到这个路由一个 Form 表单 (user=manu&password=123)
    router.POST("/loginForm", func(c *gin.Context) {
        var form Login
        // 验证数据并绑定
        if err := c.ShouldBind(&form); err == nil {
            if form.User == "manu" && form.Password == "123" {
                c.JSON(http.StatusOK, gin.H{"status": "you are logged in"})
            } else {
                c.JSON(http.StatusUnauthorized, gin.H{"status": "unauthorized"})
            }
        } else {
            c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
        }
    })

    router.Run(":8080")
}
```

除了绑定验证之外，你还可以**注册自定义的验证器**。

我们来看这段完整的代码：

```go
package main

import (
    "net/http"
    "reflect"
    "time"
    "github.com/gin-gonic/gin"
    "github.com/gin-gonic/gin/binding"
    "gopkg.in/go-playground/validator.v8"
)

// 定义的 Booking 结构体
// 注意成员变量CheckIn的标签 binding:"required,bookabledate"
// bookabledate 即下面自定义的验证函数
// 成员变量CheckOut的标签 binding:"required,gtfield=CheckIn"
// gtfield 是一个默认规则，意思是要大于某个字段的值
type Booking struct {
    CheckIn  time.Time `form:"check_in" binding:"required,bookabledate" time_format:"2006-01-02"`
    CheckOut time.Time `form:"check_out" binding:"required,gtfield=CheckIn" time_format:"2006-01-02"`
}

// 定义一个验证方法，用来验证时间是否合法
// 验证方法返回值应该是个布尔值
func bookableDate(
    v *validator.Validate, topStruct reflect.Value, currentStructOrField reflect.Value,
    field reflect.Value, fieldType reflect.Type, fieldKind reflect.Kind, param string,
) bool {
    if date, ok := field.Interface().(time.Time); ok {
        today := time.Now()
        if today.Year() > date.Year() || today.YearDay() > date.YearDay() {
            return false
        }
    }
    return true
}

func main() {
    route := gin.Default()
    // 注册一个自定义验证方法 bookabledate
    binding.Validator.RegisterValidation("bookabledate", bookableDate)
    route.GET("/bookable", getBookable)
    route.Run(":8085")
}

func getBookable(c *gin.Context) {
    var b Booking
    // 验证数据并绑定
    if err := c.ShouldBindWith(&b, binding.Query); err == nil {
        c.JSON(http.StatusOK, gin.H{"message": "Booking dates are valid!"})
    } else {
        c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
    }
}
```

Gin 提供的验证灰常灰常强大，可以帮我们很好的处理数据验证和省掉各种 `if else`的判断，建议各位使用 Gin 的道友一定要把这个东东吃透。

### 输出响应

**Web 应用的目标之一就是输出响应**。Gin 为我们提供了多种常见格式的输出，包括 `HTML`, `String`，`JSON`， `XML`， `YAML`。

#### String

```go
// 省略的代码 ...

func Handler(c *gin.Context) {
    // 使用 String 方法即可
    c.String(200, "Success")
}

// 省略的代码 ...
```

#### JSON、 XML、 YAML

Gin 输出这三种格式非常方便，直接使用对用方法并赋值一个结构体给它就行了。

你还可以使用`gin.H`。**`gin.H` 是一个很巧妙的设计，你可以像`javascript`定义`json`一样，直接一层层写键值对，只需要在每一层加上 `gin.H`即可**。看代码：

```go
// 省略的代码 ...

func main() {
    r := gin.Default()

    // gin.H 本质是 map[string]interface{}
    r.GET("/someJSON", func(c *gin.Context) {
        // 会输出头格式为 application/json; charset=UTF-8 的 json 字符串
        c.JSON(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
    })

    r.GET("/moreJSON", func(c *gin.Context) {
        // 直接使用结构体定义
        var msg struct {
            Name    string `json:"user"`
            Message string
            Number  int
        }
        msg.Name = "Lena"
        msg.Message = "hey"
        msg.Number = 123
        // 会输出  {"user": "Lena", "Message": "hey", "Number": 123}
        c.JSON(http.StatusOK, msg)
    })

    r.GET("/someXML", func(c *gin.Context) {
        // 会输出头格式为 text/xml; charset=UTF-8 的 xml 字符串
        c.XML(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
    })

    r.GET("/someYAML", func(c *gin.Context) {
        // 会输出头格式为 text/yaml; charset=UTF-8 的 yaml 字符串
        c.YAML(http.StatusOK, gin.H{"message": "hey", "status": http.StatusOK})
    })

    r.Run(":8080")
}

// 省略的代码 ...
```

#### HTML

Gin 使用 Html 模板就是官方标准包`html/template`, 模板的用法没什么太多要说明的。这里给大家说一下如何在 gin 中输出 `html`。

由于 Gin 并没有强制的文件夹命名规范，所以我们必须要告诉 Gin 我们的静态资源（如图片、Css、JS 脚本等）和我们的html 模板放在了哪里，看代码：

```go
package main

import "github.com/gin-gonic/gin"

func main() {
    engine := gin.Default()
    engine.GET("/html-test", startPage)
    // 注册一个目录，gin 会把该目录当成一个静态的资源目录
    // 该目录下的资源看可以按照路径访问
    // 如 static 目录下有图片路径 index/logo.png , 你可以通过 GET /static/index/logo.png 访问到
    engine.Static("/static", "./static")
    // 注册一个路径，gin 加载模板的时候会从该目录查找
    // 参数是一个匹配字符，如 views/*/* 的意思是 模板目录有两层
    // gin 在启动时会自动把该目录的文件编译一次缓存，不用担心效率问题
    engine.LoadHTMLGlob("views/*/*")
  
    route.Run(":9205")
}

func startPage(c *gin.Context) {
    // 输出 html
    // 三个参数，200 是http状态码
    // "shop/search" 要加载的模板路径，对应目录路径 views/shop/search.html
    // gin.H{"keywords":"macbook pro"} 是模板变量
    c.HTML(200, "shop/search", gin.H{"keywords":"macbook pro"})
}
```

编译启动后访问 `/html-test`, 就可以看到编译后的模板字符串输出.

### 同步异步

goroutine机制可以方便地实现异步处理。另外，在启动新的goroutine时，**不应该使用原始上下文，必须使用它的只读副本**

```
package main

import (
    "log"
    "time"

    "github.com/gin-gonic/gin"
)

func main() {
    // 1.创建路由
    // 默认使用了2个中间件Logger(), Recovery()
    r := gin.Default()
    // 1.异步
    r.GET("/long_async", func(c *gin.Context) {
        // 需要搞一个副本
        copyContext := c.Copy()
        // 异步处理
        go func() {
            time.Sleep(3 * time.Second)
            log.Println("异步执行：" + copyContext.Request.URL.Path)
        }()
    })
    // 2.同步
    r.GET("/long_sync", func(c *gin.Context) {
        time.Sleep(3 * time.Second)
        log.Println("同步执行：" + c.Request.URL.Path)
    })

    r.Run(":8000")
}
```

### 热加载调试 Hot Reload

Python 的 `Flask`框架，有 *debug* 模式，启动时传入 *debug=True* 就可以热加载(Hot Reload, Live Reload)了。即更改源码，保存后，自动触发更新，浏览器上刷新即可。免去了杀进程、重新启动之苦。

Gin 原生不支持，但有很多额外的库可以支持。例如

- github.com/codegangsta/gin
- github.com/pilu/fresh

这次，我们采用 *github.com/pilu/fresh* 。

```
go get -v -u github.com/pilu/fresh
```

安装好后，只需要将`go run main.go`命令换成`fresh`即可。每次更改源文件，代码将自动重新编译(Auto Compile)。

### [中间件推荐](https://topgoer.com/gin%E6%A1%86%E6%9E%B6/gin%E4%B8%AD%E9%97%B4%E4%BB%B6/%E4%B8%AD%E9%97%B4%E4%BB%B6%E6%8E%A8%E8%8D%90.html)

### 会话控制

#### Cookie的使用

- 测试服务端发送cookie给客户端，客户端请求时携带cookie

```go
package main

import (
   "github.com/gin-gonic/gin"
   "fmt"
)

func main() {
   // 1.创建路由
   // 默认使用了2个中间件Logger(), Recovery()
   r := gin.Default()
   // 服务端要给客户端cookie
   r.GET("cookie", func(c *gin.Context) {
      // 获取客户端是否携带cookie
      cookie, err := c.Cookie("key_cookie")
      if err != nil {
         cookie = "NotSet"
         // 给客户端设置cookie
         //  maxAge int, 单位为秒
         // path,cookie所在目录
         // domain string,域名
         //   secure 是否智能通过https访问
         // httpOnly bool  是否允许别人通过js获取自己的cookie
         c.SetCookie("key_cookie", "value_cookie", 60, "/",
            "localhost", false, true)
      }
      fmt.Printf("cookie的值是： %s\n", cookie)
   })
   r.Run(":8000")
}
```

####  Sessions

gorilla/sessions为自定义session后端提供cookie和文件系统session以及基础结构。

主要功能是：

- 简单的API：将其用作设置签名（以及可选的加密）cookie的简便方法。
- 内置的后端可将session存储在cookie或文件系统中。
- Flash消息：一直持续读取的session值。
- 切换session持久性（又称“记住我”）和设置其他属性的便捷方法。
- 旋转身份验证和加密密钥的机制。
- 每个请求有多个session，即使使用不同的后端也是如此。
- 自定义session后端的接口和基础结构：可以使用通用API检索并批量保存来自不同商店的session。

代码：

```go
package main

import (
    "fmt"
    "net/http"

    "github.com/gorilla/sessions"
)

// 初始化一个cookie存储对象
// something-very-secret应该是一个你自己的密匙，只要不被别人知道就行
var store = sessions.NewCookieStore([]byte("something-very-secret"))

func main() {
    http.HandleFunc("/save", SaveSession)
    http.HandleFunc("/get", GetSession)
    err := http.ListenAndServe(":8080", nil)
    if err != nil {
        fmt.Println("HTTP server failed,err:", err)
        return
    }
}

func SaveSession(w http.ResponseWriter, r *http.Request) {
    // Get a session. We're ignoring the error resulted from decoding an
    // existing session: Get() always returns a session, even if empty.

    //　获取一个session对象，session-name是session的名字
    session, err := store.Get(r, "session-name")
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }

    // 在session中存储值
    session.Values["foo"] = "bar"
    session.Values[42] = 43
    // 保存更改
    session.Save(r, w)
}
func GetSession(w http.ResponseWriter, r *http.Request) {
    session, err := store.Get(r, "session-name")
    if err != nil {
        http.Error(w, err.Error(), http.StatusInternalServerError)
        return
    }
    foo := session.Values["foo"]
    fmt.Println(foo)
}
```

删除session的值：

```
    // 删除
    // 将session的最大存储时间设置为小于零的数即为删除
    session.Options.MaxAge = -1
    session.Save(r, w)
```

### 多语言翻译验证

当业务系统对验证信息有特殊需求时，例如：返回信息需要自定义，手机端返回的信息需要是中文而pc端发挥返回的信息需要时英文，如何做到请求一个接口满足上述三种情况。

```go
package main

import (
    "fmt"

    "github.com/gin-gonic/gin"
    "github.com/go-playground/locales/en"
    "github.com/go-playground/locales/zh"
    "github.com/go-playground/locales/zh_Hant_TW"
    ut "github.com/go-playground/universal-translator"
    "gopkg.in/go-playground/validator.v9"
    en_translations "gopkg.in/go-playground/validator.v9/translations/en"
    zh_translations "gopkg.in/go-playground/validator.v9/translations/zh"
    zh_tw_translations "gopkg.in/go-playground/validator.v9/translations/zh_tw"
)

var (
    Uni      *ut.UniversalTranslator
    Validate *validator.Validate
)

type User struct {
    Username string `form:"user_name" validate:"required"`
    Tagline  string `form:"tag_line" validate:"required,lt=10"`
    Tagline2 string `form:"tag_line2" validate:"required,gt=1"`
}

func main() {
    en := en.New()
    zh := zh.New()
    zh_tw := zh_Hant_TW.New()
    Uni = ut.New(en, zh, zh_tw)
    Validate = validator.New()

    route := gin.Default()
    route.GET("/5lmh", startPage)
    route.POST("/5lmh", startPage)
    route.Run(":8080")
}

func startPage(c *gin.Context) {
    //这部分应放到中间件中
    locale := c.DefaultQuery("locale", "zh")
    trans, _ := Uni.GetTranslator(locale)
    switch locale {
    case "zh":
        zh_translations.RegisterDefaultTranslations(Validate, trans)
        break
    case "en":
        en_translations.RegisterDefaultTranslations(Validate, trans)
        break
    case "zh_tw":
        zh_tw_translations.RegisterDefaultTranslations(Validate, trans)
        break
    default:
        zh_translations.RegisterDefaultTranslations(Validate, trans)
        break
    }

    //自定义错误内容
    Validate.RegisterTranslation("required", trans, func(ut ut.Translator) error {
        return ut.Add("required", "{0} must have a value!", true) // see universal-translator for details
    }, func(ut ut.Translator, fe validator.FieldError) string {
        t, _ := ut.T("required", fe.Field())
        return t
    })

    //这块应该放到公共验证方法中
    user := User{}
    c.ShouldBind(&user)
    fmt.Println(user)
    err := Validate.Struct(user)
    if err != nil {
        errs := err.(validator.ValidationErrors)
        sliceErrs := []string{}
        for _, e := range errs {
            sliceErrs = append(sliceErrs, e.Translate(trans))
        }
        c.String(200, fmt.Sprintf("%#v", sliceErrs))
    }
    c.String(200, fmt.Sprintf("%#v", "user"))
}
```

正确的链接：http://localhost:8080/testing?user_name=枯藤&tag_line=9&tag_line2=33&locale=zh

http://localhost:8080/testing?user_name=枯藤&tag_line=9&tag_line2=3&locale=en 返回英文的验证信息

http://localhost:8080/testing?user_name=枯藤&tag_line=9&tag_line2=3&locale=zh 返回中文的验证信息

### 日志文件

```
package main

import (
    "io"
    "os"

    "github.com/gin-gonic/gin"
)

func main() {
    gin.DisableConsoleColor()

    // Logging to a file.
    f, _ := os.Create("gin.log")
    gin.DefaultWriter = io.MultiWriter(f)

    // 如果需要同时将日志写入文件和控制台，请使用以下代码。
    // gin.DefaultWriter = io.MultiWriter(f, os.Stdout)
    r := gin.Default()
    r.GET("/ping", func(c *gin.Context) {
        c.String(200, "pong")
    })
    r.Run()
}
```

![img](https://topgoer.com/static/gin/qita/1.png)

### Air实时加载

Air能够实时监听项目的代码文件，在代码发生变更之后自动重新编译并执行，大大提高gin框架项目的开发效率。

#### 为什么需要实时加载？

之前使用Python编写Web项目的时候，常见的Flask或Django框架都是支持实时加载的，你修改了项目代码之后，程序能够自动重新加载并执行（live-reload），这在日常的开发阶段是十分方便的。

在使用Go语言的gin框架在本地做开发调试的时候，经常需要在变更代码之后频繁的按下Ctrl+C停止程序并重新编译再执行，这样就不是很方便。

#### Air介绍

怎样才能在基于gin框架开发时实现实时加载功能呢？像这种烦恼肯定不会只是你一个人的烦恼，所以我报着肯定有现成轮子的心态开始了全网大搜索。果不其然就在Github上找到了一个工具：Air[1]。它支持以下特性：

- 彩色日志输出
- 自定义构建或二进制命令
- 支持忽略子目录
- 启动后支持监听新目录
- 更好的构建过程

#### 安装Air

Go

这也是最经典的安装方式：

```
    go get -u github.com/cosmtrek/air
```

MacOS

```
    curl -fLo air https://git.io/darwin_air
```

Linux

```
    curl -fLo air https://git.io/linux_air
```

Windows

```
    curl -fLo air.exe https://git.io/windows_air
```

Dcoker

```
docker run -it --rm \
    -w "<PROJECT>" \
    -e "air_wd=<PROJECT>" \
    -v $(pwd):<PROJECT> \
    -p <PORT>:<APP SERVER PORT> \
    cosmtrek/air
    -c <CONF>
```

然后按照下面的方式在docker中运行你的项目：

```
docker run -it --rm \
    -w "/go/src/github.com/cosmtrek/hub" \
    -v $(pwd):/go/src/github.com/cosmtrek/hub \
    -p 9090:9090 \
    cosmtrek/air
```

#### 使用Air

为了敲命令更简单更方便，你应该把`alias air='~/.air'`加到你的`.bashrc`或`.zshrc`中。

首先进入你的项目目录：

```
    cd /path/to/your_project
```

最简单的用法就是直接执行下面的命令：

```
# 首先在当前目录下查找 `.air.conf`配置文件，如果找不到就使用默认的
air -c .air.conf
```

推荐的使用方法是：

```
# 1. 在当前目录创建一个新的配置文件.air.conf
touch .air.conf

# 2. 复制 `air.conf.example` 中的内容到这个文件，然后根据你的需要去修改它

# 3. 使用你的配置运行 air, 如果文件名是 `.air.conf`，只需要执行 `air`。
air
```

#### air_example.conf示例

完整的air_example.conf示例配置如下，可以根据自己的需要修改。

```
# [Air](https://github.com/cosmtrek/air) TOML 格式的配置文件

# 工作目录
# 使用 . 或绝对路径，请注意 `tmp_dir` 目录必须在 `root` 目录下
root = "."
tmp_dir = "tmp"

[build]
# 只需要写你平常编译使用的shell命令。你也可以使用 `make`
cmd = "go build -o ./tmp/main ."
# 由`cmd`命令得到的二进制文件名
bin = "tmp/main"
# 自定义的二进制，可以添加额外的编译标识例如添加 GIN_MODE=release
full_bin = "APP_ENV=dev APP_USER=air ./tmp/main"
# 监听以下文件扩展名的文件.
include_ext = ["go", "tpl", "tmpl", "html"]
# 忽略这些文件扩展名或目录
exclude_dir = ["assets", "tmp", "vendor", "frontend/node_modules"]
# 监听以下指定目录的文件
include_dir = []
# 排除以下文件
exclude_file = []
# 如果文件更改过于频繁，则没有必要在每次更改时都触发构建。可以设置触发构建的延迟时间
delay = 1000 # ms
# 发生构建错误时，停止运行旧的二进制文件。
stop_on_error = true
# air的日志文件名，该日志文件放置在你的`tmp_dir`中
log = "air_errors.log"

[log]
# 显示日志时间
time = true

[color]
# 自定义每个部分显示的颜色。如果找不到颜色，使用原始的应用程序日志。
main = "magenta"
watcher = "cyan"
build = "yellow"
runner = "green"

[misc]
# 退出时删除tmp目录
clean_on_exit = true
```

###  gin验证码

在开发的过程中，我们有些接口为了防止被恶意调用，我们会采用加验证码的方式，例如：发送短信的接口，为了防止短信接口被频繁调用造成损失；注册的接口，为了防止恶意注册。在这里为大家推荐一个验证码的类库，方便大家学习使用。

```
     github.com/dchest/captcha
```

web端是怎么实现验证码的功能呢？

- 提供一个路由，**先在session里写入键值对（k->v），把值写在图片上，然后生成图片**，显示在浏览器上面
- 前端将图片中的内容发送给后后端，后端根据session中的k取得v，比对校验。如果通过继续下一步的逻辑，失败给出错误提示

API接口验证码实现方式类似，可以把键值对存储在起来，验证的时候把键值对传输过来一起校验。这里我只给出了web端的方法，爱动手的小伙伴可以自己尝试一下。

#### 后端

```go
package main

import (
    "bytes"
    "github.com/dchest/captcha"
    "github.com/gin-contrib/sessions"
    "github.com/gin-contrib/sessions/cookie"
    "github.com/gin-gonic/gin"
    "net/http"
    "time"
)

// 中间件，处理session
func Session(keyPairs string) gin.HandlerFunc {
    store := SessionConfig()
    return sessions.Sessions(keyPairs, store)
}
func SessionConfig() sessions.Store {
    sessionMaxAge := 3600
    sessionSecret := "topgoer"
    var store sessions.Store
    store = cookie.NewStore([]byte(sessionSecret))
    store.Options(sessions.Options{
        MaxAge: sessionMaxAge, //seconds
        Path:   "/",
    })
    return store
}

func Captcha(c *gin.Context, length ...int) {
    l := captcha.DefaultLen
    w, h := 107, 36
    if len(length) == 1 {
        l = length[0]
    }
    if len(length) == 2 {
        w = length[1]
    }
    if len(length) == 3 {
        h = length[2]
    }
    captchaId := captcha.NewLen(l)
    session := sessions.Default(c)
    session.Set("captcha", captchaId)
    _ = session.Save()
    _ = Serve(c.Writer, c.Request, captchaId, ".png", "zh", false, w, h)
}
func CaptchaVerify(c *gin.Context, code string) bool {
    session := sessions.Default(c)
    if captchaId := session.Get("captcha"); captchaId != nil {
        session.Delete("captcha")
        _ = session.Save()
        if captcha.VerifyString(captchaId.(string), code) {
            return true
        } else {
            return false
        }
    } else {
        return false
    }
}
func Serve(w http.ResponseWriter, r *http.Request, id, ext, lang string, download bool, width, height int) error {
    w.Header().Set("Cache-Control", "no-cache, no-store, must-revalidate")
    w.Header().Set("Pragma", "no-cache")
    w.Header().Set("Expires", "0")

    var content bytes.Buffer
    switch ext {
    case ".png":
        w.Header().Set("Content-Type", "image/png")
        _ = captcha.WriteImage(&content, id, width, height)
    case ".wav":
        w.Header().Set("Content-Type", "audio/x-wav")
        _ = captcha.WriteAudio(&content, id, lang)
    default:
        return captcha.ErrNotFound
    }

    if download {
        w.Header().Set("Content-Type", "application/octet-stream")
    }
    http.ServeContent(w, r, id+ext, time.Time{}, bytes.NewReader(content.Bytes()))
    return nil
}

func main() {
    router := gin.Default()
    router.LoadHTMLGlob("./*.html")
    router.Use(Session("topgoer"))
    router.GET("/captcha", func(c *gin.Context) {
        Captcha(c, 4)
    })
    router.GET("/", func(c *gin.Context) {
        c.HTML(http.StatusOK, "index.html", nil)
    })
    router.GET("/captcha/verify/:value", func(c *gin.Context) {
        value := c.Param("value")
        if CaptchaVerify(c, value) {
            c.JSON(http.StatusOK, gin.H{"status": 0, "msg": "success"})
        } else {
            c.JSON(http.StatusOK, gin.H{"status": 1, "msg": "failed"})
        }
    })
    router.Run(":8080")
}
```

#### 前端页面

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>www.topgoer.com验证码</title>
</head>
<body>
<img src="/captcha" onclick="this.src='/captcha?v='+Math.random()">
</body>
</html>
```

浏览器访问[http://127.0.0.1:8080](http://127.0.0.1:8080/)

访问http://127.0.0.1:8080/captcha/verify/5721 进行验证

```
    {
        "msg": "failed",
        "status": 1
    }
```

### 生成解析token

如今有很多将身份验证内置到API中的方法 -JSON Web令牌只是其中之一。JSON Web令牌（JWT）作为令牌系统而不是在每次请求时都发送用户名和密码，因此比其他方法（如基本身份验证）具有固有的优势。要了解更多信息，请直接进入jwt.io上的介绍，然后再直接学习。

以下是JWT的实际应用示例。主**要有两个部分：提供用户名和密码以获取令牌；并根据请求检查该令牌**。

在此示例中，我们使用了两个库，即Go中的JWT实现以及将其用作中间件的方式。

最后，在使用此代码之前，您需要将APP_KEY常量更改为机密（理想情况下，该常量将存储在代码库外部），并改进用户名/密码检查中的内容，TokenHandler以检查不仅仅是myusername/ mypassword组合。

下面的代码是gin框架对jwt的封装

```go
package main

import (
    "fmt"
    "net/http"
    "time"

    "github.com/dgrijalva/jwt-go"
    "github.com/gin-gonic/gin"
)

//自定义一个字符串
var jwtkey = []byte("www.topgoer.com")
var str string

type Claims struct {
    UserId uint
    jwt.StandardClaims
}

func main() {
    r := gin.Default()
    r.GET("/set", setting)
    r.GET("/get", getting)
    //监听端口默认为8080
    r.Run(":8080")
}

//颁发token
func setting(ctx *gin.Context) {
    expireTime := time.Now().Add(7 * 24 * time.Hour)
    claims := &Claims{
        UserId: 2,
        StandardClaims: jwt.StandardClaims{
            ExpiresAt: expireTime.Unix(), //过期时间
            IssuedAt:  time.Now().Unix(),
            Issuer:    "127.0.0.1",  // 签名颁发者
            Subject:   "user token", //签名主题
        },
    }
    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    // fmt.Println(token)
    tokenString, err := token.SignedString(jwtkey)
    if err != nil {
        fmt.Println(err)
    }
    str = tokenString
    ctx.JSON(200, gin.H{"token": tokenString})
}

//解析token
func getting(ctx *gin.Context) {
    tokenString := ctx.GetHeader("Authorization")
    //vcalidate token formate
    if tokenString == "" {
        ctx.JSON(http.StatusUnauthorized, gin.H{"code": 401, "msg": "权限不足"})
        ctx.Abort()
        return
    }

    token, claims, err := ParseToken(tokenString)
    if err != nil || !token.Valid {
        ctx.JSON(http.StatusUnauthorized, gin.H{"code": 401, "msg": "权限不足"})
        ctx.Abort()
        return
    }
    fmt.Println(111)
    fmt.Println(claims.UserId)
}

func ParseToken(tokenString string) (*jwt.Token, *Claims, error) {
    Claims := &Claims{}
    token, err := jwt.ParseWithClaims(tokenString, Claims, func(token *jwt.Token) (i interface{}, err error) {
        return jwtkey, nil
    })
    return token, Claims, err
}
```

### 权限管理

Casbin是用于Golang项目的功能强大且高效的开源访问控制库。

#### 特征

Casbin的作用：

- 以经典{subject, object, action}形式或您定义的自定义形式实施策略，**同时支持允许和拒绝授权**。
- 处理访问控制模型及其策略的存储。
- 管理角色用户映射和角色角色映射（[RBAC](https://zhuanlan.zhihu.com/p/63769951)中的角色层次结构）。
- 支持内置的超级用户，例如root或administrator。超级用户可以在没有显式权限的情况下执行任何操作。
- 多个内置运算符支持规则匹配。例如，keyMatch可以将资源键映射/foo/bar到模式`/foo*`。

Casbin不执行的操作：

- 身份验证（又名验证username以及password用户登录时）
- 管理用户或角色列表。我相信项目本身管理这些实体会更方便。用户通常具有其密码，而Casbin并非设计为密码容器。但是，Casbin存储RBAC方案的用户角色映射。

#### 怎么运行的

在Casbin中，基于PERM元模型（策略，效果，请求，匹配器）将访问控制模型抽象为CONF文件。因此，切换或升级项目的授权机制就像修改配置一样简单。您可以通过组合可用的模型来定制自己的访问控制模型。例如，您可以在一个模型中同时获得RBAC角色和ABAC属性，并共享一组策略规则。

Casbin中最基本，最简单的模型是ACL。ACL的CONF模型为：

```
＃请求定义
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act

[role_definition]
g = _, _

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = g(r.sub, p.sub) && r.obj == p.obj && r.act == p.act
```

ACL模型的示例策略如下：

```
p, alice, data1, read
p, bob, data2, write
```

#### 安装

```
    go get github.com/casbin/casbin
```

#### 示例代码

```go
package main

import (
    "fmt"
    "log"

    "github.com/casbin/casbin"
    xormadapter "github.com/casbin/xorm-adapter"
    "github.com/gin-gonic/gin"
    _ "github.com/go-sql-driver/mysql"
)

func main() {
    // 要使用自己定义的数据库rbac_db,最后的true很重要.默认为false,使用缺省的数据库名casbin,不存在则创建
    a, err := xormadapter.NewAdapter("mysql", "root:root@tcp(127.0.0.1:3306)/goblog?charset=utf8", true)
    if err != nil {
        log.Printf("连接数据库错误: %v", err)
        return
    }
    e, err := casbin.NewEnforcer("./rbac_models.conf", a)
    if err != nil {
        log.Printf("初始化casbin错误: %v", err)
        return
    }
    //从DB加载策略
    e.LoadPolicy()

    //获取router路由对象
    r := gin.New()

    r.POST("/api/v1/add", func(c *gin.Context) {
        fmt.Println("增加Policy")
        if ok, _ := e.AddPolicy("admin", "/api/v1/hello", "GET"); !ok {
            fmt.Println("Policy已经存在")
        } else {
            fmt.Println("增加成功")
        }
    })
    //删除policy
    r.DELETE("/api/v1/delete", func(c *gin.Context) {
        fmt.Println("删除Policy")
        if ok, _ := e.RemovePolicy("admin", "/api/v1/hello", "GET"); !ok {
            fmt.Println("Policy不存在")
        } else {
            fmt.Println("删除成功")
        }
    })
    //获取policy
    r.GET("/api/v1/get", func(c *gin.Context) {
        fmt.Println("查看policy")
        list := e.GetPolicy()
        for _, vlist := range list {
            for _, v := range vlist {
                fmt.Printf("value: %s, ", v)
            }
        }
    })
    //使用自定义拦截器中间件
    r.Use(Authorize(e))
    //创建请求
    r.GET("/api/v1/hello", func(c *gin.Context) {
        fmt.Println("Hello 接收到GET请求..")
    })

    r.Run(":9000") //参数为空 默认监听8080端口
}

//拦截器
func Authorize(e *casbin.Enforcer) gin.HandlerFunc {

    return func(c *gin.Context) {

        //获取请求的URI
        obj := c.Request.URL.RequestURI()
        //获取请求方法
        act := c.Request.Method
        //获取用户的角色
        sub := "admin"

        //判断策略中是否存在
        if ok, _ := e.Enforce(sub, obj, act); ok {
            fmt.Println("恭喜您,权限验证通过")
            c.Next()
        } else {
            fmt.Println("很遗憾,权限验证没有通过")
            c.Abort()
        }
    }
}
```

rbac_models.conf里面的内容如下：

```
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act

[role_definition]
g = _, _

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = g(r.sub, p.sub) && r.obj == p.obj && r.act == p.act
```

配置链接数据库不需要手动创建数据库，系统自动创建`casbin_rule`表

使用postman请求http://localhost:9000/api/v1/hello

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/1.jpg)

运行解决结果显示为`很遗憾,权限验证没有通过`

下面我在数据表中添加数据在演示的时候可以直接手动按照图片的格式直接添加数据表，或者使用postman POST方式请求http://localhost:9000/api/v1/add

![img](https://topgoer.com/static/gin/qita/rabc/2.jpg)

然后继续请求http://localhost:9000/api/v1/hello

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/3.jpg)

## Jsoniter

#### 使用 [jsoniter](https://github.com/json-iterator/go) 编译

Gin 使用 `encoding/json` 作为默认的 json 包，但是你可以在编译中使用标签将其修改为 [jsoniter](https://github.com/json-iterator/go)。

```sh
$ go build -tags=jsoniter .
```

## 源码解读

gin源码阅读系列就是要弄明白以下几个问题:

- request数据是如何流转的
- gin框架到底扮演了什么角色
- 请求从gin流入net/http, 最后又是如何回到gin中
- gin的context为何能承担起来复杂的需求
- gin的路由算法
- gin的中间件是什么
- gin的Engine具体是个什么东西
- net/http的requeset, response都提供了哪些有用的东西

### request数据是如何流转的

先不使用gin, 直接使用net/http来处理http请求

```go
func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello World"))
    })

    if err := http.ListenAndServe(":8000", nil); err != nil {
        fmt.Println("start http server fail:", err)
    }
}
```

在浏览器中输入localhost:8000, 会看到Hello World. 下面利用这个简单demo看下request的流转流程.

#### HTTP是如何建立起来的

简单的说一下http请求是如何建立起来的(需要有基本的网络基础, 可以找相关的书籍查看, 推荐看UNIX网络编程卷1：套接字联网API)

![img](https://topgoer.com/static/gin/yuanma/2.png)

在TCP/IP五层模型下, HTTP位于应用层, 需要有传输层来承载HTTP协议. 传输层比较常见的协议是TCP,UDP, SCTP等. 由于UDP不可靠, SCTP有自己特殊的运用场景, 所以一般情况下HTTP是由TCP协议承载的(可以使用wireshark抓包然后查看各层协议)

使用TCP协议的话, 就会涉及到TCP是如何建立起来的. 面试中能够常遇到的名词三次握手, 四次挥手就是在这里产生的. 具体的建立流程就不在陈述了, 大概流程就是图中左半边

所以说, 要想能够对客户端http请求进行回应的话, 就首先需要建立起来TCP连接, 也就是socket. 下面要看下net/http是如何建立起来socket?

#### net/http是如何建立socket的

从图上可以看出, 不管server代码如何封装, 都离不开bind,listen,accept这些函数. 就从上面这个简单的demo入手查看源码.

```go
func main() {
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello World"))
    })

    if err := http.ListenAndServe(":8000", nil); err != nil {
        fmt.Println("start http server fail:", err)
    }
}
```

#### 注册路由

```go
http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        w.Write([]byte("Hello World"))
    })
```

这段代码是在注册一个路由及这个路由的handler到DefaultServeMux中

```go
// server.go:L2366-2388
func (mux *ServeMux) Handle(pattern string, handler Handler) {
    mux.mu.Lock()
    defer mux.mu.Unlock()

    if pattern == "" {
        panic("http: invalid pattern")
    }
    if handler == nil {
        panic("http: nil handler")
    }
    if _, exist := mux.m[pattern]; exist {
        panic("http: multiple registrations for " + pattern)
    }

    if mux.m == nil {
        mux.m = make(map[string]muxEntry)
    }
    mux.m[pattern] = muxEntry{h: handler, pattern: pattern}

    if pattern[0] != '/' {
        mux.hosts = true
    }
}
```

可以看到这个路由注册太过简单了, 也就给gin, iris, echo等框架留下了扩展的空间, 后面详细说这个东西

#### 服务监听及响应

上面路由已经注册到net/http了, 下面就该如何建立socket了, 以及最后又如何取到已经注册到的路由, 将正确的响应信息从handler中取出来返回给客户端

```go
if err := http.ListenAndServe(":8000", nil); err != nil {
    fmt.Println("start http server fail:", err)
}
// net/http/server.go:L3002-3005
func ListenAndServe(addr string, handler Handler) error {
    server := &Server{Addr: addr, Handler: handler}
    return server.ListenAndServe()
}
// net/http/server.go:L2752-2765
func (srv *Server) ListenAndServe() error {
    // ... 省略代码
    ln, err := net.Listen("tcp", addr) // <-----看这里listen
    if err != nil {
        return err
    }
    return srv.Serve(tcpKeepAliveListener{ln.(*net.TCPListener)})
}
// net/http/server.go:L2805-2853
func (srv *Server) Serve(l net.Listener) error {
    // ... 省略代码
    for {
        rw, e := l.Accept() // <----- 看这里accept
        if e != nil {
            select {
            case <-srv.getDoneChan():
                return ErrServerClosed
            default:
            }
            if ne, ok := e.(net.Error); ok && ne.Temporary() {
                if tempDelay == 0 {
                    tempDelay = 5 * time.Millisecond
                } else {
                    tempDelay *= 2
                }
                if max := 1 * time.Second; tempDelay > max {
                    tempDelay = max
                }
                srv.logf("http: Accept error: %v; retrying in %v", e, tempDelay)
                time.Sleep(tempDelay)
                continue
            }
            return e
        }
        tempDelay = 0
        c := srv.newConn(rw)
        c.setState(c.rwc, StateNew) // before Serve can return
        go c.serve(ctx) // <--- 看这里
    }
}
// net/http/server.go:L1739-1878
func (c *conn) serve(ctx context.Context) {
    // ... 省略代码
    serverHandler{c.server}.ServeHTTP(w, w.req)
    w.cancelCtx()
    if c.hijacked() {
        return
    }
    w.finishRequest()
    // ... 省略代码
}
// net/http/server.go:L2733-2742
func (sh serverHandler) ServeHTTP(rw ResponseWriter, req *Request) {
    handler := sh.srv.Handler
    if handler == nil {
        handler = DefaultServeMux
    }
    if req.RequestURI == "*" && req.Method == "OPTIONS" {
        handler = globalOptionsHandler{}
    }
    handler.ServeHTTP(rw, req)
}
// net/http/server.go:L2352-2362
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request) {
    if r.RequestURI == "*" {
        if r.ProtoAtLeast(1, 1) {
            w.Header().Set("Connection", "close")
        }
        w.WriteHeader(StatusBadRequest)
        return
    }
    h, _ := mux.Handler(r) // <--- 看这里
    h.ServeHTTP(w, r)
}

// net/http/server.go:L1963-1965
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```

这基本是整个过程的代码了. 基本上是:

- ln, err := net.Listen("tcp", addr)做了初试化了socket, bind, listen的操作.
- rw, e := l.Accept()进行accept, 等待客户端进行连接
- go c.serve(ctx) 启动新的goroutine来处理本次请求. 同时主goroutine继续等待客户端连接, 进行高并发操作
- h, _ := mux.Handler(r) 获取注册的路由, 然后拿到这个路由的handler, 然后将处理结果返回给客户端

从这里也能够看出来, net/http基本上提供了全套的服务.

#### 为什么会出现很多go框架

```go
// net/http/server.go:L2218-2238
func (mux *ServeMux) match(path string) (h Handler, pattern string) {
    // Check for exact match first.
    v, ok := mux.m[path]
    if ok {
        return v.h, v.pattern
    }

    // Check for longest valid match.
    var n = 0
    for k, v := range mux.m {
        if !pathMatch(k, path) {
            continue
        }
        if h == nil || len(k) > n {
            n = len(k)
            h = v.h
            pattern = v.pattern
        }
    }
    return
}
```

从这段函数可以看出来, 匹配规则过于简单, **当能匹配到路由的时候就返回其对应的handler, 当不能匹配到时就返回/. 所以net/http的路由匹配无法满足复杂的需求开发. 所以基本所有的go框架干的最主要的一件事情就是重写net/http的route**

所以我们直接说gin就是一个httprouter也不过分, 当然gin也提供了其他比较主要的功能, 后面会一一介绍

还有一个go框架要实现的东西是http.ResponseWriter

综述, net/http基本已经提供http服务的70%的功能, 那些号称贼快的go框架, 基本上都是提供一些功能, 让我们能够更好的处理客户端发来的请求.

### 数据如何在gin中流转

```go
package main

import "github.com/gin-gonic/gin"

func main() {
    r := gin.Default()
    r.GET("/ping", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "pong",
        })
    })
    r.Run() // listen and serve on 0.0.0.0:8080
}
```

这段代码的大概流程:

1.r := gin.Default()初始化了相关的参数 

2./ping将路由及处理handler注册到路由树中 

3.启动服务

r.Run()其实调用的是err = http.ListenAndServe(address, engine), 结合上一篇文章可以看出来, gin其实利用了net/http的处理过程

#### ServeHTTP的作用

上面有提到DefaultServeMux, 其实DefaultServeMux实现了ServeHTTP(ResponseWriter, *Request), 在request执行到server.go的serverHandler{c.server}.ServeHTTP(w, w.req)这一行的时候, 从DefaultServeMux取到了相关路由的处理handler.

因此, **gin框架的Engine最重要的函数就是func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request). Engine实现了Handler(server.go#L84-86), 让net/http请求数据最终流回到gin中**, 从gin的route tree中取到相关的中间件及handler, 来处理客户端的request

#### Engine

在整个gin框架中最重要的一个struct就是Engine, 它包含路由, 中间件, 相关配置信息等. Engine的代码主要就在gin.go中

Engine中比较重要的几个属性, 其他的属性暂时全部省略掉

```go
type Engine struct {
    RouterGroup // 路由
    pool             sync.Pool  // context pool
    trees            methodTrees // 路由树
    // html template及其他相关属性先暂时忽略
}
```

Engine有几个比较主要的函数:

#### New(), Default()

```go
func New() *Engine {
    // ...
    engine := &Engine{
        RouterGroup: RouterGroup{
            Handlers: nil,
            basePath: "/",
            root:     true,
        },
        // ...
        trees: make(methodTrees, 0, 9),
    }
    engine.RouterGroup.engine = engine
    engine.pool.New = func() interface{} {
        return engine.allocateContext()
    }
    return engine
}
```

New()主要干的事情:

1.初始化了Engine 

2.将RouterGroup的Handlers(数组)设置成nil, basePath设置成/ 

3.为了使用方便, RouteGroup里面也有一个Engine指针, 这里将刚刚初始化的engine赋值给了RouterGroup的engine指针 

4.为了防止频繁的context GC造成效率的降低, 在Engine里使用了sync.Pool, 专门存储gin的Context

```go
func Default() *Engine {
    debugPrintWARNINGDefault()
    engine := New()
    engine.Use(Logger(), Recovery())
    return engine
}
```

Default()跟New()几乎一模一样, 就是调用了gin内置的Logger(), Recovery()中间件.

#### Use()

```go
func (engine *Engine) Use(middleware ...HandlerFunc) IRoutes {
    engine.RouterGroup.Use(middleware...)
    engine.rebuild404Handlers()
    engine.rebuild405Handlers()
    return engine
}
```

**Use()就是gin的引入中间件的入口了**. 仔细分析这个函数, 不难发现Use()其实是在给RouteGroup引入中间件的. 具体是如何让中间件在RouteGroup上起到作用的, 等说到RouteGroup再具体说.

```go
engine.rebuild404Handlers()
engine.rebuild405Handlers()
```

这两句函数其实在这里没有任何用处. 我感觉这里是给gin的测试代码用的. 我们在使用gin的时候, 要想在404, 405添加处理过程, 可以通过NoRoute(), NoMethod()来处理.

#### addRoute()

```go
func (engine *Engine) addRoute(method, path string, handlers HandlersChain) {
    ...
    root := engine.trees.get(method)
    if root == nil {
        root = new(node)
        engine.trees = append(engine.trees, methodTree{method: method, root: root})
    }
    root.addRoute(path, handlers)
}
```

这段代码就是利用method, path, 将handlers注册到engine的trees中. 注意这里为什么是HandlersChain呢, 可以简单说一下, 就是将中间件和处理函数都注册到method, path的tree中了.

#### Run系列函数

Run, RunTLS, RunUnix, RunFd 这些函数其实都是最终在调用net/http的http服务.

#### ServeHTTP

这个函数相当重要了, 主要有这个函数的存在, 才能将请求转到gin中, 使用gin的相关函数处理request请求.

```go
...

t := engine.trees

for i, tl := 0, len(t); i < tl; i++ {
    if t[i].method != httpMethod {
        continue
    }
    root := t[i].root

    handlers, params, tsr := root.getValue(path, c.Params, unescape)
    if handlers != nil {
        c.handlers = handlers
        c.Params = params
        c.Next()
        c.writermem.WriteHeaderNow()
        return
    }
    ...
}
```

利用request中的path, 从Engine的trees中获取已经注册的handler

```go
func (c *Context) Next() {
    c.index++
    for c.index < int8(len(c.handlers)) {
        c.handlers[c.index](c)
        c.index++
    }
}
```

在Next()执行handler的操作. 其实也就是下面的函数

```go
func(c *gin.Context) {
    c.JSON(200, gin.H{
        "message": "pong",
    })
}
```

如果在trees中没有找到对应的路由, 则会执行serveError函数, 也就是404相关的.

### context

**Gin封装的最好的地方就是context和对response的处理**. github的README的介绍,基本就是对这两个东西的解释. 本篇文章主要解释context的使用方法, 以及其设计原理

#### 为什么要将Request的处理封装到Context中

在阅读gin的源码时, 请求的处理是使用type HandlerFunc func(*Context)来处理的. 也就是

```go
func(context *gin.Context) {
    context.String(http.StatusOK, "some post")
}
```

参数是gin.Context, 但是查看源码发现其实gin.Context在整个框架处理的地方只有下面这段:

```go
func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
    c := engine.pool.Get().(*Context)
    c.writermem.reset(w)
    c.Request = req
    c.reset()
    engine.handleHTTPRequest(c)
    engine.pool.Put(c)
}
```

那为什么还要利用Context来处理呢. gin的context实现了的context.Context Interface.

经过查看context.Context相关资料, **Context的最佳运用场景就是对Http的处理.** 封装成Conetxt另外的好处就是WithCancel, WithDeadline, WithTimeout, WithValue这些context包衍生的子Context就可以直接来使用. 目前我能想到的地方就这么多, 以后发现gin.Context其他的优点再补充.

#### gin.Context的设计

gin.Context主要由下面几部分组成(这里沿用源代码里面的注释)

#### Metadata Management (我自己叫法:Key-Value)

这个模块比较简单, 就是从gin.Context中Set Key-Value, 以及各种个样的Get方法, 如GetBool, GetString等

实现这些功能也很简单, 其实就是一个map

```go
// Keys is a key/value pair exclusively for the context of each request.
Keys map[string]interface{}
```

#### Input Data

这个模块相当重要了, gin的README基本上都在介绍这个模块的用法.

#### Param (我自己的叫法: 路由变量)

gin的标准叫法是Parameters in path. restful风格api如/user/john, 这个路由在gin里面是/user/:name, 要获取john就需要使用Param函数

```go
name := c.Param("name")
```

这个方法实现也很简单, 就是在tree.go里面根据路由相关规则解析出来然后赋值给gin.Context的Params.

```go
handlers, params, tsr := root.getValue(path, c.Params, unescape)
```

#### Query

/welcome?firstname=Jane&lastname=Doe这样一个路由, first, last即是Querystring parameters, 要获取他们就需要使用Query相关函数.

```go
c.Query("first") // Jane
c.Query("last") // Doe
```

当然还有其他相关函数:

- QueryMap
- DefaultQuery 这个默认值的实现更加简单, 当QueryString中不包含这个值, 直接返回填入的值

这些方法是的实现是利用net/http的Request的方法实现的

#### PostForm

对于POST, PUT等这些能够传递参数Body的请求, 要获取其参数, 需要使用PostForm

```go
POST /user/1

{
    "name":manu,
    "message":this_is_great
}
name := c.PostForm("name")
message := c.PostForm("message")
```

其他相关函数

- DefaultPostForm

这些相关的方法是实现还是利用net/http的Request的方法实现的

#### FormFile

对于文件相关的操作, 一般生产情况下不建议这样使用, 因为把文件上传到服务器磁盘, 还得磁盘相关的监控. 我觉得最好利用云服务商相关的对象存储, 如:阿里云OSS, 七牛云对象存储, AWS的对象存储等来做文件的相关操作

#### Bind

内置的有json, xml, protobuf, form, query, yaml. 这些Bind极大的减少我们自己去解析各种个样的数据格式, 提高我们的开发速度

Bind的实现都在gin/binding里面. 这些内置的Bind都实现了Binding接口, 主要是Bind()函数.

- context.BindJSON() 支持MIME为application/json的解析
- context.BindXML() 支持MIME为application/xml的解析
- context.BindYAML() 支持MIME为application/x-yaml的解析
- context.BindQuery() 只支持QueryString的解析, 和Query()函数一样
- context.BindUri() 只支持路由变量的解析
- Context.Bind() 支持所有的类型的解析, 这个函数尽量还是少用(当QueryString, PostForm, 路由变量在一块同时使用时会产生意想不到的效果), 目前测试Bind不支持路由变量的解析, Bind()函数的解析比较复杂, 这部分代码后面再看

#### Response

#### 对Header的支持

- Header
- GetHeader

这里的**Header是写到Response里面的Header.** 对于**客户端发的请求的Header可以通过context.Request.Header.Get("Content-Type")获取**

#### Cookie

提供对session, cookie的支持

#### render

做api常用到的其实就是gin封装的各种render. 目前支持的有:

- func (c *Context) JSON(code int, obj interface{})
- func (c *Context) Protobuf(code int, obj interface{})
- func (c *Context) YAML(code int, obj interface{}) ...

当然我们可以自定义渲染, 只要实现func (c *Context) Render(code int, r render.Render)即可.

这里我们常用的是一个方法是: gin.H{"error": 111}. 这个结构相当实用, 各种render都支持. 其实这个结构很简单就是type H map[string]interface{}, 当我们要从map转换各种各样结构时, 不妨参考gin这里的代码

Context说到这里基本就说完了, 这里介绍的方法都是开发中特别实用的方法. context的代码实现也特别有条理, 建议可以看看这部分代码
