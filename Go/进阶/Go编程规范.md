# Go编程规范

[参考地址](https://www.kancloud.cn/gofor/golang-learn/2120522)

## 命名规范

### 文件命名

1. 由于 Windows平台文件名不区分大小写，所以**文件名应一律使用小写**
2. 不同单词之间用**下划线分词**，**不要使用驼峰式命名**
3. 如果是测试文件，可以以`_test.go`结尾
4. 文件若具有平台特性，**应以`文件名_平台.go`命名**
   比如 utils_ windows.go，utils_linux.go，可用的平台有：windows, unix, posix, plan9, darwin, bsd, linux, freebsd, nacl, netbsd, openbsd, solaris, dragonfly, bsd, notbsd， android，stubs
5. 一般情况下**应用的主入口应为 main.go，或者以应用的全小写形式命名**。
   比如MyBlog 的入口可以为`myblog.go`

### 常量命名

主要有两种风格的写法

1. 第一种是驼峰命名法，比如 appVersion
2. 第二种**使用全大写且用下划线分词**，比如 APP_VERSION

```
const (
    APP_VERSION = "0.1.0"
    CONF_PATH = "/etc/xx.conf"
)
```

### 变量命名

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

### 函数命名

1. 函数名使用 **驼峰命名法**
2. 有一点需要注意，在 Golang 中是用大小写来控制函数的可见性，因此当需要在包外访问，使用大写字母开头
3. 当你不需要在包外访问，使用小写字母开头

另外，函数内部的参数的排列顺序也有几点原则

1. **参数的重要程度越高，应排在越前面**
2. **简单的类型应优先复杂类型**
3. **尽可能将同种类型的参数放在相邻位置，则只需写一次类型**

### 接口命名

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

### 注释规范

#### 包注释

1. **位于 package 之前**，如果一个包有多个文件，**只需要在一个文件中编写**即可
2. 如果你想在**每个文件中的头部加上注释，需要在版权注释和 Package前面加一个空行**，否则版权注释会作为Package的注释。

```
// Copyright 2009 The Go Authors. All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.

package net
```

1. 如果是特别复杂的包，可单独创建 doc.go 文件说明

#### **代码注释**

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

#### 6.3 特别注释

- TODO：提醒维护人员此部分代码待完成
- FIXME：提醒维护人员此处有BUG待修复
- NOTE：维护人员要关注的一些问题说明

## 包的导入

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

## 代码优化点

### 使用pkg/error而不是官方error库

其实我们可以思考一下，我们在一个项目中使用错误机制，最核心的几个需求是什么？

1 附加信息：我们希望错误出现的时候能附带一些描述性的错误信息，甚至于这些信息是可以嵌套的。

2 附加堆栈：我们希望错误不仅仅打印出错误信息，也能打印出这个错误的堆栈信息，让我们可以知道错误的信息。

在Go的语言演进过程中，error传递的信息太少一直是被诟病的一点。我推荐在应用层使用 github.com/pkg/errors 来替换官方的error库。

假设我们有一个项目叫errdemo，他有sub1,sub2两个子包。sub1和sub2两个包都有Diff和IoDiff两个函数。

![image-20211219170503931](https://img2022.cnblogs.com/blog/136188/202203/136188-20220329094419544-2024043673.png)

```go
// sub2.go
package sub2
import (
    "errors"
    "io/ioutil"
)
func Diff(foo int, bar int) error {
    return errors.New("diff error")
}


// sub1.go
package sub1

import (
    "errdemo/sub1/sub2"
    "fmt"
    "errors"
)
func Diff(foo int, bar int) error {
    if foo < 0 {
        return errors.New("diff error")
    }
    if err := sub2.Diff(foo, bar); err != nil {
        return err
    }
    return nil
}

// main.go
package main

import (
    "errdemo/sub1"
    "fmt"
)
func main() {
    err := sub1.Diff(1, 2)
    fmt.Println(err)
}
```

在上述三段代码中，我们很不幸地将sub1.go中的Diff返回的error和sub2.go中Diff返回的error都定义为同样的字符串“diff error”。这个时候，在main.go中，我们返回的error，是无论如何也判断不出这个error是从sub1 还是 sub2 中抛出的。调试的时候会带来很大的困扰。

![image-20211219171226288](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/136188-20220329094419503-149190441.png)

而使用 github.com/pkg/errors ，我们所有的代码都不需要进行修改，只需要将import地方进行对应的修改即可。

在main.go中使用`fmt.Printf("%+v", err)` 就能除了打印error的信息，也能将堆栈打印出来了。

```go
// sub2.go
package sub2
import (
    "github.com/pkg/errors"
    "io/ioutil"
)
func Diff(foo int, bar int) error {
    return errors.New("diff error")
}


// sub1.go
package sub1

import (
    "errdemo/sub1/sub2"
    "fmt"
    "github.com/pkg/errors"
)
func Diff(foo int, bar int) error {
    if foo < 0 {
        return errors.New("diff error")
    }
    if err := sub2.Diff(foo, bar); err != nil {
        return err
    }
    return nil
}

// main.go
package main

import (
    "errdemo/sub1"
    "fmt"
)
func main() {
    err := sub1.Diff(1, 2)
    fmt.Printf("%+v", err)
}
```

![image-20211219171614767](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/136188-20220329094419516-526440396.png)

看到，除了"diff error" 的错误信息之外，还将堆栈大衣拿出来了，我们能明确看到是sub2.go中第7行抛出的错误。

其实 github.com/pkg/errors 的原理也是非常简单，它利用了fmt包的一个特性：

其中在打印error之前会判断当前打印的对象是否实现了Formatter接口，这个formatter接口只有一个format方法

![image-20211219171930031](https://img2022.cnblogs.com/blog/136188/202203/136188-20220329094419699-2023034924.png)

所以在 github.com/pkg/errors 中提供的各种初始化error方法（包括errors.New）就是封装了一个fundamental 结构，这个结构中带着error的信息和堆栈信息

![image-20211219172218939](https://img2022.cnblogs.com/blog/136188/202203/136188-20220329094419536-1711916463.png)

它实现了Format方法。

![image-20211219172234195](https://img2022.cnblogs.com/blog/136188/202203/136188-20220329094419515-314943379.png)

### 在初始化slice的时候尽量补全cap

当我们要创建一个slice结构，并且往slice中append元素的时候，我们可能有两种写法来初始化这个slice。

方法1:

```go
package main

import "fmt"

func main() {
	arr := []int{}
	arr = append(arr, 1,2,3,4, 5)
	fmt.Println(arr)
}
```

方法2:

```go
package main

import "fmt"

func main() {
   arr := make([]int, 0, 5)
   arr = append(arr, 1,2,3,4, 5)
   fmt.Println(arr)
}
```

方法2相较于方法1，就只有一个区别：在初始化[]int slice的时候在make中设置了cap的长度，就是slice的大小。

这两种方法对应的功能和输出结果是没有任何差别的，但是实际运行的时候，方法2会比少运行了一个growslice的命令。

这个我们可以通过打印汇编码进行查看：

方法1：

![image-20211219173237557](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/136188-20220329094419545-372103676.png)

方法2:

![image-20211219174112164](https://img2022.cnblogs.com/blog/136188/202203/136188-20220329094419693-898590550.png)

我们看到方法1中使用了growsslice方法，而方法2中是没有调用这个方法的。

**这个growslice的作用就是扩充slice的容量大小**。就好比是原先我们没有定制容量，系统给了我们一个能装两个鞋子的盒子，但是当我们装到第三个鞋子的时候，这个盒子就不够了，我们就要换一个盒子，而换这个盒子，我们势必还需要将原先的盒子里面的鞋子也拿出来放到新的盒子里面。所以这个growsslice的操作是一个比较复杂的操作，它的表现和复杂度会高于最基本的初始化make方法。对追求性能的程序来说，应该能避免尽量避免。

具体对growsslice函数具体实现同学有兴趣的可以参考源码src的 runtime/slice.go 。

当然，我们并不是每次都能在slice初始化的时候就能准确预估到最终的使用容量的。所以这里使用了一个“尽量”。明白是否设置slice容量的区别，我们在能预估容量的时候，请尽量使用方法2那种预估容量后的slice初始化方式。

### 初始化一个类的时候，如果类的构造参数较多，尽量使用Option写法

我们一定遇到需要初始化一个类的时候，大部分的时候，初始化一个类我们会使用类似下列的New方法。

```go
package newdemo

type Foo struct {
   name string
   id int
   age int

   db interface{}
}

func NewFoo(name string, id int, age int, db interface{}) *Foo {
   return &Foo{
      name: name,
      id:   id,
      age:  age,
      db:   db,
   }
}
```

我们定义一个NewFoo方法，其中存放初始化Foo结构所需要的各种字段属性。

这个写法乍看之下是没啥问题的，但是一旦Foo结构内部的字段进行了变化，增加或者减少了，那么这个初始化函数NewFoo就怎么看怎么别扭了。参数继续增加？那么所有调用方的地方也都需要进行修改了，且按照代码整洁的逻辑，参数多于5个，这个函数就很难使用了。而且，如果这5个参数都是可有可无的参数，就是有的参数可以允许不填写，有默认值，比如age这个字段，如果不填写，在后续的业务逻辑中可能没有很多影响，那么我在实际调用NewFoo的时候，age这个字段还需要传递0值。

```go
foo := NewFoo("jianfengye", 1, 0, nil)
```

这种语意逻辑就不对了。

这里其实有一种更好的写法：使用Option写法来进行改造**。Option写法顾命思议，将所有可选的参数作为一个可选方式，一般我们会一定一个“函数类型”来代表这个Option**，然后配套将所有可选字段设计一个这个函数类型的具体实现。而在具体的使用的时候，使用可变字段的方式来控制有多少个函数类型会被执行。比如上述的代码，我们会改造为：

```go
type Foo struct {
	name string
	id int
	age int

	db interface{}
}

// FooOption 代表可选参数
type FooOption func(foo *Foo)

// WithName 代表Name为可选参数
func WithName(name string) FooOption {
   return func(foo *Foo) {
      foo.name = name
   }
}

// WithAge 代表age为可选参数
func WithAge(age int) FooOption {
   return func(foo *Foo) {
      foo.age = age
   }
}

// WithDB 代表db为可选参数
func WithDB(db interface{}) FooOption {
   return func(foo *Foo) {
      foo.db = db
   }
}

// NewFoo 代表初始化
func NewFoo(id int, options ...FooOption) *Foo {
   foo := &Foo{
      name: "default",
      id:   id,
      age:  10,
      db:   nil,
   }
   for _, option := range options {
      option(foo)
   }
   return foo
}
```

解释下上面的这段代码，我们创建了一个FooOption的函数类型，这个函数类型代表的函数结构是 `func(foo *Foo)` ，很简单，将foo指针传递进去，能让内部函数进行修改。

然后我们定义了三个返回了FooOption的函数：

- WithName
- WithAge
- WithDB

以WithName为例，这个函数参数为string，返回值为FooOption。在返回值的FooOption中，根据参数修改了Foo指针。

```go
// WithName 代表Name为可选参数
func WithName(name string) FooOption {
   return func(foo *Foo) {
      foo.name = name
   }
}
```

顺便说一下，这种函数我们一般都以With开头，表示我这次初始化“带着”这个字段。

而最后NewFoo函数，参数我们就改造为两个部分，一个部分是“非Option”字段，就是必填字段，假设我们的Foo结构实际上只有一个必填字段id，而其他字段皆是选填的。而其他所有选填字段，我们使用一个可变参数 options 替换。

```objectivec
NewFoo(id int, options ...FooOption)
```

在具体的实现中，也变化成2个步骤：

- 按照默认值初始化一个foo对象
- 遍历options改造这个foo对象

按照这样改造之后，我们具体使用Foo结构的函数就变为如下样子：

```css
// 具体使用NewFoo的函数
func Bar() {
   foo := NewFoo(1, WithAge(15), WithName("foo"))
   fmt.Println(foo)
}
```

可读性是否高了很多？New一个Foo结构，id为1，并且带着指定age为15，指定name为“foo”。

后续如果Foo多了一个可变属性，那么只需要多一个WithXXX的方法，而NewFoo函数不需要任何变化，调用方只有需要指定这个可变属性的地方增加WithXXX即可。扩展性非常好。

这种Option的写法在很多著名的库中都有使用到，gorm, go-redis等。所以我们要把这种方式熟悉起来，**一旦我们在需要对一个比较复杂的类进行初始化的时候**，这种方法应该是最优的方式了。

### 巧用大括号控制变量作用域

在golang写的过程中，你一定有过为 := 和 = 烦恼的时刻。一个变量，到写的时候，我还要记得前面是否已经定义过了，如果没有定义过，使用 := ，如果已经定义过，使用 =。

当然很多时候可能你不会犯这种错误，变量命名的比较好的话，我们是很容易记得是否前面有定义过的。但是更多时候，对于err这种通用的变量名字，你可能就不一定记得了。

这个时候，巧妙使用大括号，就能很好避免这个问题。

我举一个我之前写一个命令行工具的例子，大家知道写命令行工具，对传递的参数的解析是需要有一些逻辑的，“如果参数中有某个字段，那么解析并存储到变量中，如果没有，记录error”，这里我就使用了大括号，将每个参数的解析和处理错误的逻辑都封装起来。

代码大致如下：

```go
var name string
var folder string
var mod string
...
{
   prompt := &survey.Input{
      Message: "请输入目录名称：",
   }
   err := survey.AskOne(prompt, &name)
   if err != nil {
      return err
   }

   ...
}
{
   prompt := &survey.Input{
      Message: "请输入模块名称(go.mod中的module, 默认为文件夹名称)：",
   }
   err := survey.AskOne(prompt, &mod)
   if err != nil {
      return err
   }
   ...
}
{
   // 获取hade的版本
   client := github.NewClient(nil)
   prompt := &survey.Input{
      Message: "请输入版本名称(参考 https://github.com/gohade/hade/releases，默认为最新版本)：",
   }
   err := survey.AskOne(prompt, &version)
   if err != nil {
      return err
   }
   ...
}
```

首先我将最终解析出来的最终变量在最开始做定义，然后使用三个大括号，分别将 name, mod, version 三个变量的解析逻辑封装在里面。而在每个大括号里面，err变量的作用域就完全局限在括号中了，每次都可以直接使用 := 来创建一个新的 err并处理它，不需要额外思考这个err 变量是否前面已经创建过了。

如果你自己观察，**大括号在代码语义上还有一个好处，就是归类和展示**。归类的意思是，这个大括号里面的变量和逻辑是一个完整的部分，他们内部创建的变量不会泄漏到外部。这个等于等于告诉后续的阅读者，你在阅读的时候，如果对这个逻辑不感兴趣，不阅读里面的内容，而如果你感兴趣的话，可以进入里面进行阅读。基本上所有IDE都支持对大括号封装的内容进行压缩，我使用Goland，压缩后，我的命令行的主体逻辑就更清晰了。

![image-20211220095540148](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/136188-20220329094419568-1743419297.png)

所以使用大括号，结合IDE，你的代码的可读性能得到很大的提升。

### 总结

文章中总结了四个golang中常用的写法

- 使用pkg/error而不是官方error库
- 在初始化slice的时候尽量补全cap
- 初始化一个类的时候，如果类的构造参数较多，尽量使用Option写法
- 巧用大括号控制变量作用域

# *Uber Go Style Guide*

## 01.介绍

这篇编程指南的初衷是更好的管理我们的代码，包括去编写什么样的代码，以及不要编写什么样的代码。我们希望通过这份编程指南，代码可以具有更好的维护性，同时能够让我们的开发同学更高效地编写 Go 语言代码。

这份编程指南最初由  Prashant Varanasi 和 Simon Newton 编写，旨在让其他同事快速地熟悉和编写 Go 程序。经过多年发展，现在的版本经过了多番修改和改进了。这是我们在 Uber 遵从的编程范式，但是很多都是可以通用的，如下是其他可以参考的链接：

- Effective Go
- The Go common mistakes guide

所有的提交代码都应该通过 `golint` 和 `go vet` 检测，建议在代码编辑器上面做如下设置：

- 保存的时候运行 `goimports`

- 使用 `golint` 和 `go vet` 去做错误检测（Go `vet` 命令在编写代码时非常有用。它可以帮助您检测应用程序中任何可疑、异常或无用的代码。该命令实际上由几个子分析器组成，甚至可以与您的自定义分析器一起工作。让我们首先回顾一下内置的分析器。）。

你可以通过下面链接发现更多的 Go 编辑器的插件: https://github.com/golang/go/wiki/IDEsAndTextEditorPlugins

## 02. 编程指南

#### 2.1 指向 Interface 的指针

在我们日常使用中，基本上**不会需要使用指向 interface 的指针**。当我们将 interface 作为值传递的时候，底层数据就是指针。Interface 包括两方面：

- 一个包含 type 信息的指针
- 一个指向数据的指针

如果你想要修改底层的数据，那么你只能使用 pointer。

#### 2.2 Receiver 和 Interface

使用值作为 receiver 的时候 method 可以通过指针调用，也可以通过值

```

type S struct {
  data string
}

func (s S) Read() string {
  return s.data
}

func (s *S) Write(str string) {
  s.data = str
}

sVals := map[int]S{1: {"A"}}

// You can only call Read using a value
sVals[1].Read()

// This will not compile:
//  sVals[1].Write("test")

sPtrs := map[int]*S{1: {"A"}}

// You can call both Read and Write using a pointer
sPtrs[1].Read()
sPtrs[1].Write("test")
```

相似的，pointer 也可以满足 interface 的要求，尽管 method 使用 value 作为 receiver。

```

type F interface {
  f()
}

type S1 struct{}

func (s S1) f() {}

type S2 struct{}

func (s *S2) f() {}

s1Val := S1{}
s1Ptr := &S1{}
s2Val := S2{}
s2Ptr := &S2{}

var i F
i = s1Val
i = s1Ptr
i = s2Ptr

// The following doesn't compile, since s2Val is a value, and there is no value receiver for f.
//   i = s2Val
```

Effective Go 关于如何使用指针和值也有一些不错的 practice： Pointers vs. Values.

#### 2.3 mutex 默认 0 值是合法的

`sync.Mutex` 和 `sync.RWMutex` 的 0 值也是合法的，所以我们基本不需要声明一个指针指向 mutex。

**Bad**

```
mu := new(sync.Mutex)
mu.Lock()
```

**Good**

```
var mu sync.Mutex
mu.Lock()
```

如果 struct 内部使用 mutex，在我们使用 struct 的指针类型时候，mutex 也可以是一个非指针类型的 field，或者直接嵌套在 struct 中。

Mutex 直接嵌套在 struct 中。

```
type smap struct {
  sync.Mutex

  data map[string]string
}

func newSMap() *smap {
  return &smap{
    data: make(map[string]string),
  }
}

func (m *smap) Get(k string) string {
  m.Lock()
  defer m.Unlock()

  return m.data[k]
}
```

将 Mutex 作为一个 struct 内部一个非指针类型 Field 使用。

```
type SMap struct {
  mu sync.Mutex

  data map[string]string
}

func NewSMap() *SMap {
  return &SMap{
    data: make(map[string]string),
  }
}

func (m *SMap) Get(k string) string {
  m.mu.Lock()
  defer m.mu.Unlock()

  return m.data[k]
}
```

#### 2.4 拷贝 Slice 和 Map

Slice 和 Map 都包含了对底层存储数据的指针，所以注意在修改 slice 或者 map 数据的场景下，是不是使用了引用。

##### slice 和 map 作为参数

当把 slice 和 map 作为参数的时候，如果我们对 slice 或者 map 的做了引用操作，那么修改会修改掉原始值。如果这种修改不是预期的，那么要先进行 copy。

**Bad**

```
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = trips
}

trips := ...
d1.SetTrips(trips)

// Did you mean to modify d1.trips?
trips[0] = ...
```

**Good**

```
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = make([]Trip, len(trips))
  copy(d.trips, trips)
}

trips := ...
d1.SetTrips(trips)

// We can now modify trips[0] without affecting d1.trips.
trips[0] = ...
```

##### slice 和 map 作为返回值

当我们的函数返回 slice 或者 map 的时候，也要注意是不是直接返回了内部数据的引用到外部。

**Bad**

```

type Stats struct {
  sync.Mutex

  counters map[string]int
}

// Snapshot returns the current stats.
func (s *Stats) Snapshot() map[string]int {
  s.Lock()
  defer s.Unlock()

  return s.counters
}

// snapshot is no longer protected by the lock!
snapshot := stats.Snapshot()
```

**Good**

```

type Stats struct {
  sync.Mutex

  counters map[string]int
}

func (s *Stats) Snapshot() map[string]int {
  s.Lock()
  defer s.Unlock()

  result := make(map[string]int, len(s.counters))
  for k, v := range s.counters {
    result[k] = v
  }
  return result
}

// Snapshot is now a copy.
snapshot := stats.Snapshot()

```

#### 2.5 使用 defer 做资源清理

建议使用 defer 去做资源清理工作，比如文件，锁等。

**Bad**

```
p.Lock()
if p.count < 10 {
  p.Unlock()
  return p.count
}

p.count++
newCount := p.count
p.Unlock()

return newCount

// easy to miss unlocks due to multiple returns
```

**Good**

```
p.Lock()
defer p.Unlock()

if p.count < 10 {
  return p.count
}

p.count++
return p.count

// more readable
```

**尽管使用 defer 会导致一定的性能开销，但是大部分情况下这个开销在你的整个链路上所占的比重往往是微乎其微**，除非说真的是有非常高的性能需求。另外使用 defer 带来的代码可读性的改进以及减少代码发生错误的概率都是值得的。

#### 2.6 channel 的 size 最好是 1 或者是 unbuffered

在使用 channel 的时候，最好将 size 设置为 1 或者使用 unbuffered channel。其他 size 的 channel 往往都会引入更多的复杂度，需要更多考虑上下游的设计。

**Bad**

```
// Ought to be enough for anybody!
c := make(chan int, 64)
```

**Good**

```

// Size of one
c := make(chan int, 1) // or
// Unbuffered channel, size of zero
c := make(chan int)
```

#### 2.7 枚举变量应该从 1 开始

在 Go 语言中枚举值的声明典型方式是通过 `const` 和 `iota` 来声明。**由于 0 是默认值，所以枚举值最好从一个非 0 值开始**，比如 1。

**Bad**

```

type Operation int

const (
  Add Operation = iota
  Subtract
  Multiply
)

// Add=0, Subtract=1, Multiply=2
```

**Good**

```
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

// Add=1, Subtract=2, Multiply=3
```

有一种例外情况：0 值是预期的默认行为的时候，枚举值可以从 0 开始。

```

type LogOutput int

const (
  LogToStdout LogOutput = iota
  LogToFile
  LogToRemote
)

// LogToStdout=0, LogToFile=1, LogToRemote=2
```

#### 2.8 Error 类型

在 Go 语言中声明 error 可以有多种方式：

- `errors.New` 声明包含简单静态字符串的 error
- `fmt.Errorf` 格式化 error string
- 其他自定义类型使用了 `Error()` 方法
- 使用 `"pkg/errors".Wrap`

当要把 error 作为返回值的时候，可以考虑如下的处理方式

- **是不是不需要额外信息**，如果是，`errors.New` 就足够了。
- **client 需要检测和处理返回的 error 吗**？如果是，最好使用实现了 `Error()` 方法的自定义类型，这样可以包含更多的信息。
- **error 是不是从下游函数传递过来的**？如果是，考虑一下 error wrap，参考：section on error wrapping.
- 其他情况，`fmt.Errorf` 一般足够了。

**对于 client 需要检测和处理 error 的情况**，这里详细说一下。如果你要通过 `errors.New` 声明一个简单的 error，那么可以使用一个变量声明：`var ErrCouldNotOpen = errors.New("Could not open")`

**Bad**

```
// package foo

func Open() error {
  return errors.New("could not open")
}

// package bar

func use() {
  if err := foo.Open(); err != nil {
    if err.Error() == "could not open" {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

**Good**

```
// package foo

var ErrCouldNotOpen = errors.New("could not open")

func Open() error {
  return ErrCouldNotOpen
}

// package bar

if err := foo.Open(); err != nil {
  if err == foo.ErrCouldNotOpen {
    // handle
  } else {
    panic("unknown error")
  }
}
```

如果需要 error 中包含更多的信息，而不仅仅类型原生 error 的这种简单字符串，那么最好使用一个自定义类型。

**Bad**

```
func open(file string) error {
  return fmt.Errorf("file %q not found", file)
}

func use() {
  if err := open(); err != nil {
    if strings.Contains(err.Error(), "not found") {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

**Good**

```
type errNotFound struct {
  file string
}

func (e errNotFound) Error() string {
  return fmt.Sprintf("file %q not found", e.file)
}

func open(file string) error {
  return errNotFound{file: file}
}

func use() {
  if err := open(); err != nil {
    if _, ok := err.(errNotFound); ok {
      // handle
    } else {
      panic("unknown error")
    }
  }
}
```

在直接暴露自定义的 error 类型的时候，最好 export 配套的检测自定义 error 类型的函数。

```
// package foo

type errNotFound struct {
  file string
}

func (e errNotFound) Error() string {
  return fmt.Sprintf("file %q not found", e.file)
}

func IsNotFoundError(err error) bool {
  _, ok := err.(errNotFound)
  return ok
}

func Open(file string) error {
  return errNotFound{file: file}
}

// package bar

if err := foo.Open("foo"); err != nil {
  if foo.IsNotFoundError(err) {
    // handle
  } else {
    panic("unknown error")
  }
}
```

#### 2.9 Error Wrapping

**在函数调用失败的时候，有三种方式可以将下游的 error 传递出去**：

- 直接返回失败函数返回的 error。
- 使用 `"pkg/errors".Wrap` 增加更多的上下文信息，这种情况下可以使用 `"pkg/errors".Cause` 去提取原始的 error 信息。
- 如果调用者不需要检测和处理返回的 error 信息的话，可以直接使用 `fmt.Errorf` 将需要附加的信息进行格式化添加进去。

如果条件允许，最好增加上下文信息。比如  "connection refused" 和 "call service foo: connection refused" 这两种错误信息在可读性上比较也是高下立判。当增加上下文信息的时候，尽量保持简洁。比如像 "failed to" 这种极其明显的信息就没有必要写上去了。

**Bad**

```
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "failed to create new store: %s", err)
}
```

**Good**

```
s, err := store.New()
if err != nil {
    return fmt.Errorf(
        "new store: %s", err)
}
```

另外对于需要传播到其他系统的 error，也要有明显的标识信息，比如在 log 的最前面增加 `err`等字样。

更多参考：Don't just check errors, handle them gracefully.

#### 2.10 类型转换失败处理

类型转换失败会导致进程 panic，所以对于类型转换，一定要使用 "comma ok" 的范式来处理。

**Bad**

```
t := i.(string)
```

**Good**

```
t, ok := i.(string)
if !ok {
  // handle the error gracefully
}
```

#### 2.11 不要 panic

**对于线上环境要尽量避免 panic**。在很多情况下，panic 都是引起雪崩效应的罪魁祸首。一旦 error 发生，我们应该向上游调用者返回 error，并且容许调用者对 error 进行检测和处理。

**Bad**

```

func foo(bar string) {
  if len(bar) == 0 {
    panic("bar must not be empty")
  }
  // ...
}

func main() {
  if len(os.Args) != 2 {
    fmt.Println("USAGE: foo <bar>")
    os.Exit(1)
  }
  foo(os.Args[1])
}
```

**Good**

```
func foo(bar string) error {
  if len(bar) == 0
    return errors.New("bar must not be empty")
  }
  // ...
  return nil
}

func main() {
  if len(os.Args) != 2 {
    fmt.Println("USAGE: foo <bar>")
    os.Exit(1)
  }
  if err := foo(os.Args[1]); err != nil {
    panic(err)
  }
}
```

Panic/Recover 并不是一种 error 处理策略。进程只有在某些不可恢复的错误发生的时候才需要 panic。

在跑 test case 的时候，使用 `t.Fatal` 或者 `t.FailNow` ，而不是 panic 来保证这个 test case 会被标记为失败的。

**Bad**

```
// func TestFoo(t *testing.T)

f, err := ioutil.TempFile("", "test")
if err != nil {
  panic("failed to set up test")
}
```

**Good**

```
// func TestFoo(t *testing.T)

f, err := ioutil.TempFile("", "test")
if err != nil {
  t.Fatal("failed to set up test")
}
```

#### 2.12 使用 go.uber.org/atomic

这个是 Uber 内部对原生包 `sync/atomic` 的一种封装，隐藏了底层数据类型。

**Bad**

```
type foo struct {
  running int32  // atomic
}

func (f* foo) start() {
  if atomic.SwapInt32(&f.running, 1) == 1 {
     // already running…
     return
  }
  // start the Foo
}

func (f *foo) isRunning() bool {
  return f.running == 1  // race!
}
```

**Good**

```
type foo struct {
  running atomic.Bool
}

func (f *foo) start() {
  if f.running.Swap(true) {
     // already running…
     return
  }
  // start the Foo
}

func (f *foo) isRunning() bool {
  return f.running.Load()
}
```

## 03.性能相关

#### 3.1 类型转换时，使用 strconv 替换 fmt

当基本类型和 string 互转的时候，`strconv` 要比 `fmt` 快。

**Bad**

```
for i := 0; i < b.N; i++ {
  s := fmt.Sprint(rand.Int())
}

BenchmarkFmtSprint-4    143 ns/op    2 allocs/op
```

**Good**

```
for i := 0; i < b.N; i++ {
  s := strconv.Itoa(rand.Int())
}

BenchmarkStrconv-4    64.2 ns/op    1 allocs/op
```

#### 3.2 避免 string to byte 的不必要频繁转换

**在通过 string 创建 byte slice 的时候，不要在循环语句中重复的转换，而是要将重复的转换逻辑提到循环外面，做一次即可**。(看上去很 general 的建议)

**Bad**

```
for i := 0; i < b.N; i++ {
  w.Write([]byte("Hello world"))
}

BenchmarkBad-4   50000000   22.2 ns/op
```

**Good**

```
data := []byte("Hello world")
for i := 0; i < b.N; i++ {
  w.Write(data)
}

BenchmarkGood-4  500000000   3.25 ns/op
```



## 0.4编程风格

#### 4.1 声明语句分组

import 语句分组

**Bad**

```
import "a"
import "b"
```

**Good**

```
import (
  "a"
  "b"
)
```

常量、变量以及 type 声明

**Bad**

```
const a = 1
const b = 2

var a = 1
var b = 2

type Area float64
type Volume float64
```

**Good**

```
const (
  a = 1
  b = 2
)

var (
  a = 1
  b = 2
)

type (
  Area float64
  Volume float64
)
```

import 根据导入的包进行顺序分组。（其他库我们其实可以再细分 private 库和 public 库）

- **标准库**
- **其他库**

**Bad**

```
import (
  "fmt"
  "os"
  "go.uber.org/atomic"
  "golang.org/x/sync/errgroup"
)
```

**Good**

```
import (
  "fmt"
  "os"
  
  "go.uber.org/atomic"
  "golang.org/x/sync/errgroup"
)
```

#### 4.2 package 命名

package 命名的几条规则：

- **全小写**。不包含大写字母或者下划线。
- 简洁。
- **不要使用复数**。比如，使用 `net/url`，而不是 `net/urls`。
- 避免："common", "util", "shared", "lib"，不解释。

更多参考：

- Package Names  
- Style guideline for Go packages.

#### 4.3 函数命名

函数命名遵从社区规范：MixedCaps for function names 。有一种特例是 TestCase 中为了方便测试做的函数命名，比如：`TestMyFunction_WhatIsBeingTested`。

#### 4.4 import 别名

当 package 的名字和 import 的 path 的最后一个元素不同的时候，必须要起别名。

```
import (
  "net/http"

  client "example.com/client-go"
  trace "example.com/trace/v2"
)
```

另外，**import 别名要尽量避免**，只要在不得不起别名的时候再这么做，比如避免冲突。

**Bad**

```
import (
  "fmt"
  "os"

  nettrace "golang.net/x/trace"
)
```

**Good**

```
import (
  "fmt"
  "os"
  "runtime/trace"

  nettrace "golang.net/x/trace"
)
```

#### 4.5 函数分组和排序

- **函数应该按调用顺序排序**
- **一个文件中的函数应该按 receiver 排序**

`newXYZ/NewXYZ` 最好紧接着类型声明后面，并在其他的 receiver 函数前面。

**Bad**

```
func (s *something) Cost() {
  return calcCost(s.weights)
}

type something struct{ ... }

func calcCost(n int[]) int {...}

func (s *something) Stop() {...}

func newSomething() *something {
    return &something{}
}
```

**Good**

```
type something struct{ ... }

func newSomething() *something {
    return &something{}
}

func (s *something) Cost() {
  return calcCost(s.weights)
}

func (s *something) Stop() {...}

func calcCost(n int[]) int {...}
```

#### 4.6 避免代码块嵌套

**优先处理异常情况，快速返回，避免代码块过多嵌套**。看下面代码会比较直观。

**Bad**

```

for _, v := range data {
  if v.F1 == 1 {
    v = process(v)
    if err := v.Call(); err == nil {
      v.Send()
    } else {
      return err
    }
  } else {
    log.Printf("Invalid v: %v", v)
  }
}
```

**Good**

```
for _, v := range data {
  if v.F1 != 1 {
    log.Printf("Invalid v: %v", v)
    continue
  }

  v = process(v)
  if err := v.Call(); err != nil {
    return err
  }
  v.Send()
}
```

#### 4.7 避免不必要的 else 语句

很多情况下，if - else 语句都能通过一个 if 语句表达，比如如下代码。

**Bad**

```
var a int
if b {
  a = 100
} else {
  a = 10
}
```

**Good**

```
a := 10
if b {
  a = 100
}
```



#### 4.8 两级 (two-level) 变量声明

所有两级变量声明就是一个声明的右值来自另一个表达式，这个时候第一级变量声明就不需要指明类型，除非这两个地方的数据类型不同。看代码会更直观一点。

**Bad**

```
var _s string = F()

func F() string { return "A" }
```

**Good**

```
var _s = F()
// Since F already states that it returns a string, we don't need to specify
// the type again.

func F() string { return "A" }
```

上面说的第二种两边数据类型不同的情况。

```
type myError struct{}
func (myError) Error() string { return "error" }
func F() myError { return myError{} }
var _e error = F()// F returns an object of type myError but we want error.
```

#### 4.9 对于不做 export 的全局变量使用前缀 _

对于同一个 package 下面的多个文件，一个文件中的全局变量可能会被其他文件误用，所以建议使用 _ 来做前缀。（其实这条规则有待商榷）

**Bad**

```
// foo.go

const (
  defaultPort = 8080
  defaultUser = "user"
)

// bar.go

func Bar() {
  defaultPort := 9090
  ...
  fmt.Println("Default port", defaultPort)

  // We will not see a compile error if the first line of
  // Bar() is deleted.
}
```

**Good**

```
// foo.go

const (
  _defaultPort = 8080
  _defaultUser = "user"
)
```

#### 4.10 struct 嵌套

struct 中的嵌套类型在 field 列表排在最前面，并且用空行分隔开。

**Bad**

```
type Client struct {
  version int
  http.Client
}
```

**Good**

```
type Client struct {
  http.Client

  version int
}
```

#### 4.11 struct 初始化的时候带上 Field

这样会更清晰，也是 go vet 鼓励的方式

**Bad**

```
k := User{"John", "Doe", true}
```

**Good**

```
k := User{
    FirstName: "John",
    LastName: "Doe",
    Admin: true,
}
```

#### 4.12 局部变量声明

变量声明的时候可以使用 `:=` 以表示这个变量被显示的设置为某个值。

**Bad**

```
var s = "foo"
```

**Good**

```
s := "foo"
```

但是对于某些情况使用 var 反而表示的更清晰，比如声明一个空的 slice: Declaring Empty Slices

**Bad**

```
func f(list []int) {
  filtered := []int{}
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

**Good**

```

func f(list []int) {
  var filtered []int
  for _, v := range list {
    if v > 10 {
      filtered = append(filtered, v)
    }
  }
}
```

#### 4.13 nil 是合法的 slice

在返回值是 slice 类型的时候，直接返回 nil 即可，不需要显式地返回长度为 0 的 slice。

**Bad**

```
if x == "" {
  return []int{}
}
```

**Good**

```
if x == "" {
  return nil
}
```

判断 slice 是不是空的时候，使用 `len(s) == 0`。

**Bad**

```
func isEmpty(s []string) bool {
  return s == nil
}
```

**Good**

```
func isEmpty(s []string) bool {
  return len(s) == 0
}
```

使用 var 声明的 slice 空值可以直接使用，不需要 `make()`。

**Bad**

```
nums := []int{}
// or, nums := make([]int)

if add1 {
  nums = append(nums, 1)
}

if add2 {
  nums = append(nums, 2)
}
```

**Good**

```
var nums []int

if add1 {
  nums = append(nums, 1)
}

if add2 {
  nums = append(nums, 2)
}
```

#### 4.14 避免不必要的 scope（范围）

**Bad**

```
err := ioutil.WriteFile(name, data, 0644)
if err != nil {
 return err
}
```

**Good**

```
if err := ioutil.WriteFile(name, data, 0644); err != nil {
 return err
}
```

当然某些情况下，scope 是不可避免的，比如

**Bad**

```
if data, err := ioutil.ReadFile(name); err == nil {
  err = cfg.Decode(data)
  if err != nil {
    return err
  }

  fmt.Println(cfg)
  return nil
} else {
  return err
}
```

**Good**

```
data, err := ioutil.ReadFile(name)
if err != nil {
   return err
}

if err := cfg.Decode(data); err != nil {
  return err
}

fmt.Println(cfg)
return nil
```

#### 4.15 避免参数语义不明确（Avoid Naked Parameters）

Naked Parameter 指的应该是意义不明确的参数，这种情况会破坏代码的可读性，可以使用 C 分格的注释（`/*...*/`）进行注释。

**Bad**

```
// func printInfo(name string, isLocal, done bool)
printInfo("foo", true, true)
```

**Good**

```
// func printInfo(name string, isLocal, done bool)
printInfo("foo", true /* isLocal */, true /* done */)
```

对于上面的示例代码，还有一种更好的处理方式是将上面的 bool 类型换成自定义类型。

```
type Region int

const (
  UnknownRegion Region = iota
  Local
)

type Status int

const (
  StatusReady = iota + 1
  StatusDone
  // Maybe we will have a StatusInProgress in the future.
)

func printInfo(name string, region Region, status Status)
```

**4.16 使用原生字符串，避免转义**

Go 支持使用反引号，也就是 "`" 来表示原生字符串，在需要转义的场景下，我们应该尽量使用这种方案来替换。

**Bad**

```
wantError := "unknown name:\"test\""
```

**Good**

```
wantError := `unknown error:"test"`
```

#### 4.17 Struct 引用初始化

使用 `&T{}` 而不是 `new(T)` 来声明对 T 类型的引用，使用 `&T{}` 的方式我们可以和 struct 声明方式 `T{}` 保持统一。

**Bad**

```
sval := T{Name: "foo"}

// inconsistent
sptr := new(T)
sptr.Name = "bar"
```

**Good**

```
sval := T{Name: "foo"}

sptr := &T{Name: "bar"}
```

#### 4.18 字符串 string format 

如果我们要在 Printf 外面声明 format 字符串的话，使用 const，而不是变量，这样 go vet 可以对 format 字符串做静态分析。

**Bad**

```
msg := "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

**Good**

```
const msg = "unexpected values %v, %v\n"
fmt.Printf(msg, 1, 2)
```

#### 4.19 Printf 风格函数命名

当声明 `Printf` 风格的函数时，确保 `go vet` 可以对其进行检测。可以参考：Printf family 。

另外也可以在函数名字的结尾使用 f 结尾，比如: `WrapF`，而不是 `Wrap`。然后使用 `go vet`

```
$ go vet -printfuncs=wrapf,statusf
```

更多参考: go vet: Printf family check.

## 05. 编程模式

#### 5.1 Test Tables

**当测试逻辑是重复的时候**，通过  subtests 使用 table 驱动的方式编写 case 代码看上去会更简洁。

**Bad**

```
// func TestSplitHostPort(t *testing.T)

host, port, err := net.SplitHostPort("192.0.2.0:8000")
require.NoError(t, err)
assert.Equal(t, "192.0.2.0", host)
assert.Equal(t, "8000", port)

host, port, err = net.SplitHostPort("192.0.2.0:http")
require.NoError(t, err)
assert.Equal(t, "192.0.2.0", host)
assert.Equal(t, "http", port)

host, port, err = net.SplitHostPort(":8000")
require.NoError(t, err)
assert.Equal(t, "", host)
assert.Equal(t, "8000", port)

host, port, err = net.SplitHostPort("1:8")
require.NoError(t, err)
assert.Equal(t, "1", host)
assert.Equal(t, "8", port)
```

**Good**

```
// func TestSplitHostPort(t *testing.T)

tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  {
    give:     "192.0.2.0:8000",
    wantHost: "192.0.2.0",
    wantPort: "8000",
  },
  {
    give:     "192.0.2.0:http",
    wantHost: "192.0.2.0",
    wantPort: "http",
  },
  {
    give:     ":8000",
    wantHost: "",
    wantPort: "8000",
  },
  {
    give:     "1:8",
    wantHost: "1",
    wantPort: "8",
  },
}

for _, tt := range tests {
  t.Run(tt.give, func(t *testing.T) {
    host, port, err := net.SplitHostPort(tt.give)
    require.NoError(t, err)
    assert.Equal(t, tt.wantHost, host)
    assert.Equal(t, tt.wantPort, port)
  })
}
```

很明显，使用 test table 的方式在代码逻辑扩展的时候，比如新增 test case，都会显得更加的清晰。

在命名方面，我们将 struct 的 slice 命名为 `tests`，同时每一个 test case 命名为 `tt`。而且，我们强烈建议通过 `give` 和 `want` 前缀来表示 test case 的 input 和 output 的值。

```
tests := []struct{
  give     string
  wantHost string
  wantPort string
}{
  // ...
}

for _, tt := range tests {
  // ...
}
```

#### 5.2 Functional Options

关于 functional options 简单来说就是通过类似闭包的方式来进行函数传参。

**Bad**

```
// package db

func Connect(
  addr string,
  timeout time.Duration,
  caching bool,
) (*Connection, error) {
  // ...
}

// Timeout and caching must always be provided,
// even if the user wants to use the default.

db.Connect(addr, db.DefaultTimeout, db.DefaultCaching)
db.Connect(addr, newTimeout, db.DefaultCaching)
db.Connect(addr, db.DefaultTimeout, false /* caching */)
db.Connect(addr, newTimeout, false /* caching */)
```

**Good**

```
// package db

func Connect(
  addr string,
  timeout time.Duration,
  caching bool,
) (*Connection, error) {
  // ...
}

// Timeout and caching must always be provided,
// even if the user wants to use the default.

db.Connect(addr, db.DefaultTimeout, db.DefaultCaching)
db.Connect(addr, newTimeout, db.DefaultCaching)
db.Connect(addr, db.DefaultTimeout, false /* caching */)
db.Connect(addr, newTimeout, false /* caching */)
```

更多参考：

- Self-referential functions and the design of options
- Functional options for friendly APIs

注：关于 functional option 这种方式我本人也强烈推荐，我很久以前也写过一篇类似的文章，感兴趣的可以移步： http://legendtkl.com/2016/11/05/code-scalability/



## 06. 总结

Uber 开源的这个文档，虽然说有些内容值得商榷，但是瑕不掩瑜。通篇读下来给我印象最深的就是：保持代码简洁，并具有良好可读性。不得不说，相比于国内很多 “代码能跑就完事了” 这种写代码的态度，这篇文章或许可以给我们更多的启示和参考。
