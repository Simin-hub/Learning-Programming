# Go 面试题（难重点）

## 注意点

这部分主要是收录一些易错概念，主要是作者在牛客上的错题收录。

### 指针

go语言的自动内存管理机制使得**只要还有一个指针引用一个变量**，那这个变量就会在内存中得以保留，因此在Go语言函数内部返回指向本地变量的指针是安全的。

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/go%E6%8C%87%E9%92%881.png)

代码中的指针p不是野指针（即空指针），因为返回的栈内存在函数结束时不会被释放。

**即函数返回指针时发生内存逃逸**。[逃逸场景](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E8%BF%9B%E9%98%B6/%E9%80%83%E9%80%B8%E5%88%86%E6%9E%90.md#%E9%80%83%E9%80%B8%E5%9C%BA%E6%99%AF)

考察的内容是逃逸分析。

### 数组与切片

**数组类型的值（以下简称数组）的长度是固定的数组的长度在声明它的时候就必须给定，并且在之后不会再改变。可以说，数组的长度是其类型的一部分（数组的容量永远等于其长度，都是不可变的）**

[4]int 和[3]int是不同的类型

[地址](https://golang.design/go-questions/slice/vs-array/)

**slice 的底层数据是数组，slice 是对数组的封装**，它描述一个数组的片段。两者都可以通过下标来访问单个元素。

数组就是一片连续的内存， slice 实际上是一个结构体，包含三个字段：长度、容量、底层数组。

```
// runtime/slice.go
type slice struct {
	array unsafe.Pointer // 元素指针
	len   int // 长度 
	cap   int // 容量
}
```

slice 的数据结构如下：

<img src="https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/sliceVSarray.png" alt="切片数据结构" style="zoom:50%;" />

注意，底层数组是可以被多个 slice 同时指向的，因此对一个 slice 的元素进行操作是有可能影响到其他 slice 的。

```
s0 := []int{0, 0}
s1 := append(s0, 2) //追加一个元素， s1 == []int{0, 0, 2}；
s2 := append(s1, 3, 5, 7) //追加多个元素， s2 == []int{0, 0, 2, 3, 5, 7}；
s3 := append(s2, s0...) //追加一个 slice， s3 == []int{0, 0, 2, 3, 5, 7, 0, 0}。注意这三个点！
```

截取部分切片时，容量是从截取开始到切片末尾

```
slice := []int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
s1 := slice[2:5]
s2 := s1[2:6:7]
```

<img src="https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/splitSlice.png" alt="slice origin" style="zoom:50%;" />

`s1` 从 `slice` 索引2（闭区间）到索引5（开区间，元素真正取到索引4），长度为3，，为8。 `s2` 从 `s1` 的索引2（闭区间）到索引6（开区间，元素真正取到索引5），容量到索引7（开区间，真正到索引6），为5。

**截取切片时，都是指向同一个底层数组。**

**当添加切片超过其拥有的容量时，则会复制一个新的切片并增加原先一倍的容量。**

### 函数失败回收资源

下面函数能回收资源

![img](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/%E5%9B%9E%E6%94%B6%E8%B5%84%E6%BA%90.png)

函数返回失败有3种情况：

一是第一次分配资源失败，直接返回，这时并没有分配成功的资源；

二是第一次分配资源成功，第二次分配资源失败，函数返回，第二次和第三次的资源都未成功分配，此时err不为nil，第一次分配成功的资源通过defer释放；

三是第一二次资源分配成功，第三次资源分配失败，函数返回，第一二次分配成功的资源通过defer释放；

如果第三次资源分配也成功了，则函数不会返回失败。

**若程序在定义 defer 之前退出，则程序结束后 defer 并不会执行， 若在定义 defer 之后程序退出，则会在程序退出后执行 defer**。

若使用`os.Exit(1)` 退出也不会执行 defer

```
func main() {
	defer func() {
		fmt.Println("exit")
	}()
	os.Exit(1)
}

```

[defer](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E8%BF%9B%E9%98%B6/%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.md#defer)

### panic()函数

当内置的panic()函数调用时，外围函数或方法的执行会立即终止。然后，任何延迟执行(defer)的函数或方法都会被调用，就像其外围函数正常返回一样。最后，调用返回到该外围函数的调用者，就像该外围调用函数或方法调用了panic()一样，因此该过程一直在调用栈中重复发生：函数停止执行，调用延迟执行函数等。当到达main()函数时不再有可以返回的调用者，因此这个过程会终止，并将包含传入原始panic()函数中的值的调用栈信息输出到os.Stderr。

### CGO

CGO是C语言和Go语言之间的桥梁，原则上无法直接支持C++的类。CGO不支持C++语法的根本原因是C++至今为止还没有一个二进制接口规范(ABI)。CGO只支持C语言中值类型的数据类型，所以我们是无法直接使用C++的引用参数等特性的。

### 常量定义

**go语言常量要是编译时就能确定的数据，C选项中errors.New("xxx") 要等到运行时才能确定，所以它不满足**

在 Go 语言中，你可以省略类型说明符 [type]，因为编译器可以根据变量的值来推断其类型；

存储在常量中的数据类型只可以是布尔型、数字型（整数型、浮点型和复数）和字符串型。

### 通道的类别

双向：var value chan int

单向只读：var value <-chan int

单向只写：var value chan<-int

单向一般用于参数传递和返回值。 单向通道都是由双向通道转换而来，不能自己声明单向通道，没有意义。 单向通道作为函数或者方法的参数时，表示该函数或者方法作用域内，只能执行接收或者发送操作。 单向通道作为函数或者方法返回值时，表示调用者得到该通道后只能做单向操作。

### switch

switch后面的**声明语句**和**表达式语句**都是**可选**的

### Go语言类型转换语法

类型转换语法：Type(expression)

类型断言语法为：Interface.(Type)

对于类型断言，首先 Interface必须是接口类型

### 异常触发

空指针解析、下标越界、除数为0、调用panic函数都是会报异常的。

### 关键字

var和const ：变量和常量的声明

var varName type  或者 varName : = value

package and import: 导入

func： 用于定义函数和方法

return ：用于从函数返回

defer someCode ：在函数退出之前执行

go : 用于并行

select 用于选择不同类型的通讯

interface 用于定义接口

struct 用于定义抽象数据类型

break、case、continue、for、fallthrough、else、if、switch、goto、default 流程控制

chan用于channel通讯

type用于声明自定义类型

map用于声明map类型数据

range用于读取slice、map、channel数据

### main函数

**Main函数和init函数都没有参数和返回值的定义**

### select

golang 的 select 就是监听 IO 操作，当 IO 操作发生时，触发相应的动作。 

在执行select语句的时候，运行时系统会自上而下地判断每个case中的发送或接收操作是否可以被立即执行(立即执行：意思是当前Goroutine不会因此操作而被阻塞)

select的用法与switch非常类似，由select开始一个新的选择块，每个选择条件由case语句来描述。与switch语句可以选择任何可使用相等比较的条件相比，select有比较多的限制，**其中最大的一条限制就是每个case语句里必须是一个IO操作**，确切的说，应该是一个面向channel的IO操作。

select后不用带判断条件，是case后带判断条件。

### go 的 [] rune 和 [] byte 区别

`byte` 表示一个字节，`rune` 表示四个字节，

[地址](https://learnku.com/articles/23411/the-difference-between-rune-and-byte-of-go)

### 序列化与反序列化

 在反序列化中，传入的都是地址

map、切片和结构体都是引用类型，他们的值都是地址

序列化通常将类型结构传入标准库或第三方包，**类型结构中没有大写的变量未导出，对第三方包不可见，无法进行任何操作，依旧是默认的零值**。

### 字符串的表达方式

字符串只有两种直接表达的形式，一种是双引号，一种是反引号

### 接口

拥有相同方法列表，那么两个接口实质上同一个接口

A是B的子集，意味着A的方法B中都有，那么A是B的基类，所以A=B是可行的

接口是否能够调用成功是需要运行的时候才能知道

**接口赋值是否可行在编译阶段就可以知道**

### **import后面的最后一个元素应该是路径，就是目录，并非包名**。

go语言的惯例只是一个特例，即恰好目录名与包名一致。

### 可变参数

在声明可变参数函数时，需要在参数列表的最后一个参数类型之前加上省略符号“...”，这表示该函数会接收任意数量的该类型参数。

传入切片需要在后面加上省略符号“...”

```
func sum(vals ...int) int {
    total := 0
    for _, val := range vals {
        total += val
    }
    return total
}

fmt.Println(sum())           // "0"
fmt.Println(sum(3))          // "3"
fmt.Println(sum(1, 2, 3, 4)) // "10"
fmt.Println(sum([]int{1, 2, 3, 4}...))
```

### nil赋值

nil只能赋值给指针、channel、func、interface、map或slice类型的变量。如果将nil赋值给其他变量的时候将会引发panic。

<img src="https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/242025553_1564399633384_3538CAEF7C819BA4AD8BC262C5D1EE6D" alt="img" style="zoom:50%;" />

### cap函数

cap的作用：

- array：返回数组的元素个数 
- slice：返回slice的最大容量
-  channel：返回channel的buffer容量

**cap函数不支持map**,map中使用len表示大小

### 错误和异常的区别

错误指的是可能出现问题的地方出现了问题，比如打开一个文件时失败，这种情况在人们的意料之中 ；而异常指的是不应该出现问题的地方出现了问题，比如引用了空指针，这种情况在人们的意料之外。可见，**错误是业务过程的一部分，而异常不是** 。

**Golang中引入error接口类型作为错误处理的标准模式**，如果函数要返回错误，则返回值类型列表中肯定包含error。error处理过程类似于C语言中的错误码，可逐层返回，直到被处理。

**Golang中引入两个内置函数panic和recover来触发和终止异常处理流程**，同时引入关键字defer来延迟执行defer后面的函数。

一直等到包含defer语句的函数执行完毕时，延迟函数（defer后的函数）才会被执行，而不管包含defer语句的函数是通过return的正常结束，还是由于panic导致的异常结束。你可以在一个函数中执行多条defer语句，它们的执行顺序与声明顺序相反。
当程序运行时，如果遇到引用空指针、下标越界或显式调用panic函数等情况，则先触发panic函数的执行，然后调用延迟函数。调用者继续传递panic，因此该过程一直在调用栈中重复发生：函数停止执行，调用延迟执行函数等。如果一路在延迟函数中没有recover函数的调用，则会到达该携程的起点，该携程结束，然后终止其他所有携程，包括主携程（类似于C语言中的主线程，该携程ID为1）。

错误和异常从Golang机制上讲，就是error和panic的区别。很多其他语言也一样，比如C++/Java，没有error但有errno，没有panic但有throw。

Golang错误和异常是可以互相转换的：

1. 错误转异常，比如程序逻辑上尝试请求某个URL，最多尝试三次，尝试三次的过程中请求失败是错误，尝试完第三次还不成功的话，失败就被提升为异常了。
2. 异常转错误，比如panic触发的异常被recover恢复后，将返回值中error类型的变量进行赋值，以便上层函数继续走错误处理流程。



## 面试题

面经问题

**新手**

- [Golang开发新手常犯的50个错误](https://blog.csdn.net/gezhonglei2007/article/details/52237582)

**数据类型（map、slice、数组、set）**

[基本数据类型](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E8%BF%9B%E9%98%B6/%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B.md#%E5%9F%BA%E7%A1%80%E6%95%B0%E6%8D%AE%E7%B1%BB%E5%9E%8B)

- [连nil切片和空切片一不一样都不清楚？那BAT面试官只好让你回去等通知了。](https://mp.weixin.qq.com/s/sW4PD1MiaunURNDIU4BbQQ)
- [golang 中的四种类型转换总结](https://learnku.com/articles/42797)
- [golang面试题：字符串转成byte数组，会发生内存拷贝吗？](https://mp.weixin.qq.com/s/qmlPuGVISx8NYp2b9LrqnA)
- [golang面试题：翻转含有中文、数字、英文字母的字符串](https://mp.weixin.qq.com/s/ssinnUM22PHPWRug8EzAkg)
- [golang面试题：拷贝大切片一定比小切片代价大吗？](https://mp.weixin.qq.com/s/8Dp2eCYzDdBbxAG5-jNevQ)
- [map不初始化使用会怎么样](https://blog.csdn.net/qq_39920531/article/details/88103496)
- [map不初始化长度和初始化长度的区别](https://www.kancloud.cn/kancloud/the-way-to-go/72493)
- [map承载多大，大了怎么办](https://yangxikun.com/golang/2019/10/07/golang-map.html)
- [map的iterator是否安全？能不能一边delete一边遍历？](https://www.bookstack.cn/read/qcrao-Go-Questions/map-%E5%8F%AF%E4%BB%A5%E8%BE%B9%E9%81%8D%E5%8E%86%E8%BE%B9%E5%88%A0%E9%99%A4%E5%90%97.md)
- [字符串不能改，那转成数组能改吗，怎么改](http://c.biancheng.net/view/39.html)
- [怎么判断一个数组是否已经排序](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter03/03.1.html)
- [普通map如何不用锁解决协程安全问题](https://zhuanlan.zhihu.com/p/356739568)
- [array和slice的区别](https://segmentfault.com/a/1190000013148775)
- [golang面试题：json包变量不加tag会怎么样？](https://mp.weixin.qq.com/s/bZlKV_BWSqc-qCa4DrsCbg)
- [golang面试题：reflect（反射包）如何获取字段tag？为什么json包不能导出私有变量的tag？](https://mp.weixin.qq.com/s/P7TEx2mInwEktXTEE6JDWQ)
- [零切片、空切片、nil切片是什么](https://juejin.cn/post/6844903712654098446)
- [slice深拷贝和浅拷贝](https://learnku.com/articles/59163)
- [nil 不同于 null（或是 NULL、nullptr）](https://blog.singee.me/2020/09/24/e8cb67835ea44243b136e3cdf8d5ea84/)
- [map触发扩容的时机，满足什么条件时扩容？](https://www.bookstack.cn/read/qcrao-Go-Questions/map-map%20%E7%9A%84%E6%89%A9%E5%AE%B9%E8%BF%87%E7%A8%8B%E6%98%AF%E6%80%8E%E6%A0%B7%E7%9A%84.md)
- [map扩容策略是什么](https://www.bookstack.cn/read/qcrao-Go-Questions/map-map%20%E7%9A%84%E6%89%A9%E5%AE%B9%E8%BF%87%E7%A8%8B%E6%98%AF%E6%80%8E%E6%A0%B7%E7%9A%84.md)
- [自定义类型切片转字节切片和字节切片转回自动以类型切片](https://blog.csdn.net/weixin_42506905/article/details/81359448)
- [make和new什么区别](https://juejin.cn/post/6859145664316571661)
- [slice ，map，chanel创建的时候的几个参数什么含义](https://blog.csdn.net/TCatTime/article/details/111567560)
- [slice，len，cap，共享，扩容](https://cloud.tencent.com/developer/article/1822529)
- [go struct能不能比较？](https://juejin.cn/post/6881912621616857102)
- [使用range 迭代 map 是有序的吗?  map如何顺序读取？](https://www.jianshu.com/p/65d338bb6c82)
- [go中怎么实现set](https://studygolang.com/articles/11179)
- [使用值为 nil 的 sice、map 会发生什么？](https://segmentfault.com/a/1190000038175302)
- [Golang 有没有 this 指针？](https://blog.csdn.net/ma2595162349/article/details/108632865)
- [Golang 语言中局部变量和全局变量的缺省值是什么](https://studygolang.com/articles/15282)
- [Golang 中的引用类型包含哪些?](https://studygolang.com/articles/27252)
- [slice 的扩容机制是什么？](https://juejin.cn/post/6844903812331732999)
- [Golang 中指针运算有哪些?](https://blog.csdn.net/fly910905/article/details/105989267)
- [string 类型的值可以修改吗？](https://www.cnblogs.com/brady-wang/p/15821039.html)
- [array 类型的值作为函数参数是引用传递还是值传递？](https://segmentfault.com/a/1190000037763005)
- [nil](https://segmentfault.com/a/1190000039894167)

**流程控制(for、select、defer、switch)**

- [昨天那个在for循环里append元素的同事，今天还在么？](https://studygolang.com/articles/30844)
- [golang面试官：for select时，如果通道已经关闭会怎么样？如果只有一个case呢？](https://cloud.tencent.com/developer/article/1796708)
- [go defer（for defer）](https://draveness.me/golang/docs/part2-foundation/ch05-keyword/golang-defer/)
- [select可以用于什么？](https://blog.csdn.net/zhaominpro/article/details/77570290)
- [context包的用途？](https://zhuanlan.zhihu.com/p/76555349)
- [switch 中如何强制执行下一个 case 代码块?](https://www.cnblogs.com/gwyy/p/13670090.html)
- [如何从 panic 中恢复?](https://learnku.com/docs/the-way-to-go/133-recovery-from-panic-recover/3676)

**解析**

- [解析 JSON 数据时，默认将数值当做哪种类型](https://zhuanlan.zhihu.com/p/388124005)

**包管理**

- [学go mod就够了！](https://studygolang.com/articles/27293)

**优化**

- [golang面试题：怎么避免内存逃逸？](https://mp.weixin.qq.com/s/VzRTHz1JaDUvNRVB_yJa1A)
- [golang面试题：简单聊聊内存逃逸？](https://mp.weixin.qq.com/s/wJmztRMB1ZAAIItyMcS0tw)
- [给大家丢脸了，用了三年golang，我还是没答对这道内存泄漏题](https://mp.weixin.qq.com/s/-agtdhlW7Yj7S88a0z7KHg)
- [内存碎片化问题](https://segmentfault.com/a/1190000020338427)
- [chan相关的goroutine泄露的问题](https://segmentfault.com/a/1190000040161853)
- [string相关的goroutine泄露的问题](cnblogs.com/ricklz/p/11262069.html)
- [你一定会遇到的内存回收策略导致的疑似内存泄漏的问题](https://colobu.com/2019/08/28/go-memory-leak-i-dont-think-so/)
- [sync.Pool的适用场景](https://geektutu.com/post/hpg-sync-pool.html)

**goroutine**

- [线程、进程、协程、goroutines ](https://zhuanlan.zhihu.com/p/27245377)
- [golang面试题：对已经关闭的的chan进行读写，会怎么样？为什么？](https://mp.weixin.qq.com/s/izbZ3JRqX6jI6Wn7bV6xNQ)
- [golang面试题：对未初始化的的chan进行读写，会怎么样？为什么？](https://juejin.cn/post/6844904196181852173)
- [sync.map 的优缺点和使用场景](https://studygolang.com/articles/22128)
- [主协程如何等其余协程完再操作](https://blog.csdn.net/weixin_42678507/article/details/102786680)
- [有缓存的channel和没有缓存的channel区别是什么？](https://zhuanlan.zhihu.com/p/355487940)
- [协程通信方式有哪些？](https://zhuanlan.zhihu.com/p/36907022)
- [channel底层实现](https://i6448038.github.io/2019/04/11/go-channel/)
- [读写锁底层是怎么实现的？](https://blog.csdn.net/sunxianghuang/article/details/104780010)
- [golang的CSP思想](https://zhuanlan.zhihu.com/p/313763247)
- [channel 是怎么保证线程安全？](http://www.zhoubotong.site/post/25.html)
- [协程和线程的差别](https://segmentfault.com/a/1190000040373756)
- [开多个线程和开多个协程会有什么区别](https://www.kancloud.cn/todo/go_learn/1222804)
- [协程可以自己主动让出 CPU 吗？](https://studygolang.com/articles/26795)
- [GMP](https://learnku.com/articles/41728)
- [GMP模型](https://zboya.github.io/post/go_scheduler/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)
- [动图图解，GMP里为什么要有P](https://mp.weixin.qq.com/s/SEE2TUeZQZ7W1BKkmnelAA)

**反射**

- [golang 面试题：reflect（反射包）如何获取字段 tag？为什么 json 包不能导出私有变量的 tag？](https://mp.weixin.qq.com/s/WK9StkC3Jfy-o1dUqlo7Dg)

**接口（ interface）**

- [开源库里会有一些类似下面这种奇怪的用法：`var _ io.Writer = (*myWriter)(nil)`，是为什么？](https://www.bookstack.cn/read/qcrao-Go-Questions/interface-%E7%BC%96%E8%AF%91%E5%99%A8%E8%87%AA%E5%8A%A8%E6%A3%80%E6%B5%8B%E7%B1%BB%E5%9E%8B%E6%98%AF%E5%90%A6%E5%AE%9E%E7%8E%B0%E6%8E%A5%E5%8F%A3.md)
- [两个interface{} 能不能比较](https://www.jianshu.com/p/a982807819fa)
- [断言时会发生拷贝吗](https://blog.csdn.net/qq_39397165/article/details/115500314)
- [接口是怎么实现的？](https://juejin.cn/post/6844904082453299207)

**unsafe**

- [golang面试题：能说说uintptr和unsafe.Pointer的区别吗？](https://mp.weixin.qq.com/s/PSkz0zj-vqKzmIKa_b-xAA)

**GC**

[GC](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E8%BF%9B%E9%98%B6/GC.md)

- [重点](https://www.cnblogs.com/luozhiyun/p/14564903.html)

- [垃圾回收的过程是怎么样的？](https://zhuanlan.zhihu.com/p/297177002)
- [什么是写屏障、混合写屏障，如何实现？](https://www.bookstack.cn/read/qcrao-Go-Questions/spilt.9.GC-GC.md)
- [为什么gc会让程序变慢](https://www.bookstack.cn/read/qcrao-Go-Questions/spilt.14.GC-GC.md)
- [gc的stw是怎么回事](https://www.bookstack.cn/read/qcrao-Go-Questions/spilt.5.GC-GC.md)
- [为什么小对象多了会造成 gc 压力?](https://www.modb.pro/db/148423)
- [两次 GC 周期重叠会引发什么问题，GC 触发机制是什么样的？](https://www.jianshu.com/p/bfc3c65c05d1)

**问题排查**

- [trace](https://mp.weixin.qq.com/s?__biz=MzA4ODg0NDkzOA==&mid=2247487157&idx=1&sn=cbf1c87efe98433e07a2e58ee6e9899e&source=41#wechat_redirect)
- [pprof](https://mp.weixin.qq.com/s/d0olIiZgZNyZsO-OZDiEoA)
- [什么是 goroutine 泄漏?](https://segmentfault.com/a/1190000040161853)
- [当go服务部署到线上了，发现有内存泄露，该怎么处理](https://blog.csdn.net/shudaqi2010/article/details/103362028?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-0.pc_relevant_default&spm=1001.2101.3001.4242.1&utm_relevant_index=3)

**其他**

- 利用golang特性，设计一个QPS为500的服务器
- [必须要手动对齐内存的情况](https://geektutu.com/post/hpg-struct-alignment.html)
- [go栈扩容和栈缩容，连续栈的缺点](https://segmentfault.com/a/1190000019570427)
- [golang怎么做代码优化](https://tangheng1995.github.io/golang/2020/06/03/Golang-optimize.html)
- [golang隐藏技能:怎么访问私有成员](https://www.jianshu.com/p/7b3638b47845)
- [一个协程能保证绑定在一个内核线程上吗？](https://studygolang.com/articles/26795)
- [闭包怎么实现的,闭包的主要应用场景](https://zhuanlan.zhihu.com/p/56750616)
- [Goroutinue 什么时候会被挂起？](https://developer.51cto.com/article/681462.html)
- [Data Race 问题怎么检测？怎么解决?](https://learnku.com/articles/45279)
- [Golang 触发异常的场景有哪些?](http://xueyuan.coder55.com/read/go-senior-learn/go-question-14.2)
- [net/http包中client如何实现长连接？](net/http包中client如何实现长连接？)
- [net/http怎么做连接池和长链接？](net/http怎么做连接池和长链接？)
