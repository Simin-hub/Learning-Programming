# Zap Logger

[参考地址](https://www.topgoer.com/%E9%A1%B9%E7%9B%AE/log/ZapLogger.html)

### 1.1.1. Uber-go Zap

[Zap](https://github.com/uber-go/zap)是非常快的、结构化的，分日志级别的Go日志库。

### 1.1.2. 为什么选择Uber-go zap

- 它同时提供了结构化日志记录和printf风格的日志记录
- 它非常的快

根据Uber-go Zap的文档，它的性能比类似的结构化日志包更好——也比标准库更快。 以下是Zap发布的基准测试信息

记录一条消息和10个字段:

![img](https://www.topgoer.com/static/10.3/1.png)

记录一个静态字符串，没有任何上下文或printf风格的模板：

![img](https://www.topgoer.com/static/10.3/2.png)

### 1.1.3. 安装

运行下面的命令安装zap

```
go get -u go.uber.org/zap
```

### 1.1.4. 配置Zap Logger

Zap提供了两种类型的日志记录器—Sugared Logger和Logger。

在**性能很好但不是很关键的上下文中，使用SugaredLogger**。它比其他结构化日志记录包快4-10倍，并且支持结构化和printf风格的日志记录。

在每一微秒和每一次内存分配都很重要的上下文中，使用Logger。它甚至比SugaredLogger更快，内存分配次数也更少，但**它只支持强类型的结构化日志记录**。

### 1.1.5. Logger

- 通过调用`zap.NewProduction()`/`zap.NewDevelopment()`或者`zap.Example()`创建一个Logger。
- 上面的每一个函数都将创建一个logger。**唯一的区别在于它将记录的信息不同**。例如production logger默认记录调用函数信息、日期和时间等。
- 通过Logger调用Info/Error等。
- 默认情况下日志都会打印到应用程序的console界面。

```go
package main

import (
    "net/http"

    "go.uber.org/zap"
)

var logger *zap.Logger

func main() {
    InitLogger()
    defer logger.Sync()
    simpleHttpGet("www.5lmh.com")
    simpleHttpGet("http://www.google.com")
}

func InitLogger() {
    logger, _ = zap.NewProduction()
}

func simpleHttpGet(url string) {
    resp, err := http.Get(url)
    if err != nil {
        logger.Error(
            "Error fetching url..",
            zap.String("url", url),
            zap.Error(err))
    } else {
        logger.Info("Success..",
            zap.String("statusCode", resp.Status),
            zap.String("url", url))
        resp.Body.Close()
    }
}
```

在上面的代码中，我们首先创建了一个Logger，然后使用Info/ Error等Logger方法记录消息。

日志记录器方法的语法是这样的：

```
    func (log *Logger) MethodXXX(msg string, fields ...Field)
```

**其中MethodXXX是一个可变参数函数，可以是Info / Error/ Debug / Panic等。每个方法都接受一个消息字符串和任意数量的zapcore.Field场参数。**

每个zapcore.Field其实就是一组键值对参数。

我们执行上面的代码会得到如下输出结果：

```
{"level":"error","ts":1573180648.858149,"caller":"ce2/main.go:25","msg":"Error fetching url..","url":"www.5lmh.com","error":"Get www.5lmh.com: unsupported protocol scheme \"\"","stacktrace":"main.simpleHttpGet\n\te:/goproject/src/github.com/student/log/ce2/main.go:25\nmain.main\n\te:/goproject/src/github.com/student/log/ce2/main.go:14\nruntime.main\n\tE:/go/src/runtime/proc.go:200"}

{"level":"error","ts":1573180669.9273467,"caller":"ce2/main.go:25","msg":"Error fetching url..","url":"http://www.google.com","error":"Get http://www.google.com: dial tcp 31.13.72.54:80: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.","stacktrace":"main.simpleHttpGet\n\te:/goproject/src/github.com/student/log/ce2/main.go:25\nmain.main\n\te:/goproject/src/github.com/student/log/ce2/main.go:15\nruntime.main\n\tE:/go/src/runtime/proc.go:200"}
```

### 1.1.6. Sugared Logger

现在让我们使用Sugared Logger来实现相同的功能。

- 大部分的实现基本都相同。
- 惟一的区别是，我们通过调用主logger的. Sugar()方法来获取一个SugaredLogger。
- 然后使用SugaredLogger以printf格式记录语句

下面是修改过后使用SugaredLogger代替Logger的代码：

```go
package main

import (
    "net/http"

    "go.uber.org/zap"
)

var sugarLogger *zap.SugaredLogger

func main() {
    InitLogger()
    defer sugarLogger.Sync()
    simpleHttpGet("www.5lmh.com")
    simpleHttpGet("http://www.google.com")
}

func InitLogger() {
    logger, _ := zap.NewProduction()
    sugarLogger = logger.Sugar()
}

func simpleHttpGet(url string) {
    sugarLogger.Debugf("Trying to hit GET request for %s", url)
    resp, err := http.Get(url)
    if err != nil {
        sugarLogger.Errorf("Error fetching URL %s : Error = %s", url, err)
    } else {
        sugarLogger.Infof("Success! statusCode = %s for URL %s", resp.Status, url)
        resp.Body.Close()
    }
}
{"level":"error","ts":1573180998.3522997,"caller":"ce3/main.go:27","msg":"Error fetching URL www.5lmh.com : Error = Get www.5lmh.com: unsupported protocol scheme \"\"","stacktrace":"main.simpleHttpGet\n\te:/goproject/src/github.com/student/log/ce3/main.go:27\nmain.main\n\te:/goproject/src/github.com/student/log/ce3/main.go:14\nruntime.main\n\tE:/go/src/runtime/proc.go:200"}

{"level":"error","ts":1573181019.3677258,"caller":"ce3/main.go:27","msg":"Error fetching URL http://www.google.com : Error = Get http://www.google.com: dial tcp 67.228.37.26:80: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.","stacktrace":"main.simpleHttpGet\n\te:/goproject/src/github.com/student/log/ce3/main.go:27\nmain.main\n\te:/goproject/src/github.com/student/log/ce3/main.go:15\nruntime.main\n\tE:/go/src/runtime/proc.go:200"}
```

你应该注意到的了，到目前为止这两个logger都打印输出JSON结构格式。

在本博客的后面部分，我们将更详细地讨论SugaredLogger，并了解如何进一步配置它。

### 1.1.7. 定制logger

#### 将日志写入文件而不是终端

- 我们要做的第一个更改是把日志写入文件，而不是打印到应用程序控制台。

```
    func New(core zapcore.Core, options ...Option) *Logger
```

zapcore.Core需要三个配置——Encoder，WriteSyncer，LogLevel。

1.Encoder:编码器(如何写入日志)。我们将使用开箱即用的NewJSONEncoder()，并使用预先设置的ProductionEncoderConfig()。

```
   zapcore.NewJSONEncoder(zap.NewProductionEncoderConfig())
```

2.WriterSyncer ：指定日志将写到哪里去。我们使用zapcore.AddSync()函数并且将打开的文件句柄传进去。

```
   file, _ := os.Create("./test.log")
   writeSyncer := zapcore.AddSync(file)
```

3.Log Level：哪种级别的日志将被写入。

我们将修改上述部分中的Logger代码，并重写InitLogger()方法。其余方法—main() /SimpleHttpGet()保持不变。

```go
package main

import (
    "net/http"
    "os"

    "go.uber.org/zap"
    "go.uber.org/zap/zapcore"
)

var sugarLogger *zap.SugaredLogger

func main() {
    InitLogger()
    defer sugarLogger.Sync()
    simpleHttpGet("www.5lmh.com")
    simpleHttpGet("http://www.google.com")
}

func InitLogger() {
    writeSyncer := getLogWriter()
    encoder := getEncoder()
    core := zapcore.NewCore(encoder, writeSyncer, zapcore.DebugLevel)

    logger := zap.New(core)
    sugarLogger = logger.Sugar()
}

func getEncoder() zapcore.Encoder {
    return zapcore.NewJSONEncoder(zap.NewProductionEncoderConfig())
}

func getLogWriter() zapcore.WriteSyncer {

    file, _ := os.Create("./test.log")
    return zapcore.AddSync(file)
}

func simpleHttpGet(url string) {
    sugarLogger.Debugf("Trying to hit GET request for %s", url)
    resp, err := http.Get(url)
    if err != nil {
        sugarLogger.Errorf("Error fetching URL %s : Error = %s", url, err)
    } else {
        sugarLogger.Infof("Success! statusCode = %s for URL %s", resp.Status, url)
        resp.Body.Close()
    }
}
```

当使用这些修改过的logger配置调用上述部分的main()函数时，以下输出将打印在文件——test.log中。

```
{"level":"debug","ts":1573181732.5292294,"msg":"Trying to hit GET request for www.5lmh.com"}
{"level":"error","ts":1573181732.5292294,"msg":"Error fetching URL www.5lmh.com : Error = Get www.5lmh.com: unsupported protocol scheme \"\""}
{"level":"debug","ts":1573181732.5292294,"msg":"Trying to hit GET request for http://www.google.com"}
{"level":"error","ts":1573181753.564804,"msg":"Error fetching URL http://www.google.com : Error = Get http://www.google.com: dial tcp 66.220.149.32:80: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond."}
```

#### 将JSON Encoder更改为普通的Log Encoder

现在，我们希望将编码器从JSON Encoder更改为普通Encoder。为此，我们需要将NewJSONEncoder()更改为NewConsoleEncoder()。

```
    return zapcore.NewConsoleEncoder(zap.NewProductionEncoderConfig())
```

当使用这些修改过的logger配置调用上述部分的main()函数时，以下输出将打印在文件——test.log中。

```
1.573181811861697e+09    debug    Trying to hit GET request for www.5lmh.com
1.5731818118626883e+09    error    Error fetching URL www.5lmh.com : Error = Get www.5lmh.com: unsupported protocol scheme ""
1.5731818118626883e+09    debug    Trying to hit GET request for http://www.google.com
1.5731818329012108e+09    error    Error fetching URL http://www.google.com : Error = Get http://www.google.com: dial tcp 216.58.200.228:80: connectex: A connection attempt failed because the connected party did not properly respond after a period of time, or established connection failed because connected host has failed to respond.
```

#### 更改时间编码并添加调用者详细信息

鉴于我们对配置所做的更改，有下面两个问题：

- 时间是以非人类可读的方式展示，例如1.572161051846623e+09
- 调用方函数的详细信息没有显示在日志中 我们要做的第一件事是覆盖默认的ProductionConfig()，并进行以下更改:

修改时间编码器

- 在日志文件中使用大写字母记录日志级别

```go
func getEncoder() zapcore.Encoder {
    encoderConfig := zap.NewProductionEncoderConfig()
    encoderConfig.EncodeTime = zapcore.ISO8601TimeEncoder
    encoderConfig.EncodeLevel = zapcore.CapitalLevelEncoder
    return zapcore.NewConsoleEncoder(encoderConfig)
}
```

接下来，我们将修改zap logger代码，添加将调用函数信息记录到日志中的功能。为此，我们将在zap.New(..)函数中添加一个Option。

```
    logger := zap.New(core, zap.AddCaller())
```

## 使用Lumberjack进行日志切割归档

这个日志程序中唯一缺少的就是日志切割归档功能。

> *Zap本身不支持切割归档日志文件*

为了添加日志切割归档功能，我们将使用第三方库[Lumberjack](https://link.zhihu.com/?target=https%3A//github.com/natefinch/lumberjack)来实现。

### 安装

执行下面的命令安装Lumberjack

```bash
go get -u github.com/natefinch/lumberjack
```

### zap logger中加入Lumberjack

要在zap中加入Lumberjack支持，我们需要修改`WriteSyncer`代码。我们将按照下面的代码修改`getLogWriter()`函数：

```go
func getLogWriter() zapcore.WriteSyncer {
    lumberJackLogger := &lumberjack.Logger{
        Filename:   "./test.log",
        MaxSize:    10,
        MaxBackups: 5,
        MaxAge:     30,
        Compress:   false,
    }
    return zapcore.AddSync(lumberJackLogger)
}
```

Lumberjack Logger采用以下属性作为输入:

- Filename: 日志文件的位置
- MaxSize：在进行切割之前，日志文件的最大大小（以MB为单位）
- MaxBackups：保留旧文件的最大个数
- MaxAges：保留旧文件的最大天数
- Compress：是否压缩/归档旧文件

### 测试所有功能

最终，使用Zap/Lumberjack logger的完整示例代码如下：

```go
package main

import (
    "net/http"

    "github.com/natefinch/lumberjack"
    "go.uber.org/zap"
    "go.uber.org/zap/zapcore"
)

var sugarLogger *zap.SugaredLogger

func main() {
    InitLogger()
    defer sugarLogger.Sync()
    simpleHttpGet("www.sogo.com")
    simpleHttpGet("http://www.sogo.com")
}

func InitLogger() {
    writeSyncer := getLogWriter()
    encoder := getEncoder()
    core := zapcore.NewCore(encoder, writeSyncer, zapcore.DebugLevel)

    logger := zap.New(core, zap.AddCaller())
    sugarLogger = logger.Sugar()
}

func getEncoder() zapcore.Encoder {
    encoderConfig := zap.NewProductionEncoderConfig()
    encoderConfig.EncodeTime = zapcore.ISO8601TimeEncoder
    encoderConfig.EncodeLevel = zapcore.CapitalLevelEncoder
    return zapcore.NewConsoleEncoder(encoderConfig)
}

func getLogWriter() zapcore.WriteSyncer {
    lumberJackLogger := &lumberjack.Logger{
        Filename:   "./test.log",
        MaxSize:    1,
        MaxBackups: 5,
        MaxAge:     30,
        Compress:   false,
    }
    return zapcore.AddSync(lumberJackLogger)
}

func simpleHttpGet(url string) {
    sugarLogger.Debugf("Trying to hit GET request for %s", url)
    resp, err := http.Get(url)
    if err != nil {
        sugarLogger.Errorf("Error fetching URL %s : Error = %s", url, err)
    } else {
        sugarLogger.Infof("Success! statusCode = %s for URL %s", resp.Status, url)
        resp.Body.Close()
    }
}
```

执行上述代码，下面的内容会输出到文件——test.log中。

```go
2019-10-27T15:50:32.944+0800    DEBUG   logic/temp2.go:48   Trying to hit GET request for www.sogo.com
2019-10-27T15:50:32.944+0800    ERROR   logic/temp2.go:51   Error fetching URL www.sogo.com : Error = Get www.sogo.com: unsupported protocol scheme ""
2019-10-27T15:50:32.944+0800    DEBUG   logic/temp2.go:48   Trying to hit GET request for http://www.sogo.com
2019-10-27T15:50:33.165+0800    INFO    logic/temp2.go:53   Success! statusCode = 200 OK for URL http://www.sogo.com
```

同时，可以在`main`函数中循环记录日志，测试日志文件是否会自动切割和归档（日志文件每1MB会切割并且在当前目录下最多保存5个备份）。

至此，我们总结了如何将Zap日志程序集成到Go应用程序项目中。