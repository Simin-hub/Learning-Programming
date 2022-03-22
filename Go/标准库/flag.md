# flag - 命令行参数解析

[参考](https://darjun.github.io/2020/01/10/godailylib/flag/)

[参考2](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter13/13.1.html)

[pkgdoc](https://studygolang.com/pkgdoc)

`flag`用于解析命令行选项。有过类 Unix 系统使用经验的童鞋对命令行选项应该不陌生。例如命令`ls -al`列出当前目录下所有文件和目录的详细信息，其中`-al`就是命令行选项。

命令行选项在实际开发中很常用，特别是在写工具的时候。

- 指定配置文件的路径，如`redis-server ./redis.conf`以当前目录下的配置文件`redis.conf`启动 Redis 服务器；
- 自定义某些参数，如`python -m SimpleHTTPServer 8080`启动一个 HTTP 服务器，监听 8080 端口。如果不指定，则默认监听 8000 端口。

## 示例

```go
package main

import (
  "fmt"
  "flag"
)

var (
  intflag int
  boolflag bool
  stringflag string
)

func init() {
  flag.IntVar(&intflag, "intflag", 0, "int flag value")
  flag.BoolVar(&boolflag, "boolflag", false, "bool flag value")
  flag.StringVar(&stringflag, "stringflag", "default", "string flag value")
}

func main() {
  flag.Parse()

  fmt.Println("int flag:", intflag)
  fmt.Println("bool flag:", boolflag)
  fmt.Println("string flag:", stringflag)
}
```

编译：

```
go build -o main.exe main.go
./main.exe 
012 -boolflag 1 -stringflag test
```

输出：

```
int flag: 12
bool flag: true
string flag: test
```

如果不设置某个选项，相应变量会取默认值：

```
./main.exe -intflag 12 -boolflag 1
```

输出：

可以看到没有设置的选项`stringflag`为默认值`default`。

还可以直接使用`go run`，这个命令会先编译程序生成可执行文件，然后执行该文件，将命令行中的其它选项传给这个程序。

```
go run main.go -intflag 12 -boolflag 1
```

可以使用`-h`显示选项帮助信息：

```fallback
./main.exe -h
Usage of D:\code\golang\src\github.com\darjun\cmd\flag\main.exe:
  -boolflag
        bool flag value
  -intflag int
        int flag value
  -stringflag string
        string flag value (default "default")
```

总结一下，使用`flag`库的一般步骤：

- **定义一些全局变量存储选项的值**，如这里的`intflag/boolflag/stringflag`；
- **在`init`方法中使用`flag.TypeVar`方法定义选项**，这里的`Type`可以为基本类型`Int/Uint/Float64/Bool`，还可以是时间间隔`time.Duration`。定义时传入变量的地址、选项名、默认值和帮助信息；
- 在`main`方法中调用`flag.Parse`从`os.Args[1:]`中解析选项。因为`os.Args[0]`为可执行程序路径，会被剔除。

注意点：

**`flag.Parse`方法必须在所有选项都定义之后调用，且`flag.Parse`调用之后不能再定义选项**。如果按照前面的步骤，基本不会出现问题。 因为`init`在所有代码之前执行，将选项定义都放在`init`中，`main`函数中执行`flag.Parse`时所有选项都已经定义了。

要求：

使用flag.String(), Bool(), Int()等函数注册flag，下例声明了一个整数flag，**解析结果保存在*int指针ip**里：

```
import "flag"
var ip = flag.Int("flagname", 1234, "help message for flagname")
```

如果你喜欢，也可以将flag绑定到一个变量，使用Var系列函数：

```
var flagvar int
func init() {
	flag.IntVar(&flagvar, "flagname", 1234, "help message for flagname")
}
```

或者你可以自定义一个用于flag的类型（满足Value接口）并将该类型用于flag解析，如下：

```
flag.Var(&flagVal, "name", "help message for flagname")
```

对这种flag，默认值就是该变量的初始值。

在所有flag都注册之后，调用：

```
flag.Parse()
```

来解析命令行参数写入注册的flag里。

解析之后，flag的值可以直接使用。如果你使用的是flag自身，它们是指针；如果你绑定到了某个变量，它们是值。

```
fmt.Println("ip has value ", *ip)
fmt.Println("flagvar has value ", flagvar)
```

解析后，flag后面的参数可以从flag.Args()里获取或用flag.Arg(i)单独获取。这些参数的索引为从0到flag.NArg()-1。

命令行flag语法：

```
-flag
-flag=x
-flag x  // 只有非bool类型的flag可以
```

可以使用1个或2个'-'号，效果是一样的。**最后一种格式不能用于bool类型的flag**，因为如果有文件名为0、false等时,如下命令：

```
cmd -x *
```

其含义会改变。你必须使用-flag=false格式来关闭一个bool类型flag。

Flag解析在第一个非flag参数（单个"-"不是flag参数）之前停止，或者在终止符"--"之后停止。

整数flag接受1234、0664、0x1234等类型，也可以是负数。bool类型flag可以是：

```
1, 0, t, f, T, F, true, false, TRUE, FALSE, True, False
```

时间段flag接受任何合法的可提供给time.ParseDuration的输入。

默认的命令行flag集被包水平的函数控制。FlagSet类型允许程序员定义独立的flag集，例如实现命令行界面下的子命令。FlagSet的方法和包水平的函数是非常类似的。

[Variables](https://studygolang.com/static/pkgdoc/pkg/flag.htm#pkg-variables)

[type Value](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Value)

[type Getter](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Getter)

[type ErrorHandling](https://studygolang.com/static/pkgdoc/pkg/flag.htm#ErrorHandling)

[type Flag](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Flag)

[type FlagSet](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet)

- [func NewFlagSet(name string, errorHandling ErrorHandling) *FlagSet](https://studygolang.com/static/pkgdoc/pkg/flag.htm#NewFlagSet)
- [func (f *FlagSet) Init(name string, errorHandling ErrorHandling)](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.Init)
- [func (f *FlagSet) NFlag() int](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.NFlag)
- [func (f *FlagSet) Lookup(name string) *Flag](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.Lookup)
- [func (f *FlagSet) NArg() int](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.NArg)
- [func (f *FlagSet) Args() [\]string](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.Args)
- [func (f *FlagSet) Arg(i int) string](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.Arg)
- [func (f *FlagSet) PrintDefaults()](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.PrintDefaults)
- [func (f *FlagSet) SetOutput(output io.Writer)](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.SetOutput)
- [func (f *FlagSet) Bool(name string, value bool, usage string) *bool](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.Bool)
- [func (f *FlagSet) BoolVar(p *bool, name string, value bool, usage string)](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.BoolVar)
- [func (f *FlagSet) Int(name string, value int, usage string) *int](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.Int)
- [func (f *FlagSet) IntVar(p *int, name string, value int, usage string)](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.IntVar)
- [func (f *FlagSet) Int64(name string, value int64, usage string) *int64](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.Int64)
- [func (f *FlagSet) Int64Var(p *int64, name string, value int64, usage string)](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.Int64Var)
- [func (f *FlagSet) Uint(name string, value uint, usage string) *uint](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.Uint)
- [func (f *FlagSet) UintVar(p *uint, name string, value uint, usage string)](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.UintVar)
- [func (f *FlagSet) Uint64(name string, value uint64, usage string) *uint64](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.Uint64)
- [func (f *FlagSet) Uint64Var(p *uint64, name string, value uint64, usage string)](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.Uint64Var)
- [func (f *FlagSet) Float64(name string, value float64, usage string) *float64](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.Float64)
- [func (f *FlagSet) Float64Var(p *float64, name string, value float64, usage string)](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.Float64Var)
- [func (f *FlagSet) String(name string, value string, usage string) *string](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.String)
- [func (f *FlagSet) StringVar(p *string, name string, value string, usage string)](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.StringVar)
- [func (f *FlagSet) Duration(name string, value time.Duration, usage string) *time.Duration](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.Duration)
- [func (f *FlagSet) DurationVar(p *time.Duration, name string, value time.Duration, usage string)](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.DurationVar)
- [func (f *FlagSet) Var(value Value, name string, usage string)](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.Var)
- [func (f *FlagSet) Set(name, value string) error](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.Set)
- [func (f *FlagSet) Parse(arguments [\]string) error](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.Parse)
- [func (f *FlagSet) Parsed() bool](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.Parsed)
- [func (f *FlagSet) Visit(fn func(*Flag))](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.Visit)
- [func (f *FlagSet) VisitAll(fn func(*Flag))](https://studygolang.com/static/pkgdoc/pkg/flag.htm#FlagSet.VisitAll)

[func NFlag() int](https://studygolang.com/static/pkgdoc/pkg/flag.htm#NFlag)

[func Lookup(name string) *Flag](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Lookup)

[func NArg() int](https://studygolang.com/static/pkgdoc/pkg/flag.htm#NArg)

[func Args() [\]string](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Args)

[func Arg(i int) string](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Arg)

[func PrintDefaults()](https://studygolang.com/static/pkgdoc/pkg/flag.htm#PrintDefaults)

[func Bool(name string, value bool, usage string) *bool](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Bool)

[func BoolVar(p *bool, name string, value bool, usage string)](https://studygolang.com/static/pkgdoc/pkg/flag.htm#BoolVar)

[func Int(name string, value int, usage string) *int](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Int)

[func IntVar(p *int, name string, value int, usage string)](https://studygolang.com/static/pkgdoc/pkg/flag.htm#IntVar)

[func Int64(name string, value int64, usage string) *int64](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Int64)

[func Int64Var(p *int64, name string, value int64, usage string)](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Int64Var)

[func Uint(name string, value uint, usage string) *uint](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Uint)

[func UintVar(p *uint, name string, value uint, usage string)](https://studygolang.com/static/pkgdoc/pkg/flag.htm#UintVar)

[func Uint64(name string, value uint64, usage string) *uint64](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Uint64)

[func Uint64Var(p *uint64, name string, value uint64, usage string)](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Uint64Var)

[func Float64(name string, value float64, usage string) *float64](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Float64)

[func Float64Var(p *float64, name string, value float64, usage string)](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Float64Var)

[func String(name string, value string, usage string) *string](https://studygolang.com/static/pkgdoc/pkg/flag.htm#String)

[func StringVar(p *string, name string, value string, usage string)](https://studygolang.com/static/pkgdoc/pkg/flag.htm#StringVar)

[func Duration(name string, value time.Duration, usage string) *time.Duration](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Duration)

[func DurationVar(p *time.Duration, name string, value time.Duration, usage string)](https://studygolang.com/static/pkgdoc/pkg/flag.htm#DurationVar)

[func Var(value Value, name string, usage string)](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Var)

[func Set(name, value string) error](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Set)

[func Parse()](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Parse)

[func Parsed() bool](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Parsed)

[func Visit(fn func(*Flag))](https://studygolang.com/static/pkgdoc/pkg/flag.htm#Visit)

[func VisitAll(fn func(*Flag))](https://studygolang.com/static/pkgdoc/pkg/flag.htm#VisitAll)