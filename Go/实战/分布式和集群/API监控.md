# API监控

## 概述

目前，几乎所有的研发人员每天都在跟API打交道：后端为实现业务不停的生产API，前端为实现产品功能不停的调用API。API已经成为前端与后端、产品与产品、公司与公司之间技术沟通、业务合作的桥梁。

微服务中，API几乎是服务与外界的唯一交互渠道，API服务的稳定性、可靠性越来越成为不可忽略的部分。我们需要实时了解API的运行状况（请求次数、延时、失败等），需要通过对历史数据的分析了解哪些API存在瓶颈以便后期优化。所以，为了确保系统良好的提供服务，绝大多数的微服务框架也都集成了API监控组件。

增加API监控功能：`Prometheus`作为监控组件，`Grafana`作为可视化工具，两者均通过`docker-compose`部署运行。go-kit已经提供`prometheus`组件（`metric/prometheus`），因此集成工作变得非常容易。在开始之前我们需要先了解一下所需的知识点。

### Prometheus

Prometheus （中文名称：普罗米修斯）是一套开源的系统监控报警框架。作为新一代的监控框架，Prometheus 具有以下特点：

- 提供强大的多维度数据模型，如Counter、Gauge、Histogram、Summary；
- 强大而灵活的查询语句（PromQL），可方便的实现对时间序列数据的查询、聚合操作；
- 易于管理与高效；
- 提供pull模式、push gateway方式实现时间序列数据的采集；
- 支持多种可视化图形界面：Grafana、Web UI、API clients；
- 报警规则管理、报警检测和报警推送功能。

[golang接入](https://cloud.tencent.com/document/product/1416/56033)

[示例](https://www.cnblogs.com/FG123/p/13689649.html)

**go-kit例子**

```
//下载最新版本Prometheus客户端
go get github.com/prometheus/client_golang/prometheus
```

```
// metricMiddleware 定义监控中间件，嵌入Service
// 新增监控指标项：requestCount和requestLatency
type metricMiddleware struct {
	Service		//原有服务器接口
	requestCount   metrics.Counter
	requestLatency metrics.Histogram
}
```

接下来创建为Service封装指标采集的方法`Metric`，采集请求次数和请求延迟两个指标项：

```
// Metrics 指标采集方法
func Metrics(requestCount metrics.Counter, requestLatency metrics.Histogram) ServiceMiddleware {
	return func(next Service) Service {
		return metricMiddleware{
			next,
			requestCount,
			requestLatency}
	}
}
```

然后跟限流、日志中间件的方式一致，**由于嵌入了Service接口，需要依次实现该接口的方法**，以Add为例进行说明，其他方法与之类似。

- 每接收一次请求，请求次数每次加1；
- 通过请求结束时间减去请求开始的差值（单位秒）作为请求延时；

```
func (mw metricMiddleware) Add(a, b int) (ret int) {

	defer func(beign time.Time) {
		lvs := []string{"method", "Add"}
		mw.requestCount.With(lvs...).Add(1)
		mw.requestLatency.With(lvs...).Observe(time.Since(beign).Seconds())
	}(time.Now())

	ret = mw.Service.Add(a, b)
	return ret
}
```

**开放Prometheus指标采集接口**

在`transport.go`中新增用于Prometheus轮循拉取监控指标的代码，开放API接口`/metrics`。

```
r.Path("/metrics").Handler(promhttp.Handler())
```

**修改main.go**

首先创建指标采集对象：请求次数采集和请求延时采集对象。

```
fieldKeys := []string{"method"}
requestCount := kitprometheus.NewCounterFrom(stdprometheus.CounterOpts{
	Namespace: "raysonxin",
	Subsystem: "arithmetic_service",
	Name:      "request_count",
	Help:      "Number of requests received.",
}, fieldKeys)

requestLatency := kitprometheus.NewSummaryFrom(stdprometheus.SummaryOpts{
	Namespace: "raysonxin",
	Subsystem: "arithemetic_service",
	Name:      "request_latency",
	Help:      "Total duration of requests in microseconds.",
}, fieldKeys)
复制代码
```

使用`Metrics`方法对Service对象进行封装：

```
svc = Metrics(requestCount, requestLatency)(svc)
复制代码
```

> 由于最后需要使用Postman进行接口测试，这里我将限流器的容量改为了100。

```
//add ratelimit,refill every second,set capacity 3
ratebucket := rate.NewLimiter(rate.Every(time.Second*1), 100)
endpoint = NewTokenBucketLimitterWithBuildIn(ratebucket)(endpoint)
复制代码
```

至此，代码修改完成，可通过`go build`进行编译，确保没有问题。


作者：码路印记
链接：https://juejin.cn/post/6844903780949966856
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

### Grafana

Grafana是一个跨平台的开源的度量分析和可视化工具，可以通过将采集的数据查询然后可视化的展示，并及时通知。它主要有以下六大特点：

- 展示方式：快速灵活的客户端图表，面板插件有许多不同方式的可视化指标和日志，官方库中具有丰富的仪表盘插件，比如热图、折线图、图表等多种展示方式；
- 数据源：Graphite，InfluxDB，OpenTSDB，Prometheus，Elasticsearch，CloudWatch和KairosDB等；
- 通知提醒：以可视方式定义最重要指标的警报规则，Grafana将不断计算并发送通知，在数据达到阈值时通过Slack、PagerDuty等获得通知；
- 混合展示：在同一图表中混合使用不同的数据源，可以基于每个查询指定数据源，甚至自定义数据源；
- 注释：使用来自不同数据源的丰富事件注释图表，将鼠标悬停在事件上会显示完整的事件元数据和标记；
- 过滤器：Ad-hoc过滤器允许动态创建新的键/值过滤器，这些过滤器会自动应用于使用该数据源的所有查询。

> 通过配置文件为Prometheus添加作业（job），尤其定时向本示例服务发起HTTP请求监控指标数据（即pull模式）。

[使用](https://blog.csdn.net/hjxzb/article/details/81044583)