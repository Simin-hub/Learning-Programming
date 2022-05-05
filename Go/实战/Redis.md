# Redis

[参考](https://www.bookstack.cn/books/redis-tutorial)

## 基本数据类型

Redis支持五种数据类型：string（字符串），hash（哈希），list（列表），set（集合）及zset(sorted set：有序集合)。

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

string类型是Redis最基本的数据类型，一个键最大能存储512MB。

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

### Hash 哈希

Redis hash 是一个键值对集合。

Redis hash是一个string类型的`field`和`value`的映射表，hash特别适合用于存储对象。

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

### List 列表

Redis 列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）。

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

列表最多可存储 232 - 1 元素 (4294967295, 每个列表可存储40多亿)。

### Set 集合

Redis Set是`string`类型的无序集合。

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)。

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

### zset(sorted set：有序集合)

Redis zset 和 set 一样也是string类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个double类型的分数(权重)。redis正是通过分数来为集合中的成员(权重)进行从小到大的排序。

zset的成员是唯一的,但分数(score)却可以重复。

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

## Redis 订阅发布模式

## Redis 事务

## Redis 数据备份与恢复

## Redis 配置文件

## Redis 持久化

## redis分布式锁实现

## 高可用性（主从、哨兵、集群）

所谓的高可用，也叫 HA（High Availability），是分布式系统架构设计中必须考虑的因素之一，它通常是指，通过设计减少系统不能提供服务的时间。

如果在实际生产中，如果 redis 只部署一个节点，当机器故障时，整改服务都不能提供服务了。这就是我们常说的单点故障。

如果 redis 部署了多台，当一台或几台故障时，整个系统依然可以对外提供服务，这样就提高了服务的可用性。

今天我们就聊聊 redis 高可用的三种模式：**主从模式**，**哨兵模式**，**集群模式**。

### 一、主从模式

一般，系统的高可用都是通过部署多台机器实现的。redis 为了避免单点故障，也需要部署多台机器。

因为部署了多台机器，所以就会涉及到不同机器的的数据同步问题。

为此，redis 提供了 Redis 提供了复制(replication)功能，当一台 redis 数据库中的数据发生了变化，这个变化会被自动的同步到其他的 redis 机器上去。

redis 多机器部署时，这些机器节点会被分成两类，一类是主节点（master 节点），一类是从节点（slave 节点）。一般主节点可以进行读、写操作，而从节点只能进行读操作。同时由于主节点可以写，数据会发生变化，当主节点的数据发生变化时，会将变化的数据同步给从节点，这样从节点的数据就可以和主节点的数据保持一致了。一个主节点可以有多个从节点，但是一个从节点会只会有一个主节点，也就是所谓的一主多从结构。



![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/fe86a1d3d76c8cccaff47dcd75b04a28.png)

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

首先启动主节点，然后一台一台启动从节点。

#### 1.4.主从复制的机制



![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/c0d07bc58d77a298893584b11b9f78d9.png)



- 从数据库连接主数据库，发送 SYNC 命令;
- 主数据库接收到 SYNC 命令后，可以执行 BGSAVE 命令生成 RDB 文件并使用缓冲区记录此后执行的所有写命令;
- 主数据库 BGSAVE 执行完后，向所有从数据库发送快照文件，并在发送期间继续记录被执行的写命令;
- 从数据库收到快照文件后丢弃所有旧数据，载入收到的快照;
- 主数据库快照发送完毕后开始向从数据库发送缓冲区中的写命令;
- 从数据库完成对快照的载入，开始接受命令请求，并执行来自主数据库缓冲区的写命令;(**从数据库初始化完成**)
- 主数据库每执行一个写命令就会向从数据库发送相同的写命令，从数据库接收并执行收到的写命令(**从数据库初始化完成后的操作**)
- 出现断开重连后，2.8 之后的版本会将断线期间的命令传给从数据库，增量复制。
- **主从刚刚连接的时候，进行全量同步;全同步结束后，进行增量同步**。当然，如果有需要，slave 在任何时候都可以发起全量同步。Redis 的策略是，无论如何，首先会尝试进行增量同步，如不成功，要求从机进行全量同步。

#### 1.5.主从模式的优缺点

#### 优点

