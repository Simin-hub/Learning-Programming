# Go编程规范

[参考地址](https://www.kancloud.cn/gofor/golang-learn/2120522)

## 文件命名

1. 由于 Windows平台文件名不区分大小写，所以**文件名应一律使用小写**
2. 不同单词之间用**下划线分词**，**不要使用驼峰式命名**
3. 如果是测试文件，可以以`_test.go`结尾
4. 文件若具有平台特性，**应以`文件名_平台.go`命名**
   比如 utils_ windows.go，utils_linux.go，可用的平台有：windows, unix, posix, plan9, darwin, bsd, linux, freebsd, nacl, netbsd, openbsd, solaris, dragonfly, bsd, notbsd， android，stubs
5. 一般情况下**应用的主入口应为 main.go，或者以应用的全小写形式命名**。
   比如MyBlog 的入口可以为`myblog.go`

## 常量命名

主要有两种风格的写法

1. 第一种是驼峰命名法，比如 appVersion
2. 第二种**使用全大写且用下划线分词**，比如 APP_VERSION

```
const (
    APP_VERSION = "0.1.0"
    CONF_PATH = "/etc/xx.conf"
)
```

## 变量命名

和常量不同，变量的命名，**统一使用`驼峰命名法`**

1. 在相对简单的环境（对象数量少、针对性强）中，可以将完整单词简写为单个字母，例如：user写为u
2. **若该变量为 bool 类型，则名称应以`Has`,`Is`,`Can`或`Allow`开头**。例如：isExist ，hasConflict
3. **其他一般情况下首单词全小写，其后各单词首字母大写**。例如：numShips 和 startDate 。
4. 若变量中有特有名词（以下列出），且变量为私有，则首单词还是使用全小写，如`apiClient`。
5. 若变量中有特有名词（以下列出），但变量不是私有，那首单词就要变成全大写。例如：`APIClient`，`URLString`

这里列举了一些常见的特有名词：

```
// A GonicMapper that contains a list of common initialisms taken from golang/lint
var LintGonicMapper = GonicMapper{
    "API":   true,
    "ASCII": true,
    "CPU":   true,
    "CSS":   true,
    "DNS":   true,
    "EOF":   true,
    "GUID":  true,
    "HTML":  true,
    "HTTP":  true,
    "HTTPS": true,
    "ID":    true,
    "IP":    true,
    "JSON":  true,
    "LHS":   true,
    "QPS":   true,
    "RAM":   true,
    "RHS":   true,
    "RPC":   true,
    "SLA":   true,
    "SMTP":  true,
    "SSH":   true,
    "TLS":   true,
    "TTL":   true,
    "UI":    true,
    "UID":   true,
    "UUID":  true,
    "URI":   true,
    "URL":   true,
    "UTF8":  true,
    "VM":    true,
    "XML":   true,
    "XSRF":  true,
    "XSS":   true,
}
```

## 函数命名

1. 函数名使用 **驼峰命名法**
2. 有一点需要注意，在 Golang 中是用大小写来控制函数的可见性，因此当需要在包外访问，使用大写字母开头
3. 当你不需要在包外访问，使用小写字母开头

另外，函数内部的参数的排列顺序也有几点原则

1. **参数的重要程度越高，应排在越前面**
2. **简单的类型应优先复杂类型**
3. **尽可能将同种类型的参数放在相邻位置，则只需写一次类型**

## 接口命名

**使用驼峰命名法**，可以用 type alias 来定义大写开头的 type 给包外访问

```
type helloWorld interface {
    func Hello();
}

type SayHello helloWorld

```

当你的接口只有一个函数时，接口名通常会以 er 为后缀

```
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

## 注释规范

### 包注释

1. **位于 package 之前**，如果一个包有多个文件，**只需要在一个文件中编写**即可
2. 如果你想在**每个文件中的头部加上注释，需要在版权注释和 Package前面加一个空行**，否则版权注释会作为Package的注释。

```
// Copyright 2009 The Go Authors. All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.

package net
```

1. 如果是特别复杂的包，可单独创建 doc.go 文件说明

### 代码注释

用于解释代码逻辑，可以有两种写法

单行注释使用`//`，多行注释使用`/*comment*/`

```
// 单行注释

/*
多
行
注
释
*/
```

### 6.3 特别注释

- TODO：提醒维护人员此部分代码待完成
- FIXME：提醒维护人员此处有BUG待修复
- NOTE：维护人员要关注的一些问题说明

## 7. 包的导入

单行的包导入

```
import "fmt"
```

多个包导入，请使用`()`来组织

```
import (
  "fmt"
  "os"
)
```

另外根据包的来源，对排版还有一定的要求

1. **标准库排最前面**，**第三方包次之**、**项目内的其它包**和**当前包的子包排最后**，**每种分类以一空行分隔**。
2. 尽量不要使用相对路径来导入包

```
import (
    "fmt"
    "html/template"
    "net/http"
    "os"

    "github.com/codegangsta/cli"
    "gopkg.in/macaron.v1"

    "github.com/gogits/git"
    "github.com/gogits/gfm"

    "github.com/gogits/gogs/routers"
    "github.com/gogits/gogs/routers/repo"
    "github.com/gogits/gogs/routers/user"
)
```