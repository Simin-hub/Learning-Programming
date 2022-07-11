# Kafka

[官网](https://kafka.apachecn.org/documentation.html)

[go使用Kafka](https://support.huaweicloud.com/devg-kafka/kafka-go.html)

[中文手册](https://www.orchome.com/kafka/index)

# 介绍

## 简单说明什么是kafka

Apache kafka是消息中间件的一种，我发现很多人不知道**消息中间件**是什么，在开始学习之前，我这边就先简单的解释一下什么是消息中间件，只是粗略的讲解，目前kafka已经可以做`更多`的事情。

举个例子，生产者消费者，生产者生产鸡蛋，消费者消费鸡蛋，生产者生产一个鸡蛋，消费者就消费一个鸡蛋，假设消费者消费鸡蛋的时候噎住了（系统宕机了），生产者还在生产鸡蛋，那新生产的鸡蛋就丢失了。再比如生产者很强劲（大交易量的情况），生产者1秒钟生产100个鸡蛋，消费者1秒钟只能吃50个鸡蛋，那要不了一会，消费者就吃不消了（消息堵塞，最终导致系统超时），消费者拒绝再吃了，”鸡蛋“又丢失了，这个时候我们放个篮子在它们中间，生产出来的鸡蛋都放到篮子里，消费者去篮子里拿鸡蛋，这样鸡蛋就不会丢失了，都在篮子里，而这个篮子就是”kafka“。

鸡蛋其实就是“数据流”，**系统之间的交互都是通过“数据流”来传输的（就是tcp、https什么的）**，也称为报文，也叫“消息”。

消息队列满了，其实就是篮子满了，”鸡蛋“ 放不下了，那赶紧多放几个篮子，其实就是kafka的扩容。

各位现在知道kafka是干什么的了吧，它就是那个"篮子"。

## kafka名词解释

后面大家会看到一些关于kafka的名词，比如topic、producer、consumer、broker，我这边来简单说明一下。

- `producer`：生产者，就是它来生产“鸡蛋”的。
- `consumer`：消费者，生出的“鸡蛋”它来消费。
- `topic`：你把它理解为标签，生产者每生产出来一个鸡蛋就贴上一个标签（topic），消费者可不是谁生产的“鸡蛋”都吃的，这样不同的生产者生产出来的“鸡蛋”，消费者就可以选择性的“吃”了。
- `broker`：就是篮子了。

# 入门

### Apache Kafka® 是 *一个分布式流处理平台*. 这到底意味着什么呢?

我们知道流处理平台有以下三种特性:

1. 可以让**你发布和订阅流式的记录**。这一方面与消息队列或者企业消息系统类似。
2. 可以储存流式的记录，并且有较好的容错性。
3. 可以在流式记录产生时就进行处理。

Kafka适合什么样的场景?

#### 它可以用于两大类别的应用:

1. **构造实时流数据管道**，它可以在系统或应用之间可靠地获取数据。 (相当于message queue)
2. **构建实时流式应用程序**，对这些流数据进行转换或者影响。 (就是流处理，通过kafka stream topic和topic之间内部进行变化)

为了理解Kafka是如何做到以上所说的功能，从下面开始，我们将深入探索Kafka的特性。.

#### 首先是一些概念:

- Kafka作为一个集群，运行在一台或者多台服务器上.
- Kafka 通过 *topic* 对存储的流数据进行分类。
- 每条记录中包含一个key，一个value和一个timestamp（时间戳）。

#### Kafka有四个核心的API:

- The [Producer API](https://kafka.apachecn.org/documentation.html#producerapi) 允许一个应用程序发布一串流式的数据到一个或者多个Kafka topic。
- The [Consumer API](https://kafka.apachecn.org/documentation.html#consumerapi) 允许一个应用程序订阅一个或多个 topic ，并且对发布给他们的流式数据进行处理。
- The [Streams API](https://kafka.apachecn.org/documentation/streams) 允许一个应用程序作为一个*流处理器*，**消费一个或者多个topic产生的输入流**，然后生产一个输出流到一个或多个topic中去，在输入输出流中进行有效的转换。
- The [Connector API](https://kafka.apachecn.org/documentation.html#connect) 允许**构建并运行可重用的生产者或者消费者**，将Kafka topics连接到已存在的应用程序或者数据系统。比如，连接到一个关系型数据库，捕捉表（table）的所有变更内容。

![kafka入门介绍](https://img.orchome.com/group1/M00/00/01/KmCudlf7DXiAVXBMAAFScKNS-Og538.png)

Client和Server之间的通讯，是通过一条简单、高性能并且和开发语言无关的[TCP协议](/fwd?link=https://kafka.apache.org/protocol.html)。并且该协议保持与老版本的兼容。Kafka提供了Java Client（客户端）。除了Java客户端外，还有非常多的[其它编程语言的客户端](/fwd?link=https://cwiki.apache.org/confluence/display/KAFKA/Clients)。

## 首先来了解一下Kafka所使用的基本术语：

#### Topic

Kafka将消息分门别类，每一类的消息称之为一个主题（Topic）。

#### Producer

发布消息的对象称之为**主题生产者**（Kafka topic producer）

#### Consumer

订阅消息并处理发布的消息的对象称之为**主题消费者**（consumers）

#### Broker

已发布的消息保存在一组服务器中，称之为Kafka集群。集群中的每一个服务器都是一个代理（Broker）。 消费者可以订阅一个或多个主题（topic），并从Broker拉数据，从而消费这些已发布的消息。

## 主题和日志 （Topic和Log）

让我们更深入的了解Kafka中的Topic。

Topic是发布的消息的类别名，一个topic可以有零个，一个或多个消费者订阅该主题的消息。

对于**每个topic，Kafka集群都会维护一个分区log**，就像下图中所示：

![kafka topic剖析](https://img.orchome.com/group1/M00/00/01/KmCudlf7DsaAVF0WAABMe0J0lv4158.png)

**每一个分区都是一个顺序的、不可变的消息队列**， 并且可以持续的添加。分区中的消息都被分了一个序列号，称之为偏移量(offset)，在每个分区中此偏移量都是唯一的。

**Kafka集群保持所有的消息，直到它们过期（无论消息是否被消费）**。实际上消费者所持有的仅有的元数据就是这个offset（偏移量），也就是说offset由消费者来控制：正常情况当消费者消费消息的时候，偏移量也线性的的增加。但是实际偏移量由消费者控制，消费者可以将偏移量重置为更早的位置，重新读取消息。可以看到这种设计对消费者来说操作自如，一个消费者的操作不会影响其它消费者对此log的处理。

![kafka offset偏移量](https://img.orchome.com/group1/M00/00/01/KmCudlf7D2iALXG_AAIhinsLf_Q676.png)

再说说分区。**Kafka中采用分区的设计有几个目的。一是可以处理更多的消息，不受单台服务器的限制。Topic拥有多个分区意味着它可以不受限的处理更多的数据。第二，分区可以作为并行处理的单元**，稍后会谈到这一点。

## 分布式(Distribution)

Log的分区被分布到集群中的多个服务器上。每个服务器处理它分到的分区。 根据配置每个分区还可以复制到其它服务器作为备份容错。 **每个分区有一个leader**，零或多个follower。**Leader处理此分区的所有的读写请求，而follower被动的复制数据**。如果leader宕机，其它的一个follower会被推举为新的leader。 一台服务器可能同时是一个分区的leader，另一个分区的follower。 这样可以平衡负载，避免所有的请求都只让一台或者某几台服务器处理。

## Geo-Replication(异地数据同步技术)

Kafka MirrorMaker为群集提供`geo-replication`支持。借助`MirrorMaker`，消息可以跨多个数据中心或云区域进行复制。 您可以在active/passive场景中用于备份和恢复; 或者在active/passive方案中将数据置于更接近用户的位置，或数据本地化。

## 生产者(Producers)

生产者往某个Topic上发布消息。生产者也负责选择发布到Topic上的哪一个分区。最简单的方式**从分区列表中轮流选择**。也可以根据某种算法依照权重选择分区。开发者负责如何选择分区的算法。

## 消费者(Consumers)

通常来讲，**消息模型可以分为两种， 队列和发布-订阅式**。 **队列的处理方式是 一组消费者从服务器读取消息，一条消息只有其中的一个消费者来处理。在发布-订阅模型中，消息被广播给所有的消费者，接收到消息的消费者都可以处理此消息**。Kafka为这两种模型提供了单一的消费者抽象模型： 消费者组 （consumer group）。 消费者用一个消费者组名标记自己。 一个发布在Topic上消息被分发给此消费者组中的一个消费者，**同一消费组内的消费者不能消费同一消息**。 **假如所有的消费者都在一个组中，那么这就变成了queue模型**。 **假如所有的消费者都在不同的组中，那么就完全变成了发布-订阅模型**。 更通用的， 我们可以创建一些消费者组作为逻辑上的订阅者。每个组包含数目不等的消费者， 一个组内多个消费者可以用来扩展性能和容错。

[消费组和消费者的关系](https://segmentfault.com/a/1190000039125247)

正如下图所示：

![kafka消费者](https://img.orchome.com/group1/M00/00/01/KmCudlf7D-OAEjy8AABoxGLnMI4173.png)

2个kafka集群托管4个分区（P0-P3），2个消费者组，消费组A有2个消费者实例，消费组B有4个。

正像传统的消息系统一样，Kafka保证消息的顺序不变。 再详细扯几句。传统的队列模型保持消息，并且保证它们的先后顺序不变。但是， 尽管服务器保证了消息的顺序，消息还是异步的发送给各个消费者，**消费者收到消息的先后顺序不能保证**了。这也意味着并行消费将不能保证消息的先后顺序。用过传统的消息系统的同学肯定清楚，消息的顺序处理很让人头痛。如果只让一个消费者处理消息，又违背了并行处理的初衷。 在这一点上Kafka做的更好，尽管并没有完全解决上述问题。 Kafka采用了一种分而治之的策略：分区。 因为**Topic分区中消息只能由消费者组中的唯一一个消费者处理，所以消息肯定是按照先后顺序进行处理的**。但是它也仅仅是保证Topic的一个分区顺序处理，不能保证跨分区的消息先后处理顺序。 所以，如果你想要顺序的处理Topic的所有消息，那就只提供一个分区。

## Kafka的保证(Guarantees)

- 生产者发送到**一个特定的Topic的分区上，消息将会按照它们发送的顺序依次加入**，也就是说，如果一个消息M1和M2使用相同的producer发送，M1先发送，那么M1将比M2的offset低，并且优先的出现在日志中。
- 消费者收到的消息也是此顺序。
- 如果一个Topic配置了复制因子（replication factor）为N， 那么可**以允许N-1服务器宕机而不丢失任何已经提交（committed）的消息**。

有关这些保证的更多详细信息，请参见文档的设计部分。

## kafka作为一个消息系统

##### Kafka的流与传统企业消息系统相比的概念如何？

传统的消息有两种模式：`队列`和`发布订阅`。 在队列模式中，消费者池从服务器读取消息（**每个消息只被其中一个读取**）; 发布订阅模式：**消息广播给所有的消费者**。这两种模式都有优缺点，队列的优点是允许多个消费者瓜分处理数据，这样可以扩展处理。但是，队列不像多个订阅者，一旦消息者进程读取后故障了，那么消息就丢了。而`发布和订阅`允许你广播数据到多个消费者，由于每个订阅者都订阅了消息，所以没办法缩放处理。

kafka中消费者组有两个概念：`队列`：消费者组（consumer group）允许同名的消费者组成员瓜分处理。`发布订阅`：允许你广播消息给多个消费者组（不同名）。

kafka的每个topic都具有这两种模式。

##### kafka有比传统的消息系统更强的顺序保证。

传统的消息系统按顺序保存数据，如果多个消费者从队列消费，则服务器按存储的顺序发送消息，但是，尽管服务器按顺序发送，消息异步传递到消费者，因此消息可能乱序到达消费者。这意味着消息存在并行消费的情况，顺序就无法保证。消息系统常常通过仅设1个消费者来解决这个问题，但是这意味着没用到并行处理。

kafka做的更好。通过并行topic的parition —— kafka提供了顺序保证和负载均衡。每个partition仅由同一个消费者组中的一个消费者消费到。并确保消费者是该partition的唯一消费者，并按顺序消费数据。每个topic有多个分区，则需要对多个消费者做负载均衡，但请注意，`相同的消费者组中不能有比分区更多的消费者，否则多出的消费者一直处于空等待，不会收到消息`。

## kafka作为一个存储系统

所有发布消息到`消息队列`和消费分离的系统，实际上都充当了一个存储系统（发布的消息先存储起来）。Kafka比别的系统的优势是它是一个非常高性能的`存储系统`。

写入到kafka的数据将写到磁盘并复制到集群中保证容错性。并允许生产者等待消息应答，直到消息完全写入。

kafka的磁盘结构 - 无论你服务器上有50KB或50TB，执行是相同的。

client来控制读取数据的位置。你还可以认为kafka是一种专用于高性能，低延迟，提交日志存储，复制，和传播特殊用途的`分布式文件系统`。

## kafka的流处理

仅仅读，写和存储是不够的，**kafka的目标是实时的流处理**。

在kafka中，流处理持续获取`输入topic`的数据，进行处理加工，然后写入`输出topic`。例如，一个零售APP，接收销售和出货的`输入流`，统计数量或调整价格后输出。

可以直接使用producer和consumer API进行简单的处理。对于复杂的转换，Kafka提供了更强大的Streams API。可构建`聚合计算`或`连接流到一起`的复杂应用程序。

助于解决此类应用面临的硬性问题：处理无序的数据，代码更改的再处理，执行状态计算等。

Streams API在Kafka中的核心：使用producer和consumer API作为输入，利用Kafka做状态存储，使用相同的组机制在stream处理器实例之间进行容错保障。

## 拼在一起

消息传递，存储和流处理的组合看似反常，但对于Kafka作为流式处理平台的作用至关重要。

像HDFS这样的分布式文件系统允许存储静态文件来进行批处理。这样系统可以有效地存储和处理来自过去的历史数据。

传统企业的消息系统允许在你订阅之后处理未来的消息：在未来数据到达时处理它。

Kafka结合了这两种能力，这种组合对于kafka作为流处理应用和流数据管道平台是至关重要的。

批处理以及消息驱动应用程序的流处理的概念：**通过组合存储和低延迟订阅，流处理应用可以用相同的方式对待过去和未来的数据**。它是一个单一的应用程序，它可以处理历史的存储数据，当它处理到最后一个消息时，它进入等待未来的数据到达，而不是结束。

同样，对于流数据管道（pipeline），订阅实时事件的组合使得可以将Kafka用于非常低延迟的管道；但是，可靠地存储数据的能力使得它可以将其用于必须保证传递的关键数据，或与仅定期加载数据或长时间维护的离线系统集成在一起。流处理可以在数据到达时转换它。

# 使用场景

## 消息

kafka更好的替换传统的消息系统，消息系统被用于各种场景（解耦数据生产者，缓存未处理的消息等），与大多数消息系统比较，kafka有更好的吞吐量，内置分区，副本和故障转移，这有利于处理大规模的消息。

根据我们的经验，消息往往用于较低的吞吐量，但需要低的`端到端`延迟，并需要提供强大的耐用性的保证。

在这一领域的kafka比得上传统的消息系统，如`ActiveMQ`或`RabbitMQ`。

## 网站活动追踪

**kafka原本的使用场景**：**用户的活动追踪**，**网站的活动（网页游览，搜索或其他用户的操作信息）发布到不同的主题中心**，这些消息可实时处理，实时监测，也可加载到Hadoop或离线处理数据仓库。

每个用户页面视图都会产生非常高的量。

## 指标

kafka也常常用于监测数据。分布式应用程序生成的统计数据集中聚合。

## 日志聚合

许多人使用Kafka作为日志聚合解决方案的替代品。日志聚合通常从服务器中收集物理日志文件，并将它们放在中央位置（可能是文件服务器或HDFS）进行处理。Kafka抽象出文件的细节，并将日志或事件数据更清晰地抽象为消息流。这允许更低延迟的处理并更容易支持多个数据源和分布式数据消费。

## 流处理

kafka中消息处理一般包含多个阶段。其中原始输入数据是从kafka主题消费的，然后汇总，丰富，或者以其他的方式处理转化为新主题，例如，一个推荐新闻文章，文章内容可能从“articles”主题获取；然后进一步处理内容，得到一个处理后的新内容，最后推荐给用户。这种处理是基于单个主题的实时数据流。从`0.10.0.0`开始，轻量，但功能强大的流处理，就可以这样进行数据处理了。

除了Kafka Streams，还有Apache Storm和Apache Samza可选择。

## 事件采集

事件采集是一种应用程序的设计风格，其中状态的变化根据时间的顺序记录下来，kafka支持这种非常大的存储日志数据的场景。

## 提交日志

kafka可以作为一种分布式的外部日志，可帮助节点之间复制数据，并作为失败的节点来恢复数据重新同步，kafka的日志压缩功能很好的支持这种用法，这种用法类似于`Apacha BookKeeper`项目。

# 接口API

## Kafka有5个核心API：

1. [Producer API]() 允许应用程序发送数据流到kafka集群中的topic。
2. [Consumer API]() 允许应用程序从kafka集群的topic中读取数据流。
3. [Streams API]() 允许从输入topic转换数据流到输出topic。
4. [Connect API]() 通过实现连接器（connector），不断地从一些源系统或应用程序中拉取数据到kafka，或从kafka提交数据到宿系统（sink system）或应用程序。
5. [Admin API]()  用于管理和检查topic，broker和其他Kafka对象。

kafka公开了其所有的功能协议，与语言无关。只有java客户端作为kafka项目的一部分进行维护，其他的作为开源的项目提供，这里提供了非java客户端的列表。
[https://cwiki.apache.org/confluence/display/KAFKA/Clients](/fwd?link=https://cwiki.apache.org/confluence/display/KAFKA/Clients)



![Kafka 架构原理解析- DockOne.io](http://dockone.io/uploads/article/20200328/147bcb45e162e44e7d7911d34f69b888.png)

# 面试题

[参考](https://blog.51cto.com/u_15127589/2677281)、[参考2](https://xie.infoq.cn/article/6c879c4c3b52e416f251b2909)

### 1、Kafka 是什么？主要应用场景有哪些？ 

Kafka 是一个分布式流式处理平台。 流平台具有三个关键功能： 

- 消息队列：发布和订阅消息流，这个功能类似于消息队列，这也是 Kafka  也被归类为消息队列的原因。
- 容错的持久方式存储记录消息流：Kafka 会把消息持久化到磁盘，有效避免了消息丢失的风险。 
- 流式处理平台：在消息发布的时候进行处理，Kafka 提供了一个完整的流式处理类库。 

Kafka 主要有两大应用场景： 

- 消息队列：建立实时流数据管道，以可靠地在系统或应用程序之间获取数据。 
- 数据处理：构建实时的流数据处理程序来转换或处理数据流。

#### 在哪些场景下会选择 Kafka？

- **日志收集**：一个公司可以用Kafka可以收集各种服务的log，通过kafka以统一接口服务的方式开放给各种consumer，例如hadoop、HBase、Solr等。
- **消息系统**：解耦和生产者和消费者、缓存消息等。
- **用户活动跟踪**：Kafka经常被用来记录web用户或者app用户的各种活动，如浏览网页、搜索、点击等活动，这些活动信息被各个
- 服务器发布到kafka的topic中，然后订阅者通过订阅这些topic来做实时的监控分析，或者装载到hadoop、数据仓库中做离线分析和挖掘。
- **运营指标**：Kafka也经常用来记录运营监控数据。包括收集各种分布式应用的数据，生产各种操作的集中反馈，比如报警和报告。
- **流式处理**：比如spark streaming和 Flink

### Kafka 都有哪些特点？

- **高吞吐量、低延迟**：kafka每秒可以处理几十万条消息，它的延迟最低只有几毫秒，每个topic可以分多个partition, consumer group 对partition进行consume操作。
- **可扩展性**：kafka集群支持热扩展
- **持久性、可靠性**：消息被持久化到本地磁盘，并且支持数据备份防止数据丢失
- **容错性**：允许集群中节点失败（若副本数量为n,则允许n-1个节点失败）
- **高并发**：支持数千个客户端同时读写

### Kafka 架构分为以下几个部分

- Producer ：消息生产者，就是向 kafka broker 发消息的客户端。
- Consumer ：消息消费者，向 kafka broker 取消息的客户端。
- Topic ：可以理解为一个队列，一个 Topic 又分为一个或多个分区，
- Consumer Group：这是 kafka 用来实现一个 topic 消息的广播（发给所有的 consumer）和单播（发给任意一个 consumer）的手段。**一个 topic 可以有多个 Consumer Group**。
- Broker ：一台 kafka 服务器就是一个 broker。一个集群由多个 broker 组成。一个 broker 可以容纳多个 topic。
- Partition：为了实现扩展性，一个非常大的 topic 可以分布到多个 broker上，每个 partition 是一个有序的队列。partition 中的每条消息都会被分配一个有序的id（offset）。将消息发给 consumer，kafka 只保证按一个 partition 中的消息的顺序，不保证一个 topic 的整体（多个 partition 间）的顺序。
- Offset：kafka 的存储文件都是按照 offset.kafka 来命名，用 offset 做名字的好处是方便查找。例如你想找位于 2049 的位置，只要找到 2048.kafka 的文件即可。当然 the first offset 就是 00000000000.kafka。

### 2、和其他消息队列相比，Kafka 的优势在哪里？ 

我们现在经常提到 Kafka 的时候就已经默认它是一个非常优秀的消息队列了， 我们也会经常拿它跟 RocketMQ、RabbitMQ 对比。我觉得 Kafka 相比其他消息 队列主要的优势如下：

- **极致的性能** ：基于 Scala 和 Java 语言开发，设计中大量使用了批量处理和异步的思想，最高可以每秒处理千万级别的消息。 
- **生态系统兼容性无可匹敌** ：Kafka 与周边生态系统的兼容性是最好的没 有之一，尤其在大数据和流计算领域。 实际上在早期的时候 Kafka 并不是一个合格的消息队列，早期的 Kafka 在消 息队列领域就像是一个衣衫褴褛的孩子一样，功能不完备并且有一些小问题比 如丢失消息、不保证消息可靠性等等。当然，这也和 LinkedIn 最早开发 Kafka 用于处理海量的日志有很大关系，哈哈哈，人家本来最开始就不是为了 作为消息队列滴，谁知道后面误打误撞在消息队列领域占据了一席之地。

### 3、什么是 Producer、Consumer、Broker、Topic、 Partition？ 

Kafka 将生产者发布的消息发送到 Topic（主题） 中，需要这些消息的消费者可以订阅这些 Topic（主题）。Kafka 比较重要的几个概念： 

- Producer（生产者） : 产生消息的一方。
- Consumer（消费者） : 消费消息的一方。 
- Broker（代理） : 可以看作是一个独立的 Kafka 实例。多个 Kafka  Broker 组成一个 Kafka Cluster。
- Topic（主题） : Producer 将消息发送到特定的主题，Consumer 通过订 阅特定的 Topic(主题) 来消费消息。
- Partition（分区） : Partition 属于 Topic 的一部分。一个 Topic 可 以有多个 Partition ，并且同一 Topic 下的 Partition 可以分布在不同的 Broker 上，这也就表明一个 Topic 可以横跨多个 Broker 。这正如我上面所画的图一样。

### Kafka 的高可靠性是怎么实现的？

[参考](https://blog.51cto.com/u_15127589/2677133)

Topic 分区副本

Kafka 如何保证消息不丢失

Leader 选举

### 4、Kafka 的多副本机制了解吗？

 Kafka 为**分区（Partition）引入了多副本（Replica）机制**。分区 （Partition）中的多个副本之间会有一个叫做 leader 的家伙，其他副本称为 follower。我们**发送的消息会被发送到 leader 副本，然后 follower 副本才 能从 leader 副本中拉取消息进行同步**。 **生产者和消费者只与 leader 副本交互**。你可以理解为其他副本只是 leader  副本的拷贝，它们的存在只是为了保证消息存储的安全性。当 leader 副本发生故障时会从 follower 中选举出一个 leader,但是 **follower 中如果有和 leader 同步程度达不到要求的参加不了 leader 的竞选**。

### 6、Zookeeper 在 Kafka 中的作用知道吗？

- **Broker 注册** ：在 Zookeeper 上会有一个专门用来进行 Broker 服务器 列表记录的节点。每个 Broker 在启动时，都会到 Zookeeper 上进行注 册，即到 /brokers/ids 下创建属于自己的节点。每个 Broker 就会将 自己的 IP 地址和端口等信息记录到该节点中去 
- **Topic 注册** ： 在 Kafka 中，同一个 Topic 的消息会被分成多个分区并 将其分布在多个 Broker 上，**这些分区信息及与 Broker 的对应关系也都 是由 Zookeeper 在维护**。比如我创建了一个名字为 my-topic 的主题并且 它有两个分区，对应到 zookeeper 中会创建这些文件夹： /brokers/topics/my-topic/Partitions/0、/brokers/topics/mytopic/Partitions/1 
- **负载均衡** ：上面也说过了 Kafka 通过给特定 Topic 指定多个 Partition, 而各个 Partition 可以分布在不同的 Broker 上, 这样便能 提供比较好的并发能力。 对于同一个 Topic 的不同 Partition，Kafka  会尽力将这些 Partition 分布到不同的 Broker 服务器上。当生产者产生消息后也会尽量投递到不同 Broker 的 Partition 里面。当 Consumer 消 费的时候，**Zookeeper 可以根据当前的 Partition 数量以及 Consumer 数量来实现动态负载均衡**。

### 7、Kafka 如何保证消息的消费顺序？

我们在使用消息队列的过程中经常有业务场景需要严格保证消息的消费顺序， 比如我们同时发了 2 个消息，这 2 个消息对应的操作分别对应的数据库操作 是： 

- 更改用户会员等级。 
- 根据会员等级计算订单价格。 

假如这两条消息的消费顺序不一样造成的最终结果就会截然不同。 **Kafka 中 Partition(分区)是真正保存消息的地方**，我们发送的消息都被放在了这里。而我们的 Partition(分区) 又存在于 Topic(主题) 这个概念中，并 且我们可以给特定 Topic 指定多个 Partition。 每次添加消息到 Partition(分区) 的时候都会采用尾加法。 Kafka 只能为我们保证 Partition(分区) 中的消息有序。

 消息在被追加到 Partition(分区)的时候都会分配一个特定的偏移量 （offset）。Kafka 通过偏移量（offset）来保证消息在分区内的顺序性。 所以，我们就有一种很简单的保证消息消费顺序的方法：1 个 Topic 只对应一 个 Partition。这样当然可以解决问题，但是破坏了 Kafka 的设计初衷。 Kafka 中发送 1 条消息的时候，可以指定 topic, partition, key,data（数 据） 4 个参数。如果你发送消息的时候指定了 Partition 的话，所有消息都会被发送到指定的 Partition。并且，**同一个 key 的消息可以保证只发送到同 一个 partition，这个我们可以采用表/对象的 id 来作为 key**。 

总结一下，对于如何保证 Kafka 中消息消费的顺序，有了下面两种方法： 

- **1 个 Topic 只对应一个 Partition**。 （即通过队列的方式）
- **发送消息的时候指定 key/Partition**。

### 8、Kafka 如何保证消息不丢失？ 

[参考](https://zhuanlan.zhihu.com/p/459610418)

#### 生产者丢失消息的情况 

生产者(Producer) 调用 send 方法发送消息之后，消息可能因为网络问题并没有发送过去。 所以，我们**不能默认在调用 send 方法发送消息之后消息发送成功了**。为了确定消息是发送成功，我们要判断消息发送的结果。但是要注意的是 Kafka 生产者 (Producer) 使用 send 方法发送消息实际上是**异步的操作**，我们可以**通过 get()方法获取调用结果，但是这样也让它变为了同步操作**。 

解决办法：

**更换调用方式**

**ACK 确认机制**

**重试次数**

 **重试时间**

#### Broker 端解决方案

在剖析Broker 端丢失场景的时候， 我们得出其是通过「**异步批量刷盘**」的策略，先将数据存储到「**PageCache**」，再进行异步刷盘， 由于没有提供 「**同****步刷盘**」策略，因此 Kafka 是通过「**多分区多副本**」的方式来最大限度的保证数据不丢失。

#### 消费者丢失消息的情况 

我们知道消息在被追加到 Partition(分区)的时候都会分配一个特定的偏移量 （offset）。偏移量（offset)表示 Consumer 当前消费到的 Partition(分区) 的所在的位置。Kafka 通过偏移量（offset）可以保证消息在分区内的顺序性。 当消费者拉取到了分区的某个消息之后，**消费者会自动提交了 offset**。自动提交的话会有一个问题，试想一下，当消费者刚拿到这个消息准备进行真正消费的时候，突然挂掉了，消息实际上并没有被消费，但是 offset 却被自动提交 了。 解决办法也比较粗暴，我们**手动关闭自动提交 offset**，每次在真正消费完消息之后再自己手动提交 offset 。 但是，细心的朋友一定会发现，这样会带来 消息被重新消费的问题。比如你刚刚消费完消息之后，还没提交 offset，结果自己挂掉了，那么这个消息理论上就会被消费两次。

解决办法：**拉取数据、业务逻辑处理、提交消费 Offset 位移信息**。

### ISR、OSR、AR 是什么？

- ISR：In-Sync Replicas 副本同步队列
- OSR：Out-of-Sync Replicas
- AR：Assigned Replicas 所有副本

ISR是由leader维护，follower从leader同步数据有一些延迟（具体可以参见 [图文了解 Kafka 的副本复制机制](https://blog.51cto.com/u_15127589/2682641)），超过相应的阈值会把 follower 剔除出 ISR, 存入OSR（Out-of-Sync Replicas ）列表，新加入的follower也会先存放在OSR中。AR=ISR+OSR。

### LEO、HW、LSO、LW等分别代表什么

- LEO：是 LogEndOffset 的简称，代表当前日志文件中下一条
- HW：水位或水印（watermark）一词，也可称为高水位(high watermark)，通常被用在流式处理领域（比如Apache Flink、Apache Spark等），以表征元素或事件在基于时间层面上的进度。在Kafka中，水位的概念反而与时间无关，而是与位置信息相关。严格来说，它表示的就是位置信息，即位移（offset）。取 partition 对应的 ISR中 最小的 LEO 作为 HW，consumer 最多只能消费到 HW 所在的位置上一条信息。
- LSO：是 LastStableOffset 的简称，对未完成的事务而言，LSO 的值等于事务中第一条消息的位置(firstUnstableOffset)，对已完成的事务而言，它的值同 HW 相同
- LW：Low Watermark 低水位, 代表 AR 集合中最小的 logStartOffset 值。

### 请谈一谈 Kafka 数据一致性原理

一致性就是说不论是老的 Leader 还是新选举的 Leader，Consumer 都能读到一样的数据。

![img](https://s6.51cto.com/images/blog/202103/30/1e7895970e33093dbca650efb5c4dd56.png?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

假设分区的副本为3，其中副本0是 Leader，副本1和副本2是 follower，并且在 ISR 列表里面。虽然副本0已经写入了 Message4，但是 Consumer 只能读取到 Message2。因为所有的 ISR 都同步了 Message2，**只有 High Water Mark 以上的消息才支持 Consumer 读取**，而 High Water Mark 取决于 ISR 列表里面偏移量最小的分区，对应于上图的副本2，这个很类似于木桶原理。

这样做的原因是还没有被足够多副本复制的消息被认为是“不安全”的，如果 Leader 发生崩溃，另一个副本成为新 Leader，那么这些消息很可能丢失了。如果我们允许消费者读取这些消息，可能就会破坏一致性。试想，一个消费者从当前 Leader（副本0） 读取并处理了 Message4，这个时候 Leader 挂掉了，选举了副本1为新的 Leader，这时候另一个消费者再去从新的 Leader 读取消息，发现这个消息其实并不存在，这就导致了数据不一致性问题。

当然，引入了 High Water Mark 机制，会导致 Broker 间的消息复制因为某些原因变慢，那么消息到达消费者的时间也会随之变长（因为我们会先等待消息复制完毕）。延迟时间可以通过参数 replica.lag.time.max.ms 参数配置，它指定了副本在复制消息时可被允许的最大延迟时间。

### 9、Kafka 判断一个节点是否还活着有那两个条件？ 

- 节点必须可以维护和 ZooKeeper 的连接，**Zookeeper 通过心跳机制检查**每 个节点的连接； 
- 如果节点是个 follower,他必须**能及时的同步 leader 的写操作**，延时不能太久。

### 10、producer 是否直接将数据发送到 broker 的 leader（主节点）？ 

producer 直接将数据发送到 broker 的 leader(主节点)，不需要在多个节点进行分发，为了帮助 producer 做到这点，所有的 Kafka 节点都可以及时的告知:哪些节点是活动的，目标 topic 目标分区的 leader 在哪。这样 producer 就可以直接将消息发送到目的地了。

### 11、Kafka consumer 是否可以消费指定分区消息吗？ 

Kafa consumer 消费消息时，向 broker 发出"fetch"请求去消费特定分区的消息，consumer 指定消息在日志中的偏移量（offset），就可以消费从这个位置开始的消息，customer 拥有了 offset 的控制权，可以向后回滚去重新消费之前的消息，这是很有意义的。

### 12、Kafka 高效文件存储设计特点是什么？ 

- Kafka 把 topic 中**一个 parition 大文件分成多个小文件段**，通过多个小文件段，就容易定期清除或删除已经消费完文件，减少磁盘占用。 
- 通过**索引信息**可以快速定位 message 和确定response 的最大大小。 
- 通过 **index 元数据全部映射到 memory**，可以避免 segment file 的 IO  磁盘操作。 
- 通过索引文件稀疏存储，可以大幅降低 index 文件元数据占用空间大小。

### 13、partition 的数据如何保存到硬盘？ 

topic 中的多个 partition 以文件夹的形式保存到 broker，每个分区序号从 0 递增，且消息有序。 Partition 文件下有多个 segment（xxx.index，xxx.log） segment 文件里的 大小和配置文件大小一致可以根据要求修改，默认为 1g。 如果大小大于 1g 时，会滚动一个新的 segment 并且以上一个 segment 最后 一条消息的偏移量命名。

### 14、kafka 生产数据时数据的分组策略是怎样的？ 

生产者决定数据产生到集群的哪个 partition 中，每一条消息都是以（key， value）格式，Key 是由生产者发送数据传入，所以生产者（key）决定了数据 产生到集群的哪个 partition。

### 数据传输的事务有几种？

数据传输的事务定义通常有以下三种级别：

（1）最多一次: 消息不会被重复发送，最多被传输一次，但也有可能一次不传输

（2）最少一次: 消息不会被漏发送，最少被传输一次，但也有可能被重复传输.

（3）精确的一次（Exactly once）: 不会漏传输也不会重复传输,每个消息都传输被

### 15、consumer 是推还是拉？

 customer 应该从 brokes 拉取消息还是 brokers 将消息推送到 consumer，也 就是 pull 还 push。在这方面，Kafka 遵循了一种大部分消息系统共同的传统 的设计：**producer 将消息推送到 broker，consumer 从 broker 拉取消息**。 

push 模式，将消息推送到下游的 consumer。这样做有好处也有坏处：由 broker 决定消息推送的速率，对于不同消费速率的 consumer 就不太好处理 了。消息系统都致力于让 consumer 以最大的速率最快速的消费消息，但不幸 的是，push 模式下，当 broker 推送的速率远大于 consumer 消费的速率时， consumer 恐怕就要崩溃了。最终 Kafka 还是选取了传统的 pull 模式。

### 16、Kafka 维护消费状态跟踪的方法有什么？ 

大部分消息系统在 broker 端的维护消息被消费的记录：一个消息被分发到 consumer 后 broker 就马上进行标记或者等待 customer 的通知后进行标记。 这样也可以在消息在消费后立马就删除以减少空间占用。

### Kafka创建Topic时如何将分区放置到不同的Broker中

- 副本因子不能大于 Broker 的个数；
- 第一个分区（编号为0）的第一个副本放置位置是随机从 brokerList 选择的；
- 其他分区的第一个副本放置位置相对于第0个分区依次往后移。也就是如果我们有5个 Broker，5个分区，假设第一个分区放在第四个 Broker 上，那么第二个分区将会放在第五个 Broker 上；第三个分区将会放在第一个 Broker 上；第四个分区将会放在第二个 Broker 上，依次类推；
  剩余的副本相对于第一个副本放置位置其实是由 nextReplicaShift 决定的，而这个数也是随机产生的
- 具体可以参见 Kafka创建Topic时如何将分区放置到不同的Broker中。

### Kafka新建的分区会在哪个目录下创建

在启动 Kafka 集群之前，我们需要配置好 log.dirs 参数，其值是 Kafka 数据的存放目录，这个参数可以配置多个目录，目录之间使用逗号分隔，通常这些目录是分布在不同的磁盘上用于提高读写性能。
当然我们也可以配置 log.dir 参数，含义一样。只需要设置其中一个即可。
如果 log.dirs 参数只配置了一个目录，那么分配到各个 Broker 上的分区肯定只能在这个目录下创建文件夹用于存放数据。

但是如果 log.dirs 参数配置了多个目录，那么 Kafka 会在哪个文件夹中创建分区目录呢？答案是：Kafka 会在含有分区目录最少的文件夹中创建新的分区目录，分区目录名为 Topic名+分区ID。注意，是分区文件夹总数最少的目录，而不是磁盘使用量最少的目录！也就是说，如果你给 log.dirs 参数新增了一个新的磁盘，新的分区目录肯定是先在这个新的磁盘上创建直到这个新的磁盘目录拥有的分区目录不是最少为止。

### 谈一谈 Kafka 的再均衡

在Kafka中，**当有新消费者加入或者订阅的topic数发生变化时，会触发Rebalance(再均衡**：在同一个消费者组当中，分区的所有权从一个消费者转移到另外一个消费者)机制，Rebalance顾名思义就是重新均衡消费者消费。Rebalance的过程如下：

第一步：所有成员都向coordinator发送请求，请求入组。一旦所有成员都发送了请求，coordinator会从中选择一个consumer担任leader的角色，并把组成员信息以及订阅信息发给leader。

第二步：leader开始分配消费方案，指明具体哪个consumer负责消费哪些topic的哪些partition。一旦完成分配，leader会将这个方案发给coordinator。coordinator接收到分配方案之后会把方案发给各个consumer，这样组内的所有成员就都知道自己应该消费哪些分区了。

所以对于Rebalance来说，Coordinator起着至关重要的作用

### 谈谈 Kafka 分区分配策略

[Kafka分区分配策略(Partition Assignment Strategy)](https://blog.51cto.com/u_15127589/2679415)

### Kafka 是如何实现高吞吐率的？

Kafka是分布式消息系统，需要处理海量的消息，Kafka的设计是把所有的消息都写入速度低容量大的硬盘，以此来换取更强的存储能力，但实际上，使用硬盘并没有带来过多的性能损失。kafka主要使用了以下几个方式实现了超高的吞吐率：
顺序读写；

- 零拷贝
- 文件分段
- 批量发送
- 数据压缩。

### 谈谈你对 Kafka 事务的了解？

参见这篇文章： http://www.jasongj.com/kafka/transaction/

### Kafka 缺点？

- 由于是批量发送，数据并非真正的实时；
- 对于mqtt协议不支持；
- 不支持物联网传感数据直接接入；
- 仅支持同一分区内消息有序，无法实现全局消息有序；
- 监控不完善，需要安装插件；
- 依赖zookeeper进行元数据管理；

### Kafka 分区数可以增加或减少吗？为什么？

我们可以**使用 bin/kafka-topics.sh 命令对 Kafka 增加 Kafka 的分区数据**，但是 Kafka **不支持减少分区数**。

Kafka 分区数据不支持减少是由很多原因的，比如减少的分区其数据放到哪里去？是删除，还是保留？删除的话，那么这些没消费的消息不就丢了。如果保留这些消息如何放到其他分区里面？追加到其他分区后面的话那么就破坏了 Kafka 单个分区的有序性。如果要保证删除分区数据插入到其他分区保证有序性，那么实现起来逻辑就会非常复杂。