- 支持主从复制，主机会自动将数据同步到从机，可以进行读写分离;
- 为了分载 Master 的读操作压力，Slave 服务器可以为客户端提供只读操作的服务，写服务依然必须由 Master 来完成;
- Slave 同样可以接受其他 Slaves 的连接和同步请求，这样可以有效地分载 Master 的同步压力;
- Master 是以非阻塞的方式为 Slaves 提供服务。所以在 Master-Slave 同步期间，客户端仍然可以提交查询或修改请求;
- Slave 同样是以阻塞的方式完成数据同步。在同步期间，如果有客户端提交查询请求，Redis 则返回同步之前的数据。

#### 缺点

- Redis **不具备自动容错和恢复功能，主机从机的宕机都会导致前端部分读写请求失败，需要等待机器重启或者手动切换前端的 IP 才能恢复;**
- 主机宕机，宕机前有部分数据未能及时同步到从机，**切换 IP 后还会引入数据不一致的问题，降低了系统的可用性;**
- 如果多个 Slave 断线了，需要重启的时候，尽量不要在同一时间段进行重启。因为只要 Slave 启动，就会发送 sync 请求和主机全量同步，当多个 Slave 重启的时候，可能会导致 Master IO 剧增从而宕机。
- Redis 较**难支持在线扩容**，在集群容量达到上限时在线扩容会变得很复杂;
- redis 的主节点和从节点中的数据是一样的，降低的内存的可用性

### 二、哨兵模式

主从模式下，当主服务器宕机后，需要手动把一台从服务器切换为主服务器，这就需要人工干预，费事费力，还会造成一段时间内服务不可用。这种方式并不推荐，实际生产中，我们优先考虑哨兵模式。这种模式下，master 宕机，哨兵会自动选举 master 并将其他的 slave 指向新的 master。

在主从模式下，redis 同时提供了哨兵命令`redis-sentinel`，哨兵是一个独立的进程，作为进程，它会独立运行。其原理是哨兵进程向所有的 redis 机器发送命令，等待 Redis 服务器响应，从而监控运行的多个 Redis 实例。

哨兵可以有多个，一般为了便于决策选举，使用奇数个哨兵。哨兵可以和 redis 机器部署在一起，也可以部署在其他的机器上。多个哨兵构成一个哨兵集群，哨兵直接也会相互通信，检查哨兵是否正常运行，同时发现 master 宕机哨兵之间会进行决策选举新的 master



![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/7a1e6c5ceef52c9a8c6f80e5aced9aee.png)

哨兵模式的作用:

- 通过发送命令，让 Redis 服务器返回监控其运行状态，包括主服务器和从服务器;
- 当哨兵监测到 master 宕机，会自动将 slave 切换到 master，然后通过*发布订阅模式*通过其他的从服务器，修改配置文件，让它们切换主机;
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

- 每个 Sentinel（哨兵）进程以每秒钟一次的频率向整个集群中的 Master 主服务器，Slave 从服务器以及其他 Sentinel（哨兵）进程发送一个 PING 命令。
- 如果一个实例（instance）距离最后一次有效回复 PING 命令的时间超过 down-after-milliseconds 选项所指定的值， 则这个实例会被 Sentinel（哨兵）进程标记为主观下线（SDOWN）
- 如果一个 Master 主服务器被标记为主观下线（SDOWN），则正在监视这个 Master 主服务器的所有 Sentinel（哨兵）进程要以每秒一次的频率确认 Master 主服务器的确进入了主观下线状态
- 当有足够数量的 Sentinel（哨兵）进程（大于等于配置文件指定的值）在指定的时间范围内确认 Master 主服务器进入了主观下线状态（SDOWN）， 则 Master 主服务器会被标记为客观下线（ODOWN）
- 在一般情况下， 每个 Sentinel（哨兵）进程会以每 10 秒一次的频率向集群中的所有 Master 主服务器、Slave 从服务器发送 INFO 命令。
- 当 Master 主服务器被 Sentinel（哨兵）进程标记为客观下线（ODOWN）时，Sentinel（哨兵）进程向下线的 Master 主服务器的所有 Slave 从服务器发送 INFO 命令的频率会从 10 秒一次改为每秒一次。
- 若没有足够数量的 Sentinel（哨兵）进程同意 Master 主服务器下线， Master 主服务器的客观下线状态就会被移除。若 Master 主服务器重新向 Sentinel（哨兵）进程发送 PING 命令返回有效回复，Master 主服务器的主观下线状态就会被移除。

