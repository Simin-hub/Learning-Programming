# 二维slice进行append

进行append时，是进行浅拷贝

```
package main

import (
	"fmt"
	"testing"
)

func main(t *testing.T) {
	a := make([][]int, 0)
	b := []int{1, 2, 3}
	c := []int{4, 5, 6}
	d := []int{7, 8, 9}
	fmt.Printf("%p\n", &a)
	fmt.Printf("%p\n", &b)
	fmt.Printf("%p\n", &c)
	fmt.Printf("%p\n", &d)

	fmt.Println("----")
	a = append(a, b)
	fmt.Printf("%p\n", &a)
	fmt.Printf("%p\n", &a[0][0])
	fmt.Printf("%p\n", &b[0])
	fmt.Println(a[0])
	b[0] = 2
	fmt.Println(a[0])
	fmt.Println(b)
}
```

```
0xc0000040a8
0xc0000040c0
0xc0000040d8
0xc0000040f0
----
0xc0000040a8
0xc00000c1b0  //说明底层数组相同
0xc00000c1b0
[1 2 3]
[2 2 3]
[2 2 3]
```

在二维slice append之后，若是修改原有的slice，则会发生变化。