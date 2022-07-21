# Elasticsearch

[官方地址](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)

## 入门

### 介绍

Elasticsearch 是一个开源的搜索引擎，建立在一个全文搜索引擎库 [Apache Lucene™](https://lucene.apache.org/core/) 基础之上。 Lucene 可以说是当下最先进、高性能、全功能的搜索引擎库—无论是开源还是私有。

但是 Lucene 仅仅只是一个库。为了充分发挥其功能，你需要使用 Java 并将 Lucene 直接集成到应用程序中。 更糟糕的是，您可能需要获得信息检索学位才能了解其工作原理。Lucene *非常* 复杂。

Elasticsearch 也是使用 Java 编写的，它的内部使用 Lucene 做索引与搜索，但是它的目的是**使全文检索变得简单**， 通过隐藏 Lucene 的复杂性，取而代之的**提供一套简单一致的 RESTful API**。

然而，Elasticsearch 不仅仅是 Lucene，并且也不仅仅只是一个全文搜索引擎。 它可以被下面这样准确的形容：

- 一**个分布式的实时文档存储**，*每个字段* 可以被索引与搜索
- **一个分布式实时分析搜索引擎**
- 能胜任上百个服务节点的扩展，并支持 PB 级别的结构化或者非结构化数据

Elasticsearch 将所有的功能打包成一个单独的服务，这样你可以通过程序与它提供的简单的 RESTful API 进行通信， 可以使用自己喜欢的编程语言充当 web 客户端，甚至可以使用命令行（去充当这个客户端）。

就 Elasticsearch 而言，起步很简单。对于初学者来说，它预设了一些适当的默认值，并隐藏了复杂的搜索理论知识。 它 ***开箱即用*** 。只需最少的理解，你很快就能具有生产力。

随着你知识的积累，你可以利用 Elasticsearch 更多的高级特性，它的整个引擎是可配置并且灵活的。 从众多高级特性中，挑选恰当去修饰的 Elasticsearch，使它能解决你本地遇到的问题。

### JSON

Elasticsearch 使用 JavaScript Object Notation（或者 [*JSON*](http://en.wikipedia.org/wiki/Json)）作为**文档的序列化格式**。JSON 序列化为大多数编程语言所支持，并且已经成为 NoSQL 领域的标准格式。 它简单、简洁、易于阅读。

下面这个 JSON 文档代表了一个 user 对象：

```js
{
    "email":      "john@smith.com",
    "first_name": "John",
    "last_name":  "Smith",
    "info": {
        "bio":         "Eco-warrior and defender of the weak",
        "age":         25,
        "interests": [ "dolphins", "whales" ]
    },
    "join_date": "2014/05/01"
}
```

虽然原始的 `user` 对象很复杂，但这个对象的结构和含义在 JSON 版本中都得到了体现和保留。在 Elasticsearch 中将对象转化为 JSON 后构建索引要比在一个扁平的表结构中要简单的多。

### 索引

一个 Elasticsearch 集群可以 包含**多个 *索引*** ，相应的**每个索引可以包含多个 *类型*** 。 这些**不同的类型存储着多个 *文档*** ，**每个文档又有 多个 *属性*** 。

**Index Versus Index Versus Index**

你也许已经注意到 ***索引*** 这个词在 Elasticsearch 语境中有多种含义， 这里有必要做一些说明：

> 索引（名词）：
>
> 如前所述，一个 *索引* 类似于传统关系数据库中的一个 *数据库* ，是一个**存储关系型文档的地方**。 *索引* (*index*) 的复数词为 *indices* 或 *indexes* 。
>
> 索引（动词）：
>
> *索引一个文档* 就是**存储一个文档到一个 *索引* （名词）中以便被检索和查询**。这非常类似于 SQL 语句中的 `INSERT` 关键词，除了文档已存在时，新文档会替换旧文档情况之外。
>
> 倒排索引：
>
> 关系型数据库通过增加一个 *索引* 比如一个 B树（B-tree）索引 到指定的列上，以便提升数据检索速度。Elasticsearch 和 Lucene 使用了一个叫做 ***倒排索引*** 的结构来达到相同的目的。
>
> \+ 默认的，一个文档中的每一个属性都是 *被索引* 的（有一个倒排索引）和可搜索的。一个没有倒排索引的属性是不能被搜索到的。我们将在 [倒排索引](https://www.elastic.co/guide/cn/elasticsearch/guide/current/inverted-index.html) 讨论倒排索引的更多细节。

对于员工目录，我们将做如下操作：

- 每个员工索引一个文档，文档包含该员工的所有信息。
- 每个文档都将是 `employee` *类型* 。
- 该类型位于 *索引* `megacorp` 内。
- 该索引保存在我们的 Elasticsearch 集群中。

实践中这非常简单（尽管看起来有很多步骤），我们可以通过一条命令完成所有这些动作：

```sense
PUT /megacorp/employee/1
{
    "first_name" : "John",
    "last_name" :  "Smith",
    "age" :        25,
    "about" :      "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```



拷贝为 curl[在 Sense 中查看](http://localhost:5601/app/sense/?load_from=https://www.elastic.co/guide/cn/elasticsearch/guide/current/snippets/010_Intro/25_Index.json) 

注意，路径 `/megacorp/employee/1` 包含了三部分的信息：

- **`megacorp`**

  索引名称

- **`employee`**

  类型名称

- **`1`**

  特定雇员的ID

请求体 —— JSON 文档 —— 包含了这位员工的所有详细信息，他的名字叫 John Smith ，今年 25 岁，喜欢攀岩。

### 检索文档

目前我们已经在 Elasticsearch 中存储了一些数据， 接下来就能专注于实现应用的业务需求了。第一个需求是可以检索到单个雇员的数据。

这在 Elasticsearch 中很简单。简单地执行 一个 HTTP `GET` 请求并指定文档的地址——索引库、类型和ID。 使用这三个信息可以返回原始的 JSON 文档：

```sense
GET /megacorp/employee/1
```

返回结果包含了文档的一些元数据，以及 `_source` 属性，内容是 John Smith 雇员的原始 JSON 文档：

```js
{
  "_index" :   "megacorp",
  "_type" :    "employee",
  "_id" :      "1",
  "_version" : 1,
  "found" :    true,
  "_source" :  {
      "first_name" :  "John",
      "last_name" :   "Smith",
      "age" :         25,
      "about" :       "I love to go rock climbing",
      "interests":  [ "sports", "music" ]
  }
}
```

### 什么是文档？

在大多数应用中，多数实体或对象可以被序列化为包含键值对的 JSON 对象。 **一个 *键* 可以是一个字段或字段的名称**，一个 *值* 可以是一个字符串，一个数字，一个布尔值， 另一个对象，一些数组值，或一些其它特殊类型诸如表示日期的字符串，或代表一个地理位置的对象：

```js
{
    "name":         "John Smith",
    "age":          42,
    "confirmed":    true,
    "join_date":    "2014-06-01",
    "home": {
        "lat":      51.5,
        "lon":      0.1
    },
    "accounts": [
        {
            "type": "facebook",
            "id":   "johnsmith"
        },
        {
            "type": "twitter",
            "id":   "johnsmith"
        }
    ]
}
```

通常情况下，我们使用的术语 ***对象* 和 *文档* 是可以互相替换**的。不过，有一个区别： 一个对象仅仅是类似于 hash 、 hashmap 、字典或者关联数组的 JSON 对象，对象中也可以嵌套其他的对象。 对象可能包含了另外一些对象。**在 Elasticsearch 中，术语 *文档* 有着特定的含义。它是指最顶层或者根对象, 这个根对象被序列化成 JSON 并存储到 Elasticsearch 中，指定了唯一 ID**。

> 字段的名字可以是任何合法的字符串，但 *不可以* 包含英文句号(.)。

### 文档元数据

一个文档不仅仅包含它的数据 ，也包含 *元数据* —— *有关* 文档的信息。 三个必须的元数据元素如下：

- **`_index`**

  文档在哪存放

- **`_type`**

  文档表示的对象类别

- **`_id`**

  文档唯一标识

#### _index

**一个 *索引* 应该是因共同的特性被分组到一起的文档集合**。 例如，你可能存储所有的产品在索引 `products` 中，而存储所有销售的交易到索引 `sales` 中。 虽然也允许存储不相关的数据到一个索引中，但这通常看作是一个反模式的做法。

实际上，在 Elasticsearch 中，我们的**数据是被存储和索引在 *分片* 中，而一个索引仅仅是逻辑上的命名空间， 这个命名空间由一个或者多个分片组合在一起**。 然而，这是一个内部细节，我们的应用程序根本不应该关心分片，对于应用程序而言，只需知道文档位于一个 *索引* 内。 Elasticsearch 会处理所有的细节。

我们将在 [*索引管理*](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index-management.html) 介绍如何自行创建和管理索引，但现在我们将让 Elasticsearch 帮我们创建索引。 所有需要我们做的就是选择一个索引名，**这个名字必须小写，不能以下划线开头，不能包含逗号**。我们用 `website` 作为索引名举例。

#### _type

数据可能**在索引中只是松散的组合在一起，但是通常明确定义一些数据中的子分区是很有用**的。 例如，所有的产品都放在一个索引中，但是你有许多不同的产品类别，比如 "electronics" 、 "kitchen" 和 "lawn-care"。

这些文档共享一种相同的（或非常相似）的模式：他们有一个标题、描述、产品代码和价格。他们只是正好属于“产品”下的一些子类。

Elasticsearch 公开了一个称为 ***types* （类型）的特性，它允许您在索引中对数据进行逻辑分区**。不同 types 的文档可能有不同的字段，但最好能够非常相似。 我们将在 [类型和映射](https://www.elastic.co/guide/cn/elasticsearch/guide/current/mapping.html) 中更多的讨论关于 types 的一些应用和限制。

一个 `_type` 命名**可以是大写或者小写，但是不能以下划线或者句号开头，不应该包含逗号， 并且长度限制为256个字符**. 我们使用 `blog` 作为类型名举例。

#### _id

*ID* 是一个字符串，当它和 `_index` 以及 `_type` 组合就可以**唯一确定 Elasticsearch 中的一个文档**。 当你创建一个新的文档，要么提供自己的 `_id` ，要么让 Elasticsearch 帮你生成。

#### 其他元数据

还有一些其他的元数据元素，他们在 [类型和映射](https://www.elastic.co/guide/cn/elasticsearch/guide/current/mapping.html) 进行了介绍。通过前面已经列出的元数据元素， 我们已经能存储文档到 Elasticsearch 中并通过 ID 检索它—换句话说，使用 Elasticsearch 作为文档的存储介质。

在 Elasticsearch 中每个文档都有一个版本号。当每次对文档进行修改时（包括删除）， `_version` 的值会递增。 在 [处理冲突](https://www.elastic.co/guide/cn/elasticsearch/guide/current/version-control.html) 中，我们讨论了怎样使用 `_version` 号码确保你的应用程序中的一部分修改不会覆盖另一部分所做的修改。

`_source` 字段，这个字段包含我们索引数据时发送给 Elasticsearch 的原始 JSON 文档

### 更新文档

在 Elasticsearch 中文档是 *不可改变* 的，不能修改它们。相反，**如果想要更新现有的文档，需要 *重建索引* 或者进行替换**， 我们可以使用相同的 `index` API 进行实现，在 [索引文档](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index-doc.html) 中已经进行了讨论。

```sense
PUT /website/blog/123
{
  "title": "My first blog entry",
  "text":  "I am starting to get the hang of this...",
  "date":  "2014/01/02"
}
```

在响应体中，我们能看到 Elasticsearch 已经增加了 `_version` 字段值：

```js
{
  "_index" :   "website",
  "_type" :    "blog",
  "_id" :      "123",
  "_version" : 2,
  "created":   false 
}
```

> `created` 标志设置成 `false` ，是因为相同的索引、类型和 ID 的文档已经存在。

**在内部，Elasticsearch 已将旧文档标记为已删除，并增加一个全新的文档。 尽管你不能再对旧版本的文档进行访问，但它并不会立即消失**。当继续索引更多的数据，Elasticsearch 会在后台清理这些已删除文档。

在本章的后面部分，我们会介绍 `update` API, 这个 API 可以用于 [partial updates to a document](https://www.elastic.co/guide/cn/elasticsearch/guide/current/partial-updates.html) 。 虽然它似乎对文档直接进行了修改，但实际上 Elasticsearch 按前述完全相同方式执行以下过程：

1. 从旧文档构建 JSON
2. 更改该 JSON
3. 删除旧文档
4. 索引一个新文档

唯一的区别在于, `update` API 仅仅通过一个客户端请求来实现这些步骤，而不需要单独的 `get` 和 `index` 请求。

### 搜索——最基本的工具

我们可以将一个 JSON 文档扔到 Elasticsearch 里，然后根据 ID 检索。但 Elasticsearch 真正强大之处在于可以从无规律的数据中找出有意义的信息——从“大数据”到“大信息”。

Elasticsearch 不只会_存储（stores）_ 文档，为了能被搜索到也会为文档添加_索引（indexes）_ ，这也是为什么我们使用结构化的 JSON 文档，而不是无结构的二进制数据。

*文档中的每个字段都将被索引并且可以被查询* 。不仅如此，在简单查询时，Elasticsearch 可以使用 *所有（all）* 这些索引字段，以惊人的速度返回结果。这是你永远不会考虑用传统数据库去做的一些事情。

*搜索（search）* 可以做到：

- 在**类似于 `gender` 或者 `age` 这样的字段上使用结构化查询**，**`join_date` 这样的字段上使用排序**，就像SQL的结构化查询一样。
- 全文检索，**找出所有匹配关键字的文档并按照_相关性（relevance）_ 排序后返回结果**。
- 以上二者兼而有之。

很多搜索都是开箱即用的，为了充分挖掘 Elasticsearch 的潜力，你需要理解以下三个概念：

- **映射（Mapping）**

  描述数据在每个字段内如何存储

- **分析（Analysis）**

  全文是如何处理使之可以被搜索的

- **领域特定查询语言（Query DSL）**

  Elasticsearch 中强大灵活的查询语言

以上提到的每个点都是一个大话题，我们将在 [深入搜索](https://www.elastic.co/guide/cn/elasticsearch/guide/current/search-in-depth.html) 一章详细阐述它们。本章节我们将介绍这三点的一些基本概念——仅仅帮助你大致了解搜索是如何工作的。

我们将使用最简单的形式开始介绍 `search` API。

## 简单使用

[原地址](https://cloud.tencent.com/developer/article/1911255)、[原地址](https://www.modb.pro/db/147001)

在`Go`

语言中经常使用的包有以下两个:

| 包                       | 文档                                        | Star 数量 | 说明       |
| :----------------------- | :------------------------------------------ | :-------- | :--------- |
| olivere/elastic          | https://olivere.github.io/elastic/          | 6.1k      | 社区开源   |
| elastic/go-elasticsearch | https://github.com/elastic/go-elasticsearch | 3.5k      | ES官方提供 |

### [Elasticsearch Go Client](https://www.elastic.co/guide/en/elasticsearch/client/go-api/current/index.html)

[API 文档](https://pkg.go.dev/github.com/elastic/go-elasticsearch?utm_source=godoc)

### 下载

```
# 安装v7的版本
go get github.com/olivere/elastic/v7
```



### 创建客户端

```
package test

import (
 "context"
 "fmt"
 "github.com/olivere/elastic/v7"
 "log"
 "os"
 "testing"
 "time"
)

// 连接Es
func connectEs() (*elastic.Client, error) {
 return elastic.NewClient(
  // 设置Elastic服务地址
  elastic.SetURL("http://127.0.0.1:9200"),
  // 是否转换请求地址，默认为true,当等于true时 请求http://ip:port/_nodes/http，将其返回的url作为请求路径
  elastic.SetSniff(false),
  // 心跳检查,间隔时间
  elastic.SetHealthcheckInterval(time.Second*5),
  // 设置错误日志
  elastic.SetErrorLog(log.New(os.Stderr, "ES-ERROR ", log.LstdFlags)),
  // 设置info日志
  elastic.SetInfoLog(log.New(os.Stdout, "ES-INFO ", log.LstdFlags)),
 )
}
// 测试连接
func TestConnectES(t *testing.T) {
 client, err := connectEs()
 if err != nil {
  t.Error(err)
  return
 }
 // 健康检查
 do, _ := client.ClusterHealth().Index().Do(context.TODO())
 fmt.Println("健康检查:",do)
}
/** 输出
=== RUN   TestConnectES
ES-ERROR 2021/07/04 11:41:02 Deprecation warning: 299 Elasticsearch-7.13.3-5d21bea28db1e89ecc1f66311ebdec9dc3aa7d64 "Elasticsearch built-in security features are not enabled. Without authentication, your cluster could be accessible to anyone. See https://www.elastic.co/guide/en/elasticsearch/reference/7.13/security-minimal-setup.html to enable security."
ES-INFO 2021/07/04 11:41:02 GET http://127.0.0.1:9200/_cluster/health [status:200, request:0.007s]
健康检查: &{laradock-cluster yellow false 1 1 8 8 0 0 1 0 0 0  0  88.88888888888889 map[]}
--- PASS: TestConnectES (0.02s)
PASS
*/
```

>@注意：如果你的ElasticSearch是通过docker安装，若不设置`elastic.SetSniff(false)`
>，会报错: `no active connection found: no Elasticsearch node available`

### 创建索引

```go
// 创建索引(指定mapping)
func TestCreateIndexMapping(t *testing.T) {
 userMapping := `{
    "mappings":{
        "properties":{
            "name":{
                "type":"keyword"
            },
            "age":{
                "type":"byte"
            },
            "birth":{
                "type":"date"
            }
        }
    }
}`
 client, _ := connectEs()
 // 检测索引是否存在
 indexName := "go-test"
 // 创建上下文
 ctx := context.Background()
 exist, err := client.IndexExists(indexName).Do(ctx)
 if err != nil {
  t.Errorf("检测索引失败:%s", err)
  return
 }
 if exist {
  t.Error("索引已经存在，无需重复创建！")
  return
 }
 res, err := client.CreateIndex(indexName).BodyString(userMapping).Do(ctx)
 if exist {
  t.Errorf("创建索引失败:%s", err)
  return
 }
 fmt.Println("创建成功:", res)
}
/**输出
=== RUN   TestCreateIndexMapping
创建成功: &{true true go-test}
--- PASS: TestCreateIndexMapping (0.13s)
PASS
*/
```

**如果想直接创建索引，只需删除`BodyString(userMapping)`,如下:**

```
// 指定userMapping创建
res, err := client.CreateIndex(indexName).BodyString(userMapping).Do(ctx)
// 直接创建
res, err := client.CreateIndex(indexName).Do(ctx)
```

### 添加数据

#### 1. 单条添加

```
type UserInfo struct {
 Name  string `json:"name"`
 Age   int    `json:"age"`
 Birth string `json:"birth"`
}

// 单条添加
func TestAddOne(t *testing.T) {
 client, _ := connectEs()
 ctx := context.Background()
 // 创建userInfo
 userInfo := UserInfo{
  Name:  "张三",
  Age:   18,
  Birth: "1991-03-04",
 }
 res, err := client.Index().Index("go-test").Id("1").BodyJson(userInfo).Do(ctx)
 if err != nil {
  t.Errorf("添加失败:%s",err)
 }
 fmt.Println("添加成功",res)
}
/**输出
=== RUN   TestAddOne
添加成功 &{go-test _doc 1 1 created 0xc000212100 0 1 0 false}
--- PASS: TestAddOne (0.01s)
PASS
*/
```

#### 2. 批量添加

```
// 批量添加
func TestBatchAdd(t *testing.T) {
 client, _ := connectEs()
 ctx := context.Background()
 // 创建用户
 userNames := map[string]string{
  "李四": "1992-04-25",
  "张亮": "1994-07-15",
  "小明": "1991-12-03",
 }
 rand.Seed(time.Now().Unix())
 // 创建bulk
 userBulk := client.Bulk().Index("go-test")
 id := 4
 for n, b := range userNames {
  userTmp := UserInfo{Name: n, Age: rand.Intn(50), Birth: b}
  // 批量添加到bulk
  doc := elastic.NewBulkIndexRequest().Id(strconv.Itoa(id)).Doc(userTmp)
  userBulk.Add(doc)
  id++
 }
 // 检查被添加数据是否为空
 if userBulk.NumberOfActions() < 1 {
  t.Error("被添加的数据不能为空！")
  return
 }
 // 保存
 res, err := userBulk.Do(ctx)
 if err != nil {
  t.Errorf("保存失败:%s", err)
  return
 }
 fmt.Println("保存成功: ", res)
}
/** 输出
=== RUN   TestBatchAdd
保存成功:  &{3 false [map[index:0xc000136100] map[index:0xc000136180] map[index:0xc000136200]]}
--- PASS: TestBatchAdd (0.01s)
PASS
```

### 单条更新

#### 1. 单字段更新(`Script` )

```
// 通过Script方式更新
func TestUpdateOneByScript(t *testing.T) {
 client, _ := connectEs()
 ctx := context.Background()

 // 根据id更新
 res, err := client.Update().Index("go-test").Id("1").
  Script(elastic.NewScript("ctx._source.birth='1999-09-09'")).Do(ctx)
 if err != nil {
  t.Errorf("根据ID更新单条记录失败:%s", err)
  return
 }
 fmt.Println("根据ID更新成功:", res.Result)
 
 // 根据条件更新, update .. where name = '阿三'
 res2, err := client.UpdateByQuery("go-test").Query(elastic.NewTermQuery("name", "小明")).
  Script(elastic.NewScript("ctx._source.age=22")).ProceedOnVersionConflict().Do(ctx)
 if err != nil {
  t.Errorf("根据条件更新单条记录失败:%s", err)
  return
 }
 fmt.Println("根据条件更新成功:", res2.Updated)
}
/**输出
=== RUN   TestUpdateOneByScript
根据ID更新成功: updated
根据条件更新成功: 1
--- PASS: TestUpdateOneByScript (0.02s)
PASS
*/
```

#### 2. 多字段更新(`doc` )

```
// 使用Doc更新多个字段
func TestUpdateOneByDoc(t *testing.T) {
 client, _ := connectEs()
 ctx := context.Background()
 res, _ := client.Update().Index("go-test").Id("5").Doc(map[string]interface{}{
  "name": "小白", "age": 30,
 }).Do(ctx)
 fmt.Println("更新结果:", res.Result)
}
/**输出
=== RUN   TestUpdateOneByDoc
更新结果: updated
--- PASS: TestUpdateOneByDoc (0.01s)
PASS
*/
```

### 批量更新

```
// 批量修改
func TestBatchUpdate(t *testing.T) {
 client,_ := connectEs()
 ctx := context.Background()
 bulkReq := client.Bulk().Index("go-test")
 for _, id := range []string{"4","5","6","7"} {
  doc := elastic.NewBulkUpdateRequest().Id(id).Doc(map[string]interface{}{"age": 18})
  bulkReq.Add(doc)
 }
 // 被更新的数量不能小于0
 if bulkReq.NumberOfActions() < 0 {
  t.Error("被更新的数量不能为空")
  return
 }
 // 执行操作
 do, err := bulkReq.Do(ctx)
 if err != nil {
  t.Errorf("批量更新失败:%v",err)
  return
 }
 fmt.Println("更新成功:",do.Updated())
}
/**输出
=== RUN   TestBatchUpdate
更新成功: [0xc000266000 0xc000266080 0xc000266100 0xc000266180]
--- PASS: TestBatchUpdate (0.01s)
PASS
*/
```

### 查询

#### 1. 单条查询

```
// 查询单条
func TestSearchOneEs(t *testing.T) {
 client,_ := connectEs()
 ctx := context.Background()
 // 查找一条
 getResult, err := client.Get().Index("go-test").Id("1").Do(ctx)
 if err != nil {
  t.Errorf("获取失败: %s",err)
  return
 }
 // 提取查询结果(json格式)
 json, _ := getResult.Source.MarshalJSON()
 fmt.Printf("查询单条结果:%s \n",json)
}
/**输出
=== RUN   TestSearchEs
结果:{"name":"阿三","birth":"1999-09-09","age":20} 
--- PASS: TestSearchEs (0.01s)
PASS
*/
```

#### 2. 批量查询

```
// 查询多条
func TestSearchMoreES(t *testing.T) {
 client,_ := connectEs()
 ctx := context.Background()
 searchResult, err := client.Search().Index("go-test").
  Query(elastic.NewMatchQuery("age", 18)).
  From(0). //从第几条开始取
  Size(10). // 取多少条
  Pretty(true).
  Do(ctx)
 if err != nil {
  t.Errorf("获取失败: %s",err)
  return
 }
 // 定义用户结构体
 var userList []UserInfo
 for _, val := range searchResult.Each(reflect.TypeOf(UserInfo{})) {
  tmp := val.(UserInfo)
  userList = append(userList,tmp)
 }
 fmt.Printf("查询结果:%v\n",userList)
}
/**输出
=== RUN   TestSearchMoreES
查询结果:[{小明 18 1991-12-03} {小白 18 1995-11-11} {李四 18 1992-04-25} {李亮 18 1994-07-15}]
--- PASS: TestSearchMoreES (0.01s)
PASS
*/
```

### 删除

#### 1. 根据ID删除

```
//  根据ID删除
func TestDelById(t *testing.T) {
 client, _ := connectEs()
 ctx := context.Background()
 // 根据ID删除
 do, err := client.Delete().Index("go-test").Id("1").Do(ctx)
 if err != nil {
  t.Errorf("删除失败:%s",err)
  return
 }
 fmt.Println("删除成功: ",do.Result)
}
/**输出
=== RUN   TestDelById
删除成功:  deleted
--- PASS: TestDelById (0.02s)
PASS
*/
```

#### 2. 根据条件删除

```
// 根据条件删除
func TestDelByWhere(t *testing.T) {
 client, _ := connectEs()
 ctx := context.Background()
 // 根据条件删除
 do, err := client.DeleteByQuery("go-test").Query(elastic.NewTermQuery("age", 18)).
  ProceedOnVersionConflict().Do(ctx)
 if err != nil {
  t.Errorf("删除失败:%s",err)
  return
 }
 fmt.Println("删除成功: ",do.Deleted)
}
/**输出
=== RUN   TestDelByWhere
删除成功:  4
--- PASS: TestDelByWhere (0.02s)
PASS
*/
```

## 基本查询

### 前言

搜索是 ES 最为复杂精妙的地方，这里只示例项目中较为常用的查询。

ES 中的条件查询常用的有如下几种：

- TermQuery 精确匹配单个字段
- TermsQuery 精确匹配单个字段，但使用多值进行匹配，类似于 SQL 中的 in 操作
- MatchQuery 单个字段匹配查询（匹配分词结果，不需要全文匹配）
- RangeQuery 范围查询
- BoolQuery 组合查询

### 根据 ID 查询

根据文档ID获取单个文档信息。

```javascript
// GetByID4ES 根据ID查询单个文档
func GetByID4ES(ctx context.Context, index, id string) (string, error) {
	res, err := GetESClient().Get().Index(index).Id(id).Do(ctx)
	if err != nil {
		return "", err
	}
	return string(res.Source), nil
}
```

**注意：查询不存在的 ID，会报`elastic: Error 404 (Not Found)`错误。**

对应的 [RESTful api](https://cloud.tencent.com/product/slshttp?from=10680) 为：

```javascript
GET /es_index_userinfo/_doc/1
```

![img](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/7d68ec4b4f57504b996a9ac38c5b1a04.png)

 如果只想返回部分字段，可以使用`_source_includes`或`_source_excludes`参数来包括或过滤掉特定字段。

例如不返回创建时间（create_time） 和更新时间（update_time），支持通配符。

```javascript
GET /es_index_userinfo/_doc/1?_source_includes=*&_source_excludes=*time
```

### 精确匹配单个字段

比如获指定用户名的用户。

```javascript
// 创建 term 查询条件，用于精确查询
termQuery := elastic.NewTermQuery("username", "cat")
searchResult, err := GetESClient().Search().
	Index("es_index_userinfo"). 			// 设置索引名
	Query(termQuery).           			// 设置查询条件
	Sort("create_time", true).    			// 设置排序字段，根据 create_time 字段升序排序
	From(0).                    			// 设置分页参数 - 起始偏移量，从第 0 行记录开始
	Size(10).                   			// 设置分页参数 - 每页大小
	Do(ctx)                     			// 执行请求
```

对应的 RESTful api 为：

```javascript
GET /es_index_userinfo/_search
{
  "query": {
    "term": {"username": "bob"}
  },
  "sort": [
    {"create_time": "asc"}
  ],
  "from": 0,
  "size":10
}
```

**注意：** term 精确匹配 text 类型的字段可能匹配不到，因为 text 类型的字段会被分词，如果分词的结果中不包含整个字段内容，那么将无法匹配，因为 term 匹配是和分词的结果匹配。keyword 类型字段不会进行分词，所以可以用 term 进行精确匹配。

**解决办法**：给 text 类型的字段取一个别名，别名的类型为 keyword，即不进行分词。

```javascript
"ancestral":{                 
    "type": "text",         
    "fields": {             
      "alias": {          
        "type": "keyword"
      }
    }
}
```

那么可以通过 ancestral.alias 访问字段 ancestral，其类型设为 keyword。

### 精确匹配单个字段的多个值

通过 TermsQuery 实现单个字段的多值精确匹配，类似于 SQL 的 in 查询。

比如获指定用户名的用户，只需要命中一个即可。

```javascript
// 创建 terms 查询条件，用于多值精确查询
termsQuery := elastic.NewTermsQuery("username", "cat", "bob")
searchResult, err := GetESClient().Search().
	Index("es_index_userinfo"). 			// 设置索引名
	Query(termsQuery).           			// 设置查询条件
	Sort("create_time", true).    			// 设置排序字段，根据 create_time 字段升序排序
	From(0).                    			// 设置分页参数 - 起始偏移量，从第 0 行记录开始
	Size(10).                   			// 设置分页参数 - 每页大小
	Do(ctx)                     			// 执行请求
```

对应的 RESTful api 为：

```javascript
GET /es_index_userinfo/_search
{
  "query": {
    "terms": {"username": ["bobs","bob"]}
  },
  "sort": [
    {"create_time": "asc"}
  ],
  "from": 0,
  "size":10
}
```

### 全文查询

全文查询 [Full text queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/full-text-queries.html) 是个 ES 的核心查询。

无论需要查询什么字段， MatchQuery 查询都应该会是首选的查询方式。它是一个高级全文查询 ，这表示它既能处理全文字段，又能处理精确字段。

使用 MatchQuery 对字段进行[全文搜索](https://cloud.tencent.com/product/es?from=10680)，即匹配分词结果。如果分词出现在 MatchQuery 中指定的内容（指定的内容也会分词），如果存在相同的分词，则匹配。

假设“我爱中国”的分词结果为“我”、“爱”、“中国”，那么搜索“我是第一名”也会匹配，因为“我是第一名”的分词结果中也有“我”。

ES 查看某个字段数据的分词结果。

```javascript
GET /{index}/{type}/{id}/_termvectors?fields={fields_name}
```

**注意：** （1）如果想对输入不进行分词，请使用 [term query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-term-query.html)； （2）如果想对输入的分词结果全部匹配，请使用 [match phrase query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query-phrase.html)； （3）如果想对输入的分词结果全部匹配且最后一个分词支持前缀匹配，请使用 [match phrase prefix query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query-phrase-prefix.html)； （4）如果是对 keyword 字段进行 MatchQuery，因为该类型不会分词，所以是精确匹配。

比如获取指定用户名的用户。

```javascript
// 创建 match 查询条件
matchQuery := elastic.NewMatchQuery("username", "bob")
searchResult, err := GetESClient().Search().
	Index("es_index_userinfo"). // 设置索引名
	Query(matchQuery).          // 设置查询条件
	Sort("create_time", true).  // 设置排序字段，根据 create_time 字段升序排序
	From(0).                    // 设置分页参数 - 起始偏移量，从第 0 行记录开始
	Size(10).                   // 设置分页参数 - 每页大小
	Do(ctx)                     // 执行请求
```

对应的 RESTful api 为：

```javascript
GET /es_index_userinfo/_search
{
  "query": {
    "match": {"username": "bob"}
  },
  "sort": [
    {"create_time": "asc"}
  ],
  "from": 0,
  "size":10
}
```

### 范围查询

实现类似`age >= 18 and age < 35`的范围查询条件。

```javascript
// 创建 range 查询条件
rangeQuery := elastic.NewRangeQuery("age").Gte(18).Lte(35)
searchResult, err := GetESClient().Search().
	Index("es_index_userinfo"). // 设置索引名
	Query(rangeQuery).          // 设置查询条件
	Sort("create_time", true).  // 设置排序字段，根据 create_time 字段升序排序
	From(0).                    // 设置分页参数 - 起始偏移量，从第 0 行记录开始
	Size(10).                   // 设置分页参数 - 每页大小
	Do(ctx)                     // 执行请求
```

对应的 RESTful api 为：

```javascript
GET /es_index_userinfo/_search
{
  "query": {
    "range":{"age" : {"gte" : 18, "lte": 35}}
  },
  "sort": [
    {"create_time": "asc"}
  ],
  "from": 0,
  "size":10
}
```

### bool 组合查询

BoolQuery 是一种组合查询，将多个条件通过类似 SQL 语句 and 和 or 组合在一起来作为查询条件。

其有四种类型的子句：

| 类型     | 描述                                                         |
| :------- | :----------------------------------------------------------- |
| must     | 条件必须要满足，并将对分数起作用                             |
| filter   | 条件必须要满足，但又不同于 must 子句，在 filter context 中执行，这意味着忽略评分，并考虑使用缓存。效率会高于 must |
| should   | 条件应该满足。可以通过 minimum_should_match 参数指定应该满足的条件个数。如果 bool 查询包含 should 子句，并且没有 must 和 filter 子句，则默认值为 1，否则默认值为 0 |
| must_not | 条件必须不能满足。在 filter context 中执行，这意味着评分被忽略，并考虑使用缓存。因为评分被忽略，所以会返回所有 0 分的文档 |

#### must

类似 SQL 的 and，代表必须匹配的条件。

```javascript
	// 创建 bool 查询
	boolQuery := elastic.NewBoolQuery()

	// 创建查询条件
	termQuery := elastic.NewTermQuery("username", "bob")
	rangeQuery := elastic.NewRangeQuery("age").Gte(18).Lte(35)

	// 设置 bool 查询的 must 条件, 组合了两个子查询
	// 搜索用户名为 bob 且年龄在 18～35 岁的用户
	boolQuery.Must(termQuery, rangeQuery)

	searchResult, err := GetESClient().Search().
		Index("es_index_userinfo"). // 设置索引名
		Query(boolQuery).           // 设置查询条件
		Sort("create_time", true).  // 设置排序字段，根据 create_time 字段升序排序
		From(0).                    // 设置分页参数 - 起始偏移量，从第 0 行记录开始
		Size(10).                   // 设置分页参数 - 每页大小
		Do(ctx)                     // 执行请求
```

对应的 RESTful api 为：

```javascript
GET /es_index_userinfo/_search
{
  "query":{
    "bool":{
      "must":[
        {"term":{"username": "bob"}},
        {"range":{"age":{"gte":18, "lte":35}}}
      ]
    }
  },
  "sort": [
    {"create_time": "asc"}
  ],
  "from": 0,
  "size":10
}
```

#### filter

类似 SQL 的 and，代表必须匹配的条件。不计算匹配分值，且子句被考虑用于缓存。

使用 filter 替代 must 条件，查询用户名为 bob 且年龄在 18～35 岁的用户

```javascript
	// 创建 bool 查询
	boolQuery := elastic.NewBoolQuery()

	// 创建查询条件
	termQuery := elastic.NewTermQuery("username", "bob")
	rangeQuery := elastic.NewRangeQuery("age").Gte(18).Lte(35)

	// 设置 bool 查询的 filter 条件, 组合了两个子查询
	// 搜索用户名为 bob 且年龄在 18～35 岁的用户
	boolQuery.Filter(termQuery, rangeQuery)

	searchResult, err := GetESClient().Search().
		Index("es_index_userinfo"). // 设置索引名
		Query(boolQuery).          // 设置查询条件
		Sort("create_time", true).  // 设置排序字段，根据 create_time 字段升序排序
		From(0).                    // 设置分页参数 - 起始偏移量，从第 0 行记录开始
		Size(10).                   // 设置分页参数 - 每页大小
		Do(ctx)                     // 执行请求
```

对应的 RESTful api 为：

```javascript
GET /es_index_userinfo/_search
{
  "query":{
    "bool":{
      "filter":[
        {"term":{"username": "bob"}},
        {"range":{"age":{"gte":18, "lte":35}}}
      ]
    }
  },
  "sort": [
    {"create_time": "asc"}
  ],
  "from": 0,
  "size":10
}
```

#### should

类似 SQL 中的 or， 可以通过 minimum_should_match 参数指定应该满足的条件个数。如果 bool 查询包含 should 子句，并且没有 must 和 filter 子句，则默认值为 1，否则默认值为 0。

比如查询用户名为 bob 且年龄为18 或 35 岁的用户。

```javascript
// 创建 bool 查询
boolQuery := elastic.NewBoolQuery()

// 创建查询条件
termQuery := elastic.NewTermQuery("username", "bob")
termQuery1 := elastic.NewTermQuery("age", 18)
termQuery2 := elastic.NewTermQuery("age", 35)

// 设置 bool 查询的 filter 条件, 组合了两个子查询
// 搜索用户名为 bob 且年龄为 18 或 35 岁的用户
boolQuery.Filter(termQuery, termQuery)
boolQuery.Should(termQuery, termQuery1, termQuery2)
boolQuery.MinimumNumberShouldMatch(1) // 至少满足 should 中的一个条件

searchResult, err := GetESClient().Search().
	Index("es_index_userinfo"). // 设置索引名
	Query(boolQuery).           // 设置查询条件
	Sort("create_time", true).  // 设置排序字段，根据 create_time 字段升序排序
	From(0).                    // 设置分页参数 - 起始偏移量，从第 0 行记录开始
	Size(10).                   // 设置分页参数 - 每页大小
	Do(ctx)                     // 执行请求
```



对应的 RESTful api 为：

```javascript
GET /es_index_userinfo/_search
{
  "query":{
    "bool":{
      "filter": {"term":{"username": "bob"}},
      "should":[
        {"term":{"age":18}},
        {"term":{"age":35}}
      ],
      "minimum_should_match" : 1
    }
  },
  "sort": [
    {"create_time": "asc"}
  ],
  "from": 0,
  "size":10
}
```



#### must_not

跟 must 作用相反，表示条件必须不能满足。

比如搜索用户名为 bob 且年龄不为 18 或 35 岁的用户。

```javascript
	// 创建 bool 查询
	boolQuery := elastic.NewBoolQuery()

	// 创建查询条件
	termQuery := elastic.NewTermQuery("username", "bob")
	termQuery1 := elastic.NewTermQuery("age", 18)
	termQuery2 := elastic.NewTermQuery("age", 35)

	// 设置 bool 查询的 filter 条件, 组合了两个子查询
	// 搜索用户名为 bob 且年龄不为 18 和 35 岁的用户
	boolQuery.Filter(termQuery)
	boolQuery.MustNot(termQuery1, termQuery2)

	searchResult, err := GetESClient().Search().
		Index("es_index_userinfo"). // 设置索引名
		Query(boolQuery).           // 设置查询条件
		Sort("create_time", true).  // 设置排序字段，根据 create_time 字段升序排序
		From(0).                    // 设置分页参数 - 起始偏移量，从第 0 行记录开始
		Size(10).                   // 设置分页参数 - 每页大小
		Do(ctx)                     // 执行请求
```



对应的 RESTful api 为：

```javascript
GET /es_index_userinfo/_search
{
  "query":{
    "bool":{
      "filter": {"term":{"username": "bob"}},
      "must_not":[
        {"term":{"age":18}},
        {"term":{"age":35}}
      ]
    }
  },
  "sort": [
    {"create_time": "asc"}
  ],
  "from": 0,
  "size":10
}
```



### 分页查询

我们也可以根据条件分页查询。

ES 分页搜索一般有三种方案，from + size、search after、scroll api，这三种方案分别有自己的优缺点。

#### from + size

这是 ES 分页中最常用的一种方式，与 [MySQL](https://cloud.tencent.com/product/cdb?from=10680) 类似，from 指定起始位置，size 指定返回的文档数。

这种分页方式，在分布式的环境下的深度分页是有性能问题的，一般不建议用这种方式做深度分页，可以用下面将要介绍的两种方式。

理解为什么深度分页是有问题的，假设取的页数较大时（深分页），如请求第20页，Elasticsearch 不得不取出所有分片上的第 1 页到第 20 页的所有文档，并做排序，最终再取出 from 后的 size 条结果作爲最终的返回值。

所以，当索引记录非常非常多(千万或亿)，是无法使用 from + size 做深分页的，分页越深则越容易 OOM。即便不 OOM，也很消耗 CPU 和内存资源。

所以 ES 为了避免深分页，不允许使用 from + size 的方式查询 1 万条以后的数据，即 from + size 大于 10000 会报错，不过可以通过 index.max_result_window 参数进行修改。

```javascript
// GetByQueryPage4ES 分页查询
// param: index 索引; query 查询条件; page 起始页（从 1 开始）; size 页大小
func GetByQueryPage4ES(ctx context.Context, index string, query elastic.Query, page, size int) ([]string, error) {
	start := (page - 1) * size
	res, err := GetESClient().Search(index).Query(query).From(start).Size(size).Do(ctx)
	if err != nil {
		return nil, err
	}
	sl := make([]string, 0, res.TotalHits())
	for _, hit := range res.Hits.Hits {
		sl = append(sl, string(hit.Source))
	}
	return sl, nil
}

// GetByQueryPageSort4ES 根据条件分页查询 & 指定字段排序
// param: index 索引; query 查询条件; page 起始页（从 1 开始）; size 页大小; field 排序字段; ascending 升序
func GetByQueryPageSort4ES(ctx context.Context, index string, query elastic.Query, page, size int, field string,
	ascending bool) ([]string, error) {
	from := (page - 1) * size
	res, err := GetESClient().Search(index).Query(query).Sort(field, ascending).From(from).Size(size).Do(ctx)
	if err != nil {
		return nil, err
	}
	sl := make([]string, 0, res.TotalHits())
	for _, hit := range res.Hits.Hits {
		sl = append(sl, string(hit.Source))
	}
	return sl, nil
}
```



比如分页查询年龄 >=18 且按照创建时间降序排序：

```javascript
query := elastic.NewBoolQuery()
query.Filter(elastic.NewRangeQuery("age").Gte(18))
sl, err := GetByQueryPageSort4ES(context.Background(), index, query, 1, 500, "create_time", false)
```



对应的 RESTful api 为：

```javascript
GET /es_index_userinfo/_search
{
	"query": {
		"bool": {
			"filter":[{"range" : {"age" : {"gte" : 18}}}]
		}
	},
	"from": 0, 
	"size" : 500,
	"sort" : [{"create_time":"desc"}]
}
```



注意：如果想控制返回哪些字段，可以使用 _source 来指定。比如只返回用户名（username）和年龄（age）。

```javascript
GET /es_index_userinfo/_search
{
	"query": {
		"bool": {
			"filter":[{"range" : {"age" : {"gte" : 18}}}]
		}
	},
	"from": 0, 
	"size" : 500,
	"sort" : [{"create_time":"desc"}],
	"_source": ["username", "age"]
}
```



Go 代码带上`_source`的方式。

```javascript
fsc := elastic.NewFetchSourceContext(true).Include("username", "age")
GetESClient().Search(index).Query(query).FetchSourceContext(fsc).Sort(field, ascending).From(from).Size(size).Do(ctx)
```



#### search after

search after 利用实时有游标来帮我们解决实时滚动的问题。

**第一次搜索时需要指定 sort，并且保证值是唯一的，可以通过加入 _id 保证唯一性。**

比如获取籍贯为安徽的用户，且按照创建时间降序。

```javascript
matchQuery := elastic.NewMatchQuery("ancestral", "安徽")

// 查询第一页（无需指定 SearchAfter）
searchResult, err := GetESClient().Search().
	Index("es_index_userinfo"). // 设置索引名
	Query(matchQuery).          // 设置查询条件
	Sort("create_time", false). // 按照创建时间降序
	Size(1000).                 // 设置分页参数
	Do(ctx)

// 查询第 n 页（n > 1，需要指定 SearchAfter）
searchResult, err := GetESClient().Search().
	Index("es_index_userinfo").  		// 设置索引名
	Query(matchQuery).           		// 设置查询条件
	Sort("create_time", false).  		// 按照创建时间降序
	SearchAfter(lastCreateTime, id). 	// 上页最后一条的创建时间和 ID
	Size(1000).                  		// 设置分页参数
	Do(ctx)
```



对应 RESTful api 的示例。

```javascript
GET es_index_userinfo/_search
{
  "size": 1,
  "query": {
    "match": {"ancestral": "安徽"}
  },
  "sort": [
    {"create_time": "desc"},
    {"_id": "desc"}
  ]
}
```



在返回的结果中，最后一个文档有类似下面的数据，由于我们排序用的是两个字段，返回的是两个值。

```javascript
"sort" : [
	1627522828,
	"2"
]
```



第二次搜索，带上这个 sort 信息即可，如下：

```javascript
GET es_index_userinfo/_search
{
  "size": 1,
  "query": {
    "match": {"ancestral": "安徽"}
  },
  "sort": [
    {"create_time": "desc"},
    {"_id": "desc"}
  ],
  "search_after": [
    1627522828,
    "2"
  ]
}
```



#### scroll api

创建一个快照，有新的数据写入以后，无法被查到。每次查询后，输入上一次的 scroll_id。目前官方已经不推荐使用这个 API 了，使用search_after 即可。

Go 代码示例：

```javascript
// GetByQueryPageSortScroll4ES 获取第一页数据 & 获取游标ID
// ret: 文档切片, 游标ID, error
func GetByQueryPageSortScroll4ES(ctx context.Context, index string, query elastic.Query, size int, field string,
ascending bool) ([]string, string, error) {
	res, err := GetESClient().Scroll(index).Query(query).Sort(field, ascending).Size(size).Do(ctx)
	if err != nil {
		return nil, "", err
	}
	sl := make([]string, 0, res.TotalHits())
	for _, hit := range res.Hits.Hits {
		sl = append(sl, string(hit.Source))
	}
	return sl, res.ScrollId, nil
}

// GetByQScrollID4ES 根据游标 ID 获取下一页
func GetByQScrollID4ES(ctx context.Context, scrollID string) ([]string, error) {
	res, err := GetESClient().Scroll().ScrollId(scrollID).Do(ctx)
	if err != nil {
		return nil, err
	}
	sl := make([]string, 0, res.TotalHits())
	for _, hit := range res.Hits.Hits {
		sl = append(sl, string(hit.Source))
	}
	return sl, nil
}
```



首先需要获取第一页数据并获取游标ID，然后便可以根据游标 ID 继续获取下一页数据，如果下一页如果为空的话会报 EOF 错误，此时便可知拉取结束了。

比如我们还是要分页获取籍贯为安徽的用户，且按照创建时间降序。

```javascript
GET es_index_userinfo/_search?scroll=1m
{
  "size": 1,
  "query": {
    "match": {"ancestral": "安徽"}
  },
  "sort": [
    {"create_time": "desc"}
  ]
}
```



在返回的数据中，有一个 _scroll_id 字段，下次搜索的时候带上这个数据，并且使用下面的查询语句。

```javascript
POST _search/scroll
{
  "scroll" : "1m",
  "scroll_id" : "FGluY2x1ZGVfY29udGV4dF91dWlkDXF1ZXJ5QW5kRmV0Y2gBFFRpdno4bm9CU3A1TEhvY3ktQjZzAAAAAAKolSoWbm04UWQ5SHlRdDJRRjZaeGFBdjFEQQ=="
}
```



上面的 scroll 指定搜索上下文保留的时间，1m 代表 1 分钟，还有其他时间可以选择，有 d、h、m、s 等，分别代表天、时、分钟、秒。

搜索上下文有过期自动删除，但如果自己知道什么时候该删，可以自己手动删除，减少资源占用。

```javascript
DELETE /_search/scroll
{
  "scroll_id" : "DXF1ZXJ5QW5kRmV0Y2gBAAAAAAAAA6UWWVJRTk9TUXFTLUdnU28xVFN6bEM4QQ=="
}
```



#### 小结

from + size 的优点是简单，缺点是在深度分页的场景下系统开销比较大。

search after 可以实时高效的进行分页查询，但是它只能做下一页这样的查询场景，不能随机的指定页数查询。

scroll api 方案也很高效，但是它基于快照，不能用在实时性高的业务场景，且官方已不建议使用。

### 查询文档是否存在

借助 [ExistsService](https://pkg.go.dev/github.com/olivere/elastic/v7#ExistsService) 使用 HEAD 检查文档是否存在判断。

如果文档存在， Elasticsearch 将返回一个 200 ok 的状态码，若文档不存在， Elasticsearch 将返回一个 404 Not Found 的状态码。

#### 根据ID判断文档是否存在

```javascript
// IsDocExists 某条记录是否存在
func IsDocExists(ctx context.Context, id, index string) (bool, error) {
	return GetESClient().Exists().Index(index).Id(id).Do(ctx)
}
```



RESTful api 示例：

```javascript
head es_index_userinfo/_doc/1
```



返回：

```javascript
200 - OK
```



#### 查询符合条件的文档数量

可以借助 [CountService](https://pkg.go.dev/github.com/olivere/elastic/v7#CountService) 查询符合条件的文档数量，进而判断文档是否存在。

比如查询年龄`>=18`的用户数量。

```javascript
	// 创建 range 查询条件
	rangeQuery := elastic.NewRangeQuery("age").Gte(18)
	cnt, err := GetESClient().
		Count("es_index_userinfo").
		Query(rangeQuery).
		Do(ctx)
```



RESTful api 示例：

```javascript
GET es_index_userinfo/_count
{
    "query": {
    "range": {
        "age": {"gte" : 18}
    }
  }
}
```



### 获取文档数量

上一节已经说了可以借助 [CountService](https://pkg.go.dev/github.com/olivere/elastic/v7#CountService) 查询符合条件的文档数量，如果想查询 index 下的所有文档呢？

很简单，不指定条件即可。

```javascript
// GetIndexDocNum 获取索引文档总数
func GetIndexDocNum(ctx context.Context, index string) (int64, error) {
	return GetESClient().Count(index).Do(ctx)
}
```



RESTful api 示例：

```javascript
GET es_index_userinfo/_count

# 示例结果
{
  "count" : 337,
  "_shards" : {
    "total" : 20,
    "successful" : 20,
    "skipped" : 0,
    "failed" : 0
  }
}
```