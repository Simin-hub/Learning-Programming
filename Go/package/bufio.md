# bufio

bufio包实现了有缓冲的I/O。它包装一个io.Reader或io.Writer接口对象，创建另一个也实现了该接口，且同时还提供了缓冲和一些文本I/O的帮助函数的对象。

### type [Reader](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#31)

```
type Reader struct {
    // 内含隐藏或非导出字段
}
```

Reader实现了给一个io.Reader接口对象附加缓冲。

#### func [NewReader](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#61)

```
func NewReader(rd io.Reader) *Reader
```

NewReader创建一个具有默认大小缓冲、从r读取的*Reader。

#### func [NewReaderSize](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#46)

```
func NewReaderSize(rd io.Reader, size int) *Reader
```

NewReaderSize创建一个具有最少有size尺寸的缓冲、从r读取的*Reader。如果参数r已经是一个具有足够大缓冲的* Reader类型值，会返回r。

#### func (*Reader) [Reset](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#67)

```
func (b *Reader) Reset(r io.Reader)
```

Reset丢弃缓冲中的数据，清除任何错误，将b重设为其下层从r读取数据。

#### func (*Reader) [Buffered](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#261)

```
func (b *Reader) Buffered() int
```

Buffered返回缓冲中现有的可读取的字节数。

#### func (*Reader) [Peek](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#123)

```
func (b *Reader) Peek(n int) ([]byte, error)
```

Peek返回输入流的下n个字节，而不会移动读取位置。返回的[]byte只在下一次调用读取操作前合法。如果Peek返回的切片长度比n小，它也会返会一个错误说明原因。如果n比缓冲尺寸还大，返回的错误将是ErrBufferFull。

#### func (*Reader) [Read](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#153)

```
func (b *Reader) Read(p []byte) (n int, err error)
```

Read读取数据写入p。本方法返回写入p的字节数。本方法一次调用最多会调用下层Reader接口一次Read方法，因此返回值n可能小于len(p)。读取到达结尾时，返回值n将为0而err将为io.EOF。

#### func (*Reader) [ReadByte](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#193)

```
func (b *Reader) ReadByte() (c byte, err error)
```

ReadByte读取并返回一个字节。如果没有可用的数据，会返回错误。

#### func (*Reader) [UnreadByte](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#208)

```
func (b *Reader) UnreadByte() error
```

UnreadByte吐出最近一次读取操作读取的最后一个字节。（只能吐出最后一个，多次调用会出问题）

#### func (*Reader) [ReadRune](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#228)

```
func (b *Reader) ReadRune() (r rune, size int, err error)
```

ReadRune读取一个utf-8编码的unicode码值，返回该码值、其编码长度和可能的错误。如果utf-8编码非法，读取位置只移动1字节，返回U+FFFD，返回值size为1而err为nil。如果没有可用的数据，会返回错误。

#### func (*Reader) [UnreadRune](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#250)

```
func (b *Reader) UnreadRune() error
```

UnreadRune吐出最近一次ReadRune调用读取的unicode码值。如果最近一次读取不是调用的ReadRune，会返回错误。（从这点看，UnreadRune比UnreadByte严格很多）

#### func (*Reader) [ReadLine](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#325)

```
func (b *Reader) ReadLine() (line []byte, isPrefix bool, err error)
```

ReadLine是一个低水平的行数据读取原语。大多数调用者应使用ReadBytes('\n')或ReadString('\n')代替，或者使用Scanner。

ReadLine尝试返回一行数据，不包括行尾标志的字节。如果行太长超过了缓冲，返回值isPrefix会被设为true，并返回行的前面一部分。该行剩下的部分将在之后的调用中返回。返回值isPrefix会在返回该行最后一个片段时才设为false。返回切片是缓冲的子切片，只在下一次读取操作之前有效。ReadLine要么返回一个非nil的line，要么返回一个非nil的err，两个返回值至少一个非nil。

返回的文本不包含行尾的标志字节（"\r\n"或"\n"）。如果输入流结束时没有行尾标志字节，方法不会出错，也不会指出这一情况。在调用ReadLine之后调用UnreadByte会总是吐出最后一个读取的字节（很可能是该行的行尾标志字节），即使该字节不是ReadLine返回值的一部分。

#### func (*Reader) [ReadSlice](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#273)

```
func (b *Reader) ReadSlice(delim byte) (line []byte, err error)
```

ReadSlice读取直到第一次遇到delim字节，返回缓冲里的包含已读取的数据和delim字节的切片。该返回值只在下一次读取操作之前合法。如果ReadSlice放在在读取到delim之前遇到了错误，它会返回在错误之前读取的数据在缓冲中的切片以及该错误（一般是io.EOF）。如果在读取到delim之前缓冲就被写满了，ReadSlice失败并返回ErrBufferFull。因为ReadSlice的返回值会被下一次I/O操作重写，调用者应尽量使用ReadBytes或ReadString替代本法功法。当且仅当ReadBytes方法返回的切片不以delim结尾时，会返回一个非nil的错误。

