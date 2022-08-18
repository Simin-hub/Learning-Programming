# etcd

[中文文档](https://doczhcn.gitbook.io/etcd/)、[参考](https://zhuanlan.zhihu.com/p/405811320)

[服务注册发现原理](https://github.com/Simin-hub/Learning-Programming/blob/main/Go/%E5%AE%9E%E6%88%98/%E6%9C%8D%E5%8A%A1%E6%B3%A8%E5%86%8C%E4%B8%AD%E5%BF%83.md)

etcd作为一个受到ZooKeeper与doozer启发而催生的项目，除了拥有与之类似的功能外，更专注于以下四点。

1. 简单：**基于HTTP+JSON的API**让你用curl就可以轻松使用。
2. 安全：可选SSL客户认证机制。
3. 快速：每个实例每秒支持一千次写操作。
4. 可信：使用Raft算法充分实现了分布式。

分布式系统中的数据分为控制数据和应用数据。etcd的使用场景默认处理的数据都是控制数据，对于应用数据，只推荐数据量很小，但是更新访问频繁的情况。

应用场景有如下几类：

场景一：服务发现（Service Discovery）

场景二：消息发布与订阅 

场景三：负载均衡场景四：分布式通知与协调 

场景五：分布式锁、分布式队列 

场景六：集群监控与Leader竞选

举个最简单的例子，如果你需要一个分布式存储仓库来存储配置信息，并且希望这个仓库读写速度快、支持高可用、部署简单、支持http接口，那么就可以使用etcd。目前，cloudfoundry使用etcd作为hm9000的应用状态信息存储，kubernetes用etcd来存储docker集群的配置信息等。

## 概述

### 1. ETCD是什么

**这里有一个ETCD的相关视频讲解：**[分布式注册服务中心etcd](https://www.bilibili.com/video/BV1ny4y1G7o7/)

ETCD是**用于共享配置和服务发现的分布式，一致性的KV存储系统**。ETCD是CoreOS公司发起的一个开源项目，授权协议为Apache。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/v2-438b2c4de1230e391107b5b7d9ac980a_b.jpg)

提供配置共享和服务发现的系统比较多，其中最为大家熟知的是[Zookeeper]（后文简称ZK），而ETCD可以算得上是后起之秀了。在项目实现，一致性协议易理解性，运维，安全等多个维度上，ETCD相比Zookeeper都占据优势。

### 2. ETCD vs ZK

本文选取ZK作为典型代表与ETCD进行比较，而不考虑[Consul]项目作为比较对象，原因为Consul的可靠性和稳定性还需要时间来验证（项目发起方自身服务并未使用Consul, 自己都不用)。

- 一致性协议： ETCD使用[Raft]协议， ZK使用ZAB（类PAXOS协议），前者容易理解，方便工程实现；
- 运维方面：ETCD方便运维，ZK难以运维；
- 项目活跃度：ETCD社区与开发活跃，ZK已经快死了；
- API：ETCD提供HTTP+JSON, gRPC接口，跨平台跨语言，ZK需要使用其客户端；
- 访问安全方面：ETCD支持HTTPS访问，ZK在这方面缺失；

### 3. ETCD的使用场景

和ZK类似，ETCD有很多使用场景，包括：

- 配置管理
- 服务注册于发现
- 选主
- 应用调度
- 分布式队列
- 分布式锁

### 4. ETCD读写性能

按照官网给出的[Benchmark], 在2CPU，1.8G内存，SSD磁盘这样的配置下，单节点的写性能可以达到16K QPS, 而先写后读也能达到12K QPS。这个性能还是相当可观的。

### 5. ETCD工作原理

#### etcd 的核心术语

[参考](https://blog.51cto.com/u_15301988/3085390)

Raft：etcd 所采用的**保证分布式系统数据强一致性的算法**。

Node：一个 Raft 状态机实例。

Member：一个 etcd 实例，它管理着一个 Node，并且可以为客户端请求提供服务。

Cluster：由多个 Member 构成可以协同工作的 etcd 集群。

Peer：对同一个 etcd 集群中另外一个 Member 的称呼。

Client：向 etcd 集群发送 HTTP 请求的客户端。

WAL：**预写式日志，etcd 用于持久化存储的日志格式**。

Snapshot：**etcd 防止 WAL 文件过多而设置的快照，存储 etcd 数据状态**。

Entry：Raft 算法中的日志的一个条目。

Proxy：etcd 的一种模式，为 etcd 集群提供反向代理服务。

Leader：Raft 算法中通过竞选而产生的处理所有数据提交的节点。

Follower：Raft 算法中竞选失败的节点作为从属节点，为算法提供强一致性保证。

Candidate：当 Follower 超过一定时间接收不到 Leader 的心跳时（认为 Leader 发生了故障）转变为 Candidate 开始竞选。

Term：某个节点成为 Leader 到下一次竞选时间，称为一个 Term。

Vote：选举时的一张投票。

Index：数据项编号，Raft 中通过 Term 和 Index 来定位数据。

Commit：一个提交，持久化数据写入到日志中。

Propose：一个提议，请求大部分 Node 同意数据写入。



ETCD**使用Raft协议来维护集群内各个节点状态的一致性**。简单说，ETCD集群是一个分布式系统，由多个节点相互通信构成整体对外服务，每个节点都存储了完整的数据，并且通过Raft协议保证每个节点维护的数据是一致的。

![img](https://pic1.zhimg.com/v2-b253d80ffd29f87eaba4e22b4521bbf4_r.jpg)

如图所示，**每个ETCD节点都维护了一个状态机，并且，任意时刻至多存在一个有效的主节点**。主节点处理所有来自客户端写操作，通过Raft协议保证写操作对状态机的改动会可靠的同步到其他节点。

ETCD工作原理核心部分在于Raft协议。本节接下来将简要介绍Raft协议，具体细节请参考其[论文]。

Raft协议正如论文所述，确实方便理解。主要分为三个部分：选主，日志复制，安全性。

[参考](https://blog.51cto.com/u_15301988/3085390)

#### etcd 的 K/V 存储

etcd Server **采用树形的结构来组织储存数据**，类似 Linux 的文件系统，也有目录和文件的分层结构，不过一般被称为 nodes。

例如：用户指定的 key 可以为单独的名字，如：testkey，此时 key testkey 实际上存放在根目录 “/” 下面。也可以为指定目录结构，如：/testdir/testkey，则将创建相应的目录结构。

```
etcdctl set /testdir/testkey "Hello world"
```

#### etcd 的软件架构

- 
  HTTP Server：**接受客户端发出的 API 请求以及其它 etcd 节点的同步与心跳信息请求**。

- Store：用**于处理 etcd 支持的各类功能的事务**，包括数据索引、节点状态变更、监控与反馈、事件处理与执行等等，**是 etcd 对用户提供的大多数 API 功能的具体实现**。

- Raft：强一致性算法的具体实现，是 etcd 的核心算法。

- WAL（Write Ahead Log，预写式日志）：是 etcd 的数据存储方式，etcd 会**在内存中储存所有数据的状态以及节点的索引**，此外，etcd 还会通过 WAL 进行持久化存储。WAL 中，所有的数据提交前都会事先记录日志。

  - Snapshot 是为了防止数据过多而进行的状态快照；

  - Entry 表示存储的具体日志内容。

通常，一个用户的请求发送过来，会经由 HTTP Server 转发给 Store 进行具体的事务处理，如果涉及到节点数据的修改，则交给 Raft 模块进行状态的变更、日志的记录，然后再同步给别的 etcd 节点以确认数据提交，最后进行数据的提交，再次同步。

<img src="https://raw.githubusercontent.com/Simin-hub/Picture/master/img/7ada539096fd8452708910c8fafafe3e.png" alt="etcd — 架构原理_Kubernetes 云原生_02" style="zoom:50%;" />

#### 5.1 选主

**Raft协议是用于维护一组服务节点数据一致性的协议**。这一组服务节点构成一个集群，并且有一个主节点来对外提供服务。当集群初始化，或者主节点挂掉后，面临一个选主问题。集群中每个节点，任意时刻处于Leader, Follower, Candidate这三个角色之一。选举特点如下：

- 当集群初始化时候，每个节点都是Follower角色；
- 集群中存在至多1个有效的主节点，通过心跳与其他节点同步数据；
- **当Follower在一定时间内没有收到来自主节点的心跳，会将自己角色改变为Candidate，并发起一次选主投票**；当收到包括自己在内**超过半数节点赞成后，选举成功**；当收到票数不足半数选举失败，或者选举超时。若本轮未选出主节点，将进行下一轮选举（出现这种情况，是由于多个节点同时选举，所有节点均为获得过半选票）。
- Candidate节点收到来自主节点的信息后，会立即终止选举过程，进入Follower角色。为了避免陷入选主失败循环，每个节点未收到心跳发起选举的时间是一定范围内的随机值，这样能够避免2个节点同时发起选主。

#### 5.2 日志复制

所谓日志复制，是指**主节点将每次操作形成日志条目，并持久化到本地磁盘，然后通过网络IO发送给其他节点**。其他节点根据日志的逻辑时钟(TERM)和日志编号(INDEX)来判断是否将该日志记录持久化到本地。当主节点收到包括自己在内超过半数节点成功返回，那么认为该日志是**可提交的(committed），并将日志输入到状态机**，将结果返回给客户端。

这里需要注意的是，**每次选主都会形成一个唯一的TERM编号**，相当于逻辑时钟。**每一条日志都有全局唯一的编号**。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/v2-c1dc8b61c482eb26124c2c7cc0fd290f_r.jpg)

主节点通过网络IO向其他节点追加日志。若某节点收到日志追加的消息，**首先判断该日志的TERM是否过期，以及该日志条目的INDEX是否比当前以及提交的日志的INDEX跟早**。若已过期，或者比提交的日志更早，那么就拒绝追加，并返回该节点当前的已提交的日志的编号。否则，将日志追加，并返回成功。

当主节点收到其他节点关于日志追加的回复后，若发现有拒绝，则根据该节点返回的已提交日志编号，发生其编号下一条日志。

主节点像其他节点同步日志，还作了拥塞控制。具体地说，主节点发现日志复制的目标节点拒绝了某次日志追加消息，将进入日志探测阶段，一条一条发送日志，直到目标节点接受日志，然后进入快速复制阶段，可进行批量日志追加。

按照日志复制的逻辑，我们可以看到，集群中慢节点不影响整个集群的性能。另外一个特点是，**数据只从主节点复制到Follower节点**，这样大大简化了逻辑流程。

#### 5.3 安全性

截止此刻，**选主以及日志复制并不能保证节点间数据一致**。试想，当一个某个节点挂掉了，一段时间后再次重启，并当选为主节点。而在其挂掉这段时间内，集群若有超过半数节点存活，集群会正常工作，那么会有日志提交。这些提交的日志无法传递给挂掉的节点。当挂掉的节点再次当选主节点，它将缺失部分已提交的日志。在这样场景下，按Raft协议，它将自己日志复制给其他节点，会将集群已经提交的日志给覆盖掉。

这显然是不可接受的。

其他协议解决这个问题的办法是，**新当选的主节点会询问其他节点，和自己数据对比，确定出集群已提交数据，然后将缺失的数据同步过来**。这个方案有明显缺陷，增加了集群恢复服务的时间（集群在选举阶段不可服务），并且增加了协议的复杂度。

Raft解决的办法是，**在选主逻辑中，对能够成为主的节点加以限制，确保选出的节点已定包含了集群已经提交的所有日志**。如果新选出的主节点已经包含了集群所有提交的日志，那就不需要从和其他节点比对数据了。简化了流程，缩短了集群恢复服务的时间。

这里存在一个问题，加以这样限制之后，还能否选出主呢？答案是：只要仍然有超过半数节点存活，这样的主一定能够选出。因为已经提交的日志必然被集群中超过半数节点持久化，显然前一个主节点提交的最后一条日志也被集群中大部分节点持久化。当主节点挂掉后，集群中仍有大部分节点存活，那这存活的节点中一定存在一个节点包含了已经提交的日志了。

至此，关于Raft协议的简介就全部结束了。

### 6. ETCD使用案例

据公开资料显示，至少有CoreOS, Google Kubernetes, Cloud Foundry, 以及在Github上超过500个项目在使用ETCD。

### 7. ETCD接口

ETCD提供HTTP协议，在最新版本中支持Google gRPC方式访问。具体支持接口情况如下：

- ETCD是一个高可靠的KV存储系统，支持PUT/GET/DELETE接口；
- 为了支持服务注册与发现，支持WATCH接口（通过http long poll实现）；
- 支持KEY持有TTL属性；
- CAS（compare and swap)操作;
- 支持多key的事务操作；
- 支持目录操作

## ETCD系列之二：部署集群

### 1. 概述

想必很多人都知道ZooKeeper，通常用作配置共享和服务发现。和它类似，ETCD算是一个非常优秀的后起之秀了。本文重点不在描述他们之间的不同点。首先，看看其官网关于ETCD的描述1:

> A distributed, reliable key-value store for the most critical data of a distributed system.

在云计算大行其道的今天，ETCD有很多典型的使用场景。常言道，熟悉一个系统先从部署开始。本文接下来将描述，如何部署ETCD集群。

安装官网说明文档，提供了3种集群启动方式，实际上按照其实现原理分为2类：

- 通过静态配置方式启动
- 通过服务发现方式启动

在部署集群之前，我们需要考虑集群需要配置多少个节点。这是一个重要的考量，不得忽略。

### 2. 集群节点数量与网络分割

ETCD使用RAFT协议保证各个节点之间的状态一致。**根据RAFT算法原理，节点数目越多，会降低集群的写性能**。这是因为每一次写操作，需要集群中大多数节点将日志落盘成功后，Leader节点才能将修改内部状态机，并返回将结果返回给客户端。

也就是说在等同配置下，节点数越少，集群性能越好。显然，只部署1个节点是没什么意义的。通常，按照需求将集群节点部署为3，5，7，9个节点。

这里能选择偶数个节点吗？ 最好不要这样。原因有二：

- 偶数个节点集群不可用风险更高，表现在选主过程中，有较大概率或等额选票，从而触发下一轮选举。
- 偶数个节点集群在某些网络分割的场景下无法正常工作。试想，当网络分割发生后，将集群节点对半分割开。此时集群将无法工作。按照RAFT协议，此时集群写操作无法使得大多数节点同意，从而导致写失败，集群无法正常工作。

当网络分割后，ETCD集群如何处理的呢?

- 当集群的Leader在多数节点这一侧时，集群仍可以正常工作。少数节点那一侧无法收到Leader心跳，也无法完成选举。
- 当集群的Leader在少数节点这一侧时，集群仍可以正常工作，多数派的节点能够选出新的Leader, 集群服务正常进行。

当网络分割恢复后，少数派的节点会接受集群Leader的日志，直到和其他节点状态一致。

### 3. ETCD参数说明

这里只列举一些重要的参数，以及其用途。

- —data-dir 指定节点的数据存储目录，这些数据包括节点ID，集群ID，集群初始化配置，Snapshot文件，若未指定—wal-dir，还会存储WAL文件；
- —wal-dir 指定节点的was文件的存储目录，若指定了该参数，wal文件会和其他数据文件分开存储。
- —name 节点名称
- —initial-advertise-peer-urls 告知集群其他节点url.
- — listen-peer-urls 监听URL，用于与其他节点通讯
- — advertise-client-urls 告知客户端url, 也就是服务的url
- — initial-cluster-token 集群的ID
- — initial-cluster 集群中所有节点

### 4. 通过静态配置方式启动ETCD集群

按照官网中的文档，即可完成集群启动。这里略。

### 5. 通过服务发现方式启动ETCD集群

ETCD还提供了另外一种启动方式，即通过服务发现的方式启动。这种启动方式，依赖另外一个ETCD集群，在该集群中创建一个目录，并在该目录中创建一个_config的子目录，并且在该子目录中增加一个size节点，指定集群的节点数目。

在这种情况下，将该目录在ETCD中的URL作为节点的启动参数，即可完成集群启动。使用 [--discovery]( https://myetcd.local/v2/keys/discovery/6c007a14875d53d9bf0ef5a6fc0257c817f0fb83 ) 配置项取代静态配置方式中的--initial-cluster 和inital-cluster-state参数。其中[该设置](https://myetcd.local/v2/keys/discovery/6c007a14875d53d9bf0ef5a6fc0257c817f0fb83)是在依赖etcd中创建好的目录url。

### 6. 节点迁移

在生产环境中，不可避免遇到机器硬件故障。当遇到硬件故障发生的时候，我们需要快速恢复节点。ETCD集群可以做到在不丢失数据的，并且不改变节点ID的情况下，迁移节点。
具体办法是：

- 停止待迁移节点上的etc进程；
- 将数据目录打包复制到新的节点；
- 更新该节点对应集群中peer url，让其指向新的节点；
- 使用相同的配置，在新的节点上启动etcd进程；

[本文记录了ETCD集群启动的一些注意事项，希望对你有帮助。](https://yq.aliyun.com/articles/29897?spm=5176.100239.blogcont11035.15.7bihps)

## 安装使用

静态就是在配置服务之前已经知道了节点的地址和集群的大小

### etcd 部署

#### 源码编译安装

```clean
############################
# Build the latest version
############################

# 1.下载项目并编译
$ git clone https://github.com/etcd-io/etcd.git && cd etcd
$ ./build
To build a vendored etcd from the master branch via go get:

# 2.设置GOPATH环境变量
$ export GOPATH='/Users/example/go'
$ go get -v go.etcd.io/etcd
$ go get -v go.etcd.io/etcd/etcdctl

# 3.启动服务
$ ./bin/etcd
$ $GOPATH/bin/etcd

# 4.简单使用
$ ./bin/etcdctl put foo bar
OK
```

![图片](https://segmentfault.com/img/remote/1460000039964036)

#### 部署单机单服务(静态)

```clean
##################################
# Running etcd in standalone mode
##################################

# 1.设置启动的Node地址
$ export NODE1='172.16.176.52'

# 2.创建一个逻辑存储
$ docker volume create --name etcd-data

# 3.启动etcd服务
# 正式的ectd端口是2379用于客户端连接，而2380用于伙伴通讯
# --data-dir: 到数据目录的路径
# --initial-advertise-peer-urls: 集群中节点间通讯的URL地址
# --listen-peer-urls: 集群中节点间通讯的URL地址
# --advertise-client-urls: 客户端监听的URL地址
# --listen-client-urls: 客户端监听的URL地址
# --initial-cluster: 启动初始化集群配置
$ docker run -p 2379:2379 -p 2380:2380 --name etcd \
    --volume=etcd-data:/etcd-data \
    quay.io/coreos/etcd:latest \
    /usr/local/bin/etcd \
    --data-dir=/etcd-data --name node1 \
    --initial-advertise-peer-urls http://${NODE1}:2380 \
    --listen-peer-urls http://0.0.0.0:2380 \
    --advertise-client-urls http://${NODE1}:2379 \
    --listen-client-urls http://0.0.0.0:2379 \
    --initial-cluster node1=http://${NODE1}:2380

# 4.列出现在集群中的服务状态
$ etcdctl --endpoints=http://${NODE1}:2379 member list
```

![图片](https://segmentfault.com/img/remote/1460000039964037)

#### 部署分布式集群服务(静态)

```clean
################################
# Running a 3 node etcd cluster
################################

# node1
docker run -p 2379:2379 -p 2380:2380 --name etcd-node-1 \
  --volume=/var/lib/etcd:/etcd-data \
  quay.io/coreos/etcd:latest \
  /usr/local/bin/etcd \
  --data-dir=/etcd-data \
  --initial-advertise-peer-urls "http://10.20.30.1:2380" \
  --listen-peer-urls "http://0.0.0.0:2380" \
  --advertise-client-urls "http://10.20.30.1:2379" \
  --listen-client-urls "http://0.0.0.0:2379" \
  --initial-cluster "etcd-node-1=http://10.20.30.1:2380, etcd-node-2=http://10.20.30.2:2380, etcd-node-3=http://10.20.30.3:2380" \
  --initial-cluster-state "new" \
  --initial-cluster-token "my-etcd-token"

# node2
docker run -p 2379:2379 -p 2380:2380 --name etcd-node-2 \
  --volume=/var/lib/etcd:/etcd-data \
  quay.io/coreos/etcd:latest \
  /usr/local/bin/etcd \
  --data-dir=/etcd-data \
  --initial-advertise-peer-urls "http://10.20.30.2:2380" \
  --listen-peer-urls "http://0.0.0.0:2380" \
  --advertise-client-urls "http://10.20.30.2:2379" \
  --listen-client-urls "http://0.0.0.0:2379" \
  --initial-cluster "etcd-node-1=http://10.20.30.1:2380, etcd-node-2=http://10.20.30.2:2380, etcd-node-3=http://10.20.30.3:2380" \
  --initial-cluster-state "new" \
  --initial-cluster-token "my-etcd-token"

# node3
docker run -p 2379:2379 -p 2380:2380 --name etcd-node-3 \
  --volume=/var/lib/etcd:/etcd-data \
  quay.io/coreos/etcd:latest \
  /usr/local/bin/etcd \
  --data-dir=/etcd-data \
  --initial-advertise-peer-urls "http://10.20.30.3:2380" \
  --listen-peer-urls "http://0.0.0.0:2380" \
  --advertise-client-urls "http://10.20.30.3:2379" \
  --listen-client-urls "http://0.0.0.0:2379" \
  --initial-cluster "etcd-node-1=http://10.20.30.1:2380, etcd-node-2=http://10.20.30.2:2380, etcd-node-3=http://10.20.30.3:2380" \
  --initial-cluster-state "new" \
  --initial-cluster-token "my-etcd-token"

# run etcdctl using API version 3
docker exec etcd /bin/sh -c "export ETCDCTL_API=3 && /usr/local/bin/etcdctl put foo bar"
```

#### 部署分布式集群服务

```nestedtext
# 编辑docker-compose.yml文件
version: "3.6"

services:
  node1:
    image: quay.io/coreos/etcd
    volumes:
      - node1-data:/etcd-data
    expose:
      - 2379
      - 2380
    networks:
      cluster_net:
        ipv4_address: 172.16.238.100
    environment:
      - ETCDCTL_API=3
    command:
      - /usr/local/bin/etcd
      - --data-dir=/etcd-data
      - --name
      - node1
      - --initial-advertise-peer-urls
      - http://172.16.238.100:2380
      - --listen-peer-urls
      - http://0.0.0.0:2380
      - --advertise-client-urls
      - http://172.16.238.100:2379
      - --listen-client-urls
      - http://0.0.0.0:2379
      - --initial-cluster
      - node1=http://172.16.238.100:2380,node2=http://172.16.238.101:2380,node3=http://172.16.238.102:2380
      - --initial-cluster-state
      - new
      - --initial-cluster-token
      - docker-etcd

  node2:
    image: quay.io/coreos/etcd
    volumes:
      - node2-data:/etcd-data
    networks:
      cluster_net:
        ipv4_address: 172.16.238.101
    environment:
      - ETCDCTL_API=3
    expose:
      - 2379
      - 2380
    command:
      - /usr/local/bin/etcd
      - --data-dir=/etcd-data
      - --name
      - node2
      - --initial-advertise-peer-urls
      - http://172.16.238.101:2380
      - --listen-peer-urls
      - http://0.0.0.0:2380
      - --advertise-client-urls
      - http://172.16.238.101:2379
      - --listen-client-urls
      - http://0.0.0.0:2379
      - --initial-cluster
      - node1=http://172.16.238.100:2380,node2=http://172.16.238.101:2380,node3=http://172.16.238.102:2380
      - --initial-cluster-state
      - new
      - --initial-cluster-token
      - docker-etcd

  node3:
    image: quay.io/coreos/etcd
    volumes:
      - node3-data:/etcd-data
    networks:
      cluster_net:
        ipv4_address: 172.16.238.102
    environment:
      - ETCDCTL_API=3
    expose:
      - 2379
      - 2380
    command:
      - /usr/local/bin/etcd
      - --data-dir=/etcd-data
      - --name
      - node3
      - --initial-advertise-peer-urls
      - http://172.16.238.102:2380
      - --listen-peer-urls
      - http://0.0.0.0:2380
      - --advertise-client-urls
      - http://172.16.238.102:2379
      - --listen-client-urls
      - http://0.0.0.0:2379
      - --initial-cluster
      - node1=http://172.16.238.100:2380,node2=http://172.16.238.101:2380,node3=http://172.16.238.102:2380
      - --initial-cluster-state
      - new
      - --initial-cluster-token
      - docker-etcd

volumes:
  node1-data:
  node2-data:
  node3-data:

networks:
  cluster_net:
    driver: bridge
    ipam:
      driver: default
      config:
      -
        subnet: 172.16.238.0/24
# 使用启动集群
docker-compose up -d


# 之后使用如下命令登录到任一节点测试etcd集群
docker exec -it node1 bash

# etcdctl member list
422a74f03b622fef, started, node1, http://172.16.238.100:2380, http://172.16.238.100:2379
ed635d2a2dbef43d, started, node2, http://172.16.238.101:2380, http://172.16.238.101:2379
daf3fd52e3583ffe, started, node3, http://172.16.238.102:2380, http://172.16.238.102:2379
```

#### etcd 常用配置参数

```ldif
--name       #指定节点名称
--data-dir   #指定节点的数据存储目录，用于保存日志和快照
--addr       #公布的 IP 地址和端口；默认为 127.0.0.1:2379
--bind-addr   #用于客户端连接的监听地址；默认为–addr 配置
--peers       #集群成员逗号分隔的列表；例如 127.0.0.1:2380,127.0.0.1:2381
--peer-addr   #集群服务通讯的公布的 IP 地址；默认为 127.0.0.1:2380
-peer-bind-addr  #集群服务通讯的监听地址；默认为-peer-addr 配置
--wal-dir         #指定节点的 wal 文件的存储目录，若指定了该参数 wal 文件会和其他数据文件分开存储
--listen-client-urls #监听 URL；用于与客户端通讯
--listen-peer-urls   #监听 URL；用于与其他节点通讯
--initial-advertise-peer-urls  #告知集群其他节点 URL
--advertise-client-urls  #告知客户端 URL
--initial-cluster-token  #集群的 ID
--initial-cluster        #集群中所有节点
--initial-cluster-state new  #表示从无到有搭建 etcd 集群
--discovery-srv  #用于 DNS 动态服务发现，指定 DNS SRV 域名
--discovery      #用于 etcd 动态发现，指定 etcd 发现服务的 URL
```

### 数据存储

etcd 的数据存储有点像 PG 数据库的存储方式

etcd 目前支持 V2 和 V3 两个大版本，这两个版本在实现上有比较大的不同，一方面是对外提供接口的方式，另一方面就是底层的存储引擎，V2 版本的实例是一个纯内存的实现，所有的数据都没有存储在磁盘上，而 V3 版本的实例就支持了数据的持久化。

![图片](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/1460000039964038)

我们都知道 etcd 为我们提供了 key/value 的服务目录存储。

```applescript
# 设置键值对
$ etcdctl set name escape

# 获取方式
$ etcdctl get name
escape
```

使用 etcd 之后，我们会疑问数据都存储到的那里呢？数据默认会存放在 /var/lib/etcd/default/ 目录。我们会发现数据所在的目录，会被分为两个文件夹中，分别是 snap 和 wal目录。

- snap
- 存放快照数据，存储etcd的数据状态
- etcd防止WAL文件过多而设置的快照
- wal
- 存放预写式日志
- 最大的作用是记录了整个数据变化的全部历程
- 在etcd中，所有数据的修改在提交前都要先写入到WAL中

```gradle
# 目录结构
$ tree /var/lib/etcd/default/
default
└── member
    ├── snap
    │   ├── 0000000000000006-0000000000046ced.snap
    │   ├── 0000000000000006-00000000000493fe.snap
    │   ├── 0000000000000006-000000000004bb0f.snap
    │   ├── 0000000000000006-000000000004e220.snap
    │   └── 0000000000000006-0000000000050931.snap
    └── wal
        └── 0000000000000000-0000000000000000.wal
```

使用 WAL 进行数据的存储使得 etcd 拥有两个重要功能，那就是故障快速恢复和数据回滚/重做。

- 故障快速恢复就是当你的数据遭到破坏时，就可以通过执行所有 WAL 中记录的修改操作，快速从最原始的数据恢复到数据损坏前的状态。
- 数据回滚重做就是因为所有的修改操作都被记录在 WAL 中，需要回滚或重做，只需要方向或正向执行日志中的操作即可。

既然有了 WAL 实时存储了所有的变更，为什么还需要 snapshot 呢？随着使用量的增加，WAL 存储的数据会暴增。为了防止磁盘很快就爆满，etcd 默认每 10000 条记录做一次 snapshot 操作，经过 snapshot 以后的 WAL 文件就可以删除。而通过 API 可以查询的历史 etcd 操作默认为 1000 条。

首次启动时，etcd 会把启动的配置信息存储到 data-dir 参数指定的数据目录中。配置信息包括本地节点的ID、集群ID和初始时集群信息。用户需要避免 etcd 从一个过期的数据目录中重新启动，因为使用过期的数据目录启动的节点会与集群中的其他节点产生不一致。所以，为了最大化集群的安全性，一旦有任何数据损坏或丢失的可能性，你就应该把这个节点从集群中移除，然后加入一个不带数据目录的新节点。

### Go 使用

[参考](https://learnku.com/articles/46311#4929ee)

#### 安装

```
go get go.etcd.io/etcd/clientv3	
```

#### 创建 ETCD 连接

```
func mian(){
  var (
        client *clientv3.Client
        config clientv3.Config
    err error
    )

    config = clientv3.Config{
        // 这里的 Endpoints 是一个字符串数组切片，支持配合多个节点
        Endpoints:   []string{"127.0.0.1:2379"},
        // DialTimeout 连接超时设置
        DialTimeout: time.Duration(5) * time.Millisecond,
    }
    if client, err = clientv3.New(config); err != nil {
        return
    }
}
```

#### 数据操作

**KV 的 PUT 操作**

我们先尝试往 etcd 中 put 一个数据

```
// 第一个参数为上下文，第二个为 KEY， 第三个为 VALUE。也可以传第四个参数 option，比如：给这个KEY 加一个过期时间， 在KEY过期机制里面会有详细记录
putResponse, err := kv.Put(context.TODO(),"/testDir/User/user1","user info")
```

在 etcd 的各种操作中都会有对应的 Response， put 操作也不例外，同样返回 putResponse，这是一个对象，里面包含 Header, PrevKv。 PrevKv 提供了 Put 之前的这个 key 的 KV 值。Header 提供了 etcd 的 Revision 和其他 etcd 的信息和方法。

**KV 的 GET 操作**

我们依然使用刚刚拿到的 KV 实例进行操作

```
// 用法一: 第一个参数为上下文，第二个为 KEY，即可获取对应 Key 的 Value
getResponse, err := kv.Get(context.TODO(),"/testDir/User/user1")
// 用法二:  第一个参数为上下文，第二个为 KEY，第三个为可选参数  option，这个操作会返回前缀为"/testDir/User/" 下所有 Key 的 Value， 相当于获取一个列表
gutResponse, err := kv.Get(context.TODO(),"/testDir/User/", clientv3.WithPrefix())
```

同样，getResponse 也返回了很多信息，但最主要的是我们获取的 Key 的 Value，如果是使用第二种用法，会返回一个 KVs，是一个 KV 的数组切片。也包含了 Header, Count, More, More 是一个 bool 值，指示是否有更多键可以返回要求的范围。 Count 返回返回数据的数量

**KV 的 DELETE 操作**

```
// 用法一: 第一个参数为上下文，第二个为 KEY，即可获取对应 Key 的 Value
deleteResponse, err := kv.Delete(context.TODO(),"/testDir/User/user1")
// delete 也可以删除 "/testDir/User/" 下所有的key，类似get的操作，但第三个参数要传 WithPrefix（）
```

deleteResponse 返回 Header, Deleted,PrevKvs, Header 中包含信息与之前差不多，后面的 Deleted 是一个 int64 值，代表删除数量。PrevKvs 返回删除之前的 KV 值

#### ETCD 的租约机制

[参考](https://www.topgoer.com/%E6%95%B0%E6%8D%AE%E5%BA%93%E6%93%8D%E4%BD%9C/go%E6%93%8D%E4%BD%9Cetcd/%E6%93%8D%E4%BD%9Cetcd.html)

Etcd 中支持类似 Redis 中的 key 过期机制，使用这一功能配合其他 etcd 功能可以实现非常多的强大的功能，例如：分布式锁，服务发现等。

**获取租约实例**

> 要使用 etcd 的租约需要获得租约的 Lease 实例，我们先创建一个：

```
// 使用clientv3 创建一个lease的实例，传入 etcd 的 client 实例
lease := clientv3.New(client)
```

**申请租约**

获得实例申请一个租约，然后拿到租约 ID

```
// 申请租约使用lease实例的Grant方法，第一个参数还是上下文，第二个是TTL，过期时间
grant, err := lease.Grant(context.TODO(), 5)
// 拿到租约 ID， 拿到这个租约 ID 之后，就可以使用 kv 实例进行 put 操作，加上option参数的 WithLease， 就可以给一个 key 设置过期时间
leaseID := grant.ID
```

**续租**

既然是租约，当然是可以续约的，续租有两种方式，一种是**自动续租**，一种是手动续租。

自动续租

```
// 自动续租时需要传入要续租的租约 ID,lease的 KeepAlive 会启动一个协程执行自动续约，每次续约事件是我们第一次申请租约时设置的时间。
aliveChan, err := lease.KeepAlive(context.TODO(), leaseID)
```

> 自动续租返回一个续租的结果，是一个 channel，里面放着续租应答。下面是一个完整的续租代码

```
package main

import (
    "context"
    "fmt"
    "go.etcd.io/etcd/clientv3"
    "time"
)

func main() {
    cli, err := clientv3.New(clientv3.Config{
        Endpoints:   []string{"localhost:2379"},
        DialTimeout: 5 * time.Second,
    })
    if err != nil {
        fmt.Println(err.Error())
    }
    // Get a lease instance
    lease := clientv3.NewLease(cli)
    grant, err := lease.Grant(context.TODO(), 10)
    if err != nil {
        fmt.Println(err.Error())
        return
    }
    // lease id
    leaseID := grant.ID
    // 自动续租
    alive, err := lease.KeepAlive(context.TODO(), leaseID)
    if err != nil {
        fmt.Println(err)
        return
    }
    // 处理续租应答的协程
    go func() {
        for {
            select {
            case res := &lt;-alive:
                if res == nil {
                    fmt.Println("租约已经失效")
                    goto END
                } else {
                    fmt.Println("自动续租应答：", res.ID)
                }
            }
        }
    END:
    }()
    // Get KV of client
    kv := clientv3.NewKV(cli)
    put, err := kv.Put(context.TODO(), "/testDir/User/user1", "11", clientv3.WithLease(leaseID))
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Println("写入成功：", put.Header.Revision)

    for {
        get, err := kv.Get(context.TODO(),"/testDir/User/user1)
        if err != nil {
            fmt.Println(err)
            return
        }
        if get.Count == 0 {
            fmt.Println("租约过期了")
            break
        }
        fmt.Println("还没过期", get.Kvs)
        time.Sleep(time.Second * 2)
    }
}
```


手动续租就不去记录了。

#### ETCD 的 WATCHER

etcd 的 Watcher 可以监听指定的 key 的各种操作，我们先获取 watcher 实例

```
// 获取 watcher 实例的方法跟之前的 lease 和 kv 一样
watcher := clientv3.NewWatcher(cli)
```

拿到 watcher 实例之后，还需要指定从哪个 Revision 开始监听哪个 Key，所以还需要拿一个 Key 的 Revision。比如我们还是想监听 “/testDir/User/” 下面所有 Key 的变化，我们可以这样。

```
// 先 Get 我们需要监听的 "/testDir/User/" 的getResponse，
get, err = jm.Kv.Get(context.TODO(), "/testDir/User/", clientv3.WithPrefix())
// 从get操作的下一个Revision开始监听，这就是下一个Revision
watchStartRevision = get.Header.Revision + 1
// 然后使用watcher对象对这个目录进行监听
watchChan := watcher.Watch(context.TODO(), "/testDir/User/", clientv3.WithRev(watchStartRevision), clientv3.WithPrefix())
// 监听后返回一个 Channel 里面传回监听到的 "/testDir/User/" 下面的 key 的事件变化，下面是对事件变化的处理
for w := range watchChan {
        for _, event := range w.Events {
            switch event.Type {
            case mvccpb.PUT:
                fmt.Println("修改为:", string(event.Kv.Value), "revision:", event.Kv.CreateRevision)
            case mvccpb.DELETE:
                fmt.Println("删除:", event.Kv.ModRevision)
            }
        }
    }
```

### 分布式锁

#### 机制

etcd 支持以下功能，正是依赖这些功能来实现分布式锁的：

- Lease 机制：即租约机制（TTL，Time To Live），Etcd 可以为存储的 KV 对设置租约，当租约到期，KV 将失效删除；同时也支持续约，即 KeepAlive。
- Revision 机制：每个 key 带有一个 Revision 属性值，**etcd 每进行一次事务对应的全局 Revision 值都会加一，因此每个 key 对应的 Revision 属性值都是全局唯一的**。通过比较 Revision 的大小就可以知道进行写操作的顺序。
- 在实现分布式锁时，多个程序同时抢锁，根据 Revision 值大小依次获得锁，可以避免 “羊群效应” （也称 “惊群效应”），实现公平锁。
- Prefix 机制：即前缀机制，也称目录机制。可以**根据前缀（目录）获取该目录下所有的 key 及对应的属性**（包括 key, value 以及 revision 等）。
- Watch 机制：即监听机制，**Watch 机制支持 Watch 某个固定的 key，也支持 Watch 一个目录（前缀机制）**，当被 Watch 的 key 或目录发生变化，客户端将收到通知。

#### 过程

实现过程：

- 步骤 1: 准备

客户端连接 Etcd，以 /lock/mylock 为前缀创建全局唯一的 key，假设第一个客户端对应的 key="/lock/mylock/UUID1"，第二个为 key="/lock/mylock/UUID2"；客户端分别为自己的 key 创建租约 - Lease，租约的长度根据业务耗时确定，假设为 15s；

- 步骤 2: 创建定时任务作为租约的“心跳”

当一个客户端持有锁期间，其它客户端只能等待，为了避免等待期间租约失效，客户端需创建一个定时任务作为“心跳”进行续约。此外，如果持有锁期间客户端崩溃，心跳停止，key 将因租约到期而被删除，从而锁释放，避免死锁。

- 步骤 3: 客户端将自己全局唯一的 key 写入 Etcd

进行 put 操作，将步骤 1 中创建的 key 绑定租约写入 Etcd，根据 Etcd 的 Revision 机制，假设两个客户端 put 操作返回的 Revision 分别为 1、2，客户端需记录 Revision 用以接下来判断自己是否获得锁。

- 步骤 4: 客户端判断是否获得锁

**客户端以前缀 /lock/mylock 读取 keyValue 列表（keyValue 中带有 key 对应的 Revision），判断自己 key 的 Revision 是否为当前列表中最小的，如果是则认为获得锁**；否则监听列表中前一个 Revision 比自己小的 key 的删除事件，一旦监听到删除事件或者因租约失效而删除的事件，则自己获得锁。

- 步骤 5: 执行业务

获得锁后，操作共享资源，执行业务代码。

- 步骤 6: 释放锁

完成业务流程后，删除对应的key释放锁。

#### 实现

自带的 etcdctl 可以模拟锁的使用：

```bash
// 第一个终端
$ ./etcdctl lock mutex1
mutex1/326963a02758b52d

// 第二终端
$ ./etcdctl lock mutex1

// 当第一个终端结束了，第二个终端会显示
mutex1/326963a02758b531
```

在etcd的clientv3包中，实现了分布式锁。使用起来和mutex是类似的，为了了解其中的工作机制，这里简要的做一下总结。

etcd分布式锁的实现在go.etcd.io/etcd/clientv3/concurrency包中，主要提供了以下几个方法：

```go
* func NewMutex(s *Session, pfx string) *Mutex， 用来新建一个mutex
* func (m *Mutex) Lock(ctx context.Context) error，它会阻塞直到拿到了锁，并且支持通过context来取消获取锁。
* func (m *Mutex) Unlock(ctx context.Context) error，解锁
```

因此在使用etcd提供的分布式锁式非常简单，通常就是实例化一个mutex，然后尝试抢占锁，之后进行业务处理，最后解锁即可。

demo：

```go
package main

import (  
    "context"
    "fmt"
    "github.com/coreos/etcd/clientv3"
    "github.com/coreos/etcd/clientv3/concurrency"
    "log"
    "os"
    "os/signal"
    "time"
)

func main() {  
    c := make(chan os.Signal)
    signal.Notify(c)

    cli, err := clientv3.New(clientv3.Config{
        Endpoints:   []string{"localhost:2379"},
        DialTimeout: 5 * time.Second,
    })
    if err != nil {
        log.Fatal(err)
    }
    defer cli.Close()

    lockKey := "/lock"

    go func () {
        session, err := concurrency.NewSession(cli)
        if err != nil {
            log.Fatal(err)
        }
        m := concurrency.NewMutex(session, lockKey)
        if err := m.Lock(context.TODO()); err != nil {
            log.Fatal("go1 get mutex failed " + err.Error())
        }
        fmt.Printf("go1 get mutex sucess\n")
        fmt.Println(m)
        time.Sleep(time.Duration(10) * time.Second)
        m.Unlock(context.TODO())
        fmt.Printf("go1 release lock\n")
    }()

    go func() {
        time.Sleep(time.Duration(2) * time.Second)
        session, err := concurrency.NewSession(cli)
        if err != nil {
            log.Fatal(err)
        }
        m := concurrency.NewMutex(session, lockKey)
        if err := m.Lock(context.TODO()); err != nil {
            log.Fatal("go2 get mutex failed " + err.Error())
        }
        fmt.Printf("go2 get mutex sucess\n")
        fmt.Println(m)
        time.Sleep(time.Duration(2) * time.Second)
        m.Unlock(context.TODO())
        fmt.Printf("go2 release lock\n")
    }()

    <-c
}
```

#### 原理

Lock()函数的实现很简单：

```go
// Lock locks the mutex with a cancelable context. If the context is canceled
// while trying to acquire the lock, the mutex tries to clean its stale lock entry.
func (m *Mutex) Lock(ctx context.Context) error {
    s := m.s
    client := m.s.Client()

    m.myKey = fmt.Sprintf("%s%x", m.pfx, s.Lease())
    cmp := v3.Compare(v3.CreateRevision(m.myKey), "=", 0)
    // put self in lock waiters via myKey; oldest waiter holds lock
    put := v3.OpPut(m.myKey, "", v3.WithLease(s.Lease()))
    // reuse key in case this session already holds the lock
    get := v3.OpGet(m.myKey)
    // fetch current holder to complete uncontended path with only one RPC
    getOwner := v3.OpGet(m.pfx, v3.WithFirstCreate()...)
    resp, err := client.Txn(ctx).If(cmp).Then(put, getOwner).Else(get, getOwner).Commit()
    if err != nil {
        return err
    }
    m.myRev = resp.Header.Revision
    if !resp.Succeeded {
        m.myRev = resp.Responses[0].GetResponseRange().Kvs[0].CreateRevision
    }
    // if no key on prefix / the minimum rev is key, already hold the lock
    ownerKey := resp.Responses[1].GetResponseRange().Kvs
    if len(ownerKey) == 0 || ownerKey[0].CreateRevision == m.myRev {
        m.hdr = resp.Header
        return nil
    }

    // wait for deletion revisions prior to myKey
    hdr, werr := waitDeletes(ctx, client, m.pfx, m.myRev-1)
    // release lock key if wait failed
    if werr != nil {
        m.Unlock(client.Ctx())
    } else {
        m.hdr = hdr
    }
    return werr
}
```

首先通过一个事务来尝试加锁，这个事务主要包含了4个操作: cmp、put、get、getOwner。需要注意的是，key是由pfx和Lease()组成的。

- cmp: 比较加锁的key的修订版本是否是0。如果是0就代表这个锁不存在。
- put: 向加锁的key中存储一个空值，这个操作就是一个加锁的操作，但是这把锁是有超时时间的，超时的时间是session的默认时长。超时是为了防止锁没有被正常释放导致死锁。
- get: get就是通过key来查询
- getOwner: 注意这里是用m.pfx来查询的，并且带了查询参数WithFirstCreate()。使用pfx来查询是因为其他的session也会用同样的pfx来尝试加锁，并且因为每个LeaseID都不同，所以第一次肯定会put成功。但是只有最早使用这个pfx的session才是持有锁的，所以这个getOwner的含义就是这样的。

接下来才是通过判断来检查是否持有锁

```go
m.myRev = resp.Header.Revision
if !resp.Succeeded {
    m.myRev = resp.Responses[0].GetResponseRange().Kvs[0].CreateRevision
}
// if no key on prefix / the minimum rev is key, already hold the lock
ownerKey := resp.Responses[1].GetResponseRange().Kvs
if len(ownerKey) == 0 || ownerKey[0].CreateRevision == m.myRev {
    m.hdr = resp.Header
    return nil
}
```

m.myRev是当前的版本号，resp.Succeeded是cmp为true时值为true，否则是false。这里的判断表明当同一个session非第一次尝试加锁，当前的版本号应该取这个key的最新的版本号。

下面是取得锁的持有者的key。如果当前没有人持有这把锁，那么默认当前会话获得了锁。或者锁持有者的版本号和当前的版本号一致， 那么当前的会话就是锁的持有者。

```go
// wait for deletion revisions prior to myKey
hdr, werr := waitDeletes(ctx, client, m.pfx, m.myRev-1)
// release lock key if wait failed
if werr != nil {
    m.Unlock(client.Ctx())
} else {
    m.hdr = hdr
}
```

上面这段代码就很好理解了，因为走到这里说明没有获取到锁，那么这里等待锁的删除。

waitDeletes方法的实现也很简单，但是需要注意的是，这里的getOpts只会获取比当前会话版本号更低的key，然后去监控最新的key的删除。等这个key删除了，自己也就拿到锁了。

这种分布式锁的实现和我一开始的预想是不同的。它不存在锁的竞争，不存在重复的尝试加锁的操作。而是通过使用统一的前缀pfx来put，然后根据各自的版本号来排队获取锁。效率非常的高。避免了惊群效应

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/1460000021603220)
如图所示，共有4个session来加锁，那么根据revision来排队，获取锁的顺序为session2 -> session3 -> session1 -> session4。

这里面需要注意一个惊群效应，每一个client在锁住/lock这个path的时候，实际都已经插入了自己的数据，类似/lock/LEASE_ID，并且返回了各自的index（就是raft算法里面的日志索引），而只有最小的才算是拿到了锁，其他的client需要watch等待。例如client1拿到了锁，client2和client3在等待，而client2拿到的index比client3的更小，那么对于client1删除锁之后，client3其实并不关心，并不需要去watch。所以综上，等待的节点只需要watch比自己index小并且差距最小的节点删除事件即可。

## ETCD系列之三：网络层实现

### 1. 概述

在理清ETCD的各个模块的实现细节后，方便线上运维，理解各种参数组合的意义。本文先从网络层入手，后续文章会依次介绍各个模块的实现。

本文将着重介绍ETCD服务的网络层实现细节。在目前的实现中，ETCD通过HTTP协议对外提供服务，同样通过HTTP协议实现集群节点间数据交互。

网络层的主要功能是实现了服务器与客户端(能发出HTTP请求的各种程序)消息交互，以及集群内部各节点之间的消息交互。

### 2. ETCD-SERVER整体架构

ETCD-SERVER 大体上可以分为网络层，Raft模块，复制状态机，存储模块，架构图如图1所示。

![img](https://pic3.zhimg.com/v2-83aa7f6abeef9008e0e94cb528f574e2_r.jpg)

- 网络层：提供网络数据读写功能，监听服务端口，完成集群节点之间数据通信，收发客户端数据；
- Raft模块：完整实现了Raft协议；
- 存储模块：KV存储，WAL文件，SNAPSHOT管理
- 复制状态机：**这个是一个抽象的模块，状态机的数据维护在内存中，定期持久化到磁盘**，每次写请求会持久化到WAL文件，并根据写请求的内容修改状态机数据。

### 3. 节点之间网络拓扑结构 

ETCD集群的各个节点之间需要**通过HTTP协议来传递数据**，表现在：

- Leader 向Follower发送心跳包, Follower向Leader回复消息；
- Leader向Follower发送日志追加信息；
- Leader向Follower发送Snapshot数据；
- Candidate节点发起选举，向其他节点发起投票请求；
- Follower将收的写操作转发给Leader;

各个节点在任何时候都有可能变成Leader, Follower, Candidate等角色，同时为了减少创建链接开销，ETCD节点在启动之初就创建了和集群其他节点之间的链接。

因此，**ETCD集群节点之间的网络拓扑是一个任意2个节点之间均有长链接相互连接的网状结构**。如图2所示。

![img](https://pic2.zhimg.com/v2-4a4357b24660e77bb0588149645b15f9_r.jpg)

需要注意的是，**每一个节点都会创建到其他各个节点之间的长链接**。每个节点会向其他节点宣告自己监听的端口，该端口只接受来自其他节点创建链接的请求。

### 4. 节点之间消息交互

在ETCD实现中，根据不同用途，定义了各种不同的消息类型。**各种不同的消息，最终都通过google protocol buffer协议进行封装**。这些消息携带的数据大小可能不尽相同。例如 传输SNAPSHOT数据的消息数据量就比较大，甚至超过1GB, 而leader到follower节点之间的心跳消息可能只有几十个字节。

因此，**网络层必须能够高效地处理不同数据量的消息**。ETCD在实现中，对这些消息采取了分类处理，抽象出了2种类型消息传输通道：Stream类型通道和Pipeline类型通道。这两种消息传输通道都使用HTTP协议传输数据。

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/v2-c27e543cf27cbb769a53e0d1674c1c06_r.jpg)

集群启动之初，就创建了这两种传输通道，各自特点：

- Stream类型通道：点到点之间维护HTTP长链接，主要用于**传输数据量较小的消息**，例如追加日志，心跳等；
- Pipeline类型通道：点到点之间不维护HTTP长链接，**短链接传输数据**，用完即关闭。**用于传输数据量大的消息**，例如snapshot数据。

如果非要做做一个类别的话，Stream就向点与点之间维护了双向传输带，消息打包后，放到传输带上，传到对方，对方将回复消息打包放到反向传输带上；而Pipeline就像拥有N辆汽车，大消息打包放到汽车上，开到对端，然后在回来，最多可以同时发送N个消息。

#### Stream类型通道

Stream类型通道处理数据量少的消息，例如心跳，日志追加消息。点到点之间只维护1个HTTP长链接，交替向链接中写入数据，读取数据。

Stream 类型通道是节点启动后**主动与其他每一个节点建立**。Stream类型通道通过Channel 与Raft模块传递消息。每一个Stream类型通道关联2个Goroutines, 其中一个用于建立HTTP链接，并从链接上读取数据, decode成message, 通过Channel传给Raft模块中，另外一个通过Channel 从Raft模块中收取消息，然后写入通道。

具体点，ETCD使用golang的http包实现Stream类型通道：

- 被动发起方监听端口, 并在对应的url上挂载相应的handler（当前请求来领时，handler的ServeHTTP方法会被调用）
- 主动发起方发送HTTP GET请求；
- 监听方的Handler的ServeHTTP访问被调用(框架层传入http.ResponseWriter和http.Request对象），其中http.ResponseWriter对象作为参数传入Writter-Goroutine（就这么称呼吧），该Goroutine的主循环就是将Raft模块传出的message写入到这个responseWriter对象里；http.Request的成员变量Body传入到Reader-Gorouting(就这么称呼吧），该Gorutine的主循环就是不断读取Body上的数据，decode成message 通过Channel传给Raft模块。

#### Pipeline类型通道

Pipeline类型通道处理数量大消息，例如SNAPSHOT消息。这种类型消息需要和心跳等消息分开处理，否则会阻塞心跳。

Pipeline类型通道也可以传输小数据量的消息，当且仅当Stream类型链接不可用时。

Pipeline类型通道可用并行发出多个消息，维护一组Goroutines, 每一个Goroutines都可向对端发出POST请求（携带数据），收到回复后，链接关闭。

具体地，ETCD使用golang的http包实现的：

- 根据参数配置，启动N个Goroutines；
- 每一个Goroutines的主循环阻塞在消息Channel上，当收到消息后，通过POST请求发出数据，并等待回复。

### 5. 网络层与Raft模块之间的交互

在ETCD中，Raft协议被抽象为Raft模块。按照Raft协议，节点之间需要交互数据。在ETCD中，通过Raft模块中抽象的RaftNode拥有一个message box, RaftNode将各种类型消息放入到messagebox中，有专门Goroutine将box里的消息写入管道，而管道的另外一端就链接在网络层的不同类型的传输通道上，有专门的Goroutine在等待(select）。

而网络层收到的消息，也通过管道传给RaftNode。RaftNode中有专门的Goroutine在等待消息。

也就是说，网络层与Raft模块之间通过Golang Channel完成数据通信。这个比较容易理解。

### 6 ETCD-SERVER处理请求(与客户端的信息交互)

在ETCD-SERVER启动之初，会监听服务端口，当服务端口收到请求后，解析出message后，通过管道传入给Raft模块，当Raft模块按照Raft协议完成操作后，回复该请求（或者请求超时关闭了）。

### 7 主要数据结构

网络层抽象为Transport类，该类完成网络数据收发。对Raft模块提供Send/SendSnapshot接口，提供数据读写的Channel，对外监听指定端口。

![img](https://pic1.zhimg.com/v2-e713818a34a2d5726985daf9ad65baf8_r.jpg)

### 8. 结束

本文整理了ETCD节点网络层的实现，为分析其他模块打下基础。

## Raft

[参考](https://blog.51cto.com/u_15301988/3085390)

新版本的 etcd 实现，Raft 包就是 Raft 一致性算法的具体实现。

### Raft 中一个 Term（任期）是什么意思？

Raft 算法中，从时间上，一个 Term（任期）即从一次竞选开始到下一次竞选开始之间。从功能上讲，**如果 Follower 接收不到 Leader 的心跳信息，就会结束当前 Term，变为 Candidate 继而发起竞选**，继而帮助 Leader 故障时集群的恢复。发起竞选投票时，Term Value 小的 Node 不会竞选成功。如果 Cluster 不出现故障，那么一个 Term 将无限延续下去。另外，投票出现冲突也有可能直接进入下一任再次竞选。

![etcd — 架构原理_Kubernetes 云原生_03](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/f5da0c18e48fbb36bc71af6250da92e0.png)

### Raft 状态机是怎样切换的？

**Raft 刚开始运行时，Node 默认进入 Follower 状态，等待 Leader 发来心跳信息**。若等待超时，则状态由 Follower 切换到 Candidate 进入下一轮 Term 发起竞选，**等到收到 Cluster 的 “多数节点” 的投票时，该 Node 转变为 Leader**。Leader 有可能出现网络等故障，导致别的 Nodes 发起投票成为新 Term 的 Leader，此时原先的 Old Leader 会切换为 Follower。Candidate 在等待其它 Nodes 投票的过程中如果发现已经竞选成功了一个 Leader，那么也会切换为 Follower。

![etcd — 架构原理_Kubernetes 云原生_04](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/6fb75ccad6bbaf48223a0a19be57f810.png)

### 如何保证最短时间内竞选出 Leader，防止竞选冲突？

在 Raft 状态机一图中可以看到，在 Candidate 状态下， 有一个 times out，这里的 **times out 时间是个随机值**，也就是说，每个 Node 成为 Candidate 以后，times out 发起新一轮竞选的时间是各不相同的，这就会**出现一个时间差**。在时间差内，如果 Candidate1 收到的竞选信息比自己发起的竞选信息 Term Value 大（即对方为新一轮 Term），并且在新一轮想要成为 Leader 的 Candidate2 包含了所有提交的数据，那么 Candidate1 就会投票给 Candidate2。这样就保证了只有很小的概率会出现竞选冲突。

### 如何防止别的 Candidate 在遗漏部分数据的情况下发起投票成为 Leader？

Raft 竞选的机制中，使用随机值决定 times out，第一个超时的 Node 就会提升 Term 编号发起新一轮投票，一般情况下别的 Node 收到竞选通知就会投票。但是，**如果发起竞选的 Node 在上一个 Term 中保存的已提交数据不完整，Node 就会拒绝投票给它**。通过这种机制就可以防止遗漏数据的 Node 成为 Leader。

### Raft 某个节点宕机后会如何？

通常情况下，如果是 Follower 宕机，如果剩余可用节点数量超过半数，Cluster 可以几乎没有影响的正常工作。

如果宕机的是 Leader，那么 Follower 就收不到心跳而超时，发起竞选获得投票，成为新一轮 Term 的 Leader，继续为 Cluster 提供服务。

需要注意的是：etcd **目前没有任何机制会自动去变化整个 Cluster 的 Instances（总节点数量）**，即：如果没有人为的调用 API，etcd 宕机后的 Node 仍然被计算为总节点数中，**任何请求被确认需要获得的投票数都是这个总数的半数以上。**

![etcd — 架构原理_Kubernetes 云原生_05](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/c917a5906f5a7bec3c7da8cd920fd6ad.png)

### 为什么 Raft 算法在确定可用节点数量时不需要考虑拜占庭将军问题？

**拜占庭问题中提出：允许 n 个节点宕机还能提供正常服务的分布式架构，所需要的总节点数量为 3n+1**。而 Raft 只需要 2n+1 就可以了，其主要原因在于：**拜占庭将军问题中存在数据欺骗的现象，而 etcd 中假设所有的 Node 都是诚实的**。etcd 在竞选前需要**告诉别的 Node 自身的 Term 编号以及前一轮 Term 最终结束时的 Index 值**，这些数据都是准确的，其他 Node 可以根据这些值决定是否投票。另外，etcd 严格限制 Leader 到 Follower 这样的数据流向保证数据一致不会出错。

### 客户端从集群中的哪个节点读写数据？

为了保证数据的强一致性，**etcd Cluster 中的数据流向都是从 Leader 流向 Follower，也就是所有 Follower 的数据必须与 Leader 保持一致，如果不一致则会被覆盖**。

**即所有用户更新数据的请求都最先由 Leader 获得，然后通知其他节点也进行更新，等到 “大多数节点” 反馈时，再把数据一次性提交**。一个已提交的数据项才是 Raft 真正稳定存储下来的数据项，不再被修改，**最后再把提交的数据同步给其他 Follower**。因为每个 Node 都有 Raft 已提交数据准确的备份（最坏的情况也只是已提交数据还未完全同步），所以读的请求任意一个节点都可以处理。

实际上，用户可以对 etcd Cluster 中的任意 Node 进行读写：

- **读取**：可以从任意 Node 进行读取，因为**每个节点保存的数据是强一致的**。
- **写入**：etcd Cluster 首先会选举出 Leader，如果写入请求来自 Leader 即可直接写入，然后 Leader 会把写入分发给所有 Follower；如果写入请求来自其他 Follower 节点那么写入请求会给转发给 Leader 节点，由 Leader 节点写入之后再分发给集群上的所有其他节点。

### 如何保证数据一致性？

etcd 使用 Raft 协议来维护 Cluster 内各个 Nodes 状态的一致性。简单的说，etcd Cluster 是一个分布式系统，由多个 Nodes 相互通信构成整体对外服务，每个 Node 都存储了完整的数据，并且通过 Raft 协议保证每个 Node 维护的数据是一致的。

etcd Cluster 中的每个 Node 都维护了一个状态机，并且任意时刻，Cluster 中至多存在一个有效的主节点，即：Leader Node。由 Leader 处理所有来自客户端写操作，通过 Raft 协议保证写操作对状态机的改动会可靠的同步到其他 Follower Nodes。

### 如何选举 Leader 节点？

假设 etcd Cluster 中有 3 个 Node，Cluster 启动之初并没有被选举出的 Leader。此时，Raft 算法使用随机 Timer 来初始化 Leader 选举流程。比如说上面 3 个 Node 上都运行了 Timer（每个 Timer 的持续时间是随机的），而 Node1 率先完成了 Timer，随后它就会向其他两个 Node 发送成为 Leader 的请求，其他 Node 接收到请求后会以投票回应然后第一个节点被选举为 Leader。

成为 Leader 后，该 Node 会以固定时间间隔向其他 Node 发送通知，确保自己仍是 Leader。有些情况下当 Follower 们收不到 Leader 的通知后，比如说 Leader 节点宕机或者失去了连接，其他 Node 就会重复之前的选举流程，重新选举出新的 Leader。

### 如何判断写入是否成功？

etcd 认为写入请求被 Leader 处理并分发给了其他的 “多数节点” 后，就是一个成功的写入。“多数节点” 的数量的计算公式是 Quorum=N/2+1，N 为总结点数。也就是说，etcd 并发要将数据写入所有节点才算一次写，而是写入 “多数节点” 即可。

### 如何确定 etcd Cluster 的节点数？




上图左侧给出了![etcd — 架构原理_Kubernetes 云原生_06](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/42c155ede2961e5d5b2452fbd9328db8.png)集群中 Instances（节点总数）对应的 Quorum（仲裁数）的关系，Instances - Quorom 得到的就是集群中容错节点（允许出故障的节点)的数量。

所以在 etcd Cluster 推荐最少节点数为 3 个，因为 1 和 2 个 Instance 的容错节点数都是 0，一旦有一个节点宕掉整个集群就不能正常工作了。

进一步的说，当我们需要决定 etcd Cluster 中 Instances 的数量时，强烈推荐奇数数量的节点，比如：3、5、7、…，因为 6 个节点的集群的容错能力并没有比 5 个节点的好，他们的容错节点数是一样的，一旦容错节点超过 2 后，由于 Quorum 节点数小于 4，整个集群也就变为不可用的状态了。

![etcd — 架构原理_Kubernetes 云原生_07](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/48b38787a3808e6e91928877b58ef113.png)

### etcd 实现的 Raft 算法性能如何？

单实例节点支持每秒 2000 次数据写入。Node 数量越多，由于数据同步涉及到网络延迟，会根据实际情况越来越慢，而读性能会随之变强，因为每个节点都能处理用户请求。



## Store

[参考](https://blog.51cto.com/u_15301988/3085390)

Store，顾名思义，**是 etcd 实现的各项底层逻辑，并提供了相应的 API 支持**。要理解 Store，只需要从 etcd 的 API 入手。下面列举最常见的 CURD API 调用。

- 为 etcd 存储的键赋值：

```
curl http://127.0.0.1:2379/v2/keys/message -X PUT -d value="Hello world"

{
    "action":"set",                     # 执行的操作
    "node":{
        "createdIndex":2,           # etcd Node 每次有变化时都会自增的一个值
        "key":"/message",          # 请求路径
        "modifiedIndex":2,          # 类似 node.createdIndex，能引起 modifiedIndex 变化的操作包括 set, delete, update, create, compareAndSwap and compareAndDelete
        "value":"Hello world"      # 存储的内容
    }
}
```

- 查询 etcd 某个键存储的值：

```
curl http://127.0.0.1:2379/v2/keys/message -X GET
```

- 修改键值：

```
curl http://127.0.0.1:2379/v2/keys/message -XPUT -d value="Hello etcd"
```

- 删除键值：

```
curl http://127.0.0.1:2379/v2/keys/message -XDELETE
```

## WAL

etcd 的数据存储分为两个部分：

- 内存存储：内存中的存储除了顺序化的记录下所有用户对节点数据变更的记录外，还会对用户数据进行索引、建堆等方便查询的操作。
- 持久化（硬盘）存储：持久化则使用 WAL（Write Ahead Log，预写式日志）进行记录存储。

WAL 日志是二进制的，解析出来后是以上数据结构 LogEntry。其中：

![etcd — 架构原理_Kubernetes 云原生_08](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/81a3fbd22b3321f3140b51b749ba740b.png)

- 第一个字段 type，只有两种：

  - 0 表示 Normal

  - 1 表示 ConfChange，ConfChange 表示 etcd 本身的配置变更同步，比如有新的节点加入等。

- 第二个字段是 term，**每个 term 代表一个 Leader 的任期**，每次 Leader 变更 term 就会变化。

- 第三个字段是 index，这个序号是严格有序递增的，代表变更序号。


- 第四个字段是**二进制的 data**，将 Raft Request 对象的 pb 结构整个保存下。


etcd 源码下有个 tools/etcd-dump-logs 脚本工具，可以将 WAL 日志 dump 成文本查看，可以协助分析 Raft 协议。

Raft 协议本身不关心应用数据，也就是 data 中的部分，一致性都通过同步 WAL 日志来实现，每个 Node 将从 Leader 收到的 data apply 到本地的存储，**Raft 只关心日志的同步状态**，如果本地存储实现的有 Bug，比如没有正确的将 data apply 到本地，也可能会导致数据不一致。

在 WAL 的体系中，**所有的数据在提交之前都会进行日志记录**。在 etcd 的持久化存储目录中，有两个子目录：

- 一个是 WAL：存储着所有事务的变化记录；

- 另一个是 Snapshot：存储着某一个时刻 etcd 所有目录的数据。

通过 WAL 和 Snapshot 相结合的方式，etcd 可以有效的进行数据存储和节点故障恢复等操作。

### 为什么需要 Snapshot（快照）？

因为随着使用量的增加，WAL 存储的数据会暴增，为了防止磁盘很快就爆满，**etcd 默认每 10000 条记录做一次 Snapshot，经过 Snapshot 以后的 WAL 文件就可以删除**。所以，通过 API 可以查询的操作历史记录默认为 1000 条。

首次启动时，etcd 会把启动的配置信息存储到 data-dir 配置项指定的目录路径下。配置信息包括 Local Node ID、Cluster ID 和初始时的集群信息。**用户需要避免 etcd 从一个过期的数据目录中重新启动，因为使用过期的数据目录启动的 Node 会与 Cluster 中的其他 Nodes 产生不一致性**，例如：之前已经记录并同意 Leader Node 存储某个信息，重启后又向 Leader Node 申请这个信息。所以，为了最大化集群的安全性，一旦有任何数据损坏或丢失的可能性，你就应该把这个 Node 从 Cluster 中移除，然后加入一个不带数据目录的 New Node。

WAL（Write Ahead Log）最大的作用是记录了整个数据变化的全部历程。在 etcd 中，所有数据的修改在提交前，都要先写入到 WAL 中。使用 WAL 进行数据的存储使得 etcd 拥有两个重要功能：

- **故障快速恢复**： 当你的数据遭到破坏时，就可以通过执行所有 WAL 中记录的修改操作，快速从最原始的数据恢复到数据损坏前的状态。
- **数据回滚（undo）或重做（redo）**：因为所有的修改操作都被记录在 WAL 中，需要回滚或重做，只需要方向或正向执行日志中的操作即可。

### WAL 和 Snapshot 的命名规则？

在 etcd 的数据目录中，WAL 文件以 `$seq-$index.wal`的格式存储。最初始的 WAL 文件是 0000000000000000-0000000000000000.wal，表示这是所有 WAL 文件中的第 0 个，初始的 Raft 状态编号为 0。运行一段时间后可能需要进行日志切分，把新的条目放到一个新的 WAL 文件中。

假设，当集群运行到 Raft 状态为 20 时，需要进行 WAL 文件的切分时，下一份 WAL 文件就会变为 0000000000000001-0000000000000021.wal。如果在 10 次操作后又进行了一次日志切分，那么后一次的 WAL 文件名会变为 0000000000000002-0000000000000031.wal。可以看到 “-” 符号前面的数字是每次切分后自增 1，而 “-” 符号后面的数字则是根据实际存储的 Raft 起始状态来定。

而 Snapshot 的存储命名则比较容易理解，以 `$term-$index.wal` 格式进行命名存储。term 和 index 就表示存储 Snapshot 时数据所在的 Raft 节点状态，当前的任期编号以及数据项位置信息。

### etcd 的数据模型

**etcd 的设计目的是用来存放非频繁更新的数据，提供可靠的 Watch 插件，它暴露了键值对的历史版本，以支持低成本的快照、监控历史事件**。这些设计目标要求它使用一个持久化的、多版本的、支持并发的数据数据模型。

当 etcd 键值对的新版本保存后，先前的版本依然存在。从效果上来说，键值对是不可变的，etcd 不会对其进行 in-place 的更新操作，而总是生成一个新的数据结构。为了防止历史版本无限增加，etcd 的存储支持压缩（Compact）以及删除老旧版本。

## 逻辑视图

从逻辑角度看，**etcd 的存储是一个扁平的二进制键空间**，键空间有一个针对键（字节字符串）的词典序索引，因此范围查询的成本较低。

键空间维护了多个修订版本（Revisions），每一个原子变动操作（一个事务可由多个子操作组成）都会产生一个新的修订版本。在集群的整个生命周期中，修订版都是单调递增的。修订版同样支持索引，因此基于修订版的范围扫描也是高效的。压缩操作需要指定一个修订版本号，小于它的修订版会被移除。

一个键的一次生命周期（从创建到删除）叫做 “代（Generation）”，每个键可以有多个代。创建一个键时会增加键的版本（Version），如果在当前修订版中键不存在则版本设置为 1。删除一个键会创建一个墓碑（Tombstone），将版本设置为 0，结束当前代。每次对键的值进行修改都会增加其版本号，即：在同一代中版本号是单调递增的。

**当压缩时，任何在压缩修订版之前结束的代，都会被移除。值在修订版之前的修改记录（仅仅保留最后一个）都会被移除**。

## 物理视图

**etcd 将数据存放在一个持久化的 B+ 树**中，出于效率的考虑，**每个修订版仅仅存储相对前一个修订版的数据状态变化（Delta）**。单个修订版中可能包含了 B+ 树中的多个键。

键值对的键，是三元组（Major，Sub，Type）：

- Major：存储键值的修订版。

- Sub：用于区分相同修订版中的不同键。
- Type：用于特殊值的可选后缀，例如 t 表示值包含墓碑

键值对的值，包含从上一个修订版的 Delta。B+ 树，即：键的词法字节序排列，基于修订版的范围扫描速度快，可以方便的从一个修改版到另外一个的值变更情况查找。

etcd 同时在内存中维护了一个 B 树索引，用于加速针对键的范围扫描。索引的键是物理存储的键面向用户的映射，索引的值则是指向 B+ 树修该点的指针。

### etcd 的 Proxy 模式

Proxy 模式，即：**etcd 作为一个反向代理把客户端的请求转发给可用的 etcd Cluster**。这样，你就可以在每一台机器上都部署一个 Proxy 模式的 etcd 作为本地服务，如果这些 etcd Proxy 都能正常运行，那么你的服务发现必然是稳定可靠的。

所以 Proxy 模式并不是直接加入到符合强一致性的 etcd Cluster 中，也同样的，Proxy 并没有增加集群的可靠性，当然也没有降低集群的写入性能。

![etcd — 架构原理_Kubernetes 云原生_09](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/5a974d3abea90c975eba518aeab9cf4a.png)

### Proxy 模式取代 Standby 模式的原因？

实际上 etcd 每增加一个核心节点（Peer），都会增加 Leader 一定程度的包括网络、CPU 和磁盘的负担，因为每次信息的变化都需要进行同步备份。**增加 etcd 的核心节点可以让整个集群具有更高的可靠性**，但是当数量达到一定程度以后，增加可靠性带来的好处就变得不那么明显，反倒是降低了集群写入同步的性能。因此，增加一个轻量级的 Proxy 模式 etcd Node 是对直接增加 etcd 核心节点的一个有效代替。

Proxy 模式实际上是取代了原先的 Standby 模式。Standby 模式除了转发代理的功能以外，还会在核心节点因为故障导致数量不足的时候，从 Standby 模式转为正常节点模式。而当那个故障的节点恢复时，发现 etcd 的核心节点数量已经达到的预先设置的值，就会转为 Standby 模式。

但是新版本的 etcd 中，只会在最初启动 etcd Cluster 时，发现核心节点的数量已经满足要求时，自动启用 Proxy 模式，反之则并未实现。主要原因如下：

etcd 是用来保证高可用的组件，因此它所需要的系统资源，包括：内存、硬盘和 CPU 等，都应该得到充分保障以保证高可用。任由集群的自动变换随意地改变核心节点，无法让机器保证性能。所以 etcd 官方鼓励大家在大型集群中为运行 etcd 准备专有机器集群。
因为 etcd 集群是支持高可用的，部分机器故障并不会导致功能失效。所以机器发生故障时，管理员有充分的时间对机器进行检查和修复。

自动转换使得 etcd 集群变得复杂，尤其是如今 etcd 支持多种网络环境的监听和交互。在不同网络间进行转换，更容易发生错误，导致集群不稳定。