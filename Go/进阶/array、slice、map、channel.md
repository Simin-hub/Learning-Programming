# array 、slice、map、channel

[array,slice](https://blog.thinkeridea.com/201901/go/shen_ru_pou_xi_slice_he_array.html)

`array` 和 `slice` 看似相似，却有着极大的不同，但他们之间还有着千次万缕的联系 `slice` 是引用类型、是 `array` 的引用，相当于动态数组，
这些都是 `slice` 的特性，但是 `slice` 底层如何表现，内存中是如何分配的，特别是在程序中大量使用 `slice` 的情况下，怎样可以高效使用 `slice`？

## array

在计算机科学中，数组数据结构（英语：array data structure），简称数组（英语：Array），是由相同类型的元素（element）的集合所组成的数据结构，分配一块连续的内存来存储，利用元素的索引（index）可以计算出该元素对应的存储地址。
数组设计之初是在形式上依赖内存分配而成的，所以必须在使用前预先请求空间。这使得数组有以下特性:

1. **请求空间以后大小固定，不能再改变**（数据溢出问题）；
2. 在内存中有**空间连续性的表现**，中间不会存在其他程序需要调用的数据，为此数组的专用内存空间；
3. 在旧式编程语言中（如有中阶语言之称的C），程序不会对数组的操作做下界判断，也就有潜在的越界操作的风险（比如会把数据写在运行中程序需要调用的核心部分的内存上）。

总结一下数组有一下特性：

- 分配在连续的内存地址上
- 元素类型一致，元素存储宽度一致
- 空间大小固定，不能修改
- 可以通过索引计算出元素对应存储的位置（只需要知道数组内存的起始位置和数据元素宽度即可）
- 会出现数据溢出的问题（下标越界）

数组的长度也是其类型的一种，例如：

```go
a := [4]int{}
b := [3]int{}
```

`a`和`b`并不相等。

不允许动态给数组长度

```
package main

import (
	"fmt"
	"unsafe"
)

func main() {
	n:= 10
	var arr = [n]int{}
	fmt.Println(arr)
}
```

这是错误的

## slice

`slice` 是在 `array` 的基础上实现的

`slice` 的类型规范是[]T，`slice` T元素的类型。与数组类型不同，`slice` 类型没有指定的长度。

**`slice` 申明的几种方法：**

> `s := []int{1, 2, 3}` 简短的赋值语句
>
> `var s []int` `var` 申明
>
> `make([]int, 3, 8)` 或 `make([]int, 3)` `make` 内置方法创建
>
> `s := ss[:5] `从切片或者数组创建

**`slice` 有两个内置函数来获取其属性：**

> `len` 获取 `slice` 的长度
>
> `cap` 获取 `slice` 的容量

### slice底层结构

```
//runtime/slice.go
type slice struct {
    array unsafe.Pointer
    len   int
    cap   int
}
```

- `slice` 是的起始位置是引用数组元素位置的指针。
- `slice` 的长度是引用数组元素起始位置到结束位置的长度。
- `slice` 的容量是引用数组元素起始位置到数组末尾的长度。

![slice](https://blog.thinkeridea.com/assets/image/20190111/slice_1.jpg)

### 初始化

Slice在初始化时需要初始化指针，长度和容量，**容量未指定时将自动初始化为长度的大小。可以通过直接获取数组的引用、获取数组/slice的切片构建或是make函数初始化数组**。下面给出一些slice初始化的方式示例。

```go
s := []int{1,2,3}	//通过数组的引用初始化，值为[1,2,3],长度和容量为3

arr := [5]int{1,2,3,4,5}
s := arr[0:3]	//通过数组的切片初始化，值为[1,2,3]，长度为3，容量为5

s := arr[1:3]	//通过数组的切片初始化，值为[2,3]，长度为2，容量为5

s := make([]int, 3)	//通过make函数初始化，值为[0,0,0]，长度和容量为3

s := make([]int, 3, 5)	//通过make函数初始化，值为[0,0,0]，长度为3，容量为5
```

**当通过数组初始化切片时**

切片内的指针指向开始的的地址，例如第6行中，指针指向2所在的地址

**当通过现有切片初始化新切片时**

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/a0eca91a66cd494d966e8882b1817243%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1304%3A0%3A0%3A0.awebp)



这样的初始化方式可能会导致内存被**过度占用**，如只需要使用一个极大的数组中的几个元素，但是由于需要指向整个数组，所以整个数组在GC时都无法被释放，一直占用内存空间。故使用切片操作进行初始化时，最好使用`append`函数将切片出来的数据复制到一个新的slice中，从而避免内存占用陷阱。

### append

[参考](https://blog.thinkeridea.com/201901/go/shen_ru_pou_xi_slice_he_array.html)

[参考2](https://www.zhihu.com/question/27161493?sort=created)

`slice` 是如何增长的，用 `unsafe` 分析一下看看：

```
package main

import (
	"fmt"
	"unsafe"
)

func main() {
	s := make([]int, 9, 10)

	// 引用底层的数组地址
	fmt.Printf("%x\n", *(*uintptr)(unsafe.Pointer(&s)))

	s = append(s, 1)

	// 引用底层的数组地址
	fmt.Printf("%x\n", *(*uintptr)(unsafe.Pointer(&s)))

	s = append(s, 1)

	// 引用底层的数组地址
	fmt.Printf("%x\n", *(*uintptr)(unsafe.Pointer(&s)))
}
```

以上代码的输出([Go Playground](https://play.golang.org/p/3c4ek4-0ft5)):

> c000082e90
>
> 9 10
>
> c000082e90
>
> 10 10
>
> c00009a000
>
> 11 20

从结果上看前两次地址是一样的，初始化一个长度为9，容量为10的 `slice`，当第一次 `append` 的时候容量是足够的，所以底层引用数组地址未发生变化，此时 `slice` 的长度和容量都为10，之后再次 `append` 的时候发现底层数组的地址不一样了，因为 `slice` 的长度超过了容量，但是新的 `slice` 容量并不是11而是20，这要说 `slice` 的机制了，因为数组长度不可变，想扩容 `slice`就必须分配一个更大的数组，并把之前的数据拷贝到新数组，如果一次只增加1个长度，那就会那发生大量的内存分配和数据拷贝，这个成本是很大的，所以 `slice` 是有一个增长策略的。

`Go` 标准库 `runtime/slice.go` 当中有详细的 `slice` 增长策略的逻辑：

```
func growslice(et *_type, old slice, cap int) slice {
    .....
    
    // 计算新的容量，核心算法用来决定slice容量增长
	newcap := old.cap
	doublecap := newcap + newcap
	if cap > doublecap {
		newcap = cap
	} else {
		if old.len < 1024 {
			newcap = doublecap
		} else {
			for 0 < newcap && newcap < cap {
				newcap += newcap / 4
			}
			if newcap <= 0 {
				newcap = cap
			}
		}
	}

    // 根据et.size调整新的容量
	var overflow bool
	var lenmem, newlenmem, capmem uintptr
	switch {
	case et.size == 1:
		lenmem = uintptr(old.len)
		newlenmem = uintptr(cap)
		capmem = roundupsize(uintptr(newcap))
		overflow = uintptr(newcap) > maxAlloc
		newcap = int(capmem)
	case et.size == sys.PtrSize:
		lenmem = uintptr(old.len) * sys.PtrSize
		newlenmem = uintptr(cap) * sys.PtrSize
		capmem = roundupsize(uintptr(newcap) * sys.PtrSize)
		overflow = uintptr(newcap) > maxAlloc/sys.PtrSize
		newcap = int(capmem / sys.PtrSize)
	case isPowerOfTwo(et.size):
		var shift uintptr
		if sys.PtrSize == 8 {
			// Mask shift for better code generation.
			shift = uintptr(sys.Ctz64(uint64(et.size))) & 63
		} else {
			shift = uintptr(sys.Ctz32(uint32(et.size))) & 31
		}
		lenmem = uintptr(old.len) << shift
		newlenmem = uintptr(cap) << shift
		capmem = roundupsize(uintptr(newcap) << shift)
		overflow = uintptr(newcap) > (maxAlloc >> shift)
		newcap = int(capmem >> shift)
	default:
		lenmem = uintptr(old.len) * et.size
		newlenmem = uintptr(cap) * et.size
		capmem = roundupsize(uintptr(newcap) * et.size)
		overflow = uintptr(newcap) > maxSliceCap(et.size)
		newcap = int(capmem / et.size)
	}

    ......
	
	var p unsafe.Pointer
	if et.kind&kindNoPointers != 0 {
		p = mallocgc(capmem, nil, false)  // 分配新的内存
		memmove(p, old.array, lenmem) // 拷贝数据
		memclrNoHeapPointers(add(p, newlenmem), capmem-newlenmem)
	} else {
		p = mallocgc(capmem, et, true) // 分配新的内存
		if !writeBarrier.enabled {
			memmove(p, old.array, lenmem)
		} else {
			for i := uintptr(0); i < lenmem; i += et.size {
				typedmemmove(et, add(p, i), add(old.array, i)) // 拷贝数据
			}
		}
	}

	return slice{p, old.len, newcap} // 新slice引用新的数组，长度为旧数组的长度，容量为新数组的容量
}
```

基本呢就三个步骤，**计算新的容量、分配新的数组、拷贝数据到新数组**，社区很多人分享 `slice` 的增长方法，实际都不是很精确，因为大家只分析了计算 `newcap` 的那一段，也就是上面注释的第一部分，下面的 `switch` 根据 `et.size` 来调整 `newcap` 一段被直接忽略，社区的结论是：”**如果 `selic` 的容量小于1024个元素，那么扩容的时候 `slice` 的 `cap` 就翻番，乘以2；一旦元素个数超过1024个元素，增长因子就变成1.25，即每次增加原来容量的四分之一**” 大多数情况也确实如此，但是根据 `newcap` 的计算规则，如果新的容量超过旧的容量2倍时会直接按新的容量分配，真的是这样吗?

```
package main

import (
	"fmt"
)

func main() {
	s := make([]int, 10, 10)
	fmt.Println(len(s), cap(s))
	s2 := make([]int, 40)

	s = append(s, s2...)
	fmt.Println(len(s), cap(s))

}
```

以上代码的输出([Go Playground](https://play.golang.org/p/x8kN4V5R7YW)):

> 10 10
>
> 50 52

这个结果有点出人意料， 如果是2倍增长应该是 `10 * 2 * 2 * 2` 结果应该是80， 如果说新的容量高于旧容量的两倍但结果也不是50，实际上 `newcap` 的结果就是50，那段逻辑很好理解，但是`switch` 根据 `et.size` 来调整 `newcap` 后就是52了，这段逻辑走到了 `case et.size == sys.PtrSize` 这段，详细的以后做源码分析再说。

**总结 **

- **当 `slice` 的长度超过其容量，会分配新的数组，并把旧数组上的值拷贝到新的数组**
- 逐个元素添加到 `slice` 并超过其容量， 如果 `selic` 的容量小于1024个元素，那么扩容的时候 `slice` 的 `cap` 就翻番，乘以2；一旦元素个数超过1024个元素，增长因子就变成1.25，即每次增加原来容量的四分之一。
- 批量添加元素，当新的容量高于旧容量的两倍，就会分配比新容量稍大一些，并不会按上面第二条的规则扩容。
- 当 `slice` 发生扩容，引用新数组后，`slice` 操作不会再影响旧的数组，而是新的数组（社区经常讨论的传递 `slice` 容量超出后，修改数据不会作用到旧的数据上），所以往往设计函数如果会对长度调整都会返回新的 `slice`，例如 `append` 方法。

**当slice为多维时**

进行append时，是进行浅拷贝

[参考2](https://leetcode-cn.com/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/solution/by-jiu-tian-5-7h2i/)

[参考](https://www.1024sou.com/article/532888.html)

### copy

[参考](http://c.biancheng.net/view/29.html)

slice通过copy进行深拷贝，使用`=`是进行反复浅拷贝

### slice 的三种状态

[参考](https://juejin.cn/post/6844903712654098446)

`slice` 有三种状态：零切片、空切片、nil切片。

零切片就是nil切片

```
var s1 []int // nil 切片
var s2 = []int{} // 空切片
var s3 = make([]int, 0) // 空切片
var s4 = *new([]int) // nil 切片

var a1 = *(*[3]int)(unsafe.Pointer(&s1))
var a2 = *(*[3]int)(unsafe.Pointer(&s2))
var a3 = *(*[3]int)(unsafe.Pointer(&s3))
var a4 = *(*[3]int)(unsafe.Pointer(&s4))
fmt.Println(a1)
fmt.Println(a2)
fmt.Println(a3)
fmt.Println(a4)

---------------------
[0 0 0]
[824634199592 0 0]
[824634199592 0 0]
[0 0 0]
```

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/1670b6d164141b26%7Etplv-t2oaga2asx-zoom-in-crop-mark%3A1304%3A0%3A0%3A0.awebp)

空切片指向的 zerobase 内存地址是一个神奇的地址，从 Go 语言的源代码中可以看到它的定义



```
//// runtime/malloc.go

// base address for all 0-byte allocations
var zerobase uintptr

// 分配对象内存
func mallocgc(size uintptr, typ *_type, needzero bool) unsafe.Pointer {
	...
	if size == 0 {
		return unsafe.Pointer(&zerobase)
	}
    ...
}

//// runtime/slice.go
// 创建切片
func makeslice(et *_type, len, cap int) slice {
  ...
     p := mallocgc(et.size*uintptr(cap), et, true)
	 return slice{p, len, cap}
}
复制代码
```

最后一个问题是：「 nil 切片」和 「空切片」在使用上有什么区别么？

答案是完全没有任何区别！No！不对，还有一个小小的区别！请看下面的代码

```
package main

import "fmt"

func main() {
	var s1 []int
	var s2 = []int{}

	fmt.Println(s1 == nil)
	fmt.Println(s2 == nil)

	fmt.Printf("%#v\n", s1)
	fmt.Printf("%#v\n", s2)
}

-------
true
false
[]int(nil)
[]int{}
复制代码
```

所以为了避免写代码的时候把脑袋搞昏的最好办法是不要创建「 空切片」，统一使用「 nil 切片」，同时要避免将切片和 nil 进行比较来执行某些逻辑。这是官方的标准建议。

> The former declares a nil slice value, while the latter is non-nil but zero-length. They are functionally equivalent—their len and cap are both zero—but the nil slice is the preferred style.

「空切片」和「 nil 切片」有时候会隐藏在结构体中，这时候它们的区别就被太多的人忽略了，下面我们看个例子

```
type Something struct {
	values []int
}

var s1 = Something{}
var s2 = Something{[]int{}}
fmt.Println(s1.values == nil)
fmt.Println(s2.values == nil)

--------
true
false
复制代码
```

可以发现这**两种创建结构体的结果是不一样的**！

「空切片」和「 nil 切片」还有一个极为不同的地方在于 JSON 序列化

```
type Something struct {
	Values []int
}

var s1 = Something{}
var s2 = Something{[]int{}}
bs1, _ := json.Marshal(s1)
bs2, _ := json.Marshal(s2)
fmt.Println(string(bs1))
fmt.Println(string(bs2))

---------
{"Values":null}
{"Values":[]}
复制代码
```

Ban! Ban! Ban! **它们的 json 序列化结果居然也不一样**！



## map





## channel