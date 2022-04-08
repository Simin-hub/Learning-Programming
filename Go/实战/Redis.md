# Redis



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