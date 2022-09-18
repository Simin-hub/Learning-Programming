# db/sql

##  从 database/sql 讲起

Go官方提供了`database/sql`包来给用户进行和数据库打交道的工作，实际上`database/sql`库就只是提供了一套操作数据库的接口和规范，例如抽象好的SQL预处理（prepare），连接池管理，数据绑定，事务，错误处理等等。官方并没有提供具体某种数据库实现的协议支持。

和具体的数据库，例如MySQL打交道，还需要再引入MySQL的驱动，像下面这样：

```
import "database/sql"
import _ "github.com/go-sql-driver/mysql"
db, err := sql.Open("mysql", "user:password@/dbname")
```

这一句import，实际上是调用了`mysql`包的`init`函数，做的事情也很简单：

```
func init() {
    sql.Register("mysql", &MySQLDriver{})
}
```

在`sql`包的 全局`map`里把`mysql`这个名字的`driver`注册上。实际上`Driver`在`sql`包中是一个接口：

```
type Driver interface {
    Open(name string) (Conn, error)
}
```

调用`sql.Open()`返回的`db`对象实际上就是这里的`Conn`。

```
type Conn interface {
    Prepare(query string) (Stmt, error)
    Close() error
    Begin() (Tx, error)
}
```

也是一个接口。实际上如果你仔细地查看`database/sql/driver/driver.go`的代码会发现，这个文件里所有的成员全都是接口，对这些类型进行操作，实际上还是会调用具体的`driver`里的方法。

从用户的角度来讲，在使用`database/sql`包的过程中，你能够使用的也就是这些接口里提供的函数。来看一个使用`database/sql`和`go-sql-driver/mysql`的完整的例子：

```
package main
import (
    "database/sql"
    _ "github.com/go-sql-driver/mysql"
)
func main() {
    // db 是一个 sql.DB 类型的对象
    // 该对象线程安全，且内部已包含了一个连接池
    // 连接池的选项可以在 sql.DB 的方法中设置，这里为了简单省略了
    db, err := sql.Open("mysql",
        "user:password@tcp(127.0.0.1:3306)/hello")
    if err != nil {
        log.Fatal(err)
    }
    defer db.Close()
    var (
        id int
        name string
    )
    rows, err := db.Query("select id, name from users where id = ?", 1)
    if err != nil {
        log.Fatal(err)
    }
    defer rows.Close()
    // 必须要把 rows 里的内容读完，或者显式调用 Close() 方法，
    // 否则在 defer 的 rows.Close() 执行之前，连接永远不会释放
    for rows.Next() {
        err := rows.Scan(&id, &name)
        if err != nil {
            log.Fatal(err)
        }
        log.Println(id, name)
    }
    err = rows.Err()
    if err != nil {
        log.Fatal(err)
    }
}
```



## 提高生产效率的ORM和SQL Builder

### ORM

在Web开发领域常常提到的ORM是什么？我们先看看万能的维基百科：

```
对象关系映射（英语：Object Relational Mapping，简称ORM，或O/RM，或O/R mapping），
是一种程序设计技术，用于实现面向对象编程语言里不同类型系统的数据之间的转换。
从效果上说，它其实是创建了一个可在编程语言里使用的“虚拟对象数据库”。
```

最为常见的ORM实际上做的是从db到程序的类或结构体这样的映射。所以你手边的程序可能是从MySQL的表映射你的程序内的类。我们可以先来看看其它的程序语言里的ORM写起来是怎么样的感觉：

```
>>> from blog.models import Blog
>>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
>>> b.save()
```

完全没有数据库的痕迹，**没错ORM的目的就是屏蔽掉DB层，实际上很多语言的ORM只要把你的类或结构体定义好，再用特定的语法将结构体之间的一对一或者一对多关系表达出来**。那么任务就完成了。然后你就可以对这些映射好了数据库表的对象进行各种操作，例如save，create，retrieve，delete。至于ORM在背地里做了什么阴险的勾当，你是不一定清楚的。使用ORM的时候，我们往往比较容易有一种忘记了数据库的直观感受。举个例子，我们有个需求：向用户展示最新的商品列表，我们再假设，商品和商家是1:1的关联关系，我们就很容易写出像下面这样的代码：

```
# 伪代码
shopList := []
for product in productList {
    shopList = append(shopList, product.GetShop)
}
```

### SQL Builder

相比ORM来说，SQL Builder在SQL和项目可维护性之间取得了比较好的平衡。首先sql builer不像ORM那样屏蔽了过多的细节，其次从开发的角度来讲，SQL Builder简单进行封装后也可以非常高效地完成开发，举个例子：

```
where := map[string]interface{} {
    "order_id > ?" : 0,
    "customer_id != ?" : 0,
}
limit := []int{0,100}
orderBy := []string{"id asc", "create_time desc"}
orders := orderModel.GetList(where, limit, orderBy)
```

 写SQL Builder的相关代码，或者读懂都不费劲。把这些代码脑内转换为sql也不会太费劲。所以通过代码就可以对这个查询是否命中数据库索引，是否走了覆盖索引，是否能够用上联合索引进行分析了。

说白了SQL Builder是sql在代码里的一种特殊方言，如果你们没有DBA但研发有自己分析和优化sql的能力，或者你们公司的DBA对于学习这样一些sql的方言没有异议。那么使用SQL Builder是一个比较好的选择，不会导致什么问题。

另外在一些本来也不需要DBA介入的场景内，使用SQL Builder也是可以的，例如你要做一套运维系统，且将MySQL当作了系统中的一个组件，系统的QPS不高，查询不复杂等等。

一旦你做的是高并发的OLTP在线系统，且想在人员充足分工明确的前提下最大程度控制系统的风险，使用SQL Builder就不合适了。

### 脆弱的数据库

无论是ORM还是SQL Builder都有一个致命的缺点，就是没有办法进行系统上线的事前sql审核。虽然很多ORM和SQL Builder也提供了运行期打印sql的功能，但只在查询的时候才能进行输出。而SQL Builder和ORM本身提供的功能太过灵活。使得你不可能通过测试枚举出所有可能在线上执行的sql。