假设 master 宕机，sentinel 1 先检测到这个结果，系统并不会马上进行 failover(故障转移)选出新的 master，仅仅是 sentinel 1 主观的认为 master 不可用，这个现象成为**主观下线**。当后面的哨兵也检测到主服务器不可用，并且数量达到一定值时，那么哨兵之间就会进行一次投票，投票的结果由 sentinel 1 发起，进行 failover 操作。切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从服务器实现切换主机，这个过程称为**客观下线**。这样对于客户端而言，一切都是透明的。

#### 2.5.主从模式的优缺点

#### 优点

- 哨兵模式是基于主从模式的，所有主从的优点，哨兵模式都具有。
- 主从可以自动切换，系统更健壮，可用性更高。

#### 缺点 

- 具有主从模式的缺点，每台机器上的数据是一样的，内存的可用性较低。
- Redis 较难支持在线扩容，在集群容量达到上限时在线扩容会变得很复杂。

### 三、集群模式

先说一个误区：**Redis 的集群模式本身没有使用一致性 hash 算法，而是使用 slots 插槽**。这是很多人的一个误区。这里先留个坑，后面我会出一期《 redis 系列之——一致性 hash 算法》。

Redis 的哨兵模式基本已经可以实现高可用，读写分离 ，但是在这种模式下每台 Redis 服务器都存储相同的数据，很浪费内存，所以在 redis3.0 上加入了 Cluster 集群模式，实现了 Redis 的分布式存储，对数据进行分片，也就是说每台 Redis 节点上存储不同的内容；

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/64bc0e3e8784bd3830a84be4b699df48.png)

这里的 6 台 redis 两两之间并不是独立的，每个节点都会通过集群总线(cluster bus)，与其他的节点进行通信。通讯时使用特殊的端口号，即对外服务端口号加 10000。例如如果某个 node 的端口号是 6379，那么它与其它 nodes 通信的端口号是 16379。nodes 之间的通信采用特殊的二进制协议。

对客户端来说，整个 cluster 被看做是一个整体，客户端可以连接任意一个 node 进行操作，就像操作单一 Redis 实例一样，当客户端操作的 key 没有分配到该 node 上时，Redis 会返回转向指令，指向正确的 node，这有点儿像浏览器页面的 302 redirect 跳转。

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

在 Redis 的每一个节点上，都有这么两个东西，一个是插槽（slot），它的的取值范围是：0-16383，可以从上面`redis-trib.rb`执行的结果看到这 16383 个 slot 在三个 master 上的分布。还有一个就是 cluster，可以理解为是一个集群管理的插件，类似的哨兵。

当我们的存取的 Key 到达的时候，Redis 会根据 crc16 的算法对计算后得出一个结果，然后把结果和 16384 求余数，这样每个 key 都会对应一个编号在 0-16383 之间的哈希槽，通过这个值，去找到对应的插槽所对应的节点，然后直接自动跳转到这个对应的节点上进行存取操作。

当数据写入到对应的 master 节点后，这个数据会同步给这个 master 对应的所有 slave 节点。

为了保证高可用，redis-cluster 集群引入了主从模式，一个主节点对应一个或者多个从节点。当其它主节点 ping 主节点 master 1 时，如果半数以上的主节点与 master 1 通信超时，那么认为 master 1 宕机了，就会启用 master 1 的从节点 slave 1，将 slave 1 变成主节点继续提供服务。

如果 master 1 和它的从节点 slave 1 都宕机了，整个集群就会进入 fail 状态，因为集群的 slot 映射不完整。如果集群超过半数以上的 master 挂掉，无论是否有 slave，集群都会进入 fail 状态。

redis-cluster 采用去中心化的思想，没有中心节点的说法，客户端与 Redis 节点直连，不需要中间代理层，客户端不需要连接集群所有节点，连接集群中任何一个可用节点即可。

#### 3.5.集群扩缩容

