# cobra

先了解[flag](https://studygolang.com/pkgdoc)库

[cobra](http://github.com/spf13/cobra)是一个命令行程序库，可以用来编写命令行程序。同时，它也提供了一个脚手架， 用于生成基于 cobra 的应用程序框架。非常多知名的开源项目使用了 cobra 库构建命令行，如[Kubernetes](http://kubernetes.io/)、[Hugo](http://gohugo.io/)、[etcd](https://github.com/coreos/etcd)等等等等。 

cobra 中有个重要的概念，分别是 commands、arguments 和 flags。其中 commands 代表行为，arguments 就是命令行参数(或者称为位置参数)，flags 代表对行为的改变(也就是我们常说的命令行选项)。执行命令行程序时的一般格式为：
**APPNAME COMMAND ARG --FLAG**
比如下面的例子：

```
# server是 commands，port 是 flag
hugo server --port=1313

# clone 是 commands，URL 是 arguments，brae 是 flag
git clone URL --bare
```

## 安装

```
go get -u github.com/spf13/cobra/cobra
```

## 创建 cobra 应用

然后就可以用 cobra 程序生成应用程序框架了：

先创建目录 cobra_demo 进入再初始化

```
cobra init --pkg-name cobra_demo
```

初始化成功后出现如下提示信息：

```
Your Cobra application is ready at
```

此时项目结构应如下：

```
cobra_demo/
    cmd/
　　　　root.go
    main.go
    LICENSE
```

## 使用 cobra 程序生成命令代码

```
$ cd cobra_demo
$ cobra add image
$ cobra add container
```

此时项目结构应如下：

```
cobra_demo/
    cmd/
　　　　root.go
              image.go
             container.go
    main.go
    LICENSE
```

这两条命令分别生成了 cobra_demo 程序中 image 和 container 子命令的代码，当然了，具体的功能还得靠我们自己实现。

## 为命令添加具体的功能

到目前为止，我们一共为 cobra_demo 程序添加了三个 Command，分别是 rootCmd(cobra init 命令默认生成)、imageCmd 和 containerCmd。
打开文件 root.go ，找到变量 rootCmd 的初始化过程并为之设置 Run 方法：

```
Run: func(cmd *cobra.Command, args []string) {
    fmt.Println("cobra demo program")
},
```

重新编译 cobra_demo 程序并不带参数运行，这次就不再输出帮助信息了，而是执行了 rootCmd 的 Run 方法：

![img](https://img.kancloud.cn/52/4f/524f9bcf15c3bbc893f585b2ed1979f8_511x37.png)

再创建一个 version Command 用来输出当前的软件版本

```
cobra add version
```

这时在 cmd 目录下自动生成添加 version.go 文件，编辑文件的内容如下：

```
package cmd

import (
    "fmt"

    "github.com/spf13/cobra"
)

func init() {
    rootCmd.AddCommand(versionCmd)
}

var versionCmd = &cobra.Command{
    Use:   "version",
    Short: "Print the version number of cobrademo",
    Long:  `All software has versions. This is cobrademo's`,
    Run: func(cmd *cobra.Command, args []string) {
        fmt.Println("cobrademo version is v1.0")
    },
}
```