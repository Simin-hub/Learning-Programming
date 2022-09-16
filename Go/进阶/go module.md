# go module

[参考](https://colobu.com/2021/06/28/dive-into-go-module-1/)、[参考](https://www.infoq.cn/article/xyjhjja87y7pvu1iwhz3)、[参考](https://objcoding.com/2018/09/13/go-modules/)

go官方库管理方式叫做go module。 先前，我们的库都是以package来组织的，package以一个文件或者多个文件实现单一的功能，一个项目包含一个package或者多个package。Go module就是一组统一打版和发布的package的集合，在根文件下有go.mod文件定义module path和依赖库的版本，package以子文件夹的形式存在module中，对package path就是 module path +"/"+ package name的形式。

**一般我们项目都是单module的形式，项目主文件夹下包含go.mod,子文件夹定义package，或者主文件夹也是一个package**。但是一个项目也可以包含多个module,只不过这种方式不常用而已。

## go.mod

go module最重要的是go.mod文件的定义，它用来标记一个module和它的依赖库以及依赖库的版本。会放在module的主文件夹下，一般以`go.mod`命名。

一个go.mod内容类似下面的格式:

```
module github.com/panicthis/modfile

go 1.16

require (
	github.com/cenk/backoff v2.2.1+incompatible
	github.com/coreos/bbolt v1.3.3
	github.com/edwingeng/doublejump v0.0.0-20200330080233-e4ea8bd1cbed
	github.com/stretchr/objx v0.3.0 // indirect
	github.com/stretchr/testify v1.7.0
	go.etcd.io/bbolt v1.3.6 // indirect
	go.etcd.io/etcd/client/v2 v2.305.0-rc.1
	go.etcd.io/etcd/client/v3 v3.5.0-rc.1
	golang.org/x/net v0.0.0-20210610132358-84b48f89b13b // indirect
	golang.org/x/sys v0.0.0-20210611083646-a4fc73990273 // indirect
)

exclude (
	go.etcd.io/etcd/client/v2 v2.305.0-rc.0
	go.etcd.io/etcd/client/v3 v3.5.0-rc.0
)

retract (
    v1.0.0 // 废弃的版本，请使用v1.1.0
)
```

### 语义化版本 2.0.0

Go module遵循[语义化版本规范 2.0.0](https://semver.org/lang/zh-CN/)。语义化版本规范 2.0.0规定了版本号的格式，每个字段的意义以及版本号比较的规则等等。

[![img](https://colobu.com/2021/06/28/dive-into-go-module-1/semver2.png)](https://colobu.com/2021/06/28/dive-into-go-module-1/semver2.png)

如果你想为你的项目发版，你可以设置tag为上面的格式，比如`v1.3.0`、`v2.0.0-rc.1`等等。metadata中在Go版本比较时是不参与运算的，只是一个辅助信息。

### module path

go.mod的第一行是module path, 一般采用仓库+module name的方式定义。这样我们获取一个module的时候，就可以到它的仓库中去查询，或者让go proxy到仓库中去查询。

```
module github.com/panicthis/modfile
```

如果你的版本已经大于等于2.0.0，按照Go的规范，你应该加上major的后缀，module path改成下面的方式:

```
module github.com/panicthis/modfile/v2
module github.com/panicthis/modfile/v3
```

而且引用代码的时候，也要加上`v2`、`v3`、`vx`后缀，以便和其它major版本进行区分。

这是一个很奇怪的约定，带来的好处是你一个项目中可以使用依赖库的不同的major版本，它们可以共存。

### go directive

第二行是go directive。格式是 `go 1.xx`,它并不是指你当前使用的Go版本，而是**指名你的代码所需要的Go的最低版本**。

```
go 1.16
```

因为Go的标准库也有所变化，一些新的API也被增加进来，如果你的代码用到了这些新的API,你可能需要指名它依赖的go版本。

这一行不是必须的，你可以不写。

### require

require段中列出了项目所需要的各个依赖库以及它们的版本，除了正规的`v1.3.0`这样的版本外，还有一些奇奇怪怪的版本和注释，那么它们又是什么意思呢？

正式的版本号我们就不需要介绍了，大家都懂:

```
github.com/coreos/bbolt v1.3.3
```

#### 伪版本号

[![img](https://colobu.com/2021/06/28/dive-into-go-module-1/version1.png)](https://colobu.com/2021/06/28/dive-into-go-module-1/version1.png)

```
github.com/edwingeng/doublejump v0.0.0-20200330080233-e4ea8bd1cbed
```

上面这个库中的版本号就是一个伪版本号`v0.0.0-20200330080233-e4ea8bd1cbed`,这是go module为它生成的一个类似符合语义化版本2.0.0版本，实际这个库并没有发布这个版本。

正式因为这个依赖库没有发布版本，而go module需要指定这个库的一个确定的版本，所以才创建的这样一个伪版本号。

go module的目的就是在go.mod中标记出这个项目所有的依赖以及它们确定的某个版本。

这里的`20200330080233`是这次提交的时间，格式是`yyyyMMddhhmmss`, 而`e4ea8bd1cbed`就是这个版本的commit id,通过这个字段，就可以确定这个库的特定的版本。

而前面的`v0.0.0`可能有多种生成方式，主要看你这个commit的base version:

- vX.0.0-yyyymmddhhmmss-abcdefabcdef: 如果没有base version,那么就是vX.0.0的形式
- vX.Y.Z-pre.0.yyyymmddhhmmss-abcdefabcdef： 如果base version是一个预发布的版本，比如vX.Y.Z-pre,那么它就用vX.Y.Z-pre.0的形式
- vX.Y.(Z+1)-0.yyyymmddhhmmss-abcdefabcdef: 如果base version是一个正式发布的版本，那么它就patch号加1，如vX.Y.(Z+1)-0

[搞清楚 Go Mod的版本和伪版本，下次别乱用了](https://mp.weixin.qq.com/s/_vw-coi0TB0dn7IgWpXEpA)

### indirect注释

```
   	go.etcd.io/bbolt v1.3.6 // indirect
    golang.org/x/net v0.0.0-20210610132358-84b48f89b13b // indirect
    golang.org/x/sys v0.0.0-20210611083646-a4fc73990273 // indirect
```

有些库后面加了`indirect`后缀，这又是什么意思的。

**如果用一句话总结，间接的使用了这个库，但是又没有被列到某个go.mod中**，当然这句话也不算太准确，更精确的说法是下面的情况之一就会对这个库加indirect后缀：

- 当前项目依赖A,但是A的go.mod遗漏了B, 那么就会在当前项目的go.mod中补充B, 加indirect注释
- 当前项目依赖A,但是A没有go.mod,同样就会在当前项目的go.mod中补充B, 加indirect注释
- 当前项目依赖A,A又依赖B,当对A降级的时候，降级的A不再依赖B,这个时候B就标记indirect注释

### incompatible

有些库后面加了incompatible后缀，但是你如果看这些项目，它们只是发布了v2.2.1的tag,并没有`+incompatible`后缀。

```
github.com/cenk/backoff v2.2.1+incompatible
```

这些库采用了go.mod的管理，但是不幸的是，虽然这些库的版major版本已经大于等于2了，但是他们的module path中依然没有添加v2、v3这样的后缀。

所以go module把它们标记为`incompatible`的，虽然可以引用，但是实际它们是不符合规范的。

### exclude

如果你想**在你的项目中跳过某个依赖库的某个版本**，你就可以使用这个段。

```
exclude (
	go.etcd.io/etcd/client/v2 v2.305.0-rc.0
	go.etcd.io/etcd/client/v3 v3.5.0-rc.0
)
```

这样，Go在版本选择的时候，就会主动跳过这些版本，比如你使用`go get -u ......`或者`go get github.com/xxx/xxx@latest`等命令时，会执行version query的动作，这些版本不在考虑的范围之内。

### replace

replace也是常用的一个手段，**用来解决一些错误的依赖库的引用或者调试依赖库**。

```
replace github.com/coreos/bbolt => go.etcd.io/bbolt v1.3.3
replace github.com/panicthis/A v1.1.0 => github.com/panicthis/R v1.8.0
replace github.com/coreos/bbolt => ../R
```

比如etcd v3.3.x的版本中错误的使用了`github.com/coreos/bbolt`作为bbolt的module path,其实这个库在它自己的go.mod中声明的module path是`go.etcd.io/bbolt`，又比如etcd使用的grpc版本有问题，你也可以通过replace替换成所需的grpc版本。

甚至你觉得某个依赖库有问题，自己fork到本地做修改，想调试一下，你也可以替换成本地的文件夹。

replace可以替换某个库的所有版本到另一个库的特定版本，也可以替换某个库的特定版本到另一个库的特定版本。

### retract

retract是go 1.16中新增加的内容，借用学术界期刊撤稿的术语，宣布撤回库的某个版本。

如果你误发布了某个版本，或者事后发现某个版本不成熟，那么你可以推一个新的版本，在新的版本中，声明前面的某个版本被撤回，提示大家都不要用了。

撤回的版本tag依然还存在，go proxy也存在这个版本，所以你如果强制使用，还是可以使用的，否则这些版本就会被跳过。

**和exclude的区别是retract是这个库的owner定义的， 而exclude是库的使用者在自己的go.mod中定义的**。

## 语义化版本

[参考](https://colobu.com/2021/06/28/dive-into-go-module-2/)

Go module不但遵循[语义化版本规范 2.0.0](https://semver.org/lang/zh-CN/),而且还更进一步，对语义化版本中的major还还赋予了更深的意义。



[![img](https://colobu.com/2021/06/28/dive-into-go-module-2/semver2.png)](https://colobu.com/2021/06/28/dive-into-go-module-2/semver2.png)

- v0.X.X: 对于主版本号(major)是0的情况，**隐含你当前的API还处于不稳定的状态，新的小版本可能不向下兼容**
- v1.X.X: **当前的API处于稳定状态**，minor的增加只意味着新的feature的增加，API还是向下兼容的
- v2.X.X: major的增加意味着API已经不向下兼容了

> **问题**： 你知道在go module中，哪些版本号隐含当前API是不稳定的？

但是go module与众不同鹤立鸡群卓然不群的是，一旦你的major大于等于2, 你的module path必须加上v2后缀(如果tag是v3.X.X,那就是v3后缀，以此类推)。

而且，包引用路径也要加上v2，比如 `go.etcd.io/etcd/client/v3`。

这是一个怪异的写法，相当于在正常的易于理解的module path上加了一个狗屁膏药，以提示这个引入的库是哪个版本的？

为什么要加上这个v2、v2后缀的，肯定有一定的考虑。

最主要的，Go的开发者(这里指Russ Cox)在[import compatibility rule](https://research.swtch.com/vgo-import)指出:

> If an old package and a new package have the same import path,
> the new package must be backwards compatible with the old package.

也就是相同module path应该保证新的版本向下兼容。

[![img](https://colobu.com/2021/06/28/dive-into-go-module-2/deps.png)](https://colobu.com/2021/06/28/dive-into-go-module-2/deps.png)

这种想法是好的。比如你在你的项目中可以使用同一个库的多个版本， v1版本处理以前遗留的逻辑，v2版本处理新的逻辑，v3版本试验未来的版本，同一套库的不同版本可以共存，并不会出现版本冲突的地方。

而且程序员看到这些module path,也很清楚的知道版本不兼容了，谁是更新的版本。

但是这种方式也是很有争议的，在实践中中也带来了很多问题，我在开发rpcx深受其害，又比如etcd,你可以看它的v3.4.X的版本，就是因为没有加上v3的后缀，导致go命令下载或者导入(get)这些package的时候根本就下载不了。

### vX后缀污染了package path

本来正常的package path一般是仓库路径+package name,或者go module下 module path + package的方式，可是一旦版本大于等于2,就不得不加上一个后缀v2,v3等，将package path的含义改变了。

当然忍一忍我们还能接受，大不了闭着眼睛用呗，最痛苦的很多Go的初学者并不了解这种设置，不知道导入新的库的版本要加v2后缀，一脸茫然。

### v0, v1和v2数据类型不兼容

在module path中增加了v2,v3等后缀后，也就以为着这些package都是不同的package，虽然它们中大部分的数据类型并没有做改变，还是向下兼容的，也不能直接赋值，还是需要强转一下。

比如你的项目依赖`Auth 1.0.0`, 也依赖`Auth 2.0.0`,那么即使`A.Config`在两个版本中没做任何改变，你也不能把`Auth.Config`赋值给`Auth/v2.Config`,而是需要在代码中加上强转的逻辑，两两互转。一旦发布了v3,那就得三三互转，很长的一个switch分支处理这种情况，如果发布v4，那么逻辑更复杂了。

### 给第三方库的开发者带来了很大的负担

虽然你觉得我也就发布v2,v3,v4等几个版本，版本路线很清晰，管理起来也不复杂，没什么大不了的。

但是，如果你的库是一个非常流行的库，很多开发者基于你的库开发了第三方的库的话，就非常痛苦了。

这意味着一旦你发布了一个新的版本，这些第三方的开发者就必须及时的更新他们的库，基于你的新的版本发布他们新的v2，v3版本。这就像病毒一样，初步扩展开来。给广大的开发者带来的很大的负担。

当然，见仁见智，这些情况可能你不会遇到，或者也不会给你带来困扰，所以它不是一个问题。而我，在开发rpcx，或者解答一些网友的问题的时候，深深被v2伤害到了,小小的心灵无法承受v2之重。

一些开源项目，为了避免版本号跳到v2,采用了其它的一些办法，比如protobuf-go, 正在做新的版本的重构，改动非常大，不和以前的版本兼容了，可以以前的版本都v1.X.X了，那怎么办呢？换module path名称。

- github.com/golang/protobuf: 支持先前的protobuf go,目前最高版本v1.5.2
- google.golang.org/protobuf: 新版本的module path,目前最高版本v1.27.0，初始版本v1.20.0

对于我开发的rpcx项目，因为在go module出来之前版本号已经发布到了v6.X.X。 我想回到从前，貌似回不去了。所以我采用了一个极端的做法，把tag重建，所有的版本号都定义在v1.X.X内。还好影响的用户比较少，所以也没有用户抱怨。

我这种做法比较极端，没造成用户抱怨的原因是我一直坚持go module和GOPATH并存的方式。发版的时候采用go module发版，master开发分支上采用GOPATH方式，绝大部分用户都使用master分支，或者自己fork了一个新的版本，所以造成的影响很小。

## 改进

正常情况下，我们的go.mod依赖库的版本都是符合[语义化版本 2.0.0](https://semver.org/lang/zh-CN/)的版本格式，或者[伪版本格式](https://golang.org/ref/mod#non-module-compat#glos-pseudo-version)。Go使用服务端提交的日期和commit id生成的伪版本号是符合语义化版本号2.0.0的，因为语义化版本号中规定pre-release以连接号**-**加一连串以逗号分隔的标识符组成，标识符以字母数字和连接号组成，所以你看到`-yyyyMMddhhmmss-comitid`包含两个连接号，这是正常的。

go要求依赖库要么不包含go.mod,要么依赖库中的go.mod定义的依赖库版本必须以语义化版本 2.0.0格式(或伪版本号)标志(其实更严格，除了`+incompatible`不能加meta字段)，因为这样我们你能够明确标识某个依赖库确切的版本，这样的版本号被称之为[canonical version](https://golang.org/ref/mod#glos-canonical-version)。

其实main module还可以定义non-canonical version，通过go get或者go mod tidy更新go.mod的时候，命令会尝试更新go.mod,尝试把non-canonical version转变为canonical version版本。

但是，到底有哪些non-canonical version呢？我还没看到官方文章介绍，本文尝试整理这些non-canonical version。

### 只定义major或者major.minor

你可以不指定minor.patch或者patch,而是让go命令尝试去寻找最大的minor和patch,所以你可以在go.mod只定义vmajor或者vmajor.minor:

```
github.com/panicthis/B v1.2
github.com/panicthis/C v1
github.com/panicthis/G/v2 v2
```

运行`go mod tidy`它们会被转换成

```
github.com/panicthis/B v1.2.1
github.com/panicthis/C v1.4.0
github.com/panicthis/G/v2 v2.0.0
```

`go get`命令也一样，你也可以直接指定major，忽略minor和patch, 比如

```
go get github.com/panicthis/B@v1.2
go get github.com/panicthis/C@v1
go get github.com/panicthis/G/v2@v2
```

### latest, upgrade 和 patch

有三个单词有特别的语义

- latest: **选择最高的release版本**，如果没有release版本，则**选择最高的pre-release版本**，如果根本就没有打过tag,则选择最高的伪版本号的版本(默认分支的最后的提交版本)
- upgrade: 类似latest,但是如果有比release更高的版本(比如pre-release),会选择更高的版本
- patch: major和minor和当前的版本相同，只把patch升级到最高。当然如果没有当前的版本，则无从比较，则patch退化成latest语义

比如下面的格式:

```
github.com/panicthis/D latest
github.com/panicthis/E upgrade
github.com/panicthis/F patch
```

使用go get命令也一样

```
go get github.com/panicthis/D@latest
go get github.com/panicthis/E@upgrade
go get github.com/panicthis/F@patch //因为没有本地版本，所以此命令在go 1.16下可能出错
go get -u=patch github.com/panicthis/F@v1.1.0
```

### 指定特定的commit id

因为有时候proxy有缓存时间或者更新周期，如果你提交了一个新的commit,或者新打了一个tag,通过 `latest`不一定能拉取到最新的提交，这个时候你可以通过指定commit id的方式拉取。或者你就想测试某个特定的版本。

```
github.com/panicthis/H 4f7657a
```

或者

```
go get github.com/panicthis/H@4f7657a
```

甚至，你可以使用`HEAD`，作为你最新的commit id:

```
github.com/panicthis/H HEAD
```

或者

```
go get github.com/panicthis/H@HEAD
```

### 特定的分支

你还可以拉取特定的分支

```
github.com/panicthis/J master
github.com/panicthis/K feat-123
```

或者

```
go get github.com/panicthis/J@master
go get github.com/panicthis/K@feat-123
```

≈

更有甚者，你可以使用`>`、`>=`、`<`、`<=`比较符，选取某个符合条件的最大的版本。

比如:

```
github.com/stretchr/testify >v1.7.0
github.com/panicthis/F >=v1.1.0
github.com/panicthis/F <v1.1.0
github.com/panicthis/F <=v1.1.0
```

运行go mod tidy它会转换成canonical version。

或者(注意命令行中需要使用转义符)：

```
go get github.com/stretchr/testify@\>v1.7.0
go get -u=patch github.com/panicthis/F@\>=v1.1.0
go get -u=patch github.com/panicthis/F@\<v1.1.0
go get -u=patch github.com/panicthis/F@\<=v1.1.0
```

虽然我们可以在go.mod中使用non-canonical version，但是在提交和发布的时候，我们需要使用go mod tidy把它们转换成canonical version,让依赖库的版本对应一个确定的版本，否则`master`、`HEAD`在不同的人使用的时候可能会对应不同的版本。

### none

还有一个特殊的字符可以作为non-canonical version,比如

```
go get github.com/stretchr/testify@none
```

它会从go module中移出这个依赖。

### tip

有人提议支持go从开发分支上拉取最新的版本。 有几个单次可以候选，但是感觉从 hg/mercurial中借鉴来的`tip`很合适。go最新的开发版本也叫做`tip`。