# Redis

[如何学习Redis](http://kaito-kidd.com/2020/09/09/how-i-learned-redis/)

[Redis 学习教程](https://www.bookstack.cn/read/redis-tutorial/0.md)

[《Redis设计与实现》](https://github.com/JiangRRRen/Redis-study)

## 一、如何快速上手Redis

- 学会基础数据类型的用法；
- 掌握扩展数据类型的用法；
- 积累一些Redis用作缓存的方法以及典型问题的解决方案。

在刚接触Redis时，第一步就是要学习它的基础数据结构，也就是`String`、`List`、`Hash`、`Set`、`Sorted Set`。毕竟，Redis之所以这么受欢迎，跟它丰富的数据类型是分不开的，它的数据都存储在内存中，访问速度极快，而且非常贴合我们常见的业务场景。我举几个例子：

- 如果你只需要存储简单的键值对，或者是对数字进行递增递减操作，就可以使用String存储；
- 如果需要一个简单的分布式队列服务，List就可以满足你的需求；
- 如果除了需要存储键值数据，还想单独对某个字段进行操作，使用Hash就非常方便；
- 如果想得到一个不重复的集合，就可以使用Set，而且它还可以做并集、差集和交集运算；
- 如果想实现一个带权重的评论、排行榜列表，那么，Sorted Set就能满足你。

## 二、什么是Redis

### 基本概念

redis是一个开源的、使用C语言编写的、支持网络交互的、可基于内存也可持久化的Key-Value数据库（非关系性数据库）。

### redis的优势

1. 速度快，因为数据存在内存中，类似于HashMap，HashMap的优势就是查找和操作的时间复杂度都是O(1)
2. 支持丰富数据类型，支持string，list，set，sorted set，hash
3. 支持事务，**操作都是原子性**，所谓的原子性就是对数据的更改要么全部执行，要么全部不执行
4. 丰富的特性：可用于缓存，消息，按key设置过期时间，过期后将会自动删除

### redis的应用场景

- 缓存

  (1) 对于一些要返回给前端数据的缓存，当有大量数据库sql操作时候，为了避免每次接口请求都要去查询数据库，可以把一些数据缓存到redis中，这样是直接从内存中获取数据，速度回增快很多。

  (2) web端用户，用于登陆缓存session数据，登陆的一些信息存到session中，缓存到redis中

- 队列

  redis中提供了list接口，这个list提供了lpush和rpop，这两个方法具有原子性，可以插入队列元素和弹出队列元素。

- 数据存储

  redis是非关系型数据库，可以把redis直接用于数据存储，提供了增删改查等操作，因为redis有良好的硬盘持久化机制，redis数据就可以定期持久化到硬盘中，保证了redis数据的完整性和安全性。

- redis锁实现防刷机制

  redis锁可以处理并发问题,redis数据类型中有一个set类型，set类型在存储数据的时候是无序的，而且每个值是不一样的，不能重复，这样就可以快速的查找元素中某个值是否存在，精确的进行增加删除操作。

### redis安装与启动

[windows](https://www.redis.com.cn/redis-installation.html)

[Linux](https://www.cnblogs.com/hunanzp/p/12304622.html)

[Docker](https://cloud.tencent.com/developer/article/1670205)

## 三、基本数据类型

redis是一种高级的key-value非关系型数据库。Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。

| redis类型  |   含义   |
| :--------- | :------: |
| String     |  字符串  |
| Hash       |   哈希   |
| List       |   列表   |
| Set        |   集合   |
| Sorted set | 有序集合 |

### 字符串（string）

 string是redis最基本的类型，一个key对应一个value。

string类型是二进制安全的。意思是redis的string可以包含任何数据。比如jpg图片或者序列化的对象

string类型是Redis最基本的数据类型，一个键最大能存储512MB。

```
redis 127.0.0.1:6379> SET name "itcast"
OK
redis 127.0.0.1:6379> GET name"itcast"
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
redis 127.0.0.1:6379> exists name(integer) 0
```

在上面的例子中，SET 和 GET 是 Redis STRING命令，`name` 和 `itcast` 是存储在 Redis 的键和字符串值。

**命令**



### 字符串列表（list）

 list类型是一个有序的列表，有序表示的是从左到右还是从右到左，而且数据内容是可以重复的。 代码实际操作过程：

```
[root@VM_160_197_centos /]# redis-cli
127.0.0.1:6379> lpush list1 12
(integer) 1
127.0.0.1:6379> lpush list1 13
(integer) 2
127.0.0.1:6379> lpush list1 12
(integer) 3
127.0.0.1:6379> rpop list1
"12"
127.0.0.1:6379> lpush list2 12
(integer) 1
127.0.0.1:6379> lpush list2 13
(integer) 2
127.0.0.1:6379> lpush list2 12
(integer) 3
127.0.0.1:6379> llen list2
(integer) 3
127.0.0.1:6379>
```

列表最多可存储 232 - 1 元素 (4294967295, 每个列表可存储40多亿)。

### 字符串集合（set）

set类型中提供了无序的方式来存储多个不同的元素，set类型中每个元素的值都不一样，用户可以快速对元素中的值添加删除，检查某些值是否存在，**重复的元素是无法继续插入集合的**。 代码实际操作过程：

```
127.0.0.1:6379> sadd set1 12
(integer) 1
127.0.0.1:6379> sadd set1 12
(integer) 0
127.0.0.1:6379> scard set1
(integer) 1
127.0.0.1:6379> sadd set1 13
(integer) 1
127.0.0.1:6379> scard set1
(integer) 2
127.0.0.1:6379> sadd set1 13
(integer) 0
127.0.0.1:6379> sismember set1 13  //查看13是否在集合中
(integer) 1
127.0.0.1:6379> srem set1 13    //从集合中删除13
```

### 哈希（hash）

hash类型也叫散列类型，存储的时候存的是键值对。查询条数的时候只要是健不一样，就是不同的条数，尽管值是相同的。

代码实际操作过程：

Redis hash 是一个键值对集合。

Redis hash是一个string类型的`field`和`value`的映射表，hash特别适合用于存储对象。

```
[root@VM_160_197_centos /]# redis-cli
127.0.0.1:6379> hset hash1 key1 12
(integer) 1
127.0.0.1:6379> hget hash1 key1
"12"
127.0.0.1:6379> hset hash1 key2 13
(integer) 1
127.0.0.1:6379> hset hash1 key3 13
(integer) 1
127.0.0.1:6379> hlen hash1//查询条数的时候只要是健不一样，就是不同的条数，尽管值是相同的。
(integer) 3
127.0.0.1:6379> hset hsah1 key3 14
(integer) 1
127.0.0.1:6379> hset hash1 key3 14
(integer) 0
127.0.0.1:6379> hget hash1 key3
"14"
127.0.0.1:6379> hmget hash1 key1 key2  //同时获取key1和key2的值
1) "12"
2) "13"
```

### 有序字符串集合（sort set）

Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个**double**类型的分数(权重)。redis正是通过**分数来为集合中的成员(权重)进行从小到大的排序**。

sore set也叫有序分数集，可以把它看作一个排行榜，每一个同学都有自己的分数，且排行榜中还有一个排名的属性，排行属性从0，根据分数不断变大，排行也不断变大。 ，这个类型有点复杂，上一张图吧。

![image](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/16b12177f4b5517a%7Etplv-t2oaga2asx-watermark.awebp)image

**sort set特性**

1）sore set中的值是全局唯一的。

一个值设置了之后，再次设置不会增加，只会覆盖修改。

2）如果有两条分数相同，排名应该怎那么看？ 如果两个分数值形同，会根据值两个元素变量名的字典排序顺序排列先后，可看下方操作代码。

代码实际操作过程：

```go
127.0.0.1:6379[1]> zadd zset1 10.1 val1 //添加一个值和分数
(integer) 1
127.0.0.1:6379[1]> zadd zset1 11.1 val2
(integer) 1
127.0.0.1:6379[1]> zadd zset1 9.2 val3
(integer) 1
127.0.0.1:6379[1]> zcard zset1 //统计当前key下值的个数
(integer) 3
127.0.0.1:6379[1]> zrange zset1 0 2 withscores  //查看0到2的所有值和分数按照排名
1) "val3"
2) "9.1999999999999993"
3) "val1"
4) "10.1"
5) "val2"
6) "11.1"
127.0.0.1:6379[1]> zrank zset1 val2
(integer) 2
127.0.0.1:6379[1]> zadd zset1 12.2 val3 //覆盖iu该val3
(integer) 0
127.0.0.1:6379[1]> zrange zset1 0 2 withscores//查看0到2的所有值和分数按照排名
1) "val1"
2) "10.1"
3) "val2"
4) "11.1"
5) "val3"
6) "12.199999999999999"
127.0.0.1:6379[1]> zadd zset1 12.2 val2
(integer) 0
127.0.0.1:6379[1]> zrange zset1 0 2 withscores//这时候有两个分数相同,查看0到2的所有值和分数按照排名
1) "val1"
2) "10.1"
3) "val2"
4) "12.199999999999999"
5) "val3"
6) "12.199999999999999"
```

## 四、基本命令

### 字符串

String 是redis最基本的类型，value 不仅可以是 String,也可以是数字。

使用 Strings 类型,可以完全实现目前Memcached 的功能,并且效率更高。还可以享受 Redis 的定时持久化(可以选择 RDB 模式或者 AOF 模式).

string类型是二进制安全的。意思是redis的string可以包含任何数据,比如jpg图片或者序列化的对象

string类型是Redis最基本的数据类型，一个键最大能存储512MB。

**命令示例：**

`set` ­­ 设置key对应的值为string类型的value。

```
> set name itcast
```

`setnx` ­­ 将key设置值为value，如果key不存在，这种情况下等同SET命令。 当key存在时，什么也不做。**SETNX是”SET if Not eXists”的简写**。

```
> get name
"itcast"
> setnx name itcast_new
(integer)0
>get name
"itcast"
```

`setex` ­­ 设置key对应字符串value，并且设置key在给定的`seconds`时间之后超时过期。

```
> setex color 10 red 
> get color
"red"
10秒后...
> get color (nil)
```

`setrange` ­­ 覆盖key对应的string的一部分，从指定的offset处开始，覆盖value的长度。

```
127.0.0.1:6379> set email wangbaoqiang@itcast.cnOK
127.0.0.1:6379> setrange email 13 gmail.com 
(integer) 22
127.0.0.1:6379> get email
"wangbaoqiang@gmail.com"
127.0.0.1:6379>STRLEN email
(integer) 22
```

其中的4是指从下标为13(包含13)的字符开始替换

`mset` ­­ 一次设置多个key的值,成功返回ok表示所有的值都设置了,失败返回0表示没有任何值被设置。

```
> mset key1 python key2 c++  
OK
```

`mget` ­­ 一次获取多个key的值,如果对应key不存在,则对应返回nil。

```
> mget key1 key2 key3  
1) "python"     
2) "c++"    
3) (nil)
```

`msetnx` ­­ 对应给定的keys到他们相应的values上。只要有一个key已经存在，MSETNX一个操作都不会执行。

```
> MSETNX key11 "Hello" key22 "there"
(integer) 1
> MSETNX key22 "there" key33 "world"
(integer) 0
```

认证了：MSETNX是原子的，所以所有给定的keys是一次性set的

`getset` ­­ 设置key的值,并返回key的**旧值**

```
> get name
"itcast"
> getset name itcast_new
"itcast"
> get name"itcast_new"
```

`GETRANGE key start end` ­­ 获取指定key的value值的子字符串。是由start和end位移决定的

```
> getrange name 0 4  
"itcas"
```

`incr` ­­ 对key的值加1操作

```
> set age 20 
> incr age 
(integer) 21
```

`incrby` ­­ 同incr类似,加指定值 ,key不存在时候会设置key,并认为原来的value是 0

```
> incrby age 5  
(integer) 26
> incrby age1111 5
(integer) 5
> get age1111
"5"
```

`decr` ­­ 对key的值做的是减减操作,decr一个不存在key,则设置key为­1

`decrby` ­­ 同decr,减指定值

`append` ­­ 给指定key的字符串值追加value,返回新字符串值的长度。例如我们向name的值追加一个"redis"字符串:

```go
127.0.0.1:6379> get name
"itcast_new"
127.0.0.1:6379> append name "value"
(integer) 15
127.0.0.1:6379> get name
"itcast_newvalue"
127.0.0.1:6379>
```

### HASH 哈希

Redis hash 是一个string类型的field和value的映射表，hash特别适合用于存储对象。

Redis 中每个 hash 可以存储 232 - 1 键值对（40多亿）。

**示例**

`HSET key field value` ­­ 设置 key 指定的哈希集中指定字段的值

```
> hset myhash field1 Hello
```

`hget` ­­ 获取指定的hash field。

```
> hget myhash field1   
"Hello"
> hget myhash field3   
(nil)
```

由于数据库没有field3,所以取到的是一个空值nil.

`HSETNX key field value` ­­ 只在 key 指定的哈希集中不存在指定的字段时，设置字段的值。如果 key 指定的哈希集不存在，会创建一个新的哈希集并与 key 关联。如果字段已存在，该操作无效果。

```
> hsetnx myhash field "Hello"   
(integer) 1
> hsetnx myhash field "Hello"   
(integer) 0
```

第一次执行是成功的,但第二次执行相同的命令失败,原因是field已经存在了。

`hmset` ­­ 同时设置hash的多个field。

```
> hmset myhash field1 Hello field2 World   
> OK
```

`hmget` ­­ 获取全部指定的hash filed。

```
> hmget myhash field1 field2 field3   
1) "Hello"
2) "World"
3) (nil)
```

`hincrby` ­­ 指定的hash filed 加上给定值。

```
> hset myhash field3 20   
(integer) 1
> hget myhash field3   
"20"
> hincrby myhash field3 -8   
(integer) 12
> hget myhash field3   
"12"
```

`hexists` ­­ 测试指定field是否存在。

```
> hexists myhash field1  
(integer) 1
> hexists myhash field9  
(integer) 0     
通过上例可以说明field1存在,但field9是不存在的。
```

`hdel` 从 key 指定的哈希集中移除指定的域

```
127.0.0.1:6379> hkeys myhash
1) "field1"
2) "field"
3) "field2"
4) "field3"
127.0.0.1:6379> hdel myhash field
(integer) 1
127.0.0.1:6379> hkeys myhash
1) "field1"
2) "field2"
3) "field3"
```

`hlen` ­­ 返回指定hash的field数量。

```
> hlen myhash  
(integer) 3
```

`hkeys` ­­ 返回hash的所有field。

```
> hkeys myhash   
> 1) "field2"   
> 2) "field"   
> 3) "field3"
```

说明这个hash中有3个field。

`hvals` ­­ 返回hash的所有value。

```
> hvals myhash   
1) "World"   
2)"Hello"   
3)"12"
```

说明这个hash中有3个field。

`hgetall` ­­ 获取某个hash中全部的filed及value。

```
> hgetall myhash   
1) "field2"   
2) "World" 
3) "field"  
4) "Hello"   
5) "field3"   
6) "12"
```

`HSTRLEN` — 返回 hash指定field的value的字符串长度

```go
127.0.0.1:6379> HSTRLEN myhash field1
(integer) 5
```

### List 列表

Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素导列表的头部（左边）或者尾部（右边）

一个列表最多可以包含 232 - 1 个元素 (4294967295, 每个列表超过40亿个元素)。

**RPUSH key value [value …]**

向存于 key 的列表的尾部插入所有指定的值。如果 key 不存在，那么会创建一个空的列表然后再进行 push 操作。

```
redis> RPUSH mylist "hello"
(integer) 1
redis> RPUSH mylist "world"
(integer) 2
redis> LRANGE mylist 0 -1
1) "hello"
2) "world"
```

**LPOP key**

移除并且返回 key 对应的 list 的第一个元素。

```
redis> RPUSH mylist "one"
(integer) 1
redis> RPUSH mylist "two"
(integer) 2
redis> RPUSH mylist "three"
(integer) 3
redis> LPOP mylist
"one"
redis> LRANGE mylist 0 -1
1) "two"
2) "three"
```

**LTRIM key start stop**

修剪(trim)一个已存在的 list，这样 list 就会只包含指定范围的指定元素。

> start 和 stop 都是由0开始计数的， 这里的 0 是列表里的第一个元素（表头），1 是第二个元素，以此类推。

例如： LTRIM foobar 0 2 将会对存储在 foobar 的列表进行修剪，只保留列表里的前3个元素。

start 和 end 也可以用负数来表示与表尾的偏移量，比如 -1 表示列表里的最后一个元素， -2 表示倒数第二个，等等。

**应用场景:**

1.取最新N个数据的操作

比如典型的取你网站的最新文章，通过下面方式，我们可以将最新的5000条评论的ID放在Redis的List集合中，并将超出集合部分从数据库获取

- 使用LPUSH latest.comments命令，向list集合中插入数据

- 插入完成后再用LTRIM latest.comments 0 5000命令使其永远只保存最近5000个ID

- 然后我们在客户端获取某一页评论时可以用下面的逻辑（伪代码）

  FUNCTION get_latest_comments(start,num_items):

  id_list = redis.lrange("latest.comments",start,start+num_items-1)

  IF id_list.length < num_items

  ```
  id_list = SQL_DB(&#34;SELECT ... ORDER BY time LIMIT ...&#34;)
  ```

  END

  RETURN id_list

如果你还有不同的筛选维度，比如某个分类的最新N条，那么你可以再建一个按此分类的List，只存ID的话，Redis是非常高效的。

**示例**

取最新N个评论的操作

```go
127.0.0.1:6379> lpush mycomment 100001
(integer) 1
127.0.0.1:6379> lpush mycomment 100002
(integer) 2
127.0.0.1:6379> lpush mycomment 100003
(integer) 3
127.0.0.1:6379> lpush mycomment 100004
(integer) 4
127.0.0.1:6379> LRANGE mycomment 0 -1
1) "100004"
2) "100003"
3) "100002"
4) "100001"
127.0.0.1:6379> LTRIM mycomment 0 1
OK
127.0.0.1:6379> LRANGE mycomment 0 -1
1) "100004"
2) "100003"
127.0.0.1:6379> lpush mycomment 100005
(integer) 3
127.0.0.1:6379> LRANGE mycomment 0 -1
1) "100005"
2) "100004"
3) "100003"
127.0.0.1:6379> LTRIM mycomment 0 1
OK
127.0.0.1:6379> LRANGE mycomment 0 -1
1) "100005"
2) "100004"
```

### Set 集合

Set 就是一个集合,集合的概念就是一堆不重复值的组合。利用 Redis 提供的 Set 数据结构,可以存储一些集合性的数据。

> 比如在 微博应用中,可以将一个用户所有的关注人存在一个集合中,将其所有粉丝存在一个集合。

因为 Redis 非常人性化的为集合提供了 **求交集、并集、差集**等操作, 那么就可以非常方便的实现如共同关注、共同喜好、二度好友等功能, 对上面的所有集合操作,你还可以使用不同的命令选择将结果返回给客户端还是存集到一个新的集合中。

**SADD key member [member …]**

添加一个或多个指定的member元素到集合的 key中

```
redis> SADD myset "Hello"
(integer) 1
redis> SADD myset "World"
(integer) 1
redis> SADD myset "World"
(integer) 0
redis> SMEMBERS myset
1) "World"
2) "Hello"
```

**SCARD key**

返回集合存储的key的基数 (集合元素的数量).

```
redis> SADD myset "Hello"
(integer) 1
redis> SADD myset "World"
(integer) 1
redis> SCARD myset
(integer) 2
```

**SDIFF key [key …]**

返回一个集合与给定集合的差集的元素.

```
redis> SADD key1 'a' 'b' 'c'
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SDIFF key1 key2
1) "a"
2) "b"
```

**应用场景**

1.共同好友、二度好友

2.利用唯一性,可以统计访问网站的所有独立 IP

3.好友推荐的时候,根据 tag 求交集,大于某个 临界值 就可以推荐

**示例**

以王宝强和马蓉为例，求二度好友，共同好友，推荐系统

```go
127.0.0.1:6379> sadd marong_friend 'songdan' 'wangsicong' 'songzhe'
(integer) 1
127.0.0.1:6379> SMEMBERS marong_friend
1) "songzhe"
2) "wangsicong"
3) "songdandan"
127.0.0.1:6379> sadd wangbaoqiang_friend 'dengchao' 'angelababy' 'songzhe'
(integer) 
1#求共同好友
127.0.0.1:6379> SINTER marong_friend wangbaoqiang_friend
1) "songzhe"
#推荐好友系统
127.0.0.1:6379> SDIFF marong_friend wangbaoqiang_friend
1) "wangsicong"
2) "songdandan"
```

### Sorted Set 有序集合

Redis 有序集合和集合一样也是string类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

有序集合的成员是唯一的,但分数(score)却可以重复。

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。 集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。

**ZADD key score member**

将所有指定成员添加到键为key有序集合（sorted set）里面

```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 1 "uno"
(integer) 1
redis> ZADD myzset 2 "two" 3 "three"
(integer) 2
redis> ZRANGE myzset 0 -1 WITHSCORES
1) "one"
2) "1"
3) "uno"
4) "1"
5) "two"
6) "2"
7) "three"
8) "3"
```

**ZCOUNT key min max**

返回有序集key中，score值在min和max之间(默认包括score值等于min或max)的成员

```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZCOUNT myzset -inf +inf
(integer) 3
redis> ZCOUNT myzset (1 3
(integer) 2
```

**ZINCRBY key increment member**

为有序集key的成员member的score值加上增量increment

```
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZINCRBY myzset 2 "one"
"3"
redis> ZRANGE myzset 0 -1 WITHSCORES
1) "two"
2) "2"
3) "one"
4) "3"
```

**应用场景**

1.带有权重的元素,LOL游戏大区最强王者

2 排行榜

**案例**

斗地主大赛排名

- 初始比赛

  ```
  127.0.0.1:6379> ZADD doudizhu_rank 0 "player1"
  (integer) 1
  127.0.0.1:6379> ZADD doudizhu_rank 0 "player2"
  (integer) 1
  127.0.0.1:6379> ZADD doudizhu_rank 0 "player3"
  (integer) 1
  ```

- 比赛开始，经过n轮比赛，每次统计，类似计算如下所示

  ```
  127.0.0.1:6379> ZINCRBY doudizhu_rank 3 player3
  "3"
  127.0.0.1:6379> ZINCRBY doudizhu_rank -1 player2
  "-1"
  127.0.0.1:6379> ZINCRBY doudizhu_rank -2 player1
  "-2"
  ```

- 比赛结束，进行排名

  ```
  127.0.0.1:6379> ZRANGE doudizhu_rank 0 -1
  1) "player1"
  2) "player2"
  3) "player3"
  127.0.0.1:6379> ZRANGE doudizhu_rank 0 -1 withscores
  1) "player1"
  2) "-2"
  3) "player2"
  4) "-1"
  5) "player3"
  6) "3"
  ```

逆序排序才对

```go
127.0.0.1:6379> zrevrange doudizhu_rank 0 -1 withscores
1)"player3"
2)"3"
3)"player2"
4)"-1"
5)"player1"
6)"-2"
```

## 五、Redis 订阅发布模式

Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。Redis 客户端可以订阅任意数量的频道。下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2、client5 和 client1 之间的关系：

![4. Redis 订阅和发布模式  - 图1](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/e15be5905805e04e38cc9656a7223963.png)

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：

![4. Redis 订阅和发布模式  - 图2](https://raw.githubusercontent.com/jiutiananshu/Picture/master/img/572365ab2d08fed5896a8ce543de77ca.png)

#### SUBSCRIBE channel [channel …]

订阅给指定频道的信息。

#### PUBLISH channel message

将信息 message 发送到指定的频道 channel

### 应用场景

在Redis中，你可以设定对某一个key值进行消息发布及消息订阅，当一个key值上进行了消息发布后，所有订阅它的客户端都会收到相应的消息。这一功能最明显的用法就是用作实时消息系统，比如普通的即时聊天，群聊等功能。

- 今日头条订阅号、微信订阅公众号、新浪微博关注、邮件订阅系统
- 即时通信系统（QQ、微信）
- 群聊部落系统（微信群）

### 案例

微信班级群`class:20170101`，发布订阅模型

**学生 A B C:**

------

**学生C:**

订阅一个`主题`名叫： `class:20170101`

```
127.0.0.1:6379> SUBSCRIBE class:20170101Reading messages... (press Ctrl-C to quit)1) "subscribe"2) "redisChat"3) (integer) 1
```

**学生A:**

针对 `class:20170101` 主题发送 消息，那么所有订阅该主题的用户都能够收到该数据。

```
127.0.0.1:6379> PUBLISH class:20170101 "i love peace!"(integer) 1
```

**学生B:**

针对 `class:20170101` 主题发送 消息，那么所有订阅该主题的用户都能够收到该数据。

```
127.0.0.1:6379> PUBLISH class:20170101 "go to hell"(integer) 1
```

最后学生C会收到 A 和 B 发送过来的消息。

```go
python@ubuntu:~$ redis-cli 
127.0.0.1:6379> SUBSCRIBE class:20170101
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "class:20170101"
3) (integer) 1
1) "message"
2) "class:20170101"
3) "i love peace!"
1) "message"
2) "class:20170101"
3) "i love peace!"
1) "message"
2) "class:20170101"
3) "go to hell"
```

## 六、Redis 事务

Redis事务允许一组命令在单一步骤中执行。事务有两个属性，说明如下：

- 事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。
- Redis事务是原子的。原子意味着要么所有的命令都执行，要么都不执行；

一个事务从开始到执行会经历以下三个阶段：

- 开始事务

- 命令入队

- 执行事务

  ```
  redis 127.0.0.1:6379> MULTIOKList of commands here
  redis 127.0.0.1:6379> EXEC
  ```

### 案例

银行转账，邓超给宝强转账1万元：

```go
127.0.0.1:6379> set dengchao 60000
OK
127.0.0.1:6379> set wangbaoqiang 200
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> INCRBY dengchao -10000
QUEUED
127.0.0.1:6379> INCRBY wangbaoqiang 10000
QUEUED
127.0.0.1:6379> EXEC
1) (integer) 50000
2) (integer) 10200
```

## 七、Redis 使用场景

1.取最新N个数据的操作

比如典型的取你网站的最新文章,我们可以将最新的5000条评论的ID放在Redis的List集合中,并将超出集合部分从数据库获取。

------

2.排行榜应用,取TOP N操作

这个需求与上面需求的不同之处在于,前面操作以时间为权重,这个是以某个条件为权重,比如按顶的次数排序, 这时候就需要我们的sorted set出马了,将你要排序的值设置成sorted set的score, 将具体的数据设置成相应的value,每次只需要执行一条ZADD命令即可。

------

3.需要精准设定过期时间的应用

比如你可以把上面说到的sorted set的score值设置成过期时间的时间戳,那么就可以简单地通过过期时间排序, 定时清除过期数据了,不仅是清除Redis中的过期数据,你完全可以把Redis里这个过期时间当成是对数据库中数据的索引, 用Redis来找出哪些数据需要过期删除,然后再精准地从数据库中删除相应的记录。

------

4.计数器应用

Redis的命令都是原子性的,你可以轻松地利用INCR,DECR命令来构建计数器系统。

------

5.uniq操作,获取某段时间所有数据排重值

这个使用Redis的set数据结构最合适了,只需要不断地将数据往set中扔就行了,set意为集合,所以会自动排重。

------

6.Pub/Sub构建实时消息系统

Redis的Pub/Sub系统可以构建实时的消息系统,比如很多用Pub/Sub构建的实时聊天系统的例子。

------

7.构建队列系统

使用list可以构建队列系统,使用sorted set甚至可以构建有优先级的队列系统。

------

8.缓存

最常用,性能优于Memcached(被libevent拖慢),数据结构更多样化。

## 八、Redis 数据备份与恢复

Redis SAVE 命令用于创建当前数据库的备份。

#### 语法

redis Save 命令基本语法如下：

```
redis 127.0.0.1:6379> SAVE
```

### 实例

```
redis 127.0.0.1:6379> SAVE 
OK
```

该命令将在 redis 安装目录中创建dump.rdb文件。

### 恢复数据

如果需要恢复数据，只需将备份文件 (dump.rdb) 移动到 redis 安装目录并启动服务即可。获取 redis 目录可以使用 CONFIG 命令，如下所示：

```
redis 127.0.0.1:6379> CONFIG GET dir
1) "dir"
2) "/home/python"
```

以上命令 CONFIG GET dir 输出的 redis 安装目录为 /home/python。

### Bgsave

创建 redis 备份文件也可以使用命令 BGSAVE，该命令在后台执行。

### 实例

```go
127.0.0.1:6379> BGSAVEBackground saving started
```

## 九、Redis 配置文件

在启动Redis服务器时,我们需要为其指定一个配置文件,缺省情况下配置文件在Redis的源码目录下,文件名为**redis.conf。**

redis配置文件使用`#######################`被分成了几大块区域,

主要有:

- 通用(general)
- 快照(snapshotting)
- 复制(replication)
- 安全(security)
- 限制(limits)
- 追加模式(append only mode)
- LUA脚本(lua scripting)
- REDIS集群(REDIS CLUSTER)
- 慢日志(slow log)
- 事件通知(event notification)
- ADVANCED CONFIG
  为了对Redis的系统实现有一个直接的认识,我们首先来看一下Redis的配置文件中定义了哪些主要参数以及这些参数的作用。

------

- daemonize no
  默认情况下,redis不是在后台运行的。如果需要在后台运行,把该项的值更改为yes;

------

- pidfile /var/run/redis_6379.pid
  当Redis在后台运行的时候,Redis默认会把pid文件放在/var/run/redis.pid, 你可以配置到其他地址。当运行多个redis服务时,需要指定不同的pid文件和端口;

------

- port 6379
  指定redis运行的端口,默认是6379;

作者在自己的一篇博文中解释了为什么选用6379作为默认端口，因为6379在手机按键上MERZ对应的号码，而MERZ取自意大利歌女Alessia Merz的名字

------

- bind 127.0.0.1
  指定redis只接收来自于该IP地址的请求,如果不进行设置,那么将处理所有请求。在生产环境中最好设置该项;

------

- loglevel notice
  指定日志记录级别,

其中Redis总共支持四个级别: debug 、verbose 、notice 、warning, 默认为 notice 。

- debug表示记录很多信息,用于开发和测试。
- verbose表示记录有用的信息, 但不像debug会记录那么多。
- notice表示普通的verbose,常用于生产环境。
- warning 表示只有非常重要或者严重的信息会记录到日志;

------

- logfile ""
  配置log文件地址,默认值为stdout。若后台模式会输出到/dev/null("黑洞");（可以自定义配置：/var/log/redis/redis.log）

------

- databases 16
  可用数据库数,默认值为16,默认数据库为0,数据库范围在0­ 15之间切换,彼此隔离;

------

- save
  保存数据到磁盘,格式为 `save <seconds> <changes>`,指出在多长时间内,有多少次更新操作, 就将数据同步到数据文件rdb。相当于条件触发抓取快照,这个可以多个条件配合。

保存数据到磁盘:

- save 900 1 #900秒（15分钟）内至少有1个key被改变
- save 300 10 #300秒（5分钟）内至少有10个key被改变
- save 60 10000 #60秒内至少有10000个key被改变

------

- rdbcompression yes
  存储至本地数据库时(持久化到rdb文件)是否压缩数据,默认为yes;

如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变的巨大

------

- dbfilename dump.rdb
  本地持久化数据库文件名,默认值为dump.rdb;

------

- dir ./
  工作目录,数据库镜像备份的文件放置的路径。

------

- slaveof
  主从复制,设置该数据库为其他数据库的从数据库。设置当本机为slave服务时,设置master服务的IP地址及端口。 在Redis启动时,它会自动从master进行数据同步;

------

- masterauth
  ￼当master服务设置了密码保护时(用requirepass制定的密码)slave服务连接master的密码;

------

- slave-serve-stale-data yes
  当一个slave失去和master的连接，或者同步正在进行中，slave的行为有两种可能：

\1) 如果 slave-serve-stale-data 设置为 "yes" (默认值)，slave会继续响应客户端请求，可能是正常数据，也可能是还没获得值的空数据。

\2) 如果 slave-serve-stale-data 设置为 "no"，slave会回复"正在从master同步（SYNC with master in progress）"来处理各种请求，除了 INFO 和 SLAVEOF 命令。

------

- repl-ping-slave-period 10
  从库会按照一个时间间隔向主库发送PING,可以通过repl-ping-slave-period设置这个时间间隔,默认是10秒;

------

- repl-timeout 60
  设置主库批量数据传输时间或者ping回复时间间隔,默认值是60秒,一定要确保repl-timeout大于repl-ping-slave-period ;不然会经常检测到超时。

master检测到slave上次发送的时间超过repl-timeout，即认为slave离线，清除该slave信息。

slave检测到上次和master交互的时间超过repl-timeout，则认为master离线

------

- requirepass foobared
  设置客户端连接后进行任何其他指定前需要使用的密码。因为redis速度相当快,所以在一台比较好的服务器下, 一个外部的用户可以在一秒钟进行150K次的密码尝试,这意味着你需要指定非常强大的密码来防止暴力破解;

------

- rename­command CONFIG ""
  命令重命名,在一个共享环境下可以重命名相对危险的命令,比如把CONFIG重名为一个不容易猜测的字符: rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52。 如果想删除一个命令,直接把它重命名为一个空字符""即可:rename-command CONFIG "";

------

- maxclients 128
  设置最多同时连接客户端数量。

默认没有限制，这个关系到Redis进程能够打开的文件描述符数量。

特殊值"0"表示没有限制。

一旦达到这个限制，Redis会关闭所有新连接并发送错误"达到最大用户数上限（max number of clients reached）"

------

- maxmemory
  指定Redis最大内存限制。Redis在启动时会把数据加载到内存中,达到最大内存后,Redis会先尝试清除已到期或即将到期的Key, Redis同时也会移除空的list对象。当此方法处理后,仍然到达最大内存设置,将无法再进行写入操作,但仍然可以进行读取操作。

注意:Redis新的vm机制,会把Key存放内存,Value会存放在swap区;

------

- maxmemory-policy volatile-lru
  当内存达到最大值的时候Redis会选择删除哪些数据呢?有五种方式可供选择:
- volatile-lru 代表利用LRU算法移除设置过期时间的key(LRU:最近使用 LeastRecentlyUsed)
- allkeys-lru 代表利用LRU算法移除任何key
- volatile-random 代表移除设置过过期时间的随机key
- allkeys_random 代表移除一个随机的key,
- volatile-ttl 代表移除即将过期的key(minor TTL)
- noeviction 代表不移除任何key,只是返回一个写错误。
  注意:对于上面的策略,如果没有合适的key可以移除,写的时候Redis会返回一个错误;

------

- appendonly no
  是否开启aof功能

默认情况下,redis会在后台异步的把数据库镜像备份到磁盘,但是该备份是非常耗时的,而且备份也不能很频繁。 如果发生诸如拉闸限电、拔插头等状况,那么将造成比较大范围的数据丢失,所以redis提供了另外一种更加高效的数据库备份及灾难恢复方式。

开启appendonly模式之后,redis会把所接收到的每一次写操作请求都追加到appendonly.aof文件中。

当redis重新启动时,会从该文件恢复出之前的状态

------

- appendfilename appendonly.aof
  AOF文件名称,默认为"appendonly.aof";

------

- appendfsync everysec
  Redis支持三种同步AOF文件的策略:
- no 代表不进行同步,系统去操作,
- always 代表每次有写操作都进行同步,
- everysec 代表对写操作进行累积,每秒同步一次
  默认是"everysec",按照速度和安全折中这是最好的。

------

- slowlog-log-slower-than 10000
  记录超过特定执行时间的命令。执行时间不包括I/O计算,比如连接客户端,返回结果等,只是命令执行时间。

可以通过两个参数设置slow log:一个是告诉Redis执行超过多少时间被记录的参数slowlog-log-slower-than(微妙), 另一个是slow log 的长度。当一个新命令被记录的时候最早的命令将被从队列中移除,

下面的时间以微秒(百万分之一秒,1000 * 1000)单位, 因此1000000代表一分钟。

```
1秒=1000毫秒1秒=1000000微秒
```

注意制定一个负数将关闭慢日志,而设置为0将强制每个命令都会记录;

------

- slowlog-max-len 128
  慢操作日志"保留的最大条数

"记录"将会被队列化,如果超过了此长度,旧记录将会被移除。可以通过`SLOWLOG <subcommand> args`查看慢记录的信息(SLOWLOG get 10,SLOWLOG reset)，通过"SLOWLOG get num"指令可以查看最近num条慢速记录，其中包括"记录"操作的时间/指令/K-V等信息。

------

参考：[https://github.com/linli8/cnblogs/blob/master/redis%E5%89%AF%E6%9C%AC.conf](https://github.com/linli8/cnblogs/blob/master/redis副本.conf)

### 总结

#### Select 命令

Redis Select 命令用于切换到指定的数据库，数据库索引号 index 用数字值指定，以 0 作为起始索引值。

```
python@ubuntu:/dev$ redis-cli127.0.0.1:6379> select 1OK127.0.0.1:6379[1]> set name 'itcast.cn'OK127.0.0.1:6379[1]> select 2OK127.0.0.1:6379[2]> set web 'itcast.com'OK127.0.0.1:6379[2]> get web"itcast.com"127.0.0.1:6379[2]> select 1OK127.0.0.1:6379[1]> get name"itcast.cn"127.0.0.1:6379[1]>
```

#### Shutdown 命令

Redis Shutdown 命令执行以下操作：

- 停止所有客户端

- 如果有至少一个保存点在等待，执行 SAVE 命令

- 如果 AOF 选项被打开，更新 AOF 文件

- 关闭 redis 服务器(server)

  ```
  redis 127.0.0.1:6379> PINGPONG
  redis 127.0.0.1:6379> SHUTDOWN$ redis
  ```

#### Redis 认证

```
redis-cli
AUTH "password"
```

或者

```
redis-cli -h 127.0.0.1 -p 6379 -a myPassword
```

### Redis Showlog

Redis Showlog 是 Redis 用来记录查询执行时间的日志系统。

查询执行时间指的是不包括像客户端响应(talking)、发送回复等 IO 操作，而单单是执行一个查询命令所耗费的时间。

另外，slow log 保存在内存里面，读写速度非常快，因此你可以放心地使用它，不必担心因为开启 slow log 而损害 Redis 的速度。

语法redis Showlog 命令基本语法如下：

```
redis 127.0.0.1:6379> SLOWLOG subcommand [argument]
```

查看日志信息：

```
redis 127.0.0.1:6379> slowlog get 21) 
1) (integer) 14   
2) (integer) 1309448221   
3) (integer) 15   
4) 
1) "ping"
2) 
1) (integer) 13   
2) (integer) 1309448128   
3) (integer) 30   
4) 
1) "slowlog"      
2) "get"      
3) "100"
```

> 每一个Showlog都是由四个字段组成的慢日志标识符记录的命令进行处理的Unix时间戳执行所需的时间，以微秒命令、参数。

查看当前日志的数量：

```
redis 127.0.0.1:6379> SLOWLOG LEN
(integer) 14 SLOWLOG RESET 可以清空 slow log 。redis 127.0.0.1:6379> SLOWLOG LEN(integer) 14redis 127.0.0.1:6379> SLOWLOG RESETOKredis 127.0.0.1:6379> SLOWLOG LEN(integer) 0
```

## 十、Redis 持久化

### 对Redis持久化的探讨与理解

目前Redis持久化的方式有两种： RDB 和 AOF

首先，我们应该明确持久化的数据有什么用，答案是:

> 用于重启后的数据恢复

Redis是一个内存数据库，无论是RDB还是AOF，都是其保证数据恢复的措施。

所以Redis在利用RDB和AOF进行恢复的时候，都会读取RDB或AOF文件，重新加载到内存中。

### RDB

RDB就是Snapshot快照存储，是默认的持久化方式。可理解为半持久化模式，

> 即按照一定的策略周期性的将数据保存到磁盘。对应产生的数据文件为dump.rdb，快照的周期通过配置文件中的save参数来定义。

下面是默认的快照设置：

```
dbfilename dump.rdb# save <seconds> <changes>save 900 1    #当有一条Keys数据被改变时，900秒刷新到Disk一次save 300 10   #当有10条Keys数据被改变时，300秒刷新到Disk一次save 60 10000 #当有10000条Keys数据被改变时，60秒刷新到Disk一次
```

Redis的RDB文件不会坏掉，因为其写操作是在一个新进程中进行的。当生成一个新的RDB文件时，Redis生成的子进程会先将数据写到一个临时文件中，然后通过原子性rename系统调用将临时文件重命名为RDB文件。

这样在任何时候出现故障，Redis的RDB文件都总是可用的。

同时，Redis的RDB文件也是Redis主从同步内部实现中的一环。

> 第一次Slave向Master同步的实现是：Slave向Master发出同步请求，Master先dump出rdb文件，然后将rdb文件全量传输给slave，然后Master把缓存的命令转发给Slave，初次同步完成。第二次以及以后的同步实现是：Master将变量的快照直接实时依次发送给各个Slave。但不管什么原因导致Slave和Master断开重连都会重复以上两个步骤的过程。

Redis的主从复制是建立在内存快照的持久化基础上的，只要有Slave就一定会有内存快照发生。可以很明显的看到，RDB有它的不足，就是一旦数据库出现问题，那么我们的RDB文件中保存的数据并不是全新的。

从上次RDB文件生成到Redis停机这段时间的数据全部丢掉了。

### AOF（Append-only file）方式

AOF(Append-Only File)比RDB方式有更好的持久化性。

- 在使用AOF持久化方式时，Redis会将每一个收到的写命令都通过Write函数追加到文件中，类似于MySQL的binlog。
- 当Redis重启是会通过重新执行文件中保存的写命令来在内存中重建整个数据库的内容。

在Redis重启时会逐个执行AOF文件中的命令来将硬盘中的数据载入到内存中,所以说，载入的速度相较RDB会慢一些

- 默认情况下,Redis没有开启AOF方式的持久化,可以在redis.conf中通过appendonly参数开启:

  appendonly yes #启用aof持久化方式

  # appendfsync always #每次收到写命令就立即强制写入磁盘，最慢的，但是保证完全的持久化，不推荐使用

  appendfsync everysec #每秒钟强制写入磁盘一次，在性能和持久化方面做了很好的折中，推荐

  # appendfsync no #完全依赖OS的写入，一般为30秒左右一次，性能最好但是持久化最没有保证，不被推荐。

- AOF文件和RDB文件的保存文件夹位置相同,都是通过dir参数设置的,默认的文件名是appendonly.aof,可以通过appendfilename参数修改

  appendfilename appendonly.aof

- AOF的完全持久化方式同时也带来了另一个问题，持久化文件会变得越来越大。

比如: 我们调用INCR test 命令100次，文件中就必须保存全部的100条命令，但其实99条都是多余的。
因为要恢复数据库的状态其实文件中保存一条SET test 100就够了。

为了压缩AOF的持久化文件，Redis提供了bgrewriteaof命令。收到此命令后Redis将使用与快照类似的方式将内存中的数据以命令的方式保存到临时文件中，最后替换原来的文件，以此来实现控制AOF文件的增长。

配置redis自动重写AOF文件的参数

no-appendfsync-on-rewrite yes #在AOF重写时，不进行命令追加操作，而只是将其放在缓冲区里，避免与命令的追加造成`DISK IO`上的冲突。
auto-aof-rewrite-percentage 100 #当前AOF文件大小是上次日志重写得到AOF文件大小的二倍时，自动启动新的日志重写过程。
auto-aof-rewrite-min-size 64mb #当前AOF文件启动新的日志重写过程的最小值，避免刚刚启动Reids时由于文件尺寸较小导致频繁的重写。

### 到底选择什么呢？

下面是来自官方的建议：

通常，如果你要想提供很高的数据保障性，那么建议你同时使用两种持久化方式。

> 如果你可以接受灾难带来的几分钟的数据丢失，那么你可以仅使用RDB。很多用户仅使用了AOF，但是我们建议，既然RDB可以时不时的给数据做个完整的快照，并且提供更快的重启，所以建议也使用RDB。

因此，我们希望可以在未来（长远计划）统一AOF和RDB成一种持久化模式。

在数据恢复方面：RDB的启动时间会更短，原因有两个：

> 一是RDB文件中每一条数据只有一条记录，不会像AOF日志那样可能有一条数据的多次操作记录。所以每条数据只需要写一次就行了。另一个原因是RDB文件的存储格式和Redis数据在内存中的编码格式是一致的，不需要再进行数据编码工作，所以在CPU消耗上要远小于AOF日志的加载。

二、灾难恢复模拟

既然持久化的数据的作用是用于重启后的数据恢复，那么我们有必要进行一次这样的灾难恢复模拟了。

> 如果数据要做持久化又想保证稳定性，则建议留空一半的物理内存。因为在进行快照的时候，fork出来进行dump操作的子进程会占用与父进程一样的内存，真正的copy-on-write，对性能的影响和内存的耗用都是比较大的。

目前，通常的设计思路是利用`Replication`机制来弥补aof、snapshot性能上的不足，达到了数据可持久化。

> 即Master上Snapshot和AOF都不做，来保证Master的读写性能，而Slave上则同时开启Snapshot和AOF来进行持久化，保证数据的安全性。

首先，修改Master上的如下配置：

```
$ sudo vim /redis/etc/redis.conf#save 900 1 #禁用Snapshot#save 300 10#save 60 10000appendonly no #禁用(注释)AOF
```

接着，修改Slave上的如下配置：

```
$ sudo vim /redis/etc/redis.confsave 900 1 #启用Snapshotsave 300 10save 60 10000appendonly yes #启用AOFappendfilename appendonly.aof #AOF文件的名称# appendfsync alwaysappendfsync everysec #每秒钟强制写入磁盘一次# appendfsync no  no-appendfsync-on-rewrite yes   #在日志重写时，不进行命令追加操作auto-aof-rewrite-percentage 100 #自动启动新的日志重写过程auto-aof-rewrite-min-size 64mb  #启动新的日志重写过程的最小值
```

分别启动Master与Slave

```
$ redis-server /etc/redis/redis.conf
```

启动完成后在Master中确认未启动Snapshot参数

```
redis 127.0.0.1:6379> CONFIG GET save1) "save"2) ""
```

然后通过以下脚本在Master中生成25万条数据：

```
python@redis:$ cat redis-cli-generate.temp.sh
#!/bin/bashREDISCLI="redis-cli -a slavepass -n 1 SET"ID=1while(($ID<50001))do  INSTANCE_NAME="i-2-$ID-VM"  UUID=`cat /proc/sys/kernel/random/uuid`  PRIVATE_IP_ADDRESS=10.`echo "$RANDOM % 255 + 1" | bc`.`echo "$RANDOM % 255 + 1" | bc`.`echo "$RANDOM % 255 + 1" | bc`\  CREATED=`date "+%Y-%m-%d %H:%M:%S"`  $REDISCLI vm_instance:$ID:instance_name "$INSTANCE_NAME"  $REDISCLI vm_instance:$ID:uuid "$UUID"  $REDISCLI vm_instance:$ID:private_ip_address "$PRIVATE_IP_ADDRESS"  $REDISCLI vm_instance:$ID:created "$CREATED"  $REDISCLI vm_instance:$INSTANCE_NAME:id "$ID"  ID=$(($ID+1))done
python@redis:$ ./redis-cli-generate.temp.sh
```

在数据的生成过程中，可以很清楚的看到Master上仅在第一次做Slave同步时创建了dump.rdb文件，之后就通过增量传输命令的方式给Slave了。dump.rdb文件没有再增大。

```
python@redis:/opt/redis/data/6379$ ls -lhtotal 4.0K-rw-r--r-- 1 root root 10 Sep 27 00:40 dump.rdb
```

而Slave上则可以看到dump.rdb文件和AOF文件在不断的增大，并且AOF文件的增长速度明显大于dump.rdb文件。

```
python@redis-slave:$ ls -lhtotal 24M-rw-r--r-- 1 root root 15M Sep 27 12:06 appendonly.aof-rw-r--r-- 1 root root 9.2M Sep 27 12:06 dump.rdb
```

等待数据插入完成以后，首先确认当前的数据量。

```
redis 127.0.0.1:6379> info......used_memory:33055824used_memory_human:31.52Mused_memory_rss:34717696used_memory_peak:33055800used_memory_peak_human:31.52M...# Keyspacedb0:keys=1,expires=0,avg_ttl=0db1:keys=250000,expires=0
```

当前的数据量为`db0:keys=1 db1:keys=250000`条，占用内存31.52M。

然后我们直接Kill掉Master的Redis进程，模拟灾难。

```
python@redis:$ sudo killall -9 redis-server
```

我们到Slave中查看状态：

```
redis 127.0.0.1:6379> info......used_memory:33047696used_memory_human:31.52Mused_memory_rss:34775040used_memory_peak:33064400used_memory_peak_human:31.53M...master_host:10.6.1.143master_port:6379master_link_status:downmaster_last_io_seconds_ago:-1master_sync_in_progress:0...# Keyspacedb0:keys=1,expires=0,avg_ttl=0db1:keys=250000,expires=0
```

可以看到`master_link_status`的状态已经是down了，Master已经不可访问了。而此时，Slave依然运行良好，并且保留有AOF与RDB文件。

下面我们将通过Slave上保存好的AOF与RDB文件来恢复Master上的数据。

> 首先，将Slave上的同步状态取消，避免主库在未完成数据恢复前就重启，进而直接覆盖掉从库上的数据，导致所有的数据丢失。

如果Redis服务器已经充当从站命令`SLAVEOF NO ONE` 会关掉复制，转Redis服务器为主

```
redis 127.0.0.1:6379> SLAVEOF NO ONEOK
```

确认一下已经没有了master相关的配置信息：

```
redis 127.0.0.1:6379> INFO...role:master...
```

在Slave上复制数据文件：

```
python@redis-slave:$ tar cvf data.tar *appendonly.aofdump.rdb
```

将data.tar上传到Master上，尝试恢复数据:可以看到Master目录下有一个初始化Slave的数据文件，很小，将其删除。

```
python@redis:$ ls -ltotal 4-rw-r--r-- 1 root root 10 Sep 27 00:40 dump.rdbpython@redis:/opt/redis/data/6379$ sudo rm -f dump.rdb
```

然后解压缩数据文件：

```
python@redis:$ sudo tar xf data.tarpython@redis:$ ls -lhtotal 29M-rw-r--r-- 1 root root 18M Sep 27 01:22 appendonly.aof-rw-r--r-- 1 root root 12M Sep 27 01:22 dump.rdb
```

启动Master上的Redis；

```
python@redis:$ redis-server /etc/redis/redis.confStarting Redis server...
```

查看数据是否恢复：

```
redis 127.0.0.1:6379> INFO...db0:...db1:keys=250000,expires=0
```

可以看到25万条数据已经完整恢复到了Master上。此时，可以放心的恢复Slave的同步设置了

```
redis 127.0.0.1:6379> SLAVEOF 10.6.1.143 6379OK
```

查看同步状态：

```
redis 127.0.0.1:6379> INFO...role:slavemaster_host:10.6.1.143master_port:6379master_link_status:up...
```

### 恢复数据策略

Redis允许同时开启AOF和RDB,既保证了数据安全又使得进行备份等操作十分容易, 那么到底是哪一个文件完成了数据的恢复呢？

实际上，当Redis服务器挂掉时，重启时将按照以下优先级恢复数据到内存：

- 如果只配置AOF,重启时加载AOF文件恢复数据；
- 如果同时 配置了RDB和AOF,启动是只加载AOF文件恢复数据;
- 如果只配置RDB,启动是将加载dump文件恢复数据。
  也就是说，AOF的优先级要高于RDB，这也很好理解，因为AOF本身对数据的完整性保障要高于RDB。

#### 总结

目前的线上环境中，由于数据都设置有过期时间，采用AOF的方式会不太实用，因为过于频繁的写操作会使AOF文件增长到异常的庞大，大大超过了我们实际的数据量，这也会导致在进行数据恢复时耗用大量的时间。

因此，可以在Slave上仅开启Snapshot来进行本地化，同时可以考虑将save中的频率调高一些或者调用一个计划任务来进行定期bgsave的快照存储，来尽可能的保障本地化数据的完整性。

在这样的架构下，如果仅仅是Master挂掉，Slave完整，数据恢复可达到100%。

如果Master与Slave同时挂掉的话，数据的恢复也可以达到一个可接受的程度。

## Redis 性能测试

Redis 性能测试是通过同时执行多个命令实现的。

**语法**

redis 性能测试的基本命令如下：

```
redis-benchmark [option] [option value]
```

### 实例

- 测试存取大小为100字节的数据包的性能。

  ```
  $ redis-benchmark -h 127.0.0.1 -p 6379 -q -d 100
  ```

```
 
PING_INLINE: 85910.65 requests per second PING_BULK: 123762.38 requests per second SET: 85763.29 requests per secondGET: 81699.35 requests per secondINCR: 82372.32 requests per secondLPUSH: 83472.46 requests per secondLPOP: 82712.98 requests per secondSADD: 82236.84 requests per secondSPOP: 83963.05 requests per secondLPUSH (needed to benchmark LRANGE): 82850.04 requests per second LRANGE_100 (first 100 elements): 29585.80 requests per second LRANGE_300 (first 300 elements): 9348.42 requests per second LRANGE_500 (first 450 elements): 7562.58 requests per second LRANGE_600 (first 600 elements): 6780.58 requests per second MSET (10 keys): 94428.70 requests per second
```

| 序号 | 选项     | 描述                                       | 默认值    |
| :--- | :------- | :----------------------------------------- | :-------- |
| 1    | **-h**   | 指定服务器主机名                           | 127.0.0.1 |
| 2    | **-p**   | 指定服务器端口                             | 6379      |
| 3    | **-s**   | 指定服务器 socket                          |           |
| 4    | **-c**   | 指定并发连接数                             | 50        |
| 5    | **-n**   | 指定请求数                                 | 10000     |
| 6    | **-d**   | 以字节的形式指定 SET/GET 值的数据大小      | 2         |
| 7    | **-k**   | 1=keep alive 0=reconnect                   | 1         |
| 8    | **-r**   | SET/GET/INCR 使用随机 key, SADD 使用随机值 |           |
| 9    | **-P**   | 通过管道传输 <numreq> 请求                 | 1         |
| 10   | **-q**   | 强制退出 redis。仅显示 query/sec 值        |           |
| 11   | **—csv** | 以 CSV 格式输出                            |           |
| 12   | **-l**   | 生成循环，永久执行测试                     |           |
| 13   | **-t**   | 仅运行以逗号分隔的测试命令列表。           |           |
| 14   | **-I**   | Idle 模式。仅打开 N 个 idle 连接并等待。   |           |

- 100个并发连接,100000个请求,检测host为localhost 端口为6379的redis服务器性能

  ```
  $ redis-benchmark -h 127.0.0.1 -p 6379 -c 100 -n 100000
  ```

```

```

## 十一、redis分布式锁实现



## 十二、总结