对 redis 集群的扩容就是向集群中添加机器，缩容就是从集群中删除机器，并重新将 16383 个 slots 分配到集群中的节点上（数据迁移）。

扩缩容也是使用集群管理工具 redis-tri.rb。

扩容时，先使用`redis-tri.rb add-node`将新的机器加到集群中，这是新机器虽然已经在集群中了，但是没有分配 slots，依然是不起做用的。在使用 ` redis-tri.rb reshard`进行分片重哈希（数据迁移），将旧节点上的 slots 分配到新节点上后，新节点才能起作用。

缩容时，先要使用 ` redis-tri.rb reshard`移除的机器上的 slots，然后使用`redis-tri.rb add-del`移除机器。

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



## 面试题

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

- 速度快，因为数据存在内存中，类似于 HashMap，HashMap 的优势就是查 找和操作的时间复杂度都是 O（1）
- 支持丰富数据类型，支持 string，list，set，Zset，hash 等 
- 支持事务，操作都是原子性，所谓的原子性就是对数据的更改要么全部执 行，要么全部不执行 
- 丰富的特性：可用于缓存，消息，按 key 设置过期时间，过期后将会自动 删除

### 4、Redis 相比 Memcached 有哪些优势？ 

- Memcached 所有的值均是简单的字符串，redis 作为其替代者，支持更为 丰富的数据类 
- Redis 的速度比 Memcached 快很 
- Redis 可以持久化其数据

### 5、Memcache 与 Redis 的区别都有哪些？ 

- 存储方式 Memecache 把数据全部存在内存之中，断电后会挂掉，数据不 能超过内存大小。 Redis 有部份存在硬盘上，这样能保证数据的持久性。 
- 数据支持类型 Memcache 对数据类型支持相对简单。 Redis 有复杂的数 据类型。 
- 使用底层模型不同 它们之间底层实现方式 以及与客户端之间通信的应用 协议不一样。 Redis 直接自己构建了 VM 机制 ，因为一般的系统调用系统 函 数的话，会浪费一定的时间去移动和请求。 

### 6、Redis 是单进程单线程的？ 

Redis 是单进程单线程的，redis 利用队列技术将并发访问变为串行访 问，消 除了传统数据库串行控制的开销。

###  7、一个字符串类型的值能存储最大容量是多少？ 

答：512M 

### 8、Redis 的持久化机制是什么？各自的优缺点？

 Redis 提供两种持久化机制 RDB 和 AOF 机制： 

**RDB （Redis DataBase）持久化方式：**

是指用数据集快照的方式半持久化模式

**记录 Redis 数据库的所有键值对,在某个时间点将数据写入一个临时文件，持久化结束后，用这个临时文件替换上次持久化的文件，达到数据恢复**。 

优点： 

- 只有一个文件 dump.rdb，方便持久化。 
- 容灾性好，一个文件可以保存到安全的磁盘。
- 性能最大化，fork 子进程来完成写操作，让主进程继续处理命令，所以 是 IO 最大化。使用单独子进程来进行持久化，主进程不会进行任何 IO  操 作，保证了 Redis 的高性能) 
- 相对于数据集大时，比 AOF 的启动效率更高。 

缺点：  **数据安全性低**。RDB 是间隔一段时间进行持久化，如果持久化之间 Redis 发生故障，会发生数据丢失。

所以这种方式更适合数据要求不严谨的时候。

**AOF（Append-only file）持久化方式**： 

是指**所有的命令行记录以 Redis 命令请求协议的格式完全持久化存储保存为 aof 文件**。

优点：  

- 数据安全，aof 持久化可以配置 appendfsync 属性，有 always，每进行一次命令操作就记录到 aof 文件中一次。 
- 通过 append 模式写文件，即使中途服务器宕机，可以通过 redis-check-aof 工具解决数据一致性问题。 
- AOF 机制的 rewrite 模式。AOF 文件没被 rewrite 之前（文件过大时会对命令进行合并重写），可以删除其中的某些命令（比如误操作的 flushall）) 

缺点： 

- AOF 文件比 RDB 文件大，且恢复速度慢。
- 数据集大的时候，比 rdb 启动效率低。

### 9、Redis 常见性能问题和解决方案： 

