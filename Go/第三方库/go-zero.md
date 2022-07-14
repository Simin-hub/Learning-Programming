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
