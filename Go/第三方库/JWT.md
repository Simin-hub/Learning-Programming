# JWT

## JWT介绍

**JSON Web Token(JWT)是**一个开放标准(RFC 7519)，它定义了一种紧凑且独立的方式，可以在各方之间作为JSON对象安全地传输信息。此信息可以通过数字签名进行验证和信任。JWT可以使用秘密（使用HMAC算法）或使用RSA或ECDSA的公钥/私钥对进行签名。

## 什么场景适合用JWT

**授权：这是使用JWT的最常见方案**。一旦用户登录，每个后续请求将包括JWT，允许用户访问该令牌允许的路由，服务和资源。Single Sign On(单点登录)是一种现在广泛使用JWT的功能，因为它的开销很小，并且能够在不同的域中轻松使用。
信息交换：JWT是在各方之间安全传输信息的好方法。 因为JWT可以签名 - 例如，使用公钥/私钥对 - 您可以确定发件人是他们所说的人。此外，由于使用Header和payload,Signature，您还可以验证内容是否未被篡改。

### session

熟悉session运行机制的同学都知道，用户的session数据以file或缓存（redis、memcached）等方式存储在服务器端，客户端浏览器cookie中只保存sessionid。服务器端session属于集中存储，数量不大的情况下，没什么问题，当用户数据逐渐增多到一程度，就会给服务端管理和维护带来大的负担。

session有两个弊端：

1、**无法实现跨域。**

2、由于session数据属于集中管理里**，量大的时候服务器性能是个问题**。

优点：

1、session存在服务端，数据相对比较安全。

2、session集中管理也有好处，就是用户登录、注销服务端可控。

### cookie

cookie也是一种解决网站用户认证的实现方式，用户登录时，服务器会发送包含登录凭据的Cookie到用户浏览器客户端，浏览器会将Cookie的key/value保存用户本地（内存或硬盘），用户再访问网站，浏览器会发送cookie信息到服务器端，服务器端接收cookie并解析来维护用户的登录状态。

cookie避免session集中管理的问题，但也存在弊端：

1、跨域问题。

2、数据存储在浏览器端，数据容易被窃取及被csrf攻击，安全性差。

优点：

1、相对于session简单，不用服务端维护用户认证信息。

2、数据持久性。

### jwt

jwt通过json传输，php、java、golang等很多语言支持，通用性比较好，不存在跨域问题。传输数据通过数据签名相对比较安全。客户端与服务端通过jwt交互，服务端通过解密token信息，来实现用户认证。不需要服务端集中维护token信息，便于扩展。当然jwt也有其缺点。

缺点：

1、用户无法主动登出，只要token在有效期内就有效。这里可以考虑redis设置同token有效期一直的黑名单解决此问题。

2、token过了有效期，无法续签问题。可以考虑通过判断旧的token什么时候到期，过期的时候刷新token续签接口产生新token代替旧token。

## jwt设置有效期

可以设置有效期，加入有效期是为了增加安全性，即token被黑客截获，也只能攻击较短时间。设置有效期就会面临token续签问题，解决方案如下

通常**服务端设置两个token**

- Access Token：添加到 HTTP 请求的 header 中，进行用户认证，请求接口资源。
- refresh token：用于当 Access Token过期后，客户端传递refresh token刷新 Access Token续期接口，获取新的Access Token和refresh token。其有效期比 Access Token有效期长。

## jwt构成：

- Header：**TOKEN 的类型**，就是JWT，**签名的算法**，如 HMAC SHA256、HS384
- Payload：载荷又称为Claim，**携带的信息**，比如用户名、过期时间等，一般叫做 Claim
- Signature：签名，是**由header、payload 和你自己维护的一个 secret 经过加密**得来的

如下图 红色的为Header,指定token类型与签名类型,紫色的为载荷(playload),存储用户id等关键信息,最后蓝色的为签名,保证整个信息的完整性,可靠性。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/189732-b1a2ac306b37514d.png)

#### 头部（Header）

JWT的头部用于描述关于该JWT的最基本的信息，例如其类型以及签名所用的算法等。这可以被表示成一个JSON对象。



```json
{
  "typ": "JWT",
  "alg": "HS256"
}
```

在这里，我们说明了这是一个JWT，并且我们所用的签名算法是HS256算法。对它进行Base64编码，之后的字符串就成了JWT的Header（头部）。



```undefined
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9
```

#### 载荷(playload)

在载荷(playload)中定义了以下属性



