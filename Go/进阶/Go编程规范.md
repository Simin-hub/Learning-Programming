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
