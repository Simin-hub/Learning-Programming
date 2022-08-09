# fmt

### func [Printf](https://github.com/golang/go/blob/master/src/fmt/print.go?name=release#196)

```
func Printf(format string, a ...interface{}) (n int, err error)
```

Printf根据format参数生成格式化的字符串并写入标准输出。返回写入的字节数和遇到的任何错误。

### func [Fprintf](https://github.com/golang/go/blob/master/src/fmt/print.go?name=release#186)

```
func Fprintf(w io.Writer, format string, a ...interface{}) (n int, err error)
```

Fprintf根据format参数生成格式化的字符串并写入w。返回写入的字节数和遇到的任何错误。

### func [Sprintf](https://github.com/golang/go/blob/master/src/fmt/print.go?name=release#201)

```
func Sprintf(format string, a ...interface{}) string
```

Sprintf根据format参数生成格式化的字符串并返回该字符串。

### func [Print](https://github.com/golang/go/blob/master/src/fmt/print.go?name=release#231)

```
func Print(a ...interface{}) (n int, err error)
```

Print采用默认格式将其参数格式化并写入标准输出。如果两个相邻的参数都不是字符串，会在它们的输出之间添加空格。返回写入的字节数和遇到的任何错误。

### func [Fprint](https://github.com/golang/go/blob/master/src/fmt/print.go?name=release#220)

```
func Fprint(w io.Writer, a ...interface{}) (n int, err error)
```

Fprint采用默认格式将其参数格式化并写入w。如果两个相邻的参数都不是字符串，会在它们的输出之间添加空格。返回写入的字节数和遇到的任何错误。

### func [Sprint](https://github.com/golang/go/blob/master/src/fmt/print.go?name=release#237)

```
func Sprint(a ...interface{}) string
```

Sprint采用默认格式将其参数格式化，串联所有输出生成并返回一个字符串。如果两个相邻的参数都不是字符串，会在它们的输出之间添加空格。

### func [Println](https://github.com/golang/go/blob/master/src/fmt/print.go?name=release#263)

```
func Println(a ...interface{}) (n int, err error)
```

Println采用默认格式将其参数格式化并写入标准输出。总是会在相邻参数的输出之间添加空格并在输出结束后添加换行符。返回写入的字节数和遇到的任何错误。

### func [Fprintln](https://github.com/golang/go/blob/master/src/fmt/print.go?name=release#252)

```
func Fprintln(w io.Writer, a ...interface{}) (n int, err error)
```

Fprintln采用默认格式将其参数格式化并写入w。总是会在相邻参数的输出之间添加空格并在输出结束后添加换行符。返回写入的字节数和遇到的任何错误。

### func [Sprintln](https://github.com/golang/go/blob/master/src/fmt/print.go?name=release#269)

```
func Sprintln(a ...interface{}) string
```

Sprintln采用默认格式将其参数格式化，串联所有输出生成并返回一个字符串。总是会在相邻参数的输出之间添加空格并在输出结束后添加换行符。

### func [Errorf](https://github.com/golang/go/blob/master/src/fmt/print.go?name=release#211)

```
func Errorf(format string, a ...interface{}) error
```

Errorf根据format参数生成格式化字符串并返回一个包含该字符串的错误。

### func [Scanf](https://github.com/golang/go/blob/master/src/fmt/scan.go?name=release#84) 从标准输入扫描文本，根据format 参数指定的格式将成功读取的空白分隔的值保存进成功传递给本函数的参数

```
func Scanf(format string, a ...interface{}) (n int, err error)
```

Scanf从标准输入扫描文本，根据format 参数指定的格式**将成功读取的空白分隔的值保存进成功传递给本函数的参数**。返回成功扫描的条目个数和遇到的任何错误。

### func [Fscanf](https://github.com/golang/go/blob/master/src/fmt/scan.go?name=release#143)

```
func Fscanf(r io.Reader, format string, a ...interface{}) (n int, err error)
```

Fscanf从r扫描文本，根据format 参数指定的格式将成功读取的空白分隔的值保存进成功传递给本函数的参数。返回成功扫描的条目个数和遇到的任何错误。

### func [Sscanf](https://github.com/golang/go/blob/master/src/fmt/scan.go?name=release#116)

```
func Sscanf(str string, format string, a ...interface{}) (n int, err error)
```

Sscanf从字符串str扫描文本，根据format 参数指定的格式将成功读取的空白分隔的值保存进成功传递给本函数的参数。返回成功扫描的条目个数和遇到的任何错误。

### func [Scan](https://github.com/golang/go/blob/master/src/fmt/scan.go?name=release#71) 将成功读取的空白分隔的值保存进成功传递给本函数的参数。换行视为空白

```
func Scan(a ...interface{}) (n int, err error)
```

Scan从标准输入扫描文本，将成功读取的空白分隔的值保存进成功传递给本函数的参数。换行视为空白。返回成功扫描的条目个数和遇到的任何错误。如果读取的条目比提供的参数少，会返回一个错误报告原因。

```
_, err := fmt.Scan(&n)
if err == io.EOF {
	// 
}
```

**scan将碰到第一个空格或换行符之前的内容赋值给变量**。如果scan中有多个变量，变量值用空格或换行符分割。所以换行和空格是不能存储到变量内的。

Scan和Scanln基本相同，唯一区别是当读取多个变量当时候，遇到换行符Scanln会直接结束，未读到输入值的变量为零值；Scan会等待，直到输入的值满足参数的个数后再遇到换行符才会结束。

### func [Fscan](https://github.com/golang/go/blob/master/src/fmt/scan.go?name=release#124)

```
func Fscan(r io.Reader, a ...interface{}) (n int, err error)
```

Fscan从r扫描文本，将成功读取的空白分隔的值保存进成功传递给本函数的参数。换行视为空白。返回成功扫描的条目个数和遇到的任何错误。如果读取的条目比提供的参数少，会返回一个错误报告原因。

### func [Sscan](https://github.com/golang/go/blob/master/src/fmt/scan.go?name=release#103)

```
func Sscan(str string, a ...interface{}) (n int, err error)
```

Sscan从字符串str扫描文本，将成功读取的空白分隔的值保存进成功传递给本函数的参数。换行视为空白。返回成功扫描的条目个数和遇到的任何错误。如果读取的条目比提供的参数少，会返回一个错误报告原因。

### func [Scanln](https://github.com/golang/go/blob/master/src/fmt/scan.go?name=release#77) 在换行时才停止扫描

```
func Scanln(a ...interface{}) (n int, err error)
```

Scanln类似Scan，但会在换行时才停止扫描。最后一个条目后必须有换行或者到达结束位置。

### func [Fscanln](https://github.com/golang/go/blob/master/src/fmt/scan.go?name=release#133)

```
func Fscanln(r io.Reader, a ...interface{}) (n int, err error)
```

Fscanln类似Fscan，但会在换行时才停止扫描。最后一个条目后必须有换行或者到达结束位置。

### func [Sscanln](https://github.com/golang/go/blob/master/src/fmt/scan.go?name=release#109)

```
func Sscanln(str string, a ...interface{}) (n int, err error)
```

Sscanln类似Sscan，但会在换行时才停止扫描。最后一个条目后必须有换行或者到达结束位置。