```cpp
iss: 该JWT的签发者 
sub: 该JWT所面向的用户 
aud: 接收该JWT的一方 
exp(expires): 什么时候过期，这里是一个Unix时间戳 
iat(issued at): 在什么时候签发的
```

也可以用一个JSON对象来描述
将上面的JSON对象进行[base64编码]可以得到下面的字符串。这个字符串我们将它称作JWT的Payload（载荷）。



```undefined
eyJpc3MiOiIyOWZmMDE5OGJlOGM0YzNlYTZlZTA4YjE1MGRhNTU0NC1XRUIiLCJleHAiOjE1MjI0OTE5MTV9
```

#### 签名（Signature）

在官方文档中是如下描述的

```bash
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

将上面的两个[base64编码]后的字符串都用句号.连接在一起（头部在前），就形成了如下字符串。

```css
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiIyOWZmMDE5OGJlOGM0YzNlYTZlZTA4YjE1MGRhNTU0NC1XRUIiLCJleHAiOjE1MjI0OTE5MTV9
```

最后，我们将上面拼接完的字符串用HS256算法进行加密。在加密的时候，我们还需要提供一个密钥（secret）。那么就可以得到我们加密后的内容

```undefined
P-k-vIzxElzyzFbzR4tUxAAET8xT9EP49b7hpcPazd0 
```

这个就是我们JWT的签名了。

## 签名的目的

**最后一步签名的过程，实际上是对头部以及载荷内容进行签名**。一般而言，加密算法对于不同的输入产生的输出总是不一样的。
所以，**如果有人对头部以及载荷的内容解码之后进行修改，再进行编码的话，那么新的头部和载荷的签名和之前的签名就将是不一样的**。而且，如果不知道服务器加密的时候用的密钥的话，得出来的签名也一定会是不一样的。

服务器应用在接受到JWT后，会首先**对头部和载荷的内容用同一算法再次签名**。那么服务器应用是怎么知道我们用的是哪一种算法呢？别忘了，我们在JWT的头部中已经用alg字段指明了我们的加密算法了。

如果服务器应用对头部和载荷再次以同样方法签名之后发现，自己计算出来的签名和接受到的签名不一样，那么就说明这个Token的内容被别人动过的，我们应该拒绝这个Token，返回一个HTTP 401 Unauthorized响应。

## JWT的流程

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/189732-068ae45f8d91787e.png)



流程如下：

- 客户端使用账户密码请求登录接口
- 登录成功后服务器使用签名密钥生成JWT ,然后返回JWT给客户端。
- 客户端再次向服务端请求其他接口时带上JWT
- 服务端接收到JWT后验证签名的有效性.对客户端做出相应的响应

## jwt使用

这里推荐个使用比较多的开源项目[github.com/dgrijalva/jwt-go]()，[更多文档](https://link.segmentfault.com/?enc=dW79%2B6ro6mVWLWfSIAASwQ%3D%3D.mA4XxXYXEbfp6DFnrY4P7DwNtywfU6OPh9a2khqLjqzjCzt4zimAR%2FtWJMq9ew1G)。

示例：

```go
package main

import (
    "fmt"
    "github.com/dgrijalva/jwt-go"
    "time"
)
const (
    SECRETKEY = "243223ffslsfsldfl412fdsfsdf"//私钥
)
//自定义Claims
type CustomClaims struct {
    UserId int64
    jwt.StandardClaims
}
func main() {
    //生成token
    maxAge:=60*60*24
    customClaims :=&CustomClaims{
        UserId: 11,//用户id
        StandardClaims: jwt.StandardClaims{
            ExpiresAt: time.Now().Add(time.Duration(maxAge)*time.Second).Unix(), // 过期时间，必须设置
            Issuer:"jerry",   // 非必须，也可以填充用户名，
        },
    }
    //采用HMAC SHA256加密算法
    token:=jwt.NewWithClaims(jwt.SigningMethodHS256, customClaims)
    tokenString,err:= token.SignedString([]byte(SECRETKEY))
    if err!=nil {
        fmt.Println(err)
    }
    fmt.Printf("token: %v\n", tokenString)

    //解析token
    ret,err :=ParseToken(tokenString)
    if err!=nil {
        fmt.Println(err)
    }
    fmt.Printf("userinfo: %v\n", ret)
}

