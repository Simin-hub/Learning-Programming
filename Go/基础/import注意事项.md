# 导入包必学

[参考地址](https://juejin.cn/post/6844904167073382408)

## 1. 单行导入与多行导入

在 Go 语言中，一个包可包含多个 `.go` 文件（这些文件必须得在同一级文件夹中），只要这些 `.go` 文件的头部都使用 `package` 关键字声明了同一个包。

导入包主要可分为两种方式：

- 单行导入

```
import "fmt"
import "sync" 
```

- 多行导入

```
import(
    "fmt"
    "sync"
)
```

> 如你所见，Go 语言中 导入的包，必须得用双引号包含

## 2. 使用别名

在一些场景下，我们可能需要对导入的包进行重新命名，比如

- 我们导入了**两个具有同一包名的包时产生冲突**，此时这里为其中一个包定义别名

```
import (
    "crypto/rand"
    mrand "math/rand" // 将名称替换为mrand避免冲突
)
```

- 我们导入了一个**名字很长的包**，为了避免后面都写这么长串的包名，可以这样定义别名

```
import hw "helloworldtestmodule"
```

- **防止导入的包名和本地的变量发生冲突**，比如 path 这个很常用的变量名和导入的标准包冲突。

```
import pathpkg "path"
```

## 3. 使用点操作

如里在我们程序内部里频繁使用了一个工具包，比如 fmt，那每次使用它的打印函数打印时，都要 包名+方法名。

对于这种使用高频的包，可以在导入的时，就把它定义会 "`自己人`"（方法是使用一个 `.` ），自己人的话，不分彼此，它的方法，就是我们的方法。

从此，我们打印再也不用加 `fmt `了。

```
import . "fmt"

func main() {
    Println("hello, world")
}
```

但这种用法，会有一定的隐患，就是导入的包里**可能有函数，会和我们自己的函数发生冲突**。

## 4. 包的初始化

每个包都允许有一个 `init` 函数，当这个包被导入时，会执行该包的这个 `init` 函数，做一些初始化任务。

对于 `init` 函数的执行有两点需要注意

1. **`init` 函数优先于 `main` 函数执行**
2. 在一个包引用链中，包的初始化是**深度优先**的。比如，有这样一个包引用关系：main→A→B→C，那么初始化顺序为

```
C.init→B.init→A.init→main
```

## 5. 包的匿名导入

当我们导入一个包时，如果这个包没有被使用到，在编译时，是会报错的。

但是有些情况下，我们导入一个包，**只想执行包里的 `init` 函数**，来运行一些初始化任务，此时怎么办呢？

可以使用匿名导入，用法如下，其中下划线为空白标识符，并不能被访问

```
// 注册一个PNG decoder
import _ "image/png"
```

由于导入时，会执行` init` 函数，所以编译时，仍然会将这个包编译到可执行文件中。

## 6. 导入的是路径还是包？

当我们使用 import 导入 `testmodule/foo` 时，初学者，经常会问，这个 `foo` 到底是一个包呢，还是只是包所在目录名？

```
import "testmodule/foo"
```

最后得出的结论是：

- **导入时，是按照目导入**。导入目录后，可以使用这个目录下的**所有包**。
- 出于习惯，包名和目录名通常会设置成一样，所以会让你有一种你导入的是包的错觉。

## 7. 相对导入和绝对导入

**绝对导入**：从 `$GOPATH/src` 或 `$GOROOT` 或者 `$GOPATH/pkg/mod` 目录下搜索包并导入

**相对导入**：从当前目录中搜索包并开始导入。就像下面这样

```
import (
    "./module1"
    "../module2"
    "../../module3"
    "../module4/module5"
)复制代码
```

分别举个例子吧

**一、使用绝对导入**

有如下这样的目录结构（注意确保当前目录在 GOPATH 下）

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/5/24/172455407ffbf2f3~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

其中 main.go 是这样的

```
package main

import (
    "app/utilset"   // 这种使用的就是绝对路径导入
)

func main() {
    utils.PrintHello()
}复制代码
```

而在 main.go 的同级目录下，还有另外一个文件夹 `utilset` ，为了让你理解 「**第六点：import 导入的是路径而不是包**」，我在 utilset 目录下定义了一个 `hello.go` 文件，这个go文件定义所属包为 `utils`。

```
package utils

import "fmt"

func PrintHello(){
    fmt.Println("Hello, 我在 utilset 目录下的 utils 包里")
}
```

运行结果如下

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/5/24/17245540adb8ad89~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

**二、使用相对导入**

还是上面的代码，将绝对导入改为相对导入后

将 GOPATH 路径设置回去（请对比上面使用绝对路径的 GOPATH）

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/5/24/17245540da7e7f95~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

然后再次运行

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/5/24/172455410cdb4b41~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

总结一下，**使用相对导入**，有两点需要注意

- **项目不要放在 `$GOPATH/src`** 下，否则会报错（比如我修改当前项目目录为GOPATH后，运行就会报错）

  ![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/5/24/172455413ca9379a~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

- **Go Modules 不支持相对导入**，在你开启 GO111MODULE 后，无法使用相对导入。

最后，不得不说的是：使用相对导入的方式，项目可读性会大打折扣，不利用开发者理清整个引用关系。

所以一般更推荐使用绝对引用的方式。使用绝对引用的话，又要谈及优先级了

## 8. 包导入路径优先级

前面一节，介绍了三种不同的包依赖管理方案，不同的管理模式，存放包的路径可能都不一样，有的可以将包放在 GOPATH 下，有的可以将包放在 vendor 下，还有些包是内置包放在 GOROOT 下。

那么问题就来了，如果在这三个不同的路径下，有一个相同包名但是版本不同的包，我们导入的时候，是选择哪个进行导入呢？

这就需要我们搞懂，在 Golang 中包搜索路径优先级是怎样的？

这时候就需要区分，是使用哪种模式进行包的管理的。

**如果使用 govendor**

当我们导入一个包时，它会：

1. 先从项目根目录的 `vendor` 目录中查找
2. 最后从 `$GOROOT/src` 目录下查找
3. 然后从 `$GOPATH/src` 目录下查找
4. 都找不到的话，就报错。

为了验证这个过程，我在创建中创建一个 vendor 目录后，就开启了 vendor 模式了，我在 main.go 中随便导入一个包 pkg，由于这个包是我随便指定的，当然会找不到，找不到就会报错， Golang 会在报错信息中打印中搜索的过程，从这个信息中，就可以看到 Golang 的包查找优先级了。

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/5/24/172455416b747bd4~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

**如果使用 go modules**

你导入的包如果有域名，都会先在 `$GOPATH/pkg/mod` 下查找，找不到就连网去该网站上寻找，找不到或者找到的不是一个包，则报错。

而如果你导入的包没有域名（比如 "fmt"这种），就只会到 `$GOROOT` 里查找。

还有一点很重要，当你的项目下有 vendor 目录时，不管你的包有没有域名，都只会在 vendor 目录中想找。

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/5/24/17245541a5ef5ec2~tplv-t2oaga2asx-zoom-in-crop-mark:1304:0:0:0.awebp)

通常`vendor` 目录是通过 `go mod vendor` 命令生成的，这个命令会将项目依赖全部打包到你的项目目录下的 verdor 文件夹中。

# go module导入本地包

## 前提

假设我们现在有`moduledemo`和`mypackage`两个包，其中`moduledemo`包中会导入`mypackage`包并使用它的`New`方法。

`mypackage/mypackage.go`内容如下：

```
package mypackage
import "fmt"
func New(){    
	fmt.Println("mypackage.New")
}
```

我们现在分两种情况讨论：

## 在同一个项目下

**注意**：在一个项目（project）下我们是可以定义多个包（package）的。

### 目录结构

现在的情况是，我们在`moduledemo/main.go`中调用了`mypackage`这个包。

```
moduledemo
├── go.mod
├── main.go
└── mypackage
    └── mypackage.go
```

### 导入包

这个时候，我们需要在`moduledemo/go.mod`中按如下定义：

```
module moduledemo

go 1.14
```

然后在`moduledemo/main.go`中按如下方式导入`mypackage`

```
package main

import (
    "fmt"
    "moduledemo/mypackage"  // 导入同一项目下的mypackage包
)
func main() {
    mypackage.New()
    fmt.Println("main")
}
```

### 举个例子

举一反三，假设我们现在有文件目录结构如下：

```
└── bubble
    ├── dao
    │   └── mysql.go
    ├── go.mod
    └── main.go
```

其中`bubble/go.mod`内容如下：

```
module github.com/q1mi/bubble

go 1.14
```

`bubble/dao/mysql.go`内容如下：

```
package dao
import "fmt"
func New(){    
	fmt.Println("mypackage.New")
}
```

`bubble/main.go`内容如下：

```
package main
import (    
    "fmt"    
    "github.com/q1mi/bubble/dao"
)
func main() {    
    dao.New()    
    fmt.Println("main")
}
```

## 不在同一个项目下

### 目录结构

```
├── moduledemo
│   ├── go.mod
│   └── main.go
└── mypackage
    ├── go.mod
    └── mypackage.go
```

### 导入包

这个时候，`mypackage`也需要进行module初始化，即拥有一个属于自己的`go.mod`文件，内容如下：

```
module mypackage
go 1.14
```

然后我们在`moduledemo/main.go`中按如下方式导入：

```
import (    
    "fmt"    
    "mypackage"
)
func main() {    
    mypackage.New()    
    fmt.Println("main")
}
```

因为这两个包不在同一个项目路径下，你想要导入本地包，并且这些包也没有发布到远程的github或其他代码仓库地址。这个时候我们就需要在`go.mod`文件中使用`replace`指令。

在调用方也就是`packagedemo/go.mod`中按如下方式指定使用相对路径来寻找`mypackage`这个包。

```
module moduledemo
go 1.14

require "mypackage" v0.0.0
replace "mypackage" => "../mypackage"
```

### 举个例子

最后我们再举个例子巩固下上面的内容。

我们现在有文件目录结构如下：

```
├── p1
│   ├── go.mod
│   └── main.go
└── p2
    ├── go.mod
    └── p2.go
```

`p1/main.go`中想要导入`p2.go`中定义的函数。

`p2/go.mod`内容如下：

```
module liwenzhou.com/q1mi/p2
go 1.14
```

`p1/main.go`中按如下方式导入

```
import (    
    "fmt"    
    "liwenzhou.com/q1mi/p2"
)
func main() {    
    p2.New()    
    fmt.Println("main")
}
```

因为我并没有把`liwenzhou.com/q1mi/p2`这个包上传到`liwenzhou.com`这个网站，我们只是想导入本地的包，这个时候就需要用到`replace`这个指令了。

`p1/go.mod`内容如下：

```
module github.com/q1mi/p1
go 1.14

require "liwenzhou.com/q1mi/p2" v0.0.0replace "liwenzhou.com/q1mi/p2" => "../p2"
```

此时，我们就可以正常编译`p1`这个项目了。