- Master 最好不要写内存快照，如果 Master 写内存快照，save 命令调度 rdbSave 函数，会阻塞主线程的工作，当快照比较大时对性能影响是非常大 的，会间断性暂停服务 
- 如果数据比较重要，某个 Slave 开启 AOF 备份数据，策略设置为每秒同步一 
- 为了主从复制的速度和连接的稳定性，Master 和 Slave 最好在同一个局域网 
- 尽量避免在压力很大的主库上增加从 
- **主从复制不要用图状结构，用单向链表结构更为稳定**，即：Master <- Slave1<- Slave2 <- Slave3…这样的结构方便解决单点故障问题，实现 Slave  对 Master 的替换。如果 Master 挂了，可以立刻启用 Slave1 做 Master，其 他不变。

### 10、Redis 过期键的删除策略？ 

1. **定时删除**:在**设置键的过期时间的同时，创建一个定时器 timer**. 让定时器在键的过期时间来临时，立即执行对键的删除操作。
2. **惰性删除**:放任键过期不管，但是**每次从键空间中获取键时，都检查取得的键是否过期，如果过期的话，就删除该键**;如果没有过期，就返回该键。
3. **定期删除**:每隔一段时间程序就对数据库进行一次检查，删除里面的过期 键。至于要删除多少过期键，以及要检查多少个数据库，则由算法决定。

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

Redis **为了达到最快的读写速度将数据都读到内存中**，**并通过异步的方式将数据写入磁盘**。所以 Redis 具有快速和数据持久化的特征。如果不将数据放 在 内存中，磁盘 I/O 速度为严重影响 Redis 的性能。在内存越来越便宜的今 天， Redis 将会越来越受欢迎。如果设置了最大使用的内存，则数据已有记录 数达 到内存限值后不能继续插入新值。

### 13、Redis 的同步机制了解么？

 Redis 可以使用主从同步，从从同步。第一次同步时，主节点做一次 bgsave， 并同时将后续修改操作记录到内存 buffer，待完成后将 rdb 文件全量同步到复制节点，复制节点接受完成后将 rdb 镜像加载到内存。加载完成 后，再通 知主节点将期间修改的操作记录同步到复制节点进行重放就完成了同步过程。

### 14、Pipeline 有什么好处，为什么要用 Pipeline？ 

可以将多次 IO 往返的时间缩减为一次，前提是 pipeline 执行的指令之 间没 有因果相关性。使用 Redis-benchmark 进行压测的时候可以发现影响 Redis  的 QPS 峰值的一个重要因素是 pipeline 批次指令的数目。

### 15、是否使用过 Redis 集群，集群的原理是什么？

- Redis Sentinal 着眼于高可用，在 master 宕机时会自动将 slave 提升 为 master，继续提供服务。
- Redis Cluster 着眼于扩展性，在单个 Redis 内存不足时，使用 Cluster  进行分片存储。

### 16、Redis 集群方案什么情况下会导致整个集群不可用？ 

有 A，B，C 三个节点的集群,在没有复制模型的情况下,如果节点 B 失败了， 那么整个集群就会以为缺少 5501-11000 这个范围的槽而不可用。

### 19、Redis 如何设置密码及验证密码？ 

设置密码：config set requirepass 123456 

授权密码：auth 123456

### 20、说说 Redis 哈希槽的概念？

 Redis 集群没有使用一致性 hash,而是引入了哈希槽的概念，Redis 集群 有 16384 个哈希槽，每个 key 通过 CRC16 校验后对 16384 取模来决定放置 哪 个槽，集群的每个节点负责一部分 hash 槽。 

### 21、Redis 集群的主从复制模型是怎样的？ 

为了使在部分节点失败或者大部分节点无法通信的情况下集群仍然可用， 所以 集群使用了主从复制模型,每个节点都会有 N-1 个复制品。 

### 22、Redis 集群会有写操作丢失吗？为什么？ 

Redis 并不能保证数据的强一致性，这意味这在实际中集群在特定的条件 下可 能会丢失写操作。 

### 23、Redis 集群之间是如何复制的？ 

异步复制 

### 24、Redis 集群最大节点个数是多少？ 

16384 个。 

