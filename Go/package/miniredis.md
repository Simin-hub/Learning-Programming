# miniredis

除了经常用到MySQL外，Redis在日常开发中也会经常用到。接下来的这一小节，我们将一起学习如何在单元测试中mock Redis的相关操作。

[miniredis](https://github.com/alicebob/miniredis)是一个纯go实现的用于单元测试的redis server。它是一个简单易用的、基于内存的redis替代品，它具有真正的TCP接口，你可以把它当成是redis版本的`net/http/httptest`。

当我们为一些包含Redis操作的代码编写单元测试时就可以使用它来mock Redis操作。

### 安装

```bash
go get github.com/alicebob/miniredis/v2
```

### 使用示例

这里以`github.com/go-redis/redis`库为例，编写了一个包含若干Redis操作的`DoSomethingWithRedis`函数。

```bash
// redis_op.go
package miniredis_demo

import (
	"context"
	"github.com/go-redis/redis/v8" // 注意导入版本
	"strings"
	"time"
)

const (
	KeyValidWebsite = "app:valid:website:list"
)

func DoSomethingWithRedis(rdb *redis.Client, key string) bool {
	// 这里可以是对redis操作的一些逻辑
	ctx := context.TODO()
	if !rdb.SIsMember(ctx, KeyValidWebsite, key).Val() {
		return false
	}
	val, err := rdb.Get(ctx, key).Result()
	if err != nil {
		return false
	}
	if !strings.HasPrefix(val, "https://") {
		val = "https://" + val
	}
	// 设置 blog key 五秒过期
	if err := rdb.Set(ctx, "blog", val, 5*time.Second).Err(); err != nil {
		return false
	}
	return true
}
```

下面的代码是我使用`miniredis`库为`DoSomethingWithRedis`函数编写的单元测试代码，其中`miniredis`不仅支持mock常用的Redis操作，还提供了很多实用的帮助函数，例如检查key的值是否与预期相等的`s.CheckGet()`和帮助检查key过期时间的`s.FastForward()`。

```go
// redis_op_test.go

package miniredis_demo

import (
	"github.com/alicebob/miniredis/v2"
	"github.com/go-redis/redis/v8"
	"testing"
	"time"
)

func TestDoSomethingWithRedis(t *testing.T) {
	// mock一个redis server
	s, err := miniredis.Run()
	if err != nil {
		panic(err)
	}
	defer s.Close()

	// 准备数据
	s.Set("q1mi", "liwenzhou.com")
	s.SAdd(KeyValidWebsite, "q1mi")

	// 连接mock的redis server
	rdb := redis.NewClient(&redis.Options{
		Addr: s.Addr(), // mock redis server的地址
	})

	// 调用函数
	ok := DoSomethingWithRedis(rdb, "q1mi")
	if !ok {
		t.Fatal()
	}

	// 可以手动检查redis中的值是否复合预期
	if got, err := s.Get("blog"); err != nil || got != "https://liwenzhou.com" {
		t.Fatalf("'blog' has the wrong value")
	}
	// 也可以使用帮助工具检查
	s.CheckGet(t, "blog", "https://liwenzhou.com")

	// 过期检查
	s.FastForward(5 * time.Second) // 快进5秒
	if s.Exists("blog") {
		t.Fatal("'blog' should not have existed anymore")
	}
}
```

执行执行测试，查看单元测试结果：

```bash
❯ go test -v
=== RUN   TestDoSomethingWithRedis
--- PASS: TestDoSomethingWithRedis (0.00s)
PASS
ok      golang-unit-test-demo/miniredis_demo    0.052s
```

`miniredis`基本上支持绝大多数的Redis命令，大家可以通过查看文档了解更多用法。

当然除了使用`miniredis`搭建本地redis server这种方法外，还可以使用各种打桩工具对具体方法进行打桩。在编写单元测试时具体使用哪种mock方式还是要根据实际情况来决定。