#### func (*Reader) [ReadBytes](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#367)

```
func (b *Reader) ReadBytes(delim byte) (line []byte, err error)
```

ReadBytes读取直到第一次遇到delim字节，返回一个包含已读取的数据和delim字节的切片。如果ReadBytes方法在读取到delim之前遇到了错误，它会返回在错误之前读取的数据以及该错误（一般是io.EOF）。当且仅当ReadBytes方法返回的切片不以delim结尾时，会返回一个非nil的错误。

#### func (*Reader) [ReadString](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#415)

```
func (b *Reader) ReadString(delim byte) (line string, err error)
```

ReadString读取直到第一次遇到delim字节，返回一个包含已读取的数据和delim字节的字符串。如果ReadString方法在读取到delim之前遇到了错误，它会返回在错误之前读取的数据以及该错误（一般是io.EOF）。当且仅当ReadString方法返回的切片不以delim结尾时，会返回一个非nil的错误。

#### func (*Reader) [WriteTo](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#422)

```
func (b *Reader) WriteTo(w io.Writer) (n int64, err error)
```

WriteTo方法实现了io.WriterTo接口。

### type [Writer](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#479)

```
type Writer struct {
    // 内含隐藏或非导出字段
}
```

Writer实现了为io.Writer接口对象提供缓冲。如果在向一个Writer类型值写入时遇到了错误，该对象将不再接受任何数据，且所有写操作都会返回该错误。在说有数据都写入后，调用者有义务调用Flush方法以保证所有的数据都交给了下层的io.Writer。

Example

#### func [NewWriter](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#505)

```
func NewWriter(w io.Writer) *Writer
```

NewWriter创建一个具有默认大小缓冲、写入w的*Writer。

#### func [NewWriterSize](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#489)

```
func NewWriterSize(w io.Writer, size int) *Writer
```

NewWriterSize创建一个具有最少有size尺寸的缓冲、写入w的*Writer。如果参数w已经是一个具有足够大缓冲的*Writer类型值，会返回w。

#### func (*Writer) [Reset](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#511)

```
func (b *Writer) Reset(w io.Writer)
```

Reset丢弃缓冲中的数据，清除任何错误，将b重设为将其输出写入w。

#### func (*Writer) [Buffered](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#550)

```
func (b *Writer) Buffered() int
```

Buffered返回缓冲中已使用的字节数。

#### func (*Writer) [Available](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#547)

```
func (b *Writer) Available() int
```

Available返回缓冲中还有多少字节未使用。

#### func (*Writer) [Write](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#556)

```
func (b *Writer) Write(p []byte) (nn int, err error)
```

Write将p的内容写入缓冲。返回写入的字节数。如果返回值nn < len(p)，还会返回一个错误说明原因。

#### func (*Writer) [WriteString](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#626)

```
func (b *Writer) WriteString(s string) (int, error)
```

WriteString写入一个字符串。返回写入的字节数。如果返回值nn < len(s)，还会返回一个错误说明原因。

#### func (*Writer) [WriteByte](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#581)

```
func (b *Writer) WriteByte(c byte) error
```

WriteByte写入单个字节。

#### func (*Writer) [WriteRune](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#595)

```
func (b *Writer) WriteRune(r rune) (size int, err error)
```

WriteRune写入一个unicode码值（的utf-8编码），返回写入的字节数和可能的错误。

#### func (*Writer) [Flush](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#518)

```
func (b *Writer) Flush() error
```

Flush方法将缓冲中的数据写入下层的io.Writer接口。

#### func (*Writer) [ReadFrom](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#645)

```
func (b *Writer) ReadFrom(r io.Reader) (n int64, err error)
```

ReadFrom实现了io.ReaderFrom接口。

### type [ReadWriter](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#690)

```
type ReadWriter struct {
    *Reader
    *Writer
}
```

ReadWriter类型保管了指向Reader和Writer类型的指针，（因此）实现了io.ReadWriter接口。

#### func [NewReadWriter](https://github.com/golang/go/blob/master/src/bufio/bufio.go?name=release#696)

```
func NewReadWriter(r *Reader, w *Writer) *ReadWriter
```

NewReadWriter申请创建一个新的、将读写操作分派给r和w 的ReadWriter。

### type [SplitFunc](https://github.com/golang/go/blob/master/src/bufio/scan.go?name=release#57)

```
type SplitFunc func(data []byte, atEOF bool) (advance int, token []byte, err error)
```

SplitFunc类型代表用于对输出作词法分析的分割函数。

参数data是尚未处理的数据的一个开始部分的切片，参数atEOF表示是否Reader接口不能提供更多的数据。返回值是解析位置前进的字节数，将要返回给调用者的token切片，以及可能遇到的错误。如果数据不足以（保证）生成一个完整的token，例如需要一整行数据但data里没有换行符，SplitFunc可以返回(0, nil, nil)来告诉Scanner读取更多的数据写入切片然后用从同一位置起始、长度更长的切片再试一次（调用SplitFunc类型函数）。