### 25、Redis 集群如何选择数据库？

 Redis 集群目前无法做数据库选择，默认在 0 数据库。

### 26、怎么测试 Redis 的连通性 

使用 ping 命令。 

### 27、怎么理解 Redis 事务？ 

- 事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执 行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。 
- 事务是一个原子操作：事务中的命令要么全部被执行，要么全部都不执 行。 

### 28、Redis 事务相关的命令有哪几个？ 

MULTI、EXEC、DISCARD、WATCH 

### 29、Redis key 的过期时间和永久有效分别怎么设置？

 EXPIRE 和 PERSIST 命令。

### 30、Redis 如何做内存优化？ 

尽可能使用散列表（hashes），散列表（是说散列表里面存储的数少）使用的 内存非常小，所以你应该尽可能的将你的数据模型抽象到一个散列表里面。比 如你的 web 系统中有一个用户对象，不要为这个用户的名称，姓氏，邮箱，密 码设置单独的 key,而是应该把这个用户的所有信息存储到一张散列表里面。

31、Redis 回收进程如何工作的？ 

一个客户端运行了新的命令，添加了新的数据。**Redis 检查内存使用情况， 如果大于 maxmemory 的限制, 则根据设定好的策略进行回收**。一个新的命令被 执 行，等等。所以我们不**断地穿越内存限制的边界，通过不断达到边界然后不断地回收回到边界以下**。如果一个命令的结果导致大量内存被使用（例如很大 的 集合的交集保存到一个新的键），不用多久内存限制就会被这个内存使用量超越。

### 32、都有哪些办法可以降低 Redis 的内存使用情况呢？

如果你使用的是 32 位的 Redis 实例，可以好好利用 Hash,list,sorted  set,set 等集合类型数据，因为通常情况下很多小的 Key-Value 可以用更紧凑 的方式存放到一起。 

### 33、Redis 的内存用完了会发生什么？ 

如果达到设置的上限，**Redis 的写命令会返回错误信息**（但是读命令还可以正常返回。）或者你可以将 Redis 当缓存来使用配置淘汰机制，当 Redis 达到 内存上限时会冲刷掉旧的内容。 

### 34、一个 Redis 实例最多能存放多少的 keys？ List、Set、 Sorted Set 他们最多能存放多少元素？

 理论上 **Redis 可以处理多达 232 的 keys**，并且在实际中进行了测试，每个实例至少存放了 2 亿 5 千万的 keys。我们正在测试一些较大的值。任何 list、 set、和 sorted set 都可以放 232 个元素。换句话说，Redis 的存储极限是系统中的可用内存值。

### 36、Redis 最适合的场景？ 

**会话缓存（Session Cache）** 

最常用的一种使用 Redis 的情景是会话缓存（session cache）。用 Redis 缓 存会话比其他存储（如 Memcached）的优势在于：Redis 提供持久化。当维护 一个不是严格要求一致性的缓存时，如果用户的购物车信息全部丢失，大部分 人都会不高兴的，现在，他们还会这样吗？ 幸运的是，随着 Redis 这些年的 改进，很容易找到怎么恰当的使用 Redis 来缓存会话的文档。甚至广为人知的 商业平台 Magento 也提供 Redis 的插件。 

**全页缓存（FPC）** 

除基本的会话 token 之外，Redis 还提供很简便的 FPC 平台。回到一致性问 题，即使重启了 Redis 实例，因为有磁盘的持久化，用户也不会看到页面加载 速度的下降，这是一个极大改进，类似 PHP 本地 FPC。 再次以 Magento 为 例，Magento 提供一个插件来使用 Redis 作为全页缓存后端。 此外，对 WordPress 的用户来说，Pantheon 有一个非常好的插件 wp-redis，这个插件 能帮助你以最快速度加载你曾浏览过的页面。

**队列**

Redis 在内存存储引擎领域的一大优点是提供 list 和 set 操作，这使得 Redis 能作为一个很好的消息队列平台来使用。Redis 作为队列使用的操作， 就类似于本地程序语言（如 Python）对 list 的 push/pop 操作。 如果你快 速的在 Google 中搜索“Redis queues”，你马上就能找到大量的开源项目， 这些项目的目的就是利用 Redis 创建非常好的后端工具，以满足各种队列需 求。例如，Celery 有一个后台就是使用 Redis 作为 broker，你可以从这里去 查看。 

