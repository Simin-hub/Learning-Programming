# sync/atomic标准库包中提供的原子操作

[参考地址](https://gfw.go101.org/article/concurrent-atomic-operation.html)

[源码分析](https://www.cyub.vip/2021/04/05/Golang%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90%E7%B3%BB%E5%88%97%E4%B9%8Batomic%E5%BA%95%E5%B1%82%E5%AE%9E%E7%8E%B0/)

原子操作是比其它同步技术更基础的操作。**原子操作是无锁的，常常直接通过CPU指令直接实现**。 事实上，其它同步技术的实现常常依赖于原子操作。

注意，本文中的很多例子并非并发程序。它们只是用来演示如何使用`sync/atomic`标准库包中提供的原子操作。

### Go支持的原子操作概述

对于一个整数类型`T`，`sync/atomic`标准库包提供了下列原子操作函数。 其中`T`可以是内置`int32`、`int64`、`uint32`、`uint64`和`uintptr`类型。

```go
func AddT(addr *T, delta T)(new T)
func LoadT(addr *T) (val T)
func StoreT(addr *T, val T)
func SwapT(addr *T, new T) (old T)
func CompareAndSwapT(addr *T, old, new T) (swapped bool)
```

比如，下列五个原子操作函数提供给了内置`int32`类型。

```go
func AddInt32(addr *int32, delta int32)(new int32)
func LoadInt32(addr *int32) (val int32)
func StoreInt32(addr *int32, val int32)
func SwapInt32(addr *int32, new int32) (old int32)
func CompareAndSwapInt32(addr *int32, old new int32) (swapped bool)
```

下列四个原子操作函数提供给了（安全）指针类型。因为Go目前（1.17）并不支持自定义泛型，所以这些函数是通过[非类型安全指针](https://gfw.go101.org/article/unsafe.html)`unsafe.Pointer`来实现的。

```go
func LoadPointer(addr *unsafe.Pointer) (val unsafe.Pointer)
func StorePointer(addr *unsafe.Pointer, val unsafe.Pointer)
func SwapPointer(addr *unsafe.Pointer, new unsafe.Pointer) (old unsafe.Pointer)
func CompareAndSwapPointer(addr *unsafe.Pointer, old new unsafe.Pointer) (swapped bool)
```

因为Go指针不支持算术运算，所以相对于整数类型，指针类型的原子操作少了一个`AddPointer`函数。

`sync/atomic`标准库包也提供了一个`Value`类型。以它为基的指针类型`*Value`拥有两个方法：`Load`和`Store`。 `Value`值用来原子读取和修改任何类型的Go值。

```go
func (v *Value) Load() (x interface{})
func (v *Value) Store(x interface{})
```

### 整数原子操作

下面这个例子展示了如何使用`add`原子操作来并发地递增一个`int32`值。 在此例子中，主协程中创建了1000个新协程。每个新协程将整数`n`的值增加`1`。 原子操作保证这1000个新协程之间不会发生数据竞争。此程序肯定打印出`1000`。

```go
package main

import (
	"fmt"
	"sync"
	"sync/atomic"
)

func main() {
	var n int32
	var wg sync.WaitGroup
	for i := 0; i < 1000; i++ {
		wg.Add(1)
		go func() {
			atomic.AddInt32(&n, 1)
			wg.Done()
		}()
	}
	wg.Wait()

	fmt.Println(atomic.LoadInt32(&n)) // 1000
}
```

如果我们将新协程中的语句`atomic.AddInt32(&n, 1)`替换为`n++`，则最后的输出结果很可能不是`1000`。

`StoreT`和`LoadT`原子操作函数经常被用来需要并发运行的实现setter和getter方法。下面是一个这样的例子：

```go
type Page struct {
	views uint32
}

func (page *Page) SetViews(n uint32) {
	atomic.StoreUint32(&page.views, n)
}

func (page *Page) Views() uint32 {
	return atomic.LoadUint32(&page.views)
}
```

如果`T`是一个有符号整数类型，比如`int32`或`int64`，则`AddT`函数调用的第二个实参可以是一个负数，用来实现原子减法操作。 但是如果`T`是一个无符号整数类型，比如`uint32`、`uint64`或者`uintptr`，则`AddT`函数调用的第二个实参需要为一个非负数，那么如何实现无符号整数类型`T`值的原子减法操作呢？ 毕竟`sync/atomic`标准库包没有提供`SubstractT`函数。 根据欲传递的第二个实参的特点，我们可以把`T`为一个无符号整数类型的情况细分为两类：

1. 第二个实参为类型为`T`的一个变量值`v`。 因为`-v`在Go中是合法的，所以`-v`可以直接被用做`AddT`调用的第二个实参。
2. 第二个实参为一个正整数常量`c`，这时`-c`在Go中是编译不通过的，所以它不能被用做`AddT`调用的第二个实参。 这时我们可以使用`^T(c-1)`（仍为一个正数）做为`AddT`调用的第二个实参。



此`^T(v-1)`小技巧对于无符号类型的变量`v`也是适用的，但是`^T(v-1)`比`T(-v)`的效率要低。

对于这个`^T(c-1)`小技巧，如果`c`是一个类型确定值并且它的类型确实就是`T`，则它的表示形式可以简化为`^(c-1)`。

一个例子：

```go
package main

import (
	"fmt"
	"sync/atomic"
)

func main() {
	var (
		n uint64 = 97
		m uint64 = 1
		k int    = 2
	)
	const (
		a        = 3
		b uint64 = 4
		c uint32 = 5
		d int    = 6
	)

	show := fmt.Println
	atomic.AddUint64(&n, -m)
	show(n) // 96 (97 - 1)
	atomic.AddUint64(&n, -uint64(k))
	show(n) // 94 (95 - 2)
	atomic.AddUint64(&n, ^uint64(a - 1))
	show(n) // 91 (94 - 3)
	atomic.AddUint64(&n, ^(b - 1))
	show(n) // 87 (91 - 4)
	atomic.AddUint64(&n, ^uint64(c - 1))
	show(n) // 82 (87 - 5)
	atomic.AddUint64(&n, ^uint64(d - 1))
	show(n) // 76 (82 - 6)
	x := b; atomic.AddUint64(&n, -x)
	show(n) // 72 (76 - 4)
	atomic.AddUint64(&n, ^(m - 1))
	show(n) // 71 (72 - 1)
	atomic.AddUint64(&n, ^uint64(k - 1))
	show(n) // 69 (71 - 2)
}
```



`SwapT`函数调用和`StoreT`函数调用类似，但是返回修改之前的旧值（因此称为置换操作）。

一个`CompareAndSwapT`函数调用传递的旧值和目标值的当前值匹配的情况下才会将目标值改为新值，并返回`true`；否则立即返回`false`。

一个例子：

```go
package main

import (
	"fmt"
	"sync/atomic"
)

func main() {
	var n int64 = 123
	var old = atomic.SwapInt64(&n, 789)
	fmt.Println(n, old) // 789 123
	swapped := atomic.CompareAndSwapInt64(&n, 123, 456)
	fmt.Println(swapped) // false
	fmt.Println(n)       // 789
	swapped = atomic.CompareAndSwapInt64(&n, 789, 456)
	fmt.Println(swapped) // true
	fmt.Println(n)       // 456
}
```



请注意，到目前为止（Go 1.17），一个64位字（int64或uint64值）的原子操作要求此64位字的内存地址必须是8字节对齐的。 请阅读[关于Go值的内存布局](https://gfw.go101.org/article/memory-layout.html)一文获取详情。



### 指针值的原子操作

上面已经提到了`sync/atomic`标准库包为指针值的原子操作提供了四个函数，并且指针值的原子操作是通过非类型安全指针来实现的。

从[非类型安全指针](https://gfw.go101.org/article/unsafe.html)一文，我们得知，在Go中， 任何指针类型的值可以被显式转换为非类型安全指针类型`unsafe.Pointer`，反之亦然。 所以指针类型`*unsafe.Pointer`的值也可以被显式转换为类型`unsafe.Pointer`，反之亦然。

下面这个程序不是一个并发程序。它仅仅展示了如何使用指针原子操作。在这个例子中，类型`T`可以为任何类型。

```go
package main

import (
	"fmt"
	"sync/atomic"
	"unsafe"
)

type T struct {x int}
var pT *T

func main() {
	var unsafePPT = (*unsafe.Pointer)(unsafe.Pointer(&pT))
	var ta, tb = T{1}, T{2}
	// 修改
	atomic.StorePointer(
		unsafePPT, unsafe.Pointer(&ta))
	fmt.Println(pT) // &{1}
	// 读取
	pa1 := (*T)(atomic.LoadPointer(unsafePPT))
	fmt.Println(pa1 == &ta) // true
	// 置换
	pa2 := atomic.SwapPointer(
		unsafePPT, unsafe.Pointer(&tb))
	fmt.Println((*T)(pa2) == &ta) // true
	fmt.Println(pT) // &{2}
	// 比较置换
	b := atomic.CompareAndSwapPointer(
		unsafePPT, pa2, unsafe.Pointer(&tb))
	fmt.Println(b) // false
	b = atomic.CompareAndSwapPointer(
		unsafePPT, unsafe.Pointer(&tb), pa2)
	fmt.Println(b) // true
}
```

是的，目前指针的原子操作使用起来是相当的啰嗦。 事实上，啰嗦还是次要的，更主要的是，因为指针的原子操作需要引入`unsafe`标准库包，所以这些操作函数不在[Go 1兼容性保证](https://golang.google.cn/doc/go1compat)之列。

我个人认为目前支持的这些指针原子操作在今后变为不合法的可能性很小。 即使它们变得不再合法，Go官方工具链中的`go fix`命令应该会将它们转换为今后的新的合法形式。 但是，这只是我的个人观点，它没有任何权威性。

如果你确实担忧这些指针原子操作在未来的合法性，你可以使用下一节将要介绍的原子操作。 但是下一节将要介绍的原子操作对于指针值来说比本节介绍的指针原子操作效率要低得多。



### 任何类型值的原子操作

`sync/atomic`标准库包中提供的`Value`类型可以用来读取和修改任何类型的值。

类型`*Value`有几个方法：`Load`、`Store`、`Swap`和`CompareAndSwap`（其中后两个方法实在Go 1.17中引入的）。 这些方法均以`interface{}`做为参数类型，所以传递给它们的实参可以是任何类型的值。 但是对于一个可寻址的`Value`类型的值`v`，一旦`v.Store`方法（`(&v).Store`的简写形式）被曾经调用一次，则传递给值`v`的后续方法调用的实参的具体类型必须和传递给它的第一次调用的实参的具体类型一致； 否则，将产生一个恐慌。`nil`接口类型实参也将导致`v.Store()`方法调用产生恐慌。

一个例子：

```go
package main

import (
	"fmt"
	"sync/atomic"
)

func main() {
	type T struct {a, b, c int}
	var ta = T{1, 2, 3}
	var v atomic.Value
	v.Store(ta)
	var tb = v.Load().(T)
	fmt.Println(tb)       // {1 2 3}
	fmt.Println(ta == tb) // true

	v.Store("hello") // 将导致一个恐慌
}
```

另一个例子（针对Go 1.17+）：

```go
package main

import (
	"fmt"
	"sync/atomic"
)

func main() {
	type T struct {a, b, c int}
	var x = T{1, 2, 3}
	var y = T{4, 5, 6}
	var z = T{7, 8, 9}
	var v atomic.Value
	v.Store(x)
	fmt.Println(v) // {{1 2 3}}
	old := v.Swap(y)
	fmt.Println(v)       // {{4 5 6}}
	fmt.Println(old.(T)) // {1 2 3}
	swapped := v.CompareAndSwap(x, z)
	fmt.Println(swapped, v) // false {{4 5 6}}
	swapped = v.CompareAndSwap(y, z)
	fmt.Println(swapped, v) // true {{7 8 9}}
}
```

事实上，我们也可以使用上一节介绍的指针原子操作来对任何类型的值进行原子读取和修改，不过需要多一级指针的间接引用。 两种方法有各自的好处和缺点。在实践中需要根据具体需要选择合适的方法。

### 原子操作相关的内存顺序保证

为了便于理解和使用简单，Go值的原子操作被设计的和内存顺序保证无关。 没有任何官方文档规定了原子操作应该保证的内存顺序。 详见[Go中的内存顺序保证](https://gfw.go101.org/article/memory-model.html#atomic)一文对此情况的说明。