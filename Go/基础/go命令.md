# Go命令详解

[官网地址](https://golang.google.cn/cmd/go/)

[地址](https://zhuanlan.zhihu.com/p/161494871)

## 概述

Go语言有一套完整的操作命令，非常实用，掌握它可以事半功倍。

将详细讲解以下命令（便于快速查找）：

- **go bug**
- **go build**
- **go install**
- **go get**
- **go mod**
- **go run**
- **go clean**
- **go fmt**
- **go list**
- **go version**
- **go env**



直接输入go，给出了全部命令，如下所示：

```text
go
```

![img](https://pic2.zhimg.com/v2-819aff3c843341fc757a863aea9554c1_r.jpg)



若想更加详细了解某个命令的具体用法，只需输入"go help 命令"，例如：

```text
go help build
```

![img](https://pic4.zhimg.com/v2-18e197119d9927995ada14fb8f85f5fb_r.jpg)

从上面可看到描述非常详细。同时官方也给出了非常详细的[文档说明](https://golang.google.cn/cmd/go/)。

下面将详细介绍一些命令。



## go bug

输入此命名后会直接打开默认浏览器，显示go的github页面进行bug报告，并会自动添加系统的信息。在GitHub上提交go的bug

```text
go bug
```

![img](https://pic1.zhimg.com/v2-583ffc78bec2c507f37e3638e20fe7b4_r.jpg)



## go build

此命令用于编译指定的源码文件或代码包及依赖包。

目录如下所示：

![img](https://pic1.zhimg.com/v2-13fe01a7761cfea5aad67d2f089986e4_r.jpg)

若只有一个main函数文件，进入main函数所在目录，直接输入"go build"会自动编译main函数所在文件，并生成一个可执行文件。

```text
go build
```

![img](https://pic2.zhimg.com/v2-9c7549201b6a944f8fad39b0da498afd_r.jpg)

若要指定文件名，可使用参数 -o，也可指定文件输出的目录。

```text
go build -o main.exe
```

![img](https://pic3.zhimg.com/v2-d902059239072e97d28c2546a487613e_r.jpg)



若有多个main函数文件，若只想编译其中某个文件，可在go build 后加入文件名即可，如："go build 文件名"

main2.go中也有一个main函数，若直接输入"go build"会编译出错：

```text
go build
```

![img](https://pic2.zhimg.com/v2-d0e97f2a9c5fc5fae9d785ef33023961_r.jpg)



此时需要"go build 文件名"指定一个文件名编译：

```text
go build main2.go
```

![img](https://pic1.zhimg.com/v2-907b0d28e28f5f5a2625a176a2fcc234_r.jpg)



若是在普通包下，输入"go build"后不会生成任何文件，只会检查错误。如下zarten为一个普通包，在此包下执行"go build"

```text
go build
```

![img](https://pic4.zhimg.com/v2-50144ec1ba573976c01846ae9ba8d4bf_r.jpg)



除了"-o"参数，还有一些其他参数：

- `-o` 指定输出的文件名，可以带上路径，例如 `go build -o a/b/c`
- `-i` 安装相应的包，编译+`go install`
- `-a` 更新全部已经是最新的包的，但是对标准包不适用
- `-n` 把需要执行的编译命令打印出来，但是不执行，这样就可以很容易的知道底层是如何运行的
- `-p n` 指定可以并行可运行的编译数目，默认是CPU数目
- `-race` 开启编译的时候自动检测数据竞争的情况，目前只支持64位的机器
- `-v` 打印出来我们正在编译的包名
- `-work` 打印出来编译时候的临时文件夹名称，并且如果已经存在的话就不要删除
- `-x` 打印出来执行的命令，其实就是和`-n`的结果类似，只是这个会执行
- `-ccflags 'arg list'` 传递参数给5c, 6c, 8c 调用
- `-compiler name` 指定相应的编译器，gccgo还是gc
- `-gccgoflags 'arg list'` 传递参数给gccgo编译连接调用
- `-gcflags 'arg list'` 传递参数给5g, 6g, 8g 调用
- `-installsuffix suffix` 为了和默认的安装包区别开来，采用这个前缀来重新安装那些依赖的包，`-race`的时候默认已经是`-installsuffix race`,大家可以通过`-n`命令来验证
- `-ldflags 'flag list'` 传递参数给5l, 6l, 8l 调用
- `-tags 'tag list'` 设置在编译的时候可以适配的那些tag，详细的tag限制参考里面的 [Build Constraints](http://golang.org/pkg/go/build/)

例如"-n"参数：

```text
go build -n main.go
```

![img](https://pic4.zhimg.com/v2-ed362a19a2e628977090cf05d62213bb_r.jpg)



## go install

1. compile
2. install

go install 跟 go build 类似，只是多做了一件事就是**安装编译后的文件到指定目录**。

```text
go install main.go
```

![img](https://pic1.zhimg.com/v2-cdbe942480be7ad1565e399d935be4a4_r.jpg)

出现错误：“go install: no install location for .go files listed on command line (GOBIN not set)”

这是因为没有设置环境变量GOBIN。

Go开发中一般要设置环境变量GOROOT和GOPATH。按照Go开发规范，GOPATH目录下一般分为三个子目录：src、pkg、bin

src：源代码文件

pkg：编译后的库静态文件

bin：源代码编译后的可执行文件

因此环境变量GOBIN设置的就是上面的bin目录，"go install"命令就将可执行文件自动安装到bin目录下。

**设置环境变量GOBIN**

```text
export GOBIN=$GOPATH/bin
```

![img](https://pic4.zhimg.com/v2-7bdac16165ce22b9ec0dffdce4d44a5b_r.jpg)

设置成功后再来执行上面的go install命令。

```text
go install main.go
```

![img](https://pic2.zhimg.com/v2-6fc9a1c636a62ef85b00c69a484ab929_r.jpg)

上图看到，执行后可执行文件自动安装到bin目录下了。

go install的参数跟go build一样，这里不再说明。



## go get

1. download
2. compile
3. install

go get命令用于动态获取远程代码包及其依赖包，并进行编译和安装。

执行命令后一般会下载在GOPATH的src目录下。

```text
go get github.com/PuerkitoBio/goquery
```

![img](https://pic1.zhimg.com/v2-05dcd14655955c50af1b084cf42f3778_r.jpg)

当想要修改package的版本时，只需要go get package@指定的version即可

上图所示，无法下载“golang.org”网站内容，这是由于国内网络限制原因，当出现这种情况一般有3种方法解决：

1.手动下载

2.设置代理

3.使用GOPROXY环境变量

1.手动下载方式，不推荐，因为可能有太多的依赖（或依赖的依赖）关系。



2.设置代理

若你有代理，可以使用如下命令

```text
export http_proxy=http://proxyAddress:port
export https_proxy=http://proxyAddress:port
```

或

```text
export all_proxy=http://proxyAddress:port
```



3.使用GOPROXY环境变量

推荐使用这种方法，但前提是Go版本是1.11及以后。

所以需要设置GOPROXY环境变量，设置后下载源码将通过这个环境变量设置的代理地址进行下载。

[goproxy.io](https://github.com/goproxyio/goproxy)这个开源项目帮我们实现了代理功能，并提供了公用的代理服务：

```text
https://goproxy.io
```



- **设置GOPROXY**

使用公用代理服务[https://goproxy.io](https://goproxy.io/)

```text
export GOPROXY=https://goproxy.io
```

由于需要依赖go module功能，所以需要开启MODULE

```text
export GO111MODULE=on
```

最后再来执行上面的go get命令：

```text
go get github.com/PuerkitoBio/goquery
```

![img](https://pic3.zhimg.com/v2-424bd779d55b40c8a87ee46bba9da9ea_r.jpg)

不过这种方法下载的包源码在GOPATH目录下的pkg/mod目录下。

若需要关闭GOPROXY环境变量，只需：

```text
export GOPROXY=
```



另外，国内开源项目[goproxy.cn](https://github.com/goproxy/goproxy.cn)也提供了类似服务，公用代理服务：[https://goproxy.cn](https://goproxy.cn/)



go get参数说明

go get有一些参数，如下所示：

![img](https://pic3.zhimg.com/v2-fabb87d29391edf07edc36ce161cfa12_r.jpg)

`go install` 被设计为“**用于构建和安装二进制文件**”， `go get` 则被设计为 “**用于编辑 go.mod 变更依赖**”，并且使用时，应该与 `-d` 参数共用，现有版本默认加入。

总结而言，关于 `go install` 和 `go get` 必须要注意的是：

- 基本上 `go install <package>@<version>` 是用于命令的全局安装：
  - 例如：`go install sigs.k8s.io/kind@v0.9.0`;
- `go get` 安装二进制的功能，后续版本将会删除；
- `go get` 主要被设计为修改 `go.mod` 追加依赖之类的，但还存在类似 `go mod tidy` 之类的命令，所以使用频率可能不会很高

## go mod

go mod 是 go modules的简写，用于对go包的管理。从go1.11开始实现了modules管理，mod相比以前的方式，优点主要体现在：

- 项目不需要放在GOPATH下的src目录了，可以在任意目录。
- 自动下载第三方包和依赖包。
- 第三方包会指定版本号。
- 项目内会生成一个go.mod文件，文件内指定包依赖关系。
- 有一些第三方包不存在了或转移到其他地方，不需要改代码，只需用replace命令替换即可。

要使用modules，前提是需要将环境变量GO111MODULE设为on或者auto:

```text
GO111MODULE=on
```

![img](https://pic3.zhimg.com/v2-676cd867714e5dffe24549f083c126b2_r.jpg)

GO111MODULE有三个值：

- GO111MODULE=off ： go命令行将不会支持module功能，寻找依赖包的方式将会沿用旧版本那种通过vendor目录或者GOPATH模式来查找。
- GO111MODULE=on ：go命令行会使用modules，不会去GOPATH目录下查找。
- GO111MODULE=auto ： 这是默认值，会根据当前目录的位置来决定是否启用modules功能，若项目在GOPATH的src之外且根目录下有go.mod文件时会启用modules。



**下面将演示如何使用mod来管理项目**

- 在任意目录下创建一个自己的项目

zarten.go

```text
package main

import "fmt"

func main(){
	fmt.Println("Zarten")
}
```

如下图所示：

![img](https://pic4.zhimg.com/v2-d9e219fcff2bb74b82fcabfbe78bc8b7_r.jpg)

首先使用"go mod init 模块名" 初始化项目

```text
go mod init my
```

![img](https://pic1.zhimg.com/v2-aaa2f223ea3e73fc05386048cef5b830_r.jpg)

上图看到，执行init后在该目录下生成了一个go.mod文件，此时go.mod文件中包含了模块名称和当前go的版本号。

![img](https://pic1.zhimg.com/v2-99323c45d070c267c03d6d9f44cfef74_r.jpg)



- **引用第三方包**

此时引用第三方包，以beego（一个web服务器）为例，看mod是如何工作的。

```text
package main

import (
	"fmt"
	"github.com/astaxie/beego"
)

func main(){
	fmt.Println("Zarten")
	beego.Run()
}
```

编写完成后直接"go build zarten.go" 或 "go run zarten.go"命令，go会自动查找依赖关系并下载写入go.mod文件中，并生成go.sum文件（用来检测依赖模块是否被篡改）。

注：若在检测依赖时，由于网络原因需要用代理，在上面有讲解如何使用代理！

```text
go build zarten.go
```

![img](https://pic2.zhimg.com/v2-987a65c844db84a381de560617a2fb59_r.jpg)

此时查看go.mod文件，如下所示：

![img](https://pic4.zhimg.com/v2-9bdac86ca381bfd6502ca7e858d3346f_r.jpg)

可以看到require后面是引用的包，并且还指定了包的版本v1.12.2

下载的包保存在了GOPATH/pkg/mod下面，并且可以多版本共存保存。

![img](https://pic3.zhimg.com/v2-0f3ecb64561d81b63fd53307cc4aac22_r.jpg)

若需要下载不同的版本，需要在go.mod文件的require后面指定版本号。



- **依赖包地址变更的做法**

上面我们说到，mod管理的好处是项目地址变更后无需更改源代码，只需使用replace命令即可。

例如zarten这个包从http://golang.org/x/zarten 移动到了 http://github.com/golang/zarten

在go.mod文件中，使用replace命令如下：

```text
replace golang.org/x/zarten => github.com/golang/zarten latest
```



另外go mod有如下命令：

![img](https://pic1.zhimg.com/v2-672380e7f2800f67a6a9c8dbb5fcc668_r.jpg)



可通过官网查看每个go版本中mod的变化：[点击查看](https://github.com/golang/go/wiki/Modules)

go mod的常见错误问题及解决方案查看这里：

- [在go modules里使用go get进行包管理](https://www.cnblogs.com/apocelipes/p/9537659.html)
- [go1.13 mod 实践和常见问题](https://www.cnblogs.com/mingbai/p/go13moduleuser.html)

## go run

go run用于编译并运行源码文件，由于包含编译步骤，所以go build参数都可用于go run，在go run 中只接受go源码文件而不接受代码包。

```text
go run zarten.go
```

![img](https://pic1.zhimg.com/v2-87d9e288c37e06cdea90ca064135501c_r.jpg)



## go clean

go clean命令用于删除执行其他命令时产生的文件或目录，这些文件包括：

- _obj/ 旧的object目录，由Makefiles遗留
- _test/ 旧的test目录，由Makefiles遗留
- _testmain.go 旧的gotest文件，由Makefiles遗留
- test.out 旧的test记录，由Makefiles遗留
- build.out 旧的test记录，由Makefiles遗留
- *.[568ao] object文件，由Makefiles遗留
- DIR(.exe) 由go build产生
- DIR.test(.exe) 由go test -c产生
- MAINFILE(.exe) 由go build MAINFILE.go产生
- *.so 由 SWIG 产生

参数包括：

- `-i` 清除关联的安装的包和可运行文件，也就是通过go install安装的文件
- `-n` 把需要执行的清除命令打印出来，但是不执行，这样就可以很容易的知道底层是如何运行的
- `-r` 循环的清除在import中引入的包
- `-x` 打印出来执行的详细命令，其实就是`-n`打印的执行版本

一般使用go clean命令清理干净后上传到代码托管。

执行go clean -i -n 进行清理（根据go版本不同，可能需要mod支持）

```text
go clean -i -n
```

![img](https://pic3.zhimg.com/v2-4b1bb35197a264ff1415947aa3d15bfe_r.jpg)



## go fmt

go fmt用于检查并格式化成go语言的规范格式。

我们知道，在go语言中有严格的代码规范，否则编译不通过。不过大部分编译器已经自动实现规范格式，但此命名还是非常有用的。

参数说明：

- `-l` 显示那些需要格式化的文件
- `-w` 把改写后的内容直接写入到文件中，而不是作为结果打印到标准输出。
- `-r` 添加形如“a[b:len(a)] -> a[b:]”的重写规则，方便我们做批量替换
- `-s` 简化文件中的代码
- `-d` 显示格式化前后的diff而不是写入文件，默认是false
- `-e` 打印所有的语法错误到标准输出。如果不使用此标记，则只会打印不同行的前10个错误。
- `-cpuprofile` 支持调试模式，写入相应的cpufile到指定的文件

go fmt可以格式化整个项目或某个go文件，一般使用gofmt工具。

例如在my目录下有2个不规范文件：zarten.go 和 zarten2.go

zarten.go

```text
package main

import (
	"fmt"
)

func main() {
				fmt.Println("Zarten")
	fmt.Println("Zarten")

}
```

zarten2.go

```text
package main

import (
	"fmt"
)

func main() {
fmt.Println("Zarten2") 



 fmt.Println("Zarten2")
}
```

显示项目内需要格式的文件：

```text
gofmt -l my
```

![img](https://pic3.zhimg.com/v2-89135b3f0d393f1a726f7b1ee84fbd5e_b.jpg)

加上参数-w，自动写入文件中：

```text
gofmt -l -w my
```

![img](https://pic2.zhimg.com/v2-e51e001039856efb787b5616a8ad48c5_b.jpg)

查看文件发现，格式已经改写成规范格式了。

同时gofmt也可对单个文件使用，例如zarten.go

```text
gofmt zarten.go
```

将规范化的代码打印出来，但没有将规范化写入文件中

![img](https://pic1.zhimg.com/v2-9cd313be45353df8a831cf1f966a51e4_r.jpg)

要将写入文件中需要加"-w"参数：

```text
gofmt -w zarten.go
```



## go list

go list 会列出当前安装的包

```text
go list
```

![img](https://pic1.zhimg.com/v2-0a981e3b0b6d368eff42a5bfe7a43c8c_r.jpg)



## go version

go version 可以查看当前go的版本

```text
go version
```

![img](https://pic3.zhimg.com/v2-34e2ac1685fd97dad35dea6229b46096_r.jpg)



## go env

go env 可以查看当前go的环境变量

```text
go env
```

![img](https://pic2.zhimg.com/v2-98fa411e500125207428f4ebbbf9ab51_r.jpg)



## 交叉编译各大平台命令

有时我们需要在某一平台编译出其他平台的可执行文件，go语言也提供了相关的命令或配置

**在Mac平台编译：**

- Linux

```text
CGO_ENABLED=0  GOOS=linux  GOARCH=amd64  go build main.go
```

- Windows

```text
CGO_ENABLED=0 GOOS=windows  GOARCH=amd64  go  build  main.go
```



**在Linux平台编译：**

- Mac

```text
CGO_ENABLED=0 GOOS=darwin  GOARCH=amd64  go build main.go
```

- Windows

```text
CGO_ENABLED=0 GOOS=windows  GOARCH=amd64  go build main.go
```



**在Windows平台编译：**

- Mac

```text
SET CGO_ENABLED=0 SET GOOS=darwin SET GOARCH=amd64 go build main.go
```

或写成批处理

```text
SET  CGO_ENABLED=0
SET GOOS=darwin
SET GOARCH=amd64
go build main.go
```

- Linux

```text
SET CGO_ENABLED=0 SET GOOS=linux SET GOARCH=amd64 go build main.go
```

或写成批处理

```text
SET CGO_ENABLED=0
SET GOOS=linux
SET GOARCH=amd64
go build main.go
```

## 在Goland中的设置交叉编译

例如编译在linux下的程序设置：

```text
GOARCH=amd64;GOOS=linux
```

![img](https://pic1.zhimg.com/v2-547e243567da4313a3b97152fa2c9094_r.jpg)

## 总结

上面讲到命令都是非常有用的，还有一些没讲到的可以通过“go”命令查看，具体需要查看哪个命令，可以“go help 命令”。