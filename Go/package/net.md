# net

[参考地址](https://studygolang.com/pkgdoc)

[示例](https://studygolang.com/articles/20368)

net包提供了可移植的网络I/O接口，包括TCP/IP、UDP、域名解析和Unix域socket。

虽然本包提供了对网络原语的访问，大部分使用者只需要Dial、Listen和Accept函数提供的基本接口；以及相关的Conn和Listener接口。`crypto/tls`包提供了相同的接口和类似的Dial和Listen函数。

`Dial`函数和服务端建立连接：

```
conn, err := net.Dial("tcp", "google.com:80")
if err != nil {
	// handle error
}
fmt.Fprintf(conn, "GET / HTTP/1.0\r\n\r\n")
status, err := bufio.NewReader(conn).ReadString('\n')
// ...
```

`Listen`函数创建的服务端：

```
ln, err := net.Listen("tcp", ":8080")
if err != nil {
	// handle error
}
for {
	conn, err := ln.Accept()
	if err != nil {
		// handle error
		continue
	}
	go handleConnection(conn)
}
```

```
const (
    IPv4len = 4
    IPv6len = 16
)
var (
    IPv4bcast     = IPv4(255, 255, 255, 255) // 广播地址
    IPv4allsys    = IPv4(224, 0, 0, 1)       // 所有主机和路由器
    IPv4allrouter = IPv4(224, 0, 0, 2)       // 所有路由器
    IPv4zero      = IPv4(0, 0, 0, 0)         // 本地地址，只能作为源地址（曾用作广播地址）
)
var (
    IPv6zero                   = IP{0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0}
    IPv6unspecified            = IP{0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0}
    IPv6loopback               = IP{0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1}
    IPv6interfacelocalallnodes = IP{0xff, 0x01, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0x01}
    IPv6linklocalallnodes      = IP{0xff, 0x02, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0x01}
    IPv6linklocalallrouters    = IP{0xff, 0x02, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0x02}
)
var (
    ErrWriteToConnected = errors.New("use of WriteTo with pre-connected connection")
)
type ParseError // ParseError代表一个格式错误的字符串，Type为期望的格式
func (e *ParseError) Error() string
type Error //Error代表一个网络错误
type InvalidAddrError
func (e InvalidAddrError) Error() string
func (e InvalidAddrError) Temporary() bool
func (e InvalidAddrError) Timeout() bool
type UnknownNetworkError
func (e UnknownNetworkError) Error() string
func (e UnknownNetworkError) Temporary() bool
func (e UnknownNetworkError) Timeout() bool
type DNSConfigError //DNSConfigError代表读取主机DNS配置时出现的错误。
func (e *DNSConfigError) Error() string
func (e *DNSConfigError) Temporary() bool
func (e *DNSConfigError) Timeout() bool
type DNSError //DNSError代表DNS查询的错误
func (e *DNSError) Error() string
func (e *DNSError) Temporary() bool
func (e *DNSError) Timeout() bool
type AddrError
func (e *AddrError) Error() string
func (e *AddrError) Temporary() bool
func (e *AddrError) Timeout() bool
type OpError //OpError是经常被net包的函数返回的错误类型。它描述了该错误的操作、网络类型和网络地址
func (e *OpError) Error() string
func (e *OpError) Temporary() bool
func (e *OpError) Timeout() bool
func SplitHostPort(hostport string) (host, port string, err error) //函数将格式为"host:port"、"[host]:port"或"[ipv6-host%zone]:port"的网络地址分割为host或ipv6-host%zone和port两个部分。Ipv6的文字地址或者主机名必须用方括号括起来，如"[::1]:80"、"[ipv6-host]:http"、"[ipv6-host%zone]:80"。

func JoinHostPort(host, port string) string //函数将host和port合并为一个网络地址。一般格式为"host:port"；如果host含有冒号或百分号，格式为"[host]:port"。

type HardwareAddr
func ParseMAC(s string) (hw HardwareAddr, err error)
func (a HardwareAddr) String() string
type Flags
func (f Flags) String() string
type Interface
func InterfaceByIndex(index int) (*Interface, error)
func InterfaceByName(name string) (*Interface, error)
func (ifi *Interface) Addrs() ([]Addr, error)
func (ifi *Interface) MulticastAddrs() ([]Addr, error)
func Interfaces() ([]Interface, error)
func InterfaceAddrs() ([]Addr, error)
type IP
func IPv4(a, b, c, d byte) IP
func ParseIP(s string) IP
func (ip IP) IsGlobalUnicast() bool
func (ip IP) IsLinkLocalUnicast() bool
func (ip IP) IsInterfaceLocalMulticast() bool
func (ip IP) IsLinkLocalMulticast() bool
func (ip IP) IsMulticast() bool
func (ip IP) IsLoopback() bool
func (ip IP) IsUnspecified() bool
func (ip IP) DefaultMask() IPMask
func (ip IP) Equal(x IP) bool
func (ip IP) To16() IP
func (ip IP) To4() IP
func (ip IP) Mask(mask IPMask) IP
func (ip IP) String() string
func (ip IP) MarshalText() ([]byte, error)
func (ip *IP) UnmarshalText(text []byte) error
type IPMask
func IPv4Mask(a, b, c, d byte) IPMask
func CIDRMask(ones, bits int) IPMask
func (m IPMask) Size() (ones, bits int)
func (m IPMask) String() string
type IPNet
func ParseCIDR(s string) (IP, *IPNet, error)
func (n *IPNet) Contains(ip IP) bool
func (n *IPNet) Network() string
func (n *IPNet) String() string
type Addr
type Conn
func Dial(network, address string) (Conn, error)
func DialTimeout(network, address string, timeout time.Duration) (Conn, error)
func Pipe() (Conn, Conn)
type PacketConn
func ListenPacket(net, laddr string) (PacketConn, error)
type Dialer
func (d *Dialer) Dial(network, address string) (Conn, error)
type Listener
func Listen(net, laddr string) (Listener, error)
type IPAddr
func ResolveIPAddr(net, addr string) (*IPAddr, error)
func (a *IPAddr) Network() string
func (a *IPAddr) String() string
type TCPAddr
func ResolveTCPAddr(net, addr string) (*TCPAddr, error)
func (a *TCPAddr) Network() string
func (a *TCPAddr) String() string
type UDPAddr
func ResolveUDPAddr(net, addr string) (*UDPAddr, error)
func (a *UDPAddr) Network() string
func (a *UDPAddr) String() string
type UnixAddr
func ResolveUnixAddr(net, addr string) (*UnixAddr, error)
func (a *UnixAddr) Network() string
func (a *UnixAddr) String() string
type IPConn
func DialIP(netProto string, laddr, raddr *IPAddr) (*IPConn, error)
func ListenIP(netProto string, laddr *IPAddr) (*IPConn, error)
func (c *IPConn) LocalAddr() Addr
func (c *IPConn) RemoteAddr() Addr
func (c *IPConn) SetReadBuffer(bytes int) error
func (c *IPConn) SetWriteBuffer(bytes int) error
func (c *IPConn) SetDeadline(t time.Time) error
func (c *IPConn) SetReadDeadline(t time.Time) error
func (c *IPConn) SetWriteDeadline(t time.Time) error
func (c *IPConn) Read(b []byte) (int, error)
func (c *IPConn) ReadFrom(b []byte) (int, Addr, error)
func (c *IPConn) ReadFromIP(b []byte) (int, *IPAddr, error)
func (c *IPConn) ReadMsgIP(b, oob []byte) (n, oobn, flags int, addr *IPAddr, err error)
func (c *IPConn) Write(b []byte) (int, error)
func (c *IPConn) WriteTo(b []byte, addr Addr) (int, error)
func (c *IPConn) WriteToIP(b []byte, addr *IPAddr) (int, error)
func (c *IPConn) WriteMsgIP(b, oob []byte, addr *IPAddr) (n, oobn int, err error)
func (c *IPConn) Close() error
func (c *IPConn) File() (f *os.File, err error)
type TCPConn
func DialTCP(net string, laddr, raddr *TCPAddr) (*TCPConn, error)
func (c *TCPConn) LocalAddr() Addr
func (c *TCPConn) RemoteAddr() Addr
func (c *TCPConn) SetReadBuffer(bytes int) error
func (c *TCPConn) SetWriteBuffer(bytes int) error
func (c *TCPConn) SetDeadline(t time.Time) error
func (c *TCPConn) SetReadDeadline(t time.Time) error
func (c *TCPConn) SetWriteDeadline(t time.Time) error
func (c *TCPConn) SetKeepAlive(keepalive bool) error
func (c *TCPConn) SetKeepAlivePeriod(d time.Duration) error
func (c *TCPConn) SetLinger(sec int) error
func (c *TCPConn) SetNoDelay(noDelay bool) error
func (c *TCPConn) Read(b []byte) (int, error)
func (c *TCPConn) ReadFrom(r io.Reader) (int64, error)
func (c *TCPConn) Write(b []byte) (int, error)
func (c *TCPConn) Close() error
func (c *TCPConn) CloseRead() error
func (c *TCPConn) CloseWrite() error
func (c *TCPConn) File() (f *os.File, err error)
type UDPConn
func DialUDP(net string, laddr, raddr *UDPAddr) (*UDPConn, error)
func ListenMulticastUDP(net string, ifi *Interface, gaddr *UDPAddr) (*UDPConn, error)
func ListenUDP(net string, laddr *UDPAddr) (*UDPConn, error)
func (c *UDPConn) LocalAddr() Addr
func (c *UDPConn) RemoteAddr() Addr
func (c *UDPConn) SetReadBuffer(bytes int) error
func (c *UDPConn) SetWriteBuffer(bytes int) error
func (c *UDPConn) SetDeadline(t time.Time) error
func (c *UDPConn) SetReadDeadline(t time.Time) error
func (c *UDPConn) SetWriteDeadline(t time.Time) error
func (c *UDPConn) Read(b []byte) (int, error)
func (c *UDPConn) ReadFrom(b []byte) (int, Addr, error)
func (c *UDPConn) ReadFromUDP(b []byte) (n int, addr *UDPAddr, err error)
func (c *UDPConn) ReadMsgUDP(b, oob []byte) (n, oobn, flags int, addr *UDPAddr, err error)
func (c *UDPConn) Write(b []byte) (int, error)
func (c *UDPConn) WriteTo(b []byte, addr Addr) (int, error)
func (c *UDPConn) WriteToUDP(b []byte, addr *UDPAddr) (int, error)
func (c *UDPConn) WriteMsgUDP(b, oob []byte, addr *UDPAddr) (n, oobn int, err error)
func (c *UDPConn) Close() error
func (c *UDPConn) File() (f *os.File, err error)
type UnixConn
func DialUnix(net string, laddr, raddr *UnixAddr) (*UnixConn, error)
func ListenUnixgram(net string, laddr *UnixAddr) (*UnixConn, error)
func (c *UnixConn) LocalAddr() Addr
func (c *UnixConn) RemoteAddr() Addr
func (c *UnixConn) SetReadBuffer(bytes int) error
func (c *UnixConn) SetWriteBuffer(bytes int) error
func (c *UnixConn) SetDeadline(t time.Time) error
func (c *UnixConn) SetReadDeadline(t time.Time) error
func (c *UnixConn) SetWriteDeadline(t time.Time) error
func (c *UnixConn) Read(b []byte) (int, error)
func (c *UnixConn) ReadFrom(b []byte) (int, Addr, error)
func (c *UnixConn) ReadFromUnix(b []byte) (n int, addr *UnixAddr, err error)
func (c *UnixConn) ReadMsgUnix(b, oob []byte) (n, oobn, flags int, addr *UnixAddr, err error)
func (c *UnixConn) Write(b []byte) (int, error)
func (c *UnixConn) WriteTo(b []byte, addr Addr) (n int, err error)
func (c *UnixConn) WriteToUnix(b []byte, addr *UnixAddr) (n int, err error)
func (c *UnixConn) WriteMsgUnix(b, oob []byte, addr *UnixAddr) (n, oobn int, err error)
func (c *UnixConn) Close() error
func (c *UnixConn) CloseRead() error
func (c *UnixConn) CloseWrite() error
func (c *UnixConn) File() (f *os.File, err error)
type TCPListener
func ListenTCP(net string, laddr *TCPAddr) (*TCPListener, error)
func (l *TCPListener) Addr() Addr
func (l *TCPListener) SetDeadline(t time.Time) error
func (l *TCPListener) Accept() (Conn, error)
func (l *TCPListener) AcceptTCP() (*TCPConn, error)
func (l *TCPListener) Close() error
func (l *TCPListener) File() (f *os.File, err error)
type UnixListener
func ListenUnix(net string, laddr *UnixAddr) (*UnixListener, error)
func (l *UnixListener) Addr() Addr
func (l *UnixListener) SetDeadline(t time.Time) (err error)
func (l *UnixListener) Accept() (c Conn, err error)
func (l *UnixListener) AcceptUnix() (*UnixConn, error)
func (l *UnixListener) Close() error
func (l *UnixListener) File() (f *os.File, err error)
func FileConn(f *os.File) (c Conn, err error)
func FilePacketConn(f *os.File) (c PacketConn, err error)
func FileListener(f *os.File) (l Listener, err error)
type MX
type NS
type SRV
func LookupPort(network, service string) (port int, err error)
func LookupCNAME(name string) (cname string, err error)
func LookupHost(host string) (addrs []string, err error)
func LookupIP(host string) (addrs []IP, err error)
func LookupAddr(addr string) (name []string, err error)
func LookupMX(name string) (mx []*MX, err error)
func LookupNS(name string) (ns []*NS, err error)
func LookupSRV(service, proto, name string) (cname string, addrs []*SRV, err error)
func LookupTXT(name string) (txt []string, err error)
```

## type conn

### type [Conn](https://github.com/golang/go/blob/master/src/net/net.go?name=release#62)

```
type Conn interface {
    // Read从连接中读取数据
    // Read方法可能会在超过某个固定时间限制后超时返回错误，该错误的Timeout()方法返回真
    Read(b []byte) (n int, err error)
    // Write从连接中写入数据
    // Write方法可能会在超过某个固定时间限制后超时返回错误，该错误的Timeout()方法返回真
    Write(b []byte) (n int, err error)
    // Close方法关闭该连接
    // 并会导致任何阻塞中的Read或Write方法不再阻塞并返回错误
    Close() error
    // 返回本地网络地址
    LocalAddr() Addr
    // 返回远端网络地址
    RemoteAddr() Addr
    // 设定该连接的读写deadline，等价于同时调用SetReadDeadline和SetWriteDeadline
    // deadline是一个绝对时间，超过该时间后I/O操作就会直接因超时失败返回而不会阻塞
    // deadline对之后的所有I/O操作都起效，而不仅仅是下一次的读或写操作
    // 参数t为零值表示不设置期限
    SetDeadline(t time.Time) error
    // 设定该连接的读操作deadline，参数t为零值表示不设置期限
    SetReadDeadline(t time.Time) error
    // 设定该连接的写操作deadline，参数t为零值表示不设置期限
    // 即使写入超时，返回值n也可能>0，说明成功写入了部分数据
    SetWriteDeadline(t time.Time) error
}
```

## Dail函数

#### func [Dial](https://github.com/golang/go/blob/master/src/net/dial.go?name=release#142)

```
func Dial(network, address string) (Conn, error)
```

在网络network上连接地址address，并返回一个Conn接口。可用的网络类型有：

"tcp"、"tcp4"、"tcp6"、"udp"、"udp4"、"udp6"、"ip"、"ip4"、"ip6"、"unix"、"unixgram"、"unixpacket"

对TCP和UDP网络，地址格式是host:port或[host]:port，参见函数JoinHostPort和SplitHostPort。

```
Dial("tcp", "12.34.56.78:80")
Dial("tcp", "google.com:http")
Dial("tcp", "[2001:db8::1]:http")
Dial("tcp", "[fe80::1%lo0]:80")
```

对IP网络，network必须是"ip"、"ip4"、"ip6"后跟冒号和协议号或者协议名，地址必须是IP地址字面值。

```
Dial("ip4:1", "127.0.0.1")
Dial("ip6:ospf", "::1")
```

对Unix网络，地址必须是文件系统路径。

#### func [DialTimeout](https://github.com/golang/go/blob/master/src/net/dial.go?name=release#149)

```
func DialTimeout(network, address string, timeout time.Duration) (Conn, error)
```

DialTimeout类似Dial但采用了超时。timeout参数如果必要可包含名称解析。

#### func [Pipe](https://github.com/golang/go/blob/master/src/net/pipe.go?name=release#18)

```
func Pipe() (Conn, Conn)
```

Pipe创建一个内存中的同步、全双工网络连接。连接的两端都实现了Conn接口。一端的读取对应另一端的写入，直接将数据在两端之间作拷贝；没有内部缓冲。

## Listener 函数

### type [Listener](https://github.com/golang/go/blob/master/src/net/net.go?name=release#266)

```
type Listener interface {
    // Addr返回该接口的网络地址
    Addr() Addr
    // Accept等待并返回下一个连接到该接口的连接
    Accept() (c Conn, err error)
    // Close关闭该接口，并使任何阻塞的Accept操作都会不再阻塞并返回错误。
    Close() error
}
```

Listener是一个用于面向流的网络协议的公用的网络监听器接口。多个线程可能会同时调用一个Listener的方法。

#### func [Listen](https://github.com/golang/go/blob/master/src/net/dial.go?name=release#261)

```
func Listen(net, laddr string) (Listener, error)
```

返回在一个本地网络地址laddr上监听的Listener。网络类型参数net必须是面向流的网络：

"tcp"、"tcp4"、"tcp6"、"unix"或"unixpacket"。参见Dial函数获取laddr的语法。