**排行榜/计数器** 

Redis 在内存中对数字进行递增或递减的操作实现的非常好。集合（Set）和有 序集合（Sorted Set）也使得我们在执行这些操作的时候变的非常简单，Redis  只是正好提供了这两种数据结构。所以，我们要从排序集合中获取到排名最靠 前的 10 个用户–我们称之为“user_scores”，我们只需要像下面一样执行即 可： 当然，这是假定你是根据你用户的分数做递增的排序。如果你想返回用户 及用户的分数，你需要这样执行： ZRANGE user_scores 0 10 WITHSCORES  Agora Games 就是一个很好的例子，用 Ruby 实现的，它的排行榜就是使用 Redis 来存储数据的，你可以在这里看到。 

**发布/订阅** 

最后（但肯定不是最不重要的）是 Redis 的发布/订阅功能。发布/订阅的使用 场景确实非常多。我已看见人们在社交网络连接中使用，还可作为基于发布/订 阅的脚本触发器，甚至用 Redis 的发布/订阅功能来建立聊天系统！

### 37、假如 Redis 里面有 1 亿个 key，其中有 10w 个 key 是以某 个固定的已知的前缀开头的，如果将它们全部找出来？ 

**使用 keys 指令可以扫出指定模式的 key 列表**。 对方接着追问：如果这个 Redis 正在给线上的业务提供服务，那使用 keys 指令会有什么问题？ 

这个时候你要回答 Redis 关键的一个特性：**Redis 的单线程的**。**keys 指令会 导致线程阻塞一段时间，线上服务会停顿**，直到指令执行完毕，服务才能恢 复。**这个时候可以使用 scan 指令**，scan 指令可以无阻塞的提取出指定模式的key 列表，但是会有一定的重复概率，在客户端做一次去重就可以了，但是整 体所花费的时间会比直接用 keys 指令长

### 38、如果有大量的 key 需要设置同一时间过期，一般需要注意什 么？ 

如果大量的 key 过期时间设置的过于集中，**到过期的那个时间点，Redis 可能 会出现短暂的卡顿现象**。一般需要在时间上**加一个随机值**，使得过期时间分散 一些。

### 39、使用过 Redis 做异步队列么，你是怎么用的？ 

一般**使用 list 结构作为队列**，rpush 生产消息，lpop 消费消息。当 lpop 没有消息的时候，要**适当 sleep 一会再重试**。 

**如果对方追问可不可以不 用 sleep 呢？**  

list 还有个指令叫 **blpop**，在没有消息的时候，它会阻塞住直 到消息到来。 

**如果对方追问能不能生产一次消费多次呢？** 

**使用 pub/sub 主题订阅者模式**，可以实现 1:N 的消息队列。 

**如果对方追问 pub/sub 有什么缺点？**  

**在消费者下线的情况下，生产的消息会丢失**，得使用专业的消息队列如 RabbitMQ 等。

**如果对方追问 Redis 如何实现延时队列？**  

我估计现在你很想把面试官一棒打死如果你手上有一根棒球棍的话，怎么问的 这么详细。但是你很克制，然后神态自若的回答道：**使用 sortedset，拿时间戳作为 score，消息内容作为 key 调用 zadd 来生产消息，消费者用 zrangebyscore 指令获取 N 秒之前的数据轮询进行处理**。

### 40、使用过 Redis 分布式锁么，它是什么回事 

先拿 setnx 来争抢锁，抢到之后，再用 expire 给锁加一个过期时间防止锁忘 记了释放。 这时候对方会告诉你说你回答得不错，然后接着问如果在 setnx 之后执行 expire 之前进程意外 crash 或者要重启维护了，那会怎么样？这时候你要给 予惊讶的反馈：唉，是喔，这个锁就永远得不到释放了。紧接着你需要抓一抓 自己得脑袋，故作思考片刻，好像接下来的结果是你主动思考出来的，然后回 答：我记得 set 指令有非常复杂的参数，这个应该是可以同时把 setnx 和 expire 合成一条指令来用的！对方这时会显露笑容，心里开始默念：摁，这小 子还不错。