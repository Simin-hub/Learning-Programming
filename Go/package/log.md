# log

[参考地址](https://studygolang.com/pkgdoc)

[参考地址2](https://www.kancloud.cn/gofor/golang-learn/2571756)

log包实现了简单的日志服务。本包定义了Logger类型，该类型提供了一些格式化输出的方法。本包也提供了一个预定义的“标准”Logger，可以通过辅助函数Print[f|ln]、Fatal[f|ln]和Panic[f|ln]访问，比手工创建一个Logger对象更容易使用。Logger会打印每条日志信息的日期、时间，默认输出到标准错误。Fatal系列函数会在写入日志信息后调用os.Exit(1)。Panic系列函数会在写入日志信息后panic。

## Go Logger的优势和劣势

### 优势

它最大的优点是使用非常简单。我们可以设置任何`io.Writer`作为日志记录输出并向其发送要写入的日志。

### 劣势

- 仅限基本的日志级别
- 只有一个`Print`选项。不支持`INFO`/`DEBUG`等多个级别。
- 对于错误日志，它有`Fatal`和`Panic`
- Fatal日志通过调用`os.Exit(1)`来结束程序
- Panic日志在写入日志消息之后抛出一个panic
- 但是它缺少一个ERROR日志级别，这个级别可以在不抛出panic或退出程序的情况下记录错误
- 缺乏日志格式化的能力——例如记录调用者的函数名和行号，格式化日期和时间格式。等等。
- 不提供日志切割的能力。