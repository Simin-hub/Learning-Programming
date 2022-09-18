# go-zero

## proto文件

```
// proto版本
syntax = "proto3";

// protocol文件所在文件的包
package common;

// protoc-gen-go 版本大于1.4.0, proto文件需要加上go_package,否则无法生成
// option go_package = "./common";

// 消息体
message BaseAppResp{
  int64 beid=1;
  string logo=2;
  string sname=3;
  int64 isclose=4;
  string fullwebsite=5;
}

//请求的api
message AppConfigReq {
  int64 beid=1;
  int64 ptyid=2;
}

//返回的值
message AppConfigResp {
  int64 id=1;
  int64 beid=2;
  int64 ptyid=3;
  string appid=4;
  string appsecret=5;
  string title=6;
}

// 提供什么服务
service Common {
  rpc GetAppConfig(AppConfigReq) returns(AppConfigResp);
  rpc GetBaseApp(BaseAppReq) returns(BaseAppResp);
}

```

## API文件

```
info(
	title: "user api"// TODO: add title
	desc: "用户系统"// TODO: add description
	author: "jackluo"
	email: "net.webjoy@gmail.com"
)
//注册请求
type RegisterReq struct {
	// TODO: add members here and delete this comment
	Username string `json:"username"`
	Mobile   string `json:"mobile"` //基本一个手机号码就完事
	Password string `json:"password"`
	Smscode	string `json:"smscode"` //短信码
}
//登陆请求
type LoginReq struct{
	Username string `json:"username"`
	Mobile   string `json:"mobile"` 
	Password string `json:"password"`
	Smscode	string `json:"smscode"` //短信码
}
//返回用户信息
type UserReply struct {
	auid       int64  `json:"auid"`
	Uid       int64  `json:"uid"`
	Beid	  int64  `json:"beid"` //应用id
	Ptyid	  int64  `json:"ptyid"` //对应平台
	Username string `json:"username"`
	Mobile   string `json:"mobile"`
	Nickname string `json:"nickname"`
	Openid string `json:"openid"`
	Avator string `json:"avator"`
	Gender   string `json:"gender"`
	JwtToken
}

type JwtToken struct {
	AccessToken  string `json:"accessToken,omitempty"`
	AccessExpire int64  `json:"accessExpire,omitempty"`
	RefreshAfter int64  `json:"refreshAfter,omitempty"`
}

service user-api {
	@handler ping
	post /user/ping ()
	
	@handler register
	post /user/register (RegisterReq)
	
	@handler login
	post /user/login (LoginReq) returns (UserReply)
}
@server(
    jwt: Auth
    middleware: Usercheck
)
service user-api {
    @handler userInfo
    get /user/info () returns (UserReply)
}
```

## 使用rpc

rpc 生成文件

```
goctl rpc protoc user.proto --go_out=./types --go-grpc_out=./types --zrpc_out=.
```

1. 首先设置 etc/xxxx.yaml 相关配置，例如数据库连接，缓存连接，etcd等
2. 设置 internal/config/config.go 文件中关于上一步的配置
3. 增加 model 目录，在编写数据库映射以及相关增删改查
4. 修改 internal/svc/servicecontext.go 中服务运行上下文
5. 修改 internal/logic/xxxx.go 完善逻辑

## 使用API

```
goctl api go -api order.api -dir .
```

生成etc/, internal/等文件

1. 修改 etc/xxx.yaml 文件
2. 修改 internal/config/xxx.go
3. 修改 internal/svc/servicecontext.go 中服务运行上下文
4. 修改 internal/logic/xxxx.go 完善逻辑
5. 修改 internal/middleware/xxxxxx.go 晚上中间件逻辑

## 中间件的使用

### JWT

