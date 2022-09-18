# gRPC

## 源码解析

### 服务发现和负载均衡

[参考](https://blog.cong.moe/post/2021-03-06-grpc-go-discovery-lb/)

#### 基本介绍

由于 gRPC client 和 server 建立的长连接, 因而基于连接的负载均衡没有太大意义, 所以 gRPC 负载均衡是**基于每次调用.** 也就是你在同一个 client 发的请求也希望它被负载均衡到所有服务端.

##### 为什么选择客户端负载均衡

一般来说负载均衡器是独立的, 被放置在服务消费者和提供者之间. 代理通常需要保存请求响应副本, 因此有性能消耗也会造成额外延迟. 当请求量大时, lb 可能会变成瓶颈, 并且此时 lb 单点故障会影响整个服务.

##### 原理

gRPC 采取的**客户端负载均衡**, 主要由**两个客户端组件**来完成:

1. 维护目标服务名称和真实地址列表的映射 (**resolver**)
2. 控制该和哪些真实地址建立连接, 该将请求发送给哪个服务实例 (**balancer**)

![dns_to_load_balancer_mapping_3](https://blog.cong.moe/dns_to_load_balancer_mapping_3.png)

这种方式是客户端直接请求服务端, 所以没有额外性能开销. 这种模式客户端**可能**会和多个服务端建立连接(balancer 部分详细介绍), gRPC 的 client connection 背后其实维护了一组 subConnections, 每个 subConnection 会与一个服务端建立连接. 详情参考文档 [Load Balancing in gRPC](https://github.com/grpc/grpc/blob/master/doc/load-balancing.md).

#### Resolver

`Resolver` 提供了 `server -> addrs` 的映射.

gRPC go client 中负责解析 `server -> addrs` 的模块是 [google.golang.org/grpc/resolver](https://github.com/grpc/grpc-go/tree/61f0b5fa7c1c375e2bbf29f2f5f8610d2b5ba956/resolver) 模块.

client 建立连接时, 会根据 `URI scheme` 选取 resolver 模块中全局注册的对应 resolver, 被选中的 resolver 负责根据 `uri Endpoint` 解析出对应的 `addrs`. 因此我们实现自己服务发现模块就是通过扩展全局注册自定义 `scheme resolver` 实现. 详情参考 [gRPC Name Resolution](https://github.com/grpc/grpc/blob/master/doc/naming.md) 文档.

扩展 resolver 核心就是实现 [resolver.Builder](https://github.com/grpc/grpc-go/blob/61f0b5fa7c1c375e2bbf29f2f5f8610d2b5ba956/resolver/resolver.go#L229) 这个 interface.

```go
// m is a map from scheme to resolver builder.
var  m = make(map[string]Builder)

type Target struct {
  Scheme    string
  Authority string
  Endpoint  string
}

// Builder creates a resolver that will be used to watch name resolution updates.
type Builder interface {
  // Build creates a new resolver for the given target.
  //
  // gRPC dial calls Build synchronously, and fails if the returned error is
  // not nil.
  Build(target Target, cc ClientConn, opts BuildOptions) (Resolver, error)
  // Scheme returns the scheme supported by this resolver.
  // Scheme is defined at https://github.com/grpc/grpc/blob/master/doc/naming.md.
  Scheme() string
}

// State contains the current Resolver state relevant to the ClientConn.
type State struct {
  // Addresses is the latest set of resolved addresses for the target.
  Addresses []Address

  // ServiceConfig contains the result from parsing the latest service
  // config.  If it is nil, it indicates no service config is present or the
  // resolver does not provide service configs.
  ServiceConfig *serviceconfig.ParseResult

  // Attributes contains arbitrary data about the resolver intended for
  // consumption by the load balancing policy.
  Attributes *attributes.Attributes
}

// Resolver watches for the updates on the specified target.
// Updates include address updates and service config updates.
type Resolver interface {
  // ResolveNow will be called by gRPC to try to resolve the target name
  // again. It's just a hint, resolver can ignore this if it's not necessary.
  //
  // It could be called multiple times concurrently.
  ResolveNow(ResolveNowOptions)
  // Close closes the resolver.
  Close()
}
```

gRPC 客户端在建立连接时, 地址解析部分大致会有以下几个步骤:

1. 根据传入地址的 `Scheme` 在全局 resolver map (上面代码中的 m) 中找到与之对应的 resolver (Builder)
2. 将地址解析为 `Target` 作为参数调用 `resolver.Build` 方法实例化出 `Resolver`
3. 使用用户实现 `Resolver` 中调用 `cc.UpdateState` 传入的 `State.Addrs` 中的地址建立连接

例如: 注册一个 test resolver, m 值会变为 `{test: testResolver}`, 当连接地址为 `test:///xxx` 时, 会被匹配到 `testResolver`, 并且地址会被解析为 `&Target{Scheme: "test", Authority: "", Endpoint: "xxx"}`, 之后作为调用 `testResolver.Build` 方法的参数.

整理一下:

1. 每个 Scheme 对应一个 Builder
2. 相同 Scheme 每个不同 target 对应一个 Resolver, 通过 builder.Build 实例化

##### 静态 resolver 例子

实现一个写死路由表的例子:

```go
// 定义 Scheme 名称
const exampleScheme = "example"

type exampleResolverBuilder struct {
  addrsStore map[string][]string
}

func NewExampleResolverBuilder(addrsStore map[string][]string) *exampleResolverBuilder {
  return &exampleResolverBuilder{addrsStore: addrsStore}
}

func (e *exampleResolverBuilder) Build(target resolver.Target, cc resolver.ClientConn, opts resolver.BuildOptions) (resolver.Resolver, error) {
  // 初始化 resolver, 将 addrsStore 传递进去
  r := &exampleResolver{
    target:     target,
    cc:         cc,
    addrsStore: e.addrsStore,
  }
  // 调用 start 初始化地址
  r.start()
  return r, nil
}
func (e *exampleResolverBuilder) Scheme() string { return exampleScheme }

type exampleResolver struct {
  target     resolver.Target
  cc         resolver.ClientConn
  addrsStore map[string][]string
}

func (r *exampleResolver) start() {
  // 在静态路由表中查询此 Endpoint 对应 addrs
  addrStrs := r.addrsStore[r.target.Endpoint]
  addrs := make([]resolver.Address, len(addrStrs))
  for i, s := range addrStrs {
    addrs[i] = resolver.Address{Addr: s}
  }
  // addrs 列表转化为 state, 调用 cc.UpdateState 更新地址
  r.cc.UpdateState(resolver.State{Addresses: addrs})
}
func (*exampleResolver) ResolveNow(o resolver.ResolveNowOptions) {}
func (*exampleResolver) Close()                                  {}
```

可以这么使用:

```go
// 注册我们的 resolver
resolver.Register(NewExampleResolverBuilder(map[string][]string{
  "test": []string{"localhost:8080", "localhost:8081"},
}))

// 建立对应 scheme 的连接, 并且配置负载均衡
conn, err := grpc.Dial("example:///test", grpc.WithDefaultServiceConfig(`{"loadBalancingPolicy":"round_robin"}`))
```

原理非常简单, `exampleResolver` 只是把从路由表中查到的 `addrs` 更新到底层的 `connection` 中.

##### 基于 etcd 的 resolver

etcd 作为服务发现主要原理是:

1. 服务端启动时, 向 etcd 中存一个 key 为 `{{serverName}}/{{addr}}`, 并且设置一个较短的 `Lease`
2. 服务端 `KeepAlive` 定时续约这个 key
3. 客户端启动时 `get` prefix 为 `{{serverName}}/` 的所有 key, 得到当前服务列表
4. 客户端 `watch` prefix 为 `{{serverName}}/` 的 key 就能得到服务列表变动事件

接着实现:

###### 1. 服务端注册

```go
func Register(ctx context.Context, client *clientv3.Client, service, self string) error {
  resp, err := client.Grant(ctx, 2)
  if err != nil {
    return errors.Wrap(err, "etcd grant")
  }
  _, err = client.Put(ctx, strings.Join([]string{service, self}, "/"), self,   clientv3.WithLease(resp.ID))
  if err != nil {
    return errors.Wrap(err, "etcd put")
  }
  // respCh 需要消耗, 不然会有 warning
  respCh, err := client.KeepAlive(ctx, resp.ID)
  if err != nil {
    return errors.Wrap(err, "etcd keep alive")
  }

  for {
    select {
    case <-ctx.Done():
      return nil
    case <-respCh:

    }
  }
}
```

代码很简单不做过多说明.

###### 2. 客户端

```go
const (
  // etcd resolver 负责的 scheme 类型
  Scheme      = "etcd"
  defaultFreq = time.Minute * 30
)

type Builder struct {
  client *clientv3.Client
  // 全局路由表快照, 非必要
  store  map[string]map[string]struct{}
}

func NewBuilder(client *clientv3.Client) *Builder {
  return &Builder{
    client: client,
    store:  make(map[string]map[string]struct{}),
  }
}

func (b *Builder) Build(target resolver.Target, cc resolver.ClientConn, opts resolver.BuildOptions) (resolver.Resolver, error) {
  b.store[target.Endpoint] = make(map[string]struct{})

  // 初始化 etcd resolver
  r := &etcdResolver{
    client: b.client,
    target: target,
    cc:     cc,
    store:  b.store[target.Endpoint],
    stopCh: make(chan struct{}, 1),
    rn:     make(chan struct{}, 1),
    t:      time.NewTicker(defaultFreq),
  }

  // 开启后台更新 goroutine
  go r.start(context.Background())
  // 全量更新服务地址
  r.ResolveNow(resolver.ResolveNowOptions{})

  return r, nil
}

func (b *Builder) Scheme() string {
  return Scheme
}

type etcdResolver struct {
  client *clientv3.Client
  target resolver.Target
  cc     resolver.ClientConn
  store  map[string]struct{}
  stopCh chan struct{}
  // rn channel is used by ResolveNow() to force an immediate resolution of the target.
  rn chan struct{}
  t  *time.Ticker
}

func (r *etcdResolver) start(ctx context.Context) {
  target := r.target.Endpoint

  w := clientv3.NewWatcher(r.client)
  rch := w.Watch(ctx, target+"/", clientv3.WithPrefix())
  for {
    select {
    case <-r.rn:
      r.resolveNow()
    case <-r.t.C:
      r.ResolveNow(resolver.ResolveNowOptions{})
    case <-r.stopCh:
      w.Close()
      return
    case wresp := <-rch:
      for _, ev := range wresp.Events {
        switch ev.Type {
        case mvccpb.PUT:
          r.store[string(ev.Kv.Value)] = struct{}{}
        case mvccpb.DELETE:
          delete(r.store, strings.Replace(string(ev.Kv.Key), target+"/", "", 1))
        }
      }
      r.updateTargetState()
    }
  }
}

func (r *etcdResolver) resolveNow() {
  target := r.target.Endpoint
  resp, err := r.client.Get(context.Background(), target+"/", clientv3.WithPrefix())
  if err != nil {
    r.cc.ReportError(errors.Wrap(err, "get init endpoints"))
    return
  }

  for _, kv := range resp.Kvs {
    r.store[string(kv.Value)] = struct{}{}
  }

  r.updateTargetState()
}

func (r *etcdResolver) updateTargetState() {
  addrs := make([]resolver.Address, len(r.store))
  i := 0
  for k := range r.store {
    addrs[i] = resolver.Address{Addr: k}
    i++
  }
  r.cc.UpdateState(resolver.State{Addresses: addrs})
}

// 会并发调用, 所以这里防止同时多次全量刷新
func (r *etcdResolver) ResolveNow(o resolver.ResolveNowOptions) {
  select {
  case r.rn <- struct{}{}:
  default:

  }
}

func (r *etcdResolver) Close() {
  r.t.Stop()
  close(r.stopCh)
}
```

上面代码核心在于 `func (r *etcdResolver) start(ctx context.Context)` 这个函数, 他做了下面三件事情:

1. `watch` etcd 相应的 key prefix, 变更事件发生时, 更新本地缓存, 更新底层连接的 `addrs`
2. `r.rn channel` 收到消息时做一次全量刷新, `r.rn` 消息在 `ResolveNow` 被调用时产生
3. 全局设了一个 30 分钟全量刷新的兜底方案, 周期到达时, 做一次全量刷新

使用方法和静态路由差不多, 完整代码以及事例可以查看 [zcong1993/grpc-example](https://github.com/zcong1993/grpc-example).

#### Balancer

`Balancer` 负责根据 resolver 解析到的真实地址列表, 控制客户端和服务端实例底层连接建立, 还有发送 rpc 请求时连接的选取.

首先介绍下使用, 使用内置负载均衡只需要简单一个参数: `grpc.WithDefaultServiceConfig(`{"loadBalancingPolicy":"round_robin"}`)`. 当然也是和 resolver 一样支持注册自己的负载均衡实现来进行扩展.

以前是可以使用 `grpc.WithBalancerName("round_robin")`, 但是这个方法被废弃了. 我个人认为后者更加清晰, GitHub 上面也有一个 [grpc-go/issues/3003](https://github.com/grpc/grpc-go/issues/3003) 讨论此问题, 感兴趣的可以查看.

##### 原理

gRPC balancer 的默认行为是: **对于 resolver 中的一组地址, 只会尝试建立一个连接, 并使用此连接处理所有 RPC 请求**.

首先需要介绍下 `balancer.SubConn`.

```go
type SubConn interface {
  UpdateAddresses([]resolver.Address)
  Connect()
}
```

SubConn 是 gRPC 对于连接的抽象, 代表了对于一个后端服务的连接. 它包含了 resolver 解析出来的该服务的地址列表. 当 Connect 方法被调用时, gRPC 会按照地址列表顺序尝试建立连接, 一旦有第一个连接建立成功就会停止其他连接建立(只会建立一个正常连接).

接着介绍 `balancer.Balancer` 和 `balancer.Picker` 接口, 他们就是我们扩展负载均衡需要实现的接口.

```go
type Balancer interface {
  // UpdateClientConnState 会在 ClientConnState 改变时被调用
  // 例如: resolver 更新地址列表
  UpdateClientConnState(ClientConnState) error
  ResolverError(error)
  // UpdateSubConnState 会在 SubConn 自身状态改变时被调用
  // 例如: 连接断了
  UpdateSubConnState(SubConn, SubConnState)
  Close()
}

// Picker 会在每次 rpc 调用时被调用, 他需要负责连接选取
type Picker interface {
  Pick(info PickInfo) (PickResult, error)
}
```

balancer 会再次对连接做一次抽象, 因为他可以控制建立几条连接, 所以他需要对多个连接状态做聚合, 并且需要维护更新 picker 的状态(例如剔除/增加连接).

看到这里大家应该还是一头雾水, 下面以 `round_robin` balancer 的实现来举例, 便于大家理解.

首先分析, round_robin 需要将请求按照顺序依次分发到对应的后端服务实例, 也就是说我们需要和 **所有后端实例** 都建立连接. 因此需要实现一个和所有后端实例建立连接并且维护聚合状态的 `Balancer`, 这其实是大多数负载均衡都需要做的事情(和所有后端实例建立连接), 所以 gRPC 把这个通用功能放在了 `balancer/base` 中实现.

`balancer/base` 包实现了 `Balancer` 接口解决了连接需要所有实例场景下的连接管理问题, 并抽象出了一个 `PickerBuilder` 接口, 可以通过实现此接口来实现核心的负载均衡逻辑, 简化了扩展负载均衡的难度.

```go
type PickerBuilder interface {
  Build(info PickerBuildInfo) balancer.Picker
}

type PickerBuildInfo struct {
  ReadySCs map[balancer.SubConn]SubConnInfo
}
```

`PickerBuilder.Build` 方法会在连接状态改变时被调用, 永远传递最新的所有健康连接, 实现方需要通过这些信息实现出 `Picker`.

round_robin 实现的 `PickerBuilder` 就很简单:

```go
type rrPickerBuilder struct{}

func (*rrPickerBuilder) Build(info base.PickerBuildInfo) balancer.Picker {
  if len(info.ReadySCs) == 0 {
    return base.NewErrPicker(balancer.ErrNoSubConnAvailable)
  }
  scs := make([]balancer.SubConn, 0, len(info.ReadySCs))
  for sc := range info.ReadySCs {
    scs = append(scs, sc)
  }
  // 将参数传递进来的最新健康连接保存, 并且生成一个随机数作为初始 pick index
  return &rrPicker{
    subConns: scs,
    next: grpcrand.Intn(len(scs)),
  }
}

type rrPicker struct {
  subConns []balancer.SubConn

  mu   sync.Mutex
  next int
}

// 每次 rpc 调用时, 返回 next 对应的连接
// 并且将 next 计数器加一
func (p *rrPicker) Pick(balancer.PickInfo) (balancer.PickResult, error) {
  p.mu.Lock()
  sc := p.subConns[p.next]
  p.next = (p.next + 1) % len(p.subConns)
  p.mu.Unlock()
  return balancer.PickResult{SubConn: sc}, nil
}
```

接着介绍下 `baseBalancer` 的实现.

```go
// 去除掉细节字段
type baseBalancer struct {
  // gRPC 连接, baseBalancer 需要维护连接状态
  // 在连接状态变动时生成新的 Picker,
  // 并调用 cc.UpdateState 通知变动
  cc            balancer.ClientConn
  // pickerBuilder 需要用户提供
  pickerBuilder PickerBuilder
  // 维护 addr 和 SubConn 的映射
  subConns *resolver.AddressMap
  // 维护 SubConn 和状态的映射
  scStates map[balancer.SubConn]connectivity.State
}

// UpdateClientConnState 会在连接基础信息发生改变时被调用
// 例如: resolver 更新地址列表信息
func (b *baseBalancer) UpdateClientConnState(s balancer.ClientConnState) error {
  b.resolverErr = nil

  // 遍历 s.ResolverState.Addresses 地址列表
  // 和现有的 subConns 进行比对
  // 如果发现是新的 addr, 则调用 NewSubConn 创建新连接
  addrsSet := resolver.NewAddressMap()
  for _, a := range s.ResolverState.Addresses {
    addrsSet.Set(a, nil)
    if _, ok := b.subConns.Get(a); !ok {
      // a is a new address (not existing in b.subConns).
      sc, err := b.cc.NewSubConn([]resolver.Address{a}, balancer.NewSubConnOptions{HealthCheckEnabled: b.config.HealthCheck})
      if err != nil {
        logger.Warningf("base.baseBalancer: failed to create new SubConn: %v", err)
        continue
      }
      b.subConns.Set(a, sc)
      b.scStates[sc] = connectivity.Idle
      b.csEvltr.RecordTransition(connectivity.Shutdown, connectivity.Idle)
      sc.Connect()
    }
  }
  // 将不存在于 s.ResolverState.Addresses 地址列表中的连接关闭
  for _, a := range b.subConns.Keys() {
    sci, _ := b.subConns.Get(a)
    sc := sci.(balancer.SubConn)
    // a was removed by resolver.
    if _, ok := addrsSet.Get(a); !ok {
      b.cc.RemoveSubConn(sc)
      b.subConns.Delete(a)
    }
  }

  if len(s.ResolverState.Addresses) == 0 {
    b.ResolverError(errors.New("produced zero addresses"))
    return balancer.ErrBadResolverState
  }
  return nil
}

// 从当前最新的 subConns 中筛选出状态为 Ready 的连接
// 调用用户的 pickerBuilder.Build 方法创建新的 Picker
func (b *baseBalancer) regeneratePicker() {
  if b.state == connectivity.TransientFailure {
    b.picker = NewErrPicker(b.mergeErrors())
    return
  }
  readySCs := make(map[balancer.SubConn]SubConnInfo)

  // Filter out all ready SCs from full subConn map.
  for _, addr := range b.subConns.Keys() {
    sci, _ := b.subConns.Get(addr)
    sc := sci.(balancer.SubConn)
    if st, ok := b.scStates[sc]; ok && st == connectivity.Ready {
      readySCs[sc] = SubConnInfo{Address: addr}
    }
  }
  b.picker = b.pickerBuilder.Build(PickerBuildInfo{ReadySCs: readySCs})
}

// UpdateSubConnState 会在 SubConn 自身连接状态发生改变时被调用
func (b *baseBalancer) UpdateSubConnState(sc balancer.SubConn, state balancer.SubConnState) {
  // 改变后的新状态
  s := state.ConnectivityState
  // 老状态
  oldS, ok := b.scStates[sc]
  if !ok {
    return
  }
  if oldS == connectivity.TransientFailure &&
    (s == connectivity.Connecting || s == connectivity.Idle) {
    // 从暂时失败转换为空闲状态时, 调用 Connect 发起连接
    if s == connectivity.Idle {
      sc.Connect()
    }
    // 正在连接状态, 不做处理
    return
  }
  // 更新状态为新状态
  b.scStates[sc] = s
  switch s {
  case connectivity.Idle:
    sc.Connect()
  case connectivity.Shutdown:
    // 连接断开时, 删除他的本地状态映射
    delete(b.scStates, sc)
  case connectivity.TransientFailure:
    // 保存错误
    b.connErr = state.ConnectionError
  }

  // 更新聚合状态
  b.state = b.csEvltr.RecordTransition(oldS, s)

  // 在下面两种情况时重新生成 Picker
  //  - 当前 SubConn 状态由其他变为 Ready 或者由 Ready 变为其他
  //  - 聚合状态转变为临时性错误时
  if (s == connectivity.Ready) != (oldS == connectivity.Ready) ||
    b.state == connectivity.TransientFailure {
    b.regeneratePicker()
  }
  // 调用 cc.UpdateState 更新聚合 state 和最新 Picker
  b.cc.UpdateState(balancer.State{ConnectivityState: b.state, Picker: b.picker})
}
```

可以看到如果想要自己从头实现一个 `Balancer` 还是比较困难的, 连接管理那里需要对 gRPC 足够熟悉. 所以大多数时候一般都会使用 baseBalancer 来实现自己的负载均衡器. 如果结合 `resolver.Address.Attributes` 元信息可以实现更多的功能, 例如: 流量路由.

#### 写在最后

通过学习 gRPC 负载均衡我们可以看到不同类型负载均衡器的优缺点, gRPC 所采用的客户端负载均衡虽然解决了性能问题, 但是也为客户端代码增加了很多复杂度, 虽然我们使用者不太感知得到, 而且文章开头也说明了 gRPC 是支持多种语言的, 也就意味着每种语言客户端都得实现. 然而现状是不同语言的客户端对于一些新特性的实现周期有很大差异, 例如: `c++`, `golang`, `java` 的客户端新特性支持情况会最好, 但是 NodeJS 之类的语言支持情况就不那么好, 这也是长期 gRPC 面临的问题. 例如至今 NodeJS client 库还是 callback 的形式, 并且仍不支持 `server interceptor`.

