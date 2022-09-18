# Consul

[官网](https://www.consul.io/docs/connect/native/go)

[参考地址](https://laravelacademy.org/post/21213)

[示例](https://blog.csdn.net/u013536232/article/details/104235282)

[go-kit示例](https://juejin.cn/post/6844903782925467661)

Consul 是由 HashiCorp 基于 Go 语言开发的，对服务发现、配置和分段提供完整支持的 Service Mesh 解决方案，HashiCorp 这家公司有很多知名产品，比如 Vagrant，还有前面提到的 Go Micro 内置的另外两个注册中心组件 `mdns` 和 `gossip` 也是基于 HashiCorp 公司的开源库进行开发的。

书归正传，Consul 是一个分布式高可用的系统，提供的服务发现、健康检查、配置（KV存储）和分段功能可以被独立使用，比如我们在 Go Micro 微服务这里使用的主要是服务发现功能，也可以被组合起来使用以便构建完整的 Service Mesh。

每个提供服务给 Consul 的节点都运行了一个 Consul 代理（我们之前通过 `consul agent -dev` 启动的就是 Consul 的代理），服务注册和发现时这些代理不是必须的，它们主要的作用是对服务节点进行健康检查（也可以代理对 Consul Server 的访问，如果部署的是 Consul Server 集群的话）。

服务节点注册信息保存在 Consul Server 中，每个代理可以和一个或多个 Consul Server 进行交互，在整个微服务架构中，Consul Server 作为注册中心承担着服务注册与发现任务，作用异常重要，如果出现问题，则服务注册和发现将不能完成，服务调用失败，因此，在生产环境中，我们会部署一个 Consul Server 集群（3到5台 Consul 服务器），集群会通过分布式一致性协议（基于 Paxos 的 Raft 算法）选举一个领袖并进行数据的同步，当一台机器挂掉，会选举新的领袖，从而提供了系统的可用性。

### 启动 Consul 代理

关于 Consul 的安装我们前面已经演示过，这里我们重点介绍下 Consul 代理并通过它来演示服务注册与发现的流程。

Consul 代理是 Consul 的核心，完成 Consul 的安装后，必须运行代理（Consul Agent），代理可用于处理服务注册、健康检查、服务查询等，微服务集群中的每台机器上都要运行代理。

代理通常运行为两种模式：`server` 或 `client`，前者运行在 Consul Server 上，主要参与维护集群状态，承担着额外的责任，保证数据的一致性和可用性，后者运行在部署微服务的节点机器上，用于与服务器节点进行交互实现服务注册、查询和健康检查，相对而言较为轻量级。

此外，我们在前面演示时使用的是 `dev` 模式，即开发模式，顾名思义，这种模式只能用于开发环境，用来快速启动一个单节点的 Consul，不会持久化任何状态，一旦关闭，所有数据都会消失：

![consul dev mode](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/image-15626615215296.jpg)

下面我们对其中的几个字段做一些说明：

- `Node Name` 是代理的唯一名称，默认情况下是主机名称，可以通过 `-node` 选项进行指定；
- `Datacenter` 是代理被配置运行的数据中心，默认值是 `dc1`，可以通过 `-datacenter` 选项指定；
- `Server` 表明代理以 server 还是 client 模式运行，`true` 表示 server 模式；
- `Client Addr` 表示提供给客户端接口的代理地址，默认是本地 IP 地址，可以通过 `-http-addr` 选项进行指定；
- `Cluster Addr` 用于集群中 Consul 代理间通信的地址和端口集。

通过这些启动日志可以看到，开发模式下 Consul 代理以 `server` 模式运行，并且声明作为集群的领袖，同时把自己标记为一个健康的成员。

此时，打开另一个终端窗口，运行 `consul members` 即可查看 Consul 集群中的所有成员，目前只有一个：

![consul members](https://raw.githubusercontent.com/Simin-hub/Picture/master/img/image-15626628922742.jpg)

要停止 Consul 代理，可以在在代理运行窗口通过 Ctrl+C 实现。

### 服务注册

通过`Register`函数进行服务注册

```go
import (
    /...
    ./
	"github.com/go-kit/kit/sd/consul"
	"github.com/hashicorp/consul/api"
)
func Register(consulHost, consulPort, svcHost, svcPort string, logger log.Logger) (registar sd.Registrar) {

	// 创建Consul客户端连接
	var client consul.Client
	{
		consulCfg := api.DefaultConfig()
		consulCfg.Address = consulHost + ":" + consulPort
		consulClient, err := api.NewClient(consulCfg)
		if err != nil {
			logger.Log("create consul client error:", err)
			os.Exit(1)
		}

		client = consul.NewClient(consulClient)
	}

	// 设置Consul对服务健康检查的参数
	check := api.AgentServiceCheck{
		HTTP:     "http://" + svcHost + ":" + svcPort + "/health",
		Interval: "10s",
		Timeout:  "1s",
		Notes:    "Consul check service health status.",
	}

	port, _ := strconv.Atoi(svcPort)

	//设置微服务想Consul的注册信息
	reg := api.AgentServiceRegistration{
		ID:      "arithmetic" + uuid.New(),
		Name:    "arithmetic",
		Address: svcHost,
		Port:    port,
		Tags:    []string{"arithmetic", "raysonxin"},
		Check:   &check,
	}

	// 执行注册
	registar = consul.NewRegistrar(client, &reg, logger)
	return
}

```

在服务器接口中添加健康检测函数

```
// service接口
// Service Define a service interface
type Service interface {

	//省略之前的其他方法

	// HealthCheck check service health status
	HealthCheck() bool
}

// ArithmeticService实现HealthCheck
// HealthCheck implement Service method
// 用于检查服务的健康状态，这里仅仅返回true。
func (s ArithmeticService) HealthCheck() bool {
	return true
}

// loggingMiddleware实现HealthCheck
func (mw loggingMiddleware) HealthCheck() (result bool) {
	defer func(begin time.Time) {
		mw.logger.Log(
			"function", "HealthChcek",
			"result", result,
			"took", time.Since(begin),
		)
	}(time.Now())
	result = mw.Service.HealthCheck()
	return
}

// metricMiddleware实现HealthCheck
func (mw metricMiddleware) HealthCheck() (result bool) {

	defer func(begin time.Time) {
		lvs := []string{"method", "HealthCheck"}
		mw.requestCount.With(lvs...).Add(1)
		mw.requestLatency.With(lvs...).Observe(time.Since(begin).Seconds())
	}(time.Now())

	result = mw.Service.HealthCheck()
	return
}
```

。。。。。

```
var (
	consulHost  = flag.String("consul.host", "", "consul ip address")
	consulPort  = flag.String("consul.port", "", "consul port")
	serviceHost = flag.String("service.host", "", "service ip address")
	servicePort = flag.String("service.port", "", "service port")
)
// parse
flag.Parse()

// ...

//创建注册对象
registar := Register(*consulHost, *consulPort, *serviceHost, *servicePort, logger)

go func() {
	fmt.Println("Http Server start at port:" + *servicePort)
    //启动前执行注册
	registar.Register()
	handler := r
	errChan <- http.ListenAndServe(":"+*servicePort, handler)
}()

error := <-errChan
//服务退出，取消注册
registar.Deregister()
fmt.Println(error)
```

### 服务发现

[示例](https://segmentfault.com/a/1190000037589926)