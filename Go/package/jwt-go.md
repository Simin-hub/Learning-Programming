# jwt-go

[参考](https://cloud.tencent.com/developer/article/1770768)

[参考2](https://bgbiao.top/post/golang-jwt%E7%9A%84%E4%BD%BF%E7%94%A8/)

### Json-Web-Token(JWT)介绍

一般而言，用户注册登陆后会生成一个jwt token返回给浏览器，浏览器向服务端请求数据时携带`token`，服务器端使用`signature`中定义的方式进行解码，进而对token进行解析和验证。

#### JWT Token组成部分

![JWT-Token组成部分](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/00831rSTly1gcpwngvonhj30hx03xwf0.jpg)

- header: **用来指定使用的算法**(HMAC SHA256 RSA)和token类型(如JWT)
- payload: **包含声明(要求)**，声明通常是用户信息或其他数据的声明，比如用户id，名称，邮箱等. 声明可分为三种: registered,public,private
- signature: **用来保证JWT的真实性，可以使用不同的算法**

##### **header**

```
{
    "alg": "HS256",
    "typ": "JWT"
}
```

对上面的json进行base64编码即可得到JWT的第一个部分

typ用来**标识整个token是一个jwt字符串**，**alg代表签名和摘要算法**，一般签发JWT的时候，只要typ和alg就够了，生成方式是将header部分的json字符串经过Base64Url编码：

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/5000473-166f479fbdb63fb6.png)

##### **payload**

- registered claims: **预定义的声明，通常会放置一些预定义字段**，比如过期时间，主题等(iss:issuer,exp:expiration time,sub:subject,aud:audience)
- public claims: **可以设置公开定义的字段**
- private claims: **用于统一使用他们的各方之间的共享信息**

```
{
    "sub": "xxx-api",
    "name": "bgbiao.top",
    "admin": true
}
```

对payload部分的json进行base64编码后即可得到JWT的第二个部分

`注意:` 不要在header和payload中放置敏感信息，除非信息本身已经做过脱敏处理

**playload用来承载要传递的数据，它的一个属性对被称为claim，这样的标准成为claims标准**，同样是将其用[Base64Url编码](https://www.zongscan.com/demo333/365.html)

##### **signature**

**signature**, 签名

为了得到签名部分，必须有**编码过的header和payload，以及一个秘钥**，**签名算法使用header中指定的那个**，然后对其进行签名即可

```
HMACSHA256(base64UrlEncode(header)+"."+base64UrlEncode(payload),secret)
```

签名是`用于验证消息在传递过程中有没有被更改`，并且，对于使用私钥签名的token，它还可以验证JWT的发送方是否为它所称的发送方。

在[jwt.io](https://jwt.io/)网站中，提供了一些JWT token的编码，验证以及生成jwt的工具。

下图就是一个典型的jwt-token的组成部分。

![jwt官方签名结构](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/00831rSTly1gcpxe1ungej30wq0mygo8.jpg)

#### 什么时候用JWT

- Authorization(授权): 典型场景，**用户请求的token中包含了该令牌允许的路由，服务和资源**。单点登录其实就是现在广泛使用JWT的一个特性
- Information Exchange(信息交换): 对于安全的在各方之间传输信息而言，JSON Web Tokens无疑是一种很好的方式.因为JWTs可以被签名，例如，用公钥/私钥对，你可以确定发送人就是它们所说的那个人。另外，由于签名是使用头和有效负载计算的，您还可以验证内容没有被篡改

#### JWT(Json Web Tokens)是如何工作的

![JWT认证过程](https://tva1.sinaimg.cn/large/00831rSTly1gcptxpqcrxj30ct0bsq3h.jpg)

所以，基本上整个过程分为两个阶段：

- 第一个阶段，客户端向服务端获取token
- 第二阶段，客户端带着该token去请求相关的资源.

通常比较重要的是，**服务端如何根据指定的规则进行token的生成**。

在认证的时候，当用户用他们的凭证成功登录以后，一个JSON Web Token将会被返回。

此后，token就是用户凭证了，你必须非常小心以防止出现安全问题。

一般而言，你保存令牌的时候不应该超过你所需要它的时间。

无论何时用户想要访问受保护的路由或者资源的时候，用户代理（通常是浏览器）都应该带上JWT，典型的，通常放在Authorization header中，用Bearer schema: `Authorization: Bearer <token>`

服务器上的受保护的路由将会检查Authorization header中的JWT是否有效，如果有效，则用户可以访问受保护的资源。如果JWT包含足够多的必需的数据，那么就可以减少对某些操作的数据库查询的需要，尽管可能并不总是如此。

**如果token是在授权头（Authorization header）中发送**的，那么跨源资源共享(CORS)将不会成为问题，因为它不使用cookie.

![获取JWT以及访问APIs以及资源](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/00831rSTly1gcpxi6z1dmj315o0fvab0.jpg)

- 客户端向授权接口请求授权
- 服务端授权后返回一个access token给客户端
- 客户端使用access token访问受保护的资源

#### 基于Token的身份认证和基于服务器的身份认证

**1.基于服务器的认证**

前面说到过session，cookie以及token的区别，在之前传统的做法就是基于存储在服务器上的session来做用户的身份认证，但是通常会有如下问题:

- Sessions: 认证通过后需要将用户的session数据保存在内存中，随着认证用户的增加，内存开销会大
- 扩展性: 由于session存储在内存中，扩展性会受限，虽然后期可以使用redis,memcached来缓存数据
- CORS: 当多个终端访问同一份数据时，可能会遇到禁止请求的问题
- CSRF: 用户容易受到CSRF攻击

**2.Session和JWT Token的异同**

都可以存储用户相关信息，但是session存储在服务端，JWT存储在客户端

![session和jwt数据存储位置](https://tva1.sinaimg.cn/large/00831rSTly1gcpxunv5qcj30dd0gfgmp.jpg)

**3.基于Token的身份认证如何工作**

基于Token的身份认证是无状态的，服务器或者session中不会存储任何用户信息.(很好的解决了共享session的问题)

- 用户携带用户名和密码请求获取token(接口数据中可使用appId,appKey)
- 服务端校验用户凭证，并返回用户或客户端一个Token
- 客户端存储token,并在请求头中携带Token
- 服务端校验token并返回数据

注意:

- 随后客户端的每次请求都需要使用token
- token应该放在header中
- 需要将服务器设置为接收所有域的请求: `Access-Control-Allow-Origin: *`

**4.用Token的好处**

- 无状态和可扩展性
- 安全: 防止CSRF攻击;token过期重新认证

**5.JWT和OAuth的区别**

1. **OAuth2是一种授权框架 ，JWT是一种认证协议**
2. 无论使用哪种方式切记用HTTPS来保证数据的安全性
3. OAuth2用在`使用第三方账号登录的情况`(比如使用weibo, qq, github登录某个app)，而`JWT是用在前后端分离`, 需要简单的对后台API进行保护时使用

## 安装和使用

```cmd
go get -u github.com/dgrijalva/jwt-go@latest
```

### 使用Gin框架集成JWT

在Golang语言中，[jwt-go](https://github.com/dgrijalva/jwt-go)库提供了一些jwt编码和验证的工具，因此我们很容易使用该库来实现token认证。

另外，我们也知道[gin](https://github.com/gin-gonic/gin)框架中支持用户自定义middleware，我们可以很好的将jwt相关的逻辑封装在middleware中，然后对具体的接口进行认证。

#### 自定义中间件

在gin框架中，自定义中间件比较容易，只要返回一个`gin.HandlerFunc`即完成一个中间件定义。

接下来，我们先定义一个用于jwt认证的中间件.

```
// 定义一个JWTAuth的中间件
func JWTAuth() gin.HandlerFunc {
	return func(c *gin.Context) {
		// 通过http header中的token解析来认证
		token := c.Request.Header.Get("token")
		if token == "" {
			c.JSON(http.StatusOK, gin.H{
				"status": -1,
				"msg":    "请求未携带token，无权限访问",
				"data":   nil,
			})
			c.Abort()
			return
		}

		log.Print("get token: ", token)

		// 初始化一个JWT对象实例，并根据结构体方法来解析token
		j := NewJWT()
		// 解析token中包含的相关信息(有效载荷)
		claims, err := j.ParserToken(token)

		if err != nil {
			// token过期
			if err == TokenExpired {
				c.JSON(http.StatusOK, gin.H{
					"status": -1,
					"msg":    "token授权已过期，请重新申请授权",
					"data":   nil,
				})
				c.Abort()
				return
			}
			// 其他错误
			c.JSON(http.StatusOK, gin.H{
				"status": -1,
				"msg":    err.Error(),
				"data":   nil,
			})
			c.Abort()
			return
		}

		// 将解析后的有效载荷claims重新写入gin.Context引用对象中
		c.Set("claims", claims)

	}
}
```

#### 定义jwt编码和解码逻辑

根据前面提到的jwt-token的组成部分，以及`jwt-go`中相关的定义，我们可以使用如下方法进行生成token.

```
// 定义一个jwt对象
type JWT struct {
	// 声明签名信息
	SigningKey []byte
}

// 初始化jwt对象
func NewJWT() *JWT {
	return &JWT{
		[]byte("bgbiao.top"),
	}
}

// 自定义有效载荷(这里采用自定义的Name和Email作为有效载荷的一部分)
type CustomClaims struct {
	Name  string `json:"name"`
	Email string `json:"email"`
	// StandardClaims结构体实现了Claims接口(Valid()函数)
	jwt.StandardClaims
}


// 调用jwt-go库生成token
// 指定编码的算法为jwt.SigningMethodHS256
func (j *JWT) CreateToken(claims CustomClaims) (string, error) {
	// https://gowalker.org/github.com/dgrijalva/jwt-go#Token
	// 返回一个token的结构体指针
	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
	return token.SignedString(j.SigningKey)
}


// token解码
func (j *JWT) ParserToken(tokenString string) (*CustomClaims, error) {
	// https://gowalker.org/github.com/dgrijalva/jwt-go#ParseWithClaims
	// 输入用户自定义的Claims结构体对象,token,以及自定义函数来解析token字符串为jwt的Token结构体指针
	// Keyfunc是匿名函数类型: type Keyfunc func(*Token) (interface{}, error)
	// func ParseWithClaims(tokenString string, claims Claims, keyFunc Keyfunc) (*Token, error) {}
	token, err := jwt.ParseWithClaims(tokenString, &CustomClaims{}, func(token *jwt.Token) (interface{}, error) {
		return j.SigningKey, nil
	})

	if err != nil {
		// https://gowalker.org/github.com/dgrijalva/jwt-go#ValidationError
		// jwt.ValidationError 是一个无效token的错误结构
		if ve, ok := err.(*jwt.ValidationError); ok {
			// ValidationErrorMalformed是一个uint常量，表示token不可用
			if ve.Errors&jwt.ValidationErrorMalformed != 0 {
				return nil, fmt.Errorf("token不可用")
				// ValidationErrorExpired表示Token过期
			} else if ve.Errors&jwt.ValidationErrorExpired != 0 {
				return nil, fmt.Errorf("token过期")
				// ValidationErrorNotValidYet表示无效token
			} else if ve.Errors&jwt.ValidationErrorNotValidYet != 0 {
				return nil, fmt.Errorf("无效的token")
			} else {
				return nil, fmt.Errorf("token不可用")
			}

		}
	}

	// 将token中的claims信息解析出来并断言成用户自定义的有效载荷结构
	if claims, ok := token.Claims.(*CustomClaims); ok && token.Valid {
		return claims, nil
	}

	return nil, fmt.Errorf("token无效")

}
```

#### 定义登陆验证逻辑

接下来的部分就是普通api的具体逻辑了，比如可以在登陆时进行用户校验，成功后未该次认证请求生成token。

```
// 定义一个普通controller函数，作为一个验证接口逻辑
func GetDataByTime(c *gin.Context) {
	// 上面我们在JWTAuth()中间中将'claims'写入到gin.Context的指针对象中，因此在这里可以将之解析出来
	claims := c.MustGet("claims").(*md.CustomClaims)
	if claims != nil {
		c.JSON(http.StatusOK, gin.H{
			"status": 0,
			"msg":    "token有效",
			"data":   claims,
		})
	}
}


// 在主函数中定义路由规则
	router := gin.Default()
	v1 := router.Group("/apis/v1/")
	{
		v1.POST("/register", controller.RegisterUser)
		v1.POST("/login", controller.Login)
	}

	// secure v1
	sv1 := router.Group("/apis/v1/auth/")
	// 加载自定义的JWTAuth()中间件,在整个sv1的路由组中都生效
	sv1.Use(md.JWTAuth())
	{
		sv1.GET("/time", controller.GetDataByTime)

	}
	router.Run(":8081")

```

#### 验证使用JWT后的接口

```
# 运行项目
$ go run main.go
127.0.0.1
13306
root:bgbiao.top@tcp(127.0.0.1:13306)/test_api?charset=utf8mb4&parseTime=True&loc=Local
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] POST   /apis/v1/register         --> warnning-trigger/controller.RegisterUser (3 handlers)
[GIN-debug] POST   /apis/v1/login            --> warnning-trigger/controller.Login (3 handlers)
[GIN-debug] GET    /apis/v1/auth/time        --> warnning-trigger/controller.GetDataByTime (4 handlers)
[GIN-debug] Listening and serving HTTP on :8081

# 注册用户
$ curl -i -X POST \
   -H "Content-Type:application/json" \
   -d \
'{
  "name": "hahaha1",
  "password": "hahaha1",
  "email": "hahaha1@bgbiao.top",
  "phone": 10000000000
}' \
 'http://localhost:8081/apis/v1/register'
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: Sun, 15 Mar 2020 07:09:28 GMT
Content-Length: 41

{"data":null,"msg":"success ","status":0}%


# 登陆用户以获取token
$ curl -i -X POST \
   -H "Content-Type:application/json" \
   -d \
'{
  "name":"hahaha1",
  "password":"hahaha1"
}' \
 'http://localhost:8081/apis/v1/login'
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: Sun, 15 Mar 2020 07:10:41 GMT
Content-Length: 290

{"data":{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyTmFtZSI6ImhhaGFoYTEiLCJlbWFpbCI6ImhhaGFoYTFAYmdiaWFvLnRvcCIsImV4cCI6MTU4NDI1OTg0MSwiaXNzIjoiYmdiaWFvLnRvcCIsIm5iZiI6MTU4NDI1NTI0MX0.HNXSKISZTqzjKd705BOSARmgI8FGGe4Sv-Ma3_iK1Xw","name":"hahaha1"},"msg":"登陆成功","status":0}


# 访问需要认证的接口
# 因为我们对/apis/v1/auth/的分组路由中加载了jwt的middleware，因此该分组下的api都需要使用jwt-token认证
$ curl http://localhost:8081/apis/v1/auth/time
{"data":null,"msg":"请求未携带token，无权限访问","status":-1}%

# 使用token认证
$ curl http://localhost:8081/apis/v1/auth/time -H 'token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyTmFtZSI6ImhhaGFoYTEiLCJlbWFpbCI6ImhhaGFoYTFAYmdiaWFvLnRvcCIsImV4cCI6MTU4NDI1OTg0MSwiaXNzIjoiYmdiaWFvLnRvcCIsIm5iZiI6MTU4NDI1NTI0MX0.HNXSKISZTqzjKd705BOSARmgI8FGGe4Sv-Ma3_iK1Xw'
{"data":{"userName":"hahaha1","email":"hahaha1@bgbiao.top","exp":1584259841,"iss":"bgbiao.top","nbf":1584255241},"msg":"token有效","status":0}%

```

### 第二种使用jwt-go 进行解析

#### 	生成 token

使用 jwt-go 库生成 token，我们需要**定义需求（claims）**，也就是说我们需要通过 jwt 传输的数据。假如我们需要传输 ID 和 Username，我们可以定义 Claims 结构体，其中包含 ID 和 Username 字段，还有在 jwt-go 包预定义的 jwt.StandardClaims。

```
type Claims struct {
  ID       int64
  Username string
  jwt.StandardClaims
}
```

jwt.StandardClaims 包含的字段：

```javascript
type StandardClaims struct {
  Audience  string `json:"aud,omitempty"`
  ExpiresAt int64  `json:"exp,omitempty"`
  Id        string `json:"jti,omitempty"`
  IssuedAt  int64  `json:"iat,omitempty"`
  Issuer    string `json:"iss,omitempty"`
  NotBefore int64  `json:"nbf,omitempty"`
  Subject   string `json:"sub,omitempty"`
}
```

其中 ExpiresAt 和 Issuer 分别是过期时间和签发者。

**使用 jwt-go 库根据指定的算法生成 jwt token** ，主要用到两个方法：

jwt.NewWithClaims 方法：

```javascript
func jwt.NewWithClaims(method jwt.SigningMethod, claims jwt.Claims) *jwt.Token
```

jwt.NewWithClaims 方法根据 Claims 结构体创建 Token 示例。

参数 1 是 jwt.SigningMethod，

其中包含

jwt.SigningMethodHS256，

jwt.SigningMethodHS384，

jwt.SigningMethodHS512 

三种 crypto.Hash 加密算法的方案。

参数 2 是 Claims，**包含自定义类型和 StandardClaim，StandardClaim 嵌入在自定义类型中**，以方便对标准声明进行编码，解析和验证。

SignedString 方法：

```javascript
func (*jwt.Token).SignedString(key interface{}) (string, error)
```

SignedString 方法根据传入的空接口类型参数 key，返回完整的签名令牌。

示例代码：

```javascript
func GenerateToken() (string, error) {
  nowTime := time.Now()
  expireTime := nowTime.Add(300 * time.Second)
  issuer := "frank"
  claims := MyCustomClaims{
    ID:       10001,
    Username: "frank",
    StandardClaims: jwt.StandardClaims{
      ExpiresAt: expireTime.Unix(),
      Issuer:    issuer,
    },
  }

  token, err := jwt.NewWithClaims(jwt.SigningMethodHS256, claims).SignedString([]byte("golang"))
  return token, err
}
```

#### 解析 token

使用 jwt-go 库解析 token，主要用到两个方法，分别用通过与解析传入的 token 字符串，和根据 MyCustomClaims 结构体定义的相关属性要求进行校验。

jwt.ParseWithClaims 方法：

```javascript
func jwt.ParseWithClaims(tokenString string, claims jwt.Claims, keyFunc jwt.Keyfunc) (*jwt.Token, error)
```

jwt.ParseWithClaims 方法用于解析鉴权的声明，返回 *jwt.Token。

Valid 方法用于校验鉴权的声明。

示例代码：

```javascript
func ParseToken(token string) (*MyCustomClaims, error) {
  tokenClaims, err := jwt.ParseWithClaims(token, &MyCustomClaims{}, func(token *jwt.Token) (interface{}, error) {
    return []byte("golang"), nil
  })
  if err != nil {
    return nil, err
  }

  if tokenClaims != nil {
    if claims, ok := tokenClaims.Claims.(*MyCustomClaims); ok && tokenClaims.Valid {
      return claims, nil
    }
  }

  return nil, err
}
```