//解析token
func ParseToken(tokenString string)(*CustomClaims,error)  {
    token, err := jwt.ParseWithClaims(tokenString, &CustomClaims{}, func(token *jwt.Token) (interface{}, error) {
        if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
            return nil, fmt.Errorf("Unexpected signing method: %v", token.Header["alg"])
        }
        return []byte(SECRETKEY), nil
    })
    if claims, ok := token.Claims.(*CustomClaims); ok && token.Valid {
        return claims,nil
    } else {
        return nil,err
    }
}
```

运行结果：

token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJVc2VySWQiOjExLCJleHAiOjE1OTA5MTk1NDAsImlzcyI6ImplcnJ5In0.FppmbbHRrS4wd5wen73vYPOvtzycOrn2JZlK6JRjEGk
userinfo: &{11 { 1590919540 0 jerry 0 }}

#### Claims

```vbnet
Audience string `json:"aud,omitempty"`  
ExpiresAt int64 `json:"exp,omitempty"`  
Id string `json:"jti,omitempty"` 
IssuedAt int64 `json:"iat,omitempty"`  
Issuer string `json:"iss,omitempty"`  
NotBefore int64 `json:"nbf,omitempty"`  
Subject string `json:"sub,omitempty"`

aud: 接收jwt的一方
exp: jwt的过期时间，这个过期时间必须要大于签发时间
jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。
iat: jwt的签发时间
iss: jwt签发者
nbf: 定义在什么时间之前，该jwt都是不可用的.就是这条token信息生效时间.这个值可以不设置,但是设定后,一定要大于当前Unix UTC,否则token将会延迟生效.
sub: jwt所面向的用户
```

以上用到了CustomClaims，也可以用简单的方法

示例

```go
package main

import (
    "fmt"
    "github.com/dgrijalva/jwt-go"
    "time"
)
const (
    SECRETKEY = "243223ffslsfsldfl412fdsfsdf"//私钥
)
//自定义Claims
type CustomClaims struct {
    UserId int64
    jwt.StandardClaims
}
func main() {
    //生成token
    maxAge:=60*60*24
    // Create the Claims
    //claims := &jwt.StandardClaims{
    //    //    ExpiresAt: time.Now().Add(time.Duration(maxAge)*time.Second).Unix(), // 过期时间，必须设置,
    //    //    Issuer:    "jerry",// 非必须，也可以填充用户名，
    //    //}

    //或者用下面自定义claim
    claims := jwt.MapClaims{
        "id":       11,
        "name":       "jerry",
        "exp": time.Now().Add(time.Duration(maxAge)*time.Second).Unix(), // 过期时间，必须设置,
    }

    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    tokenString, err := token.SignedString([]byte(SECRETKEY))
    if err!=nil {
        fmt.Println(err)
    }
    fmt.Printf("token: %v\n", tokenString)

    //解析token
    ret,err :=ParseToken(tokenString)
    if err!=nil {
        fmt.Println(err)
    }
    fmt.Printf("userinfo: %v\n", ret)
}

//解析token
func ParseToken(tokenString string)(jwt.MapClaims,error)  {
    token, err := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
        // Don't forget to validate the alg is what you expect:
        if _, ok := token.Method.(*jwt.SigningMethodHMAC); !ok {
            return nil, fmt.Errorf("Unexpected signing method: %v", token.Header["alg"])
        }

        // hmacSampleSecret is a []byte containing your secret, e.g. []byte("my_secret_key")
        return []byte(SECRETKEY), nil
    })
    if claims, ok := token.Claims.(jwt.MapClaims); ok && token.Valid {
        return claims,nil
    } else {
        return nil,err
    }
}
```

运行结果类似

token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1OTA5MzUzMDUsImlkIjoxMSwibmFtZSI6ImplcnJ5In0.fapE0IiOEe_TqoMCThbNTHUvgWiHPEk0rm-9uPIcvPU
userinfo: map[exp:1.590935305e+09 id:11 name:jerry]

## 小结：

- 服务端生成的jwt返回客户端可以存到cookie也可以存到localStorage中（相比cookie容量大），存在cookie中需加上 `HttpOnly` 的标记，可以防止 [XSS](https://link.segmentfault.com/?enc=Hcins9wDoELY4IgQaO3UKw%3D%3D.QTikYkA48xWCWE%2F7XmnefG4IrPA1numipU1btYmXgbiQ6kRxVnO0e9vzEqZ9BIDr%2BLGQHcpFkw%2FYa%2Bcd1uJh6Zxh%2F4z%2BoYOXs17rdbX8iePJV7uHKvPmsyddRhaOnkil7Imkie9uRVw60SoMu%2B47kg%3D%3D)) 攻击。
- 尽量用https带证书网址访问。
- session和jwt没有绝对好与不好，各有其擅长的应用环境，请根据实际情况选择。