如果返回值err非nil，扫描将终止并将该错误返回给Scanner的调用者。

除非atEOF为真，永远不会使用空切片data调用SplitFunc类型函数。然而，如果atEOF为真，data却可能是非空的、且包含着未处理的文本。

### func [ScanBytes](https://github.com/golang/go/blob/master/src/bufio/scan.go?name=release#213)

```
func ScanBytes(data []byte, atEOF bool) (advance int, token []byte, err error)
```

ScanBytes是用于Scanner类型的分割函数（符合SplitFunc），本函数会将每个字节作为一个token返回。

### func [ScanRunes](https://github.com/golang/go/blob/master/src/bufio/scan.go?name=release#228)

```
func ScanRunes(data []byte, atEOF bool) (advance int, token []byte, err error)
```

ScanRunes是用于Scanner类型的分割函数（符合SplitFunc），本函数会将每个utf-8编码的unicode码值作为一个token返回。本函数返回的rune序列和range一个字符串的输出rune序列相同。错误的utf-8编码会翻译为U+FFFD = "\xef\xbf\xbd"，但只会消耗一个字节。调用者无法区分正确编码的rune和错误编码的rune。

### func [ScanWords](https://github.com/golang/go/blob/master/src/bufio/scan.go?name=release#319)

```
func ScanWords(data []byte, atEOF bool) (advance int, token []byte, err error)
```

ScanRunes是用于Scanner类型的分割函数（符合SplitFunc），本函数会将空白（参见unicode.IsSpace）分隔的片段（去掉前后空白后）作为一个token返回。本函数永远不会返回空字符串。

### func [ScanLines](https://github.com/golang/go/blob/master/src/bufio/scan.go?name=release#274)

```
func ScanLines(data []byte, atEOF bool) (advance int, token []byte, err error)
```

ScanRunes是用于Scanner类型的分割函数（符合SplitFunc），本函数会将每一行文本去掉末尾的换行标记作为一个token返回。返回的行可以是空字符串。换行标记为一个可选的回车后跟一个必选的换行符。最后一行即使没有换行符也会作为一个token返回。

### type [Scanner](https://github.com/golang/go/blob/master/src/bufio/scan.go?name=release#30)

```
type Scanner struct {
    // 内含隐藏或非导出字段
}
```

Scanner类型提供了方便的读取数据的接口，如从换行符分隔的文本里读取每一行。

成功调用的Scan方法会逐步提供文件的token，跳过token之间的字节。token由SplitFunc类型的分割函数指定；默认的分割函数会将输入分割为多个行，并去掉行尾的换行标志。本包预定义的分割函数可以将文件分割为行、字节、unicode码值、空白分隔的word。调用者可以定制自己的分割函数。

扫描会在抵达输入流结尾、遇到的第一个I/O错误、token过大不能保存进缓冲时，不可恢复的停止。当扫描停止后，当前读取位置可能会远在最后一个获得的token后面。需要更多对错误管理的控制或token很大，或必须从reader连续扫描的程序，应使用bufio.Reader代替。

#### func [NewScanner](https://github.com/golang/go/blob/master/src/bufio/scan.go?name=release#74)

```
func NewScanner(r io.Reader) *Scanner
```

NewScanner创建并返回一个从r读取数据的Scanner，默认的分割函数是ScanLines。

#### func (*Scanner) [Split](https://github.com/golang/go/blob/master/src/bufio/scan.go?name=release#206)

```
func (s *Scanner) Split(split SplitFunc)
```

Split设置该Scanner的分割函数。本方法必须在Scan之前调用。

#### func (*Scanner) [Scan](https://github.com/golang/go/blob/master/src/bufio/scan.go?name=release#110)

```
func (s *Scanner) Scan() bool
```

**Scan方法获取当前位置的token（该token可以通过Bytes或Text方法获得），并让Scanner的扫描位置移动到下一个token。当扫描因为抵达输入流结尾或者遇到错误而停止时**，本方法会返回false。在Scan方法返回false后，Err方法将返回扫描时遇到的任何错误；除非是io.EOF，此时Err会返回nil。

#### func (*Scanner) [Bytes](https://github.com/golang/go/blob/master/src/bufio/scan.go?name=release#94)

```
func (s *Scanner) Bytes() []byte
```

Bytes方法返回最近一次Scan调用生成的token。底层数组指向的数据可能会被下一次Scan的调用重写。

#### func (*Scanner) [Text](https://github.com/golang/go/blob/master/src/bufio/scan.go?name=release#100)

```
func (s *Scanner) Text() string
```

Bytes方法返回最近一次Scan调用生成的token，会申请创建一个字符串保存token并返回该字符串。

#### func (*Scanner) [Err](https://github.com/golang/go/blob/master/src/bufio/scan.go?name=release#84)

```
func (s *Scanner) Err() error
```

Err返回Scanner遇到的第一个非EOF的错误。