[参考](https://go-zero.dev/cn/docs/advance/jwt)

[JWT详解](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E7%AC%AC%E4%B8%89%E6%96%B9%E5%BA%93/jwt-go.md)

jwt鉴权一般在api层使用，我们这次演示工程中分别在user api登录时生成jwt token，在search api查询图书时验证用户jwt token两步来实现。

### 其他中间件

[参考](https://go-zero.dev/cn/docs/advance/middleware#%E8%B7%AF%E7%94%B1%E4%B8%AD%E9%97%B4%E4%BB%B6)

在go-zero中，**中间件可以分为路由中间件和全局中间件**，路由中间件是指某一些特定路由需要实现中间件逻辑，其和jwt类似，没有放在`jwt:xxx`下的路由不会使用中间件功能， 而全局中间件的服务范围则是整个服务。

#### 路由中间件

### 路由中间件

[参考](https://go-zero.dev/cn/docs/advance/middleware#路由中间件)

- **重新编写`search.api`文件，添加`middleware`声明**

  在`api`文件中的`server`中加入`middleware: Example // 路由中间件声明` 

  ```
  $ cd service/search/api
  $ vim search.api
  ```

  ```
  type SearchReq struct {}
  
  type SearchReply struct {}
  
  @server(
      jwt: Auth
      middleware: Example // 路由中间件声明
  )
  service search-api {
      @handler search
      get /search/do (SearchReq) returns (SearchReply)
  }
  ```

- 重新生成api代码

  ```
  $ goctl api go -api search.api -dir . 
  ```

  生成完后会在`internal`目录下多一个`middleware`的目录，这里即中间件文件，后续中间件的实现逻辑也在这里编写。

- 完善资源依赖`ServiceContext`

  在服务上下文文件中（ServiceContext.go）定义中间件

  ```
  $ vim service/search/api/internal/svc/servicecontext.go
  ```

  ```
  type ServiceContext struct {
      Config config.Config
      Example rest.Middleware
  }
  
  func NewServiceContext(c config.Config) *ServiceContext {
      return &ServiceContext{
          Config: c,
          Example: middleware.NewExampleMiddleware().Handle,
      }
  }
  ```

- 编写中间件逻辑 这里仅添加一行日志，内容example middle，如果服务运行输出example middle则代表中间件使用起来了。

  在`internal/middleware/examplemiddleware.go` 文件中完善中间件逻辑

  ```
  $ vim service/search/api/internal/middleware/examplemiddleware.go
  ```

  ```
  package middleware
  
  import "net/http"
  
  type ExampleMiddleware struct {
  }
  
  func NewExampleMiddleware() *ExampleMiddleware {
          return &ExampleMiddleware{}
  }
  
  func (m *ExampleMiddleware) Handle(next http.HandlerFunc) http.HandlerFunc {
      logx.Info("example middle")
      return func(w http.ResponseWriter, r *http.Request) {
          // TODO generate middleware implement function, delete after code implementation
  
          // Passthrough to next handler if need
          next(w, r)
      }
  }
  ```

- 启动服务验证

  ```
  {"@timestamp":"2021-02-09T11:32:57.931+08","level":"info","content":"example middle"}
  ```

  

### 全局中间件

[参考](https://go-zero.dev/cn/docs/advance/middleware#全局中间件)

通过rest.Server提供的Use方法即可

```
func main() {
    flag.Parse()

    var c config.Config
    conf.MustLoad(*configFile, &c)

    ctx := svc.NewServiceContext(c)
    server := rest.MustNewServer(c.RestConf)
    defer server.Stop()

    // 全局中间件
    server.Use(func(next http.HandlerFunc) http.HandlerFunc {
        return func(w http.ResponseWriter, r *http.Request) {
            logx.Info("global middleware")
            next(w, r)
        }
    })
    handler.RegisterHandlers(server, ctx)

    fmt.Printf("Starting server at %s:%d...\n", c.Host, c.Port)
    server.Start()
}
```

### 在中间件里调用其它服务

[参考](https://go-zero.dev/cn/docs/advance/middleware#在中间件里调用其它服务)

通过闭包的方式把其它服务传递给中间件，示例如下：

```
// 模拟的其它服务
type AnotherService struct{}

func (s *AnotherService) GetToken() string {
    return stringx.Rand()
}

// 常规中间件
func middleware(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        w.Header().Add("X-Middleware", "static-middleware")
        next(w, r)
    }
}

// 调用其它服务的中间件
func middlewareWithAnotherService(s *AnotherService) rest.Middleware {
    return func(next http.HandlerFunc) http.HandlerFunc {
        return func(w http.ResponseWriter, r *http.Request) {
            w.Header().Add("X-Middleware", s.GetToken())
            next(w, r)
        }
    }
}
```

## 错误处理

错误的处理是一个服务必不可缺的环节。在平时的业务开发中，我们可以认为http状态码不为`2xx`系列的，都可以认为是http请求错误， 并伴随响应的错误信息，但这些错误信息都是以plain text形式返回的。除此之外，我在业务中还会定义一些业务性错误，常用做法都是通过 `code`、`msg` 两个字段来进行业务处理结果描述，并且希望能够以json响应体来进行响应。

## Mapreduce

[参考](https://github.com/zeromicro/go-zero/blob/master/core/mr/readme-cn.md)

### 为什么需要 MapReduce

在实际的业务场景中我们常常需要**从不同的 rpc 服务中获取相应属性来组装成复杂对象**。

比如要查询商品详情：

1. 商品服务-查询商品属性
2. 库存服务-查询库存属性
3. 价格服务-查询价格属性
4. 营销服务-查询营销属性

如果是串行调用的话响应时间会随着 rpc 调用次数呈线性增长，所以我们要优化性能一般会将串行改并行。

简单的场景下使用 `WaitGroup` 也能够满足需求，但是如果我们需要**对 rpc 调用返回的数据进行校验、数据加工转换、数据汇总**呢？继续使用 `WaitGroup` 就有点力不从心了，go 的官方库中并没有这种工具（java 中提供了 CompleteFuture），我们依据 MapReduce 架构思想实现了进程内的数据批处理 MapReduce 并发工具类。

### 设计思路

我们尝试把自己代入到作者的角色梳理一下并发工具可能的业务场景：

1. 查询商品详情：支持并发调用多个服务来组合产品属性，支持调用错误可以立即结束。
2. 商品详情页自动推荐用户卡券：支持并发校验卡券，校验失败自动剔除，返回全部卡券。

以上实际都是在进行对输入数据进行处理最后输出清洗后的数据，针对数据处理有个非常经典的异步模式：生产者消费者模式。于是我们可以抽象一下数据批处理的生命周期，大致可以分为三个阶段：

[![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/mapreduce-serial-cn.png)](https://raw.githubusercontent.com/zeromicro/zero-doc/main/doc/images/mapreduce-serial-cn.png)

1. **数据生产 generate**
2. **数据加工 mapper**
3. **数据聚合 reducer**

其中**数据生产是不可或缺的阶段，数据加工、数据聚合是可选阶段，数据生产与加工支持并发调用**，数据聚合基本属于纯内存操作单协程即可。

再来思考一下不同阶段之间数据应该如何流转，既然不同阶段的数据处理都是由不同 goroutine 执行的，那么很自然的可以考虑采用 channel 来实现 goroutine 之间的通信。

[![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/mapreduce-cn.png)](https://raw.githubusercontent.com/zeromicro/zero-doc/main/doc/images/mapreduce-cn.png)

如何实现随时终止流程呢？

`goroutine` 中监听一个全局的结束 `channel` 和调用方提供的 `ctx` 就行。

### 简单示例

并行求平方和（不要嫌弃示例简单，只是模拟并发）

```
package main

import (
    "fmt"
    "log"

    "github.com/zeromicro/go-zero/core/mr"
)

func main() {
    val, err := mr.MapReduce(func(source chan<- interface{}) {
        // generator
        for i := 0; i < 10; i++ {
            source <- i
        }
    }, func(item interface{}, writer mr.Writer, cancel func(error)) {
        // mapper
        i := item.(int)
        writer.Write(i * i)
    }, func(pipe <-chan interface{}, writer mr.Writer, cancel func(error)) {
        // reducer
        var sum int
        for i := range pipe {
            sum += i.(int)
        }
        writer.Write(sum)
    })
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println("result:", val)
}
```

更多示例：https://github.com/zeromicro/zero-examples/tree/main/mapreduce

# 源码解析

[参考](https://learnku.com/articles/66596)、[参考](https://segmentfault.com/a/1190000041623963)

## 代码结构

### rest

rest 用于构建API服务

```
rest
├── handler // 自带中间件
│   ├── authhandler.go // 权限
│   ├── breakerhandler.go // 断路器
│   ├── contentsecurityhandler.go // 安全验证
│   ├── cryptionhandler.go // 加密解密
│   ├── gunziphandler.go // zip 压缩
│   ├── loghandler.go // 日志
│   ├── maxbyteshandler.go // 最大请求数据限制
│   ├── maxconnshandler.go // 最大请求连接数限制
│   ├── metrichandler.go // 请求指标统计
│   ├── prometheushandler.go // prometheus 上报
│   ├── recoverhandler.go // 错误捕获
│   ├── sheddinghandler.go // 过载保护
│   ├── timeouthandler.go // 超时控制
│   └── tracinghandler.go // 链路追踪
├── httpx
│   ├── requests.go
│   ├── responses.go
│   ├── router.go
│   ├── util.go
│   └── vars.go
├── internal
│   ├── cors // 跨域处理
│   │   └── handlers.go
│   ├── response
│   │   ├── headeronceresponsewriter.go
│   │   └── withcoderesponsewriter.go
│   ├── security // 加密处理
│   │   └── contentsecurity.go
│   ├── log.go
│   └── starter.go
├── pathvar // path 参数解析
│   └── params.go
├── router
│   └── patrouter.go
├── token
│   └── tokenparser.go
├── config.go // 配置
├── engine.go // 引擎
├── server.go
└── types.go
```

#### 服务启动流程

我们以 go-zero-example 项目 http/demo/main.go 代码来分析

![rest启动流程](https://segmentfault.com/img/remote/1460000041623966)

go-zero 给我们提供了如下组件与服务，我们来逐一阅读分析

- http框架常规组件（路由、调度器、中间件、跨域）
- 权限控制
- 断路器
- 限流器
- 过载保护
- prometheus
- trace
- cache

#### http框架常规组件

##### 路由

路由使用的是二叉查找树，高效的路由都会使用树形结构来构建
