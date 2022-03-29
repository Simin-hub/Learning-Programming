# go-kit

[参考地址](https://juejin.cn/post/6844903794380111886)

go-kit是一套帮助开发者构建健壮、可靠、可维护的微服务的golang工具包集合。最初应用于大型企业开发，但是很快也开始为小型初创企业和组织服务。

go-kit自上而下采用**三层架构方式：Transport、Endpoint、Service**。**Transport层主要负责与传输协议HTTP、gRPC、Thrift等相关的逻辑**；**Endpoint层主要负责request／response格式的转换**，**以及公用拦截器相关的逻辑**；**Service层则专注于业务逻辑**。Endpoint层作为go-kit的核心，采用类似洋葱的模型，提供了对日志、限流、熔断、链路追踪、服务监控等方面的扩展能力。为了帮助开发者构建微服务，go-kit提供了对consul、etcd、zookeeper、eureka等注册中心的支持。

