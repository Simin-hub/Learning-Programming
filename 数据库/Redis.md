# Redis

[参考](https://www.bookstack.cn/books/redis-tutorial)、[参考](https://xiaolincoding.com/redis/data_struct/command.html)

## 基本数据类型

[参考](https://xiaolincoding.com/redis/data_struct/command.html)

Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。

随着 Redis 版本的更新，后面又支持了四种数据类型： **BitMap（2.2 版新增）、HyperLogLog（2.8 版新增）、GEO（3.2 版新增）、Stream（5.0 版新增）**。

| redis类型  | 含义     |
| :--------- | :------- |
| String     | 字符串   |
| Hash       | 哈希     |
| List       | 列表     |
| Set        | 集合     |
| Sorted set | 有序集合 |

### String 字符串

string是redis最基本的类型，一个key对应一个value。

string类型是二进制安全的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象

string类型是Redis最基本的数据类型，**一个键最大能存储512MB**。

```
redis 127.0.0.1:6379> SET name "itcast"
OK
redis 127.0.0.1:6379> GET name
"itcast"
127.0.0.1:6379> type name
string
```

删除键值

```
redis 127.0.0.1:6379> del name
```

删除这个 key 及对应的 value

验证键是否存在

```
redis 127.0.0.1:6379> exists name
(integer) 0
```

在上面的例子中，SET 和 GET 是 Redis STRING命令，`name` 和 `itcast` 是存储在 Redis 的键和字符串值。



##### 内部实现

String 类型的**底层的数据结构实现主要是 int 和 SDS（简单动态字符串）**。

SDS 和我们认识的 C 字符串不太一样，之所以没有使用 C 语言的字符串表示，因为 SDS 相比于 C 的原生字符串：

- **SDS 不仅可以保存文本数据，还可以保存二进制数据**。因为 `SDS` 使用 `len` 属性的值而不是空字符来判断字符串是否结束，并且 SDS 的所有 API 都会以处理二进制的方式来处理 SDS 存放在 `buf[]` 数组里的数据。所以 SDS 不光能存放文本数据，而且能保存图片、音频、视频、压缩文件这样的二进制数据。
- **SDS 获取字符串长度的时间复杂度是 O(1)**。因为 C 语言的字符串并不记录自身长度，所以获取长度的复杂度为 O(n)；而 SDS 结构里用 `len` 属性记录了字符串长度，所以复杂度为 `O(1)`。
- **Redis 的 SDS API 是安全的，拼接字符串不会造成缓冲区溢出**。因为 SDS 在拼接字符串之前会检查 SDS 空间是否满足要求，如果空间不够会自动扩容，所以不会导致缓冲区溢出的问题。

字符串对象的内部编码（encoding）有 3 种 ：**int、raw和 embstr**。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/数据类型/string结构.png)

如果一个字符串对象保存的是整数值，并且这个整数值可以用`long`类型来表示，那么字符串对象会将整数值保存在字符串对象结构的`ptr`属性里面（将`void*`转换成 long），并将字符串对象的编码设置为`int`。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/数据类型/int.png)

如果字符串对象保存的是一个字符串，并且这个**字符申的长度小于等于 32 字节**（redis 2.+版本），那么字符串对象将使用一个简单动态字符串（SDS）来保存这个字符串，并将对象的编码设置为`embstr`， **`embstr`编码是专门用于保存短字符串的一种优化编码方式**：

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/embstr.png)

如果字符串对象保存的是一个字符串，并且这个字符串的长度大于 32 字节（redis 2.+版本），那么字符串对象将使用一个简单动态字符串（SDS）来保存这个字符串，并将对象的编码设置为`raw`：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/数据类型/raw.png)

注意，embstr 编码和 raw 编码的边界在 redis 不同版本中是不一样的：

- redis 2.+ 是 32 字节
- redis 3.0-4.0 是 39 字节
- redis 5.0 是 44 字节

可以看到`embstr`和`raw`编码都会使用`SDS`来保存值，但不同之处在于 `embstr`会**通过一次内存分配函数来分配一块连续的内存空间**来保存`redisObject`和`SDS`，而`raw`编码会通过 **调用两次内存分配函数**来分别分配两块空间来保存`redisObject`和`SDS`。Redis这样做会有很多好处：

- `embstr`编码将创建字符串对象所需的**内存分配次数**从 `raw` 编码的两次降低为一次；
- 释放 `embstr`编码的字符串对象同样**只需要调用一次内存释放函数**；
- 因为`embstr`编码的字符串对象的**所有数据都保存在一块连续的内存**里面可以更好的利用 CPU 缓存提升性能。

但是 embstr 也有缺点的：

- **如果字符串的长度增加需要重新分配内存时，整个redisObject和sds都需要重新分配空间**，所以**embstr编码的字符串对象实际上是只读的**，redis没有为embstr编码的字符串对象编写任何相应的修改程序。当我们对embstr编码的字符串对象执行任何修改命令（例如append）时，程序会先将对象的编码从embstr转换成raw，然后再执行修改命令。

##### 常用指令

普通字符串的基本操作：

```shell
# 设置 key-value 类型的值
> SET name lin
OK
# 根据 key 获得对应的 value
> GET name
"lin"
# 判断某个 key 是否存在
> EXISTS name
(integer) 1
# 返回 key 所储存的字符串值的长度
> STRLEN name
(integer) 3
# 删除某个 key 对应的值
> DEL name
(integer) 1
```

批量设置 :

```shell
# 批量设置 key-value 类型的值
> MSET key1 value1 key2 value2 
OK
# 批量获取多个 key 对应的 value
> MGET key1 key2 
1) "value1"
2) "value2"
```

**计数器（字符串的内容为整数的时候可以使用）**：

```shell
# 设置 key-value 类型的值
> SET number 0
OK
# 将 key 中储存的数字值增一
> INCR number
(integer) 1
# 将key中存储的数字值加 10
> INCRBY number 10
(integer) 11
# 将 key 中储存的数字值减一
> DECR number
(integer) 10
# 将key中存储的数字值键 10
> DECRBY number 10
(integer) 0
```

过期（默认为永不过期）：

```bash
# 设置 key 在 60 秒后过期（该方法是针对已经存在的key设置过期时间）
> EXPIRE name  60 
(integer) 1
# 查看数据还有多久过期
> TTL name 
(integer) 51

#设置 key-value 类型的值，并设置该key的过期时间为 60 秒
> SET key  value EX 60
OK
> SETEX key  60 value
OK
```

不存在就插入：

```shell
# 不存在就插入（not exists）
>SETNX key value
(integer) 1
```

##### 应用场景

###### 缓存对象

使用 String 来缓存对象有两种方式：

- **直接缓存整个对象的 JSON**，命令例子： `SET user:1 '{"name":"xiaolin", "age":18}'`。
- 采用**将 key 进行分离为 user:ID:属性，采用 MSET 存储，用 MGET 获取各属性值**，命令例子： `MSET user:1:name xiaolin user:1:age 18 user:2:name xiaomei user:2:age 20`。

###### 常规计数

**因为 Redis 处理命令是单线程，所以执行命令的过程是原子的**。因此 String 数据类型适合计数场景，比如计算访问次数、点赞、转发、库存数量等等。

比如计算文章的阅读量：

```shell
# 初始化文章的阅读量
> SET aritcle:readcount:1001 0
OK
#阅读量+1
> INCR aritcle:readcount:1001
(integer) 1
#阅读量+1
> INCR aritcle:readcount:1001
(integer) 2
#阅读量+1
> INCR aritcle:readcount:1001
(integer) 3
# 获取对应文章的阅读量
> GET aritcle:readcount:1001
"3"
```

###### 分布式锁

SET 命令有个 NX 参数可以实现「key不存在才插入」，可以用它来实现**分布式锁**：

- 如果 key 不存在，则显示插入成功，可以用来表示加锁成功；
- 如果 key 存在，则会显示插入失败，可以用来表示加锁失败。

一般而言，还会对分布式锁加上过期时间，分布式锁的命令如下：

```shell
SET lock_key unique_value NX PX 10000
```

- lock_key 就是 key 键；
- unique_value 是客户端生成的唯一的标识；
- NX 代表只在 lock_key 不存在时，才对 lock_key 进行设置操作；
- PX 10000 表示设置 lock_key 的过期时间为 10s，这是为了避免客户端发生异常而无法释放锁。

而**解锁的过程就是将 lock_key 键删除，但不能乱删，要保证执行操作的客户端就是加锁的客户端**。所以，**解锁的时候，我们要先判断锁的 unique_value 是否为加锁客户端**，是的话，才将 lock_key 键删除。

可以看到，解锁是有两个操作，这时就需要 Lua 脚本来保证解锁的原子性，因为 Redis 在执行 Lua 脚本时，可以以原子性的方式执行，保证了锁释放操作的原子性。

```lua
// 释放锁时，先比较 unique_value 是否相等，避免锁的误释放
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

这样一来，就通过使用 SET 命令和 Lua 脚本在 Redis 单节点上完成了分布式锁的加锁和解锁。

###### 共享 Session 信息

通常我们在开发后台管理系统时，会使用 Session 来保存用户的会话(登录)状态，这些 Session 信息会被保存在服务器端，但这只适用于单系统应用，如果是分布式系统此模式将不再适用。

例如用户一的 Session 信息被存储在服务器一，但第二次访问时用户一被分配到服务器二，这个时候服务器并没有用户一的 Session 信息，就会出现需要重复登录的问题，问题在于分布式系统每次会把请求随机分配到不同的服务器。

分布式系统单独存储 Session 流程图：

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/Session1.png)

因此，我们需要借助 Redis 对这些 Session 信息进行统一的存储和管理，这样无论请求发送到那台服务器，服务器都会去同一个 Redis 获取相关的 Session 信息，这样就解决了分布式系统下 Session 存储的问题。

分布式系统使用同一个 Redis 存储 Session 流程图：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/数据类型/Session2.png)

### Hash 哈希

Redis hash 是一个键值对集合。

Redis hash是一个string类型的`field`和`value`的映射表，hash特别适合用于存储对象。

Hash 是一个键值对（key - value）集合，其中 value 的形式入： `value=[{field1，value1}，...{fieldN，valueN}]`。Hash 特别适合用于存储对象。

Hash 与 String 对象的区别如下图所示:

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/数据类型/hash.png)

```
127.0.0.1:6379> HMSET my_hash_table username itcast age 18 sex male
OK
127.0.0.1:6379> HGETALL my_hash_table
1) "username"
2) "itcast"
3) "age"
4) "18"
5) "sex"
6) "male"
```

在上面的例子中，哈希数据类型用于存储包含用户基本信息的用户对象。

这里 HSET，HGETALL 是 Redis HASHES命令, 同时 `my_hash_table` 也是一个键。

#### 内部实现

Hash 类型的底层数据结构是由**压缩列表或哈希表**实现的：

- 如果哈希类型**元素个数小于 `512` 个**（默认值，可由 `hash-max-ziplist-entries` 配置），**所有值小于 `64` 字节**（默认值，可由 `hash-max-ziplist-value` 配置）的话，Redis 会使用**压缩列表**作为 Hash 类型的底层数据结构；
- 如果哈希类型元素不满足上面条件，Redis 会使用**哈希表**作为 Hash 类型的 底层数据结构。

**在 Redis 7.0 中，压缩列表数据结构已经废弃了，交由 listpack 数据结构来实现了**。

#### 常用命令

```shell
# 存储一个哈希表key的键值
HSET key field value   
# 获取哈希表key对应的field键值
HGET key field

# 在一个哈希表key中存储多个键值对
HMSET key field value [field value...] 
# 批量获取哈希表key中多个field键值
HMGET key field [field ...]       
# 删除哈希表key中的field键值
HDEL key field [field ...]    

# 返回哈希表key中field的数量
HLEN key       
# 返回哈希表key中所有的键值
HGETALL key 

# 为哈希表key中field键的值加上增量n
HINCRBY key field n                         
```

#### 应用场景

##### 缓存对象

Hash 类型的 （key，field， value） 的结构与对象的（对象id， 属性， 值）的结构相似，也可以用来存储对象。

我们以用户信息为例，它在关系型数据库中的结构是这样的：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/数据类型/用户信息.png)

我们可以使用如下命令，将用户对象的信息存储到 Hash 类型：

```shell
# 存储一个哈希表uid:1的键值
> HSET uid:1 name Tom age 15
2
# 存储一个哈希表uid:2的键值
> HSET uid:2 name Jerry age 13
2
# 获取哈希表用户id为1中所有的键值
> HGETALL uid:1
1) "name"
2) "Tom"
3) "age"
4) "15"
```

Redis Hash 存储其结构如下图：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/数据类型/hash存储结构.png)

在介绍 String 类型的应用场景时有所介绍，String + Json也是存储对象的一种方式，那么存储对象时，到底用 String + json 还是用 Hash 呢？

一般对象用 String + Json 存储，**对象中某些频繁变化的属性可以考虑抽出来用 Hash 类型存储**。

##### 购物车

以用户 id 为 key，商品 id 为 field，商品数量为 value，恰好构成了购物车的3个要素，如下图所示。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/数据类型/购物车.png)

涉及的命令如下：

- 添加商品：`HSET cart:{用户id} {商品id} 1`
- 添加数量：`HINCRBY cart:{用户id} {商品id} 1`
- 商品总数：`HLEN cart:{用户id}`
- 删除商品：`HDEL cart:{用户id} {商品id}`
- 获取购物车所有商品：`HGETALL cart:{用户id}`

当前仅仅是将商品ID存储到了Redis 中，在回显商品具体信息的时候，还需要拿着商品 id 查询一次数据库，获取完整的商品的信息。

### List 列表

List 列表是简单的字符串列表，**按照插入顺序排序**，可以从头部或尾部向 List 列表添加元素。

列表的最大长度为 `2^32 - 1`，也即每个列表支持超过 `40 亿`个元素。

```
redis 127.0.0.1:6379> lpush tutorial_list redis
(integer) 1
redis 127.0.0.1:6379> lpush tutorial_list mongodb
(integer) 2
redis 127.0.0.1:6379> lpush tutorial_list rabbitmq
(integer) 3
redis 127.0.0.1:6379> lrange tutorial_list 0 10
1) "rabitmq"
2) "mongodb"
3) "redis"
```

#### 内部实现

List 类型的底层数据结构是由**双向链表或压缩列表**实现的：

- 如果列**表的元素个数小于 `512` 个**（默认值，可由 `list-max-ziplist-entries` 配置），列表每个元素的值都小于 `64` 字节（默认值，可由 `list-max-ziplist-value` 配置），Redis 会使用**压缩列表**作为 List 类型的底层数据结构；
- 如果列表的元素不满足上面的条件，Redis 会使用**双向链表**作为 List 类型的底层数据结构；

但是**在 Redis 3.2 版本之后，List 数据类型底层数据结构就只由 quicklist 实现了，替代了双向链表和压缩列表**。

#### 常用命令

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/list.png)

```shell
# 将一个或多个值value插入到key列表的表头(最左边)，最后的值在最前面
LPUSH key value [value ...] 
# 将一个或多个值value插入到key列表的表尾(最右边)
RPUSH key value [value ...]
# 移除并返回key列表的头元素
LPOP key     
# 移除并返回key列表的尾元素
RPOP key 

# 返回列表key中指定区间内的元素，区间以偏移量start和stop指定，从0开始
LRANGE key start stop

# 从key列表表头弹出一个元素，没有就阻塞timeout秒，如果timeout=0则一直阻塞
BLPOP key [key ...] timeout
# 从key列表表尾弹出一个元素，没有就阻塞timeout秒，如果timeout=0则一直阻塞
BRPOP key [key ...] timeout
```

#### 应用场景

##### 消息队列

消息队列在存取消息时，必须要满足三个需求，分别是**消息保序、处理重复的消息和保证消息可靠性**。

Redis 的 List 和 Stream 两种数据类型，就可以满足消息队列的这三个需求。我们先来了解下基于 List 的消息队列实现方法，后面在介绍 Stream 数据类型时候，在详细说说 Stream。

1、如何满足消息保序需求？

List 本身就是按先进先出的顺序对数据进行存取的，所以，如果使用 List 作为消息队列保存消息的话，就已经能满足消息保序的需求了。

List 可以使用 LPUSH + RPOP （或者反过来，RPUSH+LPOP）命令实现消息队列。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/数据类型/list消息队列.png)

- 生产者使用 `LPUSH key value[value...]` 将消息插入到队列的头部，如果 key 不存在则会创建一个空的队列再插入消息。
- 消费者使用 `RPOP key` 依次读取队列的消息，先进先出。

不过，在消费者读取数据时，有一个潜在的性能风险点。

在生产者往 List 中写入数据时，**List 并不会主动地通知消费者有新消息写入**，如果消费者想要及时处理消息，就需要在程序中不停地调用 `RPOP` 命令（比如使用一个while(1)循环）。如果有新消息写入，RPOP命令就会返回结果，否则，RPOP命令返回空值，再继续循环。

所以，即使没有新消息写入List，消费者也要不停地调用 RPOP 命令，这就会导致消费者程序的 CPU 一直消耗在执行 RPOP 命令上，带来不必要的性能损失。

为了解决这个问题，Redis提供了 BRPOP 命令。**BRPOP命令也称为阻塞式读取，客户端在没有读到队列数据时，自动阻塞，直到有新的数据写入队列，再开始读取新数据**。和消费者程序自己不停地调用RPOP命令相比，这种方式能节省CPU开销。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/数据类型/消息队列.png)

**2、如何处理重复的消息？**

消费者要实现重复消息的判断，需要 2 个方面的要求：

- **每个消息都有一个全局的 ID**。
- **消费者要记录已经处理过的消息的 ID**。当收到一条消息后，消费者程序就可以对比收到的消息 ID 和记录的已处理过的消息 ID，来判断当前收到的消息有没有经过处理。如果已经处理过，那么，消费者程序就不再进行处理了。

但是 **List 并不会为每个消息生成 ID 号，所以我们需要自行为每个消息生成一个全局唯一ID**，生成之后，我们在用 LPUSH 命令把消息插入 List 时，需要在消息中包含这个全局唯一 ID。

例如，我们执行以下命令，就把一条全局 ID 为 111000102、库存量为 99 的消息插入了消息队列：

```shell
> LPUSH mq "111000102:stock:99"
(integer) 1
```

**3、如何保证消息可靠性？**

当消费者程序从 List 中读取一条消息后，List 就不会再留存这条消息了。所以，如果消费者程序在处理消息的过程出现了故障或宕机，就会导致消息没有处理完成，那么，消费者程序再次启动后，就没法再次从 List 中读取消息了。

为了留存消息，List 类型提供了 `BRPOP LPUSH` 命令，这个命令的**作用是让消费者程序从一个 List 中读取消息，同时，Redis 会把这个消息再插入到另一个 List（可以叫作备份 List）留存**。

这样一来，如果消费者程序读了消息但没能正常处理，等它重启后，就可以从备份 List 中重新读取消息并进行处理了。

好了，到这里可以知道基于 List 类型的消息队列，满足消息队列的三大需求（消息保序、处理重复的消息和保证消息可靠性）。

- 消息保序：使用 LPUSH + RPOP；
- 阻塞读取：使用 BRPOP；
- 重复消息处理：生产者自行实现全局唯一 ID；
- 消息的可靠性：使用 BRPOPLPUSH

> List 作为消息队列有什么缺陷？

**List 不支持多个消费者消费同一条消息**，因为一旦消费者拉取一条消息后，这条消息就从 List 中删除了，无法被其它消费者再次消费。

要实现一条消息可以被多个消费者消费，那么就要将多个消费者组成一个消费组，使得多个消费者可以消费同一条消息，但是 **List 类型并不支持消费组的实现**。

这就要说起 Redis 从 5.0 版本开始提供的 Stream 数据类型了，Stream 同样能够满足消息队列的三大需求，而且它还支持「消费组」形式的消息读取。

### Set 集合

Redis Set是`string`类型的无序集合。Set 类型是一个无序并唯一的键值集合，它的存储顺序不会按照插入的先后顺序进行存储。

一个集合最多可以存储 `2^32-1` 个元素。概念和数学中个的集合基本类似，可以**交集，并集，差集**等等，所以 Set 类型除了**支持集合内的增删改查**，同时还支持**多个集合取交集、并集、差集**。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/set.png)

**集合是通过哈希表实现的**，所以添加，删除，查找的复杂度都是O(1)。

在 Redis 可以添加，删除和测试成员存在的时间复杂度为 O（1）。

```
127.0.0.1:6379> sadd myset redis
(integer) 1
127.0.0.1:6379> sadd myset mongodb
(integer) 1
127.0.0.1:6379> sadd myset rabitmq
(integer) 1
127.0.0.1:6379> sadd myset rabitmq
(integer) 0
127.0.0.1:6379> smembers myset
1) "mongodb"
2) "redis"
3) "rabitmq"
```

注：在上面的例子中 rabitmq 被添加两次，但由于它是只集合具有唯一特性。

集合中最大的成员数为 232 - 1(4294967295, 每个集合可存储40多亿个成员)。

Set 类型和 List 类型的区别如下：

- List 可以存储重复元素，Set 只能存储非重复元素；
- List 是按照元素的先后顺序存储元素的，而 Set 则是无序方式存储元素的。

#### 内部实现

Set 类型的底层数据结构是由**哈希表或整数集合**实现的：

- 如果集合中的元素都是**整数且元素个数小于 `512`** （默认值，`set-maxintset-entries`配置）个，Redis 会使用**整数集合**作为 Set 类型的底层数据结构；
- 如果集合中的元素不满足上面条件，则 Redis 使用**哈希表**作为 Set 类型的底层数据结构。

#### 常用命令

Set常用操作：

```shell
# 往集合key中存入元素，元素存在则忽略，若key不存在则新建
SADD key member [member ...]
# 从集合key中删除元素
SREM key member [member ...] 
# 获取集合key中所有元素
SMEMBERS key
# 获取集合key中的元素个数
SCARD key

# 判断member元素是否存在于集合key中
SISMEMBER key member

# 从集合key中随机选出count个元素，元素不从key中删除
SRANDMEMBER key [count]
# 从集合key中随机选出count个元素，元素从key中删除
SPOP key [count]
```

Set运算操作：

```shell
# 交集运算
SINTER key [key ...]
# 将交集结果存入新集合destination中
SINTERSTORE destination key [key ...]

# 并集运算
SUNION key [key ...]
# 将并集结果存入新集合destination中
SUNIONSTORE destination key [key ...]

# 差集运算
SDIFF key [key ...]
# 将差集结果存入新集合destination中
SDIFFSTORE destination key [key ...]
```

#### 应用场景

**集合的主要几个特性，无序、不可重复、支持并交差**等操作。

因此 Set 类型比较适合用来数据去重和保障数据的唯一性，还可以用来统计多个集合的交集、错集和并集等，当我们存储的数据是无序并且需要去重的情况下，比较适合使用集合类型进行存储。

但是要提醒你一下，这里有一个潜在的风险。**Set 的差集、并集和交集的计算复杂度较高，在数据量较大的情况下，如果直接执行这些计算，会导致 Redis 实例阻塞**。

在主从集群中，为了避免主库因为 Set 做聚合计算（交集、差集、并集）时导致主库被阻塞，我们可以选择一个从库完成聚合统计，或者把数据返回给客户端，由客户端来完成聚合统计。

##### 点赞

Set 类型可以保证一个用户只能点一个赞，这里举例子一个场景，key 是文章id，value 是用户id。

`uid:1` 、`uid:2`、`uid:3` 三个用户分别对 article:1 文章点赞了。

```shell
# uid:1 用户对文章 article:1 点赞
> SADD article:1 uid:1
(integer) 1
# uid:2 用户对文章 article:1 点赞
> SADD article:1 uid:2
(integer) 1
# uid:3 用户对文章 article:1 点赞
> SADD article:1 uid:3
(integer) 1
```

`uid:1` 取消了对 article:1 文章点赞。

```text
> SREM article:1 uid:1
(integer) 1
```

获取 article:1 文章所有点赞用户 :

```shell
> SMEMBERS article:1
1) "uid:3"
2) "uid:2"
```

获取 article:1 文章的点赞用户数量：

```shell
> SCARD article:1
(integer) 2
```

判断用户 `uid:1` 是否对文章 article:1 点赞了：

```shell
> SISMEMBER article:1 uid:1
(integer) 0  # 返回0说明没点赞，返回1则说明点赞了
```

##### 共同关注

Set 类型支持交集运算，所以可以用来计算共同关注的好友、公众号等。

key 可以是用户id，value 则是已关注的公众号的id。

`uid:1` 用户关注公众号 id 为 5、6、7、8、9，`uid:2` 用户关注公众号 id 为 7、8、9、10、11。

```shell
# uid:1 用户关注公众号 id 为 5、6、7、8、9
> SADD uid:1 5 6 7 8 9
(integer) 5
# uid:2  用户关注公众号 id 为 7、8、9、10、11
> SADD uid:2 7 8 9 10 11
(integer) 5
```

`uid:1` 和 `uid:2` 共同关注的公众号：

```shell
# 获取共同关注
> SINTER uid:1 uid:2
1) "7"
2) "8"
3) "9"
```

给 `uid:2` 推荐 `uid:1` 关注的公众号：

```shell
> SDIFF uid:1 uid:2
1) "5"
2) "6"
```

验证某个公众号是否同时被 `uid:1` 或 `uid:2` 关注:

```shell
> SISMEMBER uid:1 5
(integer) 1 # 返回0，说明关注了
> SISMEMBER uid:2 5
(integer) 0 # 返回0，说明没关注
```

##### 抽奖活动

存储某活动中中奖的用户名 ，Set 类型因为有去重功能，可以保证同一个用户不会中奖两次。

key为抽奖活动名，value为员工名称，把所有员工名称放入抽奖箱 ：

```shell
>SADD lucky Tom Jerry John Sean Marry Lindy Sary Mark
(integer) 5
```

如果允许重复中奖，可以使用 SRANDMEMBER 命令。

```shell
# 抽取 1 个一等奖：
> SRANDMEMBER lucky 1
1) "Tom"
# 抽取 2 个二等奖：
> SRANDMEMBER lucky 2
1) "Mark"
2) "Jerry"
# 抽取 3 个三等奖：
> SRANDMEMBER lucky 3
1) "Sary"
2) "Tom"
3) "Jerry"
```

如果不允许重复中奖，可以使用 SPOP 命令。

```shell
# 抽取一等奖1个
> SPOP lucky 1
1) "Sary"
# 抽取二等奖2个
> SPOP lucky 2
1) "Jerry"
2) "Mark"
# 抽取三等奖3个
> SPOP lucky 3
1) "John"
2) "Sean"
3) "Lindy"
```

### zset(sorted set：有序集合)

Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个double类型的分数(权重)。redis正是通过分数来为集合中的成员(权重)进行从小到大的排序。

**zset的成员是唯一的,但分数(score)却可以重复**。

Zset 类型（有序集合类型）相比于 Set 类型多了一个排序属性 score（分值），对于有序集合 ZSet 来说，每个存储元素相当于有两个值组成的，一个是有序结合的元素值，一个是排序值。

有序集合保留了集合不能有重复成员的特性（分值可以重复），但不同的是，有序集合中的元素可以排序。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/zset.png)

```
127.0.0.1:6379> zadd mysortset 0 redis
(integer) 1
127.0.0.1:6379> zadd mysortset 2 mongodb
(integer) 1
127.0.0.1:6379> zadd mysortset 1 rabitmq
(integer) 1
127.0.0.1:6379> ZRANGEBYSCORE mysortset 0 1000
1) "redis"
2) "rabitmq"
3) "mongodb"
```

全部数据类型相关操作指令在 http://redis.cn/commands.html有详细介绍

#### 内部实现

Zset 类型的底层数据结构是由**压缩列表或跳表**实现的：

- 如果有序集合的**元素个数小于 `128` 个，并且每个元素的值小于 `64` 字节时**，Redis 会使用**压缩列表**作为 Zset 类型的底层数据结构；
- 如果有序集合的元素不满足上面的条件，Redis 会使用**跳表**作为 Zset 类型的底层数据结构；

**在 Redis 7.0 中，压缩列表数据结构已经废弃了，交由 listpack 数据结构来实现了。**

#### 常用命令

Zset 常用操作：

```shell
# 往有序集合key中加入带分值元素
ZADD key score member [[score member]...]   
# 往有序集合key中删除元素
ZREM key member [member...]                 
# 返回有序集合key中元素member的分值
ZSCORE key member
# 返回有序集合key中元素个数
ZCARD key 

# 为有序集合key中元素member的分值加上increment
ZINCRBY key increment member 

# 正序获取有序集合key从start下标到stop下标的元素
ZRANGE key start stop [WITHSCORES]
# 倒序获取有序集合key从start下标到stop下标的元素
ZREVRANGE key start stop [WITHSCORES]

# 返回有序集合中指定分数区间内的成员，分数由低到高排序。
ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]

# 返回指定成员区间内的成员，按字典正序排列, 分数必须相同。
ZRANGEBYLEX key min max [LIMIT offset count]
# 返回指定成员区间内的成员，按字典倒序排列, 分数必须相同
ZREVRANGEBYLEX key max min [LIMIT offset count]
```

Zset 运算操作（相比于 Set 类型，ZSet 类型没有支持差集运算）：

```shell
# 并集计算(相同元素分值相加)，numberkeys一共多少个key，WEIGHTS每个key对应的分值乘积
ZUNIONSTORE destkey numberkeys key [key...] 
# 交集计算(相同元素分值相加)，numberkeys一共多少个key，WEIGHTS每个key对应的分值乘积
ZINTERSTORE destkey numberkeys key [key...]
```

#### 应用场景

Zset 类型（Sorted Set，有序集合） 可以根据元素的权重来排序，我们可以自己来决定每个元素的权重值。比如说，我们可以根据元素插入 Sorted Set 的时间确定权重值，先插入的元素权重小，后插入的元素权重大。

在面对**需要展示最新列表、排行榜等场景时，如果数据更新频繁或者需要分页显示**，可以优先考虑使用 Sorted Set。

##### 排行榜

有序集合比较典型的使用场景就是排行榜。例如学生成绩的排名榜、游戏积分排行榜、视频播放排名、电商系统中商品的销量排名等。

我们以博文点赞排名为例，小林发表了五篇博文，分别获得赞为 200、40、100、50、150。

```shell
# arcticle:1 文章获得了200个赞
> ZADD user:xiaolin:ranking 200 arcticle:1
(integer) 1
# arcticle:2 文章获得了40个赞
> ZADD user:xiaolin:ranking 40 arcticle:2
(integer) 1
# arcticle:3 文章获得了100个赞
> ZADD user:xiaolin:ranking 100 arcticle:3
(integer) 1
# arcticle:4 文章获得了50个赞
> ZADD user:xiaolin:ranking 50 arcticle:4
(integer) 1
# arcticle:5 文章获得了150个赞
> ZADD user:xiaolin:ranking 150 arcticle:5
(integer) 1
```

文章 arcticle:4 新增一个赞，可以使用 ZINCRBY 命令（为有序集合key中元素member的分值加上increment）：

```shell
> ZINCRBY user:xiaolin:ranking 1 arcticle:4
"51"
```

查看某篇文章的赞数，可以使用 ZSCORE 命令（返回有序集合key中元素个数）：

```shell
> ZSCORE user:xiaolin:ranking arcticle:4
"50"
```

获取小林文章赞数最多的 3 篇文章，可以使用 ZREVRANGE 命令（倒序获取有序集合 key 从start下标到stop下标的元素）：

```shell
# WITHSCORES 表示把 score 也显示出来
> ZREVRANGE user:xiaolin:ranking 0 2 WITHSCORES
1) "arcticle:1"
2) "200"
3) "arcticle:5"
4) "150"
5) "arcticle:3"
6) "100"
```

获取小林 100 赞到 200 赞的文章，可以使用 ZRANGEBYSCORE 命令（返回有序集合中指定分数区间内的成员，分数由低到高排序）：

```shell
> ZRANGEBYSCORE user:xiaolin:ranking 100 200 WITHSCORES
1) "arcticle:3"
2) "100"
3) "arcticle:5"
4) "150"
5) "arcticle:1"
6) "200"
```

##### 电话、姓名排序

使用有序集合的 `ZRANGEBYLEX` 或 `ZREVRANGEBYLEX` 可以帮助我们实现电话号码或姓名的排序，我们以 `ZRANGEBYLEX` （返回指定成员区间内的成员，按 key 正序排列，分数必须相同）为例。

**注意：不要在分数不一致的 SortSet 集合中去使用 ZRANGEBYLEX和 ZREVRANGEBYLEX 指令，因为获取的结果会不准确。**

*1、电话排序*

我们可以将电话号码存储到 SortSet 中，然后根据需要来获取号段：

```shell
> ZADD phone 0 13100111100 0 13110114300 0 13132110901 
(integer) 3
> ZADD phone 0 13200111100 0 13210414300 0 13252110901 
(integer) 3
> ZADD phone 0 13300111100 0 13310414300 0 13352110901 
(integer) 3
```

获取所有号码:

```shell
> ZRANGEBYLEX phone - +
1) "13100111100"
2) "13110114300"
3) "13132110901"
4) "13200111100"
5) "13210414300"
6) "13252110901"
7) "13300111100"
8) "13310414300"
9) "13352110901"
```

获取 132 号段的号码：

```shell
> ZRANGEBYLEX phone [132 (133
1) "13200111100"
2) "13210414300"
3) "13252110901"
```

获取132、133号段的号码：

```shell
> ZRANGEBYLEX phone [132 (134
1) "13200111100"
2) "13210414300"
3) "13252110901"
4) "13300111100"
5) "13310414300"
6) "13352110901"
```

*2、姓名排序*

```shell
> zadd names 0 Toumas 0 Jake 0 Bluetuo 0 Gaodeng 0 Aimini 0 Aidehua 
(integer) 6
```

获取所有人的名字:

```shell
> ZRANGEBYLEX names - +
1) "Aidehua"
2) "Aimini"
3) "Bluetuo"
4) "Gaodeng"
5) "Jake"
6) "Toumas"
```

获取名字中大写字母A开头的所有人：

```shell
> ZRANGEBYLEX names [A (B
1) "Aidehua"
2) "Aimini"
```

获取名字中大写字母 C 到 Z 的所有人：

```shell
> ZRANGEBYLEX names [C [Z
1) "Gaodeng"
2) "Jake"
3) "Toumas"
```

### BitMap

#### 介绍

**Bitmap，即位图**，是一串连续的二进制数组（0和1），可以通过偏移量（offset）定位元素。BitMap通过最小的单位bit来进行`0|1`的设置，表示某个元素的值或者状态，时间复杂度为O(1)。

由于 bit 是计算机中最小的单位，使用它进行储存将非常节省空间，特别适合一些数据量大且使用**二值统计的场景**。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/bitmap.png)

#### 内部实现

Bitmap 本身**是用 String 类型作为底层数据结构**实现的一种统计二值状态的数据类型。

String 类型是会保存为二进制的字节数组，所以，Redis 就把字节数组的每个 bit 位利用起来，用来表示一个元素的二值状态，你可以把 Bitmap 看作是一个 bit 数组。

#### 常用命令

bitmap 基本操作：

```shell
# 设置值，其中value只能是 0 和 1
SETBIT key offset value

# 获取值
GETBIT key offset

# 获取指定范围内值为 1 的个数
# start 和 end 以字节为单位
BITCOUNT key start end
```

bitmap 运算操作：

```shell
# BitMap间的运算
# operations 位移操作符，枚举值
  AND 与运算 &
  OR 或运算 |
  XOR 异或 ^
  NOT 取反 ~
# result 计算的结果，会存储在该key中
# key1 … keyn 参与运算的key，可以有多个，空格分割，not运算只能一个key
# 当 BITOP 处理不同长度的字符串时，较短的那个字符串所缺少的部分会被看作 0。返回值是保存到 destkey 的字符串的长度（以字节byte为单位），和输入 key 中最长的字符串长度相等。
BITOP [operations] [result] [key1] [keyn…]

# 返回指定key中第一次出现指定value(0/1)的位置
BITPOS [key] [value]
```

#### 应用场景

Bitmap 类型**非常适合二值状态统计的场景**，这里的二值状态就是指**集合元素的取值就只有 0 和 1 两种**，在记录海量数据时，Bitmap 能够有效地节省内存空间。

##### 签到统计

在签到打卡的场景中，我们只用记录签到（1）或未签到（0），所以它就是非常典型的二值状态。

签到统计时，每个用户一天的签到用 1 个 bit 位就能表示，一个月（假设是 31 天）的签到情况用 31 个 bit 位就可以，而一年的签到也只需要用 365 个 bit 位，根本不用太复杂的集合类型。

假设我们要统计 ID 100 的用户在 2022 年 6 月份的签到情况，就可以按照下面的步骤进行操作。

第一步，执行下面的命令，记录该用户 6 月 3 号已签到。

```shell
SETBIT uid:sign:100:202206 2 1
```

第二步，检查该用户 6 月 3 日是否签到。

```shell
GETBIT uid:sign:100:202206 2 
```

第三步，统计该用户在 6 月份的签到次数。

```shell
BITCOUNT uid:sign:100:202206
```

这样，我们就知道该用户在 6 月份的签到情况了。

> 如何统计这个月首次打卡时间呢？

Redis 提供了 `BITPOS key bitValue [start] [end]`指令，返回数据表示 Bitmap 中第一个值为 `bitValue` 的 offset 位置。

在默认情况下， 命令将检测整个位图， 用户可以通过可选的 `start` 参数和 `end` 参数指定要检测的范围。所以我们可以通过执行这条命令来获取 userID = 100 在 2022 年 6 月份**首次打卡**日期：

```text
BITPOS uid:sign:100:202206 1
```

需要注意的是，因为 offset 从 0 开始的，所以我们需要将返回的 value + 1 。

##### 判断用户登陆态

Bitmap 提供了 `GETBIT、SETBIT` 操作，**通过一个偏移值 offset 对 bit 数组的 offset 位置的 bit 位进行读写操作**，需要注意的是 offset 从 0 开始。

只需要一个 key = login_status 表示存储用户登陆状态集合数据， 将用户 ID 作为 offset，在线就设置为 1，下线设置 0。通过 `GETBIT`判断对应的用户是否在线。 50000 万 用户只需要 6 MB 的空间。

假如我们要判断 ID = 10086 的用户的登陆情况：

第一步，执行以下指令，表示用户已登录。

```shell
SETBIT login_status 10086 1
```

第二步，检查该用户是否登陆，返回值 1 表示已登录。

```text
GETBIT login_status 10086
```

第三步，登出，将 offset 对应的 value 设置成 0。

```shell
SETBIT login_status 10086 0
```

##### 连续签到用户总数

如何统计出这连续 7 天连续打卡用户总数呢？

我们把每天的日期作为 Bitmap 的 key，userId 作为 offset，若是打卡则将 offset 位置的 bit 设置成 1。

key 对应的集合的每个 bit 位的数据则是一个用户在该日期的打卡记录。

一共有 7 个这样的 Bitmap，如果我们能对这 7 个 Bitmap 的对应的 bit 位做 『与』运算。同样的 UserID offset 都是一样的，当一个 userID 在 7 个 Bitmap 对应对应的 offset 位置的 bit = 1 就说明该用户 7 天连续打卡。

结果保存到一个新 Bitmap 中，我们再通过 `BITCOUNT` 统计 bit = 1 的个数便得到了连续打卡 3 天的用户总数了。

Redis 提供了 `BITOP operation destkey key [key ...]`这个指令用于对一个或者多个 key 的 Bitmap 进行位元操作。

- `opration` 可以是 `and`、`OR`、`NOT`、`XOR`。当 BITOP 处理不同长度的字符串时，较短的那个字符串所缺少的部分会被看作 `0` 。空的 `key` 也被看作是包含 `0` 的字符串序列。

举个例子，比如将三个 bitmap 进行 AND 操作，并将结果保存到 destmap 中，接着对 destmap 执行 BITCOUNT 统计。

```shell
# 与操作
BITOP AND destmap bitmap:01 bitmap:02 bitmap:03
# 统计 bit 位 =  1 的个数
BITCOUNT destmap
```

即使一天产生一个亿的数据，Bitmap 占用的内存也不大，大约占 12 MB 的内存（10^8/8/1024/1024），7 天的 Bitmap 的内存开销约为 84 MB。同时我们最好给 Bitmap 设置过期时间，让 Redis 删除过期的打卡数据，节省内存。

### HyperLogLog

#### 介绍

Redis HyperLogLog 是 Redis 2.8.9 版本新增的数据类型，是一种**用于「统计基数」的数据集合类型**，基数统计就是指统计一个集合中不重复的元素个数。但要注意，HyperLogLog 是统计规则是基于概率完成的，不是非常准确，标准误算率是 0.81%。

所以，简单来说 HyperLogLog **提供不精确的去重计数**。

HyperLogLog 的**优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的内存空间总是固定的、并且是很小的**。

在 Redis 里面，**每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 `2^64` 个不同元素的基数**，和元素越多就越耗费内存的 Set 和 Hash 类型相比，HyperLogLog 就非常节省空间。

这什么概念？举个例子给大家对比一下。

用 Java 语言来说，一般 long 类型占用 8 字节，而 1 字节有 8 位，即：1 byte = 8 bit，即 long 数据类型最大可以表示的数是：`2^63-1`。对应上面的`2^64`个数，假设此时有`2^63-1`这么多个数，从 `0 ~ 2^63-1`，按照`long`以及`1k = 1024 字节`的规则来计算内存总数，就是：`((2^63-1) * 8/1024)K`，这是很庞大的一个数，存储空间远远超过`12K`，而 `HyperLogLog` 却可以用 `12K` 就能统计完。

#### 内部实现

HyperLogLog 的实现涉及到很多数学问题，太费脑子了，我也没有搞懂，如果你想了解一下，课下可以看看这个：[HyperLogLog (opens new window)](https://en.wikipedia.org/wiki/HyperLogLog)。

#### 常见命令

HyperLogLog 命令很少，就三个。

```shell
# 添加指定元素到 HyperLogLog 中
PFADD key element [element ...]

# 返回给定 HyperLogLog 的基数估算值。
PFCOUNT key [key ...]

# 将多个 HyperLogLog 合并为一个 HyperLogLog
PFMERGE destkey sourcekey [sourcekey ...]
```

#### 应用场景

##### 百万级网页 UV 计数

Redis HyperLogLog 优势在于只需要花费 12 KB 内存，就可以计算接近 2^64 个元素的基数，和元素越多就越耗费内存的 Set 和 Hash 类型相比，HyperLogLog 就非常节省空间。

所以，非常适合统计百万级以上的网页 UV 的场景。

在统计 UV 时，你可以用 PFADD 命令（用于向 HyperLogLog 中添加新元素）把访问页面的每个用户都添加到 HyperLogLog 中。

```shell
PFADD page1:uv user1 user2 user3 user4 user5
```

接下来，就可以用 PFCOUNT 命令直接获得 page1 的 UV 值了，这个命令的作用就是返回 HyperLogLog 的统计结果。

```shell
PFCOUNT page1:uv
```

不过，有一点需要你注意一下，HyperLogLog 的统计规则是基于概率完成的，所以它给出的统计结果是有一定误差的，标准误算率是 0.81%。

这也就意味着，你使用 HyperLogLog 统计的 UV 是 100 万，但实际的 UV 可能是 101 万。虽然误差率不算大，但是，如果你需要精确统计结果的话，最好还是继续用 Set 或 Hash 类型。

### GEO

Redis GEO 是 Redis 3.2 版本新增的数据类型，主要**用于存储地理位置信息，并对存储的信息进行操作**。

在日常生活中，我们越来越依赖搜索“附近的餐馆”、在打车软件上叫车，这些都离不开基于位置信息服务（Location-Based Service，LBS）的应用。LBS 应用访问的数据是和人或物关联的一组经纬度信息，而且要能查询相邻的经纬度范围，GEO 就非常适合应用在 LBS 服务的场景中。

#### 内部实现

GEO 本身并没有设计新的底层数据结构，而是**直接使用了 Sorted Set 集合类型**。

GEO 类型**使用 GeoHash 编码**方法实现了**经纬度到 Sorted Set 中元素权重分数的转换**，这其中的两个关键机制就是「对二维地图做区间划分」和「对区间进行编码」。一组经纬度落在某个区间后，就用区间的编码值来表示，并把编码值作为 Sorted Set 元素的权重分数。

这样一来，我们就可以把经纬度保存到 Sorted Set 中，利用 Sorted Set 提供的“按权重进行有序范围查找”的特性，实现 LBS 服务中频繁使用的“搜索附近”的需求。

#### 常用命令

```shell
# 存储指定的地理空间位置，可以将一个或多个经度(longitude)、纬度(latitude)、位置名称(member)添加到指定的 key 中。
GEOADD key longitude latitude member [longitude latitude member ...]

# 从给定的 key 里返回所有指定名称(member)的位置（经度和纬度），不存在的返回 nil。
GEOPOS key member [member ...]

# 返回两个给定位置之间的距离。
GEODIST key member1 member2 [m|km|ft|mi]

# 根据用户给定的经纬度坐标来获取指定范围内的地理位置集合。
GEORADIUS key longitude latitude radius m|km|ft|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] [ASC|DESC] [STORE key] [STOREDIST key]
```

#### 应用场景

##### 滴滴叫车

这里以滴滴叫车的场景为例，介绍下具体如何使用 GEO 命令：GEOADD 和 GEORADIUS 这两个命令。

假设车辆 ID 是 33，经纬度位置是（116.034579，39.030452），我们可以用一个 GEO 集合保存所有车辆的经纬度，集合 key 是 cars:locations。

执行下面的这个命令，就可以把 ID 号为 33 的车辆的当前经纬度位置存入 GEO 集合中：

```shell
GEOADD cars:locations 116.034579 39.030452 33
```

当用户想要寻找自己附近的网约车时，LBS 应用就可以使用 GEORADIUS 命令。

例如，LBS 应用执行下面的命令时，Redis 会根据输入的用户的经纬度信息（116.054579，39.030452 ），查找以这个经纬度为中心的 5 公里内的车辆信息，并返回给 LBS 应用。

```shell
GEORADIUS cars:locations 116.054579 39.030452 5 km ASC COUNT 10
```

### Stream

#### 介绍

Redis Stream 是 Redis 5.0 版本新增加的数据类型，Redis **专门为消息队列设计的数据类型**。

在 Redis 5.0 Stream 没出来之前，消息队列的实现方式都有着各自的缺陷，例如：

- 发布订阅模式，不能持久化也就无法可靠的保存消息，并且对于离线重连的客户端不能读取历史消息的缺陷；
- List 实现消息队列的方式不能重复消费，一个消息消费完就会被删除，而且生产者需要自行实现全局唯一 ID。

基于以上问题，Redis 5.0 便推出了 Stream 类型也是此版本最重要的功能，用于完美地实现消息队列，它支持消息的持久化、支持自动生成全局唯一 ID、支持 ack 确认消息的模式、支持消费组模式等，让消息队列更加的稳定和可靠。

#### 常见命令

Stream 消息队列操作命令：

- XADD：插入消息，保证有序，可以自动生成全局唯一 ID；
- XLEN ：查询消息长度；
- XREAD：用于读取消息，可以按 ID 读取数据；
- XDEL ： 根据消息 ID 删除消息；
- DEL ：删除整个 Stream；
- XRANGE ：读取区间消息
- XREADGROUP：按消费组形式读取消息；
- XPENDING 和 XACK：
  - XPENDING 命令可以用来查询每个消费组内所有消费者「已读取、但尚未确认」的消息；
  - XACK 命令用于向消息队列确认消息处理已完成；

#### 应用场景

##### 消息队列

生产者通过 XADD 命令插入一条消息：

```shell
# * 表示让 Redis 为插入的数据自动生成一个全局唯一的 ID
# 往名称为 mymq 的消息队列中插入一条消息，消息的键是 name，值是 xiaolin
> XADD mymq * name xiaolin
"1654254953808-0"
```

**插入成功后会返回全局唯一的 ID**："1654254953808-0"。消息的全局唯一 ID 由两部分组成：

- 第一部分“1654254953808”是数据插入时，以毫秒为单位计算的当前服务器时间；
- 第二部分表示插入消息在当前毫秒内的消息序号，这是从 0 开始编号的。例如，“1654254953808-0”就表示在“1654254953808”毫秒内的第 1 条消息。

消费者通过 XREAD 命令从消息队列中读取消息时，可以指定一个消息 ID，并从这个消息 ID 的下一条消息开始进行读取（注意是输入消息 ID 的下一条信息开始读取，不是查询输入ID的消息）。

```shell
# 从 ID 号为 1654254953807-0 的消息开始，读取后续的所有消息（示例中一共 1 条）。
> XREAD STREAMS mymq 1654254953807-0
1) 1) "mymq"
   2) 1) 1) "1654254953808-0"
         2) 1) "name"
            2) "xiaolin"
```

如果**想要实现阻塞读（当没有数据时，阻塞住），可以调用 XRAED 时设定 BLOCK 配置项**，实现类似于 BRPOP 的阻塞读取操作。

比如，下面这命令，设置了 BLOCK 10000 的配置项，10000 的单位是毫秒，表明 XREAD 在读取最新消息时，如果没有消息到来，XREAD 将阻塞 10000 毫秒（即 10 秒），然后再返回。

```shell
# 命令最后的“$”符号表示读取最新的消息
> XREAD BLOCK 10000 STREAMS mymq $
(nil)
(10.00s)
```

Stream 的基础方法，使用 xadd 存入消息和 xread 循环阻塞读取消息的方式可以实现简易版的消息队列，交互流程如下图所示：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/数据类型/Stream简易.png)

> 前面介绍的这些操作 List 也支持的，接下来看看 Stream 特有的功能。

Stream 可以以使用 **XGROUP 创建消费组**，创建消费组之后，Stream 可以使用 XREADGROUP 命令让消费组内的消费者读取消息。

创建两个消费组，这两个消费组消费的消息队列是 mymq，都指定从第一条消息开始读取：

```shell
# 创建一个名为 group1 的消费组，0-0 表示从第一条消息开始读取。
> XGROUP CREATE mymq group1 0-0
OK
# 创建一个名为 group2 的消费组，0-0 表示从第一条消息开始读取。
> XGROUP CREATE mymq group2 0-0
OK
```

消费组 group1 内的消费者 consumer1 从 mymq 消息队列中读取所有消息的命令如下：

```shell
# 命令最后的参数“>”，表示从第一条尚未被消费的消息开始读取。
> XREADGROUP GROUP group1 consumer1 STREAMS mymq >
1) 1) "mymq"
   2) 1) 1) "1654254953808-0"
         2) 1) "name"
            2) "xiaolin"
```

**消息队列中的消息一旦被消费组里的一个消费者读取了，就不能再被该消费组内的其他消费者读取了，即同一个消费组里的消费者不能消费同一条消息**。

比如说，我们执行完刚才的 XREADGROUP 命令后，再执行一次同样的命令，此时读到的就是空值了：

```shell
> XREADGROUP GROUP group1 consumer1 STREAMS mymq >
(nil)
```

但是，**不同消费组的消费者可以消费同一条消息（但是有前提条件，创建消息组的时候，不同消费组指定了相同位置开始读取消息）**。

比如说，刚才 group1 消费组里的 consumer1 消费者消费了一条 id 为 1654254953808-0 的消息，现在用 group2 消费组里的 consumer1 消费者消费消息：

```shell
> XREADGROUP GROUP group2 consumer1 STREAMS mymq >
1) 1) "mymq"
   2) 1) 1) "1654254953808-0"
         2) 1) "name"
            2) "xiaolin"
```

因为我创建两组的消费组都是从第一条消息开始读取，所以可以看到第二组的消费者依然可以消费 id 为 1654254953808-0 的这一条消息。因此，不同的消费组的消费者可以消费同一条消息。

**使用消费组的目的是让组内的多个消费者共同分担读取消息**，所以，我们通常会让每个消费者读取部分消息，从而实现消息读取负载在多个消费者间是均衡分布的。

例如，我们执行下列命令，让 group2 中的 consumer1、2、3 各自读取一条消息。

```shell
# 让 group2 中的 consumer1 从 mymq 消息队列中消费一条消息
> XREADGROUP GROUP group2 consumer1 COUNT 1 STREAMS mymq >
1) 1) "mymq"
   2) 1) 1) "1654254953808-0"
         2) 1) "name"
            2) "xiaolin"
# 让 group2 中的 consumer2 从 mymq 消息队列中消费一条消息
> XREADGROUP GROUP group2 consumer2 COUNT 1 STREAMS mymq >
1) 1) "mymq"
   2) 1) 1) "1654256265584-0"
         2) 1) "name"
            2) "xiaolincoding"
# 让 group2 中的 consumer3 从 mymq 消息队列中消费一条消息
> XREADGROUP GROUP group2 consumer3 COUNT 1 STREAMS mymq >
1) 1) "mymq"
   2) 1) 1) "1654256271337-0"
         2) 1) "name"
            2) "Tom"
```

> 基于 Stream 实现的消息队列，如何保证消费者在发生故障或宕机再次重启后，仍然可以读取未处理完的消息？

Streams 会**自动使用内部队列（也称为 PENDING List）留存消费组里每个消费者读取的消息，直到消费者使用 XACK 命令通知 Streams“消息已经处理完成**”。

消费确认增加了消息的可靠性，一般在业务处理完成之后，需要执行 XACK 命令确认消息已经被消费完成，整个流程的执行如下图所示：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/数据类型/消息确认.png)

如果消费者没有成功处理消息，它就不会给 Streams 发送 XACK 命令，消息仍然会留存。此时，**消费者可以在重启后，用 XPENDING 命令查看已读取、但尚未确认处理完成的消息**。

例如，我们来查看一下 group2 中各个消费者已读取、但尚未确认的消息个数，命令如下：

```shell
127.0.0.1:6379> XPENDING mymq group2
1) (integer) 3
2) "1654254953808-0"  # 表示 group2 中所有消费者读取的消息最小 ID
3) "1654256271337-0"  # 表示 group2 中所有消费者读取的消息最大 ID
4) 1) 1) "consumer1"
      2) "1"
   2) 1) "consumer2"
      2) "1"
   3) 1) "consumer3"
      2) "1"
```

如果想查看某个消费者具体读取了哪些数据，可以执行下面的命令：

```shell
# 查看 group2 里 consumer2 已从 mymq 消息队列中读取了哪些消息
> XPENDING mymq group2 - + 10 consumer2
1) 1) "1654256265584-0"
   2) "consumer2"
   3) (integer) 410700
   4) (integer) 1
```

可以看到，consumer2 已读取的消息的 ID 是 1654256265584-0。

**一旦消息 1654256265584-0 被 consumer2 处理了，consumer2 就可以使用 XACK 命令通知 Streams，然后这条消息就会被删除**。

```shell
> XACK mymq group2 1654256265584-0
(integer) 1
```

当我们再使用 XPENDING 命令查看时，就可以看到，consumer2 已经没有已读取、但尚未确认处理的消息了。

```shell
> XPENDING mymq group2 - + 10 consumer2
(empty array)
```

好了，基于 Stream 实现的消息队列就说到这里了，小结一下：

- 消息保序：XADD/XREAD
- 阻塞读取：XREAD block
- 重复消息处理：Stream 在使用 XADD 命令，会自动生成全局唯一 ID；
- 消息可靠性：内部使用 PENDING List 自动保存消息，使用 XPENDING 命令查看消费组已经读取但是未被确认的消息，消费者使用 XACK 确认消息；
- 支持消费组形式消费数据

> Redis 基于 Stream 消息队列与专业的消息队列有哪些差距？

一个专业的消息队列，必须要做到两大块：

- 消息不丢。
- 消息可堆积。

*1、Redis Stream 消息会丢失吗？*

使用一个消息队列，其实就分为三大块：**生产者、队列中间件、消费者**，所以要保证消息就是保证三个环节都不能丢失数据。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/数据类型/消息队列三个阶段.png)

Redis Stream 消息队列能不能保证三个环节都不丢失数据？

- Redis 生产者会不会丢消息？**生产者会不会丢消息，取决于生产者对于异常情况的处理是否合理**。 从消息被生产出来，然后提交给 MQ 的过程中，只要能正常收到 （ MQ 中间件） 的 ack 确认响应，就表示发送成功，所以只要处理好返回值和异常，如果返回异常则进行消息重发，那么这个阶段是不会出现消息丢失的。

- **Redis 消费者会不会丢消息？不会**，因为 Stream （ MQ 中间件）会自动使用内部队列（也称为 PENDING List）留存消费组里每个消费者读取的消息，但是未被确认的消息。消费者可以在重启后，用 XPENDING 命令查看已读取、但尚未确认处理完成的消息。等到消费者执行完业务逻辑后，再发送消费确认 XACK 命令，也能保证消息的不丢失。

- Redis 消息中间件会不会丢消息？

  会

  ，**Redis 在以下 2 个场景下，都会导致数据丢失**：

  - **AOF 持久化配置为每秒写盘**，但这个写盘过程是异步的，**Redis 宕机时会存在数据丢失的可能**
  - **主从复制也是异步**的，[主从切换时，也存在丢失数据的可能 (opens new window)](https://xiaolincoding.com/redis/cluster/master_slave_replication.html#redis-主从切换如何减少数据丢失)。

可以看到，Redis 在队列中间件环节无法保证消息不丢。像 RabbitMQ 或 Kafka 这类专业的队列中间件，在使用时是部署一个集群，生产者在发布消息时，队列中间件通常会写「多个节点」，也就是有多个副本，这样一来，即便其中一个节点挂了，也能保证集群的数据不丢失。

*2、Redis Stream 消息可堆积吗？*

Redis 的数据都**存储在内存中，这就意味着一旦发生消息积压，则会导致 Redis 的内存持续增长**，如果超过机器内存上限，就会面临被 OOM 的风险。

所以 Redis 的 Stream 提供了**可以指定队列最大长度**的功能，就是为了避免这种情况发生。

**当指定队列最大长度时，队列长度超过上限后，旧消息会被删除，只保留固定长度的新消息**。这么来看，Stream 在消息积压时，如果指定了最大长度，还是有可能丢失消息的。

但 Kafka、RabbitMQ 专业的消息队列它们的数据都是存储在磁盘上，当消息积压时，无非就是多占用一些磁盘空间。

因此，把 Redis 当作队列来使用时，会面临的 2 个问题：

- Redis 本身可能会丢数据；
- 面对消息挤压，内存资源会紧张；

所以，能不能将 Redis 作为消息队列来使用，关键看你的业务场景：

- 如果你的业**务场景足够简单，对于数据丢失不敏感，而且消息积压概率比较小的情况下**，把 Redis 当作队列是完全可以的。
- 如果你的业务有海量消息，消息积压的概率比较大，并且不能接受数据丢失，那么还是用专业的消息队列中间件吧。

> 补充：Redis 发布/订阅机制为什么不可以作为消息队列？

发布订阅机制存在以下缺点，都是跟丢失数据有关：

1. 发布/订阅机制没有基于任何数据类型实现，所以**不具备「数据持久化」的能力**，也就是发布/订阅机制的相关操作，不会写入到 RDB 和 AOF 中，当 Redis 宕机重启，发布/订阅机制的数据也会全部丢失。
2. **发布订阅模式是“发后既忘”的工作模式**，如果有订阅者离线重连之后不能消费之前的历史消息。
3. 当消费端有一定的消息积压时，也就是生产者发送的消息，消费者消费不过来时，如果超过 32M 或者是 60s 内持续保持在 8M 以上，消费端会被强行断开，这个参数是在配置文件中设置的，默认值是 `client-output-buffer-limit pubsub 32mb 8mb 60`。

所以，**发布/订阅机制只适合即使通讯**的场景，比如[构建哨兵集群 (opens new window)](https://xiaolincoding.com/redis/cluster/sentinel.html#哨兵集群是如何组成的)的场景采用了发布/订阅机制。

### 总结

Redis 常见的五种数据类型：**String（字符串），Hash（哈希），List（列表），Set（集合）及 Zset(sorted set：有序集合)**。

这五种数据类型都由多种数据结构实现的，主要是出于时间和空间的考虑，当数据量小的时候使用更简单的数据结构，有利于节省内存，提高性能。

这五种数据类型与底层数据结构对应关系图如下，左边是 Redis 3.0版本的，也就是《Redis 设计与实现》这本书讲解的版本，现在看还是有点过时了，右边是现在 Github 最新的 Redis 代码的。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/9fa26a74965efbf0f56b707a03bb9b7f.png)

可以看到，Redis 数据类型的底层数据结构随着版本的更新也有所不同，比如：

- 在 Redis 3.0 版本中 List 对象的底层数据结构由「双向链表」或「压缩表列表」实现，但是在 3.2 版本之后，List 数据类型底层数据结构是由 quicklist 实现的；
- 在最新的 Redis 代码中，压缩列表数据结构已经废弃了，交由 listpack 数据结构来实现了。

Redis 五种数据类型的应用场景：

- **String 类型的应用场景：缓存对象、常规计数、分布式锁、共享session信息等。**
- **List 类型的应用场景：消息队列（有两个问题：1. 生产者需要自行实现全局唯一 ID；2. 不能以消费组形式消费数据）等**。
- **Hash 类型：缓存对象、购物车**等。
- **Set 类型：聚合计算（并集、交集、差集）场景，比如点赞、共同关注、抽奖活动**等。
- **Zset 类型：排序场景，比如排行榜、电话和姓名排序**等。

Redis 后续版本又支持四种数据类型，它们的应用场景如下：

- **BitMap（2.2 版新增）：二值状态统计的场景，比如签到、判断用户登陆状态、连续签到用户总数**等；
- **HyperLogLog（2.8 版新增）：海量数据基数统计的场景，比如百万级网页 UV 计数**等；
- **GEO（3.2 版新增）：存储地理位置信息的场景，比如滴滴叫车**；
- **Stream（5.0 版新增）：消息队列，相比于基于 List 类型实现的消息队列**，有这两个特有的特性：自动生成全局唯一消息ID，支持以消费组形式消费数据。

针对 Redis 是否适合做消息队列，关键看你的业务场景：

- 如果你的业务场景足够简单，对于数据丢失不敏感，而且消息积压概率比较小的情况下，把 Redis 当作队列是完全可以的。
- 如果你的业务有海量消息，消息积压的概率比较大，并且不能接受数据丢失，那么还是用专业的消息队列中间件吧。

## 数据结构

**Redis 数据结构并不是指 String（字符串）对象、List（列表）对象、Hash（哈希）对象、Set（集合）对象和 Zset（有序集合）对象，因为这些是 Redis 键值对中值的数据类型，也就是数据的保存形式，这些对象的底层实现的方式就用到了数据结构**。

![img](https://img-blog.csdnimg.cn/img_convert/9fa26a74965efbf0f56b707a03bb9b7f.png)

可以看到，Redis 数据类型的底层数据结构随着版本的更新也有所不同，比如：

- 在 Redis 3.0 版本中 List 对象的底层数据结构由「双向链表」或「压缩表列表」实现，但是在 3.2 版本之后，List 数据类型底层数据结构是由 quicklist 实现的；
- 在最新的 Redis 代码（还未发布正式版本）中，压缩列表数据结构已经废弃了，交由 listpack 数据结构来实现了。

**共有 9 种数据结构：SDS、双向链表、压缩列表、哈希表、跳表、整数集合、quicklist、listpack。**

不多 BB 了，直接发车！

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/a9c3e7dc4ac79363d8eb8eb2290a58e6.png)

### 键值对数据库是怎么实现的？

在开始讲数据结构之前，先给介绍下 Redis 是怎样实现键值对（key-value）数据库的。

Redis 的键值对中的 key 就是字符串对象，而 **value 可以是字符串对象，也可以是集合数据类型的对象**，比如 List 对象、Hash 对象、Set 对象和 Zset 对象。

举个例子，我这里列出几种 Redis 新增键值对的命令：

```text
> SET name "xiaolincoding"
OK

> HSET person name "xiaolincoding" age 18
0

> RPUSH stu "xiaolin" "xiaomei"
(integer) 4
```

这些命令代表着：

- 第一条命令：name 是一个**字符串键**，因为键的**值是一个字符串对象**；
- 第二条命令：person 是一个**哈希表键**，因为键的**值是一个包含两个键值对的哈希表对象**；
- 第三条命令：stu 是一个**列表键**，因为键的**值是一个包含两个元素的列表对象**；

#### 这些键值对是如何保存在 Redis 中的呢？

Redis 是**使用了一个「哈希表」保存所有键值对**，哈希表的最大好处就是让我们可以用 O(1) 的时间复杂度来快速查找到键值对。哈希表其实就是一个数组，数组中的元素叫做哈希桶。

#### Redis 的哈希桶是怎么保存键值对数据的呢？

哈希桶存放的是指向键值对数据的指针（dictEntry*），这样通过指针就能找到键值对数据，然后因为键值对的值可以保存字符串对象和集合数据类型的对象，所以键值对的数据结构中并不是直接保存值本身，而是保存了 void * key 和 void * value 指针，分别指向了实际的键对象和值对象，这样一来，即使值是集合数据，也可以通过 void * value 指针找到。

我这里画了一张 Redis 保存键值对所涉及到的数据结构。

![img](https://img-blog.csdnimg.cn/img_convert/f302fce6c92c0682024f47bf7579b44c.png)

![在这里插入图片描述](https://img-bc.icode.best/e934d45667994807befb2fdd96d34827.png)

这些数据结构的内部细节，我先不展开讲，后面在讲哈希表数据结构的时候，在详细的说说，因为用到的数据结构是一样的。这里先大概说下图中涉及到的数据结构的名字和用途：

- redisDb 结构，**表示 Redis 数据库的结构**，结构体里存放了指向了 dict 结构的指针；

  > *RedisDb是Redis的底层数据结构的开始，里面存放着Redis的数据，一般默认有16个，这个是可以配置的，Redis的数据是以字典的形式在底层展示的。*

- dict 结构，结构体里存放了 2 个哈希表，**正常情况下都是用「哈希表1」**，「哈希表2」只有在 rehash 的时候才用，具体什么是 rehash，我在本文的哈希表数据结构会讲；

  > *里面最重要的有两个变量*
  > 1、type：*标识整个hashTable的数据类型是什么,(string, list, hashmap. set, zset…..)。*
  > 2、dictht ht[2]：*表示有两个hashTable，目的是用来扩容，**ht[0]是当前数据的存放变量**，**ht[1]是将来需要扩容时**的新hashTable*

- ditctht 结构，表示哈希表的结构，结构里存放了哈希表数组，数组中的**每个元素都是指向一个哈希表节点结构**（dictEntry）的指针；

  > *dictht里面的四个参数*
  > 1、dictEntry：*这个就是具体**表示Key-Value形式的数据结构**。*
  > 2、size：*表示当前HashTable的大小。*
  > 3、sizemark：*永远都是size-1，为了到时候计算hashcode时可以将 % 转换成 & 。*
  > 4、used：*表示hashtable当前已经使用的容量。*

- dictEntry 结构，表示哈希表节点的结构，结构里存放了 **void * key 和 void * value 指针， *key 指向的是 String 对象，而 \*value 则可以指向 String 对象，也可以指向集合类型的对象，比如 List 对象、Hash 对象、Set 对象和 Zset 对象**。

  > *dictEntry里面最重要的三个参数*
  > 1、key：*这个就是具体表示Key-Value形式的数据结构的key，底层就是SDS。*
  > 2、value：*这个就是具体表示Key-Value形式的数据结构的value数据结构。*
  > 3、next：*指向下一个节点的指针。*

- value 结构，主要表示 value 的结构

  > *value里面最重要的五个参数*
  > 1、type：*代表当前数据的类型是什么，比如string、list、hash、set、zset等。*
  > 2、encoding：*表示当前数据在内存中的数据结构是什么，比如raw、embstr、int、ziplist等。*
  > 3、lru：*表示过期算法是哪一种。*
  > 4、refcount：*C语言得程序员自己管理内存，不像Java有垃圾收集器，这个就是引用计数器，来管理自己的内存，防止内存溢出。*
  > 3、*ptr：*指向真正得数据，如果是整型值等，则直接存储，如果是很长的字符串（这个到底有多长，和CPU的缓存行有关，在），则存放指向数据的地址。*

特别说明下，void * key 和 void * value 指针指向的是 **Redis 对象（即上文的value）**，Redis 中的每个对象都由 redisObject 结构表示，如下图：

![img](https://img-blog.csdnimg.cn/img_convert/58d3987af2af868dca965193fb27c464.png)

对象结构里包含的成员变量：

- type，标识该对象是什么类型的对象（String 对象、 List 对象、Hash 对象、Set 对象和 Zset 对象）；
- encoding，标识该对象使用了哪种底层的数据结构；
- **ptr，指向底层数据结构的指针**。

我画了一张 Redis 键值对数据库的全景图，你就能清晰知道 Redis 对象和数据结构的关系了：

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/3c386666e4e7638a07b230ba14b400fe.png)

接下里，就好好聊一下底层数据结构！

### SDS

字符串在 Redis 中是很常用的，键值对中的**键是字符串类型**，值有时也是字符串类型。

Redis 是用 C 语言实现的，但是它没有直接使用 C 语言的 char* 字符数组来实现字符串，而是自己封装了一个名为**简单动态字符串**（simple dynamic string，SDS） 的数据结构来表示字符串，也就是 Redis 的 String 数据类型的底层数据结构是 SDS。

既然 Redis 设计了 SDS 结构来表示字符串，肯定是 C 语言的 char* 字符数组存在一些缺陷。

要了解这一点，得先来看看 char* 字符数组的结构。

#### C 语言字符串的缺陷

C 语言的**字符串其实就是一个字符数组**，即数组中每个元素是字符串中的一个字符。

比如，下图就是字符串“xiaolin”的 char* 字符数组的结构：

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/376128646c75a893ad47914858fa2131.png)

没学过 C 语言的同学，可能会好奇为什么最后一个字符是“\0”？

在 C 语言里，对字符串操作时，char * 指针只是指向字符数组的起始位置，而**字符数组的结尾位置就用“\0”表示，意思是指字符串的结束**。

因此，C 语言标准库中的字符串操作函数就通过判断字符是不是 “\0” 来决定要不要停止操作，如果当前字符不是 “\0” ，说明字符串还没结束，可以继续操作，如果当前字符是 “\0” 是则说明字符串结束了，就要停止操作。

举个例子，C 语言获取字符串长度的函数 `strlen`，就是通过字符数组中的每一个字符，并进行计数，等遇到字符为 “\0” 后，就会停止遍历，然后返回已经统计到的字符个数，即为字符串长度。下图显示了 strlen 函数的执行流程：

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/bcf6bde3b647bdc343efcbc1a8f10579.png)

很明显，**C 语言获取字符串长度的时间复杂度是 O（N）（\*这是一个可以改进的地方\***）

C 语言字符串用 “\0” 字符作为结尾标记有个缺陷。假设有个字符串中有个 “\0” 字符，这时在操作这个字符串时就会**提早结束**，比如 “xiao\0lin” 字符串，计算字符串长度的时候则会是 4，如下图：

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/6286480eb1840a8930e18fd215d82565.png)

因此，除了字符串的末尾之外，**字符串里面不能含有 “\0” 字符**，否则最先被程序读入的 “\0” 字符将被误认为是字符串结尾，这个限制使得 C 语言的字符串只能保存文本数据，**不能保存像图片、音频、视频文化这样的二进制数据（\*这也是一个可以改进的地方\*）**

另外， C 语言标准库中字符串的操作函数是很不安全的，对程序员很不友好，稍微一不注意，就会导致缓冲区溢出。

举个例子，strcat 函数是可以将两个字符串拼接在一起。

```c
//将 src 字符串拼接到 dest 字符串后面
char *strcat(char *dest, const char* src);
```

**C 语言的字符串是不会记录自身的缓冲区大小的**，所以 strcat 函数假定程序员在执行这个函数时，已经为 dest 分配了足够多的内存，可以容纳 src 字符串中的所有内容，而**一旦这个假定不成立，就会发生缓冲区溢出将可能会造成程序运行终止，（\*这是一个可以改进的地方\***）。

而且，strcat 函数和 strlen 函数类似，时间复杂度也很高，也都需要先通过遍历字符串才能得到目标字符串的末尾。然后对于 strcat 函数来说，还要再遍历源字符串才能完成追加，**对字符串的操作效率不高**。

好了， 通过以上的分析，我们可以得知 C 语言的字符串不足之处以及可以改进的地方：

- 获取字符串长度的**时间复杂度为 O（N）**；
- 字符串的结尾是以 “\0” 字符标识，**字符串里面不能包含有 “\0” 字符**，因此不能保存二进制数据；
- 字符串操作函数不高效且不安全，比如有**缓冲区溢出的风险**，有可能会造成程序运行终止；

Redis 实现的 SDS 的结构就把上面这些问题解决了，接下来我们一起看看 Redis 是如何解决的。

#### SDS 结构设计

下图就是 Redis 5.0 的 SDS 的数据结构：

![img](https://img-blog.csdnimg.cn/img_convert/516738c4058cdf9109e40a7812ef4239.png)

结构中的每个成员变量分别介绍下：

- **len，记录了字符串长度**。这样获取字符串长度的时候，只需要返回这个成员变量值就行，时间复杂度只需要 O（1）。
- **alloc，分配给字符数组的空间长度**。这样在修改字符串的时候，可以通过 `alloc - len` 计算出剩余的空间大小，可以用来判断空间是否满足修改需求，如果不满足的话，就会自动将 SDS 的空间扩展至执行修改所需的大小，然后才执行实际的修改操作，所以使用 SDS 既不需要手动修改 SDS 的空间大小，也不会出现前面所说的缓冲区溢出的问题。
- **flags，用来表示不同类型的 SDS**。一共设计了 5 种类型，分别是 sdshdr5、sdshdr8、sdshdr16、sdshdr32 和 sdshdr64，后面在说明区别之处。
- **buf[]，字符数组，用来保存实际数据**。不仅可以保存字符串，也可以保存二进制数据。

总的来说，Redis 的 SDS 结构在原本字符数组之上，增加了三个元数据：len、alloc、flags，用来解决 C 语言字符串的缺陷。

##### O（1）复杂度获取字符串长度

C 语言的字符串长度获取 strlen 函数，需要通过遍历的方式来统计字符串长度，时间复杂度是 O（N）。

而 Redis 的 SDS 结构因为加入了 len 成员变量，那么**获取字符串长度的时候，直接返回这个成员变量的值就行，所以复杂度只有 O（1）**。

##### 二进制安全

因为 SDS 不需要用 “\0” 字符来标识字符串结尾了，而是**有个专门的 len 成员变量来记录长度，所以可存储包含 “\0” 的数据**。但是 SDS 为了兼容部分 C 语言标准库的函数， SDS 字符串结尾还是会加上 “\0” 字符。

因此， **SDS 的 API 都是以处理二进制的方式来处理 SDS 存放在 buf[] 里的数据**，程序不会对其中的数据做任何限制，数据写入的时候时什么样的，它被读取时就是什么样的。

通过使用二进制安全的 SDS，而不是 C 字符串，使得 Redis 不仅可以保存文本数据，也可以保存任意格式的二进制数据。

##### 不会发生缓冲区溢出

C 语言的字符串标准库提供的字符串操作函数，大多数（比如 strcat 追加字符串函数）都是不安全的，因为这些函数把缓冲区大小是否满足操作需求的工作交由开发者来保证，程序内部并不会判断缓冲区大小是否足够用，当发生了缓冲区溢出就有可能造成程序异常结束。

所以，Redis 的 SDS 结构里引入了 alloc 和 len 成员变量，这样 SDS API 通过 `alloc - len` 计算，可以算出剩余可用的空间大小，这样在对字符串做修改操作的时候，就可以由程序内部判断缓冲区大小是否足够用。

而且，**当判断出缓冲区大小不够用时，Redis 会自动将扩大 SDS 的空间大小（小于 1MB 翻倍扩容，大于 1MB 按 1MB 扩容）**，以满足修改所需的大小。

在扩展 SDS 空间之前，SDS API 会优先检查未使用空间是否足够，如果不够的话，API 不仅会为 SDS 分配修改所必须要的空间，还会给 SDS 分配额外的「未使用空间」。

这样的好处是，下次在操作 SDS 时，如果 SDS 空间够的话，API 就会直接使用「未使用空间」，而无须执行内存分配，**有效的减少内存分配次数**。

所以，使用 SDS 即不需要手动修改 SDS 的空间大小，也不会出现缓冲区溢出的问题。

##### 节省内存空间

SDS 结构中有个 flags 成员变量，表示的是 SDS 类型。

Redis 一共设计了 5 种类型，分别是 sdshdr5、sdshdr8、sdshdr16、sdshdr32 和 sdshdr64。

这 5 种类型的主要**区别就在于，它们数据结构中的 len 和 alloc 成员变量的数据类型不同**。

比如 sdshdr16 和 sdshdr32 这两个类型，它们的定义分别如下：

```c
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len;
    uint16_t alloc; 
    unsigned char flags; 
    char buf[];
};


struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len;
    uint32_t alloc; 
    unsigned char flags;
    char buf[];
};
```

可以看到：

- sdshdr16 类型的 len 和 alloc 的数据类型都是 uint16_t，表示**字符数组长度和分配空间大小不能超过 2 的 16 次方。**
- sdshdr32 则都是 uint32_t，表示表示字符数组长度和分配空间大小不能超过 2 的 32 次方。

**之所以 SDS 设计不同类型的结构体，是为了能灵活保存不同大小的字符串，从而有效节省内存空间**。比如，在保存小字符串时，结构头占用空间也比较少。

除了设计不同类型的结构体，Redis 在编程上还**使用了专门的编译优化来节省内存空间**，即在 struct 声明了 `__attribute__ ((packed))` ，它的作用是：**告诉编译器取消结构体在编译过程中的优化对齐，按照实际占用字节数进行对齐**。

比如，sdshdr16 类型的 SDS，默认情况下，编译器会按照 2 字节对齐的方式给变量分配内存，这意味着，即使一个变量的大小不到 2 个字节，编译器也会给它分配 2 个字节。

举个例子，假设下面这个结构体，它有两个成员变量，类型分别是 char 和 int，如下所示：

```c
#include <stdio.h>

struct test1 {
    char a;
    int b;
 } test1;
 
int main() {
     printf("%lu\n", sizeof(test1));
     return 0;
}
```

大家猜猜这个结构体大小是多少？我先直接说答案，这个结构体大小计算出来是 8。

![img](https://img-blog.csdnimg.cn/img_convert/35820959e8cf4376391c427ed7f81495.png)

这是因为默认情况下，编译器是使用「字节对齐」的方式分配内存，虽然 char 类型只占一个字节，但是由于成员变量里有 int 类型，它占用了 4 个字节，所以在成员变量为 char 类型分配内存时，会分配 4 个字节，其中这多余的 3 个字节是为了字节对齐而分配的，相当于有 3 个字节被浪费掉了。

如果不想编译器使用字节对齐的方式进行分配内存，可以采用了 `__attribute__ ((packed))` 属性定义结构体，这样一来，结构体实际占用多少内存空间，编译器就分配多少空间。

比如，我用 `__attribute__ ((packed))` 属性定义下面的结构体 ，同样包含 char 和 int 两个类型的成员变量，代码如下所示：

```c
#include <stdio.h>

struct __attribute__((packed)) test2  {
    char a;
    int b;
 } test2;
 
int main() {
     printf("%lu\n", sizeof(test2));
     return 0;
}
```

这时打印的结果是 5（1 个字节 char + 4 字节 int）。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/47e6c8fbc17fd6c89bdfcb5eedaaacff.png)

可以看得出，这是按照实际占用字节数进行分配内存的，这样可以节省内存空间。

------

### 链表

大家最熟悉的数据结构除了数组之外，我相信就是链表了。

Redis 的 List 对象的底层实现之一就是链表。C 语言本身没有链表这个数据结构的，所以 Redis 自己设计了一个链表数据结构。

#### 链表节点结构设计

先来看看「链表节点」结构的样子：

```c
typedef struct listNode {
    //前置节点
    struct listNode *prev;
    //后置节点
    struct listNode *next;
    //节点的值
    void *value;
} listNode;
```

有前置节点和后置节点，可以看的出，这个是一个双向链表。

![img](https://img-blog.csdnimg.cn/img_convert/4fecbf7f63c73ec284a4821e0bfe2843.png)

#### 链表结构设计

不过，Redis 在 listNode 结构体基础上又封装了 list 这个数据结构，这样操作起来会更方便，链表结构如下：

```c
typedef struct list {
    //链表头节点
    listNode *head;
    //链表尾节点
    listNode *tail;
    //节点值复制函数
    void *(*dup)(void *ptr);
    //节点值释放函数
    void (*free)(void *ptr);
    //节点值比较函数
    int (*match)(void *ptr, void *key);
    //链表节点数量
    unsigned long len;
} list;
```

list 结构为链表提供了链表头指针 head、链表尾节点 tail、链表节点数量 len、以及可以自定义实现的 dup、free、match 函数。

举个例子，下面是由 list 结构和 3 个 listNode 结构组成的链表。

![img](https://img-blog.csdnimg.cn/img_convert/cadf797496816eb343a19c2451437f1e.png)

#### 链表的优势与缺陷

Redis 的链表实现优点如下：

- listNode 链表节点的结构里带有 prev 和 next 指针，**获取某个节点的前置节点或后置节点的时间复杂度只需O(1)，而且这两个指针都可以指向 NULL，所以链表是无环链表**；
- list 结构因为提供了表头指针 head 和表尾节点 tail，所以**获取链表的表头节点和表尾节点的时间复杂度只需O(1)**；
- list 结构因为提供了链表节点数量 len，所以**获取链表中的节点数量的时间复杂度只需O(1)**；
- listNode 链表节使用 void* 指针保存节点值，并且可以通过 list 结构的 dup、free、match 函数指针为节点设置该节点类型特定的函数，因此**链表节点可以保存各种不同类型的值**；

链表的缺陷也是有的：

- 链表每个节点之间的内存都是不连续的，意味着**无法很好利用 CPU 缓存**。能很好利用 CPU 缓存的数据结构就是数组，因为数组的内存是连续的，这样就可以充分利用 CPU 缓存来加速访问。
- 还有一点，保存一个链表节点的值都需要一个链表节点结构头的分配，**内存开销较大**。

因此，Redis 3.0 的 List 对象在数据量比较少的情况下，会采用「压缩列表」作为底层数据结构的实现，它的优势是节省内存空间，并且是内存紧凑型的数据结构。

不过，压缩列表存在性能问题（具体什么问题，下面会说），所以 Redis 在 3.2 版本设计了新的数据结构 quicklist，并将 List 对象的底层数据结构改由 quicklist 实现。

然后在 Redis 5.0 设计了新的数据结构 listpack，沿用了压缩列表紧凑型的内存布局，最终在最新的 Redis 版本，将 Hash 对象和 Zset 对象的底层数据结构实现之一的压缩列表，替换成由 listpack 实现。

------

### 压缩列表

压缩列表的最大特点，就是它被设计成**一种内存紧凑型的数据结构**，占用一块连续的内存空间，不仅可以利用 CPU 缓存，而且会**针对不同长度的数据，进行相应编码**，这种方法可以有效地节省内存开销。

但是，压缩列表的缺陷也是有的：

- **不能保存过多的元素**，否则查询效率就会降低；
- **新增或修改某个元素时，压缩列表占用的内存空间需要重新分配**，甚至可能引发连锁更新的问题。

因此，Redis 对象（List 对象、Hash 对象、Zset 对象）包含的元素数量较少，或者元素值不大的情况才会使用压缩列表作为底层数据结构。

接下来，就跟大家详细聊下压缩列表。

#### 压缩列表结构设计

压缩列表是 Redis 为了节约内存而开发的，它是**由连续内存块组成的顺序型数据结构**，有点类似于数组。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/ab0b44f557f8b5bc7acb3a53d43ebfcb.png)

压缩列表在表头有三个字段：

- ***zlbytes***，记录**整个压缩列表**占用对内存字节数；
- ***zltail***，记录压缩列表「**尾部**」节点距离**起始地址**由多少字节，也就是列表尾的偏移量；
- ***zllen***，记录压缩列表包含的**节点数量**；
- ***zlend***，标记压缩列表的**结束点**，固定值 0xFF（十进制255）。

在压缩列表中，如果我们要查找定位第一个元素和最后一个元素，可以通过表头三个字段的长度直接定位，复杂度是 O(1)。而**查找其他元素时，就没有这么高效了，只能逐个查找，此时的复杂度就是 O(N) 了，因此压缩列表不适合保存过多的元素**。

另外，压缩列表节点（entry）的构成如下：

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/a3b1f6235cf0587115b21312fe60289c.png)

压缩列表节点包含三部分内容：

- ***prevlen***，记录了**「前一个节点」的长度**；
- ***encoding***，记录了当前节点实际数据的类型以及长度；
- ***data***，记录了当前节点的实际数据；

当我们往压缩列表中插入数据时，压缩列表就会根据数据是字符串还是整数，以及数据的大小，会使用不同空间大小的 prevlen 和 encoding 这两个元素里保存的信息，**这种根据数据大小和类型进行不同的空间大小分配的设计思想，正是 Redis 为了节省内存而采用的**。

分别说下，prevlen 和 encoding 是如何根据数据的大小和类型来进行不同的空间大小分配。

压缩列表里的每个节点中的 prevlen 属性都记录了「前一个节点的长度」，而且 prevlen 属性的空间大小跟前一个节点长度值有关，比如：

- 如果**前一个节点的长度小于 254 字节**，那么 prevlen 属性需要用 **1 字节的空间**来保存这个长度值；
- 如果**前一个节点的长度大于等于 254 字节**，那么 prevlen 属性需要用 **5 字节的空间**来保存这个长度值；

encoding 属性的空间大小跟数据是字符串还是整数，以及字符串的长度有关：

- 如果**当前节点的数据是整数**，则 encoding 会使用 **1 字节的空间**进行编码。
- 如果**当前节点的数据是字符串，根据字符串的长度大小**，encoding 会使用 **1 字节/2字节/5字节的空间**进行编码。

#### 连锁更新

压缩列表除了查找复杂度高的问题，还有一个问题。

压缩列表新增某个元素或修改某个元素时，如果空间不不够，压缩列表占用的内存空间就需要重新分配。而**当新插入的元素较大时，可能会导致后续元素的 prevlen 占用空间都发生变化**，从而引起「连锁更新」问题，导致每个元素的空间都要重新分配，造成访问压缩列表性能的下降。

前面提到，压缩列表节点的 prevlen 属性会根据前一个节点的长度进行不同的空间大小分配：

- 如果前一个**节点的长度小于 254 字节**，那么 prevlen 属性需要用 **1 字节的空间**来保存这个长度值；
- 如果前一个**节点的长度大于等于 254 字节**，那么 prevlen 属性需要用 **5 字节的空间**来保存这个长度值；

现在假设一个压缩列表中有多个连续的、长度在 250～253 之间的节点，如下图：

![img](https://img-blog.csdnimg.cn/img_convert/462c6a65531667f2bcf420953b0aded9.png)

因为这些节点长度值小于 254 字节，所以 prevlen 属性需要用 1 字节的空间来保存这个长度值。

这时，如果将一个长度大于等于 254 字节的新节点加入到压缩列表的表头节点，即新节点将成为 e1 的前置节点，如下图：

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/d1a6deff4672580609c99a5b06bf3429.png)

因为 e1 节点的 prevlen 属性只有 1 个字节大小，无法保存新节点的长度，此时就需要对压缩列表的空间重分配操作，并将 e1 节点的 prevlen 属性从原来的 1 字节大小扩展为 5 字节大小。

多米诺牌的效应就此开始。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/1f0e5ae7ab749078cadda5ba0ed98eac.png)

e1 原本的长度在 250～253 之间，因为刚才的扩展空间，此时 e1 的长度就大于等于 254 了，因此原本 e2 保存 e1 的 prevlen 属性也必须从 1 字节扩展至 5 字节大小。

正如扩展 e1 引发了对 e2 扩展一样，扩展 e2 也会引发对 e3 的扩展，而扩展 e3 又会引发对 e4 的扩展.... 一直持续到结尾。

**这种在特殊情况下产生的连续多次空间扩展操作就叫做「连锁更新」**，就像多米诺牌的效应一样，第一张牌倒下了，推动了第二张牌倒下；第二张牌倒下，又推动了第三张牌倒下....，

#### 压缩列表的缺陷

空间扩展操作也就是重新分配内存，因此**连锁更新一旦发生，就会导致压缩列表占用的内存空间要多次重新分配，这就会直接影响到压缩列表的访问性能**。

所以说，**虽然压缩列表紧凑型的内存布局能节省内存开销，但是如果保存的元素数量增加了，或是元素变大了，会导致内存重新分配，最糟糕的是会有「连锁更新」的问题**。

因此，**压缩列表只会用于保存的节点数量不多的场景**，只要节点数量足够小，即使发生连锁更新，也是能接受的。

虽说如此，Redis 针对压缩列表在设计上的不足，在后来的版本中，新增设计了两种数据结构：quicklist（Redis 3.2 引入） 和 listpack（Redis 5.0 引入）。这两种数据结构的设计目标，就是尽可能地保持压缩列表节省内存的优势，同时解决压缩列表的「连锁更新」的问题。

### 哈希表

哈希表是一种保存键值对（key-value）的数据结构。

哈希表中的每一个 key 都是独一无二的，程序可以根据 key 查找到与之关联的 value，或者通过 key 来更新 value，又或者根据 key 来删除整个 key-value等等。

在讲压缩列表的时候，提到过 Redis 的 Hash 对象的底层实现之一是压缩列表（最新 Redis 代码已将压缩列表替换成 listpack）。Hash 对象的另外一个底层实现就是哈希表。

哈希表优点在于，它**能以 O(1) 的复杂度快速查询数据**。怎么做到的呢？将 key 通过 Hash 函数的计算，就能定位数据在表中的位置，因为哈希表实际上是数组，所以可以通过索引值快速查询到数据。

但是存在的风险也是有，在哈希表大小固定的情况下，随着数据不断增多，那么**哈希冲突**的可能性也会越高。

解决哈希冲突的方式，有很多种。

**Redis 采用了「链式哈希」来解决哈希冲突**，在不扩容哈希表的前提下，将具有相同哈希值的数据串起来，形成链接起，以便这些数据在表中仍然可以被查询到。

接下来，详细说说哈希表。

#### 哈希表结构设计

Redis 的哈希表结构如下：

```c
typedef struct dictht {
    //哈希表数组
    dictEntry **table;
    //哈希表大小
    unsigned long size;  
    //哈希表大小掩码，用于计算索引值
    unsigned long sizemask;
    //该哈希表已有的节点数量
    unsigned long used;
} dictht;
```

可以看到，哈希表是一个数组（dictEntry **table），数组的每个元素是一个指向「哈希表节点（dictEntry）」的指针。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/dc495ffeaa3c3d8cb2e12129b3423118.png)

哈希表节点的结构如下：

```c
typedef struct dictEntry {
    //键值对中的键
    void *key;
  
    //键值对中的值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    //指向下一个哈希表节点，形成链表
    struct dictEntry *next;
} dictEntry;
```

dictEntry 结构里不仅包含指向键和值的指针，还包含了指向下一个哈希表节点的指针，这个指针可以将多个哈希值相同的键值对链接起来，以此来解决哈希冲突的问题，这就是链式哈希。

另外，这里还跟你提一下，dictEntry 结构里键值对中的值是一个「联合体 v」定义的，因此，**键值对中的值可以是一个指向实际值的指针，或者是一个无符号的 64 位整数或有符号的 64 位整数或double 类的值**。这么做的好处是可以节省内存空间，因为当「值」是整数或浮点数时，就可以将值的数据内嵌在 dictEntry 结构里，无需再用一个指针指向实际的值，从而节省了内存空间。

#### 哈希冲突

哈希表实际上是一个数组，数组里多每一个元素就是一个哈希桶。

当一个键值对的键经过 Hash 函数计算后得到哈希值，再将(哈希值 % 哈希表大小)取模计算，得到的结果值就是该 key-value 对应的数组元素位置，也就是第几个哈希桶。

> 什么是哈希冲突呢？

举个例子，有一个可以存放 8 个哈希桶的哈希表。key1 经过哈希函数计算后，再将「哈希值 % 8 」进行取模计算，结果值为 1，那么就对应哈希桶 1，类似的，key9 和 key10 分别对应哈希桶 1 和桶 6。

![img](https://img-blog.csdnimg.cn/img_convert/753724a072e77d139c926ecf1f049b29.png)

此时，key1 和 key9 对应到了相同的哈希桶中，这就发生了哈希冲突。

因此，**当有两个以上数量的 kay 被分配到了哈希表中同一个哈希桶上时，此时称这些 key 发生了冲突。**

#### 链式哈希

Redis 采用了「**链式哈希**」的方法来解决哈希冲突。

> 链式哈希是怎么实现的？

实现的方式就是每个哈希表节点都有一个 next 指针，用于指向下一个哈希表节点，因此多个哈希表节点可以用 next 指针构成一个单项链表，**被分配到同一个哈希桶上的多个节点可以用这个单向链表连接起来**，这样就解决了哈希冲突。

还是用前面的哈希冲突例子，key1 和 key9 经过哈希计算后，都落在同一个哈希桶，链式哈希的话，key1 就会通过 next 指针指向 key9，形成一个单向链表。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/675c23857a36b2dab26ed2e6a7b94b5d.png)

不过，链式哈希局限性也很明显，随着链表长度的增加，在查询这一位置上的数据的耗时就会增加，毕竟链表的查询的时间复杂度是 O(n)。

要想解决这一问题，就需要进行 rehash，也就是对哈希表的大小进行扩展。

接下来，看看 Redis 是如何实现的 rehash 的。

#### rehash

哈希表结构设计的这一小节，我给大家介绍了 Redis 使用 dictht 结构体表示哈希表。不过，在实际使用哈希表时，Redis 定义一个 dict 结构体，这个结构体里定义了**两个哈希表（ht[2]）**。

```c
typedef struct dict {
    …
    //两个Hash表，交替使用，用于rehash操作
    dictht ht[2]; 
    …
} dict;
```

之所以定义了 2 个哈希表，是因为进行 rehash 的时候，需要用上 2 个哈希表了。

![img](https://img-blog.csdnimg.cn/img_convert/2fedbc9cd4cb7236c302d695686dd478.png)

在**正常服务请求阶段，插入的数据，都会写入到「哈希表 1」**，此时的「哈希表 2 」 并没有被分配空间。

随着数据逐步增多，触发了 rehash 操作，这个过程分为三步：

- 给「哈希表 2」 分配空间，一般会比「哈希表 1」 **大 2 倍**；
- 将「哈希表 1 」的数据迁移到「哈希表 2」 中；
- 迁移完成后，「哈希表 1 」的空间会被释放，并把「哈希表 2」 设置为「哈希表 1」，然后在「哈希表 2」 新创建一个空白的哈希表，为下次 rehash 做准备。

为了方便你理解，我把 rehash 这三个过程画在了下面这张图：

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/cabce0ce7e320bc9d9b5bde947b6811b.png)

这个过程看起来简单，但是其实第二步很有问题，**如果「哈希表 1 」的数据量非常大，那么在迁移至「哈希表 2 」的时候，因为会涉及大量的数据拷贝，此时可能会对 Redis 造成阻塞，无法服务其他请求**。

#### 渐进式 rehash

为了避免 rehash 在数据迁移过程中，因拷贝数据的耗时，影响 Redis 性能的情况，所以 Redis 采用了**渐进式 rehash**，也就是将数据的迁移的工作不再是一次性迁移完成，而是分多次迁移。

渐进式 rehash 步骤如下：

- 给「哈希表 2」 分配空间；
- **在 rehash 进行期间，每次哈希表元素进行新增、删除、查找或者更新操作时，Redis 除了会执行对应的操作之外，还会顺序将「哈希表 1 」中索引位置上的所有 key-value 迁移到「哈希表 2」 上**；
- 随着处理客户端发起的哈希表操作请求数量越多，最终在某个时间点会把「哈希表 1 」的所有 key-value 迁移到「哈希表 2」，从而完成 rehash 操作。

这样就巧妙地**把一次性大量数据迁移工作的开销，分摊到了多次处理请求的过程中，避免了一次性 rehash 的耗时操作**。

在进行渐进式 rehash 的过程中，会有两个哈希表，所以在渐进式 rehash 进行期间，哈希表元素的删除、查找、更新等操作都会在这两个哈希表进行。

比如，**查找**一个 key 的值的话，**先会在「哈希表 1」 里面进行查找**，如果没找到，就会继续**到哈希表 2 里面进行找到**。

另外，在渐进式 rehash 进行期间，**新增一个 key-value 时，会被保存到「哈希表 2 」里面**，而「哈希表 1」 则不再进行任何添加操作，这样保证了「哈希表 1 」的 key-value 数量只会减少，随着 rehash 操作的完成，最终「哈希表 1 」就会变成空表。

#### rehash 触发条件

介绍了 rehash 那么多，还没说什么时情况下会触发 rehash 操作呢？

rehash 的触发条件跟**负载因子**（load factor） 有关系。

负载因子可以通过下面这个公式计算：

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/85f597f7851b90d6c78bb0d8e39690fc.png)

触发 rehash 操作的条件，主要有两个：

- **当负载因子大于等于 1 ，并且 Redis 没有在执行 bgsave 命令或者 bgrewiteaof 命令，也就是没有执行 RDB 快照或没有进行 AOF 重写的时候，就会进行 rehash 操作。**
- **当负载因子大于等于 5 时，此时说明哈希冲突非常严重了，不管有没有有在执行 RDB 快照或 AOF 重写，都会强制进行 rehash 操作。**

### 整数集合

**整数集合是 Set 对象的底层实现之一**。当一个 Set 对象只包含整数值元素，并且元素数量不大时，就会使用整数集这个数据结构作为底层实现。

#### 整数集合结构设计

整数集合本质上是一块连续内存空间，它的结构定义如下：

```c
typedef struct intset {
    //编码方式
    uint32_t encoding;
    //集合包含的元素数量
    uint32_t length;
    //保存元素的数组
    int8_t contents[];
} intset;
```

可以看到，保存元素的容器是一个 contents 数组，虽然 contents 被声明为 int8_t 类型的数组，但是实际上 contents 数组并不保存任何 int8_t 类型的元素，**contents 数组的真正类型取决于 intset 结构体里的 encoding 属性的值**。比如：

- 如果 encoding 属性值为 INTSET_ENC_INT16，那么 contents 就是一个 int16_t 类型的数组，数组中每一个元素的类型都是 int16_t；
- 如果 encoding 属性值为 INTSET_ENC_INT32，那么 contents 就是一个 int32_t 类型的数组，数组中每一个元素的类型都是 int32_t；
- 如果 encoding 属性值为 INTSET_ENC_INT64，那么 contents 就是一个 int64_t 类型的数组，数组中每一个元素的类型都是 int64_t；

不同类型的 contents 数组，意味着数组的大小也会不同。

#### 整数集合的升级操作

整数集合会有一个升级规则，就是**当我们将一个新元素加入到整数集合里面，如果新元素的类型（int32_t）比整数集合现有所有元素的类型（int16_t）都要长时，整数集合需要先进行升级**，也就是按新元素的类型（int32_t）扩展 contents 数组的空间大小，然后才能将新元素加入到整数集合里，当然升级的过程中，也要维持整数集合的有序性。

整数集合升级的过程不会重新分配一个新类型的数组，而是在原本的数组上扩展空间，然后在将每个元素按间隔类型大小分割，如果 encoding 属性值为 INTSET_ENC_INT16，则每个元素的间隔就是 16 位。

举个例子，假设有一个整数集合里有 3 个类型为 int16_t 的元素。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/5dbdfa7cfbdd1d12a4d9458c6c90d472.png)

现在，往这个整数集合中加入一个新元素 65535，这个新元素需要用 int32_t 类型来保存，所以整数集合要进行升级操作，首先需要为 contents 数组扩容，**在原本空间的大小之上再扩容多 80 位（4x32-3x16=80），这样就能保存下 4 个类型为 int32_t 的元素**。

![img](https://img-blog.csdnimg.cn/img_convert/e2e3e19fc934e70563fbdfde2af39a2b.png)

扩容完 contents 数组空间大小后，需要将之前的三个元素转换为 int32_t 类型，并将转换后的元素放置到正确的位上面，并且需要维持底层数组的有序性不变，整个转换过程如下：

![img](https://img-blog.csdnimg.cn/img_convert/e84b052381e240eeb8cc97d6b729968b.png)

> 整数集合升级有什么好处呢？

如果要让一个数组同时保存 int16_t、int32_t、int64_t 类型的元素，最简单做法就是直接使用 int64_t 类型的数组。不过这样的话，当如果元素都是 int16_t 类型的，就会造成内存浪费的情况。

整数集合升级就能避免这种情况，如果一直向整数集合添加 int16_t 类型的元素，那么整数集合的底层实现就一直是用 int16_t 类型的数组，只有在我们要将 int32_t 类型或 int64_t 类型的元素添加到集合时，才会对数组进行升级操作。

因此，整数集合升级的好处是**节省内存资源**。

> 整数集合支持降级操作吗？

**不支持降级操作**，一旦对数组进行了升级，就会一直保持升级后的状态。比如前面的升级操作的例子，如果删除了 65535 元素，整数集合的数组还是 int32_t 类型的，并不会因此降级为 int16_t 类型。

------

### 跳表

Redis 只有在 Zset 对象的底层实现用到了跳表，**跳表的优势是能支持平均 O(logN) 复杂度的节点查找**。

**Zset 对象是唯一一个同时使用了两个数据结构来实现的 Redis 对象**，这**两个数据结构一个是跳表，一个是哈希表**。这样的好处是既能进行高效的范围查询，也能进行高效单点查询。

```c
typedef struct zset {
    dict *dict;
    zskiplist *zsl;
} zset;
```

Zset 对象能支持范围查询（如 ZRANGEBYSCORE 操作），这是因为它的数据结构设计采用了跳表，而又能以常数复杂度获取元素权重（如 ZSCORE 操作），这是因为它同时**采用了哈希表进行索引**。

接下来，详细的说下跳表。

#### 跳表结构设计

链表在查找元素的时候，因为需要逐一查找，所以查询效率非常低，时间复杂度是O(N)，于是就出现了跳表。**跳表是在链表基础上改进过来的，实现了一种「多层」的有序链表**，这样的好处是能快读定位数据。

那跳表长什么样呢？我这里举个例子，下图展示了一个层级为 3 的跳表。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/2ae0ed790c7e7403f215acb2bd82e884.png)

图中头节点有 L0~L2 三个头指针，分别指向了不同层级的节点，然后每个层级的节点都通过指针连接起来：

- L0 层级共有 5 个节点，分别是节点1、2、3、4、5；
- L1 层级共有 3 个节点，分别是节点 2、3、5；
- L2 层级只有 1 个节点，也就是节点 3 。

如果我们要在链表中查找节点 4 这个元素，只能从头开始遍历链表，需要查找 4 次，而使用了跳表后，只需要查找 2 次就能定位到节点 4，因为可以在头节点直接从 L2 层级跳到节点 3，然后再往前遍历找到节点 4。

可以看到，这个查找过程就是在多个层级上跳来跳去，最后定位到元素。当数据量很大时，跳表的查找复杂度就是 O(logN)。

那跳表节点是怎么实现多层级的呢？这就需要看「跳表节点」的数据结构了，如下：

```c
typedef struct zskiplistNode {
    //Zset 对象的元素值
    sds ele;
    //元素权重值
    double score;
    //后向指针
    struct zskiplistNode *backward;
  
    //节点的level数组，保存每层上的前向指针和跨度
    struct zskiplistLevel {
        struct zskiplistNode *forward;
        unsigned long span;
    } level[];
} zskiplistNode;
```

Zset 对象要同时保存元素和元素的权重，对应到跳表节点结构里就是 sds 类型的 ele 变量和 double 类型的 score 变量。每个跳表节点都有一个后向指针，指向前一个节点，目的是为了方便从跳表的尾节点开始访问节点，这样倒序查找时很方便。

跳表是一个带有层级关系的链表，而且每一层级可以包含多个节点，每一个节点通过指针连接起来，实现这一特性就是靠跳表节点结构体中的**zskiplistLevel 结构体类型的 level 数组**。

level 数组中的每一个元素代表跳表的一层，也就是由 zskiplistLevel 结构体表示，比如 leve[0] 就表示第一层，leve[1] 就表示第二层。zskiplistLevel 结构体里定义了「指向下一个跳表节点的指针」和「跨度」，跨度时用来记录两个节点之间的距离。

比如，下面这张图，展示了各个节点的跨度。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/a5b6512be0da36f87c5d906cc278526c.png)

第一眼看到跨度的时候，以为是遍历操作有关，实际上并没有任何关系，遍历操作只需要用前向指针就可以完成了。

**跨度实际上是为了计算这个节点在跳表中的排位**。具体怎么做的呢？因为跳表中的节点都是按序排列的，那么计算某个节点排位的时候，从头节点点到该结点的查询路径上，将沿途访问过的所有层的跨度累加起来，得到的结果就是目标节点在跳表中的排位。

举个例子，查找图中节点 3 在跳表中的排位，从头节点开始查找节点 3，查找的过程只经过了一个层（L3），并且层的跨度是 3，所以节点 3 在跳表中的排位是 3。

另外，图中的头节点其实也是 zskiplistNode 跳表节点，只不过头节点的后向指针、权重、元素值都会被用到，所以图中省略了这部分。

问题来了，由谁定义哪个跳表节点是头节点呢？这就介绍「跳表」结构体了，如下所示：

```c
typedef struct zskiplist {
    struct zskiplistNode *header, *tail;
    unsigned long length;
    int level;
} zskiplist;
```

跳表结构里包含了：

- 跳表的头尾节点，便于在O(1)时间复杂度内访问跳表的头节点和尾节点；
- 跳表的长度，便于在O(1)时间复杂度获取跳表节点的数量；
- 跳表的最大层数，便于在O(1)时间复杂度获取跳表中层高最大的那个节点的层数量；

#### 跳表节点查询过程

查找一个跳表节点的过程时，**跳表会从头节点的最高层开始，逐一遍历每一层**。在遍历某一层的跳表节点时，会用跳表节点中的 SDS 类型的元素和元素的权重来进行判断，共有两个判断条件：

- 如果当前节点的权重「小于」要查找的权重时，跳表就会访问该层上的下一个节点。
- 如果当前节点的权重「等于」要查找的权重时，并且当前节点的 SDS 类型数据「小于」要查找的数据时，跳表就会访问该层上的下一个节点。

如果上面两个条件都不满足，或者下一个节点为空时，跳表就会使用目前遍历到的节点的 level 数组里的下一层指针，然后沿着下一层指针继续查找，这就相当于跳到了下一层接着查找。

举个例子，下图有个 3 层级的跳表。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/数据类型/3层跳表-跨度.drawio.png)

如果要查找「元素：abcd，权重：4」的节点，查找的过程是这样的：

- 先从头节点的最高层开始，L2 指向了「元素：abc，权重：3」节点，这个节点的权重比要查找节点的小，所以要访问该层上的下一个节点；
- 但是该层上的下一个节点是空节点，于是就会跳到「元素：abc，权重：3」节点的下一层去找，也就是 leve[1];
- 「元素：abc，权重：3」节点的 leve[1] 的下一个指针指向了「元素：abcde，权重：4」的节点，然后将其和要查找的节点比较。虽然「元素：abcde，权重：4」的节点的权重和要查找的权重相同，但是当前节点的 SDS 类型数据「大于」要查找的数据，所以会继续跳到「元素：abc，权重：3」节点的下一层去找，也就是 leve[0]；
- 「元素：abc，权重：3」节点的 leve[0] 的下一个指针指向了「元素：abcd，权重：4」的节点，该节点正是要查找的节点，查询结束。

#### 跳表节点层数设置

跳表的相邻两层的节点数量的比例会影响跳表的查询性能。

举个例子，下图的跳表，第二层的节点数量只有 1 个，而第一层的节点数量有 6 个。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/2802786ab4f52c1e248904e5cef33a74.png)

这时，如果想要查询节点 6，那基本就跟链表的查询复杂度一样，就需要在第一层的节点中依次顺序查找，复杂度就是 O(N) 了。所以，为了降低查询复杂度，我们就需要维持相邻层结点数间的关系。

**跳表的相邻两层的节点数量最理想的比例是 2:1，查找复杂度可以降低到 O(logN)**。

下图的跳表就是，相邻两层的节点数量的比例是 2 : 1。

![img](https://img-blog.csdnimg.cn/img_convert/cdc14698f629c74bf5a239cc8a611aeb.png)

> 那怎样才能维持相邻两层的节点数量的比例为 2 : 1 呢？

如果采用新增节点或者删除节点时，来调整跳表节点以维持比例的方法的话，会带来额外的开销。

Redis 则采用一种巧妙的方法是，**跳表在创建节点的时候，随机生成每个节点的层数**，并没有严格维持相邻两层的节点数量比例为 2 : 1 的情况。

具体的做法是，**跳表在创建节点时候，会生成范围为[0-1]的一个随机数，如果这个随机数小于 0.25（相当于概率 25%），那么层数就增加 1 层，然后继续生成下一个随机数，直到随机数的结果大于 0.25 结束，最终确定该节点的层数**。

这样的做法，相当于每增加一层的概率不超过 25%，层数越高，概率越低，层高最大限制是 64。

[跳表详细说明](https://www.jianshu.com/p/9d8296562806)

### quicklist

在 Redis 3.0 之前，List 对象的底层数据结构是双向链表或者压缩列表。然后在 Redis 3.2 的时候，**List 对象的底层改由 quicklist 数据结构实现**。

其实 quicklist 就是「**双向链表 + 压缩列表**」组合，因为一个 quicklist 就是一个链表，而链表中的**每个元素又是一个压缩列表**。

在前面讲压缩列表的时候，我也提到了压缩列表的不足，虽然压缩列表是通过紧凑型的内存布局节省了内存开销，但是因为它的结构设计，如果保存的元素数量增加，或者元素变大了，压缩列表会有「连锁更新」的风险，一旦发生，会造成性能下降。

quicklist 解决办法，**通过控制每个链表节点中的压缩列表的大小或者元素个数，来规避连锁更新的问题。因为压缩列表元素越少或越小，连锁更新带来的影响就越小，从而提供了更好的访问性能。**

#### quicklist 结构设计

quicklist 的结构体跟链表的结构体类似，都包含了表头和表尾，区别在于 quicklist 的节点是 quicklistNode。

```c
typedef struct quicklist {
    //quicklist的链表头
    quicklistNode *head;      //quicklist的链表头
    //quicklist的链表头
    quicklistNode *tail; 
    //所有压缩列表中的总元素个数
    unsigned long count;
    //quicklistNodes的个数
    unsigned long len;       
    ...
} quicklist;
```

接下来看看，quicklistNode 的结构定义：

```c
typedef struct quicklistNode {
    //前一个quicklistNode
    struct quicklistNode *prev;     //前一个quicklistNode
    //下一个quicklistNode
    struct quicklistNode *next;     //后一个quicklistNode
    //quicklistNode指向的压缩列表
    unsigned char *zl;              
    //压缩列表的的字节大小
    unsigned int sz;                
    //压缩列表的元素个数
    unsigned int count : 16;        //ziplist中的元素个数 
    ....
} quicklistNode;
```

可以看到，quicklistNode 结构体里包含了前一个节点和下一个节点指针，这样每个 quicklistNode 形成了一个双向链表。但是**链表节点的元素不再是单纯保存元素值，而是保存了一个压缩列表**，所以 quicklistNode 结构体里有个指向压缩列表的指针 *zl。

我画了一张图，方便你理解 quicklist 数据结构。

![img](https://img-blog.csdnimg.cn/img_convert/f46cbe347f65ded522f1cc3fd8dba549.png)

在向 quicklist 添加一个元素的时候，不会像普通的链表那样，直接新建一个链表节点。而是会**检查插入位置的压缩列表是否能容纳该元素，如果能容纳就直接保存到 quicklistNode 结构里的压缩列表**，如果不能容纳，才会新建一个新的 quicklistNode 结构。

quicklist 会**控制** quicklistNode 结构里的**压缩列表的大小或者元素个数**，来规避潜在的连锁更新的风险，但是这并没有完全解决连锁更新的问题。

### listpack

quicklist 虽然通过控制 quicklistNode 结构里的压缩列表的大小或者元素个数，来减少连锁更新带来的性能影响，但是并没有完全解决连锁更新的问题。

因为 quicklistNode 还是用了压缩列表来保存元素，压缩列表连锁更新的问题，来源于它的结构设计，所以要想彻底解决这个问题，需要设计一个新的数据结构。

于是，Redis 在 5.0 新设计一个数据结构叫 listpack，目的是替代压缩列表，它最大特点**是 listpack 中每个节点不再包含前一个节点的长度了**，压缩列表每个节点正因为需要保存前一个节点的长度字段，就会有连锁更新的隐患。

我看了 Redis 的 Github，在最新 6.2 发行版本中，Redis Hash 对象、ZSet 对象的底层数据结构的压缩列表还未被替换成 listpack，而 Redis 的最新代码（还未发布版本）已经**将所有用到压缩列表底层数据结构的 Redis 对象替换成 listpack 数据结构来实现**，估计不久将来，Redis 就会发布一个将压缩列表为 listpack 的发行版本。

#### listpack 结构设计

listpack(**紧凑列表**) 采用了压缩列表的很多优秀的设计，比如还是**用一块连续的内存空间来紧凑地保存数据，并且为了节省内存的开销，listpack 节点会采用不同的编码方式保存不同大小的数据**。

我们先看看 **listpack 结构**：

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/4d2dc376b5fd68dae70d9284ae82b73a.png)

listpack 头包含两个属性，分别记录了 listpack **总字节数和元素数量**，然后 listpack 末尾也**有个结尾标识**。图中的 listpack entry 就是 listpack 的节点了。

每个 listpack **节点结构**如下：

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/c5fb0a602d4caaca37ff0357f05b0abf.png)

主要包含三个方面内容：

- encoding，定义**该元素的编码类型**，会对不同长度的整数和字符串进行编码；
- data，实际存放的数据；
- len，encoding+data的总长度；

可以看到，**listpack 没有压缩列表中记录前一个节点长度的字段了，listpack 只记录当前节点的长度，当我们向 listpack 加入一个新元素的时候，不会影响其他节点的长度字段的变化，从而避免了压缩列表的连锁更新问题**。

## Redis 事务

[参考](https://segmentfault.com/a/1190000023951592)、[参考](https://redisbook.readthedocs.io/en/latest/feature/transaction.html)

### 事务

事务提供了一种“将多个命令打包， 然后一次性、按顺序地执行”的机制， 并且**事务在执行的期间不会主动中断** —— 服务器在执行完事务中的所有命令之后， 才会继续处理其他客户端的其他命令。

以下是一个事务的例子， 它先以 [MULTI](http://redis.readthedocs.org/en/latest/transaction/multi.html#multi) 开始一个事务， 然后将多个命令入队到事务中， 最后由 [EXEC](http://redis.readthedocs.org/en/latest/transaction/exec.html#exec) 命令触发事务， 一并执行事务中的所有命令：

```
redis> MULTI
OK

redis> SET book-name "Mastering C++ in 21 days"
QUEUED

redis> GET book-name
QUEUED

redis> SADD tag "C++" "Programming" "Mastering Series"
QUEUED

redis> SMEMBERS tag
QUEUED

redis> EXEC
1) OK
2) "Mastering C++ in 21 days"
3) (integer) 3
4) 1) "Mastering Series"
   2) "C++"
   3) "Programming"
```

Redis 事务可以一次执行多个命令，本质是一组命令的集合。一个事务中的所有命令都会序列化，按顺序地串行化执行而不会被其它命令插入，不许加塞。

可以保证一个队列中，一次性、顺序性、排他性的执行一系列命令（Redis 事务的**主要作用其实就是串联多个命令防止别的命令插队**）

官方文档是这么说的

> 事务可以一次执行多个命令， 并且带有以下两个重要的保证：
>
> - 事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
> - 事务是一个原子操作：事务中的命令要么全部被执行，要么全部都不执行

这个原子操作，和关系型 DB 的原子性不太一样，它不能完全保证原子性

### Redis 事务的几个命令

| 命令    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| MULTI   | 标记一个事务块的开始                                         |
| EXEC    | 执行所有事务块内的命令                                       |
| DISCARD | 取消事务，放弃执行事务块内的所有命令                         |
| WATCH   | 监视一个（或多个）key，如果在事务执行之前这个（或多个）key被其他命令所改动，那么事务将被打断 |
| UNWATCH | 取消 WATCH 命令对所有 keys 的监视                            |

[MULTI](https://link.segmentfault.com/?enc=QXXqyNTVnFD1eNMacmRA1g%3D%3D.HvDaXZ1xFBtWHNVouWsQW2uKX%2FED9Gj5ubMxxM8JhMDdvtBBBrDMFJHRj%2Bx9zgJh%2FjaNc%2FNi6doGfpmmCnPgzw%3D%3D) 命令用于开启一个事务，它总是返回 OK 。

MULTI 执行之后， **客户端可以继续向服务器发送任意多条命令， 这些命令不会立即被执行， 而是被放到一个队列中**， 当 [EXEC](https://link.segmentfault.com/?enc=LxPBkQczQRcuZRv48UDI7g%3D%3D.LHnekw8eUf0cMqZFrV4k9ElTK1uQ8OgY32awpe27KOiqE%2BM%2BZs%2FdQ63rFhpKRCpb) 命令被调用时， 所有队列中的命令才会被执行。

另一方面， 通过调用 [DISCARD](https://link.segmentfault.com/?enc=GszRRjvdYOdjzYGh2M1TXw%3D%3D.Om%2B%2BIL6Xbwj66WLBzBRF96srbl2LkDq7X3zOkgPTQqgzMCbLhK0fxxxVtHpyOghn%2FwPHcTXi%2BQnD25PzsIcjYg%3D%3D) ， 客户端可以清空事务队列， 并放弃执行事务。

#### 开始事务

[MULTI](http://redis.readthedocs.org/en/latest/transaction/multi.html#multi) 命令的执行标记着事务的开始：

```
redis> MULTI
OK
```

这个命令唯一做的就是， 将客户端的 `REDIS_MULTI` 选项打开， 让客户端从非事务状态切换到事务状态。

![digraph normal_to_transaction {      rankdir = LR;      node [shape = circle, style = filled];      edge [style = bold];      label = "客户端状态的切换";      normal [label = "非事务状态", fillcolor = "#FADCAD"];      transaction [label = "事务状态", fillcolor = "#A8E270"];      normal -> transaction [label = "打开选项\nREDIS_MULTI"]; }](https://redisbook.readthedocs.io/en/latest/_images/graphviz-0ff9f2e58803dbb8c1c400e1f8191f77d4c2917e.svg)

#### 命令入队

当客户端处于非事务状态下时， 所有发送给服务器端的命令都会立即被服务器执行：

```
redis> SET msg "hello moto"
OK

redis> GET msg
"hello moto"
```

但是， 当客户端进入事务状态之后， 服务器在收到来自客户端的命令时， 不会立即执行命令， 而是将这些命令全部放进一个事务队列里， 然后返回 `QUEUED` ， 表示命令已入队：

```
redis> MULTI
OK

redis> SET msg "hello moto"
QUEUED

redis> GET msg
QUEUED
```

以下流程图展示了这一行为：

![digraph enqueue {      node [shape = plaintext, style = filled];      edge [style = bold];      command_in [label = "服务器接到来自客户端的命令"];      in_transaction_or_not [label = "客户端是否正处于事务状态？", shape = diamond, fillcolor = "#95BBE3"];      enqueu_command [label = "将命令放进事务队列里", fillcolor = "#A8E270"];      return_enqueued [label = "向客户端返回 QUEUED 字符串\n表示命令已入队", fillcolor = "#A8E270"];      exec_command [label = "执行命令", fillcolor = "#FADCAD"];      return_command_result [label = "向客户端返回命令的执行结果", fillcolor = "#FADCAD"];      //       command_in -> in_transaction_or_not;      in_transaction_or_not -> enqueu_command [label = "是"];      in_transaction_or_not -> exec_command [label = "否"];      exec_command -> return_command_result;      enqueu_command -> return_enqueued; }](https://redisbook.readthedocs.io/en/latest/_images/graphviz-8a0f8eae0bb8180e877b799921dd690267c2d3b4.svg)

事务队列是一个数组， 每个数组项是都包含三个属性：

1. 要执行的命令（cmd）。
2. 命令的参数（argv）。
3. 参数的个数（argc）。

举个例子， 如果客户端执行以下命令：

```
redis> MULTI
OK

redis> SET book-name "Mastering C++ in 21 days"
QUEUED

redis> GET book-name
QUEUED

redis> SADD tag "C++" "Programming" "Mastering Series"
QUEUED

redis> SMEMBERS tag
QUEUED
```

那么程序将为客户端创建以下事务队列：

| 数组索引 | cmd        | argv                                                | argc |
| :------- | :--------- | :-------------------------------------------------- | :--- |
| `0`      | `SET`      | `["book-name", "Mastering C++ in 21 days"]`         | `2`  |
| `1`      | `GET`      | `["book-name"]`                                     | `1`  |
| `2`      | `SADD`     | `["tag", "C++", "Programming", "Mastering Series"]` | `4`  |
| `3`      | `SMEMBERS` | `["tag"]`                                           | `1`  |

#### 执行事务

前面说到， 当客户端进入事务状态之后， 客户端发送的命令就会被放进事务队列里。

但其实**并不是所有的命令都会被放进事务队列**， 其中的例外就是 [EXEC](http://redis.readthedocs.org/en/latest/transaction/exec.html#exec) 、 [DISCARD](http://redis.readthedocs.org/en/latest/transaction/discard.html#discard) 、 [MULTI](http://redis.readthedocs.org/en/latest/transaction/multi.html#multi) 和 [WATCH](http://redis.readthedocs.org/en/latest/transaction/watch.html#watch) 这四个命令 —— 当这四个命令从客户端发送到服务器时， 它们会像客户端处于非事务状态一样， 直接被服务器执行：

![digraph not_enque_command {      node [shape = plaintext, style = filled];      edge [style = bold];      command_in [label = "服务器接到来自客户端的命令"];      in_transaction_or_not [label = "客户端是否正处于事务状态？", shape = diamond, fillcolor = "#95BBE3"];      not_exec_and_discard [label = "命令是否\nEXEC 、 DISCARD 、\nMULTI 或 WATCH ？", shape = diamond, fillcolor = "#FFC1C1"];      enqueu_command [label = "将命令放进事务队列里", fillcolor = "#A8E270"];      return_enqueued [label = "向客户端返回 QUEUED 字符串\n表示命令已入队", fillcolor = "#A8E270"];      exec_command [label = "执行命令", fillcolor = "#FADCAD"];      return_command_result [label = "向客户端返回命令的执行结果", fillcolor = "#FADCAD"];      //       command_in -> in_transaction_or_not;      in_transaction_or_not -> not_exec_and_discard [label = "是"];      not_exec_and_discard -> enqueu_command [label = "否"];      not_exec_and_discard -> exec_command [label = "是"];      in_transaction_or_not -> exec_command [label = "否"];      exec_command -> return_command_result;      enqueu_command -> return_enqueued; }](https://redisbook.readthedocs.io/en/latest/_images/graphviz-836c8a3dc33526a649d9ecf5b7b959d72b38cc7d.svg)

如果客户端正处于事务状态， 那么当 [EXEC](http://redis.readthedocs.org/en/latest/transaction/exec.html#exec) 命令执行时， 服务器根据客户端所保存的事务队列， 以先进先出（FIFO）的方式执行事务队列中的命令： 最先入队的命令最先执行， 而最后入队的命令最后执行。

比如说，对于以下事务队列：

| 数组索引 | cmd        | argv                                                | argc |
| :------- | :--------- | :-------------------------------------------------- | :--- |
| `0`      | `SET`      | `["book-name", "Mastering C++ in 21 days"]`         | `2`  |
| `1`      | `GET`      | `["book-name"]`                                     | `1`  |
| `2`      | `SADD`     | `["tag", "C++", "Programming", "Mastering Series"]` | `4`  |
| `3`      | `SMEMBERS` | `["tag"]`                                           | `1`  |

程序会首先执行 [SET](http://redis.readthedocs.org/en/latest/string/set.html#set) 命令， 然后执行 [GET](http://redis.readthedocs.org/en/latest/string/get.html#get) 命令， 再然后执行 [SADD](http://redis.readthedocs.org/en/latest/set/sadd.html#sadd) 命令， 最后执行 [SMEMBERS](http://redis.readthedocs.org/en/latest/set/smembers.html#smembers) 命令。

执行事务中的命令所得的结果会以 FIFO 的顺序保存到一个回复队列中。

比如说，对于上面给出的事务队列，程序将为队列中的命令创建如下回复队列：

| 数组索引 | 回复类型          | 回复内容                                     |
| :------- | :---------------- | :------------------------------------------- |
| `0`      | status code reply | `OK`                                         |
| `1`      | bulk reply        | `"Mastering C++ in 21 days"`                 |
| `2`      | integer reply     | `3`                                          |
| `3`      | multi-bulk reply  | `["Mastering Series", "C++", "Programming"]` |

当事务队列里的所有命令被执行完之后， [EXEC](http://redis.readthedocs.org/en/latest/transaction/exec.html#exec) 命令会将回复队列作为自己的执行结果返回给客户端， 客户端从事务状态返回到非事务状态， 至此， 事务执行完毕。

事务的整个执行过程可以用以下伪代码表示：

```
def execute_transaction():

    # 创建空白的回复队列
    reply_queue = []

    # 取出事务队列里的所有命令、参数和参数数量
    for cmd, argv, argc in client.transaction_queue:

        # 执行命令，并取得命令的返回值
        reply = execute_redis_command(cmd, argv, argc)

        # 将返回值追加到回复队列末尾
        reply_queue.append(reply)

    # 清除客户端的事务状态
    clear_transaction_state(client)

    # 清空事务队列
    clear_transaction_queue(client)

    # 将事务的执行结果返回给客户端
    send_reply_to_client(client, reply_queue)
```

#### 在事务和非事务状态下执行命令

无论在事务状态下， 还是在非事务状态下， Redis 命令都由同一个函数执行， 所以它们共享很多服务器的一般设置， 比如 AOF 的配置、RDB 的配置，以及内存限制，等等。

不过事务中的命令和普通命令在执行上还是有一点区别的，其中最重要的两点是：

1. 非事务状态下的命令以单个命令为单位执行，**前一个命令和后一个命令的客户端不一定是同一个**；

   而事务状态则是以一个事务为单位，执行事务队列中的所有命令：除非当前事务执行完毕，否则**服务器不会中断事务，也不会执行其他客户端的其他命令**。

2. 在非事务状态下，执行命令所得的结果会立即被返回给客户端；

   而事务则是将所有命令的结果集合到回复队列，再作为 [EXEC](http://redis.readthedocs.org/en/latest/transaction/exec.html#exec) 命令的结果返回给客户端。

#### 事务状态下的 DISCARD 、 MULTI 和 WATCH 命令

除了 [EXEC](http://redis.readthedocs.org/en/latest/transaction/exec.html#exec) 之外， 服务器在客户端处于事务状态时， 不加入到事务队列而直接执行的另外三个命令是 [DISCARD](http://redis.readthedocs.org/en/latest/transaction/discard.html#discard) 、 [MULTI](http://redis.readthedocs.org/en/latest/transaction/multi.html#multi) 和 [WATCH](http://redis.readthedocs.org/en/latest/transaction/watch.html#watch) 。

[DISCARD](http://redis.readthedocs.org/en/latest/transaction/discard.html#discard) 命令用于取消一个事务， 它清空客户端的整个事务队列， 然后将客户端从事务状态调整回非事务状态， 最后返回字符串 `OK` 给客户端， 说明事务已被取消。

Redis 的事务是不可嵌套的， 当客户端已经处于事务状态， 而客户端又再向服务器发送 [MULTI](http://redis.readthedocs.org/en/latest/transaction/multi.html#multi) 时， 服务器只是简单地向客户端发送一个错误， 然后继续等待其他命令的入队。 [MULTI](http://redis.readthedocs.org/en/latest/transaction/multi.html#multi) 命令的发送不会造成整个事务失败， 也不会修改事务队列中已有的数据。

[WATCH](http://redis.readthedocs.org/en/latest/transaction/watch.html#watch) 只能**在客户端进入事务状态之前执行**， 在事务状态下发送 [WATCH](http://redis.readthedocs.org/en/latest/transaction/watch.html#watch) 命令会引发一个错误， 但它不会造成整个事务失败， 也不会修改事务队列中已有的数据（和前面处理 [MULTI](http://redis.readthedocs.org/en/latest/transaction/multi.html#multi) 的情况一样）。

#### 带 WATCH 的事务

[WATCH](http://redis.readthedocs.org/en/latest/transaction/watch.html#watch) 命令**用于在事务开始之前监视任意数量的键**： 当调用 [EXEC](http://redis.readthedocs.org/en/latest/transaction/exec.html#exec) 命令执行事务时， **如果任意一个被监视的键已经被其他客户端修改了， 那么整个事务不再执行， 直接返回失败**。

以下示例展示了一个执行失败的事务例子：

```
redis> WATCH name
OK

redis> MULTI
OK

redis> SET name peter
QUEUED

redis> EXEC
(nil)
```

以下执行序列展示了上面的例子是如何失败的：

| 时间 | 客户端 A         | 客户端 B        |
| :--- | :--------------- | :-------------- |
| T1   | `WATCH name`     |                 |
| T2   | `MULTI`          |                 |
| T3   | `SET name peter` |                 |
| T4   |                  | `SET name john` |
| T5   | `EXEC`           |                 |

在时间 T4 ，客户端 B 修改了 `name` 键的值， 当客户端 A 在 T5 执行 [EXEC](http://redis.readthedocs.org/en/latest/transaction/exec.html#exec) 时，Redis 会发现 `name` 这个被监视的键已经被修改， 因此客户端 A 的事务不会被执行，而是直接返回失败。

下文就来介绍 [WATCH](http://redis.readthedocs.org/en/latest/transaction/watch.html#watch) 的实现机制，并且看看事务系统是如何检查某个被监视的键是否被修改，从而保证事务的安全性的。

#### WATCH 命令的实现

在每个代表数据库的 `redis.h/redisDb` 结构类型中， 都保存了一个 `watched_keys` 字典， 字典的键是这个数据库被监视的键， 而字典的值则是一个链表， **链表中保存了所有监视这个键的客户端**。

比如说，以下字典就展示了一个 `watched_keys` 字典的例子：

![digraph watched_keys {      rankdir = LR;      node [shape = record, style = filled];      edge [style = bold];      // keys      watched_keys [label = "watched_keys |<key1> key1 |<key2> key2 |<key3> key3 | ... |<keyN> keyN", fillcolor = "#A8E270"];      // clients blocking for key1     client1 [label = "client1", fillcolor = "#95BBE3"];     client5 [label = "client5", fillcolor = "#95BBE3"];     client2 [label = "client2", fillcolor = "#95BBE3"];     null_1 [label = "NULL", shape = plaintext];          watched_keys:key1 -> client2;     client2 -> client5;     client5 -> client1;     client1 -> null_1;      // clients blocking for key2     client7 [label = "client7", fillcolor = "#95BBE3"];     null_2 [label = "NULL", shape = plaintext];      watched_keys:key2 -> client7;     client7 -> null_2;      // key3      client3 [label = "client3", fillcolor = "#95BBE3"];     client4 [label = "client4", fillcolor = "#95BBE3"];     client6 [label = "client6", fillcolor = "#95BBE3"];     null_3 [label = "NULL", shape = plaintext];      watched_keys:key3 -> client3;     client3 -> client4;     client4 -> client6;     client6 -> null_3; }](https://redisbook.readthedocs.io/en/latest/_images/graphviz-9aea81f33da1373550c590eb0b7ca0c2b3d38366.svg)

其中， 键 `key1` 正在被 `client2` 、 `client5` 和 `client1` 三个客户端监视， 其他一些键也分别被其他别的客户端监视着。

[WATCH](http://redis.readthedocs.org/en/latest/transaction/watch.html#watch) 命令的作用， 就是**将当前客户端和要监视的键在 `watched_keys` 中进行关联**。

举个例子， 如果当前客户端为 `client10086` ， 那么当客户端执行 `WATCH key1 key2` 时， 前面展示的 `watched_keys` 将被修改成这个样子：

![digraph new_watched_keys {      rankdir = LR;      node [shape = record, style = filled];      edge [style = bold];      // keys      watched_keys [label = "watched_keys |<key1> key1 |<key2> key2 |<key3> key3 | ... |<keyN> keyN", fillcolor = "#A8E270"];      // clients blocking for key1     client1 [label = "client1", fillcolor = "#95BBE3"];     client5 [label = "client5", fillcolor = "#95BBE3"];     client2 [label = "client2", fillcolor = "#95BBE3"];     client10086 [label = "client10086", fillcolor = "#FFC1C1"];     null_1 [label = "NULL", shape = plaintext];          watched_keys:key1 -> client2;     client2 -> client5;     client5 -> client1;     client1 -> client10086;     client10086 -> null_1;      // clients blocking for key2     client7 [label = "client7", fillcolor = "#95BBE3"];     client10086_2 [label = "client10086", fillcolor = "#FFC1C1"];     null_2 [label = "NULL", shape = plaintext];      watched_keys:key2 -> client7;     client7 -> client10086_2;     client10086_2 -> null_2;      // key3      client3 [label = "client3", fillcolor = "#95BBE3"];     client4 [label = "client4", fillcolor = "#95BBE3"];     client6 [label = "client6", fillcolor = "#95BBE3"];     null_3 [label = "NULL", shape = plaintext];      watched_keys:key3 -> client3;     client3 -> client4;     client4 -> client6;     client6 -> null_3; }](https://redisbook.readthedocs.io/en/latest/_images/graphviz-fe5e31054c282a3cdd86656994fe1678a3d4f201.svg)

通过 `watched_keys` 字典， 如果程序想检查某个键是否被监视， 那么它只要检查字典中是否存在这个键即可； 如果程序要获取监视某个键的所有客户端， 那么只要取出键的值（一个链表）， 然后对链表进行遍历即可。

#### WATCH 的触发

在任何对数据库键空间（key space）进行修改的命令成功执行之后 （比如 [FLUSHDB](http://redis.readthedocs.org/en/latest/server/flushdb.html#flushdb) 、 [SET](http://redis.readthedocs.org/en/latest/string/set.html#set) 、 [DEL](http://redis.readthedocs.org/en/latest/key/del.html#del) 、 [LPUSH](http://redis.readthedocs.org/en/latest/list/lpush.html#lpush) 、 [SADD](http://redis.readthedocs.org/en/latest/set/sadd.html#sadd) 、 [ZREM](http://redis.readthedocs.org/en/latest/sorted_set/zrem.html#zrem) ，诸如此类）， `multi.c/touchWatchedKey` 函数都会被调用 —— 它检查数据库的 `watched_keys` 字典， 看是否有客户端在监视已经被命令修改的键， 如果有的话， 程序将**所有监视这个/这些被修改键的客户端的 `REDIS_DIRTY_CAS` 选项打开**：

![digraph dirty_cas {      rankdir = LR;      node [shape = circle, style = filled];      edge [style = bold];      label = "客户端状态的切换";      normal [label = "非事务状态", fillcolor = "#FADCAD"];      transaction [label = "事务状态", fillcolor = "#A8E270"];      dirty_cas [label = "事务安全性\n已被破坏", fillcolor = "#B22222"];      normal -> transaction [label = "打开选项\nREDIS_MULTI"];      transaction -> dirty_cas [label = "打开选项\nREDIS_DIRTY_CAS"]; }](https://redisbook.readthedocs.io/en/latest/_images/graphviz-e5c66122242aa10939b696dfeeb905343c5202bd.svg)

当客户端发送 [EXEC](http://redis.readthedocs.org/en/latest/transaction/exec.html#exec) 命令、触发事务执行时， 服务器会对客户端的状态进行检查：

- 如果客户端的 `REDIS_DIRTY_CAS` 选项已经被打开，那么说明被客户端监视的键至少有一个已经被修改了，事务的安全性已经被破坏。服务器会放弃执行这个事务，直接向客户端返回空回复，表示事务执行失败。
- 如果 `REDIS_DIRTY_CAS` 选项没有被打开，那么说明所有监视键都安全，服务器正式执行事务。

可以用一段伪代码来表示这个检查：

```
def check_safety_before_execute_trasaction():

    if client.state & REDIS_DIRTY_CAS:
        # 安全性已破坏，清除事务状态
        clear_transaction_state(client)
        # 清空事务队列
        clear_transaction_queue(client)
        # 返回空回复给客户端
        send_empty_reply(client)
    else:
        # 安全性完好，执行事务
        execute_transaction()
```

举个例子，假设数据库的 `watched_keys` 字典如下图所示：

![digraph watched_keys {      rankdir = LR;      node [shape = record, style = filled];      edge [style = bold];      // keys      watched_keys [label = "watched_keys |<key1> key1 |<key2> key2 |<key3> key3 | ... |<keyN> keyN", fillcolor = "#A8E270"];      // clients blocking for key1     client1 [label = "client1", fillcolor = "#95BBE3"];     client5 [label = "client5", fillcolor = "#95BBE3"];     client2 [label = "client2", fillcolor = "#95BBE3"];     null_1 [label = "NULL", shape = plaintext];          watched_keys:key1 -> client2;     client2 -> client5;     client5 -> client1;     client1 -> null_1;      // clients blocking for key2     client7 [label = "client7", fillcolor = "#95BBE3"];     null_2 [label = "NULL", shape = plaintext];      watched_keys:key2 -> client7;     client7 -> null_2;      // key3      client3 [label = "client3", fillcolor = "#95BBE3"];     client4 [label = "client4", fillcolor = "#95BBE3"];     client6 [label = "client6", fillcolor = "#95BBE3"];     null_3 [label = "NULL", shape = plaintext];      watched_keys:key3 -> client3;     client3 -> client4;     client4 -> client6;     client6 -> null_3; }](https://redisbook.readthedocs.io/en/latest/_images/graphviz-9aea81f33da1373550c590eb0b7ca0c2b3d38366.svg)

如果某个客户端对 `key1` 进行了修改（比如执行 `DEL key1` ）， 那么所有监视 `key1` 的客户端， 包括 `client2` 、 `client5` 和 `client1` 的 `REDIS_DIRTY_CAS` 选项都会被打开， 当客户端 `client2` 、 `client5` 和 `client1` 执行 [EXEC](http://redis.readthedocs.org/en/latest/transaction/exec.html#exec) 的时候， 它们的事务都会以失败告终。

最后，当一个客户端结束它的事务时，无论事务是成功执行，还是失败， `watched_keys` 字典中和这个客户端相关的资料都会被清除。

### 事务的 ACID 性质

在传统的关系式数据库中，常常用 [ACID 性质](http://en.wikipedia.org/wiki/ACID)来检验事务功能的安全性。

**Redis 事务保证了其中的一致性（C）和隔离性（I）**，但并不保证原子性（A）和持久性（D）。

以下四小节是关于这四个性质的详细讨论。

#### 原子性（Atomicity）

**单个 Redis 命令的执行是原子性的，但 Redis 没有在事务上增加任何维持原子性的机制**，所以 Redis 事务的执行并不是原子性的。

如果一个事务队列中的所有命令都被成功地执行，那么称这个事务执行成功。

另一方面，如果 Redis 服务器进程在执行事务的过程中被停止 —— 比如接到 KILL 信号、宿主机器停机，等等，那么事务执行失败。

**当事务失败时，Redis 也不会进行任何的重试或者回滚动作**。

#### 一致性（Consistency）

Redis 的一致性问题可以分为三部分来讨论：入队错误、执行错误、Redis 进程被终结。

##### 入队错误

在命令入队的过程中，**如果客户端向服务器发送了错误的命令**，比如命令的参数数量不对，等等， 那么**服务器将向客户端返回一个出错信息， 并且将客户端的事务状态设为 `REDIS_DIRTY_EXEC`** 。

当客户端执行 [EXEC](http://redis.readthedocs.org/en/latest/transaction/exec.html#exec) 命令时， Redis 会拒绝执行状态为 `REDIS_DIRTY_EXEC` 的事务， 并返回失败信息。

```
redis 127.0.0.1:6379> MULTI
OK

redis 127.0.0.1:6379> set key
(error) ERR wrong number of arguments for 'set' command

redis 127.0.0.1:6379> EXISTS key
QUEUED

redis 127.0.0.1:6379> EXEC
(error) EXECABORT Transaction discarded because of previous errors.
```

因此，带有不正确入队命令的事务不会被执行，也不会影响数据库的一致性。

##### 执行错误

**如果命令在事务执行的过程中发生错误，比如说，对一个不同类型的 key 执行了错误的操作， 那么 Redis 只会将错误包含在事务的结果中**， 这不会引起事务中断或整个失败，不会影响已执行事务命令的结果，也不会影响后面要执行的事务命令， 所以它对事务的一致性也没有影响。

##### Redis 进程被终结

如果 Redis 服务器进程**在执行事务的过程中被其他进程终结，或者被管理员强制杀死**，那么根据 Redis 所使用的持久化模式，可能有以下情况出现：

- **内存模式**：如果 Redis 没有采取任何持久化机制，那么重启之后的数据库总是空白的，所以数据总是一致的。

- **RDB 模式**：**在执行事务时，Redis 不会中断事务去执行保存 RDB 的工作，只有在事务执行之后，保存 RDB 的工作才有可能开始**。所以当 RDB 模式下的 Redis 服务器进程在事务中途被杀死时，事务内执行的命令，不管成功了多少，都不会被保存到 RDB 文件里。恢复数据库需要使用现有的 RDB 文件，而这个 RDB 文件的数据保存的是最近一次的数据库快照（snapshot），所以它的数据可能不是最新的，但只要 RDB 文件本身没有因为其他问题而出错，那么还原后的数据库就是一致的。

- **AOF 模式**：**因为保存 AOF 文件的工作在后台线程进行，所以即使是在事务执行的中途，保存 AOF 文件的工作也可以继续进行**，因此，根据事务语句是否被写入并保存到 AOF 文件，有以下两种情况发生：

  1）如果事务语句未写入到 AOF 文件，或 AOF 未被 SYNC 调用保存到磁盘，那么当进程被杀死之后，Redis 可以根据最近一次成功保存到磁盘的 AOF 文件来还原数据库，只要 AOF 文件本身没有因为其他问题而出错，那么还原后的数据库总是一致的，但其中的数据不一定是最新的。

  2）如果事务的部分语句被写入到 AOF 文件，并且 AOF 文件被成功保存，那么不完整的事务执行信息就会遗留在 AOF 文件里，当重启 Redis 时，**程序会检测到 AOF 文件并不完整，Redis 会退出，并报告错误。需要使用 redis-check-aof 工具将部分成功的事务命令移除之后，才能再次启动服务器**。还原之后的数据总是一致的，而且数据也是最新的（直到事务执行之前为止）。

#### 隔离性（Isolation）

Redis 是单进程程序，并且它保证在执行事务时，不会对事务进行中断，事务可以运行直到执行完所有事务队列中的命令为止。因此，Redis 的事务是总是带有隔离性的。

#### 持久性（Durability）

因为事务不过是**用队列包裹起了一组 Redis 命令，并没有提供任何额外的持久性功能**，所以事务的持久性由 Redis 所使用的持久化模式决定：

- 在单纯的内存模式下，事务肯定是不持久的。

- 在 RDB 模式下，服务器可能在事务执行之后、RDB 文件更新之前的这段时间失败，所以 RDB 模式下的 Redis 事务也是不持久的。

- 在 AOF 的“总是 SYNC ”模式下，事务的每条命令在执行成功之后，都会立即调用 `fsync` 或 `fdatasync` 将事务数据写入到 AOF 文件。但是，这种保存是由后台线程进行的，主线程不会阻塞直到保存成功，所以从命令执行成功到数据保存到硬盘之间，还是有一段非常小的间隔，所以这种模式下的事务也是不持久的。

  其他 AOF 模式也和“总是 SYNC ”模式类似，所以它们都是不持久的。

### 小结

- 事务提供了一种将多个命令打包，然后一次性、有序地执行的机制。
- 事务在执行过程中不会被中断，所有事务命令执行完之后，事务才能结束。
- 多个命令会被入队到事务队列中，然后按先进先出（FIFO）的顺序执行。
- 带 `WATCH` 命令的事务会将客户端和被监视的键在数据库的 `watched_keys` 字典中进行关联，当键被修改时，程序会将所有监视被修改键的客户端的 `REDIS_DIRTY_CAS` 选项打开。
- 只有在客户端的 `REDIS_DIRTY_CAS` 选项未被打开时，才能执行事务，否则事务直接返回失败。
- Redis 的事务保证了 ACID 中的一致性（C）和隔离性（I），但并不保证原子性（A）和持久性（D）。



## Redis 过期删除策略和内存淘汰策略

[参考](https://xiaolincoding.com/redis/module/strategy.html)

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/过期策略/提纲.jpg)

### 过期删除策略

Redis 是可以**对 key 设置过期时间**的，因此需要有相应的机制将已过期的键值对删除，而做这个工作的就是过期键值删除策略。

#### 如何设置过期时间？

先说一下对 key 设置过期时间的命令。 设置 key 过期时间的命令一共有 4 个：

- `expire <key> <n>`：**设置 key 在 n 秒后过期**，比如 expire key 100 表示设置 key 在 100 秒后过期；
- `pexpire <key> <n>`：**设置 key 在 n 毫秒后过期**，比如 pexpire key2 100000 表示设置 key2 在 100000 毫秒（100 秒）后过期。
- `expireat <key> <n>`：**设置 key 在某个时间戳（精确到秒）之后过期**，比如 expireat key3 1655654400 表示 key3 在时间戳 1655654400 后过期（精确到秒）；
- `pexpireat <key> <n>`：**设置 key 在某个时间戳（精确到毫秒）之后过期**，比如 pexpireat key4 1655654400000 表示 key4 在时间戳 1655654400000 后过期（精确到毫秒）

当然，在设置字符串时，也可以同时对 key 设置过期时间，共有 3 种命令：

- `set <key> <value> ex <n>` ：设置键值对的时候，同时指定过期时间（精确到秒）；
- `set <key> <value> px <n>` ：设置键值对的时候，同时指定过期时间（精确到毫秒）；
- `setex <key> <n> <valule>` ：设置键值对的时候，同时指定过期时间（精确到秒）。

如果你想查看某个 key **剩余的存活时间**，可以使用 `TTL <key>` 命令。

```bash
# 设置键值对的时候，同时指定过期时间位 60 秒
> setex key1 60 value1
OK

# 查看 key1 过期时间还剩多少
> ttl key1
(integer) 56
> ttl key1
(integer) 52
```

如果突然反悔，取消 key 的过期时间，则可以使用 `PERSIST <key>` 命令。

```bash
# 取消 key1 的过期时间
> persist key1
(integer) 1

# 使用完 persist 命令之后，
# 查下 key1 的存活时间结果是 -1，表明 key1 永不过期 
> ttl key1 
(integer) -1
```

#### 如何判定 key 已过期了？

每当我们对一个 key 设置了过期时间时，Redis 会把该 key 带上过期时间存储到一个**过期字典**（expires dict）中，也就是说「过期字典」保存了数据库中所有 key 的过期时间。

过期字典存储在 redisDb 结构中，如下：

```c
typedef struct redisDb {
    dict *dict;    /* 数据库键空间，存放着所有的键值对 */
    dict *expires; /* 键的过期时间 */
    ....
} redisDb;
```

过期字典数据结构结构如下：

- 过期字典的 key 是一个指针，指向某个键对象；
- 过期字典的 value 是一个 long long 类型的整数，这个整数保存了 key 的过期时间；

过期字典的数据结构如下图所示：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/过期策略/过期字典数据结构.png)

字典实际上是哈希表，哈希表的最大好处就是让我们可以用 O(1) 的时间复杂度来快速查找。当我们查询一个 key 时，Redis 首先检查该 key 是否存在于过期字典中：

- 如果不在，则正常读取键值；
- 如果存在，则会获取该 key 的过期时间，然后与当前系统时间进行比对，如果比系统时间大，那就没有过期，否则判定该 key 已过期。

过期键判断流程如下图所示：

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/过期策略/过期判断流程.jpg" alt="img" style="zoom: 40%;" />

#### 过期删除策略有哪些？

在说 Redis 过期删除策略之前，先跟大家介绍下，常见的三种过期删除策略：

- **定时删除**；
- **惰性删除**；
- **定期删除**；

接下来，分别分析它们的优缺点。

> 定时删除策略是怎么样的？

定时删除策略的做法是，**在设置 key 的过期时间时，同时创建一个定时事件，当时间到达时，由事件处理器自动执行 key 的删除操作。**

定时删除策略的**优点**：

- 可以**保证过期 key 会被尽快删除**，也就是内存可以被尽快地释放。因此，定时删除对内存是最友好的。

定时删除策略的**缺点**：

- 在过期 key 比较多的情况下，删除过期 key 可能会**占用相当一部分 CPU 时间**，在内存不紧张但 CPU 时间紧张的情况下，将 CPU 时间用于删除和当前任务无关的过期键上，无疑会对服务器的响应时间和吞吐量造成影响。所以，定时删除策略对 CPU 不友好。

> 惰性删除策略是怎么样的？

惰性删除策略的做法是，**不主动删除过期键，每次从数据库访问 key 时，都检测 key 是否过期，如果过期则删除该 key。**

惰性删除策略的**优点**：

- 因为每次访问时，才会检查 key 是否过期，所以此策略只会使用很少的系统资源，因此，惰性删除策略对 CPU 时间最友好。

惰性删除策略的**缺点**：

- 如果一个 key 已经过期，而这个 key 又仍然保留在数据库中，那么只要这个过期 key 一直没有被访问，它所占用的内存就不会释放，造成了一定的内存空间浪费。所以，**惰性删除策略对内存不友好**。

> 定期删除策略是怎么样的？

定期删除策略的做法是，**每隔一段时间「随机」从数据库中取出一定数量的 key 进行检查，并删除其中的过期key。**

定期删除策略的**优点**：

- 通过限制删除操作执行的时长和频率，来减少删除操作对 CPU 的影响，同时也能删除一部分过期的数据减少了过期键对空间的无效占用。

定期删除策略的**缺点**：

- 内存清理方面没有定时删除效果好，同时没有惰性删除使用的系统资源少。
- 难以确定删除操作执行的时长和频率。如果执行的太频繁，定期删除策略变得和定时删除策略一样，对CPU不友好；如果执行的太少，那又和惰性删除一样了，过期 key 占用的内存不会及时得到释放。

#### Redis 过期删除策略是什么？

前面介绍了三种过期删除策略，每一种都有优缺点，仅使用某一个策略都不能满足实际需求。

所以， **Redis 选择「惰性删除+定期删除」这两种策略配和使用**，以求在合理使用 CPU 时间和避免内存浪费之间取得平衡。

> Redis 是怎么实现惰性删除的？

Redis 的惰性删除策略由 db.c 文件中的 `expireIfNeeded` 函数实现，代码如下：

```c
int expireIfNeeded(redisDb *db, robj *key) {
    // 判断 key 是否过期
    if (!keyIsExpired(db,key)) return 0;
    ....
    /* 删除过期键 */
    ....
    // 如果 server.lazyfree_lazy_expire 为 1 表示异步删除，反之同步删除；
    return server.lazyfree_lazy_expire ? dbAsyncDelete(db,key) :
                                         dbSyncDelete(db,key);
}
```

Redis 在访问或者修改 key 之前，都会调用 expireIfNeeded 函数对其进行检查，检查 key 是否过期：

- 如果过期，则删除该 key，至于选择异步删除，还是选择同步删除，根据 `lazyfree_lazy_expire` 参数配置决定（Redis 4.0版本开始提供参数），然后返回 null 客户端；
- 如果没有过期，不做任何处理，然后返回正常的键值对给客户端；

惰性删除的流程图如下：

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/过期策略/惰性删除.jpg" alt="img" style="zoom:40%;" />

> Redis 是怎么实现定期删除的？

再回忆一下，定期删除策略的做法：**每隔一段时间「随机」从数据库中取出一定数量的 key 进行检查，并删除其中的过期key。**

*1、这个间隔检查的时间是多长呢？*

在 Redis 中，**默认每秒进行 10 次过期检查一次数据库**，此配置可通过 Redis 的配置文件 redis.conf 进行配置，配置键为 hz 它的默认值是 hz 10。

特别强调下，每次检查数据库并不是遍历过期字典中的所有 key，而是从数据库中**随机抽取一定数量的 key** 进行过期检查。

*2、随机抽查的数量是多少呢？*

我查了下源码，定期删除的实现在 expire.c 文件下的 `activeExpireCycle` 函数中，其中随机抽查的数量由 `ACTIVE_EXPIRE_CYCLE_LOOKUPS_PER_LOOP` 定义的，它是写死在代码中的，**数值是 20**。

也就是说，数据库每轮抽查时，会随机选择 20 个 key 判断是否过期。

接下来，详细说说 Redis 的定期删除的流程：

1. 从过期字典中随机抽取 20 个 key；
2. 检查这 20 个 key 是否过期，并删除已过期的 key；
3. **如果本轮检查的已过期 key 的数量，超过 5 个（20/4）**，也就是「已过期 key 的数量」占比「随机抽取 key 的数量」大于 25%，则**继续重复步骤 1**；如果已过期的 key 比例小于 25%，则停止继续删除过期 key，然后等待下一轮再检查。

可以看到，定期删除是一个循环的流程。

那 Redis 为了保证定期删除不会出现循环过度，导致线程卡死现象，为此增加了定期删除循环流程的时间上限，默认不会超过 25ms。

针对定期删除的流程，我写了个伪代码：

```c
do {
    //已过期的数量
    expired = 0；
    //随机抽取的数量
    num = 20;
    while (num--) {
        //1. 从过期字典中随机抽取 1 个 key
        //2. 判断该 key 是否过期，如果已过期则进行删除，同时对 expired++
    }
    
    // 超过时间限制则退出
    if (timelimit_exit) return;

  /* 如果本轮检查的已过期 key 的数量，超过 25%，则继续随机抽查，否则退出本轮检查 */
} while (expired > 20/4); 
```

定期删除的流程如下：

<img src="https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/过期策略/定时删除流程.jpg" alt="img" style="zoom:40%;" />

### 内存淘汰策略

前面说的过期删除策略，是删除已过期的 key，而**当 Redis 的运行内存已经超过 Redis 设置的最大内存之后，则会使用内存淘汰策略删除符合条件的 key**，以此来保障 Redis 高效的运行。

#### 如何设置 Redis 最大运行内存？

在配置文件 redis.conf 中，可以通过参数 `maxmemory <bytes>` 来设定最大运行内存，只有在 Redis 的运行内存达到了我们设置的最大运行内存，才会触发内存淘汰策略。 不同位数的操作系统，maxmemory 的默认值是不同的：

- 在 64 位操作系统中，maxmemory 的默认值是 0，表示没有内存大小限制，那么不管用户存放多少数据到 Redis 中，Redis 也不会对可用内存进行检查，直到 Redis 实例因内存不足而崩溃也无作为。
- 在 32 位操作系统中，maxmemory 的默认值是 3G，因为 32 位的机器最大只支持 4GB 的内存，而系统本身就需要一定的内存资源来支持运行，所以 32 位操作系统限制最大 3 GB 的可用内存是非常合理的，这样可以避免因为内存不足而导致 Redis 实例崩溃。

#### Redis 内存淘汰策略有哪些？

Redis 内存淘汰策略共有八种，这八种策略大体分为「不进行数据淘汰」和「进行数据淘汰」两类策略。

*1、不进行数据淘汰的策略*

**noeviction**（Redis3.0之后，默认的内存淘汰策略） ：它表示当运行内存超过最大设置内存时，不淘汰任何数据，而是**不再提供服务，直接返回错误**。

*2、进行数据淘汰的策略*

针对「进行数据淘汰」这一类策略，又可以细分为「在设置了过期时间的数据中进行淘汰」和「在所有数据范围内进行淘汰」这两类策略。

**在设置了过期时间的数据中**进行淘汰：

- **volatile-random**：**随机**淘汰设置了过期时间的**任意键值**；
- **volatile-ttl**：优先淘汰**更早过期**的键值。
- **volatile-lru**（Redis3.0 之前，默认的内存淘汰策略）：淘汰所有设置了过期时间的键值中，**最久未使用**的键值；
- **volatile-lfu**（Redis 4.0 后新增的内存淘汰策略）：淘汰所有设置了过期时间的键值中，**最少使用**的键值；

**在所有数据范围内**进行淘汰：

- **allkeys-random**：**随机**淘汰任意键值;
- **allkeys-lru**：淘汰整个键值中**最久未使用**的键值；
- **allkeys-lfu**（Redis 4.0 后新增的内存淘汰策略）：淘汰整个键值中**最少使用**的键值。

> 如何查看当前 Redis 使用的内存淘汰策略？

可以使用 `config get maxmemory-policy` 命令，来查看当前 Redis 的内存淘汰策略，命令如下：

```bash
127.0.0.1:6379> config get maxmemory-policy
1) "maxmemory-policy"
2) "noeviction"
```

可以看出，当前 Redis 使用的是 `noeviction` 类型的内存淘汰策略，它是 Redis 3.0 之后默认使用的内存淘汰策略，表示当运行内存超过最大设置内存时，不淘汰任何数据，但新增操作会报错。

> 如何修改 Redis 内存淘汰策略？

设置内存淘汰策略有两种方法：

- 方式一：通过“`config set maxmemory-policy <策略>`”命令设置。它的优点是设置之后立即生效，不需要重启 Redis 服务，缺点是重启 Redis 之后，设置就会失效。
- 方式二：通过修改 Redis 配置文件修改，设置“`maxmemory-policy <策略>`”，它的优点是重启 Redis 服务后配置不会丢失，缺点是必须重启 Redis 服务，设置才能生效。

#### LRU 算法和 LFU 算法有什么区别？

LFU 内存淘汰算法是 Redis 4.0 之后新增内存淘汰策略，那为什么要新增这个算法？那肯定是为了解决 LRU 算法的问题。

接下来，就看看这两个算法有什么区别？Redis 又是如何实现这两个算法的？

> 什么是 LRU 算法？

**LRU** 全称是 Least Recently Used 翻译为**最近最少使用**，会选择淘汰**最近最少使用的数据**。

传统 LRU 算法的实现是基于「链表」结构，链表中的元素按照操作顺序从前往后排列，最新操作的键会被移动到表头，当需要内存淘汰时，只需要删除链表尾部的元素即可，因为链表尾部的元素就代表最久未被使用的元素。

Redis 并没有使用这样的方式实现 LRU 算法，因为传统的 LRU 算法存在两个问题：

- 需要用链表管理所有的缓存数据，这会带来额外的空间开销；
- 当有数据被访问时，需要在链表上把该数据移动到头端，如果有大量数据被访问，就会带来很多链表移动操作，会很耗时，进而会降低 Redis 缓存性能。

> Redis 是如何实现 LRU 算法的？
>
> [参考](https://www.modb.pro/db/41628)

在Redis中，Redis的key的底层结构是 redisObject，**redisObject 中 lru:LRU_BITS 字段用于记录该key最近一次被访问时的Redis时钟 server.lruclock**（Redis在处理数据时，都会调用lookupKey方法用于更新该key的时钟）。

不太理解Redis时钟的同学，可以将其先简单理解成时间戳（不影响我们理解近似LRU算法原理），server.lruclock 实际是一个 24bit 的整数，默认是 Unix 时间戳对 2^24 取模的结果，其精度是毫秒。

**server.lruclock 的值是如何更新的呢**？

Redis启动时，initServer 方法中通过 aeCreateTimeEvent 将 serverCron 注册为时间事件（serverCron 是Redis中最核心的定时处理函数）， serverCron 中 则会 触发 更新Redis时钟的方法 server.lruclock = getLRUClock() 

当 mem_used > maxmemory 时，Redis通过 freeMemoryIfNeeded 方法完成数据淘汰。LRU策略淘汰核心逻辑在 evictionPoolPopulate（淘汰数据集合填充） 方法。

**Redis 近似LRU 淘汰策略逻辑：**

- **首次淘汰：随机抽样选出【最多N个数据】放入【待淘汰数据池 evictionPoolEntry】；**

- - 数据量N：由 redis.conf 配置的 maxmemory-samples 决定，默认值是5，配置为10将非常接近真实LRU效果，但是更消耗CPU；
  - samples：n.样本；v.抽样；

- **再次淘汰：随机抽样选出【最多N个数据】，只要数据比【待淘汰数据池 evictionPoolEntry】中的【任意一条】数据的 lru 小，则将该数据填充至 【待淘汰数据池】；**

- - evictionPoolEntry 的容容量是 EVPOOL_SIZE = 16；
  - 详见 源码 中 evictionPoolPopulate 方法的注释；

- **执行淘汰：挑选【待淘汰数据池】中 lru 最小的一条数据进行淘汰；**

Redis为了避免长时间或一直找不到足够的数据填充【待淘汰数据池】，代码里（dictGetSomeKeys 方法）强制写死了单次寻找数据的最大次数是 [maxsteps = count*10; ]，count 的值其实就是 maxmemory-samples。从这里我们也可以获得另一个重要信息：**单次获取的数据可能达不到 maxmemory-samples 个**。此外，如果Redis数据量（所有数据 或 有过期时间 的数据）本身就比 maxmemory-samples 小，那么 count 值等于 Redis 中数据量个数。

Redis 实现的 LRU 算法的优点：

- 不用为所有的数据维护一个大链表，节省了空间占用；
- 不用在每次数据访问时都移动链表项，提升了缓存的性能；

但是 LRU 算法有一个问题，**无法解决缓存污染问题**，比如应用一次读取了大量的数据，而这些数据只会被读取这一次，那么这些数据会留存在 Redis 缓存中很长一段时间，造成缓存污染。

因此，在 Redis 4.0 之后引入了 LFU 算法来解决这个问题。

> 什么是 LFU 算法？

LFU 全称是 Least Frequently Used 翻译为**最近最不常用的，**LFU 算法是**根据数据访问次数来淘汰数据**的，它的核心思想是“如果数据过去被访问多次，那么将来被访问的频率也更高”。

所以， LFU 算法会记录每个数据的访问次数。当一个数据被再次访问时，就会增加该数据的访问次数。这样就解决了偶尔被访问一次之后，数据留存在缓存中很长一段时间的问题，相比于 LRU 算法也更合理一些。

> Redis 是如何实现 LFU 算法的？

LFU 算法相比于 LRU 算法的实现，多记录了「数据的访问频次」的信息。Redis 对象的结构如下：

```c
typedef struct redisObject {
    ...
      
    // 24 bits，用于记录对象的访问信息
    unsigned lru:24;  
    ...
} robj;
```

Redis 对象头中的 lru 字段，在 LRU 算法下和 LFU 算法下使用方式并不相同。

**在 LRU 算法中**，Redis 对象头的 24 bits 的 lru 字段是用来记录 key 的访问时间戳，因此在 LRU 模式下，Redis可以根据对象头中的 lru 字段记录的值，来比较最后一次 key 的访问时间长，从而淘汰最久未被使用的 key。

**在 LFU 算法中**，Redis对象头的 24 bits 的 lru 字段被分成两段来存储，高 16bit 存储 ldt(Last Decrement Time)，低 8bit 存储 logc(Logistic Counter)。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/过期策略/lru字段.png)

- ldt 是用来记录 key 的访问时间戳；
- logc 是用来记录 key 的访问频次，它的值越小表示使用频率越低，越容易淘汰，每个新加入的 key 的logc 初始值为 5。

注意，logc 并不是单纯的访问次数，而是访问频次（访问频率），因为 **logc 会随时间推移而衰减的**。

在每次 key 被访问时，会先对 logc 做一个衰减操作，衰减的值跟前后访问时间的差距有关系，如果上一次访问的时间与这一次访问的时间差距很大，那么衰减的值就越大，这样实现的 LFU 算法是根据**访问频率**来淘汰数据的，而不只是访问次数。访问频率需要考虑 key 的访问是多长时间段内发生的。key 的先前访问距离当前时间越长，那么这个 key 的访问频率相应地也就会降低，这样被淘汰的概率也会更大。

对 logc 做完衰减操作后，就开始对 logc 进行增加操作，增加操作并不是单纯的 + 1，而是根据概率增加，如果 logc 越大的 key，它的 logc 就越难再增加。

所以，Redis 在访问 key 时，对于 logc 是这样变化的：

1. 先按照上次访问距离当前的时长，来对 logc 进行衰减；
2. 然后，再按照一定概率增加 logc 的值

redis.conf 提供了两个配置项，用于调整 LFU 算法从而控制 logc 的增长和衰减：

- `lfu-decay-time` 用于调整 logc 的衰减速度，它是一个以分钟为单位的数值，默认值为1，lfu-decay-time 值越大，衰减越慢；
- `lfu-log-factor` 用于调整 logc 的增长速度，lfu-log-factor 值越大，logc 增长越慢。

### 总结

Redis 使用的过期删除策略是「惰性删除+定期删除」，删除的对象是已过期的 key。

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/过期策略/过期删除策略.jpg)

内存淘汰策略是解决内存过大的问题，当 Redis 的运行内存超过最大运行内存时，就会触发内存淘汰策略，Redis 4.0 之后共实现了 8 种内存淘汰策略，我也对这 8 种的策略进行分类，如下：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/redis/过期策略/内存淘汰策略.jpg)

## Redis 配置文件

### 1.Redis文件目录

（1）全局搜索配置文件 redis.conf 所在目录，和Redis相关的参数配置都在此目录下设置。

```
[root@bigdata1 ~]# find / -name redis.conf
```

（2）命令所在目录

在Redis安装目录下的bin目录，例如：/opt/redis-5.0.7/bin

### 2.Redis配置字段解析

```
[root@bigdata1 config]# cat redis.conf 
```

```
  1 #是否在后台执行，yes：后台运行；no：不是后台运行
  2 daemonize yes
  3  
  4 #是否开启保护模式，默认开启。要是配置里没有指定bind和密码。开启该参数后，redis只会本地进行访问，拒绝外部访问。
  5 protected-mode yes
  6  
  7 #redis的进程文件
  8 pidfile /var/run/redis/redis-server.pid
  9  
 10 #redis监听的端口号。
 11 port 6379
 12  
 13 #此参数确定了TCP连接中已完成队列(完成三次握手之后)的长度， 当然此值必须不大于Linux系统定义的/proc/sys/net/core/somaxconn值，默认是511，而Linux的默认参数值是128。当系统并发量大并且客户端速度缓慢的时候，可以将这二个参数一起参考设定。该内核参数默认值一般是128，对于负载很大的服务程序来说大大的不够。一般会将它修改为2048或者更大。在/etc/sysctl.conf中添加:net.core.somaxconn = 2048，然后在终端中执行sysctl -p。
 14 tcp-backlog 511
 15  
 16 #指定 redis 只接收来自于该 IP 地址的请求，如果不进行设置，那么将处理所有请求
 17 #bind 127.0.0.1
 18 bind 0.0.0.0
 19  
 20 #配置unix socket来让redis支持监听本地连接。
 21 # unixsocket /var/run/redis/redis.sock
 22  
 23 #配置unix socket使用文件的权限
 24 # unixsocketperm 700
 25  
 26 # 此参数为设置客户端空闲超过timeout，服务端会断开连接，为0则服务端不会主动断开连接，不能小于0。
 27 timeout 0
 28  
 29 #tcp keepalive参数。如果设置不为0，就使用配置tcp的SO_KEEPALIVE值，使用keepalive有两个好处:检测挂掉的对端。降低中间设备出问题而导致网络看似连接却已经与对端端口的问题。在Linux内核中，设置了keepalive，redis会定时给对端发送ack。检测到对端关闭需要两倍的设置值。
 30 tcp-keepalive 0
 31  
 32 #指定了服务端日志的级别。级别包括：debug（很多信息，方便开发、测试），verbose（许多有用的信息，但是没有debug级别信息多），notice（适当的日志级别，适合生产环境），warn（只有非常重要的信息）
 33 loglevel notice
 34  
 35 #指定了记录日志的文件。空字符串的话，日志会打印到标准输出设备。后台运行的redis标准输出是/dev/null。
 36 logfile /var/log/redis/redis-server.log
 37  
 38 #是否打开记录syslog功能
 39 # syslog-enabled no
 40  
 41 #syslog的标识符。
 42 # syslog-ident redis
 43  
 44 #日志的来源、设备
 45 # syslog-facility local0
 46  
 47 #数据库的数量，默认使用的数据库是DB 0。可以通过SELECT命令选择一个db
 48 databases 16
 49  
 50 # redis是基于内存的数据库，可以通过设置该值定期写入磁盘。
 51 # 注释掉“save”这一行配置项就可以让保存数据库功能失效
 52 # 900秒（15分钟）内至少1个key值改变（则进行数据库保存--持久化） 
 53 # 300秒（5分钟）内至少10个key值改变（则进行数据库保存--持久化） 
 54 # 60秒（1分钟）内至少10000个key值改变（则进行数据库保存--持久化）
 55 save 900 1
 56 save 300 10
 57 save 60 10000
 58  
 59 #当RDB持久化出现错误后，是否依然进行继续进行工作，yes：不能进行工作，no：可以继续进行工作，可以通过info中的rdb_last_bgsave_status了解RDB持久化是否有错误
 60 stop-writes-on-bgsave-error yes
 61  
 62 #使用压缩rdb文件，rdb文件压缩使用LZF压缩算法，yes：压缩，但是需要一些cpu的消耗。no：不压缩，需要更多的磁盘空间
 63 rdbcompression yes
 64  
 65 #是否校验rdb文件。从rdb格式的第五个版本开始，在rdb文件的末尾会带上CRC64的校验和。这跟有利于文件的容错性，但是在保存rdb文件的时候，会有大概10%的性能损耗，所以如果你追求高性能，可以关闭该配置。
 66 rdbchecksum yes
 67  
 68 #rdb文件的名称
 69 dbfilename dump.rdb
 70  
 71 #数据目录，数据库的写入会在这个目录。rdb、aof文件也会写在这个目录
 72 dir /var/lib/redis
 73  
 74  
 75 ############### 主从复制 ###############
 76  
 77 #复制选项，slave复制对应的master。
 78 # slaveof <masterip> <masterport>
 79  
 80 #如果master设置了requirepass，那么slave要连上master，需要有master的密码才行。masterauth就是用来配置master的密码，这样可以在连上master后进行认证。
 81 # masterauth <master-password>
 82  
 83 #当从库同主机失去连接或者复制正在进行，从机库有两种运行方式：1) 如果slave-serve-stale-data设置为yes(默认设置)，从库会继续响应客户端的请求。2) 如果slave-serve-stale-data设置为no，除去INFO和SLAVOF命令之外的任何请求都会返回一个错误”SYNC with master in progress”。
 84 slave-serve-stale-data yes
 85  
 86 #作为从服务器，默认情况下是只读的（yes），可以修改成NO，用于写（不建议）。
 87 slave-read-only yes
 88  
 89 #是否使用socket方式复制数据。目前redis复制提供两种方式，disk和socket。如果新的slave连上来或者重连的slave无法部分同步，就会执行全量同步，master会生成rdb文件。有2种方式：disk方式是master创建一个新的进程把rdb文件保存到磁盘，再把磁盘上的rdb文件传递给slave。socket是master创建一个新的进程，直接把rdb文件以socket的方式发给slave。disk方式的时候，当一个rdb保存的过程中，多个slave都能共享这个rdb文件。socket的方式就的一个个slave顺序复制。在磁盘速度缓慢，网速快的情况下推荐用socket方式。
 90 repl-diskless-sync no
 91  
 92 #diskless复制的延迟时间，防止设置为0。一旦复制开始，节点不会再接收新slave的复制请求直到下一个rdb传输。所以最好等待一段时间，等更多的slave连上来。
 93 repl-diskless-sync-delay 5
 94  
 95 #slave根据指定的时间间隔向服务器发送ping请求。时间间隔可以通过 repl_ping_slave_period 来设置，默认10秒。
 96 # repl-ping-slave-period 10
 97  
 98 #复制连接超时时间。master和slave都有超时时间的设置。master检测到slave上次发送的时间超过repl-timeout，即认为slave离线，清除该slave信息。slave检测到上次和master交互的时间超过repl-timeout，则认为master离线。需要注意的是repl-timeout需要设置一个比repl-ping-slave-period更大的值，不然会经常检测到超时。
 99 # repl-timeout 60
100  
101 #是否禁止复制tcp链接的tcp nodelay参数，可传递yes或者no。默认是no，即使用tcp nodelay。如果master设置了yes来禁止tcp nodelay设置，在把数据复制给slave的时候，会减少包的数量和更小的网络带宽。但是这也可能带来数据的延迟。默认我们推荐更小的延迟，但是在数据量传输很大的场景下，建议选择yes。
102 repl-disable-tcp-nodelay no
103  
104 #复制缓冲区大小，这是一个环形复制缓冲区，用来保存最新复制的命令。这样在slave离线的时候，不需要完全复制master的数据，如果可以执行部分同步，只需要把缓冲区的部分数据复制给slave，就能恢复正常复制状态。缓冲区的大小越大，slave离线的时间可以更长，复制缓冲区只有在有slave连接的时候才分配内存。没有slave的一段时间，内存会被释放出来，默认1m。
105 # repl-backlog-size 5mb
106  
107 #master没有slave一段时间会释放复制缓冲区的内存，repl-backlog-ttl用来设置该时间长度。单位为秒。
108 # repl-backlog-ttl 3600
109  
110 #当master不可用，Sentinel会根据slave的优先级选举一个master。最低的优先级的slave，当选master。而配置成0，永远不会被选举。
111 slave-priority 100
112  
113 #redis提供了可以让master停止写入的方式，如果配置了min-slaves-to-write，健康的slave的个数小于N，mater就禁止写入。master最少得有多少个健康的slave存活才能执行写命令。这个配置虽然不能保证N个slave都一定能接收到master的写操作，但是能避免没有足够健康的slave的时候，master不能写入来避免数据丢失。设置为0是关闭该功能。
114 # min-slaves-to-write 3
115  
116 #延迟小于min-slaves-max-lag秒的slave才认为是健康的slave。
117 # min-slaves-max-lag 10
118  
119 # 设置1或另一个设置为0禁用这个特性。
120 # Setting one or the other to 0 disables the feature.
121 # By default min-slaves-to-write is set to 0 (feature disabled) and
122 # min-slaves-max-lag is set to 10.
123  
124  
125 ############### 安全相关 ###############
126  
127 #requirepass配置可以让用户使用AUTH命令来认证密码，才能使用其他命令。这让redis可以使用在不受信任的网络中。为了保持向后的兼容性，可以注释该命令，因为大部分用户也不需要认证。使用requirepass的时候需要注意，因为redis太快了，每秒可以认证15w次密码，简单的密码很容易被攻破，所以最好使用一个更复杂的密码。注意只有密码没有用户名。
128 # requirepass foobared
129  
130 #把危险的命令给修改成其他名称。比如CONFIG命令可以重命名为一个很难被猜到的命令，这样用户不能使用，而内部工具还能接着使用。
131 # rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
132  
133 #设置成一个空的值，可以禁止一个命令
134 # rename-command CONFIG ""
135  
136  
137 ############### 进程限制相关 ###############
138  
139 # 设置能连上redis的最大客户端连接数量。默认是10000个客户端连接。由于redis不区分连接是客户端连接还是内部打开文件或者和slave连接等，所以maxclients最小建议设置到32。如果超过了maxclients，redis会给新的连接发送’max number of clients reached’，并关闭连接。
140 # maxclients 10000
141  
142 #redis配置的最大内存容量。当内存满了，需要配合maxmemory-policy策略进行处理。注意slave的输出缓冲区是不计算在maxmemory内的。所以为了防止主机内存使用完，建议设置的maxmemory需要更小一些。
143 # maxmemory <bytes>
144  
145 #内存容量超过maxmemory后的处理策略。
146 #volatile-lru：利用LRU算法移除设置过过期时间的key。
147 #volatile-random：随机移除设置过过期时间的key。
148 #volatile-ttl：移除即将过期的key，根据最近过期时间来删除（辅以TTL）
149 #allkeys-lru：利用LRU算法移除任何key。
150 #allkeys-random：随机移除任何key。
151 #noeviction：不移除任何key，只是返回一个写错误。
152 #上面的这些驱逐策略，如果redis没有合适的key驱逐，对于写命令，还是会返回错误。redis将不再接收写请求，只接收get请求。写命令包括：set setnx setex append incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby getset mset msetnx exec sort。
153 # maxmemory-policy noeviction
154  
155 #lru检测的样本数。使用lru或者ttl淘汰算法，从需要淘汰的列表中随机选择sample个key，选出闲置时间最长的key移除。
156 # maxmemory-samples 5
157  
158  
159 ############### APPEND ONLY 持久化方式 ###############
160  
161 #默认redis使用的是rdb方式持久化，这种方式在许多应用中已经足够用了。但是redis如果中途宕机，会导致可能有几分钟的数据丢失，根据save来策略进行持久化，Append Only File是另一种持久化方式，可以提供更好的持久化特性。Redis会把每次写入的数据在接收后都写入 appendonly.aof 文件，每次启动时Redis都会先把这个文件的数据读入内存里，先忽略RDB文件。
162 appendonly no
163  
164 #aof文件名
165 appendfilename "appendonly.aof"
166  
167 #aof持久化策略的配置
168 #no表示不执行fsync，由操作系统保证数据同步到磁盘，速度最快。
169 #always表示每次写入都执行fsync，以保证数据同步到磁盘。
170 #everysec表示每秒执行一次fsync，可能会导致丢失这1s数据。
171 appendfsync everysec
172  
173 # 在aof重写或者写入rdb文件的时候，会执行大量IO，此时对于everysec和always的aof模式来说，执行fsync会造成阻塞过长时间，no-appendfsync-on-rewrite字段设置为默认设置为no。如果对延迟要求很高的应用，这个字段可以设置为yes，否则还是设置为no，这样对持久化特性来说这是更安全的选择。设置为yes表示rewrite期间对新写操作不fsync,暂时存在内存中,等rewrite完成后再写入，默认为no，建议yes。Linux的默认fsync策略是30秒。可能丢失30秒数据。
174 no-appendfsync-on-rewrite no
175  
176 #aof自动重写配置。当目前aof文件大小超过上一次重写的aof文件大小的百分之多少进行重写，即当aof文件增长到一定大小的时候Redis能够调用bgrewriteaof对日志文件进行重写。当前AOF文件大小是上次日志重写得到AOF文件大小的二倍（设置为100）时，自动启动新的日志重写过程。
177 auto-aof-rewrite-percentage 100
178 #设置允许重写的最小aof文件大小，避免了达到约定百分比但尺寸仍然很小的情况还要重写
179 auto-aof-rewrite-min-size 64mb
180  
181 #aof文件可能在尾部是不完整的，当redis启动的时候，aof文件的数据被载入内存。重启可能发生在redis所在的主机操作系统宕机后，尤其在ext4文件系统没有加上data=ordered选项（redis宕机或者异常终止不会造成尾部不完整现象。）出现这种现象，可以选择让redis退出，或者导入尽可能多的数据。如果选择的是yes，当截断的aof文件被导入的时候，会自动发布一个log给客户端然后load。如果是no，用户必须手动redis-check-aof修复AOF文件才可以。
182 aof-load-truncated yes
183  
184  
185 ############### LUA SCRIPTING ###############
186  
187 # 如果达到最大时间限制（毫秒），redis会记个log，然后返回error。当一个脚本超过了最大时限。只有SCRIPT KILL和SHUTDOWN NOSAVE可以用。第一个可以杀没有调write命令的东西。要是已经调用了write，只能用第二个命令杀。
188 lua-time-limit 5000
189  
190  
191 ############### 集群相关 ###############
192  
193 #集群开关，默认是不开启集群模式。
194 # cluster-enabled yes
195  
196 #集群配置文件的名称，每个节点都有一个集群相关的配置文件，持久化保存集群的信息。这个文件并不需要手动配置，这个配置文件有Redis生成并更新，每个Redis集群节点需要一个单独的配置文件，请确保与实例运行的系统中配置文件名称不冲突
197 # cluster-config-file nodes-6379.conf
198  
199 #节点互连超时的阀值。集群节点超时毫秒数
200 # cluster-node-timeout 15000
201  
202 #在进行故障转移的时候，全部slave都会请求申请为master，但是有些slave可能与master断开连接一段时间了，导致数据过于陈旧，这样的slave不应该被提升为master。该参数就是用来判断slave节点与master断线的时间是否过长。判断方法是：
203 #比较slave断开连接的时间和(node-timeout * slave-validity-factor) + repl-ping-slave-period
204 #如果节点超时时间为三十秒, 并且slave-validity-factor为10,假设默认的repl-ping-slave-period是10秒，即如果超过310秒slave将不会尝试进行故障转移 
205 # cluster-slave-validity-factor 10
206  
207 #master的slave数量大于该值，slave才能迁移到其他孤立master上，如这个参数若被设为2，那么只有当一个主节点拥有2 个可工作的从节点时，它的一个从节点会尝试迁移。
208 # cluster-migration-barrier 1
209  
210 #默认情况下，集群全部的slot有节点负责，集群状态才为ok，才能提供服务。设置为no，可以在slot没有全部分配的时候提供服务。不建议打开该配置，这样会造成分区的时候，小分区的master一直在接受写请求，而造成很长时间数据不一致。
211 # cluster-require-full-coverage yes
212  
213  
214 ############### SLOW LOG 慢查询日志 ###############
215  
216 ###slog log是用来记录redis运行中执行比较慢的命令耗时。当命令的执行超过了指定时间，就记录在slow log中，slog log保存在内存中，所以没有IO操作。
217 #执行时间比slowlog-log-slower-than大的请求记录到slowlog里面，单位是微秒，所以1000000就是1秒。注意，负数时间会禁用慢查询日志，而0则会强制记录所有命令。
218 slowlog-log-slower-than 10000
219  
220 #慢查询日志长度。当一个新的命令被写进日志的时候，最老的那个记录会被删掉。这个长度没有限制。只要有足够的内存就行。你可以通过 SLOWLOG RESET 来释放内存。
221 slowlog-max-len 128
222  
223 ############### 延迟监控 ###############
224 #延迟监控功能是用来监控redis中执行比较缓慢的一些操作，用LATENCY打印redis实例在跑命令时的耗时图表。只记录大于等于下边设置的值的操作。0的话，就是关闭监视。默认延迟监控功能是关闭的，如果你需要打开，也可以通过CONFIG SET命令动态设置。
225 latency-monitor-threshold 0
226  
227 ############### EVENT NOTIFICATION 订阅通知 ###############
228 #键空间通知使得客户端可以通过订阅频道或模式，来接收那些以某种方式改动了 Redis 数据集的事件。因为开启键空间通知功能需要消耗一些 CPU ，所以在默认配置下，该功能处于关闭状态。
229 #notify-keyspace-events 的参数可以是以下字符的任意组合，它指定了服务器该发送哪些类型的通知：
230 ##K 键空间通知，所有通知以 __keyspace@__ 为前缀
231 ##E 键事件通知，所有通知以 __keyevent@__ 为前缀
232 ##g DEL 、 EXPIRE 、 RENAME 等类型无关的通用命令的通知
233 ##$ 字符串命令的通知
234 ##l 列表命令的通知
235 ##s 集合命令的通知
236 ##h 哈希命令的通知
237 ##z 有序集合命令的通知
238 ##x 过期事件：每当有过期键被删除时发送
239 ##e 驱逐(evict)事件：每当有键因为 maxmemory 政策而被删除时发送
240 ##A 参数 g$lshzxe 的别名
241 #输入的参数中至少要有一个 K 或者 E，否则的话，不管其余的参数是什么，都不会有任何 通知被分发。详细使用可以参考http://redis.io/topics/notifications
243 notify-keyspace-events ""
244  
245 ############### ADVANCED CONFIG 高级配置 ###############
246 #数据量小于等于hash-max-ziplist-entries的用ziplist，大于hash-max-ziplist-entries用hash
247 hash-max-ziplist-entries 512
248 #value大小小于等于hash-max-ziplist-value的用ziplist，大于hash-max-ziplist-value用hash。
249 hash-max-ziplist-value 64
250  
251 #数据量小于等于list-max-ziplist-entries用ziplist，大于list-max-ziplist-entries用list。
252 list-max-ziplist-entries 512
253 #value大小小于等于list-max-ziplist-value的用ziplist，大于list-max-ziplist-value用list。
254 list-max-ziplist-value 64
255  
256 #数据量小于等于set-max-intset-entries用iniset，大于set-max-intset-entries用set。
257 set-max-intset-entries 512
258  
259 #数据量小于等于zset-max-ziplist-entries用ziplist，大于zset-max-ziplist-entries用zset。
260 zset-max-ziplist-entries 128
261 #value大小小于等于zset-max-ziplist-value用ziplist，大于zset-max-ziplist-value用zset。
262 zset-max-ziplist-value 64
263  
264 #value大小小于等于hll-sparse-max-bytes使用稀疏数据结构（sparse），大于hll-sparse-max-bytes使用稠密的数据结构（dense）。一个比16000大的value是几乎没用的，建议的value大概为3000。如果对CPU要求不高，对空间要求较高的，建议设置到10000左右。
265 hll-sparse-max-bytes 3000
266  
267 #Redis将在每100毫秒时使用1毫秒的CPU时间来对redis的hash表进行重新hash，可以降低内存的使用。当你的使用场景中，有非常严格的实时性需要，不能够接受Redis时不时的对请求有2毫秒的延迟的话，把这项配置为no。如果没有这么严格的实时性要求，可以设置为yes，以便能够尽可能快的释放内存。
268 activerehashing yes
269  
270 ##对客户端输出缓冲进行限制可以强迫那些不从服务器读取数据的客户端断开连接，用来强制关闭传输缓慢的客户端。
271 #对于normal client，第一个0表示取消hard limit，第二个0和第三个0表示取消soft limit，normal client默认取消限制，因为如果没有寻问，他们是不会接收数据的。
272 client-output-buffer-limit normal 0 0 0
273 #对于slave client和MONITER client，如果client-output-buffer一旦超过256mb，又或者超过64mb持续60秒，那么服务器就会立即断开客户端连接。
274 client-output-buffer-limit slave 256mb 64mb 60
275 #对于pubsub client，如果client-output-buffer一旦超过32mb，又或者超过8mb持续60秒，那么服务器就会立即断开客户端连接。
276 client-output-buffer-limit pubsub 32mb 8mb 60
277  
278 #redis执行任务的频率为1s除以hz。
279 hz 10
280  
281 #在aof重写的时候，如果打开了aof-rewrite-incremental-fsync开关，系统会每32MB执行一次fsync。这对于把文件写入磁盘是有帮助的，可以避免过大的延迟峰值。
282 aof-rewrite-incremental-fsync yes
```

### 3.查看内存使用情况

进入Redis交互界面

```
[root@bigdata1 bin]# ./redis-cli
127.0.0.1:6379> AUTH [password]
OK
127.0.0.1:6379> INFO1.2.3.4.
```

## Redis 持久化

### AOF 日志

试想一下，如果 Redis 每执行一条**写操作**命令，就把该命令以**追加的方式**写入到一个文件里，然后重启 Redis 的时候，先去读取这个文件里的命令，并且执行它，这不就相当于恢复了缓存数据了吗？

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/6f0ab40396b7fc2c15e6f4487d3a0ad7.png)

这种保存写操作命令到日志的持久化方式，就是 Redis 里的 **AOF(Append Only File)** 持久化功能，**注意只会记录写操作命令，读操作命令是不会被记录的**，因为没意义。

在 Redis 中 AOF 持久化功能**默认是不开启的**，需要我们修改 `redis.conf` 配置文件中的以下参数：

![img](https://img-blog.csdnimg.cn/img_convert/0e2d081af084c41802c7b5de8aa41bd4.png)

AOF 日志文件其实就是普通的文本，我们可以通过 `cat` 命令查看里面的内容，不过里面的内容如果不知道一定的规则的话，可能会看不懂。

我这里以「*set name xiaolin*」命令作为例子，Redis 执行了这条命令后，记录在 AOF 日志里的内容如下图：

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/337021a153944fd0f964ca834e34d0f2.png)

我这里给大家解释下。

「`*3`」表示当前命令有三个部分，每部分都是以「`$+数字`」开头，后面紧跟着具体的命令、键或值。然后，这里的「`数字`」**表示这部分中的命令、键或值一共有多少字节**。例如，「`$3 set`」表示这部分有 3 个字节，也就是「`set`」命令这个字符串的长度。

不知道大家注意到没有，Redis 是**先执行写操作命令后，才将该命令记录到 AOF 日志**里的，这么做其实有两个好处。

第一个好处，**避免额外的检查开销。**

因为如果先将写操作命令记录到 AOF 日志里，再执行该命令的话，如果当前的命令语法有问题，那么如果不进行命令语法检查，该错误的命令记录到 AOF 日志里后，Redis 在使用日志恢复数据时，就可能会出错。

而如果先执行写操作命令再记录日志的话，只有在该命令执行成功后，才将命令记录到 AOF 日志里，这样就不用额外的检查开销，保证记录在 AOF 日志里的命令都是可执行并且正确的。

第二个好处，**不会阻塞当前写操作命令的执行**，因为当写操作命令执行成功后，才会将命令记录到 AOF 日志。

当然，AOF 持久化功能也不是没有潜在风险。

第一个风险，执行写操作命令和记录日志是两个过程，那当 Redis 在还没来得及将命令写入到硬盘时，服务器发生宕机了，这个数据就会有**丢失的风险**。

第二个风险，前面说道，由于写操作命令执行成功后才记录到 AOF 日志，所以不会阻塞当前写操作命令的执行，但是**可能会给「下一个」命令带来阻塞风险**。

因为将命令写入到日志的这个操作也是在主进程完成的（执行命令也是在主进程），也就是说这两个操作是同步的。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/28afd536c57a46447ddab0a2062abe84.png)

如果在将日志内容写入到硬盘时，服务器的硬盘的 I/O 压力太大，就会导致写硬盘的速度很慢，进而阻塞住了，也就会导致后续的命令无法执行。

认真分析一下，其实这两个风险都有一个共性，都跟「 AOF 日志写回硬盘的时机」有关。

#### 三种写回策略

Redis 写入 AOF 日志的过程，如下图：

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/4eeef4dd1bedd2ffe0b84d4eaa0dbdea.png)

我先来具体说说：

1. Redis 执行完写操作命令后，会将命令追加到 `server.aof_buf` 缓冲区；
2. 然后通过 write() 系统调用，将 aof_buf 缓冲区的数据写入到 AOF 文件，此时数据并没有写入到硬盘，而是拷贝到了内核缓冲区 page cache，等待内核将数据写入硬盘；
3. 具体内核缓冲区的数据什么时候写入到硬盘，由内核决定。

Redis 提供了 3 种写回硬盘的策略，控制的就是上面说的第三步的过程。

在 `redis.conf` 配置文件中的 `appendfsync` 配置项可以有以下 3 种参数可填：

- **Always**，这个单词的意思是「总是」，所以它的意思是每次写操作命令执行完后，**同步将 AOF 日志数据写回硬盘**；
- **Everysec**，这个单词的意思是「每秒」，所以它的意思是每次写操作命令执行完后，先将命令写入到 AOF 文件的内核缓冲区，然后**每隔一秒将缓冲区里的内容写回到硬盘**；
- **No**，意味着**不由 Redis 控制写回硬盘的时机**，转交给操作系统控制写回的时机，也就是每次写操作命令执行完后，先将命令写入到 AOF 文件的内核缓冲区，再由操作系统决定何时将缓冲区内容写回硬盘。

这 3 种写回策略都无法能完美解决「主进程阻塞」和「减少数据丢失」的问题，因为两个问题是对立的，偏向于一边的话，就会要牺牲另外一边，原因如下：

- Always 策略的话，可以最大程度保证数据不丢失，但是由于它每执行一条写操作命令就同步将 AOF 内容写回硬盘，所以是不可避免会影响主进程的性能；
- No 策略的话，是交由操作系统来决定何时将 AOF 日志内容写回硬盘，相比于 Always 策略性能较好，但是操作系统写回硬盘的时机是不可预知的，如果 AOF 日志内容没有写回硬盘，一旦服务器宕机，就会丢失不定数量的数据。
- Everysec 策略的话，是折中的一种方式，避免了 Always 策略的性能开销，也比 No 策略更能避免数据丢失，当然如果上一秒的写操作命令日志没有写回到硬盘，发生了宕机，这一秒内的数据自然也会丢失。

大家根据自己的业务场景进行选择：

- 如果要高性能，就选择 No 策略；
- 如果要高可靠，就选择 Always 策略；
- 如果允许数据丢失一点，但又想性能高，就选择 Everysec 策略。

我也把这 3 个写回策略的优缺点总结成了一张表格：

![img](https://img-blog.csdnimg.cn/img_convert/98987d9417b2bab43087f45fc959d32a.png)

大家知道这三种策略是怎么实现的吗？

深入到源码后，你就会发现这三种策略只是在控制 `fsync()` 函数的调用时机。

当应用程序向文件写入数据时，内核通常先将数据复制到内核缓冲区中，然后排入队列，然后由内核决定何时写入硬盘。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/f64829ffc2e9e006b090f9aae51035ee.png)

如果想要应用程序向文件写入数据后，能立马将数据同步到硬盘，就可以调用 `fsync()` 函数，这样内核就会将内核缓冲区的数据直接写入到硬盘，等到硬盘写操作完成后，该函数才会返回。

- Always 策略就是每次写入 AOF 文件数据后，就执行 fsync() 函数；
- Everysec 策略就会创建一个异步任务来执行 fsync() 函数；
- No 策略就是永不执行 fsync() 函数;

#### AOF 重写机制

AOF 日志是一个文件，随着执行的写操作命令越来越多，文件的大小会越来越大。

如果当 AOF 日志文件过大就会带来性能问题，比如重启 Redis 后，需要读 AOF 文件的内容以恢复数据，如果文件过大，整个恢复的过程就会很慢。

所以，Redis 为了避免 AOF 文件越写越大，提供了 **AOF 重写机制**，当 AOF 文件的大小超过所设定的阈值后，Redis 就会启用 AOF 重写机制，来压缩 AOF 文件。

AOF 重写机制是在重写时，**读取当前数据库**中的所有键值对，然后将**每一个键值对用一条命令记录到「新的 AOF 文件」**，等到全部记录完后，就将新的 AOF 文件替换掉现有的 AOF 文件。

举个例子，在没有使用重写机制前，假设前后执行了「*set name xiaolin*」和「*set name xiaolincoding*」这两个命令的话，就会将这两个命令记录到 AOF 文件。

![img](https://img-blog.csdnimg.cn/img_convert/723d6c580c05400b3841bc69566dd61b.png)

但是在使用重写机制后，就会读取 name 最新的 value（键值对） ，然后用一条 「set name xiaolincoding」命令记录到新的 AOF 文件，之前的第一个命令就没有必要记录了，因为它属于「历史」命令，没有作用了。这样一来，**一个键值对在重写日志中只用一条命令**就行了。

重写工作完成后，就会将新的 AOF 文件覆盖现有的 AOF 文件，这就相当于压缩了 AOF 文件，使得 AOF 文件体积变小了。

然后，在通过 AOF 日志恢复数据时，只用执行这条命令，就可以直接完成这个键值对的写入了。

所以，重写机制的妙处在于，尽管某个键值对被多条写命令反复修改，**最终也只需要根据这个「键值对」当前的最新状态，然后用一条命令去记录键值对**，代替之前记录这个键值对的多条命令，这样就减少了 AOF 文件中的命令数量。最后在重写工作完成后，将新的 AOF 文件覆盖现有的 AOF 文件。

这里说一下为什么重写 AOF 的时候，不直接复用现有的 AOF 文件，而是先写到新的 AOF 文件再覆盖过去。

因为**如果 AOF 重写过程中失败了，现有的 AOF 文件就会造成污染**，可能无法用于恢复使用。

所以 AOF 重写过程，先重写到新的 AOF 文件，重写失败的话，就直接删除这个文件就好，不会对现有的 AOF 文件造成影响。

#### AOF 后台重写

写入 AOF 日志的操作虽然是在主进程完成的，因为它写入的内容不多，所以一般不太影响命令的操作。

但是在触发 AOF 重写时，比如当 AOF 文件大于 64M 时，就会对 AOF 文件进行重写，这时是需要读取所有缓存的键值对数据，并为每个键值对生成一条命令，然后将其写入到新的 AOF 文件，重写完后，就把现在的 AOF 文件替换掉。

这个过程其实是很耗时的，所以重写的操作不能放在主进程里。

所以，Redis 的**重写 AOF 过程是由后台子进程 bgrewriteaof 来完成的**，这么做可以达到两个好处：

- 子进程进行 AOF 重写期间，主进程可以继续处理命令请求，从而避免阻塞主进程；
- 子进程带有主进程的数据副本（*数据副本怎么产生的后面会说*），这里使用子进程而不是线程，因为如果是使用线程，多线程之间会共享内存，那么**在修改共享内存数据的时候，需要通过加锁来保证数据的安全，而这样就会降低性能**。而使用子进程，创建子进程时，父子进程是共享内存数据的，不过这个**共享的内存只能以只读的方式**，而当父子进程任意一方修改了该共享内存，就会发生「写时复制」，于是父子进程就有了独立的数据副本，就不用加锁来保证数据安全。

子进程是怎么拥有主进程一样的数据副本的呢？

主进程在通过 `fork` 系统调用生成 bgrewriteaof 子进程时，操作系统会把主进程的「**页表**」复制一份给子进程，这个页表记录着虚拟地址和物理地址映射关系，而不会复制物理内存，也就是说，**两者的虚拟空间不同，但其对应的物理空间是同一个**。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/5a1f2a90b5f3821c19bea3b7a5f27fa1.png)

这样一来，子进程就共享了父进程的物理内存数据了，这样能够**节约物理内存资源**，页表对应的页表项的属性会标记该物理内存的权限为**只读**。

不过，当父进程或者子进程在向这个内存发起写操作时，CPU 就会触发**缺页中断**，这个缺页中断是由于违反权限导致的，然后操作系统会在「缺页异常处理函数」里进行**物理内存的复制**，并重新设置其内存映射关系，将父子进程的内存读写权限设置为**可读写**，最后才会对内存进行写操作，这个过程被称为「**写时复制(Copy On Write)**」。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/d4cfac545377b54dd035c775603b4936.png)

写时复制顾名思义，**在发生写操作的时候，操作系统才会去复制物理内存**，这样是为了防止 fork 创建子进程时，由于物理内存数据的复制时间过长而导致父进程长时间阻塞的问题。

当然，操作系统复制父进程页表的时候，父进程也是阻塞中的，不过页表的大小相比实际的物理内存小很多，所以通常复制页表的过程是比较快的。

不过，如果父进程的内存数据非常大，那自然页表也会很大，这时父进程在通过 fork 创建子进程的时候，阻塞的时间也越久。

所以，有两个阶段会导致阻塞父进程：

- **创建子进程的途中**，由于要复制父进程的页表等数据结构，阻塞的时间跟页表的大小有关，页表越大，阻塞的时间也越长；
- **创建完子进程后**，如果子进程或者父进程**修改了共享数据**，就会发生写时复制，这期间会拷贝物理内存，如果内存越大，自然阻塞的时间也越长；

触发重写机制后，主进程就会创建重写 AOF 的子进程，此时父子进程共享物理内存，重写子进程只会对这个内存进行只读，重写 AOF 子进程会读取数据库里的所有数据，并逐一把内存数据的键值对转换成一条命令，再将命令记录到重写日志（新的 AOF 文件）。

但是子进程重写过程中，主进程依然可以正常处理命令。

如果此时**主进程修改了已经存在 key-value，就会发生写时复制，注意这里只会复制主进程修改的物理内存数据，没修改物理内存还是与子进程共享的**。

所以如果这个阶段修改的是一个 bigkey，也就是数据量比较大的 key-value 的时候，这时复制的物理内存数据的过程就会比较耗时，有阻塞主进程的风险。

还有个问题，重写 AOF 日志过程中，如果主进程修改了已经存在 key-value，此时这个 key-value 数据在子进程的内存数据就跟主进程的内存数据不一致了，这时要怎么办呢？

为了解决这种数据不一致问题，Redis 设置了一个 **AOF 重写缓冲区**，这个缓冲区在创建 bgrewriteaof 子进程之后开始使用。

在重写 AOF 期间，当 Redis 执行完一个写命令之后，它会**同时将这个写命令写入到 「AOF 缓冲区」和 「AOF 重写缓冲区」**。

![在这里插入图片描述](https://img-blog.csdnimg.cn/202105270918298.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0ODI3Njc0,size_16,color_FFFFFF,t_70#pic_center)

也就是说，在 bgrewriteaof 子进程执行 AOF 重写期间，主进程需要执行以下三个工作:

- 执行客户端发来的命令；
- 将执行后的写命令追加到 「AOF 缓冲区」；
- 将执行后的写命令追加到 「AOF 重写缓冲区」；

当子进程完成 AOF 重写工作（*扫描数据库中所有数据，逐一把内存数据的键值对转换成一条命令，再将命令记录到重写日志*）后，会向主进程发送一条信号，信号是进程间通讯的一种方式，且是异步的。

主进程收到该信号后，会调用一个信号处理函数，该函数主要做以下工作：

- 将 **AOF 重写缓冲区中的所有内容追加到新的 AOF 的文件**中，**使得新旧两个 AOF 文件所保存的数据库状态一致**；
- 新的 AOF 的文件进行改名，覆盖现有的 AOF 文件。

信号函数执行完后，主进程就可以继续像往常一样处理命令了。

在整个 AOF 后台重写过程中，除了发生写时复制会对主进程造成阻塞，还有信号处理函数执行时也会对主进程造成阻塞，在其他时候，AOF 后台重写都不会阻塞主进程。

#### 总结

Redis 持久化技术中的 AOF 方法，这个方法是每执行一条写操作命令，就**将该命令以追加的方式写入到 AOF 文件，然后在恢复时，以逐一执行命令的方式来进行数据恢复**。

Redis 提供了三种将 AOF 日志写**回硬盘的策略，分别是 Always、Everysec 和 No**，这三种策略在可靠性上是从高到低，而在性能上则是从低到高。

随着执行的命令越多，AOF 文件的体积自然也会越来越大，为了避免日志文件过大， Redis 提供了 **AOF 重写机制，它会直接扫描数据中所有的键值对数据，然后为每一个键值对生成一条写操作命令**，接着将该命令写入到新的 AOF 文件，重写完成后，就替换掉现有的 AOF 日志。重写的过程是由后台子进程完成的，这样可以使得主进程可以继续正常处理命令。

用 AOF 日志的方式来恢复数据其实是很慢的，因为 Redis 执行命令由单线程负责的，而 AOF 日志恢复数据的方式是顺序执行日志里的每一条命令，如果 AOF 日志很大，这个「重放」的过程就会很慢了。

### RDB 快照

虽说 Redis 是内存数据库，但是它为数据的持久化提供了两个技术。

分别是「 AOF 日志和 RDB 快照」。

这两种技术都会用各用一个日志文件来记录信息，但是记录的内容是不同的。

- AOF 文件的内容是操作命令；
- **RDB 文件的内容是二进制数据**。

关于 AOF 持久化的原理我在上一篇已经介绍了，今天主要讲下 **RDB 快照**。

所谓的快照，就是记录某一个瞬间东西，比如当我们给风景拍照时，那一个瞬间的画面和信息就记录到了一张照片。

所以，**RDB 快照就是记录某一个瞬间的内存数据，记录的是实际数据**，而 AOF 文件记录的是命令操作的日志，而不是实际的数据。

因此在 Redis 恢复数据时， **RDB 恢复数据的效率会比 AOF 高些，**因为直接将 RDB 文件读入内存就可以，不需要像 AOF 那样还需要额外执行操作命令的步骤才能恢复数据。

接下来，就来具体聊聊 RDB 快照 。

#### 快照怎么用？

要熟悉一个东西，先看看怎么用是比较好的方式。

两种触发方式：

- 主动触发（ `save`  、`bgsave`）
- 自动执行（在一定时间内至少多少次修改）

##### **主动触发**

Redis 提供了两个命令来生成 RDB 文件，分别是 `save` 和 `bgsave`，他们的区别就在于是否在「主线程」里执行：

- 执行了 save 命令，就会在主线程生成 RDB 文件，由于和执行操作命令在同一个线程，所以如果写入 RDB 文件的时间太长，**会阻塞主线程**；
- 执行了 bgsave 命令，会创建一个子进程来生成 RDB 文件，这样可以**避免主线程的阻塞**；

RDB 文件的加载工作是在服务器启动时**自动执行**的，Redis 并没有提供专门用于加载 RDB 文件的命令。

##### 被动触发

Redis 还可以通过配置文件的选项来实现每隔一段时间自动执行一次 bgsave 命令，默认会提供以下配置：

```text
save 900 1
save 300 10
save 60 10000
```

别看选项名叫 save，实际上执行的是 bgsave 命令，也就是会创建子进程来生成 RDB 快照文件。

只要满足上面条件的任意一个，就会执行 bgsave，它们 的意思分别是：

- 900 秒之内，对数据库进行了至少 1 次修改；
- 300 秒之内，对数据库进行了至少 10 次修改；
- 60 秒之内，对数据库进行了至少 10000 次修改。

这里提一点，Redis 的快照是**全量快照**，也就是说每次执行快照，都是**把内存中的「所有数据」都记录到磁盘中**。

所以可以认为，执行快照是一个比较重的操作，如果频率太频繁，可能会对 Redis 性能产生影响。如果频率太低，服务器故障时，丢失的数据会更多。

通常可能设置**至少 5 分钟才保存一次快照**，这时如果 Redis 出现宕机等情况，则意味着最多可能丢失 5 分钟数据。

这就是 RDB 快照的缺点，在服务器发生故障时，丢失的数据会比 AOF 持久化的方式更多，因为 RDB 快照是全量快照的方式，因此执行的频率不能太频繁，否则会影响 Redis 性能，而 AOF 日志可以以秒级的方式记录操作命令，所以丢失的数据就相对更少。

#### 执行快照时，数据能被修改吗？

那问题来了，执行 bgsave 过程中，由于是交给子进程来构建 RDB 文件，主线程还是可以继续工作的，此时主线程可以修改数据吗？

如果不可以修改数据的话，那这样性能一下就降低了很多。如果可以修改数据，又是如何做到到呢？

直接说结论吧，执行 bgsave 过程中，Redis 依然**可以继续处理操作命令**的，也就是数据是能被修改的。

那具体如何做到到呢？关键的技术就在于**写时复制技术（Copy-On-Write, COW）。**

执行 bgsave 命令的时候，会**通过 `fork()` 创建子进程**，此时子进程和父进程是共享同一片内存数据的，因为创建子进程的时候，会复制父进程的页表，但是页表指向的物理内存还是一个。

![图片](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/c34a9d1f58d602ff1fe8601f7270baa7.png)

**只有在发生修改内存数据的情况时，物理内存才会被复制一份**。

![图片](https://img-blog.csdnimg.cn/img_convert/ebd620db8a1af66fbeb8f4d4ef6adc68.png)

这样的目的是为了减少创建子进程时的性能损耗，从而加快创建子进程的速度，毕竟创建子进程的过程中，是会阻塞主线程的。

所以，创建 bgsave 子进程后，由于共享父进程的所有内存数据，于是就可以直接读取主线程（父进程）里的内存数据，并将数据写入到 RDB 文件。

当主线程（父进程）对这些共享的内存数据也都是只读操作，那么，主线程（父进程）和 bgsave 子进程相互不影响。

但是，如果主线程（父进程）要**修改共享数据里的某一块数据**（比如键值对 `A`）时，就会发生写时复制，于是这块数据的**物理内存就会被复制一份（键值对 `A'`）**，然后**主线程在这个数据副本（键值对 `A'`）进行修改操作**。与此同时，**bgsave 子进程可以继续把原来的数据（键值对 `A`）写入到 RDB 文件**。

就是这样，Redis 使用 bgsave 对当前内存中的所有数据做快照，这个操作是由 bgsave 子进程在后台完成的，执行时不会阻塞主线程，这就使得主线程同时可以修改数据。

细心的同学，肯定发现了，bgsave 快照过程中，如果主线程修改了共享数据，**发生了写时复制后，RDB 快照保存的是原本的内存数据**，而主线程刚修改的数据，是没办法在这一时间写入 RDB 文件的，**只能交由下一次的 bgsave 快照**。

所以 Redis 在使用 bgsave 快照过程中，如果主线程修改了内存数据，不管是否是共享的内存数据，RDB 快照都无法写入主线程刚修改的数据，因为此时主线程（父进程）的内存数据和子进程的内存数据已经分离了，子进程写入到 RDB 文件的内存数据只能是原本的内存数据。

如果系统恰好在 RDB 快照文件创建完毕后崩溃了，那么 **Redis 将会丢失主线程在快照期间修改的数据**。

另外，写时复制的时候会出现这么个极端的情况。

在 Redis 执行 RDB 持久化期间，刚 fork 时，主进程和子进程共享同一物理内存，但是途中主进程处理了写操作，修改了共享内存，于是当前被修改的数据的物理内存就会被复制一份。

那么极端情况下，**如果所有的共享内存都被修改，则此时的内存占用是原先的 2 倍。**

所以，针对写操作多的场景，我们要留意下快照过程中内存的变化，防止内存被占满了。

#### RDB 和 AOF 合体

尽管 RDB 比 AOF 的数据恢复速度快，但是快照的频率不好把握：

- 如果频率太低，两次快照间一旦服务器发生宕机，就可能会比较多的数据丢失；
- 如果频率太高，频繁写入磁盘和创建子进程会带来额外的性能开销。

那有没有什么方法不仅有 RDB 恢复速度快的优点和，又有 AOF 丢失数据少的优点呢？

当然有，那就是将 RDB 和 AOF 合体使用，这个方法是在 Redis 4.0 提出的，该方法叫**混合使用 AOF 日志和内存快照**，也叫混合持久化。

如果想要开启混合持久化功能，可以在 Redis 配置文件将下面这个配置项设置成 yes：

```text
aof-use-rdb-preamble yes
```

混合持久化工作在 **AOF 日志重写过程**。

当开启了混合持久化时，在 AOF 重写日志时，`fork` 出来的重写子进程会先**将与主线程共享的内存数据以 RDB 方式写入到 AOF 文件，然后主线程处理的操作命令会被记录在重写缓冲区里，重写缓冲区里的增量命令会以 AOF 方式写入到 AOF 文件，写入完成后通知主进程将新的含有 RDB 格式和 AOF 格式的 AOF 文件替换旧的的 AOF 文件**。

也就是说，使用了混合持久化，AOF 文件的**前半部分是 RDB 格式的全量数据，后半部分是 AOF 格式的增量数据**。

![图片](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/f67379b60d151262753fec3b817b8617.png)

这样的好处在于，重启 Redis 加载数据的时候，由于前半部分是 RDB 内容，这样**加载的时候速度会很快**。

加载完 RDB 的内容后，才会加载后半部分的 AOF 内容，这里的内容是 Redis 后台子进程重写 AOF 期间，主线程处理的操作命令，可以使得**数据更少的丢失**。

### 对比和总结

#### 1、RDB的优点

（1）比起AOF，在数据量比较大的情况下，RDB的启动速度更快。

（2）RDB文件是一个很简洁的单文件，它保存了某个时间点的Redis数据，很适合用于做备份。

（3）RDB的性能很好，需要进行持久化时，主进程会fork一个子进程出来，然后把持久化的工作交给子进程，自己不会有相关的I/O操作。

#### 2、RDB缺点

（1）RDB容易造成数据的丢失。假设每5分钟保存一次快照，如果Redis因为某些原因不能正常工作，那么从上次产生快照到Redis出现问题这段时间的数据就会丢失了。

（2）RDB使用fork()产生子进程的过程会堵塞主进程，所以数据比较大的话 fork() 可能很耗时，就会造成Redis停止服务几毫秒。

#### 3、AOF优点

（1）该机制可以带来更高的数据安全性，即数据持久性。Redis中提供了3中同步策略，即每秒同步、每修改同步和不同步。事实上，每秒同步也是异步完成的，其效率也是非常

高的，如果发生灾难，您只可能会丢失1秒的数据。

（2）AOF日志文件是一个纯追加的文件。就算服务器突然Crash，也不会出现日志的定位或者损坏问题。甚至如果因为某些原因（例如磁盘满了）命令只写了一半到日志文件里，

我们也可以用redis-check-aof这个工具很简单的进行修复。

（3）当AOF文件太大时，Redis会自动在后台进行重写。重写很安全，因为重写是在一个新的文件上进行。

#### 4、AOF缺点

（1）在相同的数据集下，AOF文件的大小一般会比RDB文件大。

（2）AOF开启后，写QPS会比RDB的低。通常fsync设置为每秒一次就能获得比较高的性能，而在禁止fsync的情况下速度可以达到RDB的水平。

### 生产配置持久化的一些意见

（1）官方建议：是**同时开启两种持久化策略**。因为有时需要RDB快照是进行数据库备份，更快重启以及发生AOF引擎错误的解决办法。（换句话就是通过RDB来多备份一份数据总是好的）

（2） 因为**RDB文件只用作后备用途**，建议**只在Slave上持久化RDB文件**，而且只要**15分钟备份一次**就够了，只保留save 900 1这条规则。

（3）如果选择AOF，只要硬盘许可，应该**尽量减少AOF rewrite的频率**。因为一是带来了持续的IO，二是AOF rewrite的最后将rewrite过程中产生的新数据写到新文件造成的阻塞

几乎是不可避免的。AOF重写的基础大小默认值64M太小了，可以设到2G以上。

## Redis分布式锁

[参考](https://juejin.cn/post/6936956908007850014)

### 前言

日常开发中，秒杀下单、抢红包等等业务场景，都需要用到分布式锁。而Redis非常适合作为分布式锁使用。本文将分七个方案展开，跟大家探讨Redis分布式锁的正确使用方式。如果有不正确的地方，欢迎大家指出哈，一起学习一起进步。

- 什么是分布式锁
- 方案一：SETNX + EXPIRE
- 方案二：SETNX + value值是（系统时间+过期时间）
- 方案三：使用Lua脚本(包含SETNX + EXPIRE两条指令)
- 方案四：SET的扩展命令（SET EX PX NX）
- 方案五：SET EX PX NX  + 校验唯一随机值,再释放锁
- 方案六: 开源框架:Redisson
- 方案七：多机实现的分布式锁Redlock
- github地址，感谢每颗star

> [github.com/whx123/Java…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fwhx123%2FJavaHome)

### 什么是分布式锁

> 分布式锁其实就是，**控制分布式系统不同进程共同访问共享资源的一种锁的实现**。如果不同的系统或同一个系统的不同主机之间共享了某个临界资源，往往需要互斥来防止彼此干扰，以保证一致性。

我们先来看下，一把靠谱的分布式锁应该有哪些特征：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42884a1613344c11be5fef3b9e8ed7c5~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

- **互斥性**: 任意时刻，只有一个客户端能持有锁。
- **锁超时释放**：持有锁超时，可以释放，防止不必要的资源浪费，也可以防止死锁。
- **可重入性**:一个线程如果获取了锁之后,可以再次对其请求加锁。
- **高性能和高可用**：加锁和解锁需要开销尽可能低，同时也要保证高可用，避免分布式锁失效。
- **安全性**：**锁只能被持有的客户端删除，不能被其他客户端删除**

### Redis分布式锁方案一：SETNX + EXPIRE

提到Redis的分布式锁，很多小伙伴马上就会想到`setnx`+ `expire`命令。即先用`setnx`来抢锁，如果抢到之后，再用`expire`给锁设置一个过期时间，防止锁忘记了释放。

> SETNX 是SET IF NOT EXISTS的简写.日常命令格式是SETNX key value，如果 key不存在，则SETNX成功返回1，如果这个key已经存在了，则返回0。

假设某电商网站的某商品做秒杀活动，key可以设置为key_resource_id,value设置任意值，伪代码如下：

```csharp
if（jedis.setnx(key_resource_id,lock_value) == 1）{ //加锁
    expire（key_resource_id，100）; //设置过期时间
    try {
        do something  //业务请求
    }catch(){
　　}
　　finally {
       jedis.del(key_resource_id); //释放锁
    }
}
```

但是这个方案中，`setnx`和`expire`两个命令分开了，**不是原子操作**。如果执行完`setnx`加锁，正要执行`expire`设置过期时间时，进程crash或者要重启维护了，那么这个锁就“长生不老”了，**别的线程永远获取不到锁啦**。

### Redis分布式锁方案二：SETNX + value值是(系统时间+过期时间)

为了解决方案一，**发生异常锁得不到释放的场景**，有小伙伴认为，可以把过期时间放到`setnx`的value值里面。如果加锁失败，再拿出value值校验一下即可。加锁代码如下：

```kotlin
long expires = System.currentTimeMillis() + expireTime; //系统时间+设置的过期时间
String expiresStr = String.valueOf(expires);

// 如果当前锁不存在，返回加锁成功
if (jedis.setnx(key_resource_id, expiresStr) == 1) {
        return true;
} 
// 如果锁已经存在，获取锁的过期时间
String currentValueStr = jedis.get(key_resource_id);

// 如果获取到的过期时间，小于系统当前时间，表示已经过期
if (currentValueStr != null && Long.parseLong(currentValueStr) < System.currentTimeMillis()) {

     // 锁已过期，获取上一个锁的过期时间，并设置现在锁的过期时间（不了解redis的getSet命令的小伙伴，可以去官网看下哈）
    String oldValueStr = jedis.getSet(key_resource_id, expiresStr);
    
    if (oldValueStr != null && oldValueStr.equals(currentValueStr)) {
         // 考虑多线程并发的情况，只有一个线程的设置值和当前值相同，它才可以加锁
         return true;
    }
}
        
//其他情况，均返回加锁失败
return false;
}
```

这个方案的优点是，巧妙移除`expire`单独设置过期时间的操作，把**过期时间放到setnx的value值**里面来。解决了方案一发生异常，锁得不到释放的问题。但是这个方案还有别的缺点：

> - 过期时间是客户端自己生成的（System.currentTimeMillis()是当前系统的时间），必须要求分布式环境下，每个客户端的时间必须同步。
> - 如果锁过期的时候，并发多个客户端同时请求过来，都执行jedis.getSet()，最终只能有一个客户端加锁成功，但是该客户端锁的过期时间，可能被别的客户端覆盖
> - 该锁没有保存持有者的唯一标识，可能被别的客户端释放/解锁。

### Redis分布式锁方案三：使用Lua脚本(包含SETNX + EXPIRE两条指令)

实际上，我们还可以使用Lua脚本来保证原子性（包含setnx和expire两条指令），lua脚本如下：

```vbnet
if redis.call('setnx',KEYS[1],ARGV[1]) == 1 then
   redis.call('expire',KEYS[1],ARGV[2])
else
   return 0
end;
复制代码
```

加锁代码如下：

```ini
 String lua_scripts = "if redis.call('setnx',KEYS[1],ARGV[1]) == 1 then" +
            " redis.call('expire',KEYS[1],ARGV[2]) return 1 else return 0 end";   
Object result = jedis.eval(lua_scripts, Collections.singletonList(key_resource_id), Collections.singletonList(values));
//判断是否成功
return result.equals(1L);
复制代码
```

这个方案还是有缺点的哦，至于哪些缺点，你先思考一下。也可以想下。跟方案二对比，哪个更好？

### Redis分布式锁方案方案四：SET的扩展命令（SET EX PX NX）

除了使用，使用Lua脚本，保证`SETNX + EXPIRE`两条指令的原子性，我们还可以巧用Redis的SET指令扩展参数！（`SET key value[EX seconds][PX milliseconds][NX|XX]`），它也是原子性的！

> SET key value[EX seconds][PX milliseconds][NX|XX]
>
> - NX :表示key不存在的时候，才能set成功，也即保证只有第一个客户端请求才能获得锁，而其他客户端请求只能等其释放锁，才能获取。
> - EX seconds :设定key的过期时间，时间单位是秒。
> - PX milliseconds: 设定key的过期时间，单位为毫秒
> - XX: 仅当key存在时设置值

伪代码demo如下：

```csharp
if（jedis.set(key_resource_id, lock_value, "NX", "EX", 100s) == 1）{ //加锁
    try {
        do something  //业务处理
    }catch(){
　　}
　　finally {
       jedis.del(key_resource_id); //释放锁
    }
}
复制代码
```

但是呢，这个方案还是可能存在问题：

- 问题一：**锁过期释放了，业务还没执行完**。假设线程a获取锁成功，一直在执行临界区的代码。但是100s过去后，它还没执行完。但是，这时候锁已经过期了，此时线程b又请求过来。显然线程b就可以获得锁成功，也开始执行临界区的代码。那么问题就来了，临界区的业务代码都不是严格串行执行的啦。
- 问题二：**锁被别的线程误删**。假设线程a执行完后，去释放锁。但是它不知道当前的锁可能是线程b持有的（线程a去释放锁时，有可能过期时间已经到了，此时线程b进来占有了锁）。那线程a就把线程b的锁释放掉了，但是线程b临界区业务代码可能都还没执行完呢。

### 方案五：SET EX PX NX  + 校验唯一随机值,再删除

既然锁可能被别的线程误删，那我们给value值设置一个标记当前线程唯一的随机数，在删除的时候，校验一下，不就OK了嘛。伪代码如下：

```csharp
if（jedis.set(key_resource_id, uni_request_id, "NX", "EX", 100s) == 1）{ //加锁
    try {
        do something  //业务处理
    }catch(){
　　}
　　finally {
       //判断是不是当前线程加的锁,是才释放
       if (uni_request_id.equals(jedis.get(key_resource_id))) {
        jedis.del(lockKey); //释放锁
        }
    }
}
复制代码
```

在这里，**判断是不是当前线程加的锁**和**释放锁**不是一个原子操作。如果调用jedis.del()释放锁的时候，可能这把锁已经不属于当前客户端，会解除他人加的锁。

![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9237655e6b1a47038d2774231e507e11~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

为了更严谨，一般也是用lua脚本代替。lua脚本如下：

```vbnet
if redis.call('get',KEYS[1]) == ARGV[1] then 
   return redis.call('del',KEYS[1]) 
else
   return 0
end;
复制代码
```

### Redis分布式锁方案六：Redisson框架

方案五还是可能存在**锁过期释放，业务没执行完**的问题。有些小伙伴认为，稍微把锁过期时间设置长一些就可以啦。其实我们设想一下，是否可以给获得锁的线程，开启一个定时守护线程，每隔一段时间检查锁是否还存在，存在则对锁的过期时间延长，防止锁过期提前释放。

当前开源框架Redisson解决了这个问题。我们一起来看下Redisson底层原理图吧：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/367cd1a7a3fb4d398988e4166416d71d~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

只要线程一加锁成功，就会启动一个`watch dog`看门狗，它是一个后台线程，会每隔10秒检查一下，如果线程1还持有锁，那么就会不断的延长锁key的生存时间。因此，Redisson就是使用Redisson解决了**锁过期释放，业务没执行完**问题。

### Redis分布式锁方案七：多机实现的分布式锁Redlock+Redisson

前面六种方案都只是基于单机版的讨论，还不是很完美。其实Redis一般都是集群部署的：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7349794feeee458aa71c27f27a0b2428~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

如果线程一在Redis的master节点上拿到了锁，但是加锁的key还没同步到slave节点。恰好这时，master节点发生故障，一个slave节点就会升级为master节点。线程二就可以获取同个key的锁啦，但线程一也已经拿到锁了，锁的安全性就没了。

为了解决这个问题，Redis作者 antirez提出一种高级的分布式锁算法：Redlock。Redlock核心思想是这样的：

> 搞多个Redis master部署，以保证它们不会同时宕掉。并且这些master节点是完全相互独立的，相互之间不存在数据同步。同时，需要确保在这多个master实例上，是与在Redis单实例，使用相同方法来获取和释放锁。

我们假设当前有5个Redis master节点，在5台服务器上面运行这些Redis实例。

![img](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0df0a36c7ccd439291a8a869ff4ddad3~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

RedLock的实现步骤:如下

> - 1.获取当前时间，以毫秒为单位。
> - 2.按顺序向5个master节点请求加锁。客户端设置网络连接和响应超时时间，并且超时时间要小于锁的失效时间。（假设锁自动失效时间为10秒，则超时时间一般在5-50毫秒之间,我们就假设超时时间是50ms吧）。如果超时，跳过该master节点，尽快去尝试下一个master节点。
> - 3.客户端使用当前时间减去开始获取锁时间（即步骤1记录的时间），得到获取锁使用的时间。当且仅当超过一半（N/2+1，这里是5/2+1=3个节点）的Redis master节点都获得锁，并且使用的时间小于锁失效时间时，锁才算获取成功。（如上图，10s> 30ms+40ms+50ms+4m0s+50ms）
> - 如果取到了锁，key的真正有效时间就变啦，需要减去获取锁所使用的时间。
> - 如果获取锁失败（没有在至少N/2+1个master实例取到锁，有或者获取锁时间已经超过了有效时间），客户端要在所有的master节点上解锁（即便有些master节点根本就没有加锁成功，也需要解锁，以防止有些漏网之鱼）。

简化下步骤就是：

- 按顺序向5个master节点请求加锁
- 根据设置的超时时间来判断，是不是要跳过该master节点。
- 如果大于等于三个节点加锁成功，并且使用的时间小于锁的有效期，即可认定加锁成功啦。
- 如果获取锁失败，解锁！

Redisson实现了redLock版本的锁，有兴趣的小伙伴，可以去了解一下哈~

## 高可用性（主从、哨兵、集群）

所谓的高可用，也叫 HA（High Availability），是分布式系统架构设计中必须考虑的因素之一，它通常是指，通过设计减少系统不能提供服务的时间。

如果在实际生产中，如果 redis 只部署一个节点，当机器故障时，整改服务都不能提供服务了。这就是我们常说的单点故障。

如果 redis 部署了多台，当一台或几台故障时，整个系统依然可以对外提供服务，这样就提高了服务的可用性。

今天我们就聊聊 redis 高可用的三种模式：**主从模式**，**哨兵模式**，**集群模式**。

### 一、主从模式

一般，系统的高可用都是通过部署多台机器实现的。redis 为了避免单点故障，也需要部署多台机器。

因为部署了多台机器，所以就会涉及到不同机器的的数据同步问题。

为此，redis 提供了 Redis 提供了复制(replication)功能，当一台 redis 数据库中的数据发生了变化，这个变化会被自动的同步到其他的 redis 机器上去。

redis 多机器部署时，这些机器节点会被分成两类，一类是主节点（master 节点），一类是从节点（slave 节点）。一般**主节点可以进行读、写操作**，而**从节点只能进行读操作**。同时由于主节点可以写，数据会发生变化，当主节点的数据发生变化时，会将变化的数据同步给从节点，这样从节点的数据就可以和主节点的数据保持一致了。一个主节点可以有多个从节点，但是一个从节点会只会有一个主节点，也就是所谓的一主多从结构。



![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/fe86a1d3d76c8cccaff47dcd75b04a28.png)

![图片](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/2b7231b6aabb9a9a2e2390ab3a280b2d.png)

#### 1.1.机器规划

|机器名称|IP|端口|

| ---- | ---- | ---- |

| master | 192.168.1.10 | 6379 |

| slave1 | 192.168.1.11 | 6379 |

| slave2 | 192.168.1.12 | 6379 |

| slave3 | 192.168.1.13 | 6379 |

#### 1.2.配置

**主节点配置**

主节点按照正常的配置配好即可。

**从节点配置**

使用默认的配置启动机器，机器都是主节点。如果想要让机器变成从节点，需要在 conf 服务器上配置主从复制的相关参数。

- 在从节点的配置文件 redis.conf 中指定主节点的信息（如果需要的话，可以配置主节点的登录密码，主从复制相关的参数）。三台从节点的配置是一样的。

```
# 配置主节点的ip和端口
slaveof 192.168.1.10 6379
# 从redis2.6开始，从节点默认是只读的
slave-read-only yes
# 假设主节点有登录密码，是123456
masterauth 123456
```

- 也可以不配置上面的文件，使用 redis-server 命令，在启动从节点时，通过参数--slaveof 指定主节点是谁。。

```
./redis-server --slaveof 192.168.1.10 6379
```

- 也可以不配上面的文件，正常启动 redis 机器，然后通过`redis-cli`的命令行执行`slaveof 192.168.1.10 6379`指定主节点是谁。

系统运行时，如果 master 挂掉了，可以在一个从库（如 slave1）上手动执行命令`slaveof no one`，将 slave1 变成新的 master；在 slave2 和 slave3 上分别执行`slaveof 192.168.1.11 6379` 将这两个机器的主节点指向的这个新的 master；同时，挂掉的原 master 启动后作为新的 slave 也指向新的 master 上。

执行命令`slaveof no one`命令，可以关闭从服务器的复制功能。同时原来同步的所得的数据集都不会被丢弃。

#### 1.3.机器启动

**首先启动主节点，然后一台一台启动从节点**。

#### 1.4.主从复制的机制



![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/c0d07bc58d77a298893584b11b9f78d9.png)



- 从数据库连接主数据库，发送 SYNC 命令;
- 主数据库接收到 SYNC 命令后，可以**执行 BGSAVE 命令生成 RDB 文件并使用缓冲区记录此后执行的所有写命令**;
- 主数据库 BGSAVE 执行完后，**向所有从数据库发送快照文件，并在发送期间继续记录被执行的写命令**;
- 从数据库收到快照文件后**丢弃所有旧数据**，载入收到的快照;
- 主数据库快照发送完毕后开始向从数据库发送缓冲区中的写命令;
- 从数据库**完成对快照的载入**，开始接受命令请求，并**执行来自主数据库缓冲区的写命令**;(**从数据库初始化完成**)
- **主数据库**每执行一个写命令就会**向从数据库发送相同的写命令**，从数据库接收并执行收到的写命令(**从数据库初始化完成后的操作**)
- 出现断开重连后，2.8 之后的版本会将断线期间的命令传给从数据库，增量复制。
- **主从刚刚连接的时候，进行全量同步;全同步结束后，进行增量同步**。当然，如果有需要，slave 在任何时候都可以发起全量同步。Redis 的策略是，无论如何，首先会尝试进行增量同步，如不成功，要求从机进行全量同步。

主从服务器间的第一次同步的过程可分为三个阶段：

- 第一阶段是建立链接、协商同步；
- 第二阶段是主服务器同步数据给从服务器；
- 第三阶段是主服务器发送新写操作命令给从服务器。

为了让你更清楚了解这三个阶段，我画了一张图。

![图片](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/ea4f7e86baf2435af3999e5cd38b6a26.png)

接下来，我在具体介绍每一个阶段都做了什么。

*第一阶段：建立链接、协商同步*

执行了 replicaof 命令后，从服务器就会给主服务器发送 `psync` 命令，表示要进行数据同步。

psync 命令包含两个参数，分别是**主服务器的 runID** 和**复制进度 offset**。

- runID，每个 Redis 服务器在启动时都会自动生产一个随机的 ID 来唯一标识自己。当从服务器和主服务器第一次同步时，因为不知道主服务器的 run ID，所以将其设置为 "?"。
- offset，表示复制的进度，第一次同步时，其值为 -1。

主服务器收到 psync 命令后，会用 `FULLRESYNC` 作为响应命令返回给对方。

并且这个响应命令会带上两个参数：**主服务器的 runID** 和**主服务器目前的复制进度 offset**。从服务器收到响应后，会记录这两个值。

FULLRESYNC 响应命令的意图是采用**全量复制**的方式，也就是主服务器会把所有的数据都同步给从服务器。

所以，第一阶段的工作时为了全量复制做准备。

那具体怎么全量同步呀呢？我们可以往下看第二阶段。

*第二阶段：主服务器同步数据给从服务器*

接着，**主服务器会执行 bgsave 命令来生成 RDB 文件，然后把文件发送给从服务器**。

从服务器收到 RDB 文件后，会先清空当前的数据，然后载入 RDB 文件。

这里有一点要注意，主服务器生成 RDB 这个过程是不会阻塞主线程的，因为 bgsave 命令是产生了一个子进程来做生成 RDB 文件的工作，是异步工作的，这样 Redis 依然可以正常处理命令。

但是，**这期间的写操作命令并没有记录到刚刚生成的 RDB 文件中**，这时主从服务器间的数据就不一致了。那么为了保证主从服务器的数据一致性，**主服务器在下面这三个时间间隙中将收到的写操作命令，写入到 replication buffer 缓冲区里。**

- 主服务器生成 RDB 文件期间；
- 主服务器发送 RDB 文件给从服务器期间；
- 「从服务器」加载 RDB 文件期间；

*第三阶段：主服务器发送新写操作命令给从服务器*

在主服务器生成的 RDB 文件发送完，从服务器加载完 RDB 文件后，然后将 replication buffer 缓冲区里所记录的写操作命令发送给从服务器，然后「从服务器」重新执行这些操作，至此主从服务器的数据就一致了。

至此，主从服务器的第一次同步的工作就完成了。

#### 命令传播

主从服务器在完成第一次同步后，双方之间就**会维护一个 TCP 连接**。

![图片](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/03eacec67cc58ff8d5819d0872ddd41e.png)

后续主服务器可以通过这个连接继续将写操作命令传播给从服务器，然后从服务器执行该命令，使得与主服务器的数据库状态相同。

而且这个连接是**长连接**的，目的是避免频繁的 TCP 连接和断开带来的性能开销。

上面的这个过程被称为**基于长连接的命令传播**，通过这种方式来保证第一次同步后的主从服务器的数据一致性。

#### 分摊主服务器的压力

在前面的分析中，我们可以知道主从服务器在第一次数据同步的过程中，**主服务器会做两件耗时的操作：生成 RDB 文件和传输 RDB 文件**。

主服务器是可以有多个从服务器的，如果从服务器数量非常多，而且都与主服务器进行全量同步的话，就会带来两个问题：

- 由于是通过 bgsave 命令来生成 RDB 文件的，那么主服务器就会忙于使用 fork() 创建子进程，如果主服务器的内存数据非大，在执行 fork() 函数时是会阻塞主线程的，从而使得 Redis 无法正常处理请求；
- 传输 RDB 文件会占用主服务器的网络带宽，会对主服务器响应命令请求产生影响。

这种情况就好像，刚创业的公司，由于人不多，所以员工都归老板一个人管，但是随着公司的发展，人员的扩充，老板慢慢就无法承担全部员工的管理工作了。

要解决这个问题，老板就需要设立经理职位，由经理管理多名普通员工，然后老板只需要管理经理就好。

Redis 也是一样的，从服务器可以有自己的从服务器，我们可以把拥有从服务器的从服务器当作经理角色，它不仅可以接收主服务器的同步数据，自己也可以同时作为主服务器的形式将数据同步给从服务器，组织形式如下图：

![图片](https://img-blog.csdnimg.cn/img_convert/4d850bfe8d712d3d67ff13e59b919452.png)

通过这种方式，**主服务器生成 RDB 和传输 RDB 的压力可以分摊到充当经理角色的从服务器**。

那具体怎么做到的呢？

其实很简单，我们在「从服务器」上执行下面这条命令，使其作为目标服务器的从服务器：

```text
replicaof <目标服务器的IP> 6379
```

此时如果目标服务器本身也是「从服务器」，那么该目标服务器就会成为「经理」的角色，不仅可以接受主服务器同步的数据，也会把数据同步给自己旗下的从服务器，从而减轻主服务器的负担。

#### 增量复制

主从服务器在完成第一次同步后，就会基于长连接进行命令传播。

可是，网络总是不按套路出牌的嘛，说延迟就延迟，说断开就断开。

如果主从服务器间的**网络连接断开了，那么就无法进行命令传播了**，这时从服务器的数据就没办法和主服务器保持一致了，客户端就可能从「从服务器」读到旧的数据。

![图片](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/4845008abadaa871613873f5ffdcb542.png)

那么问题来了，如果此时断开的网络，又恢复正常了，要怎么继续保证主从服务器的数据一致性呢？

在 Redis 2.8 之前，如果主从服务器在命令同步时出现了网络断开又恢复的情况，从服务器就会和主服务器重新进行一次全量复制，很明显这样的开销太大了，必须要改进一波。

所以，从 Redis 2.8 开始，网络断开又恢复后，从主从服务器会采用**增量复制**的方式继续同步，也就是只会**把网络断开期间主服务器接收到的写操作命令，同步给从服务器**。

网络恢复后的增量复制过程如下图：

![图片](https://img-blog.csdnimg.cn/img_convert/e081b470870daeb763062bb873a4477e.png)

主要有三个步骤：

- 从服务器在恢复网络后，会发送 psync 命令给主服务器，此时的 psync 命令里的 offset 参数不是 -1；
- 主服务器收到该命令后，然后用 CONTINUE 响应命令告诉从服务器接下来采用增量复制的方式同步数据；
- 然后主服务将主从服务器断线期间，所执行的写命令发送给从服务器，然后从服务器执行这些命令。

那么关键的问题来了，**主服务器怎么知道要将哪些增量数据发送给从服务器呢？**

答案藏在这两个东西里：

- **repl_backlog_buffer**，是一个「**环形**」缓冲区，用于主从服务器断连后，从中找到差异的数据；
- **replication offset**，标记上面那个缓冲区的同步进度，主从服务器都有各自的偏移量，主服务器使用 master_repl_offset 来记录自己「*写*」到的位置，从服务器使用 slave_repl_offset 来记录自己「*读*」到的位置。

那repl_backlog_buffer 缓冲区是什么时候写入的呢？

在**主服务器进行命令传播时，不仅会将写命令发送给从服务器，还会将写命令写入到 repl_backlog_buffer 缓冲区**里，因此 这个缓冲区里会保存着最近传播的写命令。

网络断开后，当从服务器重新连上主服务器时，从服务器会通过 psync 命令将自己的复制偏移量 slave_repl_offset 发送给主服务器，主服务器根据自己的 master_repl_offset 和 slave_repl_offset 之间的差距，然后来决定对从服务器执行哪种同步操作：

- 如果判断出从服务器要读取的数据还在 repl_backlog_buffer 缓冲区里，那么主服务器将采用**增量同步**的方式；
- 相反，如果判断出从服务器要读取的数据**已经不存在 repl_backlog_buffer 缓冲区**里，那么主服务器将采用**全量同步**的方式。

当主服务器在 repl_backlog_buffer 中找到主从服务器差异（增量）的数据后，就会将增量的数据写入到 replication buffer 缓冲区，这个缓冲区我们前面也提到过，它是缓存将要传播给从服务器的命令。

![图片](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/2db4831516b9a8b79f833cf0593c1f12.png)

repl_backlog_buffer 缓行缓冲区的默认大小是 1M，并且由于它是一个环形缓冲区，所以当缓冲区写满后，主服务器继续写入的话，就会覆盖之前的数据。

因此，当主服务器的写入速度远超于从服务器的读取速度，缓冲区的数据一下就会被覆盖。

那么在网络恢复时，如果从服务器想读的数据已经被覆盖了，主服务器就会采用全量同步，这个方式比增量同步的性能损耗要大很多。

因此，为了避免在网络恢复时，主服务器频繁地使用全量同步的方式，我们应该调整下 repl_backlog_buffer 缓冲区大小，尽可能的大一些，减少出现从服务器要读取的数据被覆盖的概率，从而使得主服务器采用增量同步的方式。

那 repl_backlog_buffer 缓冲区具体要调整到多大呢？

repl_backlog_buffer 最小的大小可以根据这面这个公式估算。

![图片](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/5e9e65a4a59b3688fa37cadbd87bb5ac.png)

我来解释下这个公式的意思：

- second 为从服务器断线后重新连接上主服务器所需的平均 时间(以秒计算)。
- write_size_per_second 则是主服务器平均每秒产生的写命令数据量大小。

举个例子，如果主服务器平均每秒产生 1 MB 的写命令，而从服务器断线之后平均要 5 秒才能重新连接主服务器。

那么 repl_backlog_buffer 大小就不能低于 5 MB，否则新写地命令就会覆盖旧数据了。

当然，为了应对一些突发的情况，可以将 repl_backlog_buffer 的大小设置为此基础上的 2 倍，也就是 10 MB。

关于 repl_backlog_buffer 大小修改的方法，只需要修改配置文件里下面这个参数项的值就可以。

```shell
repl-backlog-size 1mb
```

#### 总结

主从复制共有三种模式：**全量复制、基于长连接的命令传播、增量复制**。

主从服务器第一次同步的时候，就是采用全量复制，此时主服务器会两个耗时的地方，分别是生成 RDB 文件和传输 RDB 文件。为了避免过多的从服务器和主服务器进行全量复制，可以把一部分从服务器升级为「经理角色」，让它也有自己的从服务器，通过这样可以分摊主服务器的压力。

第一次同步完成后，主从服务器都会维护着一个长连接，主服务器在接收到写操作命令后，就会通过这个连接将写命令传播给从服务器，来保证主从服务器的数据一致性。

如果遇到网络断开，增量复制就可以上场了，不过这个还跟 repl_backlog_size 这个大小有关系。

如果它配置的过小，主从服务器网络恢复时，可能发生「从服务器」想读的数据已经被覆盖了，那么这时就会导致主服务器采用全量复制的方式。所以为了避免这种情况的频繁发生，要调大这个参数的值，以降低主从服务器断开后全量同步的概率。

##### 优点

- 支持主从复制，主机会自动将数据同步到从机，可以进行读写分离;
- 为了分载 Master 的读操作压力，Slave 服务器可以为客户端提供只读操作的服务，写服务依然必须由 Master 来完成;
- Slave 同样可以接受其他 Slaves 的连接和同步请求，这样可以有效地分载 Master 的同步压力;
- Master 是以非阻塞的方式为 Slaves 提供服务。所以在 Master-Slave 同步期间，客户端仍然可以提交查询或修改请求;
- Slave 同样是以阻塞的方式完成数据同步。在同步期间，如果有客户端提交查询请求，Redis 则返回同步之前的数据。

##### 缺点

- Redis **不具备自动容错和恢复功能，主机从机的宕机都会导致前端部分读写请求失败，需要等待机器重启或者手动切换前端的 IP 才能恢复;**
- 主机宕机，宕机前有部分数据未能及时同步到从机，**切换 IP 后还会引入数据不一致的问题，降低了系统的可用性;**
- 如果多个 Slave 断线了，需要重启的时候，尽量不要在同一时间段进行重启。因为只要 Slave 启动，就会发送 sync 请求和主机全量同步，当多个 Slave 重启的时候，可能会导致 Master IO 剧增从而宕机。
- Redis 较**难支持在线扩容**，在集群容量达到上限时在线扩容会变得很复杂;
- redis 的主节点和从节点中的数据是一样的，降低的内存的可用性

### 二、哨兵模式

主从模式下，当主服务器宕机后，需要手动把一台从服务器切换为主服务器，这就需要人工干预，费事费力，还会造成一段时间内服务不可用。这种方式并不推荐，实际生产中，我们优先考虑哨兵模式。这种模式下，master 宕机，哨兵会自动选举 master 并将其他的 slave 指向新的 master。

在主从模式下，redis 同时提供了哨兵命令`redis-sentinel`，**哨兵是一个独立的进程，作为进程，它会独立运行**。其原理是哨兵进程向所有的 redis 机器发送命令，等待 Redis 服务器响应，从而监控运行的多个 Redis 实例。

**哨兵可以有多个，一般为了便于决策选举，使用奇数个哨兵**。哨兵可以和 redis 机器部署在一起，也可以部署在其他的机器上。多个哨兵构成一个哨兵集群，哨兵直接也会相互通信，检查哨兵是否正常运行，同时**发现 master 宕机哨兵之间会进行决策选举新的 master**



![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/7a1e6c5ceef52c9a8c6f80e5aced9aee.png)

哨兵模式的作用:

- 通过发送命令，让 Redis 服务器返回监控其运行状态，包括主服务器和从服务器;
- 当哨兵监测到 master 宕机，会自动将 slave 切换到 master，然后通过**发布订阅模式**通过其他的从服务器，修改配置文件，让它们切换主机;
- 然而一个哨兵进程对 Redis 服务器进行监控，也可能会出现问题，为此，我们可以使用多个哨兵进行监控。各个哨兵之间还会进行监控，这样就形成了多哨兵模式。

哨兵很像 kafka 集群中的 zookeeper 的功能。

#### 2.1.机器规划

| 机器名称   | IP           | 端口  |

| ---------- | ------------ | ----- |

| master     | 192.168.1.10 | 6379  |

| slave 1    | 192.168.1.11 | 6379  |

| slave 2    | 192.168.1.12 | 6379  |

| slave 3    | 192.168.1.13 | 6379  |

| sentinel 1 | 192.168.1.14 | 26379 |

| sentinel 2 | 192.168.1.15 | 26379 |

| sentinel 3 | 192.168.1.16 | 26379 |

这里我们将哨兵进程和 redis 分别部署在不同的机器上，避免因为 redis 宕机导致 sentinel 进程不可用。

#### 2.2.配置

redis.conf 的配置和上面主从模式一样，不用变。这里主要说一下哨兵的配置。

每台机器的哨兵进程都需要一个哨兵的配置文件`sentinel.conf`，三台机器的哨兵配置是一样的。

```
# 禁止保护模式
protected-mode no
# 配置监听的主服务器，这里sentinel monitor代表监控，mymaster代表服务器的名称，可以自定义，
#192.168.1.10代表监控的主服务器，6379代表端口，2代表只有两个或两个以上的哨兵认为主服务器不可用的时候，才会进行failover操作。
sentinel monitor mymaster 192.168.1.10 6379 2
# sentinel author-pass定义服务的密码，mymaster是服务名称，123456是Redis服务器密码
sentinel auth-pass mymaster 123456
```

#### 2.3.机器启动

首先启动主节点，然后一台一台启动从节点。

redis 集群启动完成后，分别启动哨兵集群所在机器的三个哨兵，使用`redis-sentinel /path/to/sentinel.conf`命令。

#### 2.4.哨兵模式的工作

- 每个 Sentinel（哨兵）进程以**每秒钟一次**的频率向整个集群中的 Master 主服务器，Slave 从服务器以及其他 Sentinel（哨兵）进程发送一个 PING 命令。
- 如果一个实例（instance）距离最后一次有效回复 PING 命令的时间超过 down-after-milliseconds 选项所指定的值， 则这个实例会被 Sentinel（哨兵）进程标记为**主观下线**（SDOWN）
- 如果一个 Master 主服务器被标记为主观下线（SDOWN），则正在监视这个 Master 主服务器的所有 Sentinel（哨兵）进程要以**每秒一次的频率确认 Master 主服务器的确进入了主观下线状态**
- 当**有足够数量的 Sentinel（哨兵）进程（大于等于配置文件指定的值）在指定的时间范围内确认 Master 主服务器进入了主观下线状态（SDOWN）， 则 Master 主服务器会被标记为客观下线**（ODOWN）
- 在一般情况下， 每个 Sentinel（哨兵）进程会以**每 10 秒一次的频率向集群中的所有 Master 主服务器、Slave 从服务器**发送 INFO 命令。
- 当 Master 主服务器被 Sentinel（哨兵）进程标记为客观下线（ODOWN）时，**Sentinel（哨兵）进程向下线的 Master 主服务器的所有 Slave 从服务器发送 INFO 命令的频率会从 10 秒一次改为每秒一次**。
- 若**没有足够数量的 Sentinel（哨兵）进程同意 Master 主服务器下线， Master 主服务器的客观下线状态就会被移除**。若 Master 主服务器重新向 Sentinel（哨兵）进程发送 PING 命令返回有效回复，Master 主服务器的主观下线状态就会被移除。

假设 master 宕机，sentinel 1 先检测到这个结果，系统并不会马上进行 failover(故障转移)选出新的 master，仅仅是 sentinel 1 主观的认为 master 不可用，这个现象成为**主观下线**。当后面的哨兵也检测到主服务器不可用，并且数量达到一定值时，那么哨兵之间就会进行一次投票，投票的结果由 sentinel 1 发起，进行 failover 操作。切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从服务器实现切换主机，这个过程称为**客观下线**。这样对于客户端而言，一切都是透明的。

#### 2.5.主从模式的优缺点

#### 优点

- 哨兵模式是基于主从模式的，所有主从的优点，哨兵模式都具有。
- 主从可以自动切换，系统更健壮，可用性更高。

#### 缺点 

- 具有主从模式的缺点，每台机器上的数据是一样的，内存的可用性较低。
- Redis 较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂。

### 三、集群模式

先说一个误区：**Redis 的集群模式本身没有使用一致性 hash 算法，而是使用 slots 插槽**。这是很多人的一个误区。这里先留个坑，后面我会出一期《 [redis 系列之——一致性 hash 算法](https://xie.infoq.cn/article/541eb1cf68415982a99b01cdd)》。

Redis 的哨兵模式基本已经可以实现高可用，读写分离 ，但是在这种模式下每台 Redis 服务器都存储相同的数据，很浪费内存，所以在 redis3.0 上加入了 Cluster 集群模式，实现了 Redis 的分布式存储，对数据进行分片，也就是说每台 Redis 节点上存储不同的内容；

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/64bc0e3e8784bd3830a84be4b699df48.png)

这里的 6 台 redis 两两之间并不是独立的，每个节点都会通过**集群总线(cluster bus)，与其他的节点进行通信**。通讯时使用特殊的端口号，即对外服务端口号加 10000。例如如果某个 node 的端口号是 6379，那么它与其它 nodes 通信的端口号是 16379。nodes 之间的通信采用特殊的二进制协议。

对客户端来说，**整个 cluster 被看做是一个整体**，客户端可以连接任意一个 node 进行操作，就像操作单一 Redis 实例一样，当客户端操作的 key 没有分配到该 node 上时，Redis 会返回转向指令，指向正确的 node，这有点儿像浏览器页面的 302 redirect 跳转。

根据官方推荐，集群部署至少要 3 台以上的 master 节点，最好使用 3 主 3 从六个节点的模式。测试时，也可以在一台机器上部署这六个实例，通过端口区分出来。

#### 3.1.机器规划

| 机器名称 | IP           | 端口 |

| -------- | ------------ | ---- |

| master 1 | 192.168.1.11 | 6379 |

| master 2 | 192.168.1.12 | 6379 |

| master 3 | 192.168.1.13 | 6379 |

| slave 1  | 192.168.1.21 | 6379 |

| slave 2  | 192.168.1.22 | 6379 |

| slave 3  | 192.168.1.23 | 6379 |

#### 3.2.配置

修改`redis.conf` 的配置文件:

```
# 开启redis的集群模式
cluster-enabled yes
# 配置集群模式下的配置文件名称和位置,redis-cluster.conf这个文件是集群启动后自动生成的，不需要手动配置。
cluster-config-file redis-cluster.conf
```

#### 3.3.机器启动

6 个 Redis 服务分别启动成功之后，这时虽然配置了集群开启，但是这六台机器还是独立的。使用集群管理命令将这 6 台机器添加到一个集群中。

借助 redis-tri.rb 工具可以快速的部署集群。

只需要执行`redis-trib.rb create --replicas 1 192.168.1.11:6379 192.168.1.21:6379 192.168.1.12:6379 192.168.1.22:6379 192.168.1.13:6379 192.168.1.23:6379`就可以成功创建集群。

该命令执行创建完成后会有响应的日志，通过相关的日志就可以看出集群中机器的关系(不一定和上图对应)，执行的日志如下：

```
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.168.1.21:6379 to 192.168.1.11:6379
Adding replica 192.168.1.22:6379 to 192.168.1.12:6379
Adding replica 192.168.1.23:6379 to 192.168.1.13:6379
M: 80c80a3f3e33872c047a8328ad579b9bea001ad8 192.168.1.11:6379
   slots:[0-5460] (5461 slots) master
S: b4d3eb411a7355d4767c6c23b4df69fa183ef8bc 192.168.1.21:6379
   replicates 6788453ee9a8d7f72b1d45a9093838efd0e501f1
M: 4d74ec66e898bf09006dac86d4928f9fad81f373 192.168.1.12:6379
   slots:[5461-10922] (5462 slots) master
S: b6331cbc986794237c83ed2d5c30777c1551546e 192.168.1.22:6379
   replicates 80c80a3f3e33872c047a8328ad579b9bea001ad8
M: 6788453ee9a8d7f72b1d45a9093838efd0e501f1 192.168.1.13:6379
   slots:[10923-16383] (5461 slots) master
S: 277daeb8660d5273b7c3e05c263f861ed5f17b92 192.168.1.23:6379
   replicates 4d74ec66e898bf09006dac86d4928f9fad81f373
Can I set the above configuration? (type 'yes' to accept): yes                  #输入yes，接受上面配置
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
```

执行完成后自动生成配置的 redis-cluster.conf 文件。

登录集群：`redis-cli -c -h 192.168.1.11 -p 6379 -a 123456                  # -c，使用集群方式登录`。

查看集群信息：`192.168.1.11:6379> CLUSTER INFO                   #集群状态`。

列出节点信息：`192.168.1.11:6379> CLUSTER NODES                  #列出节点信息`。

```
192.168.1.11:6379> set name aaa
-> Redirected to slot [13680] located at 192.168.1.13:6379                #说明最终将数据写到了192.168.1.13:6379上
OK
```

获取数据：

```
192.168.1.11:6379> get name
-> Redirected to slot [13680] located at 192.168.1.13:6379                #说明最终到192.168.1.13:6379上读数据
"aaa"
```

#### 3.4.运行机制

在 Redis 的每一个节点上，都有这么两个东西，**一个是插槽（slot），它的的取值范围是：0-16383**，可以从上面`redis-trib.rb`执行的结果看到这 16383 个 slot 在三个 master 上的分布。还有一个就是 **cluster，可以理解为是一个集群管理的插件**，类似的哨兵。

当我们的存取的 Key 到达的时候，Redis 会**根据 crc16 的算法对计算后得出一个结果，然后把结果和 16384 求余数**，这样每个 key 都会对应一个编号在 0-16383 之间的哈希槽，通过这个值，去找到对应的插槽所对应的节点，然后直接自动跳转到这个对应的节点上进行存取操作。

当数据写入到对应的 master 节点后，这个**数据会同步给这个 master 对应的所有 slave 节点**。

为了保证高可用，**redis-cluster 集群引入了主从模式**，一个主节点对应一个或者多个从节点。当其它主节点 ping 主节点 master 1 时，如果半数以上的主节点与 master 1 通信超时，那么认为 master 1 宕机了，就会启用 master 1 的从节点 slave 1，将 slave 1 变成主节点继续提供服务。

**如果 master 1 和它的从节点 slave 1 都宕机了，整个集群就会进入 fail 状态**，因为集群的 slot 映射不完整。**如果集群超过半数以上的 master 挂掉，无论是否有 slave，集群都会进入 fail 状态**。

redis-cluster 采用去中心化的思想，没有中心节点的说法，客户端与 Redis 节点直连，不需要中间代理层，客户端不需要连接集群所有节点，连接集群中任何一个可用节点即可。

#### 原理介绍

- 选举
  集群启动后，主从已分配完成，经过了 N 轮的选举。当某一个主节点宕机，那么从节点需要经过选举成为主节点。简单介绍选举过程：
  **所有子节点向其他节点发送请求**，请求自身成为主节点，其他节点收到请求后，返回投票信息，**只有主节点 master 有权投票**，且只能投一次，当**获取到的票数大于一半人数时（master 个数），就当选 master**。期间，所有子节点发送请求的时机有所有不同。所以基本都有先后顺序，所以很少会出现票数相同情况，如果相同，则重新选举，直到选出 master 为止。故，需要至少 3 主 3 从，否则节点出现问题，将选举失败。
- 槽位
  在 Redis 集群中，定义了 16384 个逻辑上的槽位。**将这些槽位均匀分配给 N 个节点**（一主一从为一个节点），此文 3 个节点，自动均匀分配。意思为，0-5460 个槽位分配给第一个节点。当用户 set 一个值时，除了计算 key 本身的 hashCode 之外，还会调用 C 语言的一个 CRC16 算法，将 key 当 hash 值再计算出一个数字，然后与 16384 取模，得到的数字落在哪个槽位，则会将数据放在对应的节点。
  比如，计算出的数字为 16387，则取模 16384 后，得到 3，在 0-5460 之间，则放入对于的第一个节点。其他以此类推。
- 跳转
  大家都知道，主从模式中，**只有主节点可以写入数据，而从节点只能读取数据**。在 Cluster 集群中，设置值时，如果计算出的槽位在另一台服务器上，则集群连接会自动跳转至相应服务器。
- 网络抖动
  网络抖动，表示网络很不稳定。当出现这样的情况，可能一小段时间连接不上，可能就认为了节点挂了。这里就涉及到连接到超时时间，在网络不稳定的情况下，可以将超时时间设置长一些。

#### 特点

- **所有的redis节点彼此互联(**PING-PONG机制),内部使用二进制协议优化传输速度和带宽。
- 多数派原则：对于集群中的任何一个节点，需要**超过半数的节点检测到它失效（pFail）**，才会将其判定为失效（Fail）；
- 客户端与redis节点直连,**不需要中间代理层**.客户端不需要连接集群所有节点,**连接集群中任何一个可用节点即可**。
- **集群对外统一**。在使用集群时，只需要关注 Redis 各个节点的 IP 和端口，至于读写节点、主从等无需关注；
- 去中心化。本模式不再如哨兵、Codise 等，需要第三方监控。Cluster自行加入选举，完成主节点选举，以及读写访问控制。
- 数据分片：Redis Cluster 的键空间被分割为 16384 个 Slot，这些 Slot 被分别指派给主节点，当存储 Key-Value 时，根据 CRC16(key) Mod 16384的值，决定将一个 Key-Value 放到哪个 Slot 中；
- 自动 Failover：当集群中某个主节点故障后（Fail），**其它主节点会从故障主节点的从节点中选举一个“最佳”从节点升主**，替代故障的主节点；
- 功能弱化：集群模式下，由于数据分布在多个节点，**不支持单机模式下的集合操作**，也**不支持多数据库功能**，集群只能使用默认的0号数据库；
- 集群规模：官方推荐的最大节点数量为 1000 个左右，这是因为当集群规模过大时，Gossip 协议的效率会显著下降，通信成本剧增。

#### 分片

**Redis 集群实现的基础是分片**，即**将数据集有机的分割为多个片，并将这些分片指派给多个 Redis 实例**，每个实例只保存总数据集的一个子集。利用多台计算机内存和来支持更大的数据库，而避免受限于单机的内存容量；通过多核计算机集群，可有效扩展计算能力；通过多台计算机和网络适配器，允许我们扩展网络带宽。

基于“分片”的思想，Redis 提出了 Hash Slot。Redis Cluster 把所有的物理节点映射到预先分好的16384个 Slot 上，当需要在 Redis 集群中放置一个 Key-Value 时，根据 CRC16(key) Mod 16384的值，决定将一个 Key 放到哪个 Slot 中。
![在这里插入图片描述](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/20200619001025887.png)

#### 请求路由方式

客户端直连 Redis 服务，进行读写操作时，Key 对应的 Slot 可能并不在当前直连的节点上，经过“重定向”才能转发到正确的节点。客户端进行 Set 操作，当 Key 对应的 Slot 不在当前节点时，客户端会报错并返回正确节点的 IP 和端口。Set 成功则返回 OK。

以集群模式登录 127.0.0.1:6379 客户端（注意命令的差别：-c 表示集群模式)，则可以清楚的看到“重定向”的信息，并且客户端也发生了切换：“6379” -> “6381”。
![在这里插入图片描述](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/202006190015471.png)

Redis Cluster 借助客户端实现的请求路由是一种**混合形式的查询路由**，它并非从一个 Redis 节点到另外一个 Redis，而是**借助客户端转发到正确的节点**。

实际应用中，可以**在客户端缓存 Slot 与 Redis 节点的映射关系**，当接收到 MOVED 响应时修改缓存中的映射关系。如此，基于保存的映射关系，请求时会直接发送到正确的节点上，从而减少一次交互，提升效率。

#### 节点通信原理：Gossip 算法

在分布式系统中，需要提供维护节点元数据信息的机制，所谓元数据是指节点负责哪些数据、主从属性、是否出现故障等状态信息。

常见的**元数据维护方式分为集中式和无中心式**。Redis Cluster 采用 Gossip 协议实现了**无中心式**。

Redis Cluster 中的每个 Redis 实例监听两个 TCP 端口，**6379（默认）用于服务客户端查询，16379（默认服务端口+10000）用于集群内部通信**。集群中节点通信方式如下：

- 每个节点在**固定周期内通过特定规则选择几个节点发送 Ping 消息**；
- 接收到 Ping 消息的节点用 Pong 消息作为响应。

##### 1.节点间是如何交换信息的

Redis 节点启动之后，会**每间隔 100ms 执行一次集群的周期性函数** clusterCron()。

一、当前节点向另一个节点发送 Ping 消息时，**携带的其它节点的消息数量至少为3**，最大等于集群节点总数-2；

二、为 Ping 消息体中选择携带的其它节点的信息时，采用的是混合选择模式：随机选择+偏好性选择，这样不仅可以保证 Gossip 协议随机传播的原则，还可以尽量将当前节点掌握的其它节点的故障信息传播出去。

##### 2.如何保证消息传播的效率

前面已经提到，集群的周期性函数 clusterCron() 执行周期是 100ms，为了保证传播效率，每10个周期，也就是 1s，每个节点都会随机选择5个其它节点，并从中选择一个最久没有通信的节点发送 ing消息。

当然，这样还是没法保证效率，毕竟5个节点是随机选出来的，其中最久没有通信的节点不一定是全局“最久”。因此，**对哪些长时间没有“被” 随机到的节点进行特殊照顾**：每个周期（100ms）内扫描一次本地节点列表，如果发现节点最近一次接受 Pong 消息的时间大于 cluster_node_timeout/2，则立刻发送 Ping 消息，防止该节点信息太长时间未更新。

cluster_node_timeout 参数对消息发送的节点数量影响非常大。当带宽资源紧张时，可以适当调大这个参数，如从默认15秒改为30秒来降低带宽占用率。但是，过度调大 cluster_node_timeout 会影响消息交换的频率从而影响故障转移、槽信息更新、新节点发现的速度，因此需要根据业务容忍度和资源消耗进行平衡。同时整个集群消息总交换量也跟节点数成正比。

每个 Ping 消息的数据量体现在消息头和消息体中，其中消息头空间占用相对固定。消息体会携带一定数量的其它节点信息用于信息交换，消息体携带数据量跟集群的节点数息息相关，更大的集群每次消息通信的成本也就更高，因此对于 Redis 集群来说并不是越大越好。

#### 3.5.集群扩缩容

[参考](https://blog.csdn.net/suifeng629/article/details/106761559)

对 redis 集群的扩容就是向集群中添加机器，缩容就是从集群中删除机器，并**重新将 16383 个 slots 分配到集群中的节点上（数据迁移）**。

扩缩容也是使用集群管理工具 redis-tri.rb。

扩容时，先使用`redis-tri.rb add-node`将新的机器加到集群中，这是新机器虽然已经在集群中了，但是没有分配 slots，依然是不起做用的。在使用 ` redis-tri.rb reshard`进行分片重哈希（数据迁移），将旧节点上的 slots 分配到新节点上后，新节点才能起作用。

缩容时，先要使用 ` redis-tri.rb reshard`移除的机器上的 slots，然后使用`redis-tri.rb add-del`移除机器。

#### 故障转移

Redis Cluster 的故障转移可划分为三大步骤：故障检测、从节点选举以及故障倒换，以下详细介绍。

##### 故障检测

- 单点视角检测
  集群中的每个节点都会定期通过集群内部通信总线向集群中的其它节点发送 Ping 消息，用于检测对方是否在线。如果接收 Ping 消息的节点没有在规定的时间内向发送 Ping 消息的节点返回 Pong 消息，那么，发送 Ping 消息的节点就会将接收 Ping 消息的节点标注为疑似下线状态（Probable Fail，Pfail）。
- 检测信息传播
  集群中的各个节点会通过相互发送消息的方式来交换自己掌握的集群中各个节点的状态信息，如在线、疑似下线（Pfail）、下线（Fail）。例如，当一个主节点 A 通过消息得知主节点 B 认为主节点 C 疑似下线时，主节点 A 会更新自己保存的集群状态信息，将从 B 获得的下线报告保存起来。
- 基于检测信息作下线判决
  如果在一个集群里，超过半数的持有 Slot（槽）的主节点都将某个主节点 X 报告为疑似下线，那么，主节点 X 将被标记为下线（Fail），并广播出去，所有收到这条 Fail 消息的节点都会立即将主节点 X 标记为 Fail。至此，故障检测完成。

##### 选举

主节点被标记为 Fail 后，对应的从节点会发起投票，竞争升主。

- 从节点拉票
  当从节点发现自己复制的主节点状态为已下线时，从节点就会向集群广播一条请求消息，请求所有收到这条消息并且具有投票权的主节点给自己投票。

- 拉票优先级
  从节点在发现其主节点下线时，并非立即发起故障转移流程而进行“拉票”的，而是要等待一段时间，**在未来的某个时间点才发起选举**。

  ```
  mstime() + 500ms + random()%500ms + rank*1000ms
  ```

  rank即排名，排名是指当前从节点在下线主节点的所有从节点中的排名，排名主要是**根据复制数据量来定，复制数据量越多，排名越靠前**，因此，具有较多复制数据量的从节点可以更早发起故障转移流程，从而更可能成为新的主节点。

- 主节点投票
  如果一个主节点具有投票权（负责处理 Slot 的主节点)，并且这个主节点尚未投票给其它从节点，那么这个主节点将向请求投票的从节点返回一条回应消息，表示支持该从节点升主。

- 根据投票结果决策
  在一个具有 N 个主节点投票的集群中，理论上每个参与拉票的从节点都可以收到一定数量的主节点投票，但是，在同一轮选举中，**只可能有一个从节点收到的票数大于 N/2 + 1**，也只有这个从节点可以升级为主节点，并代替已下线的主节点继续工作。

- 选举失败
  没有一个候选从节点获得超过半数的主节点投票。遇到这种情况，集群将会进入下一轮选举，直到选出新的主节点为止。

- 选举算法
  选举新主节点的算法是基于 Raft 算法的 Leader Election 方法来实现的。

##### 故障转移

选举完成后，获胜的从节点将发起故障转移（Failover），角色从 Slave 切换为 Master，并接管原来主节点的 Slots，详细过程如下。

- 新的主节点会通过轮询所有 Slot，撤销所有对已下线主节点的 Slot 指派，消除影响，并且将这些 Slot 全部指派给自己。
- 新的主节点会向集群中广播一条 Pong 消息，将自己升主的信息通知到集群中所有节点。
- 新的主节点开始处理自己所负责 Slot 对应的请求，至此，故障转移完成。

#### 3.8.集群模式的优缺点

#### 优点

采用去中心化思想，数据按照 slot 存储分布在多个节点，节点间数据共享，可动态调整数据分布;

可扩展性：可线性扩展到 1000 多个节点，节点可动态添加或删除;

高可用性：部分节点不可用时，集群仍可用。通过增加 Slave 做 standby 数据副本，能够实现故障自动 failover，节点之间通过 gossip 协议交换状态信息，用投票机制完成 Slave 到 Master 的角色提升;

降低运维成本，提高系统的扩展性和可用性。

#### 缺点

1.Redis Cluster 是无中心节点的集群架构，依靠 Goss 协议(谣言传播)协同自动化修复集群的状态

但 GosSIp 有消息延时和消息冗余的问题，在集群节点数量过多的时候，节点之间需要不断进行 PING/PANG 通讯，不必须要的流量占用了大量的网络资源。虽然 Reds4.0 对此进行了优化，但这个问题仍然存在。

2.数据迁移问题

Redis Cluster 可以进行节点的动态扩容缩容，这一过程，在目前实现中，还处于半自动状态，需要人工介入。在扩缩容的时候，需要进行数据迁移。

而 Redis 为了保证迁移的一致性，迁移所有操作都是同步操作，执行迁移时，两端的 Redis 均会进入时长不等的阻塞状态，对于小 Key，该时间可以忽略不计，但如果一旦 Key 的内存使用过大，严重的时候会接触发集群内的故障转移，造成不必要的切换。

##### 四、总结

主从模式：master 节点挂掉后，需要手动指定新的 master，可用性不高，基本不用。

哨兵模式：master 节点挂掉后，哨兵进程会主动选举新的 master，可用性高，但是每个节点存储的数据是一样的，浪费内存空间。数据量不是很多，集群规模不是很大，需要自动容错容灾的时候使用。

集群模式：数据量比较大，QPS 要求较高的时候使用。 **Redis Cluster 是 Redis 3.0 以后才正式推出，时间较晚，目前能证明在大规模生产环境下成功的案例还不是很多，需要时间检验。**



## Redis作为缓存

[参考](https://blog.csdn.net/youth_shouting/article/details/111788869)

[参考2](http://www.tkxiong.com/archives/1979)

[参考3](https://blog.csdn.net/youth_shouting/article/details/111788869)

### 1. 设置并读取配置文件



### 2. 定义缓存接口

```
type CacheStore interface {
	Get(key string,value interface{}) error
	Set(key string,value interface{},expires time.Duration) error
	Add(key string,value interface{},expires time.Duration) error
	Delete(key string) error
	Replace(key string,value interface{},expire time.Duration) error
}
```



### 什么是缓存雪崩、击穿、穿透？

用户的数据一般都是存储于数据库，数据库的数据是落在磁盘上的，磁盘的读写速度可以说是计算机里最慢的硬件了。

当用户的请求，都访问数据库的话，请求数量一上来，数据库很容易就奔溃的了，所以为了避免用户直接访问数据库，会用 Redis 作为缓存层。

因为 Redis 是内存数据库，我们可以将数据库的数据缓存在 Redis 里，相当于数据缓存在内存，内存的读写速度比硬盘快好几个数量级，这样大大提高了系统性能。

![图片](https://img-blog.csdnimg.cn/img_convert/37e4378d2edcb5e217b00e5f12973efd.png)

引入了缓存层，就会有缓存异常的三个问题，分别是**缓存雪崩、缓存击穿、缓存穿透**。

这三个问题也是面试中很常考察的问题，我们不光要清楚地知道它们是怎么发生，还需要知道如何解决它们。

话不多说，**发车！**

![图片](https://img-blog.csdnimg.cn/img_convert/61781cd6d82e4a0cc5f7521333049f0d.png)

------

#### 缓存雪崩

通常我们为了保证缓存中的数据与数据库中的数据一致性，会给 Redis 里的数据设置过期时间，当缓存数据过期后，用户访问的数据如果不在缓存里，业务系统需要重新生成缓存，因此就会访问数据库，并将数据更新到 Redis 里，这样后续请求都可以直接命中缓存。

![图片](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/e2b8d2eb5536aa71664772457792ec40.png)

那么，当**大量缓存数据在同一时间过期（失效）或者 Redis 故障宕机**时，如果此时有大量的用户请求，都无法在 Redis 中处理，于是全部请求都直接访问数据库，从而导致数据库的压力骤增，严重的会造成数据库宕机，从而形成一系列连锁反应，造成整个系统崩溃，这就是**缓存雪崩**的问题。

![图片](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/717343a0da7a1b05edab1d1cdf8f28e5.png)

可以看到，发生缓存雪崩有两个原因：

- 大量数据同时过期；
- Redis 故障宕机；

不同的诱因，应对的策略也会不同。

##### 大量数据同时过期

针对大量数据同时过期而引发的缓存雪崩问题，常见的应对方法有下面这几种：

- **均匀设置过期时间**；
- **互斥锁**；
- 双 key 策略；
- 后台更新缓存；

*1. 均匀设置过期时间*

如果要给缓存数据设置过期时间，应该避免将大量的数据设置成同一个过期时间。我们可以在对缓存数据设置过期时间时，**给这些数据的过期时间加上一个随机数**，这样就保证数据不会在同一时间过期。

*2. 互斥锁*

当业务线程在处理用户请求时，**如果发现访问的数据不在 Redis 里，就加个互斥锁，保证同一时间内只有一个请求来构建缓存**（从数据库读取数据，再将数据更新到 Redis 里），当缓存构建完成后，再释放锁。未能获取互斥锁的请求，要么等待锁释放后重新读取缓存，要么就返回空值或者默认值。

实现互斥锁的时候，最好设置**超时时间**，不然第一个请求拿到了锁，然后这个请求发生了某种意外而一直阻塞，一直不释放锁，这时其他请求也一直拿不到锁，整个系统就会出现无响应的现象。

*3. 双 key 策略*

我们对**缓存数据可以使用两个 key**，一个是**主 key，会设置过期时间**，一个是**备 key，不会设置过期**，它们只是 key 不一样，但是 value 值是一样的，相当于给缓存数据做了个副本。

当业务线程访问不到「主 key 」的缓存数据时，就直接返回「备 key 」的缓存数据，然后在更新缓存的时候，**同时更新「主 key 」和「备 key 」的数据。**

*4. 后台更新缓存*

业务线程不再负责更新缓存，缓存也不设置有效期，而是**让缓存“永久有效”，并将更新缓存的工作交由后台线程定时更新**。

事实上，缓存数据不设置有效期，并不是意味着数据一直能在内存里，因为**当系统内存紧张的时候，有些缓存数据会被“淘汰”**，而在缓存被“淘汰”到下一次后台定时更新缓存的这段时间内，业务线程读取缓存失败就返回空值，业务的视角就以为是数据丢失了。

解决上面的问题的方式有两种。

第一种方式，后台线程不仅负责**定时更新缓存**，而且也负责**频繁地检测缓存是否有效**，检测到缓存失效了，原因可能是系统紧张而被淘汰的，于是就要马上从数据库读取数据，并更新到缓存。

这种方式的检测时间间隔不能太长，太长也导致用户获取的数据是一个空值而不是真正的数据，所以检测的间隔最好是毫秒级的，但是总归是有个间隔时间，用户体验一般。

第二种方式，在业务线程发现缓存数据失效后（缓存数据被淘汰），**通过消息队列发送一条消息通知后台线程更新缓存**，后台线程收到消息后，在更新缓存前可以判断缓存是否存在，存在就不执行更新缓存操作；不存在就读取数据库数据，并将数据加载到缓存。这种方式相比第一种方式缓存的更新会更及时，用户体验也比较好。

在业务刚上线的时候，我们最好提前把数据缓起来，而不是等待用户访问才来触发缓存构建，这就是所谓的**缓存预热**，后台更新缓存的机制刚好也适合干这个事情。

##### Redis 故障宕机

针对 Redis 故障宕机而引发的缓存雪崩问题，常见的应对方法有下面这几种：

- 服务熔断或请求限流机制；
- 构建 Redis 缓存高可靠集群；

*1. 服务熔断或请求限流机制*

因为 Redis 故障宕机而导致缓存雪崩问题时，我们可以启动**服务熔断**机制，**暂停业务应用对缓存服务的访问，直接返回错误**，不用再继续访问数据库，从而降低对数据库的访问压力，保证数据库系统的正常运行，然后等到 Redis 恢复正常后，再允许业务应用访问缓存服务。

服务熔断机制是保护数据库的正常允许，但是暂停了业务应用访问缓存服系统，全部业务都无法正常工作

为了减少对业务的影响，我们可以启用**请求限流**机制，**只将少部分请求发送到数据库进行处理，再多的请求就在入口直接拒绝服务**，等到 Redis 恢复正常并把缓存预热完后，再解除请求限流的机制。

*2. 构建 Redis 缓存高可靠集群*

服务熔断或请求限流机制是缓存雪崩发生后的应对方案，我们最好通过**主从节点的方式构建 Redis 缓存高可靠集群**。

如果 Redis 缓存的主节点故障宕机，从节点可以切换成为主节点，继续提供缓存服务，避免了由于 Redis 故障宕机而导致的缓存雪崩问题。

------

#### 缓存击穿

我们的业务通常会有几个数据会被频繁地访问，比如秒杀活动，这类被频地访问的数据被称为热点数据。

如果缓存中的**某个热点数据过期**了，此时大量的请求访问了该热点数据，就无法从缓存中读取，直接访问数据库，数据库很容易就被高并发的请求冲垮，这就是**缓存击穿**的问题。

![图片](https://img-blog.csdnimg.cn/img_convert/acb5f4e7ef24a524a53c39eb016f63d4.png)

可以发现缓存击穿跟缓存雪崩很相似，你可以认为缓存击穿是缓存雪崩的一个子集。

应对缓存击穿可以采取前面说到两种方案：

- **互斥锁方案，保证同一时间只有一个业务线程更新缓存**，未能获取互斥锁的请求，要么等待锁释放后重新读取缓存，要么就返回空值或者默认值。
- 不**给热点数据设置过期时间，由后台异步更新缓存，或者在热点数据准备要过期前**，提前通知后台线程更新缓存以及重新设置过期时间；

------

#### 缓存穿透

当发生缓存雪崩或击穿时，数据库中还是保存了应用要访问的数据，一旦缓存恢复相对应的数据，就可以减轻数据库的压力，而缓存穿透就不一样了。

当用户访问的数据，**既不在缓存中，也不在数据库中**，导致请求在访问缓存时，发现缓存缺失，再去访问数据库时，发现数据库中也没有要访问的数据，没办法构建缓存数据，来服务后续的请求。那么当有大量这样的请求到来时，数据库的压力骤增，这就是**缓存穿透**的问题。

![图片](https://img-blog.csdnimg.cn/img_convert/b7031182f770a7a5b3c82eaf749f53b0.png)

缓存穿透的发生一般有这两种情况：

- **业务误操作**，缓存中的数据和数据库中的数据都被误删除了，所以导致缓存和数据库中都没有数据；
- **黑客恶意攻击**，故意大量访问某些读取不存在数据的业务；

应对缓存穿透的方案，常见的方案有三种。

- 第一种方案，**非法请求的限制**；
- 第二种方案，**缓存空值或者默认值**；
- 第三种方案，**使用布隆过滤器快速判断数据是否存在，避免通过查询数据库来判断数据是否存在**；

第一种方案，非法请求的限制

当有大量恶意请求访问不存在的数据的时候，也会发生缓存穿透，因此**在 API 入口处我们要判断求请求参数是否合理，请求参数是否含有非法值、请求字段是否存在**，如果判断出是恶意请求就直接返回错误，避免进一步访问缓存和数据库。

第二种方案，缓存空值或者默认值

当我们线上业务发现缓存穿透的现象时，可以针对查询的数据，**在缓存中设置一个空值或者默认值**，这样后续请求就可以从缓存中读取到空值或者默认值，返回给应用，而不会继续查询数据库。

*第三种方案，使用布隆过滤器快速判断数据是否存在，避免通过查询数据库来判断数据是否存在。*

我们可以在写入数据库数据时，**使用布隆过滤器做个标记，然后在用户请求到来时，业务线程确认缓存失效后，可以通过查询布隆过滤器快速判断数据是否存在，如果不存在，就不用通过查询数据库来判断数据是否存在**。

即使发生了缓存穿透，大量请求只会查询 Redis 和布隆过滤器，而不会查询数据库，保证了数据库能正常运行，Redis 自身也是支持布隆过滤器的。

那问题来了，布隆过滤器是如何工作的呢？接下来，我介绍下。

布隆过滤器由「初始值都为 0 的位图数组」和「 N 个哈希函数」两部分组成。当我们在写入数据库数据时，在布隆过滤器里做个标记，这样下次查询数据是否在数据库时，只需要查询布隆过滤器，如果查询到数据没有被标记，说明不在数据库中。

布隆过滤器会通过 3 个操作完成标记：

- 第一步，使用 N 个哈希函数分别对数据做哈希计算，得到 N 个哈希值；
- 第二步，将第一步得到的 N 个哈希值对位图数组的长度取模，得到每个哈希值在位图数组的对应位置。
- 第三步，将每个哈希值在位图数组的对应位置的值设置为 1；

举个例子，假设有一个位图数组长度为 8，哈希函数 3 个的布隆过滤器。

![图片](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/86b0046c2622b2c4bda697f9bc0f5b28.png)

在数据库写入数据 x 后，把数据 x 标记在布隆过滤器时，数据 x 会被 3 个哈希函数分别计算出 3 个哈希值，然后在对这 3 个哈希值对 8 取模，假设取模的结果为 1、4、6，然后把位图数组的第 1、4、6 位置的值设置为 1。**当应用要查询数据 x 是否数据库时，通过布隆过滤器只要查到位图数组的第 1、4、6 位置的值是否全为 1，只要有一个为 0，就认为数据 x 不在数据库中**。

布隆过滤器由于是基于哈希函数实现查找的，高效查找的同时**存在哈希冲突的可能性**，比如数据 x 和数据 y 可能都落在第 1、4、6 位置，而事实上，可能数据库中并不存在数据 y，存在误判的情况。

所以，**查询布隆过滤器说数据存在，并不一定证明数据库中存在这个数据，但是查询到数据不存在，数据库中一定就不存在这个数据**。

#### 总结

缓存异常会面临的三个问题：缓存雪崩、击穿和穿透。

其中，**缓存雪崩和缓存击穿主要原因是数据不在缓存中，而导致大量请求访问了数据库**，数据库压力骤增，容易引发一系列连锁反应，导致系统奔溃。不过，一旦数据被重新加载回缓存，应用又可以从缓存快速读取数据，不再继续访问数据库，数据库的压力也会瞬间降下来。因此，缓存雪崩和缓存击穿应对的方案比较类似。

而**缓存穿透主要原因是数据既不在缓存也不在数据库中**。因此，缓存穿透与缓存雪崩、击穿应对的方案不太一样。

我这里整理了表格，你可以从下面这张表格很好的知道缓存雪崩、击穿和穿透的区别以及应对方案。

![图片](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/061e2c04e0ebca3425dd75dd035b6b7b.png)

### 数据库和缓存如何保证一致性？

阿旺登陆到了服务器，经过一番排查后，确认服务器的**性能瓶颈是在数据库**。

这好办，给服务器加上 Redis，让其作为数据库的缓存。

这样，在客户端请求数据时，如果能在缓存中命中数据，那就查询缓存，不用在去查询数据库，从而减轻数据库的压力，提高服务器的性能。

#### 先更新数据库，还是先更新缓存？

阿旺有了这个想法后，就准备开始着手优化服务器，但是挡在在他前面的是这样的一个问题。

![图片](https://img-blog.csdnimg.cn/img_convert/b3bc9c4851ed731a36c9fee0f64264fe.png)

**由于引入了缓存，那么在数据更新时，不仅要更新数据库，而且要更新缓存，这两个更新操作存在前后的问题**：

- 先更新数据库，再更新缓存；
- 先更新缓存，再更新数据库；

阿旺没想到太多，他觉得最新的数据肯定要先更新数据库，这样才可以确保数据库里的数据是最新的，于是他就采用了「**先更新数据库，再更新缓存**」的方案。

阿旺经过几个夜晚的折腾，终于「优化好了服务器」，然后就直接上线了，自信心满满跑去跟老板汇报。

老板不懂技术，自然也没多虑，就让后续阿旺观察下服务器的情况，如果效果不错，就跟阿旺谈画饼的事情。

阿旺观察了好几天，发现数据库的压力大大减少了，访问速度也提高了不少，心想这事肯定成的了。

好景不长，突然老板收到一个客户的投诉，客户说他刚发起了**两次更新年龄的操作**，但是显示的年龄确还是第一次更新时的年龄，而第二次更新年龄并没有生效。

老板立马就找了阿旺，训斥着阿旺说：「*这么简单的更新操作，都有 bug？我脸往哪儿放？你的饼还要不要了？*」

听到自己准备到手的饼要没了的阿旺瞬间就慌了，立马登陆服务器排查问题，阿旺查询缓存和数据库的数据后发现了问题。

数据库的数据是客户第二次更新操作的数据，而缓存确还是第一次更新操作的数据，也就是**出现了数据库和缓存的数据不一致的问题**。

这个问题可大了，阿旺经过一轮的分析，造成缓存和数据库的数据不一致的现象，是因为**并发问题**！

#### 先更新数据库，再更新缓存

举个例子，比如「请求 A 」和「请求 B 」两个请求，同时更新「同一条」数据，则可能出现这样的顺序：

![图片](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/8febac10b14bed16cb96d1d944cd08da.png)

A 请求先将数据库的数据更新为 1，然后在更新缓存前，请求 B 将数据库的数据更新为 2，紧接着也把缓存更新为 2，然后 A 请求更新缓存为 1。

此时，数据库中的数据是 2，而缓存中的数据却是 1，**出现了缓存和数据库中的数据不一致的现象**。

#### [#](https://xiaolincoding.com/redis/architecture/mysql_redis_consistency.html#先更新缓存-再更新数据库)先更新缓存，再更新数据库

那换成「**先更新缓存，再更新数据库**」这个方案，还会有问题吗？

依然还是存在并发的问题，分析思路也是一样。

假设「请求 A 」和「请求 B 」两个请求，同时更新「同一条」数据，则可能出现这样的顺序：

![图片](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/454a8228a6549176ad7e0484fba3c92b.png)

A 请求先将缓存的数据更新为 1，然后在更新数据库前，B 请求来了， 将缓存的数据更新为 2，紧接着把数据库更新为 2，然后 A 请求将数据库的数据更新为 1。

此时，数据库中的数据是 1，而缓存中的数据却是 2，**出现了缓存和数据库中的数据不一致的现象**。

所以，**无论是「先更新数据库，再更新缓存」，还是「先更新缓存，再更新数据库」，这两个方案都存在并发问题，当两个请求并发更新同一条数据的时候，可能会出现缓存和数据库中的数据不一致的现象**。

#### 先更新数据库，还是先删除缓存？

阿旺定位出问题后，思考了一番后，决定在更新数据时，**不更新缓存，而是删除缓存中的数据。然后，到读取数据时，发现缓存中没了数据之后，再从数据库中读取数据，更新到缓存中。**

阿旺想的这个策略是有名字的，是叫 **Cache Aside 策略**，中文是叫旁路缓存策略。

该策略又可以细分为「读策略」和「写策略」。

![图片](https://img-blog.csdnimg.cn/img_convert/6e3db3ba2f829ddc14237f5c7c00e7ce.png)

**写策略的步骤：**

- 更新数据库中的数据；
- 删除缓存中的数据。

**读策略的步骤：**

- 如果读取的数据命中了缓存，则直接返回数据；
- 如果读取的数据没有命中缓存，则从数据库中读取数据，然后将数据写入到缓存，并且返回给用户。

阿旺在想到「写策略」的时候，又陷入更深层次的思考，到底该选择哪种顺序呢？

- 先删除缓存，再更新数据库；
- 先更新数据库，再删除缓存。

阿旺这次经过上次教训，不再「想当然」的乱选方案，因为老板这次给的饼很大啊，必须把握住。

于是阿旺用并发的角度来分析，看看这两种方案哪个可以保证数据库与缓存的数据一致性。

#### 先删除缓存，再更新数据库

阿旺还是以用户表的场景来分析。

假设某个用户的年龄是 20，请求 A 要更新用户年龄为 21，所以它会删除缓存中的内容。这时，另一个请求 B 要读取这个用户的年龄，它查询缓存发现未命中后，会从数据库中读取到年龄为 20，并且写入到缓存中，然后请求 A 继续更改数据库，将用户的年龄更新为 21。

![图片](https://img-blog.csdnimg.cn/img_convert/cc208c2931b4e889d1a58cb655537767.png)

最终，该用户年龄在缓存中是 20（旧值），在数据库中是 21（新值），缓存和数据库的数据不一致。

可以看到，**先删除缓存，再更新数据库，在「读 + 写」并发的时候，还是会出现缓存和数据库的数据不一致的问题**。

#### 先更新数据库，再删除缓存

继续用「读 + 写」请求的并发的场景来分析。

假如某个用户数据在缓存中不存在，请求 A 读取数据时从数据库中查询到年龄为 20，在未写入缓存中时另一个请求 B 更新数据。它更新数据库中的年龄为 21，并且清空缓存。这时请求 A 把从数据库中读到的年龄为 20 的数据写入到缓存中。

![图片](https://img-blog.csdnimg.cn/img_convert/1cc7401143e79383ead96582ac11b615.png)

最终，该用户年龄在缓存中是 20（旧值），在数据库中是 21（新值），缓存和数据库数据不一致。

从上面的理论上分析，先更新数据库，再删除缓存也是会出现数据不一致性的问题，**但是在实际中，这个问题出现的概率并不高**。

**因为缓存的写入通常要远远快于数据库的写入**，所以在实际中很难出现请求 B 已经更新了数据库并且删除了缓存，请求 A 才更新完缓存的情况。

而一旦请求 A 早于请求 B 删除缓存之前更新了缓存，那么接下来的请求就会因为缓存不命中而从数据库中重新读取数据，所以不会出现这种不一致的情况。

所以，**「先更新数据库 + 再删除缓存」的方案，是可以保证数据一致性的**。

而且阿旺为了确保万无一失，还给缓存数据加上了「**过期时间**」，就算在这期间存在缓存数据不一致，有过期时间来兜底，这样也能达到最终一致。

阿旺思考到这一步后，觉得自己真的是个小天才，因为他竟然想到了个「天衣无缝」的方案，他二话不说就采用了这个方案，又经过几天的折腾，终于完成了。

他自信满满的向老板汇报，已经解决了上次客户的投诉的问题了。老板觉得阿旺这小伙子不错，这么快就解决问题了，然后让阿旺在观察几天。

事情哪有这么顺利呢？结果又没过多久，老板又收到客户的投诉了，**说自己明明更新了数据，但是数据要过一段时间才生效**，客户接受不了。

老板面无表情的找上阿旺，让阿旺尽快查出问题。

阿旺得知又有 Bug 就更慌了，立马就登录服务器去排查问题，查看日志后得知了原因。

「先更新数据库， 再删除缓存」其实是两个操作，前面的所有分析都是建立在这两个操作都能同时执行成功，而这次客户投诉的问题就在于，**在删除缓存（第二个操作）的时候失败了，导致缓存中的数据是旧值**。

好在之前给缓存加上了过期时间，所以才会出现客户说的过一段时间才更新生效的现象，假设如果没有这个过期时间的兜底，那后续的请求读到的就会一直是缓存中的旧数据，这样问题就更大了。

所以新的问题来了，**如何保证「先更新数据库 ，再删除缓存」这两个操作能执行成功？**

阿旺分析出问题后，慌慌张张的向老板汇报了问题。

老板知道事情后，又给了阿旺几天来解决这个问题，画饼的事情这次没有再提了。

**阿旺会用什么方式来解决这个问题呢？**

**老板画的饼事情，能否兑现给阿旺呢？**

预知后事，且听下回阿旺的故事。

![图片](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/e51903be6ada20f87bb1c8015ba30631.png)

#### 小结

阿旺的事情就聊到这，我们继续说点其他。

「先更新数据库，再删除缓存」的方案虽然保证了数据库与缓存的数据一致性，但是每次更新数据的时候，缓存的数据都会被删除，这样会对缓存的命中率带来影响。

所以，**如果我们的业务对缓存命中率有很高的要求，我们可以采用「更新数据库 + 更新缓存」的方案，因为更新缓存并不会出现缓存未命中的情况**。

但是这个方案前面我们也分析过，在两个更新请求并发执行的时候，会出现数据不一致的问题，因为更新数据库和更新缓存这两个操作是独立的，而我们又没有对操作做任何并发控制，那么当两个线程并发更新它们的话，就会因为写入顺序的不同造成数据的不一致。

所以我们得增加一些手段来解决这个问题，这里提供两种做法：

- 在更新缓存前先加个**分布式锁**，保证同一时间只运行一个请求更新缓存，就会不会产生并发问题了，当然引入了锁后，对于写入的性能就会带来影响。
- 在更新完缓存时，给缓存加上较短的**过期时间**，这样即时出现缓存不一致的情况，缓存的数据也会很快过期，对业务还是能接受的。

对了，针对「先删除缓存，再更新数据库」方案在「读 + 写」并发请求而造成缓存不一致的解决办法是「**延迟双删**」。

延迟双删实现的伪代码如下：

```text
#删除缓存
redis.delKey(X)
#更新数据库
db.update(X)
#睡眠
Thread.sleep(N)
#再删除缓存
redis.delKey(X)
```

加了个睡眠时间，主要是为了确保请求 A 在睡眠的时候，请求 B 能够在这这一段时间完成「从数据库读取数据，再把缺失的缓存写入缓存」的操作，然后请求 A 睡眠完，再删除缓存。

所以，请求 A 的睡眠时间就需要大于请求 B 「从数据库读取数据 + 写入缓存」的时间。

但是具体睡眠多久其实是个**玄学**，很难评估出来，所以这个方案也只是**尽可能**保证一致性而已，极端情况下，依然也会出现缓存不一致的现象。

因此，还是比较建议用「先更新数据库，再删除缓存」的方案。

------

#### 前情回顾

上回程序员阿旺为了提升数据访问的性能，引入 Redis 作为 MySQL 缓存层，但是这件事情并不是那么简单，因为还要考虑 Redis 和 MySQL 双写一致性的问题。

阿旺经过一番周折，最终选用了「**先更新数据库，再删缓存**」的策略，原因是这个策略即使在并发读写时，也能最大程度保证数据一致性。

聪明的阿旺还搞了个兜底的方案，就是给缓存加上了过期时间。

本以为就这样不会在出现数据一致性的问题，结果将功能上线后，老板还是收到用户的投诉「说自己明明更新了数据，但是数据要过一段时间才生效」，客户接受不了。

老板转告给了阿旺，阿旺得知又有 Bug 就更慌了，立马就登录服务器去排查问题，查看日志后得知了原因。

「先更新数据库， 再删除缓存」其实是两个操作，这次客户投诉的问题就在于，**在删除缓存（第二个操作）的时候失败了，导致缓存中的数据是旧值，而数据库是最新值**。

好在之前给缓存加上了过期时间，所以才会出现客户说的过一段时间才更新生效的现象，假设如果没有这个过期时间的兜底，那后续的请求读到的就会一直是缓存中的旧数据，这样问题就更大了。

所以新的问题来了，**如何保证「先更新数据库 ，再删除缓存」这两个操作能执行成功？**

阿旺分析出问题后，慌慌张张的向老板汇报了问题。

老板知道事情后，又给了阿旺几天来解决这个问题，画饼的事情这次没有再提了。

- 阿旺会用什么方式来解决这个问题呢？
- 老板画的饼事情，能否兑现给阿旺呢？

#### 如何保证两个操作都能执行成功？

这次用户的投诉是因为在删除缓存（第二个操作）的时候失败了，导致缓存还是旧值，而数据库是最新值，造成数据库和缓存数据不一致的问题，会对敏感业务造成影响。

举个例子，来说明下。

应用要把数据 X 的值从 1 更新为 2，先成功更新了数据库，然后在 Redis 缓存中删除 X 的缓存，但是这个操作却失败了，这个时候数据库中 X 的新值为 2，Redis 中的 X 的缓存值为 1，出现了数据库和缓存数据不一致的问题。

![图片](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/2a2ea2854bbc3ae8ae86d7da45fa32ee.png)

那么，后续有访问数据 X 的请求，会先在 Redis 中查询，因为缓存并没有 诶删除，所以会缓存命中，但是读到的却是旧值 1。

其实不管是先操作数据库，还是先操作缓存，只要第二个操作失败都会出现数据一致的问题。

问题原因知道了，该怎么解决呢？有两种方法：

- 重试机制。
- 订阅 MySQL binlog，再操作缓存。

先来说第一种。

##### 重试机制

我们可以引入**消息队列**，将第二个操作（删除缓存）要操作的数据加入到消息队列，由消费者来操作数据。

- 如果应用**删除缓存失败**，可以从消息队列中重新读取数据，然后再次删除缓存，这个就是**重试机制**。当然，如果重试超过的一定次数，还是没有成功，我们就需要向业务层发送报错信息了。
- 如果**删除缓存成功**，就要把数据从消息队列中移除，避免重复操作，否则就继续重试。

举个例子，来说明重试机制的过程。

![图片](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/a4440f0d572612e0832b903e4a62bd2b.png)

##### 订阅 MySQL binlog，再操作缓存

「**先更新数据库，再删缓存**」的策略的第一步是更新数据库，那么更新数据库成功，就会产生一条变更日志，记录在 binlog 里。

于是我们就可以通过订阅 binlog 日志，拿到具体要操作的数据，然后再执行缓存删除，阿里巴巴开源的 Canal 中间件就是基于这个实现的。

Canal 模拟 MySQL 主从复制的交互协议，把自己伪装成一个 MySQL 的从节点，向 MySQL 主节点发送 dump 请求，MySQL 收到请求后，就会开始推送 Binlog 给 Canal，Canal 解析 Binlog 字节流之后，转换为便于读取的结构化数据，供下游程序订阅使用。

下图是 Canal 的工作原理：

![图片](https://img-blog.csdnimg.cn/img_convert/2ee2280e9f59b6b4879ebdec6eb0cf52.png)

所以，**如果要想保证「先更新数据库，再删缓存」策略第二个操作能执行成功，我们可以使用「消息队列来重试缓存的删除」，或者「订阅 MySQL binlog 再操作缓存」，这两种方法有一个共同的特点，都是采用异步操作缓存。**



## [Redis作为消息队列](https://zhuanlan.zhihu.com/p/403638258)

[参考](https://blog.csdn.net/li_qinging/article/details/117450163)

### 概述

早期，基于Redis实现轻量化的消息队列有3种实现方式，分别是基于**List的LPUSH+BRPOP （BRPOPLPUSH）**的实现、**PUB/SUB发布订阅模式**以及基于**Sorted-Set**实现方式，但是，这三种模式分别有其相应的缺点。

|       实现方式        |                             缺点                             |
| :-------------------: | :----------------------------------------------------------: |
| 基于List的LPUSH+BRPOP | 做消费者确认ACK比较麻烦，不能保证消费者消费消息后是否成功处理的问题，通常需要维护一个额外的列表，且**不支持重复消费和分组消费**。 |
|  PUB/SUB发布订阅模式  | 若客户端不在线时发布消息会丢失，且消费者客户端出现消息积压，到一定程度，会被强制断开，导致消息意外丢失，可见**PUB/SUB模式不适合做消息存储，消息积压类的业务**。 |
|  基于Sorted-Set实现   | 由于集合的特点，**不允许重复消息，而且消息ID确定有误的话会导致消息的顺序出错**。 |

Redis5.0中发布的Stream类型，也用来实现典型的消息队列。该Stream类型的出现，几乎满足了消息队列具备的全部内容，包括但不限于：

- 消息ID的序列化生成
- 消息遍历
- 消息的阻塞和非阻塞读取
- 消息的分组消费
- 未完成消息的处理
- 消息队列监控

### 什么是Stream?

**Stream 实际上是一个具有消息发布/订阅功能的组件，也就常说的消息队列**。其实这种类似于 broker/consumer(生产者/消费者)的数据结构很常见，比如 RabbitMQ 消息中间件、Celery 消息中间件，以及 Kafka 分布式消息系统等，而 Redis Stream 正是借鉴了 Kafaka 系统。

#### 1) 优点

Strean 除了拥有很高的性能和内存利用率外, 它最大的特点就是提供了消息的持久化存储，以及主从复制功能，从而解决了网络断开、Redis 宕机情况下，消息丢失的问题，即便是重启 Redis，存储的内容也会存在。

#### 2) 流程

**Stream 消息队列主要由四部分组成，分别是：消息本身、生产者、消费者和消费组**，对于前述三者很好理解，下面了解什么是消费组。

一个 Stream 队列可以拥有多个消费组，每个消费组中又包含了多个消费者，组内消费者之间存在竞争关系。当某个消费者消费了一条消息时，同组消费者，都不会再次消费这条消息。被消费的消息 ID 会被放入等待处理的 Pending_ids 中。每消费完一条信息，消费组的游标就会向前移动一位，组内消费者就继续去争抢下消息。

Redis Stream 消息队列结构程如下图所示：



![Redis Stream结构图](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/15253613F-0.gif)
图1：Redis Stream流程处理图

**下面对上图涉及的专有名词做简单解释：**

- **Stream direction：表示数据流，它是一个消息链，将所有的消息都串起来，每个消息都有一个唯一标识 ID 和对应的消息内容**（Message content）。
- **Consumer Group ：表示消费组，拥有唯一的组名，使用 XGROUP CREATE 命令创建。**一个 Stream 消息链上可以有多个消费组，一个消费组内拥有多个消费者，每一个消费者也有一个唯一的 ID 标识。
- **last_delivered_id ：表示消费组游标，每个消费组都会有一个游标 last_delivered_id，任意一个消费者读取了消息都会使游标 last_delivered_id 往前移动。**
- **pending_ids ：Redis 官方称为 PEL，表示消费者的状态变量，它记录了当前已经被客户端读取的消息 ID，但是这些消息没有被 ACK(确认字符)。如果客户端没有 ACK，那么这个变量中的消息 ID 会越来越多，一旦被某个消息被 ACK，它就开始减少。**

#### 3) ACK 

**ACK（Acknowledge character）即确认字符**，在数据通信中，接收方传递给发送方的一种传输类控制字符。表示发来的数据已确认接收无误。在 TCP/IP 协议中，如果接收方成功的接收到数据，那么会回复一个 ACK 数据。**通常 ACK 信号有自己固定的格式，长度大小，由接收方回复给发送方。**

### 相关命令

Stream相关的命令主要可以分为2大类，一类是与消息队列相关的命令，另一类是与消费者组相关的命令。

**与消息队列相关的命令：**

| 命令      | 说明                                     |
| --------- | ---------------------------------------- |
| XADD      | 添加消息到末尾。                         |
| XTRIM     | 对 Stream 流进行修剪，限制长度。         |
| XDEL      | 删除指定的消息。                         |
| XLEN      | 获取流包含的元素数量，即消息长度。       |
| XRANGE    | 获取消息列表，会自动过滤已经删除的消息。 |
| XREVRANGE | 反向获取消息列表，ID 从大到小。          |
| XREAD     | 以阻塞或非阻塞方式获取消息列表。         |

与消费者组相关的命令：

| 命令                      | 说明                                         |
| ------------------------- | -------------------------------------------- |
| XGROUP CREATE             | 创建消费者组。                               |
| XREADGROUP GROUP          | 读取消费者组中的消息。                       |
| XACK                      | 将消息标记为"已处理"。                       |
| XGROUP SETID              | 为消费者组设置新的最后递送消息ID。           |
| XGROUP DELCONSUMER        | 删除消费者。                                 |
| XGROUP DESTROY            | 删除消费者组。                               |
| XPENDING                  | 显示待处理消息的相关信息。                   |
| XCLAIM                    | 转移消息的归属权。                           |
| XINFO                     | 查看 Stream 流、消费者和消费者组的相关信息。 |
| XINFO GROUPS              | 查看消费者组的信息。                         |
| XINFO STREAM              | 查看 Stream 流信息。                         |
| XINFO CONSUMERS key group | 查看组内消费者流信息。                       |

#### XADD

```
XADD key [NOMKSTREAM] [ MAXLEN | MINID [ = | ~] threshold [LIMIT count]] * | id field value [ field value ...]
```

解释：XADD命令用于往某个消息队列中添加消息。

- key：表示消息队列的名称，如果不存在就创建。
- [NOMKSTREAM]：可选参数，表示第一个参数key不存在时不创建。
- [MAXLEN|MINID [=|~] threshold [LIMIT count]] ：可选参数，**MAXLEN|MINID表示指定消息队列中消息的最大长度或者是消息ID的最小值**。**=|~表示设置精确的值或者是大约值**，**threshold 表示具体设置的值，超过threshold值以后，旧的消息将会被删掉**。LIMIT count如果设置了会被当做键值对的形式保存在消息体中的第一个位置，另外**设置LIMIT时MAXLEN和MINID只能使用~设置大约值（Redis6.2版本后才加入了LIMIT参数）**，**由于队列中的消息不会主动被删除**，但是在设置MAXLEN后，当消息队列长度超过MAXLEN时，会删除老的消息，保证消息队列长度不会一直累加。
- |ID：表示消息ID，*表示由Redis生成（建议方案），ID表示自己指定。
- field value [field value …]：**用于保存消息具体内容的键值对，可以传入多组键值对**。

#### XREAD 

```
XREAD [COUNT count] [BLOCK milliseconds] STREAMS key [key ...] id [id ...]
```

解释：XREAD命令用于从某个消息队列中读取消息，**分为阻塞模式和非阻塞模式**。

- [COUNT count]：可选参数，COUNT为关键字，**表示指定读取消息的数量**，count表示具体的值。
- [BLOCK milliseconds]：可选参数，**BLOCK为关键字，表示设置XREAD为阻塞模式，默认是非阻塞模式**，milliseconds表示具体阻塞的时间。
- STREAMS ：关键字。
- key [key …]：表示消息队列的名称，**可以传入多个消息队列名称**。
- ID [ID …]：用于表示从哪个消息ID开始读取（不包含），与前面的key一一对应。0表示从第一条消息开始。在阻塞模式中，可以使用$用于表示最新的消息ID。（在非阻塞模式下$无意义）。

#### XDEL

```
XDEL key id [id ...]
```

解释：XDEL命令用于进行消息删除，注意**XACK进行消息确认只是进行了标记，消息还是会存在消息队列中，并没有删除**。使用XDEL命令才会将消息从消息队列中删除。

- key：表示消息队列的名称。
- ID [ID …]：表示消息ID，可以传入多个消息ID。

#### XLEN

```
XLEN key
```

解释：XLEN命令用于获取消息队列的长度。

- key：表示消息队列的名称。

#### XRANGE

```
XRANGE key start end [COUNT count]
```

解释：**XRANGE命令用于获取消息队列中的消息**，和XREAD有点类似，**XREAD只能指定开始消息ID（不包含）**，**XRANGE可以指定开始和结束消息ID**。另外还有个XREVRANGE命令用于反向获取消息列表，与XRANGE不同的是消息ID是从大到小。

- key：表示消息队列的名称。
- start：表示起始消息ID（包含）。
- end：表示结束消息ID（包含）。
- [COUNT count]：COUNT为关键字，表示指定读取消息的数量，count表示具体的值。同XREAD命令。

####  XTRIM

```
XTRIM key MAXLEN | MINID [ = | ~] threshold [LIMIT count]
```

解释：XTRIM命令用于对消息队列进行修剪，限制长度。

- key：表示消息队列的名称。
- MAXLEN|MINID [=|~] threshold [LIMIT count]：同XADD命令中的同名可选参数意义相同。

#### XGROUP

##### CREATE

```
XGROUP CREATE key groupname id | $ [MKSTREAM] [ENTRIESREAD entries_read]
```

解释：XGROUP CREATE命令用于**创建消费者组**。

- CREATE：关键字，表示创建消费者组命令。
- key：表示消息队列的名称。
- groupname：表示要创建的消费者组名称。
- ID|$：表示该消费者组中的消费者将从哪个消息ID开始消费消息，ID表示指定的消息ID，$表示只消费新产生的消息。
- [MKSTREAM]：可选参数，表示在创建消费者组时，如果指定的消息队列不存在，会自动创建。但是这种方式创建的消息队列其长度为0

##### SETID

```
XGROUP SETID key groupname id | $ [ENTRIESREAD entries_read]
```

解释：XGROUP SETID命令用于设置消费者组中下一条要读取的消息ID。

- SETID：关键字，表示设置消费者组中下一条要读取的消息ID命令
- key：表示消息队列的名称。
- groupname：表示消费者组名称。
- ID|$：表示指定具体的消息ID，0可以表示重新开始处理消费者组中的所有消息，$**表示只处理消费者组中新产生的消息。**

##### DESTROY

```
XGROUP DESTROY key groupname
```

解释：XGROUP DESTROY命令用于销毁消费者组。

- DESTROY：关键字，表示销毁消费者组命令。
- key：表示消息队列的名称。
- groupname：表示要销毁的消费者组名称。

##### CREATECONSUMER

```
XGROUP CREATECONSUMER key groupname consumername
```

解释：XGROUP CREATECONSUMER命令用于**创建消费者**。

- CREATECONSUMER：关键字，表示创建消费者命令。
- key：表示消息队列的名称。
- groupname：表示要创建的消费者所属的消费者组名称。
- consumername：表示要创建的消费者名称。

##### DELCONSUMER

```
XGROUP DELCONSUMER key groupname consumername
```

解释：XGROUP DELCONSUMER命令用于**删除消费者**。

- DELCONSUMER：关键字，表示删除消费者命令。
- key：表示消息队列的名称。
- groupname：表示要删除的消费者所属的消费者组名称。
- consumername：表示要删除的消费者名称。

#### XREADGROUP

```
XREADGROUP GROUP group consumer [COUNT count] [BLOCK milliseconds] [NOACK] STREAMS key [key ...] id [id ...]
```

解释：XREADGROUP命令**用于分组消费消息**。

- GROUP：关键字。
- group：表示消费者组名称。
- consumer：表示消费者名称。
- [COUNT count]：可选参数，COUNT为关键字，表示指定读取消息的数量，count表示具体的值。同XREAD命令。
- [BLOCK milliseconds]：可选参数，可选参数，**BLOCK为关键字，表示设置XREAD为阻塞模式，默认是非阻塞模式**，milliseconds表示具体阻塞的时间。同XREAD命令。
- [NOACK]：可选参数，**表示不要将消息加入到PEL队列（Pending等待队列）中，相当于在消息读取时直接进行消息确认**。在可靠性要求不高和偶尔丢失消息可接受的场景下可以使用。
- STREAMS：关键字。
- key [key …]：表示消息队列的名称，可以传入多个消息队列名称。同XREAD命令。
- ID [ID …]：用于表示从哪个消息ID开始读取，与前面的key一一对应。0表示从第一条消息开始。在阻塞模式中，可以使用$用于表示最新的消息ID。（在非阻塞模式下$无意义）。同XREAD命令。

#### XPENDING

```
XPENDING key group [ [IDLE min-idle-time] start end count [consumer]]
```

解释：XPENDING命令**用于获取等待队列**，等待队列中保存的是消费者组内被读取，但是还未完成处理的消息，也就是还没有ACK的消息。

- key：表示消息队列的名称。
- group：表示消费者组名称。
- [IDLE min-idle-time]：可选参数，IDLE表示指定消息已读取时长，min-idle-time表示具体的值。
- start：表示起始消息ID（包含）。
- end：表示结束消息ID（包含）。
- count：指定读取消息的条数。
- [consumer]：可选参数，表示消费者名称。

#### XACK

```
XACK key group id [id ...]
```

解释：XACK命令用于进行消息确认。

- key：表示消息队列的名称。
- group：表示消费者组名称。
- ID [ID …]：表示消息ID，可以传入多个消息ID。

#### XCLAIM

```
XCLAIM key group consumer min-idle-time id [id ...] [IDLE ms] [TIME unix-time-milliseconds] [RETRYCOUNT count] [FORCE] [JUSTID]
```

解释：XCLAIM命令用于进行消息转移，当某个等待队列中的消息长时间没有被处理（没有ACK）的时候，可以用XCLAIM命令将其转移到其他消费者的等待列表中。

- key：表示消息队列的名称。
- group：表示消费者组名称。
- consumer：表示消费者名称。
- min-idle-time：表示消息空闲时长（表示消息已经读取，但还未处理）。
- ID [ID …]：可选参数，表示要转移的消息的消息ID，可传入多个消息ID。
- [IDLE ms]：可选参数，设置消息空闲时间（上一次读取消息的时间），如果未指定，这假定IDLE为0，即每次转移消息之后重置消息空闲时间。因为如果空闲时间一直累加的话，消息会一直转移。
- [TIME ms-unix-time]：可选参数，与IDLE参数相同，只是它将空闲时间设置为特定的Unix时间（以毫秒为单位），而不是相对的毫秒量。这对于重写生成XCLAIM命令的AOF文件非常有用。
- [RETRYCOUNT count]：可选参数，设置重试计数器的值，每次消息被读取时，该计数器都会递增。一般XCLAIM命令不需要修改重试计数器的值。
- [FORCE]：可选参数，即使指定要转移的消息的消息ID在其他等待列表中不存在，也强制将该消息ID加入到执行消费者的等待列表中。
- [JUSTID]：可选参数，仅返回要转移消息的消息ID，使用此参数意味着重试计数器不会递增。

#### XAUTOCLAIM 

```
XAUTOCLAIM key group consumer min-idle-time start [COUNT count] [JUSTID]
```

XAUTOCLAIM 等价于先调用 XPENDING 再调用 XCLAIM，但通过类似 SCAN 的语义提供了一种更直接的方式来处理消息传递失败。

#### XINFO

##### CONSUMERS

```
XINFO CONSUMERS key groupname
```

解释：XINFO CONSUMERS命令用于监控消费者。

- CONSUMERS：关键字，表示查看消费者信息命令。
- key：表示消息队列的名称。
- groupname：表示消费者组名称。

##### GROUPS

```
XINFO GROUPS key
```

解释：XINFO GROUPS命令用于监控消费者组。

- GROUPS：关键字，表示查看消费者组信息命令。
- key：表示消息队列的名称。

##### STREAM

```
XINFO STREAM key [FULL [COUNT count]]
```

解释：XINFO STREAM命令用于监控消息队列。

- STREAM：关键字，表示查看消息队列信息命令。
- key：表示消息队列的名称。

##### HELP

解释：XINFO HELP 命令用于获取帮助。

- HELP：关键字，表示获取帮助信息命令。

### XADD/XREAD模式和消费者组模式

#### XADD/XREAD模式

普通场景下，生产者生产消息，消费者消费消息，多个消费者可以重复的消费相同的消息，比较类似常规的发布订阅模式，订阅了某个消息队列的消费者，能够获取到生产者投放的消息。当然消费者可以采用阻塞或者非阻塞的模式进行读取消息，业务处理等。一个典型的阻塞模式使用方式如下：

```
#Producer
127.0.0.1:6379> XADD test-mq * key1 value1
"1622605684330-0"
127.0.0.1:6379> 
127.0.0.1:6379> XADD test-mq * key2 value2
"1622605691371-0"
127.0.0.1:6379> 
127.0.0.1:6379> XADD test-mq * key3 value3
"1622605698309-0"
127.0.0.1:6379> 
127.0.0.1:6379> XADD test-mq * key4 value4
"1622605707261-0"
127.0.0.1:6379> 
127.0.0.1:6379> XADD test-mq * key5 value5 key6 value6
"1622605714081-0"
127.0.0.1:6379>
```

```
#Consumer
127.0.0.1:6379> XREAD BLOCK 10000 STREAMS test-mq $
1) 1) "test-mq"
   2) 1) 1) "1622605684330-0"
         2) 1) "key1"
            2) "value1"
(3.32s)
127.0.0.1:6379> 
127.0.0.1:6379> XREAD BLOCK 10000 STREAMS test-mq $
1) 1) "test-mq"
   2) 1) 1) "1622605691371-0"
         2) 1) "key2"
            2) "value2"
(2.88s)
127.0.0.1:6379> 
127.0.0.1:6379> XREAD BLOCK 10000 STREAMS test-mq $
1) 1) "test-mq"
   2) 1) 1) "1622605698309-0"
         2) 1) "key3"
            2) "value3"
(3.37s)
127.0.0.1:6379> 
127.0.0.1:6379> XREAD BLOCK 10000 STREAMS test-mq $
1) 1) "test-mq"
   2) 1) 1) "1622605707261-0"
         2) 1) "key4"
            2) "value4"
(3.75s)
127.0.0.1:6379> 
127.0.0.1:6379> XREAD BLOCK 10000 STREAMS test-mq $
1) 1) "test-mq"
   2) 1) 1) "1622605714081-0"
         2) 1) "key5"
            2) "value5"
            3) "key6"
            4) "value6"
(2.47s)
127.0.0.1:6379>

```

说明：使用阻塞模式的XREAD，XREAD BLOCK 10000 STREAMS test-mq  `$`，最后一个参数`$`表示读取最新的消息，**所以需要先启动消费者，阻塞等待消息，然后生产者添加消息，消费者接受消息完成处理**。

### 消费者组模式

在有些场景下，我们**需要多个消费者配合来消费同一个消息队列中的消息，而不是多个消费者重复的消息，以此来提高消息处理的效率。这种模式也就是消费者组模式了**。消费者组模式如下图所示：

![consumergroup](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/20210602150314753.PNG)

下面是Redis Stream的结构图：

![stream](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/20210602115618156.PNG)

上图解释：

Consumer Group ：消费组，使用 XGROUP CREATE 命令创建，一个消费组有多个消费者(Consumer)。

last_delivered_id ：游标，每个消费组会有个游标 last_delivered_id，任意一个消费者读取了消息都会使游标 last_delivered_id 往前移动。

pending_ids ：消费者(Consumer)的状态变量，作用是维护消费者的未确认的 id。 pending_ids 记录了当前已经被客户端读取的消息，但是还没有 ack (Acknowledge character：确认字符）

### 基于[Go语言](https://so.csdn.net/so/search?q=Go语言&spm=1001.2101.3001.7020)封装Redis Stream客户端Demo

[参考地址](https://blog.csdn.net/li_qinging/article/details/117450163)



## 进阶

### Redis 慢查询

[参考](https://juejin.cn/post/6904197863820296200/)

#### 概述

Mysql 的慢查询我想各位好哥哥都很熟悉了，那所谓慢查询日志就是系统在命令执行前后计算每条命令的执行时间，当超过预设阀值，就将这条命令的相关信息（例如：发生时间，耗时，命令的详细信息）记录下来。这功能这么香，我大 Redis 怎么可能会没有。

#### 生命周期

在 Redis 中，执行一条命令可以拆分成四个步骤**发送命令、命令排队、命令执行、结果响应**，需要注意的是 Redis 中的**慢查询只会统计第三个步骤（命令执行）**。我画你猜
![生命周期](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dbdcac711d72428d9b0dac0ed6081afb~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 最佳实践

那问题来了，我们要怎么像 Mysql 一样设置一个时间，超过这个时间的就记录到慢查询日志里面呢？要怎么看这些慢查询日志呢？猛男我已经帮好哥哥们整理好了。

##### 1 配置阈值

Redis 提供了**lowlog-log-slower-than**配置来设置预设阀值，它的单位是微秒（1 秒=1000 毫秒=1000000 微秒），默认值是 10000，假如执行了一条“很慢”的命令（例如 keys*），如果它的执行时间超过了 10000 微秒，那么它将被记录在慢查询日志中。 需要注意的是如果 slowlog-log-slower-than=0 会记录所有的命令，slowlog-log-slower- than<0 对于任何命令都不会进行记录。

##### 2 日志存放

Redis 提供了**slowlog-max-len**命令，看字面意思只是说明了慢查询日志最多存储多少条，并没有说明存放在哪里是吧。是的，好哥哥们是对的。实际上 Redis **使用了一个列表来存储慢查询日志**，slowlog-max-len 就是列表的最大长度。一个新的命令满足**慢查询条件时被插入到这个列表中**，当慢查询日志列表**已处于其最大长度时，最早插入的一个命令将从列表中移出**，例如 slowlog-max-len 默认设置为 128，当有第 129 条慢查询插入的话，那么队头的第一条数据就出列，第 129 条慢查询就会入列。

##### 3 配置

Redis 提供了两种方式来该配置，第一种修改启动配置文件，这种方式的话需要找到好哥哥对应启动的 Redis 配置（千万别搞错了，不然试不出来的）。例如我的配置在/home/redis-4.0.0，然后执行下面的命令。连执行顺序、敲什么都弄出来了，确定不关注点赞吗（还没有点赞加关注的都是渣男，喜欢白嫖。哎哎哎，好了，别打了别打了）。
另一种是使用 config set 命令动态修改，这种方式的最好不要用默认的配置文件启动，不然持久化配置的时候会报错的（亲测）。

###### 第一种

```powershell
 ## 编辑配置，当然你可以使用linux的其他修改命令
 vim redis.conf
 ## 打开后使用英文/进行搜索
 /slowlog-log-slower-than
 ## 这个时候就能找到了，然后就是修改值按i编辑设置成1秒
 slowlog-log-slower-than 1000000
 ## 按ESC键退出编辑，然后保存,重启Redis，还不会重启，看我第二篇Linux、docker Redis安装与配置就好了
 :wq
```

###### 第二种

```powershell
 ## 进入redis客户端
 redis-cli
 ## 动态配置超时时间为1秒
 config set slowlog-log-slower-than 1000000
 ## 动态配置对列长度为500
 config set slowlog-max-len 500
 ## 持久化写入配置文件
 config rewrite
```

##### 4 获取日志

哎，好哥哥是不是也发现了，刚刚说的日志是存在队列中，那也没告诉我在哪个队列呀，我要怎么拿日志呢。嘿呀，Antirez 早就帮我们想好了（Antirez yyds）。提供以下的命令能获取到对应的慢查询日志。

```powershell
 ## 格式。n代表条数
 slowlog get [n]
 ## 格式。查看有多少条日志 
 slowlog len 
 ## 格式。重置日志 
 slowlog reset 
```

##### 5 日志解析

执行上面的命令会返回如图内容，分别是慢查询日志的标识 id、执行时间戳、命令耗时、执行命令和参数（这个好哥哥们真的要实操一下，不然真的不会呀）。
![日志结构](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/09123042c3d6447cb077a06d26877314%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A3024%3A0%3A0%3A0.awebp)

#### 建议

1. 线上建议调大慢查询列表参数 slowlog-max-len，记录慢查询时 Redis 会对长命令做截断操作，并不会占用大量内存。增大慢查询列表可以减缓慢查询被剔除的可能。
2. slowlog-log-slower-than 配置默认值超过 10 毫秒判定为慢查询，需要根据 Redis 并发量调整该值。由于 Redis 是单线程的，建议这个参数配置的低一点。
3. 慢查询只记录命令执行时间，并不包括命令排队和网络传输时间。因此客户端执行命令的时间会大于命令实际执行时间。
4. 由于慢查询日志是一个先进先出的队列，也就是说如果慢查询比较多的情况下，可能会丢失部分慢查询命令，为了防止这种情况发生，可以定期执行 slow get 命令将慢查询日志持久化到其他存储中（例如 MySQL）。

### Redis Pipeline

#### 概述

`发送命令`、`命令排队`、`命令执行`、`结果响应`四个步骤。由于 Redis 本身是基于 `Request/Response`协议（停等机制）的，虽然 Redis 已经提供了像 `mget` 、`mset` 这种批量的命令，但是好哥哥们想一下，如果某些操作根本就不支持或没有批量的操作，是不是就要一条一条的执行命令。那这样岂不是和我大 Redis 高性能背道而驰了（因为每执行一条命令都要消耗请求与响应的时间）。好哥哥们会问了，那有什么办法能解决这个问题呢，答案就是`Pipeline`。

`Pipeline` 字面意思是管道也可以说是流水线。`Pipeline`也并不是什么新的技术或机制，像在`Jenkins` 、`Netty` 都有运用到。 在 Redis 中通过`Pipeline`机制能改善上面这类问题，**它能将一组 Redis 命令进行组装，通过一次传输给 Redis 并返回结果集**。如下图
![对比](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/2b99717bf9c8477dbcf02e53b5001848%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A3024%3A0%3A0%3A0.awebp)

#### 怎么用

用的话很简单，`redis-cli` 的 `--pipe`参数实际上就是使用 Pipeline 机制，例如下面操作将 `set hello world` 和 `incr count` 两条命令组装。如下代码。当然，正常我们都是通过 Redis 客户端像 Jedis 来操作，这个会放在后面讲。

```powershell
 ## 格式
 echo -en '*3\r\n$3\r\nSET\r\n$5\r\nhello\r\n$5\r\nworld\r\n*2\r\n$4\r\nincr\r\
n$7\r\ncounter\r\n' | redis-cli --pipe
```

#### 原理

需要实现`Pipeline` 功能，需要客户端和服务器端的支持。

**Redis 服务器端支持处理一个客户端通过同一个 TCP 连接发来的多个命令**。可以理解为，这里将多个命令切分，和处理单个命令一样，处理完成后会将处理结果缓存起来，所有命令执行完毕后统一打包返回。这里就会涉及到一个问题就是如果`Pipeline`一次的命令条数过多时，会使响应结果撑爆`Socket`接收缓冲区，所以好哥哥们要控制一下每次的命令条数。

客户端方面像`Jedis`实现的逻辑是，通过 API 生成一个`Pipeline`对象，每次往`Pipeline`中添加命令时，`Jedis`会将命令写入（只是写入还没有发送给 Redis）到`Outputstream`（Jdedis 封装了自己的输入/输出流）。当真正调用获取结果时会调`flush()`方法然后阻塞获取返回结果，然后将结果包装返回给调用方。

#### 性能测试

在不同网络下，10000 条 `set` 非 Pipeline 和 Pipeline 的执行时间对比。 需要注意的是，针对于环境和网络的不同，测试出来的结果肯定是不一样的。如果网络延迟也高，那么 Pipeline 的性能肯定是越好的。 ![对比](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/aac710e8f290409988c1dda8a3068747%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A3024%3A0%3A0%3A0.awebp)

#### 批量命令、Pipeline 对比

1. 原生批量命令是原子的，Pipeline 是非原子的。
2. 原生批量命令是一个命令对应多个 key，Pipeline 支持多个命令。
3. 原生批量命令是 Redis 服务端支持实现的，而 Pipeline 需要服务端和客户端的共同实现

#### 适用场景

`Peline`是 Redis 的一个提高吞吐量的机制，适用于**多 key 读写场景**，比如同时读取多个`key` 的`value`，或者更新多个`key`的`value`，并且允许一定比例的**写入失败**、**实时性**也没那么高，那么这种场景就可以使用了。比如 10000 条一下进入 redis，可能失败了 2 条无所谓，后期有补偿机制就行了，像短信群发这种场景，这时候用 pipeline 最好了。

#### 注意点

1. `Pipeline`是非原子的，在上面原理解析那里已经说了就是 Redis 实际上还是一条一条的执行的，而执行命令是需要排队执行的，所以就会出现原子性问题。
2. `Pipeline`中包含的命令不要包含过多。
3. `Pipeline`每次只能作用在一个 Redis 节点上。
4. `Pipeline` 不支持事务，因为命令是一条一条执行的。

## 面试题

[参考](http://learn.lianglianglee.com/%E6%96%87%E7%AB%A0/%E6%9C%80%E5%85%A8%E7%9A%84%20116%20%E9%81%93%20Redis%20%E9%9D%A2%E8%AF%95%E9%A2%98%E8%A7%A3%E7%AD%94.md)

### 1、什么是 Redis? 

Redis 是完全开源免费的，遵守 BSD 协议，是一个高性能的 key-value 数据 库。 Redis 与其他 key - value 缓存产品有以下三个特点： 

- Redis 支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。 
- Redis 不仅仅支持简单的 key-value 类型的数据，同时还提供 list， set，zset，hash 等数据结构的存储。 
- Redis 支持数据的备份，即 master-slave 模式的数据备份。 

Redis 优势： 

- 性能极高 – Redis 能读的速度是 110000 次/s,写的速度是 81000 次 /s 。 
- 丰富的数据类型 – Redis 支持二进制案例的 Strings, Lists, Hashes,  Sets 及 Ordered Sets 数据类型操作。 
- 原子 – **Redis 的所有操作都是原子性**的，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性， 通过 MULTI 和 EXEC 指令包起来。
- 丰富的特性 – Redis 还支持 publish/subscribe, 通知, key 过期等等特性。

Redis 与其他 key-value 存储有什么不同？ 

- **Redis 有着更为复杂的数据结构并且提供对他们的原子性操作**，这是一个不同于其他数据库的进化路径。Redis 的数据类型都是基于基本数据结构 的同 时对程序员透明，无需进行额外的抽象。 
- **Redis 运行在内存中但是可以持久化到磁盘**，所以在对不同数据集进行高 速读写时需要权衡内存，因为数据量不能大于硬件内存。在内存数据库方 面的 另一个优点是，相比在磁盘上相同的复杂的数据结构，在内存中操作 起来非常 简单，这样 Redis 可以做很多内部复杂性很强的事情。同时，在 磁盘格式方面 他们是紧凑的以追加的方式产生的，因为他们并不需要进行 随机访问。

### 2、Redis 的数据类型？

 Redis 支持五种数据类型：string（字符串），hash（哈希），list（列 表），set（集合）及 zsetsorted set：有序集合)。 

我们实际项目中比较常用的是 string，hash 

如果你是 Redis 中高级用户，还 需要加上下面几种数据结构 HyperLogLog、Geo、Pub/Sub。 如果你说还玩过 Redis Module，像 BloomFilter，RedisSearch，Redis-ML， 面试官得眼睛就开始发亮了。

### 3、使用 Redis 有哪些好处？

- **速度快**，因为数据存在内存中，类似于 HashMap，HashMap 的优势就是查找和操作的时间复杂度都是 O（1）
- **支持丰富数据类型**，支持 string，list，set，Zset，hash 等 
- **支持事务**，操作都是原子性，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行 ，Redis 事务仅支持一致性和隔离性
- **丰富的特性**：可用于缓存，消息，按 key 设置过期时间，过期后将会自动删除

### Redis 通讯协议是什么？有什么特点？

**Redis 客户端和 Redis 服务器通信**时使用的是 **[RESP（Redis 序列化协议）通讯协议](https://juejin.cn/post/6844903959883153421)**，该协议是专门为 Redis 设计的，但是也可以用于其他客户端和服务器软件项目中。

RESP 的特点为：实现简单、快速解析、可读性好。

### 可以介绍一些 Redis 常用的安全设置吗？

常用的设置有：

**1. 端口设置**

只允许信任的客户端发送过来的请求，对其他所有请求都拒绝。如果存在暴露在外网的服务，那么需要使用防火墙阻止外部访问 Redis 端口。

**2. 身份验证**

使用 Redis 提供的身份验证功能，在 redis.conf 文件中进行配置生效，客户端可以发送（AUTH 密码）命令进行身份认证。

**3. 禁用特定的命令集**

可以考虑禁止一些容易产生安全问题的命令，预防被人恶意操作。

### 4、Redis 相比 Memcached 有哪些优势？ 

- Memcached 所有的值均是简单的字符串，redis 作为其替代者，支持更为丰富的数据类 
- Redis 的速度比 Memcached 快很 
- Redis 可以持久化其数据

### 5、Memcache 与 Redis 的区别都有哪些？ 

- 存储方式 Memecache 把数据全部存在内存之中，断电后会挂掉，数据不 能超过内存大小。 Redis 有部份存在硬盘上，这样能保证数据的持久性。 
- 数据支持类型 Memcache 对数据类型支持相对简单。 Redis 有复杂的数据类型。 
- 使用底层模型不同它们之间底层实现方式 以及与客户端之间通信的应用协议不一样。 Redis 直接自己构建了 VM 机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求。 

### 6、Redis 是单进程单线程的？ 

Redis 是单进程单线程的，**redis 利用队列技术将并发访问变为串行访问**，消除了传统数据库串行控制的开销。

### Redis 为什么设计成单线程的？

[参考](https://www.jianshu.com/p/bc6904abc330)

多线程处理会涉及到锁，并且多线程处理会涉及到线程切换而消耗 CPU。采用单线程，**避免了不必要的上下文切换和竞争条件**。其次 CPU 不是 Redis 的瓶颈，**Redis 的瓶颈最有可能是机器内存或者网络带宽**。

Redis 客户端对服务端的每次调用都经历了`发送命令，执行命令，返回结果`三个过程。其中执行命令阶段，由于 Redis 是单线程来处理命令的，**所有到达服务端的命令都不会立刻执行**，所有的命令都会进入一个[队列](https://www.jianshu.com/p/30f69aac6350)中，然后逐个执行，并且**多个客户端发送的命令的执行顺序是不确定的，但是可以确定的是不会有两条命令被同时执行**，不会产生并发问题，这就是 Redis 的单线程基本模型。

### Redis 是单线程的，如何提高多核 CPU 的利用率？

可以在同一个服务器部署多个 Redis 的实例，并把他们当作不同的服务器来使用，在某些时候，无论如何一个服务器是不够的，所以，如果你想使用多个 CPU，你可以考虑一下分片（shard）。

###  7、一个字符串类型的值能存储最大容量是多少？ 

答：512M 

### 8、Redis 的持久化机制是什么？各自的优缺点？

 Redis 提供两种持久化机制 RDB 和 AOF 机制： 

**RDB （Redis DataBase）持久化方式：**

是指用数据集快照的方式半持久化模式

**记录 Redis 数据库的所有键值对, 在某个时间点将数据写入一个临时文件，持久化结束后，用这个临时文件替换上次持久化的文件，达到数据恢复**。 

优点： 

- 只有一个文件 dump.rdb，方便持久化。 
- 容灾性好，一个文件可以保存到安全的磁盘。
- 性能最大化，fork 子进程来完成写操作，让主进程继续处理命令，所以 是 IO 最大化。使用单独子进程来进行持久化，主进程不会进行任何 IO  操 作，保证了 Redis 的高性能) 
- 相对于数据集大时，比 AOF 的启动效率更高。 

缺点：  **数据安全性低**。RDB 是间隔一段时间进行持久化，如果持久化之间 Redis 发生故障，会发生数据丢失。

所以这种方式更适合数据要求不严谨的时候。

**AOF（Append-only file）持久化方式**： 

是指**所有的写命令行记录以 Redis 命令请求协议的格式完全持久化存储保存为 aof 文件**。

优点：  

- 数据安全，aof 持久化可以配置 appendfsync 属性，有 always，每进行一次命令操作就记录到 aof 文件中一次。 
- 通过 append 模式写文件，即使中途服务器宕机，可以通过 redis-check-aof 工具解决数据一致性问题。 
- AOF 机制的 rewrite 模式。AOF 文件没被 rewrite 之前（文件过大时会对命令进行合并重写），可以删除其中的某些命令（比如误操作的 flushall）) 

缺点： 

- AOF 文件比 RDB 文件大，且恢复速度慢。
- 数据集大的时候，比 rdb 启动效率低。

### 9、Redis 常见性能问题和解决方案： 

- Master 最好不要写内存快照，如果 Master 写内存快照，save 命令调度 rdbSave 函数，会阻塞主线程的工作，当快照比较大时对性能影响是非常大的，会间断性暂停服务 
- 如果数据比较重要，某个 Slave 开启 AOF 备份数据，策略设置为每秒同步一 次
- 为了主从复制的速度和连接的稳定性，Master 和 Slave 最好在同一个局域网 
- 尽量避免在压力很大的主库上增加从 
- **主从复制不要用图状结构，用单向链表结构更为稳定**，即：Master <- Slave1<- Slave2 <- Slave3…这样的结构方便解决单点故障问题，实现 Slave  对 Master 的替换。如果 Master 挂了，可以立刻启用 Slave1 做 Master，其 他不变。

### 10、Redis 过期键的删除策略？ 

1. **定时删除**:在**设置键的过期时间的同时，创建一个定时器 timer**. 让定时器在键的过期时间来临时，立即执行对键的删除操作。
2. **惰性删除**:放任键过期不管，但是**每次从键空间中获取键时，都检查取得的键是否过期，如果过期的话，就删除该键**;如果没有过期，就返回该键。
3. **定期删除**:每隔一段时间程序就对数据库进行一次检查，删除里面的过期键。至于要删除多少过期键，以及要检查多少个数据库，则由算法决定。

### 11、Redis 的回收策略（淘汰策略）?

volatile-lru：从**已设置过期时间**的数据集（server.db[i].expires）中挑选**最近最少使用**的数据淘汰

volatile-ttl：从**已设置过期时间**的数据集（server.db[i].expires）中挑选**将要过期**的数据淘汰 

volatile-random：从**已设置过期时间**的数据集（server.db[i].expires）中**任意选择**数据淘汰 

allkeys-lru：从**数据集**（server.db[i].dict）中挑选**最近最少使用**的数据淘 汰 

allkeys-random：从**数据集**（server.db[i].dict）中**任意选择**数据淘汰

no-enviction（驱逐）：禁止驱逐数据 

注意这里的 6 种机制，volatile 和 allkeys 规定了是对已设置过期时间的数据集淘汰数据还是从全部数据集淘汰数据，后面的 lru、ttl 以及 random 是三种不同的淘汰策略，再加上一种 no-enviction 永不回收的策略。 

**使用策略规则：**  

- 如果数据呈现**幂律分布**，也就是**一部分数据访问频率高**，一部分数据访问频率低，则使用 allkeys-lr
- 如果数据呈现平等分布，也就是所有的数据访问频率都相同，则使用 allkeys-random

### 12、为什么 Redis 需要把所有数据放到内存中？ 

Redis **为了达到最快的读写速度将数据都读到内存中**，**并通过异步的方式将数据写入磁盘**。所以 Redis 具有快速和数据持久化的特征。如果不将数据放在内存中，磁盘 I/O 速度为严重影响 Redis 的性能。在内存越来越便宜的今 天， Redis 将会越来越受欢迎。如果设置了最大使用的内存，则数据已有记录数达到内存限值后不能继续插入新值。

### 13、Redis 的同步机制了解么？

 Redis 可以使用主从同步，从从同步。第一次同步时，主节点做一次 bgsave， 并同时将后续修改操作记录到内存 buffer，待完成后将 rdb 文件全量同步到复制节点，复制节点接受完成后将 rdb 镜像加载到内存。加载完成后，再通知主节点将期间修改的操作记录同步到复制节点进行重放就完成了同步过程。

### 14、Pipeline 有什么好处，为什么要用 Pipeline？ 

使用 pipeline（管道）的好处在于可以**将多次 I/O 往返的时间缩短为一次，但是要求管道中执行的指令间没有因果关系**。

用 pipeline 的原因在于可以实现请求/响应服务器的功能，当客户端尚未读取旧响应时，它也可以处理新的请求。如果客户端存在多个命令发送到服务器时，那么客户端无需等待服务端的每次响应才能执行下个命令，只需最后一步从服务端读取回复即可。

可以适用场景有：如果存在批量数据需要写入 Redis，并且这些数据**允许一定比例的写入失败**，那么可以使用 Pipeline，后期再对失败的数据进行补偿即可。

### 请说明一下 Redis 的批量命令与 Pipeline 有什么不同？

它们主要的不同有：

- 原子性不同；**批量命令操作需要保证原子性**的，Pipeline 执行是非原子性的；
- 支持的命令不同；批量命令操作是**一个命令对应多个 key**，Pipeline 支持**多个命令**的执行；
- 实现方式的不同；批量命令操作是由 Redis 服务端实现的，而 Pipeline 是需要服务端和客户端共同实现的。

### 15、是否使用过 Redis 集群，集群的原理是什么？

- Redis Sentinal 着眼于高可用，在 master 宕机时会自动将 slave 提升 为 master，继续提供服务。
- Redis Cluster 着眼于扩展性，在单个 Redis 内存不足时，使用 Cluster  进行分片存储。

### 16、Redis 集群方案什么情况下会导致整个集群不可用？ 

有 A，B，C 三个节点的集群, 在没有复制模型的情况下,如果节点 B 失败了， 那么整个集群就会以为缺少 5501-11000 这个范围的槽而不可用。

1. 当访问一个 Master 和 Slave 节点都挂了的槽的时候，会报槽无法获取。

2. 当**集群 Master 节点个数小于 3 个的时候，或者集群可用节点个数为偶数的时候**，基于 fail 的这种选举机制的自动主从切换过程可能会不能正常工作，一个是标记 fail 的过程，一个是选举新的 master 的过程，都有可能异常。

### 19、Redis 如何设置密码及验证密码？ 

设置密码：config set requirepass 123456 

授权密码：auth 123456

### 20、说说 Redis 哈希槽的概念？

 Redis 集群没有使用一致性 hash, 而是引入了哈希槽的概念，Redis 集群有 16384 个哈希槽，每个 key 通过 CRC16 校验后对 16384 取模来决定放置哪个槽，集群的每个节点负责一部分 hash 槽。 

### 21、Redis 集群的主从复制模型是怎样的？ 

为了使在部分节点失败或者大部分节点无法通信的情况下集群仍然可用， 所以集群使用了主从复制模型, 每个节点都会有 N-1 个复制品。 

### 22、Redis 集群会有写操作丢失吗？为什么？ 

Redis 集群中**有可能存在写操作丢失的问题**，但是丢失概率一般可以忽略不计。主要是 Redis 并没有一个机制保证数据一定写不丢失。在以下问题中可能出现键值丢失的问题：

**1. 超过内存的最大值，键值被清理**

Redis 中可以设置缓存的最大内存值，当内存中的数据总量大于设置的最大内存值，会导致 Redis 对部分数据进行清理，导致键值丢失的问题。

**2. 大量的 key 过期，被清理**

这种情况比较正常，只是因为键值设置的时间过期了，被自动清理了。

**3. Redis 主库服务器故障重启**

由于 Redis 的数据是缓存在内存中的，如果 Redis 主库服务器出现故障重启，会出现数据被清空的问题。这时可能**导致从库的数据同步被清空**。如果有使用数据持久化，那么故障重启后数据是可以自动恢复的。

**4. 网络问题**

可能出现网络故障，导致短时间内数据写入失败。

### 23、Redis 集群之间是如何复制的？ 

异步复制

### Redis 集群的主从复制模型是怎样的？

Redis 集群支持的主从复制，数据同步主要有两种方法：一种是全量同步，一种是增量同步。

**1. 全量同步**

刚开始搭建主从模式时，从机需要从主机上获取所有数据，这时就需要 Slave 将 Master 上所有的数据进行同步复制。复制的步骤为：

- 从服务器发送 SYNC 命令，链接主服务器；
- 主服务器收到 SYNC 命令后，进行存盘的操作，并继续收集后续的写命令，存储缓冲区；
- 存盘结束后，将对应的数据文件发送到 Slave 中，完成一次全量同步；
- 主服务数据发送完毕后，将进行增量的缓冲区数据同步；
- Slave 加载数据文件和缓冲区数据，开始接受命令请求，提供操作。

**2. 增量同步**

从节点完成了全量同步后，就可以正式的开启增量备份。当 Master 节点有写操作时，都会自动同步到 Slave 节点上。Master 节点每执行一个命令，都会同步向 Slave 服务器发送相同的写命令，当从服务器接收到命令，会同步执行。

### 24、Redis 集群最大节点个数是多少？ 

16384 个。 

### 25、Redis 集群如何选择数据库？

 Redis 集群目前无法做数据库选择，默认在 0 数据库。

### 怎么让相关键都分配到集群中的同一个节点

[参考](https://blog.csdn.net/weixin_38106322/article/details/108543596)

### 26、怎么测试 Redis 的连通性 

使用 ping 命令。 

### 27、怎么理解 Redis 事务？ 

- 事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。 
- 事务是一个原子操作：事务中的命令要么全部被执行，要么全部都不执行，不支持回滚。 

### 28、Redis 事务相关的命令有哪几个？ 

| 命令    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| MULTI   | 标记一个事务块的开始                                         |
| EXEC    | 执行所有事务块内的命令                                       |
| DISCARD | 取消事务，放弃执行事务块内的所有命令                         |
| WATCH   | 监视一个（或多个）key，如果在事务执行之前这个（或多个）key被其他命令所改动，那么事务将被打断 |
| UNWATCH | 取消 WATCH 命令对所有 keys 的监视                            |

### Redis 事务的注意点有哪些？

需要注意的点有：

- Redis 事务是**不支持回滚**的，不像 MySQL 的事务一样，要么都执行要么都不执行；
- Redis 服务端在执行事务的过程中，**不会被其他客户端发送来的命令请求打断**。直到事务命令全部执行完毕才会执行其他客户端的命令。

### Redis 为什么不支持回滚？

Redis 的事务不支持回滚，但是执行的命令有语法错误，Redis 会执行失败，这些问题可以从程序层面捕获并解决。但是如果出现其他问题，则依然会继续执行余下的命令。这样做的原因是因为回滚需要增加很多工作，而不支持回滚则可以保持简单、快速的特性。

### 29、Redis key 的过期时间和永久有效分别怎么设置？

 EXPIRE 和 PERSIST 命令。

### 30、Redis 如何做内存优化？ 

尽可能**使用散列表**（hashes），散列表（是说散列表里面存储的数少）使用的 内存非常小，所以你应该尽可能的将你的数据模型抽象到一个散列表里面。比 如你的 web 系统中有一个用户对象，不要为这个用户的名称，姓氏，邮箱，密码设置单独的 key,而是应该**把这个用户的所有信息存储到一张散列表里面**。

### 31、Redis 回收进程如何工作的？ 

一个客户端运行了新的命令，添加了新的数据。**Redis 检查内存使用情况， 如果大于 maxmemory 的限制, 则根据设定好的策略进行回收**。一个新的命令被执行，等等。所以我们**不断地穿越内存限制的边界，通过不断达到边界然后不断地回收回到边界以下**。如果一个命令的结果导致大量内存被使用（例如很大 的 集合的交集保存到一个新的键），不用多久内存限制就会被这个内存使用量超越。

### 32、都有哪些办法可以降低 Redis 的内存使用情况呢？

如果你使用的是 32 位的 Redis 实例，可以好好利用 Hash,list,sorted  set,set 等集合类型数据，因为通常情况下很多小的 Key-Value 可以用更紧凑的方式存放到一起。 

### 33、Redis 的内存用完了会发生什么？ 

如果达到设置的上限，**Redis 的写命令会返回错误信息**（但是读命令还可以正常返回。）或者你可以将 Redis 当缓存来使用配置淘汰机制，当 Redis 达到 内存上限时会冲刷掉旧的内容。 

### 34、一个 Redis 实例最多能存放多少的 keys？ List、Set、 Sorted Set 他们最多能存放多少元素？

 理论上 **Redis 可以处理多达 232 的 keys**，并且在实际中进行了测试，每个实例至少存放了 2 亿 5 千万的 keys。我们正在测试一些较大的值。任何 list、 set、和 sorted set 都可以放 232 个元素。换句话说，Redis 的存储极限是系统中的可用内存值。

### 36、Redis 最适合的场景？ 

**会话缓存（Session Cache）** 

最常用的一种使用 Redis 的情景是会话缓存（session cache）。用 Redis 缓 存会话比其他存储（如 Memcached）的优势在于：Redis 提供持久化。当维护 一个不是严格要求一致性的缓存时，如果用户的购物车信息全部丢失，大部分人都会不高兴的，现在，他们还会这样吗？ 幸运的是，随着 Redis 这些年的改进，很容易找到怎么恰当的使用 Redis 来缓存会话的文档。甚至广为人知的商业平台 Magento 也提供 Redis 的插件。 

**全页缓存（FPC）** 

除基本的会话 token 之外，Redis 还提供很简便的 FPC 平台。回到一致性问题，即使重启了 Redis 实例，因为有磁盘的持久化，用户也不会看到页面加载速度的下降，这是一个极大改进，类似 PHP 本地 FPC。 再次以 Magento 为例，Magento 提供一个插件来使用 Redis 作为全页缓存后端。 此外，对 WordPress 的用户来说，Pantheon 有一个非常好的插件 wp-redis，这个插件能帮助你以最快速度加载你曾浏览过的页面。

**队列**

Redis 在内存存储引擎领域的一大优点是提供 list 和 set 操作，这使得 Redis 能作为一个很好的消息队列平台来使用。Redis 作为队列使用的操作， 就类似于本地程序语言（如 Python）对 list 的 push/pop 操作。 如果你快 速的在 Google 中搜索“Redis queues”，你马上就能找到大量的开源项目， 这些项目的目的就是利用 Redis 创建非常好的后端工具，以满足各种队列需 求。例如，Celery 有一个后台就是使用 Redis 作为 broker，你可以从这里去 查看。 

**排行榜/计数器** 

Redis **在内存中对数字进行递增或递减的操作实现的非常好**。集合（Set）和有序集合（Sorted Set）也使得我们在执行这些操作的时候变的非常简单，Redis  只是正好提供了这两种数据结构。所以，我们要从排序集合中获取到排名最靠 前的 10 个用户–我们称之为“user_scores”，我们只需要像下面一样执行即可： 当然，这是假定你是根据你用户的分数做递增的排序。如果你想返回用户及用户的分数，你需要这样执行： ZRANGE user_scores 0 10 WITHSCORES  Agora Games 就是一个很好的例子，用 Ruby 实现的，它的排行榜就是使用 Redis 来存储数据的，你可以在这里看到。 

**发布/订阅** 

最后（但肯定不是最不重要的）是 Redis 的发布/订阅功能。发布/订阅的使用 场景确实非常多。我已看见人们在社交网络连接中使用，还可作为基于发布/订 阅的脚本触发器，甚至用 Redis 的发布/订阅功能来建立聊天系统！

### 37、假如 Redis 里面有 1 亿个 key，其中有 10w 个 key 是以某 个固定的已知的前缀开头的，如果将它们全部找出来？ 

**使用 keys 指令可以扫出指定模式的 key 列表**。 对方接着追问：如果这个 Redis 正在给线上的业务提供服务，那使用 keys 指令会有什么问题？ 

这个时候你要回答 Redis 关键的一个特性：**Redis 的单线程的**。**keys 指令会导致线程阻塞一段时间，线上服务会停顿**，直到指令执行完毕，服务才能恢复。**这个时候可以使用 scan 指令**，scan 指令可以**无阻塞的提取出指定模式的key 列表**，但是会有一定的重复概率，在客户端做一次去重就可以了，但是整 体所花费的时间会比直接用 keys 指令长

### 38、如果有大量的 key 需要设置同一时间过期，一般需要注意什 么？ 

如果大量的 key 过期时间设置的过于集中，**到过期的那个时间点，Redis 可能 会出现短暂的卡顿现象**。一般需要在时间上**加一个随机值**，使得过期时间分散 一些。

### 39、使用过 Redis 做异步队列么，你是怎么用的？ 

一般**使用 list 结构作为队列**，rpush 生产消息，lpop 消费消息。当 lpop 没有消息的时候，要**适当 sleep 一会再重试**。 

**如果对方追问可不可以不 用 sleep 呢？**  

list 还有个指令叫 **blpop**，在没有消息的时候，它会**阻塞住直到消息到来**。 

**如果对方追问能不能生产一次消费多次呢？** 

**使用 pub/sub 主题订阅者模式**，可以实现 1:N 的消息队列。 

**如果对方追问 pub/sub 有什么缺点？**  

**在消费者下线的情况下，生产的消息会丢失**，得使用专业的消息队列如 RabbitMQ 等。

**如果对方追问 Redis 如何实现延时队列？**  

我估计现在你很想把面试官一棒打死如果你手上有一根棒球棍的话，怎么问的 这么详细。但是你很克制，然后神态自若的回答道：**使用 sortedset，拿时间戳作为 score，消息内容作为 key 调用 zadd 来生产消息，消费者用 zrangebyscore 指令获取 N 秒之前的数据轮询进行处理**。

### 40、使用过 Redis 分布式锁么，它是什么回事 

锁的作用是保证多个进程或线程在并发操作操作共享资源时资源的正确性。在分布式应用中，一个服务需要部署多个实例，对于操作分布式环境下的共享资源，就需要使用分布式锁来保证操作的正确性。

**分布式锁应该具有互斥、可重入、锁超时，高可用等特点**。其中前面几个特点和本地锁具体的特点相同，高可用是分布式锁需要具备的重要的特点。

**先拿 setnx 来争抢锁，抢到之后，再用 expire 给锁加一个过期时间防止锁忘记了释放**。 这时候对方会告诉你说你回答得不错，然后接着问如果在 setnx 之后执行 expire 之前进程意外 crash 或者要重启维护了，那会怎么样？这时候你要给予惊讶的反馈：唉，是喔，这个锁就永远得不到释放了。紧接着你需要抓一抓 自己得脑袋，故作思考片刻，好像接下来的结果是你主动思考出来的，然后回 答：我记得 set 指令有非常复杂的参数，这个应该是可以**同时把 setnx 和 expire 合成一条指令来用的**！对方这时会显露笑容，心里开始默念：摁，这小子还不错。

### 缓存击穿

缓存击穿是指**访问某个非常热的数据，缓存不存在，导致大量的请求发送到了数据库**，这会导致数据库压力陡增，缓存击穿经常发生在热点数据过期失效时，如下图所示：

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7188ec79c6174037990e198d17c56ea4~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

既然缓存击穿经常发生在热点数据过期失效的时候，那么我们**不让缓存失效**不就好了，每次查询缓存的时候不要使用Exists来判断key是否存在，而是使用Expire给缓存续期，通过Expire返回结果判断key是否存在，既然是热点数据通过不断地续期也就不会过期了

还有一种简单有效的方法就是通过singleflight来控制，singleflight的原理是当同时有很多请求同时到来时，最终只有一个请求会最终访问到资源，其他请求都会等待结果然后返回。获取商品详情使用singleflight进行保护示例如下：

```go
func (l *ProductLogic) Product(in *product.ProductItemRequest) (*product.ProductItem, error) {
  v, err, _ := l.svcCtx.SingleGroup.Do(fmt.Sprintf("product:%d", in.ProductId), func() (interface{}, error) {
    return l.svcCtx.ProductModel.FindOne(l.ctx, in.ProductId)
  })
  if err != nil {
    return nil, err
  }
  p := v.(*model.Product)
  return &product.ProductItem{
    ProductId: p.Id,
    Name:      p.Name,
  }, nil
}
```

### 缓存穿透

缓存穿透是指要访问的数据**既不在缓存中，也不在数据库中，导致请求在访问缓存时，发生缓存缺失，再去访问数据库时，发现数据库中也没有要访问的数据。此时也就没办法从数据库中读出数据再写入缓存来服务后续的请求，类似的请求如果多的话就会给缓存和数据库带来巨大的压力**。

针对缓存穿透问题，**解决办法其实很简单，就是缓存一个空值，避免每次都透传到数据库，缓存的时间可以设置短一点**，比如1分钟，其实上文已经有提到了，当我们访问不存在的数据的时候，go-zero框架会帮我们自动加上空缓存，比如我们访问id为999的商品，该商品在数据库中是不存在的。

```sh
grpcurl -plaintext -d '{"product_id": 999}' 127.0.0.1:8081 product.Product.Product
```

此时查看缓存，已经帮我添加好了空缓存

```shell
127.0.0.1:6379> get cache:product:product:id:999
"*"
```

### 缓存雪崩

缓存雪崩时指**大量的的应用请求无法在Redis缓存中进行处理，紧接着应用将大量的请求发送到数据库，导致数据库被打挂**，好惨呐！！缓存雪崩一般是由两个原因导致的，应对方案也不太一样。

第一个原因是：缓存中**有大量的数据同时过期**，导致大量的请求无法得到正常处理。

针对大量数据同时失效带来的缓存雪崩问题，一般的解决方案是要避免大量的数据设置相同的过期时间，如果业务上的确有要求数据要同时失效，那么可以**在过期时间上加一个较小的随机数**，这样不同的数据过期时间不同，但差别也不大，避免大量数据同时过期，也基本能满足业务的需求。

第二个原因是：**Redis出现了宕机，没办法正常响应请求了**，这就会导致大量请求直接打到数据库，从而发生雪崩

针对这类原因一般我们需要**让我们的数据库支持熔断**，让数据库压力比较大的时候就触发熔断，丢弃掉部分请求，当然熔断是对业务有损的。

在go-zero的数据库客户端是支持熔断的，如下在ExecCtx方法中使用熔断进行保护

```go
func (db *commonSqlConn) ExecCtx(ctx context.Context, q string, args ...interface{}) (
  result sql.Result, err error) {
  ctx, span := startSpan(ctx, "Exec")
  defer func() {
    endSpan(span, err)
  }()

  err = db.brk.DoWithAcceptable(func() error {
    var conn *sql.DB
    conn, err = db.connProv()
    if err != nil {
      db.onError(err)
      return err
    }

    result, err = exec(ctx, conn, q, args...)
    return err
  }, db.acceptable)

  return
}
```

### 如何解决缓存与数据库不一致

[参考](https://note.dolyw.com/cache/00-DataBaseConsistency.html#%E6%9B%B4%E6%96%B0%E7%BC%93%E5%AD%98-vs-%E5%88%A0%E9%99%A4%E7%BC%93%E5%AD%98)

#### 缓存和数据库的数据不一致是如何发生的？

首先，我们得清楚“数据的一致性”具体是啥意思。其实，这里的“一致性”包含了两种情况：

- 缓存中有数据，那么，**缓存的数据值需要和数据库中的值相同**；
- **缓存中本身没有数据，那么，数据库中的值必须是最新值**。

不符合这两种情况的，就属于缓存和数据库的数据不一致问题了。不过，当缓存的读写模式不同时，缓存数据不一致的发生情况不一样，我们的应对方法也会有所不同，所以，我们先按照缓存读写模式，来分别了解下不同模式下的缓存不一致情况。**根据是否接收写请求，我们可以把缓存分成读写缓存和只读缓存**。

对于**读写缓存**来说，如果要对数据进行增删改，就需要在缓存中进行，同时还要根据采取的写回策略，决定是否同步写回到数据库中。

- **同步直写策略**：写缓存时，也同步写数据库，缓存和数据库中的数据一致；
- **异步写回策略**：写缓存时不同步写数据库，等到数据从缓存中淘汰时，再写回数据库。使用这种策略时，如果数据还没有写回数据库，缓存就发生了故障，那么，此时，数据库就没有最新的数据了。

所以**，对于读写缓存来说，要想保证缓存和数据库中的数据一致，就要采用同步直写策略**。不过，需要注意的是，如果采用这种策略，就需要同时更新缓存和数据库。所以，我们要**在业务应用中使用事务机制，来保证缓存和数据库的更新具有原子性**，也就是说，两者要不一起更新，要不都不更新，返回错误信息，进行重试。否则，我们就无法实现同步直写。

当然，在有些场景下，我们对数据一致性的要求可能不是那么高，比如说缓存的是电商商品的非关键属性或者短视频的创建或修改时间等，那么，我们可以使用异步写回策略。

#### 先删缓存再更新数据库

假设线程A删除缓存后，还没来得及更新数据库，这时候线程B开始读数据，线程B发现缓存缺失就只能去读数据库，等到线程B从数据库中读取完数据回塞缓存后，线程A才开始更新数据库，此时，缓存中的数据是旧值，而数据库中是最新值，两者已经不一致了。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c73e097fe8ba494f80a409b997cfa5b9~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

这种场景的解决方案是在线程A更新完数据库的值后，可以让它sleep一小段时间，**再进行一次缓存删除操作**，之所以要加上sleep的一段时间，就是为了让线程B能够先从数据库读取出数据然后再把缓存miss的数据回塞到缓存，然后线程A再进行删除。所以线程A的sleep时间就需要大于线程B读取数据再写入缓存的时间。这个时间是多少呢？这个是需要我们在业务中加入打点监控来统计的，根据这个统计值来估算该时间。这样一来，其他线程读取数据时，会发现缓存缺失，就会从数据库中读取最新的值。我们把这种模型叫做 "延时双删"。

#### 先更新数据库再删除缓存

如果线程A更新了数据库中的值，但还没来得及删除缓存中的值，线程B这时候开始读取数据，此时，线程B查询缓存时，命中了缓存，就会直接使用缓存中的值，该值为旧值。不过在这种场景下，如果并发请求量不高的话，其实基本上不会有线程读到旧值，而且线程A更新完数据库后，删除缓存是非常快的操作，所以，这种情况总体对业务影响较小。一般在生产环境中，也推荐大家采用该模式。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d92a3154cd064337b77160e5827ff269~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 重试机制

可以**把要删除的缓存值或者要更新的数据库的值放到消息队列**中，当应用没能够成功地删除缓存或者是更新数据库的值的时候，可以从消息队列中消费这些值，这里消费消息队列的服务叫job，然后再次进行删除或者更新，起到一个兜底补偿的作用，以此来保证最终的一致性。

如果能够成功地删除或更新，就需要把这些值从消息队列中去除，以免重复操作，此时，我们也可以保证数据库和缓存数据的一致了，否则的话，我们还需要再次进行重试，如果重试超过一定次数还是失败，这时候一般都需要记录错误日志或者发送告警通知。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d6f3523777344447b811bcf8a60b7af3~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 并发读写

首先第一步线程A读取缓存，这时候缓存没有命中，由于使用的是cache aside这种模式，所以接下来第二步线程A会去读数据库，这个时候线程B更新数据库，更新完数据库后通过set cache更新了缓存，最后第五步线程A把从数据库读到的值通过set cache也更新了缓存，但是这时候线程A中的数据已经是脏数据了，由于第四步和第五步都是设置缓存，导致写入的值相互覆盖，并且操作的顺序具有不确定性，从而导致了缓存不一致情况的发生。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ba86a5e43ca4d609df5f28b83f2037e~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

怎么解决这个问题呢？其实非常地简单，我们只需要**把第五步的set cache操作替换成add cache即可，add cache即setnx操作，只有缓存不存在的时候才会成功写入，相当于加了优先级，即更新数据库后的更新缓存优先级更高，而读数据库后回塞缓存的优先级较低，从而保证写操作的最新数据不会被读操作的回塞数据覆盖。**

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4642443d785441aaa784506454f3bd9e~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

#### 解决方案

那怎么解决呢，可以采用**延时双删策略(缓存双淘汰法)**，可以将前面所造成的缓存脏数据，再次删除

> 1. 先删除(淘汰)缓存
> 2. 再写数据库（这两步和原来一样）
> 3. 休眠1秒，再次删除(淘汰)缓存

或者是

> 1. 先写数据库
> 2. 再删除(淘汰)缓存（这两步和原来一样）
> 3. 休眠1秒，再次删除(淘汰)缓存

这个1秒应该看你的业务场景，应该自行评估自己的项目的读数据业务逻辑的耗时，然后写数据的休眠时间则在读数据业务逻辑的耗时基础上，加几百ms即可，这么做确保读请求结束，写请求可以删除读请求造成的缓存脏数据

**如果你用了MySql的读写分离架构怎么办？**，例如

> 1. 请求A进行写操作，删除缓存
> 2. 请求A将数据写入数据库了，(或者是先更新数据库，后删除缓存)
> 3. 请求B查询缓存发现，缓存没有值
> 4. 请求B去从库查询，这时，还没有完成主从同步，因此查询到的是旧值
> 5. 请求B将旧值写入缓存
> 6. 数据库完成主从同步，从库变为新值

这种情景，就是数据不一致的原因，还是采用**延时双删策略(缓存双淘汰法)**，只是，休眠时间修改为在主从同步的延时时间基础上，加几百ms

**并且为了性能更快，可以把第二次删除缓存可以做成异步的，这样不会阻塞请求了，如果再严谨点，防止第二次删除缓存失败，这个异步删除缓存可以加上重试机制，失败一直重试，直到成功**

这里给出两种重试机制参考

**方案一**

> 1. 更新数据库数据
> 2. 缓存因为种种问题删除失败
> 3. 将需要删除的key发送至消息队列
> 4. 自己消费消息，获得需要删除的key
> 5. 继续重试删除操作，直到成功

![图片](https://cdn.dolyw.com/2019/11/20191105003.png)

然而，该方案有一个缺点，对业务线代码造成大量的侵入，于是有了方案二，**启动一个订阅程序去订阅数据库的Binlog，获得需要操作的数据**。在应用程序中，另起一段程序，获得这个订阅程序传来的信息，进行删除缓存操作

**方案二**

> 1. 更新数据库数据
> 2. 数据库会将操作信息写入binlog日志当中
> 3. 订阅程序提取出所需要的数据以及key
> 4. 另起一段非业务代码，获得该信息
> 5. 尝试删除缓存操作，发现删除失败
> 6. 将这些信息发送至消息队列
> 7. 重新从消息队列中获得该数据，重试操作

![图片](https://cdn.dolyw.com/2019/11/20191105004.png)

上述的订阅Binlog程序在MySql中有现成的中间件叫Canal，可以完成订阅Binlog日志的功能，另外，重试机制，这里采用的是消息队列的方式。如果对一致性要求不是很高，直接在程序中另起一个线程，每隔一段时间去重试即可，这些大家可以灵活自由发挥，只是提供一个思路

#### 最后的总结

大部分应该使用的都是第三种或第四种方式，如果都是采用**延时双删策略(缓存双淘汰法)**，可能区别不会很大，不过**第四种方式出现脏数据概率是更小点**，更多的话还是要结合自身业务场景使用，自己灵活变通

### 缓存使用技巧

下面介绍一些缓存使用的小技巧

- **key 的命名要尽量易读，即见名知意**，在易读的前提下长度要尽可能的小，以减少资源的占用，对于value来说可以用int就尽量不要用string，对于小于N的value，redis内部有shared_object缓存。
- 在redis使用hash的情况下进行key的拆分，同一个hash key会落到同一个redis节点，hash过大的情况下会导致内存以及请求分布的不均匀，考虑对hash进行拆分为小的hash，使得节点内存均匀避免单节点请求热点。
- 为了避免不存在的数据请求，导致每次请求都缓存miss直接打到数据库中，进行空缓存的设置。
- 缓存中需要存对象的时候，序列化尽量使用protobuf，尽可能减少数据大小。
- 新增数据的时候要保证缓存务必存在的情况下再去操作新增，使用Expire来判断缓存是否存在。
- 对于存储每日登录场景的需求，可以使用BITSET，为了避免单个BITSET过大或者热点，可以进行sharding。
- 在使用sorted set的时候，避免使用zrange或者zrevrange返回过大的集合，复杂度较高。
- 在进行缓存操作的时候尽量使用PIPELINE，但也要注意避免集合过大。
- 避免超大的value。
- 缓存尽量要设置过期时间。
- 慎用全量操作命令，比如Hash类型的HGETALL、Set类型的SMEMBERS等，这些操作会对Hash和Set的底层数据结构进行全量扫描，如果数据量较多的话，会阻塞Redis主线程。
- 获取集合类型的全量数据可以使用SSCAN、HSCAN等命令分批返回集合中的数据，减少对主线程的阻塞。
- 慎用MONITOR命令，MONITOR命令会把监控到的内容持续写入输出缓冲区，如果线上命令操作很多，输出缓冲区很快就会溢出，会对Redis性能造成影响。
- 生产环境禁用KEYS、FLUSHALL、FLUSHDB等命令。

###  Redis 的 String 类型使用 SDS 方式实现的好处？

![](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/68747470733a2f2f696d672d626c6f672e6373646e696d672e636e2f696d675f636f6e766572742f35313637333863343035386364663931303965343061373831326566343233392e706e67)

结构中的每个成员变量分别介绍下：

- **len，记录了字符串长度**。这样获取字符串长度的时候，只需要返回这个成员变量值就行，时间复杂度只需要 O（1）。
- **alloc，分配给字符数组的空间长度**。这样在修改字符串的时候，可以通过 `alloc - len` 计算出剩余的空间大小，可以用来判断空间是否满足修改需求，如果不满足的话，就会自动将 SDS 的空间扩展至执行修改所需的大小，然后才执行实际的修改操作，所以使用 SDS 既不需要手动修改 SDS 的空间大小，也不会出现前面所说的缓冲区溢出的问题。
- **flags，用来表示不同类型的 SDS**。一共设计了 5 种类型，分别是 sdshdr5、sdshdr8、sdshdr16、sdshdr32 和 sdshdr64，后面在说明区别之处。
- **buf[]，字符数组，用来保存实际数据**。不仅可以保存字符串，也可以保存二进制数据。

SDS 实现方式相对 C 语言的 String 的好处有：

- **避免缓冲区溢出**；对字符修改时，可以根据 len 属性检查空间是否满足要求。
- **常数复杂度获取字符串长度**；获取字符串的长度直接读取 len 属性就可以获取。而 C 语言中，因为字符串是简单的字符数组，求长度时内部其实是直接顺序遍历数组内容，找到 '\0' 对应的字符，计算出字符串的长度。复杂度即 O(N)。
- **减少内存分配次数**；通过结构中的 len 和 free 两个属性，更好的协助空间预分配以及惰性空间释放。
- **二进制安全**；SDS不是以空字符串来判断是否结束，而是以 len 属性来判断字符串是否结束。而在 C 语言中，字符串要求除了末尾之外不能出现空字符，否则会被程序认为是字符串的结尾。这就使得 C 字符串只能存储文本数据，而不能保存图像，音频等二进制数据。
- **兼容 C 字符串函数**；可以重用 C 语言库的 的一部分函数。

以上几点好处可以概括如下：

| C 字符串                                       | SDS                                            |
| :--------------------------------------------- | :--------------------------------------------- |
| 获取字符串长度的复杂度为 O(N)                  | 获取字符串长度的复杂度为 O(1)                  |
| API 是不安全的，可能会造成缓冲区溢出           | API 是安全的，不会造成缓冲区溢出               |
| 修改字符串长度 N 次必然需要执行 N 次内存重分配 | 修改字符串长度 N 次最多需要执行 N 次内存重分配 |
| 只能保存文本数据                               | 可以保存文本或者二进制数据                     |
| 可以使用所有库中的函数                         | 可以使用一部分库的函数                         |

### 请介绍几个可能导致 Redis 阻塞的原因

Redis 产生阻塞的原因主要有内部和外部两个原因导致：

**1. 内部原因**

- 如果 Redis 主机的 CPU 负载过高，也会导致系统崩溃；
- 数据持久化占用资源过多；
- 对 Redis 的 API 或指令使用不合理，导致 Redis 出现问题。

**2. 外部原因**

外部原因主要是服务器的原因，例如服务器的 CPU 线程在切换过程中竞争过大，内存出现问题、网络问题等。

### 如何处理 Redis 集群中 big key 和 hot key？

对于 big key **先分析业务存在大键值是否合理**，如果不合理我们可以把它们拆分为多个小的存储。或者看是否可以使用别的存储介质存储这种 big key 解决占用内存空间大的问题。

对于 hot key 我们可以**在其他机器上复制这个 key 的备份**，让请求进来的时候，去随机的到各台机器上访问缓存。所以剩下的问题就是如何做到让请求平均的分布到各台机器上。

### 怎么去发现 Redis 阻塞异常情况？

可以从以下两方面准备：

**1. 使用 Redis 自身监控系统**

使用 Redis 自身监控系统，可以对 CPU、内存、磁盘、命令耗时等阻塞问题进行监控，当监控系统发现各个监控指标存在异常的时候，发送报警。

**2. 使用应用服务监控**

当 Redis 存在阻塞时，应用响应时间就会延长，应用可以感知发现问题，并发送报警给管理人员。

### 介绍一下分布式锁实现需要注意的事项？

分布式锁实现需要保证以下几点：

- **互斥性**：任意时刻，只有一个资源能够获取到锁；
- **容灾性**：在未成功释放锁的的情况下，一定时限内能够恢复锁的正常功能；
- **统一性**：加锁和解锁保证同一资源来进行操作。

### 如果 AOF 文件的数据出现异常，Redis 服务怎么处理？

如果 AOF 文件数据出现异常，为了保证数据的一致性，Redis 服务器**会拒绝加载 AOF 文件**。可以尝试使用 redis-check-aof -fix 命令修复。

### Redis 的慢查询修复经验有哪些？怎么修复的？

可以对大对象数据进行拆分，防止执行一次命令操作过多的数据。也可以将一些算法复杂度高的命令或执行效率低的命令，禁用或者替换成高的指令。

### Redis 慢查询是什么？通过什么配置？

慢查询是指系统执行命令之后，当计算系统执行的指令时间超过设置的阀值，该命令就会被记录到慢查询中，该命令叫做慢指令。

Redis 慢查询的参数配置有：

**1. slowlog-log-slower-than**

该参数的作用为设置慢查询的阈值，当命令执行时间超过这个阈值就认为是慢查询。单位为微妙，默认为 10000。

可以根据自己线上的并发量进行调整这个值。如果存在高流量的场景，那么建议设置这个值为 1 毫秒，因为每个命令执行的时间如果超过 1 毫秒，那么 Redis 的每秒操作数最多只能到 1000。

**2. slowlog-max-len**

该参数的作用为设置慢查询日志列表的最大长度，当慢查询日志列表处于最大长度时，最早插入的一个命令将会被从列表中移除。

### Redis 如何做内存优化？

内存优化可以从以下几点进行：

- Redis 对所有数据都进行 redisObject 封装；
- 对键、值对象的长度进行缩减，简化键名，提高内存使用效率；
- 减少 key 的数量，尽量使用哈希数据类型；
- 共享对象池，并设置空间极限值。

### 使用 Redis，设计一下在交易网站首页展示当天最热门售卖商品的前五十名商品列表？

我们从题目中分析知道，首页的热门售卖商品知道，访问的数据是热数据，且访问量很大，还需要对当天的热门售卖商品进行排行。

可以**使用 zset 数据类型对热数据进行缓存**，zset 可以对数据进行排行，把 key 作为商品的 ID，score 作为商品当天销售的数量，可以根据 score对商品排行。

### 怎么提高缓存命中率？

主要常用的手段有：

- **提前加载数据**到缓存中；
- 增加缓存的存储空间，提高缓存的数据；
- 调整缓存的存储数据类型；
- 提升缓存的更新频率。

### 缓存命中率表示什么？

缓存命中率表示从缓存中读取数据时可以获取到数据的次数，命中率越高代表缓存的使用效率越高，应用的性能越好。

它的公式为：

> 缓存命中率 = 缓存中获取数据次数/获取数据总次数

- 缓存命中的意思是：从缓存中获取到需要的数据的次数；
- 缓存不命中的意思是：从缓存中无法获取所需数据，需要再次查询数据库或者其他数据存储载体的次数。