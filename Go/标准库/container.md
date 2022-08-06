# container

[参考](https://books.studygolang.com/The-Golang-Standard-Library-by-Example/chapter03/03.3.html)

该包实现了三个复杂的数据结构：堆，链表，环。 这个包就意味着你使用这三个数据结构的时候不需要再费心从头开始写算法了。

## heap

heap包提供了对任意类型（实现了heap.Interface接口）的堆操作。（最小）堆是具有“每个节点都是以其为根的子树中最小值”属性的树。

**树的最小元素为其根元素，索引0的位置**。

**heap是常用的实现优先队列的方法**。要创建一个优先队列，实现一个具有使用（负的）优先级作为比较的依据的Less方法的Heap接口，如此一来可用Push添加项目而用Pop取出队列最高优先级的项目。

### 介绍

[type Interface](https://studygolang.com/static/pkgdoc/pkg/container_heap.htm#Interface)

[func Init(h Interface)](https://studygolang.com/static/pkgdoc/pkg/container_heap.htm#Init)

[func Push(h Interface, x interface{})](https://studygolang.com/static/pkgdoc/pkg/container_heap.htm#Push)

[func Pop(h Interface) interface{}](https://studygolang.com/static/pkgdoc/pkg/container_heap.htm#Pop)

[func Remove(h Interface, i int) interface{}](https://studygolang.com/static/pkgdoc/pkg/container_heap.htm#Remove)

[func Fix(h Interface, i int)](https://studygolang.com/static/pkgdoc/pkg/container_heap.htm#Fix)

#### type [Interface](https://github.com/golang/go/blob/master/src/container/heap/heap.go?name=release#30)

```
type Interface interface {
    sort.Interface
    Push(x interface{}) // 向末尾添加元素
    Pop() interface{}   // 从末尾删除元素
}
```

**任何实现了本接口的类型都可以用于构建最小堆**。最小堆可以通过heap.Init建立，数据是递增顺序或者空的话也是最小堆。最小堆的约束条件是：

```
!h.Less(j, i) for 0 <= i < h.Len() and 2*i+1 <= j <= 2*i+2 and j < h.Len()
```

注意接口的Push和Pop方法是供heap包调用的，请使用heap.Push和heap.Pop来向一个堆添加或者删除元素。

#### func [Init](https://github.com/golang/go/blob/master/src/container/heap/heap.go?name=release#41)

```
func Init(h Interface)
```

一个堆在使用任何堆操作之前应先初始化。Init函数对于堆的约束性是幂等的（多次执行无意义），并可能在任何时候堆的约束性被破坏时被调用。本函数复杂度为O(n)，其中n等于h.Len()。

#### func [Push](https://github.com/golang/go/blob/master/src/container/heap/heap.go?name=release#52)

```
func Push(h Interface, x interface{})
```

向堆h中插入元素x，并保持堆的约束性。复杂度O(log(n))，其中n等于h.Len()。

#### func [Pop](https://github.com/golang/go/blob/master/src/container/heap/heap.go?name=release#61)

```
func Pop(h Interface) interface{}
```

删除并返回堆h中的最小元素（不影响约束性）。复杂度O(log(n))，其中n等于h.Len()。等价于Remove(h, 0)。

#### func [Remove](https://github.com/golang/go/blob/master/src/container/heap/heap.go?name=release#71)

```
func Remove(h Interface, i int) interface{}
```

删除堆中的第i个元素，并保持堆的约束性。复杂度O(log(n))，其中n等于h.Len()。

#### func [Fix](https://github.com/golang/go/blob/master/src/container/heap/heap.go?name=release#85)

```
func Fix(h Interface, i int)
```

**在修改第i个元素后，调用本函数修复堆，比删除第i个元素后插入新元素更有效率**。

复杂度O(log(n))，其中n等于h.Len()。

### 具体使用

可以看出，这个堆结构继承自 sort.Interface, 回顾下 sort.Interface，它需要实现三个方法

- Len() int
- Less(i, j int) bool
- Swap(i, j int)

加上堆接口定义的两个方法

- Push(x interface{})
- Pop() interface{}

就是说你定义了一个堆，就要实现五个方法，直接拿 package doc 中的 example 做例子：

```golang
type IntHeap []int

func (h IntHeap) Len() int           { return len(h) }
func (h IntHeap) Less(i, j int) bool { return h[i] < h[j] }
func (h IntHeap) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }

func (h *IntHeap) Push(x interface{}) {
    *h = append(*h, x.(int))
}

func (h *IntHeap) Pop() interface{} {
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[0 : n-1]
    return x
}
```

那么 IntHeap 就实现了这个堆结构，我们就可以使用堆的方法来对它进行操作：

```golang
h := &IntHeap{2, 1, 5}
heap.Init(h)
heap.Push(h, 3)
heap.Pop(h)
```

具体说下内部实现，是使用最小堆，索引排序从根节点开始，然后左子树，右子树的顺序方式。索引布局如下：

> ```
>                   0
>             1            2
>          3    4       5      6
>         7 8  9 10   11   
> ```
>
> 假设 (heap[1]== 小明 ) 它的左子树 (heap[3]== 小黑 ) 和右子树 (heap[4]== 大黄 ) 且 小明 > 小黑 > 大黄 ;

堆内部实现了 down 和 up 函数 : down 函数用于将索引 i 处存储的值 ( 设 i=1, 即小明 ) 与它的左子树 ( 小黑 ) 和右子树 ( 大黄 ) 相比 , 将三者最小的值大黄与小明的位置交换，交换后小明继续与交换后的子树 (heap[9]和 heap[10]) 相比，重复以上步骤，直到小明位置不变。

```
up 函数用于将索引 i 处的值 ( 设 i=3, 即小黑 ) 与他的父节点 ( 小明 ) 比较，将两者较小的值放到父节点上，本例中即交换小黑和小明的位置，之后小黑继续与其父节点比较，重复以上步骤，直到小黑位置不变。
```

假设 heap[11]== 阿花 当从堆中 Pop 一个元素的时候，先把元素和最后一个节点的值 ( 阿花 ) 交换，然后弹出，然后对阿花调用 down，向下保证最小堆。

当往堆中 Push 一个元素的时候，这个元素插入到最后一个节点，本例中为 heap[12]，即作为 heap[5]的右子树，然后调用 up 函数向上比较。

#### 示例

小红拿到了一个数组 *a*，每次操作小红可以选择数组中的任意一个数减去 x，小红一共能进行 *k* 次。

小红想在 k 次操作之后，数组的最大值尽可能小。请你返回这个最大值。

```
package main
import (
    "container/heap"
)
// 定义大根堆，实现heap中的接口
type MyHeap []int

func (h MyHeap) Len()int{return len(h)}
func (h MyHeap)Less(i, j int)bool{return h[i] > h[j]}
func (h MyHeap)Swap(i, j int){ h[i], h[j] = h[j], h[i] }
func (h *MyHeap)Push(x interface{}){
    *h = append(*h, x.(int))
}
func (h *MyHeap)Pop()interface{}{
    old := *h
    n := len(old)
    x := old[n-1]
    *h = old[0 : n-1]
    return x
}

func minMax( a []int ,  k int ,  x int ) int {
    // write code here
    my := make(MyHeap, len(a))
    copy(my, a)
    //初始化
    heap.Init(&my)
    for i := 0; i < k;i++{
        my[0] -= x
        heap.Fix(&my, 0)
    }
    return my[0]
}
```

## list

list包实现了双向链表。

- [type Element](https://studygolang.com/static/pkgdoc/pkg/container_list.htm#Element)

  - [func (e *Element) Next() *Element](https://studygolang.com/static/pkgdoc/pkg/container_list.htm#Element.Next)

  - [func (e *Element) Prev() *Element](https://studygolang.com/static/pkgdoc/pkg/container_list.htm#Element.Prev)

- [type List](https://studygolang.com/static/pkgdoc/pkg/container_list.htm#List)

  - [func New() *List](https://studygolang.com/static/pkgdoc/pkg/container_list.htm#New)

  - [func (l *List) Init() *List](https://studygolang.com/static/pkgdoc/pkg/container_list.htm#List.Init)

  - [func (l *List) Len() int](https://studygolang.com/static/pkgdoc/pkg/container_list.htm#List.Len)

  - [func (l *List) Front() *Element](https://studygolang.com/static/pkgdoc/pkg/container_list.htm#List.Front)

  - [func (l *List) Back() *Element](https://studygolang.com/static/pkgdoc/pkg/container_list.htm#List.Back)

  - [func (l *List) PushFront(v interface{}) *Element](https://studygolang.com/static/pkgdoc/pkg/container_list.htm#List.PushFront)

  - [func (l *List) PushFrontList(other *List)](https://studygolang.com/static/pkgdoc/pkg/container_list.htm#List.PushFrontList)

  - [func (l *List) PushBack(v interface{}) *Element](https://studygolang.com/static/pkgdoc/pkg/container_list.htm#List.PushBack)

  - [func (l *List) PushBackList(other *List)](https://studygolang.com/static/pkgdoc/pkg/container_list.htm#List.PushBackList)

  - [func (l *List) InsertBefore(v interface{}, mark *Element) *Element](https://studygolang.com/static/pkgdoc/pkg/container_list.htm#List.InsertBefore)

  - [func (l *List) InsertAfter(v interface{}, mark *Element) *Element](https://studygolang.com/static/pkgdoc/pkg/container_list.htm#List.InsertAfter)

  - [func (l *List) MoveToFront(e *Element)](https://studygolang.com/static/pkgdoc/pkg/container_list.htm#List.MoveToFront)

  - [func (l *List) MoveToBack(e *Element)](https://studygolang.com/static/pkgdoc/pkg/container_list.htm#List.MoveToBack)

  - [func (l *List) MoveBefore(e, mark *Element)](https://studygolang.com/static/pkgdoc/pkg/container_list.htm#List.MoveBefore)

  - [func (l *List) MoveAfter(e, mark *Element)](https://studygolang.com/static/pkgdoc/pkg/container_list.htm#List.MoveAfter)

  - [func (l *List) Remove(e *Element) interface{}](https://studygolang.com/static/pkgdoc/pkg/container_list.htm#List.Remove)

### type [Element](https://github.com/golang/go/blob/master/src/container/list/list.go?name=release#15)

```
type Element struct {
    next, prev *Element  // 上一个元素和下一个元素
    list *List  // 元素所在链表
    Value interface{}  // 元素
}
```

Element类型代表是双向链表的一个元素。

#### func (*Element) [Next](https://github.com/golang/go/blob/master/src/container/list/list.go?name=release#31)

```
func (e *Element) Next() *Element
```

Next返回链表的后一个元素或者nil。

#### func (*Element) [Prev](https://github.com/golang/go/blob/master/src/container/list/list.go?name=release#39)

```
func (e *Element) Prev() *Element
```

Prev返回链表的前一个元素或者nil。

### type [List](https://github.com/golang/go/blob/master/src/container/list/list.go?name=release#48)

```
type List struct {
    root Element  // 链表的根元素
    len  int      // 链表的长度
}
```

List代表一个双向链表。List零值为一个空的、可用的链表。

#### func [New](https://github.com/golang/go/blob/master/src/container/list/list.go?name=release#62) 创建列表

```
func New() *List
```

New创建一个链表。

#### func (*List) [Init](https://github.com/golang/go/blob/master/src/container/list/list.go?name=release#54) 清空列表

```
func (l *List) Init() *List
```

Init清空链表。

#### func (*List) [Len](https://github.com/golang/go/blob/master/src/container/list/list.go?name=release#66) 列表长度

```
func (l *List) Len() int
```

Len返回链表中元素的个数，复杂度O(1)。

#### func (*List) [Front](https://github.com/golang/go/blob/master/src/container/list/list.go?name=release#69) 返回列表头接点

```
func (l *List) Front() *Element
```

Front返回链表第一个元素或nil。

#### func (*List) [Back](https://github.com/golang/go/blob/master/src/container/list/list.go?name=release#77) 返回列表尾节点

```
func (l *List) Back() *Element
```

Back返回链表最后一个元素或nil。

#### func (*List) [PushFront](https://github.com/golang/go/blob/master/src/container/list/list.go?name=release#131) 将一个元素插入到第一个位置

```
func (l *List) PushFront(v interface{}) *Element
```

PushBack将一个值为v的新元素插入链表的第一个位置，返回生成的新元素。

#### func (*List) [PushFrontList](https://github.com/golang/go/blob/master/src/container/list/list.go?name=release#211) 创建拷贝列表，并将新列表最后一个元素连接到原列表的第一个位置

```
func (l *List) PushFrontList(other *List)
```

PushFrontList创建链表other的拷贝，并将拷贝的最后一个位置连接到链表l的第一个位置。即 other -> l

#### func (*List) [PushBack](https://github.com/golang/go/blob/master/src/container/list/list.go?name=release#137) 将一个元素插入到最后一个位置

```
func (l *List) PushBack(v interface{}) *Element
```

PushBack将一个值为v的新元素插入链表的最后一个位置，返回生成的新元素。

#### func (*List) [PushBackList](https://github.com/golang/go/blob/master/src/container/list/list.go?name=release#202) 创建拷贝列表，并将新列表的第一个元素连接到原列表的最后一个元素

```
func (l *List) PushBackList(other *List)
```

PushBack创建链表other的拷贝，并将链表l的最后一个位置连接到拷贝的第一个位置。

#### func (*List) [InsertBefore](https://github.com/golang/go/blob/master/src/container/list/list.go?name=release#144) 将一个值为v的新元素插入到mark前面

```
func (l *List) InsertBefore(v interface{}, mark *Element) *Element
```

InsertBefore将一个值为v的新元素插入到mark前面，并返回生成的新元素。如果mark不是l的元素，l不会被修改。

#### func (*List) [InsertAfter](https://github.com/golang/go/blob/master/src/container/list/list.go?name=release#154) 将一个值为v的新元素插入到mark后面

```
func (l *List) InsertAfter(v interface{}, mark *Element) *Element
```

InsertAfter将一个值为v的新元素插入到mark后面，并返回新生成的元素。如果mark不是l的元素，l不会被修改。

#### func (*List) [MoveToFront](https://github.com/golang/go/blob/master/src/container/list/list.go?name=release#164) 将元素e移动到链表的第一个位置

```
func (l *List) MoveToFront(e *Element)
```

MoveToFront将元素e移动到链表的第一个位置，如果e不是l的元素，l不会被修改。

#### func (*List) [MoveToBack ](https://github.com/golang/go/blob/master/src/container/list/list.go?name=release#174) 将元素e移动到链表的最后一个位置

```
func (l *List) MoveToBack(e *Element)
```

MoveToBack将元素e移动到链表的最后一个位置，如果e不是l的元素，l不会被修改。

#### func (*List) [MoveBefore](https://github.com/golang/go/blob/master/src/container/list/list.go?name=release#184) 将元素e移动到mark的前面

```
func (l *List) MoveBefore(e, mark *Element)
```

MoveBefore将元素e移动到mark的前面。如果e或mark不是l的元素，或者e==mark，l不会被修改。

#### func (*List) [MoveAfter](https://github.com/golang/go/blob/master/src/container/list/list.go?name=release#193) 将元素e移动到mark的后面

```
func (l *List) MoveAfter(e, mark *Element)
```

MoveAfter将元素e移动到mark的后面。如果e或mark不是l的元素，或者e==mark，l不会被修改。

#### func (*List) [Remove](https://github.com/golang/go/blob/master/src/container/list/list.go?name=release#121) 删除链表中的元素e，并返回e.Value

```
func (l *List) Remove(e *Element) interface{}
```

Remove删除链表中的元素e，并返回e.Value。

### 具体使用

基本使用是先创建 list，然后往 list 中插入值，list 就内部创建一个 Element，并内部设置好 Element 的 next,prev 等。具体可以看下例子：

```
// This example demonstrates an integer heap built using the heap interface.
package main

import (
    "container/list"
    "fmt"
)

func main() {
    list := list.New()
    list.PushBack(1)
    list.PushBack(2)

    fmt.Printf("len: %v\n", list.Len())
    fmt.Printf("first: %#v\n", list.Front())
    fmt.Printf("second: %#v\n", list.Front().Next())
}

output:
len: 2
first: &list.Element{next:(*list.Element)(0x2081be1b0), prev:(*list.Element)(0x2081be150), list:(*list.List)(0x2081be150), Value:1}
second: &list.Element{next:(*list.Element)(0x2081be150), prev:(*list.Element)(0x2081be180), list:(*list.List)(0x2081be150), Value:2}
```

## ring

ring实现了环形链表的操作。

### 介绍

[type Ring](https://studygolang.com/static/pkgdoc/pkg/container_ring.htm#Ring)

- [func New(n int) *Ring](https://studygolang.com/static/pkgdoc/pkg/container_ring.htm#New)
- [func (r *Ring) Len() int](https://studygolang.com/static/pkgdoc/pkg/container_ring.htm#Ring.Len)
- [func (r *Ring) Next() *Ring](https://studygolang.com/static/pkgdoc/pkg/container_ring.htm#Ring.Next)
- [func (r *Ring) Prev() *Ring](https://studygolang.com/static/pkgdoc/pkg/container_ring.htm#Ring.Prev)
- [func (r *Ring) Move(n int) *Ring](https://studygolang.com/static/pkgdoc/pkg/container_ring.htm#Ring.Move)
- [func (r *Ring) Link(s *Ring) *Ring](https://studygolang.com/static/pkgdoc/pkg/container_ring.htm#Ring.Link)
- [func (r *Ring) Unlink(n int) *Ring](https://studygolang.com/static/pkgdoc/pkg/container_ring.htm#Ring.Unlink)
- [func (r *Ring) Do(f func(interface{}))](https://studygolang.com/static/pkgdoc/pkg/container_ring.htm#Ring.Do)

#### type [Ring](https://github.com/golang/go/blob/master/src/container/ring/ring.go?name=release#14)

```
type Ring struct {
    Value interface{} // 供调用者使用，本包不会操作该字段
    // 包含隐藏或非导出字段
}
```

Ring类型代表环形链表的一个元素，同时也代表链表本身。环形链表没有头尾；指向环形链表任一元素的指针都可以作为整个环形链表看待。Ring零值是具有一个（Value字段为nil的）元素的链表。

#### func [New](https://github.com/golang/go/blob/master/src/container/ring/ring.go?name=release#62)

```
func New(n int) *Ring
```

New创建一个具有n个元素的环形链表。

#### func (*Ring) [Len](https://github.com/golang/go/blob/master/src/container/ring/ring.go?name=release#121)

```
func (r *Ring) Len() int
```

Len返回环形链表中的元素个数，复杂度O(n)。

#### func (*Ring) [Next](https://github.com/golang/go/blob/master/src/container/ring/ring.go?name=release#26)

```
func (r *Ring) Next() *Ring
```

返回后一个元素，r不能为空。

#### func (*Ring) [Prev](https://github.com/golang/go/blob/master/src/container/ring/ring.go?name=release#34)

```
func (r *Ring) Prev() *Ring
```

返回前一个元素，r不能为空。

#### func (*Ring) [Move](https://github.com/golang/go/blob/master/src/container/ring/ring.go?name=release#44)

```
func (r *Ring) Move(n int) *Ring
```

返回移动n个位置（n>=0向前移动，n<0向后移动）后的元素，r不能为空。

#### func (*Ring) [Link](https://github.com/golang/go/blob/master/src/container/ring/ring.go?name=release#93)

```
func (r *Ring) Link(s *Ring) *Ring
```

Link连接r和s，并返回r原本的后继元素r.Next()。r不能为空。

如果r和s指向同一个环形链表，则会删除掉r和s之间的元素，删掉的元素构成一个子链表，返回指向该子链表的指针（r的原后继元素）；如果没有删除元素，则仍然返回r的原后继元素，而不是nil。如果r和s指向不同的链表，将创建一个单独的链表，将s指向的链表插入r后面，返回s原最后一个元素后面的元素（即r的原后继元素）。

#### func (*Ring) [Unlink](https://github.com/golang/go/blob/master/src/container/ring/ring.go?name=release#111)

```
func (r *Ring) Unlink(n int) *Ring
```

删除链表中n % r.Len()个元素，从r.Next()开始删除。如果n % r.Len() == 0，不修改r。返回删除的元素构成的链表，r不能为空。

#### func (*Ring) [Do](https://github.com/golang/go/blob/master/src/container/ring/ring.go?name=release#134)

```
func (r *Ring) Do(f func(interface{}))
```

对链表的每一个元素都执行f（正向顺序），注意如果f改变了*r，Do的行为是未定义的。