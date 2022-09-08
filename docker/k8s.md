# Kubernetes

[官方手册](https://kubernetes.io/zh-cn/docs/concepts/overview/)、[一关系图让你理解K8s中的概念](https://zhuanlan.zhihu.com/p/105006577)

## 入门

### 简介

Kubernetes是容器集群管理系统，是一个开源的平台，可以实现容器集群的自动化部署、自动扩缩容、维护等功能。

通过Kubernetes你可以：

- 快速部署应用
- 快速扩展应用
- 无缝对接新的应用功能
- 节省资源，优化硬件资源的使用

我们的目标是促进完善组件和工具的生态系统，以减轻应用程序在公有云或私有云中运行的负担。

#### Kubernetes 特点

- **可移植**: 支持公有云，私有云，混合云，多重云（multi-cloud）
- **可扩展**: 模块化, 插件化, 可挂载, 可组合
- **自动化**: 自动部署，自动重启，自动复制，自动伸缩/扩展

Kubernetes是Google 2014年创建管理的，是Google 10多年大规模容器管理技术Borg的开源版本。

#### Why containers?

为什么要使用[容器](https://aucouranton.com/2014/06/13/linux-containers-parallels-lxc-openvz-docker-and-more/)？通过以下两个图对比：

![为什么是容器？](https://d33wubrfki0l68.cloudfront.net/e7b766e0175f30ae37f7e0e349b87cfe2034a1ae/3e391/images/docs/why_containers.svg)

**传统的应用部署方式是通过插件或脚本来安装应用。**这样做的缺点是应用的运行、配置、管理、所有生存周期将与当前操作系统绑定，这样做并不利于应用的升级更新/回滚等操作，当然也可以通过创建虚机的方式来实现某些功能，但是虚拟机非常重，并不利于可移植性。

新的方式是**通过部署容器方式实现，每个容器之间互相隔离，每个容器有自己的文件系统 ，容器之间进程不会相互影响，能区分计算资源**。相对于虚拟机，容器能快速部署，由于容器与底层设施、机器文件系统解耦的，所以它能在不同云、不同版本操作系统间进行迁移。

容器占用资源少、部署快，每个应用可以被打包成一个容器镜像，每个应用与容器间成一对一关系也使容器有更大优势，使用容器可以在build或release 的阶段，为应用创建容器镜像，因为每个应用不需要与其余的应用堆栈组合，也不依赖于生产环境基础结构，这使得从研发到测试、生产能提供一致环境。类似地，容器比虚机轻量、更“透明”，这更便于监控和管理。最后，

容器优势总结：

- **快速创建/部署应用：**与VM虚拟机相比，容器镜像的创建更加容易。
- **持续开发、集成和部署：**提供可靠且频繁的容器镜像构建/部署，并使用快速和简单的回滚(由于镜像不可变性)。
- **开发和运行相分离：**在build或者release阶段创建容器镜像，使得应用和基础设施解耦。
- **开发，测试和生产环境一致性：**在本地或外网（生产环境）运行的一致性。
- **云平台或其他操作系统：**可以在 Ubuntu、RHEL、 CoreOS、on-prem、Google Container Engine或其它任何环境中运行。
- **Loosely coupled，分布式，弹性，微服务化：**应用程序分为更小的、独立的部件，可以动态部署和管理。
- **资源隔离**
- **资源利用：**更高效

#### 使用Kubernetes能做什么？

可以在物理或虚拟机的Kubernetes集群上运行容器化应用，Kubernetes能提供一个以“**容器为中心的基础架构**”，满足在生产环境中运行应用的一些常见需求，如：

Kubernetes 为你提供：

- **服务发现和负载均衡**

  Kubernetes 可以使用 DNS 名称或自己的 IP 地址来曝露容器。 如果进入容器的流量很大， Kubernetes 可以负载均衡并分配网络流量，从而使部署稳定。

- **存储编排**

  Kubernetes 允许你自动挂载你选择的存储系统，例如本地存储、公共云提供商等。

- **自动部署和回滚**

  你可以使用 Kubernetes 描述已部署容器的所需状态， 它可以以受控的速率将实际状态更改为期望状态。 例如，你可以自动化 Kubernetes 来为你的部署创建新容器， 删除现有容器并将它们的所有资源用于新容器。

- **自动完成装箱计算**

  你为 Kubernetes 提供许多节点组成的集群，在这个集群上运行容器化的任务。 你告诉 Kubernetes 每个容器需要多少 CPU 和内存 (RAM)。 Kubernetes 可以将这些容器按实际情况调度到你的节点上，以最佳方式利用你的资源。

- **自我修复**

  Kubernetes 将重新启动失败的容器、替换容器、杀死不响应用户定义的运行状况检查的容器， 并且在准备好服务之前不将其通告给客户端。

- **密钥与配置管理**

  Kubernetes 允许你存储和管理敏感信息，例如密码、OAuth 令牌和 ssh 密钥。 你可以在不重建容器镜像的情况下部署和更新密钥和应用程序配置，也无需在堆栈配置中暴露密钥。

#### Kubernetes不是什么？

Kubernetes 不是传统的、包罗万象的 PaaS（平台即服务）系统。 由于 Kubernetes 是在容器级别运行，而非在硬件级别，它提供了 PaaS 产品共有的一些普遍适用的功能， 例如部署、扩展、负载均衡，允许用户集成他们的日志记录、监控和警报方案。 但是，Kubernetes 不是单体式（monolithic）系统，那些默认解决方案都是可选、可插拔的。 Kubernetes 为构建开发人员平台提供了基础，但是在重要的地方保留了用户选择权，能有更高的灵活性。

Kubernetes：

- 不限制支持的应用程序类型。 Kubernetes 旨在支持极其多种多样的工作负载，包括无状态、有状态和数据处理工作负载。 如果应用程序可以在容器中运行，那么它应该可以在 Kubernetes 上很好地运行。
- 不部署源代码，也不构建你的应用程序。 持续集成（CI）、交付和部署（CI/CD）工作流取决于组织的文化和偏好以及技术要求。
- 不提供应用程序级别的服务作为内置服务，例如中间件（例如消息中间件）、 数据处理框架（例如 Spark）、数据库（例如 MySQL）、缓存、集群存储系统 （例如 Ceph）。这样的组件可以在 Kubernetes 上运行，并且/或者可以由运行在 Kubernetes 上的应用程序通过可移植机制 （例如[开放服务代理](https://openservicebrokerapi.org/)）来访问。

- 不是日志记录、监视或警报的解决方案。 它集成了一些功能作为概念证明，并提供了收集和导出指标的机制。
- 不提供也不要求配置用的语言、系统（例如 jsonnet），它提供了声明性 API， 该声明性 API 可以由任意形式的声明性规范所构成。
- 不提供也不采用任何全面的机器配置、维护、管理或自我修复系统。
- 此外，**Kubernetes 不仅仅是一个编排系统，实际上它消除了编排的需要**。 **编排的技术定义是执行已定义的工作流程：首先执行 A，然后执行 B，再执行 C**。 而 Kubernetes 包含了一组独立可组合的控制过程，可以连续地将当前状态驱动到所提供的预期状态。 你不需要在乎如何从 A 移动到 C，也不需要集中控制，这使得系统更易于使用 且功能更强大、系统更健壮，更为弹性和可扩展。

#### 名词解释

- **标签（Label）**

  用来为对象设置可标识的属性标记；这些标记对用户而言是有意义且重要的。

  标签是一些关联到 [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 这类对象上的键值对。 它们通常用来组织和选择对象子集。

- **Pod**

  Pod 是 Kubernetes 的原子对象。Pod 表示您的集群上**一组**正在运行的[容器（containers）](https://kubernetes.io/zh-cn/docs/concepts/overview/what-is-kubernetes/#why-containers)（一个或者多个容器）。

  **通常创建 Pod 是为了运行单个主容器**。Pod 还可以运行可选的边车（sidecar）容器，以添加诸如日志记录之类的补充特性。通常用 [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/) 来管理 Pod。

- **Job**

  Job 是**需要运行完成的确定性的或批量的任务**。

  Job 创建一个或多个 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 对象，并确保指定数量的 Pod 成功终止。 随着各 Pod 成功结束，Job 会跟踪记录成功完成的个数

- **Deployment**

  **管理多副本应用的一种 API 对象**，通常通过运行没有本地状态的 Pod 来完成工作。

  每个副本表现为一个 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)， Pod 分布在集群中的[节点](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/)上。 对于确实需要本地状态的工作负载，请考虑使用 [StatefulSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/)。

- **对象（Object）**

  **Kubernetes 系统中的实体**。Kubernetes API 用这些实体表示集群的状态。

  Kubernetes 对象通常是一个“目标记录”-一旦你创建了一个对象，Kubernetes [控制平面（Control Plane）](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane) 不断工作，以确保它代表的项目确实存在。 创建一个对象相当于告知 Kubernetes 系统：你期望这部分集群负载看起来像什么；这也就是你集群的期望状态。

- **服务（Service）**

  将运行在一组 [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 上的应用程序公开为网络服务的抽象方法。

  服务所针对的 Pod 集（通常）由[选择算符](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/)确定。 如果有 Pod 被添加或被删除，则与选择算符匹配的 Pod 集合将发生变化。 服务确保可以将网络流量定向到该工作负载的当前 Pod 集合。

- **工作负载（Workload）**

  工作负载是在 Kubernetes 上运行的应用程序。

  代表不同类型或部分工作负载的各种核心对象包括 DaemonSet， Deployment， Job， ReplicaSet， and StatefulSet。

  例如，具有 Web 服务器和数据库的工作负载可能在一个 [StatefulSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/) 中运行数据库， 而 Web 服务器运行在 [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/)

- **ReplicaSets**

  ReplicaSet是下一代复本控制器。ReplicaSet和 [*Replication Controller*](https://www.kubernetes.org.cn/replication-controller-kubernetes)之间的唯一区别是现在的选择器支持。*Replication Controller*只支持基于等式的selector（env=dev或environment!=qa），但ReplicaSet还支持新的，基于集合的selector（version in (v1.0, v2.0)或env notin (dev, qa)）。在试用时官方推荐ReplicaSet。

- **选择算符（Selector）**

  选择算符允许用户通过[标签（labels）](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/)对一组资源对象进行筛选过滤。

  在查询资源列表时，选择算符可以通过标签对资源进行过滤筛选。

### Kubernetes 组件

A Kubernetes cluster consists of 集群是由一组被称作[节点（node）](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/)的机器组成， 这些节点上会运行由 Kubernetes 所管理的容器化应用。 且**每个集群至少有一个工作节点**。

工作节点会托管所谓的 Pods，而 Pod 就是作为应用负载的组件。 控制平面管理集群中的工作节点和 Pods。 为集群提供故障转移和高可用性， 这些控制平面一般跨多主机运行，而集群也会跨多个节点运行。

This document outlines the various components you need to have for a complete and working Kubernetes cluster.

![Components of Kubernetes](https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg)

- 控制平面组件（Control Plane Components）
  - [kube-apiserver](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#kube-apiserver)
  - [etcd](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#etcd)
  - [kube-scheduler](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#kube-scheduler)
  - [kube-controller-manager](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#kube-controller-manager)
  - [cloud-controller-manager](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#cloud-controller-manager)
- Node 组件
  - [kubelet](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#kubelet)
  - [kube-proxy](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#kube-proxy)
  - [容器运行时（Container Runtime）](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#container-runtime)
- 插件（Addons）
  - [DNS](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#dns)
  - [Web 界面（仪表盘）](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#web-ui-dashboard)
  - [容器资源监控](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#container-resource-monitoring)
  - [集群层面日志](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#cluster-level-logging)

![使用kubeadm 搭建集群环境· 从Docker 到Kubernetes 进阶手册](https://www.qikqiak.com/k8s-book/docs/images/k8s-structure.jpeg)

#### 控制平面组件（Control Plane Components）

**控制平面组件会为集群做出全局决策，比如资源的调度。 以及检测和响应集群事件**，例如当不满足部署的 `replicas` 字段时， 要启动新的 [pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)）。

**控制平面组件可以在集群中的任何节点上运行**。 然而，为了简单起见，设置脚本通常会在同一个计算机上启动所有控制平面组件， 并且不会在此计算机上运行用户容器。 

##### kube-apiserver

API 服务器是 Kubernetes [控制平面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane)的组件， 该组件**负责公开了 Kubernetes API，负责处理接受请求的工作**。 API 服务器是 Kubernetes 控制平面的前端。

Kubernetes API 服务器的主要实现是 [kube-apiserver](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-apiserver/)。 `kube-apiserver` 设计上考虑了水平扩缩，也就是说，它可通过部署多个实例来进行扩缩。 你可以运行 `kube-apiserver` 的多个实例，并在这些实例之间平衡流量。

##### etcd

`etcd` 是兼顾一致性与高可用性的键值数据库，可以作为**保存 Kubernetes 所有集群数据的后台数据库**。

你的 Kubernetes 集群的 `etcd` 数据库通常需要有个[备份](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster)计划。

如果想要更深入的了解 `etcd`，请参考 [etcd 文档](https://etcd.io/docs/)。

##### kube-scheduler

`kube-scheduler` 是[控制平面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane)的组件， **负责监视新创建的、未指定运行[节点（node）](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/)的 [Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)， 并选择节点来让 Pod 在上面运行**。

调度决策考虑的因素包括单个 Pod 及 Pods 集合的资源需求、软硬件及策略约束、 亲和性及反亲和性规范、数据位置、工作负载间的干扰及最后时限。

##### kube-controller-manager

[kube-controller-manager](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-controller-manager/) 是[控制平面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane)的组件， **负责运行[控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/)进程**。

从逻辑上讲， 每个[控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/)都是一个单独的进程， 但是为了降低复杂性，它们都被编译到同一个可执行文件，并在同一个进程中运行。

这些控制器包括：

- 节点控制器（Node Controller）：**负责在节点出现故障时进行通知和响应**
- 任务控制器（Job Controller）：**监测代表一次性任务的 Job 对象，然后创建 Pods 来运行这些任务直至完成**
- 端点控制器（Endpoints Controller）：**填充端点（Endpoints）对象（**即加入 Service 与 Pod）
- 服务帐户和令牌控制器（Service Account & Token Controllers）：**为新的命名空间创建默认帐户和 API 访问令牌**

##### cloud-controller-manager

一个 Kubernetes [控制平面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane)组件， 嵌入了特定于云平台的控制逻辑。 云控制器管理器（Cloud Controller Manager）**允许你将你的集群连接到云提供商的 API 之上， 并将与该云平台交互的组件同与你的集群交互的组件分离开来**。

`cloud-controller-manager` 仅运行特定于云平台的控制器。 因此如果你在自己的环境中运行 Kubernetes，或者在本地计算机中运行学习环境， 所部署的集群不需要有云控制器管理器。

与 `kube-controller-manager` 类似，`cloud-controller-manager` 将若干逻辑上独立的控制回路组合到同一个可执行文件中， 供你以同一进程的方式运行。 你可以对其执行水平扩容（运行不止一个副本）以提升性能或者增强容错能力。

下面的控制器都包含对云平台驱动的依赖：

- 节点控制器（Node Controller）：**用于在节点终止响应后检查云提供商以确定节点是否已被删除**
- 路由控制器（Route Controller）：**用于在底层云基础架构中设置路由**
- 服务控制器（Service Controller）：**用于创建、更新和删除云提供商负载均衡器**

#### Node 组件

节点组件会在每个节点上运行，**负责维护运行的 Pod 并提供 Kubernetes 运行环境**。

##### kubelet

`kubelet` 会**在集群中每个[节点（node）](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/)上运行。 它保证[容器（containers）](https://kubernetes.io/zh-cn/docs/concepts/overview/what-is-kubernetes/#why-containers)都运行在 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 中**。

kubelet 接收一组通过各类机制提供给它的 PodSpecs， 确保这些 PodSpecs 中描述的容器处于运行状态且健康。 kubelet 不会管理不是由 Kubernetes 创建的容器。

##### kube-proxy

[kube-proxy](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-proxy/) **是集群中每个[节点（node）](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/)所上运行的网络代理**， 实现 Kubernetes [服务（Service）](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/) 概念的一部分。

kube-proxy 维护节点上的一些网络规则， 这些网络规则会允许从集群内部或外部的网络会话与 Pod 进行网络通信。

如果操作系统提供了可用的数据包过滤层，则 kube-proxy 会通过它来实现网络规则。 否则，kube-proxy 仅做流量转发。

##### 容器运行时（Container Runtime）

容器运行环境是**负责运行容器的软件**。

Kubernetes 支持许多容器运行环境，例如 [containerd](https://containerd.io/docs/)、 [CRI-O](https://cri-o.io/#what-is-cri-o) 以及 [Kubernetes CRI (容器运行环境接口)](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md) 的其他任何实现。

#### 插件（Addons）

插件使用 Kubernetes 资源（[DaemonSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/daemonset/)、 [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/) 等）实现集群功能。 因为这些插件**提供集群级别的功能**，插件中命名空间域的资源属于 `kube-system` 命名空间。

下面描述众多插件中的几种。有关可用插件的完整列表，请参见 [插件（Addons）](https://kubernetes.io/zh-cn/docs/concepts/cluster-administration/addons/)。

##### DNS

尽管其他插件都并非严格意义上的必需组件，但几乎所有 Kubernetes 集群都应该有[集群 DNS](https://kubernetes.io/zh-cn/docs/concepts/services-networking/dns-pod-service/)， 因为很多示例都需要 DNS 服务。

集群 DNS 是一个 DNS 服务器，和环境中的其他 DNS 服务器一起工作，它为 Kubernetes 服务提供 DNS 记录。

Kubernetes 启动的容器自动将此 DNS 服务器包含在其 DNS 搜索列表中。

##### Web 界面（仪表盘）

[Dashboard](https://kubernetes.io/zh-cn/docs/tasks/access-application-cluster/web-ui-dashboard/) 是 Kubernetes 集群的通用的、基于 Web 的用户界面。 它使用户可以管理集群中运行的应用程序以及集群本身， 并进行故障排除。

##### 容器资源监控

[容器资源监控](https://kubernetes.io/zh-cn/docs/tasks/debug/debug-cluster/resource-usage-monitoring/) 将关于容器的一些常见的时间序列度量值保存到一个集中的数据库中， 并提供浏览这些数据的界面。

##### 集群层面日志

[集群层面日志](https://kubernetes.io/zh-cn/docs/concepts/cluster-administration/logging/)机制负责将容器的日志数据保存到一个集中的日志存储中， 这种集中日志存储提供搜索和浏览接口。

### Kubernetes API

Kubernetes [控制面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane)的核心是 [API 服务器](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#kube-apiserver)。 **API 服务器负责提供 HTTP API，以供用户、集群中的不同部分和集群外部组件相互通信**。

Kubernetes API 使你可以查询和操纵 Kubernetes API 中对象（例如：Pod、Namespace、ConfigMap 和 Event）的状态。

大部分操作都可以通过 [kubectl](https://kubernetes.io/zh-cn/docs/reference/kubectl/) 命令行接口或类似 [kubeadm](https://kubernetes.io/zh-cn/docs/reference/setup-tools/kubeadm/) 这类命令行工具来执行， 这些工具在背后也是调用 API。不过，你也可以使用 REST 调用来访问这些 API。

如果你正在编写程序来访问 Kubernetes API， 可以考虑使用[客户端库](https://kubernetes.io/zh-cn/docs/reference/using-api/client-libraries/)之一。

#### OpenAPI 规范

完整的 API 细节是用 [OpenAPI](https://www.openapis.org/) 来表述的。

##### OpenAPI V2

Kubernetes API 服务器通过 `/openapi/v2` 端点提供聚合的 OpenAPI v2 规范。 你可以按照下表所给的请求头部，指定响应的格式：

| 头部               | 可选值                                                       | 说明                     |
| ------------------ | ------------------------------------------------------------ | ------------------------ |
| `Accept-Encoding`  | `gzip`                                                       | *不指定此头部也是可以的* |
| `Accept`           | `application/com.github.proto-openapi.spec.v2@v1.0+protobuf` | *主要用于集群内部*       |
| `application/json` | *默认值*                                                     |                          |
| `*`                | *提供*`application/json`                                     |                          |

Kubernetes 为 API 实现了一种基于 Protobuf 的序列化格式，主要用于集群内部通信。 关于此格式的详细信息，可参考 [Kubernetes Protobuf 序列化](https://git.k8s.io/design-proposals-archive/api-machinery/protobuf.md)设计提案。 每种模式对应的接口描述语言（IDL）位于定义 API 对象的 Go 包中。

##### OpenAPI V3

**特性状态：** `Kubernetes v1.24 [beta]`

Kubernetes v1.25 提供将其 API 以 OpenAPI v3 形式发布的 beta 支持； 这一功能特性处于 beta 状态，默认被开启。 你可以通过为 kube-apiserver 组件关闭 `OpenAPIV3` [特性门控](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/feature-gates/)来禁用此 beta 特性。

发现端点 `/openapi/v3` 被提供用来查看可用的所有组、版本列表。 此列表仅返回 JSON。这些组、版本以下面的格式提供：

```yaml
{
    "paths": {
        ...,
        "api/v1": {
            "serverRelativeURL": "/openapi/v3/api/v1?hash=CC0E9BFD992D8C59AEC98A1E2336F899E8318D3CF4C68944C3DEC640AF5AB52D864AC50DAA8D145B3494F75FA3CFF939FCBDDA431DAD3CA79738B297795818CF"
        },
        "apis/admissionregistration.k8s.io/v1": {
            "serverRelativeURL": "/openapi/v3/apis/admissionregistration.k8s.io/v1?hash=E19CC93A116982CE5422FC42B590A8AFAD92CDE9AE4D59B5CAAD568F083AD07946E6CB5817531680BCE6E215C16973CD39003B0425F3477CFD854E89A9DB6597"
        },
        ....
    }
}
```

为了改进客户端缓存，相对的 URL 会指向不可变的 OpenAPI 描述。 为了此目的，API 服务器也会设置正确的 HTTP 缓存标头 （`Expires` 为未来 1 年，和 `Cache-Control` 为 `immutable`）。 当一个过时的 URL 被使用时，API 服务器会返回一个指向最新 URL 的重定向。

Kubernetes API 服务器会在端点 `/openapi/v3/apis/<group>/<version>?hash=<hash>` 发布一个 Kubernetes 组版本的 OpenAPI v3 规范。

请参阅下表了解可接受的请求头部。

| 头部               | 可选值                                                       | 说明                       |
| ------------------ | ------------------------------------------------------------ | -------------------------- |
| `Accept-Encoding`  | `gzip`                                                       | *不提供此头部也是可接受的* |
| `Accept`           | `application/com.github.proto-openapi.spec.v3@v1.0+protobuf` | *主要用于集群内部使用*     |
| `application/json` | *默认*                                                       |                            |
| `*`                | *以* `application/json` 形式返回                             |                            |

#### 持久化

Kubernetes 通过将序列化状态的对象写入到 [etcd](https://kubernetes.io/zh-cn/docs/tasks/administer-cluster/configure-upgrade-etcd/) 中完成存储操作。

#### API 组和版本控制

为了更容易消除字段或重组资源的呈现方式，Kubernetes 支持多个 API 版本，每个版本位于不同的 API 路径， 例如 `/api/v1` 或 `/apis/rbac.authorization.k8s.io/v1alpha1`。

版本控制是在 API 级别而不是在资源或字段级别完成的，以确保 API 呈现出清晰、一致的系统资源和行为视图， 并能够控制对生命结束和/或实验性 API 的访问。

为了更容易演进和扩展其 API，Kubernetes 实现了 [API 组](https://kubernetes.io/zh-cn/docs/reference/using-api/#api-groups)， 这些 API 组可以被[启用或禁用](https://kubernetes.io/zh-cn/docs/reference/using-api/#enabling-or-disabling)。

API 资源通过其 API 组、资源类型、名字空间（用于名字空间作用域的资源）和名称来区分。 API 服务器透明地处理 API 版本之间的转换：所有不同的版本实际上都是相同持久化数据的呈现。 API 服务器可以通过多个 API 版本提供相同的底层数据。

例如，假设针对相同的资源有两个 API 版本：`v1` 和 `v1beta1`。 如果你最初使用其 API 的 `v1beta1` 版本创建了一个对象， 你稍后可以使用 `v1beta1` 或 `v1` API 版本来读取、更新或删除该对象。

#### API 变更

任何成功的系统都要随着新的使用案例的出现和现有案例的变化来成长和变化。 为此，Kubernetes 已设计了 Kubernetes API 来持续变更和成长。 Kubernetes 项目的目标是 **不要** 给现有客户端带来兼容性问题，并在一定的时期内维持这种兼容性， 以便其他项目有机会作出适应性变更。

一般而言，新的 API 资源和新的资源字段可以被频繁地添加进来。 删除资源或者字段则要遵从 [API 废弃策略](https://kubernetes.io/zh-cn/docs/reference/using-api/deprecation-policy/)。

Kubernetes 对维护达到正式发布（GA）阶段的官方 API 的兼容性有着很强的承诺， 通常这一 API 版本为 `v1`。此外，Kubernetes 在可能的时候还会保持 Beta API 版本的兼容性：如果你采用了 Beta API，你可以继续在集群上使用该 API， 即使该功能特性已进入稳定期也是如此。

**说明：**

尽管 Kubernetes 也努力为 **Alpha** API 版本维护兼容性，在有些场合兼容性是无法做到的。 如果你使用了任何 Alpha API 版本，需要在升级集群时查看 Kubernetes 发布说明， 以防 API 的确发生变更。

关于 API 版本分级的定义细节，请参阅 [API 版本参考](https://kubernetes.io/zh-cn/docs/reference/using-api/#api-versioning)页面。

#### API 扩展

有两种途径来扩展 Kubernetes API：

1. 你可以使用[自定义资源](https://kubernetes.io/zh-cn/docs/concepts/extend-kubernetes/api-extension/custom-resources/)来以声明式方式定义 API 服务器如何提供你所选择的资源 API。
2. 你也可以选择实现自己的[聚合层](https://kubernetes.io/zh-cn/docs/concepts/extend-kubernetes/api-extension/apiserver-aggregation/)来扩展 Kubernetes API。



### 使用 Kubernetes 对象

在 Kubernetes 系统中，**Kubernetes 对象** 是持久化的实体。 Kubernetes 使用**这些实体去表示整个集群的状态**。 比较特别地是，它们描述了如下信息：

- 哪些容器化应用正在运行（以及在哪些节点上运行）
- 可以被应用使用的资源
- 关于应用运行时表现的策略，比如重启策略、升级策略以及容错策略

Kubernetes 对象是“目标性记录” —— 一旦创建该对象，Kubernetes 系统将不断工作以确保该对象存在。 通**过创建对象，你就是在告知 Kubernetes 系统，你想要的集群工作负载状态看起来应是什么样子的**， 这就是 Kubernetes 集群所谓的 **期望状态（Desired State）**。

操作 Kubernetes 对象 —— 无论是创建、修改或者删除 —— 需要使用 [Kubernetes API](https://kubernetes.io/zh-cn/docs/concepts/overview/kubernetes-api)。 比如，当使用 `kubectl` 命令行接口（CLI）时，CLI 会调用必要的 Kubernetes API； 也可以在程序中使用[客户端库](https://kubernetes.io/zh-cn/docs/reference/using-api/client-libraries/)， 来直接调用 Kubernetes API。

#### 对象规约（Spec）与状态（Status）

几乎每个 Kubernetes 对象包含两个嵌套的对象字段，它们负责管理对象的配置： 对象 **`spec`（规约）** 和 对象 **`status`（状态）**。 对于具有 `spec` 的对象，你必须在创建对象时设置其内容，描述你希望对象所具有的特征： **期望状态（Desired State）**。

`status` 描述了对象的**当前状态（Current State）**，它是由 Kubernetes 系统和组件设置并更新的。 在任何时刻，Kubernetes [控制平面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane) 都一直都**在积极地管理着对象的实际状态，以使之达成期望状态**。

例如，Kubernetes 中的 Deployment 对象能够表示运行在集群中的应用。 当创建 Deployment 时，可能会去设置 Deployment 的 `spec`，以指定该应用要有 3 个副本运行。 Kubernetes 系统读取 Deployment 的 `spec`， 并启动我们所期望的应用的 3 个实例 —— 更新状态以与规约相匹配。 如果这些实例中有的失败了（一种状态变更），Kubernetes 系统会通过执行修正操作来响应 `spec` 和状态间的不一致 —— 意味着它会启动一个新的实例来替换。

关于对象 spec、status 和 metadata 的更多信息，可参阅 [Kubernetes API 约定](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md)。

#### 描述 Kubernetes 对象

**创建 Kubernetes 对象时，必须提供对象的 `spec`，用来描述该对象的期望状态， 以及关于对象的一些基本信息**（例如名称）。 当使用 Kubernetes API 创建对象时（直接创建，或经由 `kubectl`）， API 请求必须在请求本体中包含 JSON 格式的信息。 **大多数情况下，你需要提供 `.yaml` 文件为 kubectl 提供这些信息**。 `kubectl` 在发起 API 请求时，将这些信息转换成 JSON 格式。

这里有一个 `.yaml` 示例文件，展示了 Kubernetes Deployment 的必需字段和对象 `spec`：

[`application/deployment.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/zh-cn/examples/application/deployment.yaml) 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # 告知 Deployment 运行 2 个与该模板匹配的 Pod
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

相较于上面使用 `.yaml` 文件来创建 Deployment，另一种类似的方式是使用 `kubectl` 命令行接口（CLI）中的 [`kubectl apply`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply) 命令， 将 `.yaml` 文件作为参数。下面是一个示例：

```shell
kubectl apply -f https://k8s.io/examples/application/deployment.yaml
```

输出类似下面这样：

```
deployment.apps/nginx-deployment created
```

#### 必需字段

在想要创建的 Kubernetes 对象所对应的 `.yaml` 文件中，需要配置的字段如下：

- `apiVersion` - 创建该对象所使用的 Kubernetes API 的版本
- `kind` - 想要创建的对象的类别
- `metadata` - 帮助**唯一标识对象**的一些数据，包括一个 `name` 字符串、`UID` 和可选的 `namespace`
- `spec` - 你**所期望的该对象的状态**

对每个 Kubernetes 对象而言，其 `spec` 之精确格式都是不同的，包含了特定于该对象的嵌套字段。 我们能在 [Kubernetes API 参考](https://kubernetes.io/zh-cn/docs/reference/kubernetes-api/) 找到我们想要在 Kubernetes 上创建的任何对象的规约格式。

例如，参阅 Pod API 参考文档中 [`spec` 字段](https://kubernetes.io/zh-cn/docs/reference/kubernetes-api/workload-resources/pod-v1/#PodSpec)。 对于每个 Pod，其 `.spec` 字段设置了 Pod 及其期望状态（例如 Pod 中每个容器的容器镜像名称）。 另一个对象规约的例子是 StatefulSet API 中的 [`spec` 字段](https://kubernetes.io/zh-cn/docs/reference/kubernetes-api/workload-resources/stateful-set-v1/#StatefulSetSpec)。 对于 StatefulSet 而言，其 `.spec` 字段设置了 StatefulSet 及其期望状态。 在 StatefulSet 的 `.spec` 内，有一个为 Pod 对象提供的[模板](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/#pod-templates)。 该模板描述了 StatefulSet 控制器为了满足 StatefulSet 规约而要创建的 Pod。 不同类型的对象可以由不同的 `.status` 信息。API 参考页面给出了 `.status` 字段的详细结构， 以及针对不同类型 API 对象的具体内容。

## Kubernetes 架构

### 节点

Kubernetes 通过将容器放入在节点（Node）上运行的 Pod 中来执行你的工作负载。 **节点可以是一个虚拟机或者物理机器，取决于所在的集群配置**。 每个节点包含运行 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 所需的服务； 这些节点由 [控制面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane) 负责管理。

通常集群中会有若干个节点；而在一个学习所用或者资源受限的环境中，你的集群中也可能只有一个节点。

节点上的[组件](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#node-components)包括 [kubelet](https://kubernetes.io/docs/reference/generated/kubelet)、 [容器运行时](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes)以及 [kube-proxy](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-proxy/)。

#### 管理

向 [API 服务器](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#kube-apiserver)添加节点的方式主要有两种：

1. 节点上的 `kubelet` 向控制面执行自注册；
2. 你（或者别的什么人）手动添加一个 Node 对象。

在你创建了 Node [对象](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/kubernetes-objects/#kubernetes-objects)或者节点上的 `kubelet` 执行了自注册操作之后，**控制面会检查新的 Node 对象是否合法**。 例如，如果你尝试使用下面的 JSON 对象来创建 Node 对象：

```json
{
  "kind": "Node",
  "apiVersion": "v1",
  "metadata": {
    "name": "10.240.79.157",
    "labels": {
      "name": "my-first-k8s-node"
    }
  }
}
```

Kubernetes 会在内部创建一个 Node 对象作为节点的表示。Kubernetes 检查 `kubelet` 向 API 服务器注册节点时使用的 `metadata.name` 字段是否匹配。 如果节点是健康的（即所有必要的服务都在运行中），则该节点可以用来运行 Pod。 否则，直到该节点变为健康之前，所有的集群活动都会忽略该节点。

> **说明：**
>
> Kubernetes 会一直保存着非法节点对应的对象，并持续检查该节点是否已经变得健康。 你，或者某个[控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/)必须显式地删除该 Node 对象以停止健康检查操作。

Node 对象的名称必须是合法的 [DNS 子域名](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/names#dns-subdomain-names)。

##### 节点名称唯一性

节点的[名称](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/names#names)用来标识 Node 对象。 **没有两个 Node 可以同时使用相同的名称**。 Kubernetes 还**假定名字相同的资源是同一个对象**。 就 Node 而言，隐式假定使用相同名称的实例会具有相同的状态（例如网络配置、根磁盘内容） 和类似节点标签这类属性。这**可能在节点被更改但其名称未变时导致系统状态不一致**。 如果某个 Node 需要被替换或者大量变更，需要从 API 服务器移除现有的 Node 对象， 之后再在更新之后重新将其加入。

##### [节点自注册](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/#self-registration-of-nodes)

当 kubelet 标志 `--register-node` 为 true（默认）时，它会尝试向 API 服务注册自己。 这是首选模式，被绝大多数发行版选用。

对于自注册模式，kubelet 使用下列参数启动：

- `--kubeconfig` - 用于向 API 服务器执行身份认证所用的凭据的路径。
- `--cloud-provider` - 与某[云驱动](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-cloud-provider) 进行通信以读取与自身相关的元数据的方式。
- `--register-node` - 自动向 API 服务注册。
- `--register-with-taints` - 使用所给的[污点](https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/taint-and-toleration/)列表 （逗号分隔的 `<key>=<value>:<effect>`）注册节点。当 `register-node` 为 false 时无效。
- `--node-ip` - 节点 IP 地址。
- `--node-labels` - 在集群中注册节点时要添加的[标签](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/)。 （参见 [NodeRestriction 准入控制插件](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/admission-controllers/#noderestriction)所实施的标签限制）。
- `--node-status-update-frequency` - 指定 kubelet 向控制面发送状态的频率。

启用 [Node 鉴权模式](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/node/)和 [NodeRestriction 准入插件](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/admission-controllers/#noderestriction)时， 仅授权 `kubelet` 创建或修改其自己的节点资源。

**说明：**

正如[节点名称唯一性](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/#node-name-uniqueness)一节所述，当 Node 的配置需要被更新时， 一种好的做法是重新向 API 服务器注册该节点。例如，如果 kubelet 重启时其 `--node-labels` 是新的值集，但同一个 Node 名称已经被使用，则所作变更不会起作用， 因为节点标签是在 Node 注册时完成的。

如果在 kubelet 重启期间 Node 配置发生了变化，已经被调度到某 Node 上的 Pod 可能会出现行为不正常或者出现其他问题，例如，已经运行的 Pod 可能通过污点机制设置了与 Node 上新设置的标签相排斥的规则，也有一些其他 Pod， 本来与此 Pod 之间存在不兼容的问题，也会因为新的标签设置而被调到到同一节点。 节点重新注册操作可以确保节点上所有 Pod 都被排空并被正确地重新调度。

##### [手动节点管理](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/#manual-node-administration)

你可以使用 [kubectl](https://kubernetes.io/docs/user-guide/kubectl-overview/) 来创建和修改 Node 对象。

#### 节点状态

一个节点的状态包含以下信息:

- [地址（Addresses）](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/#addresses)
- [状况（Condition）](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/#condition)
- [容量与可分配（Capacity）](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/#capacity)
- [信息（Info）](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/#info)

你可以使用 `kubectl` 来查看节点状态和其他细节信息：

```shell
kubectl describe node <节点名称>
```

#### 心跳

Kubernetes 节点发送的心跳帮助你的集群确定每个节点的可用性，并在检测到故障时采取行动。

对于节点，有两种形式的心跳:

- 更新节点的 `.status`
- `kube-node-lease` [名字空间](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/namespaces/)中的 [Lease（租约）](https://kubernetes.io/zh-cn/docs/reference/kubernetes-api/cluster-resources/lease-v1/)对象。 每个节点都有一个关联的 Lease 对象。

#### 节点控制器

节点[控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/)是 Kubernetes 控制面组件， **管理节点的方方面面**。

节点控制器在节点的生命周期中扮演多个角色。 第一个是当节点注册时为它分配一个 CIDR（[无类别域间路由](https://zh.m.wikipedia.org/zh-hans/无类别域间路由)） 区段（如果启用了 CIDR 分配）。

第二个是**保持节点控制器内的节点列表与云服务商所提供的可用机器列表同步**。 如果在云环境下运行，只要某节点不健康，节点控制器就会询问云服务是否节点的虚拟机仍可用。 如果不可用，节点控制器会将该节点从它的节点列表删除。

第三个是**监控节点的健康状况**。节点控制器负责：

- 在节点不可达的情况下，在 Node 的 `.status` 中更新 `Ready` 状况。 在这种情况下，节点控制器将 NodeReady 状况更新为 `Unknown`。
- 如果节点仍然无法访问：对于不可达节点上的所有 Pod 触发 [API 发起的逐出](https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/api-eviction/)操作。 默认情况下，节点控制器在将节点标记为 `Unknown` 后等待 5 分钟提交第一个驱逐请求。

默认情况下，节点控制器每 5 秒检查一次节点状态，可以使用 `kube-controller-manager` 组件上的 `--node-monitor-period` 参数来配置周期。

### 节点与控制面之间的通信

#### 节点到控制面

**Kubernetes 采用的是中心辐射型（Hub-and-Spoke）API 模式**。 所有从节点（或运行于其上的 Pod）发出的 API 调用都终止于 API 服务器。 其它控制面组件都没有被设计为可暴露远程服务。 **API 服务器被配置为在一个安全的 HTTPS 端口（通常为 443）上监听远程连接请求， 并启用一种或多种形式的客户端[身份认证](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/authentication/)机制**。 一种或多种客户端[鉴权机制](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/authorization/)应该被启用， 特别是在允许使用[匿名请求](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/authentication/#anonymous-requests) 或[服务账户令牌](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/authentication/#service-account-tokens)的时候。

应该使用集群的公共根证书开通节点，这样它们就能够基于有效的客户端凭据安全地连接 API 服务器。 一种好的方法是以客户端证书的形式将客户端凭据提供给 kubelet。 请查看 [kubelet TLS 启动引导](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/kubelet-tls-bootstrapping/) 以了解如何自动提供 kubelet 客户端证书。

想要连接到 API 服务器的 Pod 可以使用服务账号安全地进行连接。 当 Pod 被实例化时，Kubernetes 自动把公共根证书和一个有效的持有者令牌注入到 Pod 里。 `kubernetes` 服务（位于 `default` 名字空间中）配置了一个虚拟 IP 地址， 用于（通过 kube-proxy）转发请求到 API 服务器的 HTTPS 末端。

控制面组件也通过安全端口与集群的 API 服务器通信。

这样，从集群节点和节点上运行的 Pod 到控制面的连接的缺省操作模式即是安全的， 能够在不可信的网络或公网上运行。

#### 控制面到节点

从控制面（API 服务器）到节点有两种主要的通信路径。 第一种是从 API 服务器到集群中**每个节点上运行的 kubelet 进程**。 第二种是从 API 服务器通过它的代理功能**连接到任何节点、Pod 或者服务**。

##### API 服务器到 kubelet

从 API 服务器到 kubelet 的连接用于：

- 获取 Pod 日志
- 挂接（通过 kubectl）到运行中的 Pod
- 提供 kubelet 的端口转发功能。

这些连接终止于 kubelet 的 HTTPS 末端。 默认情况下，API 服务器不检查 kubelet 的服务证书。这使得此类连接容易受到中间人攻击， 在非受信网络或公开网络上运行也是 **不安全的**。

为了对这个连接进行认证，使用 `--kubelet-certificate-authority` 标志给 API 服务器提供一个根证书包，用于 kubelet 的服务证书。

如果无法实现这点，又要求避免在非受信网络或公共网络上进行连接，可在 API 服务器和 kubelet 之间使用 [SSH 隧道](https://kubernetes.io/zh-cn/docs/concepts/architecture/control-plane-node-communication/#ssh-tunnels)。

最后，应该启用 [Kubelet 认证/鉴权](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/kubelet-authn-authz/) 来保护 kubelet API。

##### API 服务器到节点、Pod 和服务

**从 API 服务器到节点、Pod 或服务的连接默认为纯 HTTP 方式**，因此既没有认证，也没有加密。 这些连接可通过给 API URL 中的节点、Pod 或服务名称添加前缀 `https:` 来运行在安全的 HTTPS 连接上。 不过这些连接既不会验证 HTTPS 末端提供的证书，也不会提供客户端证书。 因此，虽然连接是加密的，仍无法提供任何完整性保证。 这些连接 **目前还不能安全地** 在非受信网络或公共网络上运行。

##### SSH 隧道

Kubernetes 支持使用 SSH 隧道来保护从控制面到节点的通信路径。在这种配置下， API 服务器建立一个到集群中各节点的 SSH 隧道（连接到在 22 端口监听的 SSH 服务器） 并通过这个隧道传输所有到 kubelet、节点、Pod 或服务的请求。 这一隧道保证通信不会被暴露到集群节点所运行的网络之外。

> **说明：**
>
> SSH 隧道目前已被废弃。除非你了解个中细节，否则不应使用。 [Konnectivity 服务](https://kubernetes.io/zh-cn/docs/concepts/architecture/control-plane-node-communication/#konnectivity-service)是 SSH 隧道的替代方案。

##### Konnectivity 服务

**特性状态：** `Kubernetes v1.18 [beta]`

作为 SSH 隧道的替代方案，**Konnectivity 服务提供 TCP 层的代理，以便支持从控制面到集群的通信**。 Konnectivity 服务包含两个部分：Konnectivity 服务器和 Konnectivity 代理， 分别运行在控制面网络和节点网络中。 Konnectivity 代理建立并维持到 Konnectivity 服务器的网络连接。 启用 Konnectivity 服务之后，所有控制面到节点的通信都通过这些连接传输。

请浏览 [Konnectivity 服务任务](https://kubernetes.io/zh-cn/docs/tasks/extend-kubernetes/setup-konnectivity/) 在你的集群中配置 Konnectivity 服务。

### 控制器

在 Kubernetes 中，控制器通过监控[集群](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-cluster) 的公共状态，并致力于将当前状态转变为期望的状态。

#### 控制器模式

**一个控制器至少追踪一种类型的 Kubernetes 资源**。这些 [对象](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/kubernetes-objects/#kubernetes-objects) 有一个代表期望状态的 `spec` 字段。 该资源的控制器负责确保其当前状态接近期望状态。

**控制器可能会自行执行操作**；在 Kubernetes 中更常见的是一个控制器会发送信息给 [API 服务器](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#kube-apiserver)，这会有副作用。 具体可参看后文的例子。

##### 通过 API 服务器来控制

[Job](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/job/) 控制器是一个 Kubernetes 内置控制器的例子。 内置控制器通过和集群 API 服务器交互来管理状态。

Job 是一种 Kubernetes 资源，它运行一个或者多个 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/)， 来执行一个任务然后停止。 （一旦[被调度了](https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/)，对 `kubelet` 来说 Pod 对象就会变成了期望状态的一部分）。

在集群中，当 Job 控制器拿到新任务时，它会保证一组 Node 节点上的 `kubelet` 可以运行正确数量的 Pod 来完成工作。 Job 控制器不会自己运行任何的 Pod 或者容器。Job 控制器是通知 API 服务器来创建或者移除 Pod。 [控制面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane)中的其它组件 根据新的消息作出反应（调度并运行新 Pod）并且最终完成工作。

创建新 Job 后，所期望的状态就是完成这个 Job。Job 控制器会让 Job 的当前状态不断接近期望状态：创建为 Job 要完成工作所需要的 Pod，使 Job 的状态接近完成。

控制器也会更新配置对象。例如：一旦 Job 的工作完成了，Job 控制器会更新 Job 对象的状态为 `Finished`。

（这有点像温度自动调节器关闭了一个灯，以此来告诉你房间的温度现在到你设定的值了）。

##### 直接控制

相比 Job 控制器，有些控制器需要对集群外的一些东西进行修改。

例如，如果你使用一个控制回路来保证集群中有足够的 [节点](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/)，那么控制器就需要当前集群外的 一些服务在需要时创建新节点。

和外部状态交互的控制器从 API 服务器获取到它想要的状态，然后直接和外部系统进行通信 并使当前状态更接近期望状态。

（实际上有一个[控制器](https://github.com/kubernetes/autoscaler/) 可以水平地扩展集群中的节点。）

这里，很重要的一点是，控制器做出了一些变更以使得事物更接近你的期望状态， 之后将当前状态报告给集群的 API 服务器。 其他控制回路可以观测到所汇报的数据的这种变化并采取其各自的行动。

在温度计的例子中，如果房间很冷，那么某个控制器可能还会启动一个防冻加热器。 就 Kubernetes 集群而言，控制面间接地与 IP 地址管理工具、存储服务、云驱动 APIs 以及其他服务协作，通过[扩展 Kubernetes](https://kubernetes.io/zh-cn/docs/concepts/extend-kubernetes/) 来实现这点。

#### 期望状态与当前状态

Kubernetes 采用了系统的云原生视图，并且可以处理持续的变化。

在任务执行时，集群随时都可能被修改，并且控制回路会自动修复故障。 这意味着很可能集群永远不会达到稳定状态。

只要集群中的控制器在运行并且进行有效的修改，整体状态的稳定与否是无关紧要的。

#### 设计

作为设计原则之一，Kubernetes 使用了很多控制器，**每个控制器管理集群状态的一个特定方面**。 最常见的**一个特定的控制器使用一种类型的资源作为它的期望状态， 控制器管理控制另外一种类型的资源向它的期望状态演化**。 例如，Job 的控制器跟踪 Job 对象（以发现新的任务）和 Pod 对象（以运行 Job，然后查看任务何时完成）。 在这种情况下，新任务会创建 Job，而 Job 控制器会创建 Pod。

使用简单的控制器而不是一组相互连接的单体控制回路是很有用的。 控制器会失败，所以 Kubernetes 的设计正是考虑到了这一点。

> **说明：**
>
> 可以有多个控制器来创建或者更新相同类型的对象。 在后台，Kubernetes 控制器确保它们只关心与其控制资源相关联的资源。
>
> 例如，你可以创建 Deployment 和 Job；它们都可以创建 Pod。 Job 控制器不会删除 Deployment 所创建的 Pod，因为有信息 （[标签](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/)）让控制器可以区分这些 Pod。

#### 运行控制器的方式

Kubernetes 内置一组控制器，运行在 [kube-controller-manager](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-controller-manager/) 内。 这些内置的控制器提供了重要的核心功能。

Deployment 控制器和 Job 控制器是 Kubernetes 内置控制器的典型例子。 Kubernetes 允许你运行一个稳定的控制平面，这样即使某些内置控制器失败了， 控制平面的其他部分会接替它们的工作。

你**会遇到某些控制器运行在控制面之外，用以扩展 Kubernetes**。 或者，如果你愿意，你也可以自己编写新控制器。 你可以以一组 Pod 来运行你的控制器，或者运行在 Kubernetes 之外。 最合适的方案取决于控制器所要执行的功能是什么。

### 云控制器管理器

使用云基础设施技术，你可以在公有云、私有云或者混合云环境中运行 Kubernetes。 Kubernetes 的信条是基于自动化的、API 驱动的基础设施，同时避免组件间紧密耦合。

组件 cloud-controller-manager 是指云控制器管理器， 一个 Kubernetes [控制平面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane)组件， 嵌入了特定于云平台的控制逻辑。 云控制器管理器（Cloud Controller Manager）允许你将你的集群连接到云提供商的 API 之上， 并将与该云平台交互的组件同与你的集群交互的组件分离开来。

通过分离 Kubernetes 和底层云基础设置之间的互操作性逻辑， `cloud-controller-manager` 组件使云提供商能够以不同于 Kubernetes 主项目的步调发布新特征。

`cloud-controller-manager` 组件是**基于一种插件机制来构造的**， 这种机制使得不同的云厂商都能将其平台与 Kubernetes 集成。

- 云控制器管理器的功能
  - [节点控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/cloud-controller/#node-controller)
  - [路由控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/cloud-controller/#route-controller)
  - [服务控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/cloud-controller/#service-controller)
- 鉴权
  - [节点控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/cloud-controller/#authorization-node-controller)
  - [路由控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/cloud-controller/#authorization-route-controller)
  - [服务控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/cloud-controller/#authorization-service-controller)
  - [其他](https://kubernetes.io/zh-cn/docs/concepts/architecture/cloud-controller/#authorization-miscellaneous)

### 垃圾收集

垃圾收集是 Kubernetes **用于清理集群资源的各种机制的统称**。 垃圾收集允许系统清理如下资源：

- [终止的 Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#pod-garbage-collection)

- [已完成的 Job](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/ttlafterfinished/)

- [不再存在属主引用的对象](https://kubernetes.io/zh-cn/docs/concepts/architecture/garbage-collection/#owners-dependents)

- [未使用的容器和容器镜像](https://kubernetes.io/zh-cn/docs/concepts/architecture/garbage-collection/#containers-images)

- [动态制备的、StorageClass 回收策略为 Delete 的 PV 卷](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/#delete)

- [阻滞或者过期的 CertificateSigningRequest (CSRs)](https://kubernetes.io/zh-cn/docs/reference/access-authn-authz/certificate-signing-requests/#request-signing-process)

- 在以下情形中删除了的

  节点

  对象：

  - 当集群使用[云控制器管理器](https://kubernetes.io/zh-cn/docs/concepts/architecture/cloud-controller/)运行于云端时；
  - 当集群使用类似于云控制器管理器的插件运行在本地环境中时。

- [节点租约对象](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/#heartbeats)

- [属主与依赖](https://kubernetes.io/zh-cn/docs/concepts/architecture/garbage-collection/#owners-dependents)
- 级联删除
  - [前台级联删除](https://kubernetes.io/zh-cn/docs/concepts/architecture/garbage-collection/#foreground-deletion)
  - [后台级联删除](https://kubernetes.io/zh-cn/docs/concepts/architecture/garbage-collection/#background-deletion)
  - [被遗弃的依赖对象](https://kubernetes.io/zh-cn/docs/concepts/architecture/garbage-collection/#orphaned-dependents)
- 未使用容器和镜像的垃圾收集
  - [容器镜像生命周期](https://kubernetes.io/zh-cn/docs/concepts/architecture/garbage-collection/#container-image-lifecycle)
  - [容器垃圾收集](https://kubernetes.io/zh-cn/docs/concepts/architecture/garbage-collection/#container-image-garbage-collection)
- [配置垃圾收集](https://kubernetes.io/zh-cn/docs/concepts/architecture/garbage-collection/#configuring-gc)

### 容器运行时接口（CRI）



## 容器

每个运行的容器都是可重复的； 包含依赖环境在内的标准，意味着无论你在哪里运行它都会得到相同的行为。

容器将应用程序从底层的主机设施中解耦。 这使得在不同的云或 OS 环境中部署更加容易。

### 容器镜像

[容器镜像](https://kubernetes.io/zh-cn/docs/concepts/containers/images/)是一个随时可以运行的软件包， 包含运行应用程序所需的一切：代码和它需要的所有运行时、应用程序和系统库，以及一些基本设置的默认值。

根据设计，**容器是不可变的：你不能更改已经运行的容器的代码**。 如果有一个容器化的应用程序需要修改，则需要构建包含更改的新镜像，然后再基于新构建的镜像重新运行容器。

### 容器运行时

容器运行环境是负责运行容器的软件。

Kubernetes 支持许多容器运行环境，例如 [containerd](https://containerd.io/docs/)、 [CRI-O](https://cri-o.io/#what-is-cri-o) 以及 [Kubernetes CRI (容器运行环境接口)](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md) 的其他任何实现。

### 容器环境

Kubernetes 的容器环境给容器提供了几个重要的资源：

- 文件系统，其中包含一个[镜像](https://kubernetes.io/zh-cn/docs/concepts/containers/images/) 和一个或多个的[卷](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/)
- 容器自身的信息
- 集群中其他对象的信息

### 容器生命周期回调



## 工作负载

工作负载是在 Kubernetes 上运行的应用程序。

在 Kubernetes 中，无论你的负载是**由单个组件还是由多个一同工作的组件构成**， 你都可以在一组 [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods) 中运行它。 在 Kubernetes 中，Pod 代表的是集群上处于运行状态的一组 [容器](https://kubernetes.io/zh-cn/docs/concepts/overview/what-is-kubernetes/#why-containers) 的集合。

Kubernetes Pods 遵循[预定义的生命周期](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/)。 例如，当在你的集群中运行了某个 Pod，但是 Pod 所在的 [节点](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/) 出现致命错误时， 所有该节点上的 Pods 的状态都会变成失败。Kubernetes 将这类失败视为最终状态： 即使该节点后来恢复正常运行，你也需要创建新的 Pod 以恢复应用。

不过，为了减轻用户的使用负担，通常不需要用户直接管理每个 Pod。 而是使用**负载资源**来替用户管理一组 Pod。 这些负载资源通过配置 [控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/) 来确保正确类型的、处于运行状态的 Pod 个数是正确的，与用户所指定的状态相一致。

Kubernetes 提供若干种内置的工作负载资源：

- [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/) 和 [ReplicaSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/replicaset/) （替换原来的资源 [ReplicationController](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-replication-controller)）。 `Deployment` 很适合用来管理你的集群上的**无状态应用**，`Deployment` 中的所有 `Pod` 都是相互等价的，并且在需要的时候被替换。

  > 无状态：**任意一个Web请求**端提出请求时，请求本身包含了响应端为响应这一**请求所需的全部信息**（认证信息等）
  >
  > 有状态：**Web请求**端的请求**必须被提交到保存有其相关状态信息（比如session）的服务器上**，否则这些请求可能无法被理解，这也就意味着在此模式下**服务器端无法对用户请求进行自由调度**。
  >
  > 再说直白一点就是状态（公共交互）信息是由请求方还是响应方负责保存，请求方保存就是无状态，响应方保存就是有状态。
  >
  > 无状态应用不关心响应方是谁，需不需要同步各个响应方之间的信息，响应服务可随时被删除也不会影响别人，容错性高，分布式服务的负载均衡失效不会丢数据，无内存消耗，直接部署上线即可使用
  >
  > 有状态应用需要及时同步数据，可能存在数据同步不玩丢失数据，消耗内存资源保存数据等。

- [StatefulSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/) 让你能够运行一个或者多个以某种方式跟踪应用状态的 Pods。 例如，如果你的负载会将数据作持久存储，你可以运行一个 `StatefulSet`，将每个 `Pod` 与某个 [`PersistentVolume`](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/) 对应起来。你在 `StatefulSet` 中各个 `Pod` 内运行的代码可以将数据复制到同一 `StatefulSet` 中的其它 `Pod` 中以提高整体的服务可靠性。

- [DaemonSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/daemonset/) 定义提供节点本地支撑设施的 `Pods`。这些 Pods 可能对于你的集群的运维是 非常重要的，例如作为网络链接的辅助工具或者作为网络 [插件](https://kubernetes.io/zh-cn/docs/concepts/cluster-administration/addons/) 的一部分等等。每次你向集群中添加一个新节点时，如果该节点与某 `DaemonSet` 的规约匹配，则控制面会为该 `DaemonSet` 调度一个 `Pod` 到该新节点上运行。
- [Job](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/job/) 和 [CronJob](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/cron-jobs/)。 定义一些一直运行到结束并停止的任务。`Job` 用来表达的是一次性的任务，而 `CronJob` 会根据其时间规划反复运行。

在庞大的 Kubernetes 生态系统中，你还可以找到一些提供额外操作的第三方工作负载相关的资源。 通过使用[定制资源定义（CRD）](https://kubernetes.io/zh-cn/docs/concepts/extend-kubernetes/api-extension/custom-resources/)， 你可以添加第三方工作负载资源，以完成原本不是 Kubernetes 核心功能的工作。 例如，如果你希望运行一组 `Pods`，但要求所有 Pods 都可用时才执行操作 （比如针对某种高吞吐量的分布式任务），你可以基于定制资源实现一个能够满足这一需求的扩展， 并将其安装到集群中运行。

### Pod

**Pod** 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。

**Pod**（就像在鲸鱼荚或者豌豆荚中）是**一组（一个或多个）** [容器](https://kubernetes.io/zh-cn/docs/concepts/overview/what-is-kubernetes/#why-containers)； 这些容器共享存储、网络、以及怎样运行这些容器的声明。 Pod 中的内容总是并置（colocated）的并且一同调度，在共享的上下文中运行。 Pod 所建模的是特定于应用的 “逻辑主机”，其中包含一个或多个应用容器， 这些容器相对紧密地耦合在一起。 在非云环境中，在相同的物理机或虚拟机上运行的应用类似于在同一逻辑主机上运行的云应用。

除了应用容器，Pod 还可以包含在 Pod 启动期间运行的 [Init 容器](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/init-containers/)。 你也可以在集群中支持[临时性容器](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/ephemeral-containers/) 的情况下，为调试的目的注入临时性容器。

#### 什么是 Pod？

**说明：** 除了 Docker 之外，Kubernetes 支持 很多其他[容器运行时](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes)， [Docker](https://www.docker.com/) 是最有名的运行时， 使用 Docker 的术语来描述 Pod 会很有帮助。

**Pod 的共享上下文包括一组 Linux 名字空间、控制组（cgroup）和可能一些其他的隔离方面**， 即用来隔离 Docker 容器的技术。 在 Pod 的上下文中，每个独立的应用可能会进一步实施隔离。

就 Docker 概念的术语而言，Pod 类似于共享名字空间和文件系统卷的一组 Docker 容器。

#### 使用 Pod

你很少在 Kubernetes 中直接创建一个个的 Pod，甚至是单实例（Singleton）的 Pod。 这是因为 Pod 被设计成了相对临时性的、用后即抛的一次性实体。 当 Pod 由你或者间接地由[控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/) 创建时，它被调度在集群中的[节点](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/)上运行。 Pod 会保持在该节点上运行，直到 Pod 结束执行、Pod 对象被删除、Pod 因资源不足而被**驱逐**或者节点失效为止。

**说明：** 重启 Pod 中的容器不应与重启 Pod 混淆。 **Pod 不是进程，而是容器运行的环境**。 在被删除之前，Pod 会一直存在。

当你为 Pod 对象创建清单时，要确保所指定的 Pod 名称是合法的 [DNS 子域名](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/names#dns-subdomain-names)。

下面是一个 Pod 示例，它由一个运行镜像 `nginx:1.14.2` 的容器组成。

[`pods/simple-pod.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/zh-cn/examples/pods/simple-pod.yaml) 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

要创建上面显示的 Pod，请运行以下命令：

```shell
kubectl apply -f https://k8s.io/examples/pods/simple-pod.yaml
```

**Pod 通常不是直接创建的，而是使用工作负载资源创建的**。 有关如何将 Pod 用于工作负载资源的更多信息，请参阅[使用 Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/#working-with-pods)。

##### 用于管理 pod 的工作负载资源

通常你不需要直接创建 Pod，甚至单实例 Pod。 相反，你会使用诸如 [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/) 或 [Job](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/job/) 这类工作负载资源来创建 Pod。 如果 Pod 需要跟踪状态，可以考虑 [StatefulSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/) 资源。

Kubernetes 集群中的 Pod 主要有两种用法：

- **运行单个容器的 Pod**。"每个 Pod 一个容器" 模型是最常见的 Kubernetes 用例； 在这种情况下，可以将 Pod 看作单个容器的包装器，并且 Kubernetes 直接管理 Pod，而不是容器。

- **运行多个协同工作的容器的 Pod**。 Pod 可能封装由多个紧密耦合且需要共享资源的共处容器组成的应用程序。 这些位于同一位置的容器可能形成单个内聚的服务单元 —— 一个容器将文件从共享卷提供给公众， 而另一个单独的 “边车”（sidecar）容器则刷新或更新这些文件。 Pod 将这些容器和存储资源打包为一个可管理的实体。

  **说明：** 将多个并置、同管的容器组织到一个 Pod 中是一种相对高级的使用场景。 只有在一些场景中，**容器之间紧密关联时你才应该使用这种模式**。

每个 Pod 都旨在运行给定应用程序的单个实例。如果希望横向扩展应用程序 （例如，运行多个实例以提供更多的资源），则应该使用多个 Pod，每个实例使用一个 Pod。 在 Kubernetes 中，这通常被称为**副本（Replication）**。 通常使用一种工作负载资源及其[控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/) 来创建和管理一组 Pod 副本。

参见 [Pod 和控制器](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/#pods-and-controllers)以了解 Kubernetes 如何使用工作负载资源及其控制器以实现应用的扩缩和自动修复

##### Pod 和控制器

你可以使用工作负载资源来创建和管理多个 Pod。 资源的控制器能够处理副本的管理、上线，并在 Pod 失效时提供自愈能力。 例如，如果一个节点失败，控制器注意到该节点上的 Pod 已经停止工作， 就可以创建替换性的 Pod。调度器会将替身 Pod 调度到一个健康的节点执行。

下面是一些管理一个或者多个 Pod 的工作负载资源的示例：

- [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/)
- [StatefulSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/)
- [DaemonSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/daemonset/)

##### Pod 模板

[负载](https://kubernetes.io/zh-cn/docs/concepts/workloads/)资源的控制器通常使用 **Pod 模板（Pod Template）**来替你创建 Pod 并管理它们。

Pod 模板是包含在工作负载对象中的规范，用来创建 Pod。这类负载资源包括 [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/)、 [Job](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/job/) 和 [DaemonSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/daemonset/) 等。

工作负载的控制器会使用负载对象中的 `PodTemplate` 来生成实际的 Pod。 `PodTemplate` 是你用来运行应用时指定的负载资源的目标状态的一部分。

下面的示例是一个简单的 Job 的清单，其中的 `template` 指示启动一个容器。 该 Pod 中的容器会打印一条消息之后暂停。

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello
spec:
  template:
    # 这里是 Pod 模板
    spec:
      containers:
      - name: hello
        image: busybox:1.28
        command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']
      restartPolicy: OnFailure
    # 以上为 Pod 模板
```

修改 Pod 模板或者切换到新的 Pod 模板都不会对已经存在的 Pod 直接起作用。 如果改变工作负载资源的 Pod 模板，工作负载资源需要使用更新后的模板来创建 Pod， 并使用新创建的 Pod 替换旧的 Pod。

例如，StatefulSet 控制器针对每个 StatefulSet 对象确保运行中的 Pod 与当前的 Pod 模板匹配。如果编辑 StatefulSet 以更改其 Pod 模板， StatefulSet 将开始基于更新后的模板创建新的 Pod。

每个工作负载资源都实现了自己的规则，用来处理对 Pod 模板的更新。 如果你想了解更多关于 StatefulSet 的具体信息， 请阅读 StatefulSet 基础教程中的[更新策略](https://kubernetes.io/zh-cn/docs/tutorials/stateful-application/basic-stateful-set/#updating-statefulsets)。

在节点上，[kubelet](https://kubernetes.io/docs/reference/generated/kubelet) 并不直接监测 或管理与 Pod 模板相关的细节或模板的更新，这些细节都被抽象出来。 这种抽象和关注点分离简化了整个系统的语义， 并且使得用户可以在不改变现有代码的前提下就能扩展集群的行为。

##### Pod 怎样管理多个容器

Pod 被设计成支持形成内聚服务单元的多个协作过程（形式为容器）。 Pod 中的容器被自动安排到集群中的同一物理机或虚拟机上，并可以一起进行调度。 **容器之间可以共享资源和依赖、彼此通信、协调何时以及何种方式终止自身**。

例如，你可能有一个容器，为共享卷中的文件提供 Web 服务器支持，以及一个单独的 "边车 (sidercar)" 容器负责从远端更新这些文件，如下图所示：

<img src="https://d33wubrfki0l68.cloudfront.net/aecab1f649bc640ebef1f05581bfcc91a48038c4/728d6/images/docs/pod.svg" alt="Pod 创建示意图" />

有些 Pod 具有 [Init 容器](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-init-container)和 [应用容器](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-app-container)。 Init 容器会在启动应用容器之前运行并完成。

Pod 天生地为其成员容器提供了两种共享资源：[网络](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/#pod-networking)和[存储](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/#pod-storage)。

#### Pod 更新与替换

正如前面章节所述，当某工作负载的 Pod 模板被改变时， 控制器会**基于更新的模板创建新的 Pod 对象而不是对现有 Pod 执行更新或者修补操作**。

Kubernetes 并不禁止你直接管理 Pod。对运行中的 Pod 的**某些字段执行就地更新操作** 还是可能的。不过，类似 [`patch`](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.25/#patch-pod-v1-core) 和 [`replace`](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.25/#replace-pod-v1-core) 这类更新操作有一些限制：

- **Pod 的绝大多数元数据都是不可变的**。例如，你不可以改变其 `namespace`、`name`、 `uid` 或者 `creationTimestamp` 字段；`generation` 字段是比较特别的， 如果更新该字段，只能增加字段取值而不能减少。
- 如果 `metadata.deletionTimestamp` 已经被设置，则不可以向 `metadata.finalizers` 列表中添加新的条目。
- Pod 更新不可以改变除 `spec.containers[*].image`、`spec.initContainers[*].image`、 `spec.activeDeadlineSeconds` 或 `spec.tolerations` 之外的字段。 对于 `spec.tolerations`，你只被允许添加新的条目到其中。
- 在更新`spec.activeDeadlineSeconds` 字段时，以下两种更新操作是被允许的：
  1. 如果该字段尚未设置，可以将其设置为一个正数；
  2. 如果该字段已经设置为一个正数，可以将其设置为一个更小的、非负的整数。

##### 资源共享和通信

Pod 使它的成员容器间能够进行数据共享和通信。

##### Pod 中的存储

一个 Pod 可以设置一组共享的存储[卷](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/)。 Pod 中的所有容器都可以访问该共享卷，从而允许这些容器共享数据。 卷还允许 Pod 中的持久数据保留下来，即使其中的容器需要重新启动。 有关 Kubernetes 如何在 Pod 中实现共享存储并将其提供给 Pod 的更多信息， 请参考[卷](https://kubernetes.io/zh-cn/docs/concepts/storage/)。

##### Pod 联网

每个 Pod 都在每个地址族中获得一个唯一的 IP 地址。 **Pod 中的每个容器共享网络名字空间，包括 IP 地址和网络端口**。 **Pod 内**的容器可以使用 `localhost` 互相通信。 当 Pod 中的容器与 **Pod 之外**的实体通信时，它们必须协调如何使用共享的网络资源（例如端口）。

在同一个 Pod 内，所有容器共享一个 IP 地址和端口空间，并且可以通过 `localhost` 发现对方。 他们也**能通过如 SystemV 信号量或 POSIX 共享内存这类标准的进程间通信方式互相通信**。 不同 Pod 中的容器的 IP 地址互不相同，如果没有特殊配置，就无法通过 OS 级 IPC 进行通信。 如果某容器希望与运行于其他 Pod 中的容器通信，可以通过 IP 联网的方式实现。

Pod 中的容器所看到的系统主机名与为 Pod 配置的 `name` 属性值相同。 [网络](https://kubernetes.io/zh-cn/docs/concepts/cluster-administration/networking/)部分提供了更多有关此内容的信息。

#### 容器的特权模式

在 Linux 中，Pod 中的任何容器都可以使用容器规约中的 [安全性上下文](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/security-context/)中的 `privileged`（Linux）参数启用特权模式。 这对于想要使用操作系统管理权能（Capabilities，如操纵网络堆栈和访问设备）的容器很有用。

如果你的集群启用了 `WindowsHostProcessContainers` 特性，你可以使用 Pod 规约中安全上下文的 `windowsOptions.hostProcess` 参数来创建 [Windows HostProcess Pod](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/create-hostprocess-pod/)。 这些 Pod 中的所有容器都必须以 Windows HostProcess 容器方式运行。 HostProcess Pod 可以直接运行在主机上，它也能像 Linux 特权容器一样，用于执行管理任务。

**说明：** 你的[容器运行时](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes) 必须支持特权容器的概念才能使用这一配置。

#### 静态 Pod

**静态 Pod（Static Pod）**直接由特定节点上的 `kubelet` 守护进程管理， 不需要 [API 服务器](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#kube-apiserver)看到它们。 尽管大多数 Pod 都是通过控制面（例如，[Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/)） 来管理的，对于静态 Pod 而言，`kubelet` 直接监控每个 Pod，并在其失效时重启之。

静态 Pod 通常绑定到某个节点上的 [kubelet](https://kubernetes.io/docs/reference/generated/kubelet)。 其主要用途是运行自托管的控制面。 在自托管场景中，使用 `kubelet` 来管理各个独立的 [控制面组件](https://kubernetes.io/zh-cn/docs/concepts/overview/components/#control-plane-components)。

`kubelet` 自动尝试为每个静态 Pod 在 Kubernetes API 服务器上创建一个 [镜像 Pod](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-mirror-pod)。 这意味着在节点上运行的 Pod 在 API 服务器上是可见的，但不可以通过 API 服务器来控制。

**说明：**

静态 Pod 的 `spec` 不能引用其他的 API 对象（例如： [ServiceAccount](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-service-account/)、 [ConfigMap](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-pod-configmap/)、 [Secret](https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/) 等）。

#### 容器探针

**Probe** 是由 kubelet 对容器执行的定期诊断。要执行诊断，kubelet 可以执行三种动作：

- `ExecAction`（借助容器运行时执行）
- `TCPSocketAction`（由 kubelet 直接检测）
- `HTTPGetAction`（由 kubelet 直接检测）

#### Pod 的生命周期

 Pod 遵循一个预定义的生命周期，起始于 `Pending` [阶段](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase)，如果至少 其中有一个主要容器正常启动，则进入 `Running`，之后取决于 Pod 中是否有容器以 失败状态结束而进入 `Succeeded` 或者 `Failed` 阶段。

在 Pod 运行期间，`kubelet` 能够重启容器以处理一些失效场景。 在 Pod 内部，Kubernetes 跟踪不同容器的[状态](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#container-states) 并确定使 Pod 重新变得健康所需要采取的动作。

在 Kubernetes API 中，Pod 包含规约部分和实际状态部分。 Pod 对象的状态包含了一组 [Pod 状况（Conditions）](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#pod-conditions)。 如果应用需要的话，你也可以向其中注入[自定义的就绪性信息](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#pod-readiness-gate)。

Pod 在其生命周期中**只会被[调度](https://kubernetes.io/zh-cn/docs/concepts/scheduling-eviction/)一次**。 一旦 Pod 被调度（分派）到某个节点，Pod 会一直在该节点运行，直到 Pod 停止或者 被[终止](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination)。

和一个个独立的应用容器一样，**Pod 也被认为是相对临时性（而不是长期存在）的实体**。 Pod 会被创建、赋予一个唯一的 ID（[UID](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/names/#uids)）， 并被调度到节点，并在终止（根据重启策略）或删除之前一直运行在该节点。

如果一个[节点](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/)死掉了，调度到该节点 的 Pod 也被计划在给定超时期限结束后[删除](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#pod-garbage-collection)。

Pod 自身不具有自愈能力。如果 Pod 被调度到某[节点](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/) 而该节点之后失效，Pod 会被删除；类似地，Pod 无法在因节点资源 耗尽或者节点维护而被驱逐期间继续存活。Kubernetes 使用一种高级抽象 来管理这些相对而言可随时丢弃的 Pod 实例，称作 [控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/)。

任何给定的 Pod （由 UID 定义）从不会被“重新调度（rescheduled）”到不同的节点； 相反，这一 Pod 可以被一个新的、几乎完全相同的 Pod 替换掉。 如果需要，新 Pod 的名字可以不变，但是其 UID 会不同。

如果某物声称其生命期与某 Pod 相同，例如存储[卷](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/)， 这就意味着该对象在此 Pod （UID 亦相同）存在期间也一直存在。 如果 Pod 因为任何原因被删除，甚至某完全相同的替代 Pod 被创建时， 这个相关的对象（例如这里的卷）也会被删除并重建。

#### Pod 阶段

Pod 的 `status` 字段是一个 [PodStatus](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.25/#podstatus-v1-core) 对象，其中包含一个 `phase` 字段。

Pod 的阶段（Phase）是 Pod 在其生命周期中所处位置的简单宏观概述。 该阶段并不是对容器或 Pod 状态的综合汇总，也不是为了成为完整的状态机。

Pod 阶段的数量和含义是严格定义的。 除了本文档中列举的内容外，不应该再假定 Pod 有其他的 `phase` 值。

下面是 `phase` 可能的值：

| 取值                | 描述                                                         |
| :------------------ | :----------------------------------------------------------- |
| `Pending`（悬决）   | Pod 已被 Kubernetes 系统接受，但有一个或者多个容器尚未创建亦未运行。此阶段包括等待 Pod 被调度的时间和通过网络下载镜像的时间。 |
| `Running`（运行中） | Pod 已经绑定到了某个节点，Pod 中所有的容器都已被创建。至少有一个容器仍在运行，或者正处于启动或重启状态。 |
| `Succeeded`（成功） | Pod 中的所有容器都已成功终止，并且不会再重启。               |
| `Failed`（失败）    | Pod 中的所有容器都已终止，并且至少有一个容器是因为失败终止。也就是说，容器以非 0 状态退出或者被系统终止。 |
| `Unknown`（未知）   | 因为某些原因无法取得 Pod 的状态。这种情况通常是因为与 Pod 所在主机通信失败。 |

**说明：** 当一个 Pod 被删除时，执行一些 kubectl 命令会展示这个 Pod 的状态为 `Terminating`（终止）。 这个 `Terminating` 状态并不是 Pod 阶段之一。 Pod 被赋予一个可以体面终止的期限，默认为 30 秒。 你可以使用 `--force` 参数来[强制终止 Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#pod-termination-forced)。

如果某节点死掉或者与集群中其他节点失联，Kubernetes 会实施一种策略，将失去的节点上运行的所有 Pod 的 `phase` 设置为 `Failed`。

#### 容器状态

Kubernetes 会**跟踪 Pod 中每个容器的状态**，就像它跟踪 Pod 总体上的[阶段](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase)一样。 你可以使用[容器生命周期回调](https://kubernetes.io/zh-cn/docs/concepts/containers/container-lifecycle-hooks/) 来在容器生命周期中的特定时间点触发事件。

一旦[调度器](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/kube-scheduler/)将 Pod 分派给某个节点，`kubelet` 就通过 [容器运行时](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes) 开始为 Pod 创建容器。 容器的状态有三种：`Waiting`（等待）、`Running`（运行中）和 `Terminated`（已终止）。

要检查 Pod 中容器的状态，你可以使用 `kubectl describe pod <pod 名称>`。 其输出中包含 Pod 中每个容器的状态。

每种状态都有特定的含义：

##### `Waiting` （等待）

如果容器并不处在 `Running` 或 `Terminated` 状态之一，它就处在 `Waiting` 状态。 处于 `Waiting` 状态的容器仍在运行它完成启动所需要的操作：例如，从某个容器镜像 仓库拉取容器镜像，或者向容器应用 [Secret](https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/) 数据等等。 当你使用 `kubectl` 来查询包含 `Waiting` 状态的容器的 Pod 时，你也会看到一个 Reason 字段，其中给出了容器处于等待状态的原因。

##### `Running`（运行中）

`Running` 状态表明容器正在执行状态并且没有问题发生。 如果配置了 `postStart` 回调，那么该回调已经执行且已完成。 如果你使用 `kubectl` 来查询包含 `Running` 状态的容器的 Pod 时，你也会看到 关于容器进入 `Running` 状态的信息。

##### `Terminated`（已终止）

处于 `Terminated` 状态的容器已经开始执行并且或者正常结束或者因为某些原因失败。 如果你使用 `kubectl` 来查询包含 `Terminated` 状态的容器的 Pod 时，你会看到 容器进入此状态的原因、退出代码以及容器执行期间的起止时间。

如果容器配置了 `preStop` 回调，则该回调会在容器进入 `Terminated` 状态之前执行。

#### Init 容器

Init 容器是一种特殊容器，在 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 内的应用容器启动之前运行。**Init 容器可以包括一些应用镜像中不存在的实用工具和安装脚本**。

你可以在 Pod 的规约中与用来描述应用容器的 `containers` 数组平行的位置指定 Init 容器。

##### 理解 Init 容器

每个 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 中可以包含多个容器， 应用运行在这些容器里面，同时 Pod 也可以有一个或多个先于应用容器启动的 Init 容器。

Init 容器与普通的容器非常像，除了如下两点：

- 它们总是运行到完成。
- 每个都必须在下一个启动之前成功完成。

**如果 Pod 的 Init 容器失败，kubelet 会不断地重启该 Init 容器直到该容器成功为止**。 然而，如果 Pod 对应的 `restartPolicy` 值为 "Never"，并且 Pod 的 Init 容器失败， 则 Kubernetes 会将整个 Pod 状态设置为失败。

为 Pod 设置 Init 容器需要在 [Pod 规约](https://kubernetes.io/zh-cn/docs/reference/kubernetes-api/workload-resources/pod-v1/#PodSpec) 中添加 `initContainers` 字段， 该字段以 [Container](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.25/#container-v1-core) 类型对象数组的形式组织，和应用的 `containers` 数组同级相邻。

Init 容器的状态在 `status.initContainerStatuses` 字段中以容器状态数组的格式返回 （类似 `status.containerStatuses` 字段）。

##### 与普通容器的不同之处

**Init 容器支持应用容器的全部字段和特性**，包括资源限制、数据卷和安全设置。 然而，Init 容器对资源请求和限制的处理稍有不同，在下面[资源](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/init-containers/#resources)节有说明。

同时 Init 容器不支持 `lifecycle`、`livenessProbe`、`readinessProbe` 和 `startupProbe`， 因为它们必须在 Pod 就绪之前运行完成。

如果为一个 Pod 指定了多个 Init 容器，这些容器会**按顺序逐个运行**。 每个 Init 容器**必须运行成功，下一个才能够运行**。当所有的 Init 容器运行完成时， Kubernetes 才会为 Pod 初始化应用容器并像平常一样运行。

##### 使用 Init 容器

因为 Init 容器具有与应用容器分离的单独镜像，其启动相关代码具有如下优势：

- Init 容器可以包含一些安装过程中应用容器中不存在的实用工具或个性化代码。 例如，没有必要仅为了在安装过程中使用类似 `sed`、`awk`、`python` 或 `dig` 这样的工具而去 `FROM` 一个镜像来生成一个新的镜像。
- Init 容器可以安全地运行这些工具，避免这些工具导致应用镜像的安全性降低。
- 应用镜像的创建者和部署者可以各自独立工作，而没有必要联合构建一个单独的应用镜像。
- Init 容器能以不同于 Pod 内应用容器的文件系统视图运行。因此，Init 容器可以访问 应用容器不能访问的 [Secret](https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/) 的权限。
- 由于 Init 容器必须在应用容器启动之前运行完成，因此 Init 容器提供了一种机制来阻塞或延迟应用容器的启动，直到满足了一组先决条件。 一旦前置条件满足，Pod 内的所有的应用容器会并行启动。

##### 示例

下面是一些如何使用 Init 容器的想法：

- 等待一个 Service 完成创建，通过类似如下 Shell 命令：

  ```shell
  for i in {1..100}; do sleep 1; if dig myservice; then exit 0; fi; done; exit 1
  ```

- 注册这个 Pod 到远程服务器，通过在命令中调用 API，类似如下：

  ```shell
  curl -X POST http://$MANAGEMENT_SERVICE_HOST:$MANAGEMENT_SERVICE_PORT/register -d 'instance=$(<POD_NAME>)&ip=$(<POD_IP>)'
  ```

- 在启动应用容器之前等一段时间，使用类似命令：

  ```shell
  sleep 60
  ```

- 克隆 Git 仓库到[卷](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/)中。
- 将配置值放到配置文件中，运行模板工具为主应用容器动态地生成配置文件。 例如，在配置文件中存放 `POD_IP` 值，并使用 Jinja 生成主应用配置文件

### 工作负载资源

#### Deployments

一个 Deployment 为 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 和 [ReplicaSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/replicaset/) 提供声明式的更新能力。

你负责描述 Deployment 中的 **目标状态**，而 Deployment [控制器（Controller）](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/) 以受控速率更改实际状态， 使其变为期望状态。你可以定义 Deployment 以创建新的 ReplicaSet，或删除现有 Deployment， 并通过新的 Deployment 收养其资源。

> **说明：**
>
> 不要管理 Deployment 所拥有的 ReplicaSet 。 如果存在下面未覆盖的使用场景，请考虑在 Kubernetes 仓库中提出 Issue。

##### 用例

以下是 Deployments 的典型用例：

- [创建 Deployment 以将 ReplicaSet 上线](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/#creating-a-deployment)。ReplicaSet 在后台创建 Pod。 检查 ReplicaSet 的上线状态，查看其是否成功。
- 通过更新 Deployment 的 PodTemplateSpec，[声明 Pod 的新状态](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/#updating-a-deployment) 。 新的 ReplicaSet 会被创建，Deployment 以受控速率将 Pod 从旧 ReplicaSet 迁移到新 ReplicaSet。 每个新的 ReplicaSet 都会更新 Deployment 的修订版本。

- 如果 Deployment 的当前状态不稳定，[回滚到较早的 Deployment 版本](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment)。 每次回滚都会更新 Deployment 的修订版本。
- [扩大 Deployment 规模以承担更多负载](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/#scaling-a-deployment)。
- [暂停 Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/#pausing-and-resuming-a-deployment) 以应用对 PodTemplateSpec 所作的多项修改， 然后恢复其执行以启动新的上线版本。
- [使用 Deployment 状态](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/#deployment-status)来判定上线过程是否出现停滞。
- [清理较旧的不再需要的 ReplicaSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/#clean-up-policy) 。

##### 创建 Deployment

下面是一个 Deployment 示例。其中创建了一个 ReplicaSet，负责启动三个 `nginx` Pod：

[`controllers/nginx-deployment.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/zh-cn/examples/controllers/nginx-deployment.yaml) 

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

在该例中：

- 创建名为 `nginx-deployment`（由 `.metadata.name` 字段标明）的 Deployment。
- 该 Deployment 创建三个（由 `.spec.replicas` 字段标明）Pod 副本。
- `.spec.selector` 字段定义了 Deployment 如何查找要管理的 Pod。

- `selector` 字段**定义 Deployment 如何查找要管理的 Pod**。 在这里，你选择在 Pod 模板中定义的标签（`app: nginx`）。 不过，更复杂的选择规则是也可能的，只要 Pod 模板本身满足所给规则即可。

  > **说明：**
  >
  > `spec.selector.matchLabels` 字段是 `{key,value}` 键值对映射。 在 `matchLabels` 映射中的每个 `{key,value}` 映射等效于 `matchExpressions` 中的一个元素， 即其 `key` 字段是 “key”，`operator` 为 “In”，`values` 数组仅包含 “value”。 在 `matchLabels` 和 `matchExpressions` 中给出的所有条件都必须满足才能匹配。

- `template`字段包含以下子字段：
  - Pod 被使用 `.metadata.labels` 字段打上 `app: nginx` 标签。
  - Pod 模板规约（即 `.template.spec` 字段）指示 Pod 运行一个 `nginx` 容器， 该容器运行版本为 1.14.2 的 `nginx` [Docker Hub](https://hub.docker.com/) 镜像。
  - 创建一个容器并使用 `.spec.template.spec.containers[0].name` 字段将其命名为 `nginx`。

开始之前，请确保的 Kubernetes 集群已启动并运行。 按照以下步骤创建上述 Deployment ：

1. 通过运行以下命令创建 Deployment ：

```shell
kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
```

2. 运行 `kubectl get deployments` 检查 Deployment 是否已创建。 如果仍在创建 Deployment，则输出类似于：

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   0/3     0            0           1s
```

在检查集群中的 Deployment 时，所显示的字段有：

- `NAME` 列出了名字空间中 Deployment 的名称。
- `READY` 显示应用程序的可用的“副本”数。显示的模式是“就绪个数/期望个数”。
- `UP-TO-DATE` 显示为了达到期望状态已经更新的副本数。
- `AVAILABLE` 显示应用可供用户使用的副本数。
- `AGE` 显示应用程序运行的时间。

请注意期望副本数是根据 `.spec.replicas` 字段设置 3。

3. 要查看 Deployment 上线状态，运行 `kubectl rollout status deployment/nginx-deployment`。

输出类似于：

```
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
deployment "nginx-deployment" successfully rolled out
```

4. 几秒钟后再次运行 `kubectl get deployments`。输出类似于：

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           18s
```

注意 Deployment 已创建全部三个副本，并且所有副本都是最新的（它们包含最新的 Pod 模板） 并且可用。

5. 要查看 Deployment 创建的 ReplicaSet（`rs`），运行 `kubectl get rs`。 输出类似于：

```
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-75675f5897   3         3         3       18s
```

ReplicaSet 输出中包含以下字段：

- `NAME` 列出名字空间中 ReplicaSet 的名称；
- `DESIRED` 显示应用的期望副本个数，即在创建 Deployment 时所定义的值。 此为期望状态；
- `CURRENT` 显示当前运行状态中的副本个数；
- `READY` 显示应用中有多少副本可以为用户提供服务；
- `AGE` 显示应用已经运行的时间长度。

注意 ReplicaSet 的名称始终被格式化为`[Deployment名称]-[随机字符串]`。 其中的随机字符串是使用 `pod-template-hash` 作为种子随机生成的。

6. 要查看每个 Pod 自动生成的标签，运行 `kubectl get pods --show-labels`。 输出类似于：

```
NAME                                READY     STATUS    RESTARTS   AGE       LABELS
nginx-deployment-75675f5897-7ci7o   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
nginx-deployment-75675f5897-kzszj   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
nginx-deployment-75675f5897-qqcnn   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
```

所创建的 ReplicaSet 确保总是存在三个 `nginx` Pod。

> **说明：**
>
> 你必须在 Deployment 中指定适当的选择算符和 Pod 模板标签（在本例中为 `app: nginx`）。 标签或者选择算符不要与其他控制器（包括其他 Deployment 和 StatefulSet）重叠。 Kubernetes 不会阻止你这样做，但是如果多个控制器具有重叠的选择算符， 它们可能会发生冲突执行难以预料的操作。

##### 更新 Deployment

> **说明：**
>
> **仅当 Deployment Pod 模板（即 `.spec.template`）发生改变时**，例如模板的标签或容器镜像被更新， 才会触发 Deployment 上线。其他更新（如对 Deployment 执行扩缩容的操作）不会触发上线动作。

##### 回滚 Deployment

有时，你可能想要回滚 Deployment；例如，当 Deployment 不稳定时（例如进入反复崩溃状态）。 默认情况下，Deployment 的所有上线记录都保留在系统中，以便可以随时回滚 （你可以通过修改修订历史记录限制来更改这一约束）。

> **说明：**
>
> **Deployment 被触发上线时，系统就会创建 Deployment 的新的修订版本**。 这意味着仅当 Deployment 的 Pod 模板（`.spec.template`）发生更改时，才会创建新修订版本 -- 例如，模板的标签或容器镜像发生变化。 其他更新，如 Deployment 的扩缩容操作不会创建 Deployment 修订版本。 这是为了方便同时执行手动缩放或自动缩放。 换言之，当你回滚到较早的修订版本时，只有 Deployment 的 Pod 模板部分会被回滚。

##### 缩放 Deployment



##### 暂停、恢复 Deployment 的上线过程

在你更新一个 Deployment 的时候，或者计划更新它的时候， 你可以**在触发一个或多个更新之前暂停 Deployment 的上线过程**。 当你准备应用这些变更时，你可以重新恢复 Deployment 上线过程。 这样做使得你能够在暂停和恢复执行之间应用多个修补程序，而不会触发不必要的上线操作。

##### Deployment 状态

Deployment 的生命周期中会有许多状态。上线新的 ReplicaSet 期间可能处于 [Progressing（进行中）](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/#progressing-deployment)，可能是 [Complete（已完成）](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/#complete-deployment)，也可能是 [Failed（失败）](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/#failed-deployment)以至于无法继续进行。

##### 清理策略

你可以在 Deployment 中设置 `.spec.revisionHistoryLimit` 字段以指定保留此 Deployment 的多少个旧有 ReplicaSet。其余的 ReplicaSet 将在后台被垃圾回收。 默认情况下，此值为 10。

> **说明：**
>
> 显式将此字段设置为 0 将导致 Deployment 的所有历史记录被清空，因此 Deployment 将无法回滚。

##### 金丝雀部署

如果要使用 Deployment 向用户子集或服务器子集上线版本， 则可以遵循[资源管理](https://kubernetes.io/zh-cn/docs/concepts/cluster-administration/manage-deployment/#canary-deployments)所描述的金丝雀模式， 创建多个 Deployment，每个版本一个。

##### 编写 Deployment 规约

同其他 Kubernetes 配置一样， Deployment 需要 `.apiVersion`，`.kind` 和 `.metadata` 字段。 有关配置文件的其他信息，请参考[部署 Deployment](https://kubernetes.io/zh-cn/docs/tasks/run-application/run-stateless-application-deployment/)、 配置容器和[使用 kubectl 管理资源](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/object-management/)等相关文档。

Deployment 对象的名称必须是合法的 [DNS 子域名](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/names#dns-subdomain-names)。 Deployment 还需要 [`.spec` 部分](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status)

###### Pod 模板

`.spec` 中只有 `.spec.template` 和 `.spec.selector` 是必需的字段。

`.spec.template` 是一个 [Pod 模板](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/#pod-templates)。 它和 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 的语法规则完全相同。 只是这里它是嵌套的，因此不需要 `apiVersion` 或 `kind`。

除了 Pod 的必填字段外，Deployment 中的 Pod 模板必须指定适当的标签和适当的重新启动策略。 对于标签，请确保不要与其他控制器重叠。请参考[选择算符](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/#selector)。

只有 [`.spec.template.spec.restartPolicy`](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) 等于 `Always` 才是被允许的，这也是在没有指定时的默认设置。

###### 副本

`.spec.replicas` 是指定所需 Pod 的可选字段。它的默认值是1。

如果你对某个 Deployment 执行了手动扩缩操作（例如，通过 `kubectl scale deployment deployment --replicas=X`）， 之后基于清单对 Deployment 执行了更新操作（例如通过运行 `kubectl apply -f deployment.yaml`），那么通过应用清单而完成的更新会覆盖之前手动扩缩所作的变更。

如果一个 [HorizontalPodAutoscaler](https://kubernetes.io/zh-cn/docs/tasks/run-application/horizontal-pod-autoscale/) （或者其他执行水平扩缩操作的类似 API）在管理 Deployment 的扩缩， 则不要设置 `.spec.replicas`。

恰恰相反，应该允许 Kubernetes [控制面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane)来自动管理 `.spec.replicas` 字段。

###### 选择算符

`.spec.selector` 是指定本 Deployment 的 Pod [标签选择算符](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/)的必需字段。

`.spec.selector` 必须匹配 `.spec.template.metadata.labels`，否则请求会被 API 拒绝。

在 API `apps/v1`版本中，`.spec.selector` 和 `.metadata.labels` 如果没有设置的话， 不会被默认设置为 `.spec.template.metadata.labels`，所以需要明确进行设置。 同时在 `apps/v1`版本中，Deployment 创建后 `.spec.selector` 是不可变的。

当 Pod 的标签和选择算符匹配，但其模板和 `.spec.template` 不同时，或者此类 Pod 的总数超过 `.spec.replicas` 的设置时，Deployment 会终结之。 如果 Pod 总数未达到期望值，Deployment 会基于 `.spec.template` 创建新的 Pod。

**说明：**

你不应直接创建与此选择算符匹配的 Pod，也不应通过创建另一个 Deployment 或者类似于 ReplicaSet 或 ReplicationController 这类控制器来创建标签与此选择算符匹配的 Pod。 如果这样做，第一个 Deployment 会认为它创建了这些 Pod。 Kubernetes 不会阻止你这么做。

如果有多个控制器的选择算符发生重叠，则控制器之间会因冲突而无法正常工作。

###### 策略

`.spec.strategy` 策略指定用于用新 Pod 替换旧 Pod 的策略。 `.spec.strategy.type` 可以是 “Recreate” 或 “RollingUpdate”。“RollingUpdate” 是默认值。

**重新创建 Deployment**

如果 `.spec.strategy.type==Recreate`，在创建新 Pod 之前，所有现有的 Pod 会被杀死。

**说明：**

这只会确保为了升级而创建新 Pod 之前其他 Pod 都已终止。如果你升级一个 Deployment， 所有旧版本的 Pod 都会立即被终止。控制器等待这些 Pod 被成功移除之后， 才会创建新版本的 Pod。如果你手动删除一个 Pod，其生命周期是由 ReplicaSet 来控制的， 后者会立即创建一个替换 Pod（即使旧的 Pod 仍然处于 Terminating 状态）。 如果你需要一种“最多 n 个”的 Pod 个数保证，你需要考虑使用 [StatefulSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/)。

**滚动更新 Deployment**

Deployment 会在 `.spec.strategy.type==RollingUpdate`时，采取 滚动更新的方式更新 Pod。你可以指定 `maxUnavailable` 和 `maxSurge` 来控制滚动更新 过程。

**最大不可用**

`.spec.strategy.rollingUpdate.maxUnavailable` 是一个可选字段，用来指定 更新过程中不可用的 Pod 的个数上限。该值可以是绝对数字（例如，5），也可以是所需 Pod 的百分比（例如，10%）。百分比值会转换成绝对数并去除小数部分。 如果 `.spec.strategy.rollingUpdate.maxSurge` 为 0，则此值不能为 0。 默认值为 25%。

例如，当此值设置为 30% 时，滚动更新开始时会立即将旧 ReplicaSet 缩容到期望 Pod 个数的70%。 新 Pod 准备就绪后，可以继续缩容旧有的 ReplicaSet，然后对新的 ReplicaSet 扩容， 确保在更新期间可用的 Pod 总数在任何时候都至少为所需的 Pod 个数的 70%。

**最大峰值**

`.spec.strategy.rollingUpdate.maxSurge` 是一个可选字段，用来指定可以创建的超出期望 Pod 个数的 Pod 数量。此值可以是绝对数（例如，5）或所需 Pod 的百分比（例如，10%）。 如果 `MaxUnavailable` 为 0，则此值不能为 0。百分比值会通过向上取整转换为绝对数。 此字段的默认值为 25%。

例如，当此值为 30% 时，启动滚动更新后，会立即对新的 ReplicaSet 扩容，同时保证新旧 Pod 的总数不超过所需 Pod 总数的 130%。一旦旧 Pod 被杀死，新的 ReplicaSet 可以进一步扩容， 同时确保更新期间的任何时候运行中的 Pod 总数最多为所需 Pod 总数的 130%。

###### 进度期限秒数

`.spec.progressDeadlineSeconds` 是一个可选字段，用于指定系统在报告 Deployment [进展失败](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/#failed-deployment) 之前等待 Deployment 取得进展的秒数。 这类报告会在资源状态中体现为 `type: Progressing`、`status: False`、 `reason: ProgressDeadlineExceeded`。Deployment 控制器将在默认 600 毫秒内持续重试 Deployment。 将来，一旦实现了自动回滚，Deployment 控制器将在探测到这样的条件时立即回滚 Deployment。

如果指定，则此字段值需要大于 `.spec.minReadySeconds` 取值。

###### 最短就绪时间

`.spec.minReadySeconds` 是一个可选字段，用于指定新创建的 Pod 在没有任意容器崩溃情况下的最小就绪时间， 只有超出这个时间 Pod 才被视为可用。默认值为 0（Pod 在准备就绪后立即将被视为可用）。 要了解何时 Pod 被视为就绪， 可参考[容器探针](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)。

###### 修订历史限制

Deployment 的修订历史记录存储在它所控制的 ReplicaSets 中。

`.spec.revisionHistoryLimit` 是一个可选字段，用来设定出于会滚目的所要保留的旧 ReplicaSet 数量。 这些旧 ReplicaSet 会消耗 etcd 中的资源，并占用 `kubectl get rs` 的输出。 每个 Deployment 修订版本的配置都存储在其 ReplicaSets 中；因此，一旦删除了旧的 ReplicaSet， 将失去回滚到 Deployment 的对应修订版本的能力。 默认情况下，系统保留 10 个旧 ReplicaSet，但其理想值取决于新 Deployment 的频率和稳定性。

更具体地说，将此字段设置为 0 意味着将清理所有具有 0 个副本的旧 ReplicaSet。 在这种情况下，无法撤消新的 Deployment 上线，因为它的修订历史被清除了。

###### paused（暂停的）

`.spec.paused` 是用于暂停和恢复 Deployment 的可选布尔字段。 暂停的 Deployment 和未暂停的 Deployment 的唯一区别是，Deployment 处于暂停状态时， PodTemplateSpec 的任何修改都不会触发新的上线。 Deployment 在创建时是默认不会处于暂停状态。

#### ReplicaSet

ReplicaSet 的目的是**维护一组在任何时候都处于运行状态的 Pod 副本的稳定集合**。 因此，它通常用来保证给定数量的、完全相同的 Pod 的可用性。

##### ReplicaSet 的工作原理

ReplicaSet 是通过一组字段来定义的，包括一个用来识别可获得的 Pod 的集合的选择算符、一个用来标明应该维护的副本个数的数值、一个用来指定应该创建新 Pod 以满足副本个数条件时要使用的 Pod 模板等等。 每个 ReplicaSet 都通过根据需要创建和删除 Pod 以使得副本个数达到期望值， 进而实现其存在价值。当 ReplicaSet 需要创建新的 Pod 时，会使用所提供的 Pod 模板。

ReplicaSet 通过 Pod 上的 [metadata.ownerReferences](https://kubernetes.io/zh-cn/docs/concepts/architecture/garbage-collection/#owners-and-dependents) 字段连接到附属 Pod，**该字段给出当前对象的属主资源**。 ReplicaSet 所获得的 Pod 都在其 ownerReferences 字段中**包含了属主 ReplicaSet 的标识信息**。正是通过这一连接，ReplicaSet 知道它所维护的 Pod 集合的状态， 并据此计划其操作行为。

ReplicaSet 使用其选择算符来辨识要获得的 Pod 集合。如果某个 Pod 没有 OwnerReference 或者其 OwnerReference 不是一个[控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/)， 且其匹配到某 ReplicaSet 的选择算符，则该 Pod 立即被此 ReplicaSet 获得。

##### 何时使用 ReplicaSet

ReplicaSet 确保任何时间都有指定数量的 Pod 副本在运行。 然而，Deployment 是一个更高级的概念，它管理 ReplicaSet，并向 Pod 提供声明式的更新以及许多其他有用的功能。 因此，我们建议使用 Deployment 而不是直接使用 ReplicaSet， 除非你需要自定义更新业务流程或根本不需要更新。

这实际上意味着，你可能永远不需要操作 ReplicaSet 对象：而是使用 Deployment，并在 spec 部分定义你的应用。

##### 示例

[`controllers/frontend.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/zh-cn/examples/controllers/frontend.yaml) 

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: frontend
  labels:
    app: guestbook
    tier: frontend
spec:
  # 按你的实际情况修改副本数
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: gcr.io/google_samples/gb-frontend:v3
```

将此清单保存到 `frontend.yaml` 中，并将其提交到 Kubernetes 集群， 就能创建 yaml 文件所定义的 ReplicaSet 及其管理的 Pod。

```shell
kubectl apply -f https://kubernetes.io/examples/controllers/frontend.yaml
```

你可以看到当前被部署的 ReplicaSet：

```shell
kubectl get rs
```

并看到你所创建的前端：

```
NAME       DESIRED   CURRENT   READY   AGE
frontend   3         3         3       6s
```

你也可以查看 ReplicaSet 的状态：

```shell
kubectl describe rs/frontend
```

你会看到类似如下的输出：

```
Name:         frontend
Namespace:    default
Selector:     tier=frontend
Labels:       app=guestbook
              tier=frontend
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"apps/v1","kind":"ReplicaSet","metadata":{"annotations":{},"labels":{"app":"guestbook","tier":"frontend"},"name":"frontend",...
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  tier=frontend
  Containers:
   php-redis:
    Image:        gcr.io/google_samples/gb-frontend:v3
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  117s  replicaset-controller  Created pod: frontend-wtsmm
  Normal  SuccessfulCreate  116s  replicaset-controller  Created pod: frontend-b2zdv
  Normal  SuccessfulCreate  116s  replicaset-controller  Created pod: frontend-vcmts
```

最后可以查看启动了的 Pod 集合：

```shell
kubectl get pods
```

你会看到类似如下的 Pod 信息：

```
NAME             READY   STATUS    RESTARTS   AGE
frontend-b2zdv   1/1     Running   0          6m36s
frontend-vcmts   1/1     Running   0          6m36s
frontend-wtsmm   1/1     Running   0          6m36s
```

你也可以查看 Pod 的属主引用被设置为前端的 ReplicaSet。 要实现这点，可取回运行中的某个 Pod 的 YAML：

```shell
kubectl get pods frontend-b2zdv -o yaml
```

输出将类似这样，frontend ReplicaSet 的信息被设置在 metadata 的 `ownerReferences` 字段中：

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2020-02-12T07:06:16Z"
  generateName: frontend-
  labels:
    tier: frontend
  name: frontend-b2zdv
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: frontend
    uid: f391f6db-bb9b-4c09-ae74-6a1f77f3d5cf
...
```

##### 非模板 Pod 的获得

尽管你完全可以直接创建裸的 Pod，强烈建议你确保这些裸的 Pod 并不包含可能与你的某个 ReplicaSet 的选择算符相匹配的标签。原因在于 ReplicaSet 并不仅限于拥有在其模板中设置的 Pod，它还可以像前面小节中所描述的那样获得其他 Pod。

[`pods/pod-rs.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/zh-cn/examples/pods/pod-rs.yaml) 

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
  labels:
    tier: frontend
spec:
  containers:
  - name: hello1
    image: gcr.io/google-samples/hello-app:2.0

---

apiVersion: v1
kind: Pod
metadata:
  name: pod2
  labels:
    tier: frontend
spec:
  containers:
  - name: hello2
    image: gcr.io/google-samples/hello-app:1.0
```

由于这些 Pod 没有控制器（Controller，或其他对象）作为其属主引用， **并且其标签与 frontend ReplicaSet 的选择算符匹配**，它们会立即被该 ReplicaSet 获取。

假定你在 frontend ReplicaSet 已经被部署之后创建 Pod，并且你已经在 ReplicaSet 中设置了其初始的 Pod 副本数以满足其副本计数需要：

```shell
kubectl apply -f https://kubernetes.io/examples/pods/pod-rs.yaml
```

新的 Pod 会被该 ReplicaSet 获取，并立即被 ReplicaSet 终止， 因为它们的存在会使得 ReplicaSet 中 Pod 个数超出其期望值。

取回 Pod：

```shell
kubectl get pods
```

输出显示新的 Pod 或者已经被终止，或者处于终止过程中：

```
NAME             READY   STATUS        RESTARTS   AGE
frontend-b2zdv   1/1     Running       0          10m
frontend-vcmts   1/1     Running       0          10m
frontend-wtsmm   1/1     Running       0          10m
pod1             0/1     Terminating   0          1s
pod2             0/1     Terminating   0          1s
```

如果你先行创建 Pod：

```shell
kubectl apply -f https://kubernetes.io/examples/pods/pod-rs.yaml
```

之后再创建 ReplicaSet：

```shell
kubectl apply -f https://kubernetes.io/examples/controllers/frontend.yaml
```

你会看到 ReplicaSet 已经获得了该 Pod，并仅根据其规约创建新的 Pod， 直到新的 Pod 和原来的 Pod 的总数达到其预期个数。 这时取回 Pod 列表：

```shell
kubectl get pods
```

将会生成下面的输出：

```
NAME             READY   STATUS    RESTARTS   AGE
frontend-hmmj2   1/1     Running   0          9s
pod1             1/1     Running   0          36s
pod2             1/1     Running   0          36s
```

采用这种方式，一个 ReplicaSet 中可以包含异质的 Pod 集合。

##### 编写 ReplicaSet 的清单

与所有其他 Kubernetes API 对象一样，ReplicaSet 也需要 `apiVersion`、`kind`、和 `metadata` 字段。 对于 ReplicaSet 而言，其 `kind` 始终是 ReplicaSet。

ReplicaSet 对象的名称必须是合法的 [DNS 子域名](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/names#dns-subdomain-names)。

ReplicaSet 也需要 [`.spec`](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status) 部分。

###### Pod 模板

`.spec.template` 是一个 [Pod 模板](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/#pod-templates)， 要求设置标签。在 `frontend.yaml` 示例中，我们指定了标签 `tier: frontend`。 注意不要将标签与其他控制器的选择算符重叠，否则那些控制器会尝试收养此 Pod。

对于模板的[重启策略](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) 字段，`.spec.template.spec.restartPolicy`，唯一允许的取值是 `Always`，这也是默认值.

###### Pod 选择算符

`.spec.selector` 字段是一个[标签选择算符](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/)。 如前文中[所讨论的](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/replicaset/#how-a-replicaset-works)，这些是用来标识要被获取的 Pod 的标签。在签名的 `frontend.yaml` 示例中，选择算符为：

```yaml
matchLabels:
  tier: frontend
```

在 ReplicaSet 中，`.spec.template.metadata.labels` 的值必须与 `spec.selector` 值相匹配，否则该配置会被 API 拒绝。

**说明：**

对于设置了相同的 `.spec.selector`，但 `.spec.template.metadata.labels` 和 `.spec.template.spec` 字段不同的两个 ReplicaSet 而言，每个 ReplicaSet 都会忽略被另一个 ReplicaSet 所创建的 Pod。

###### Replicas

你可以通过设置 `.spec.replicas` 来指定要同时运行的 Pod 个数。 ReplicaSet 创建、删除 Pod 以与此值匹配。

如果你没有指定 `.spec.replicas`，那么默认值为 1。

##### 使用 ReplicaSet

###### 删除 ReplicaSet 和它的 Pod

要删除 ReplicaSet 和它的所有 Pod，使用 [`kubectl delete`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete) 命令。 默认情况下，[垃圾收集器](https://kubernetes.io/zh-cn/docs/concepts/architecture/garbage-collection/) 自动删除所有依赖的 Pod。

当使用 REST API 或 `client-go` 库时，你必须在 `-d` 选项中将 `propagationPolicy` 设置为 `Background` 或 `Foreground`。例如：

```shell
kubectl proxy --port=8080
curl -X DELETE  'localhost:8080/apis/apps/v1/namespaces/default/replicasets/frontend' \
  -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Foreground"}' \
  -H "Content-Type: application/json"
```

###### 只删除 ReplicaSet

你可以只删除 ReplicaSet 而不影响它的各个 Pod，方法是使用 [`kubectl delete`](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete) 命令并设置 `--cascade=orphan` 选项。

当使用 REST API 或 `client-go` 库时，你必须将 `propagationPolicy` 设置为 `Orphan`。 例如：

```shell
kubectl proxy --port=8080
curl -X DELETE  'localhost:8080/apis/apps/v1/namespaces/default/replicasets/frontend' \
  -d '{"kind":"DeleteOptions","apiVersion":"v1","propagationPolicy":"Orphan"}' \
  -H "Content-Type: application/json"
```

一旦删除了原来的 ReplicaSet，就可以创建一个新的来替换它。 由于新旧 ReplicaSet 的 `.spec.selector` 是相同的，新的 ReplicaSet 将接管老的 Pod。 但是，它不会努力使现有的 Pod 与新的、不同的 Pod 模板匹配。 若想要以可控的方式更新 Pod 的规约，可以使用 [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/#creating-a-deployment) 资源，因为 ReplicaSet 并不直接支持滚动更新。

###### 将 Pod 从 ReplicaSet 中隔离

可以通过改变标签来从 ReplicaSet 中移除 Pod。 这种技术可以用来从服务中去除 Pod，以便进行排错、数据恢复等。 以这种方式移除的 Pod 将被自动替换（假设副本的数量没有改变）。

###### 扩缩 RepliaSet

通过更新 `.spec.replicas` 字段，ReplicaSet 可以被轻松地进行扩缩。ReplicaSet 控制器能确保匹配标签选择器的数量的 Pod 是可用的和可操作的。

在降低集合规模时，ReplicaSet 控制器通过对可用的所有 Pod 进行排序来优先选择要被删除的那些 Pod。 其一般性算法如下：

1. 首先选择剔除悬决（Pending，且不可调度）的各个 Pod
2. 如果设置了 `controller.kubernetes.io/pod-deletion-cost` 注解，则注解值较小的优先被裁减掉
3. 所处节点上副本个数较多的 Pod 优先于所处节点上副本较少者
4. 如果 Pod 的创建时间不同，最近创建的 Pod 优先于早前创建的 Pod 被裁减。 （当 `LogarithmicScaleDown` 这一[特性门控](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/feature-gates/) 被启用时，创建时间是按整数幂级来分组的）。

如果以上比较结果都相同，则随机选择。

###### Pod 删除开销

**特性状态：** `Kubernetes v1.22 [beta]`

通过使用 [`controller.kubernetes.io/pod-deletion-cost`](https://kubernetes.io/zh-cn/docs/reference/labels-annotations-taints/#pod-deletion-cost) 注解，用户可以对 ReplicaSet 缩容时要先删除哪些 Pod 设置偏好。

此注解要设置到 Pod 上，取值范围为 [-2147483647, 2147483647]。 所代表的是删除同一 ReplicaSet 中其他 Pod 相比较而言的开销。 删除开销较小的 Pod 比删除开销较高的 Pod 更容易被删除。

Pod 如果未设置此注解，则隐含的设置值为 0。负值也是可接受的。 如果注解值非法，API 服务器会拒绝对应的 Pod。

此功能特性处于 Beta 阶段，默认被启用。你可以通过为 kube-apiserver 和 kube-controller-manager 设置[特性门控](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/feature-gates/) `PodDeletionCost` 来禁用此功能。

**说明：**

- 此机制实施时仅是尽力而为，并不能对 Pod 的删除顺序作出任何保证；
- 用户应避免频繁更新注解值，例如根据某观测度量值来更新此注解值是应该避免的。 这样做会在 API 服务器上产生大量的 Pod 更新操作。

###### 使用场景示例

同一应用的不同 Pod 可能其利用率是不同的。在对应用执行缩容操作时， 可能希望移除利用率较低的 Pod。为了避免频繁更新 Pod，应用应该在执行缩容操作之前更新一次 `controller.kubernetes.io/pod-deletion-cost` 注解值 （将注解值设置为一个与其 Pod 利用率对应的值）。 如果应用自身控制器缩容操作时（例如 Spark 部署的驱动 Pod），这种机制是可以起作用的。

###### ReplicaSet 作为水平的 Pod 自动扩缩器目标

ReplicaSet 也可以作为[水平的 Pod 扩缩器 (HPA)](https://kubernetes.io/zh-cn/docs/tasks/run-application/horizontal-pod-autoscale/) 的目标。也就是说，ReplicaSet 可以被 HPA 自动扩缩。 以下是 HPA 以我们在前一个示例中创建的副本集为目标的示例。

[`controllers/hpa-rs.yaml`](https://raw.githubusercontent.com/kubernetes/website/main/content/zh-cn/examples/controllers/hpa-rs.yaml) 

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: frontend-scaler
spec:
  scaleTargetRef:
    kind: ReplicaSet
    name: frontend
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 50
```

将这个列表保存到 `hpa-rs.yaml` 并提交到 Kubernetes 集群，就能创建它所定义的 HPA，进而就能根据复制的 Pod 的 CPU 利用率对目标 ReplicaSet 进行自动扩缩。

```shell
kubectl apply -f https://k8s.io/examples/controllers/hpa-rs.yaml
```

或者，可以使用 `kubectl autoscale` 命令完成相同的操作（而且它更简单！）

```shell
kubectl autoscale rs frontend --max=10 --min=3 --cpu-percent=50
```

##### ReplicaSet 的替代方案

###### Deployment（推荐）

[`Deployment`](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/) 是一个可以拥有 ReplicaSet 并使用声明式方式在服务器端完成对 Pod 滚动更新的对象。 尽管 ReplicaSet 可以独立使用，目前它们的主要用途是提供给 Deployment 作为编排 Pod 创建、删除和更新的一种机制。当使用 Deployment 时，你不必关心如何管理它所创建的 ReplicaSet，Deployment 拥有并管理其 ReplicaSet。 因此，建议你在需要 ReplicaSet 时使用 Deployment。

###### 裸 Pod

与用户直接创建 Pod 的情况不同，ReplicaSet 会替换那些由于某些原因被删除或被终止的 Pod，例如在节点故障或破坏性的节点维护（如内核升级）的情况下。 因为这个原因，我们建议你使用 ReplicaSet，即使应用程序只需要一个 Pod。 想像一下，ReplicaSet 类似于进程监视器，只不过它在多个节点上监视多个 Pod， 而不是在单个节点上监视单个进程。 ReplicaSet 将本地容器重启的任务委托给了节点上的某个代理（例如，Kubelet）去完成。

###### Job

使用[`Job`](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/job/) 代替 ReplicaSet， 可以用于那些期望自行终止的 Pod。

###### DaemonSet

对于管理那些提供主机级别功能（如主机监控和主机日志）的容器， 就要用 [`DaemonSet`](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/daemonset/) 而不用 ReplicaSet。 这些 Pod 的寿命与主机寿命有关：这些 Pod 需要先于主机上的其他 Pod 运行， 并且在机器准备重新启动/关闭时安全地终止。

###### ReplicationController

ReplicaSet 是 [ReplicationController](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/replicationcontroller/) 的后继者。二者目的相同且行为类似，只是 ReplicationController 不支持 [标签用户指南](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/labels/#label-selectors) 中讨论的基于集合的选择算符需求。 因此，相比于 ReplicationController，应优先考虑 ReplicaSet。

#### StatefulSet

StatefulSet 是**用来管理有状态应用的工作负载 API 对象**。

StatefulSet 用来管理某 [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) 集合的部署和扩缩， 并为这些 Pod 提供持久存储和持久标识符。

和 [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/) 类似， StatefulSet 管理基于相同容器规约的一组 Pod。但和 Deployment 不同的是， **StatefulSet 为它们的每个 Pod 维护了一个有粘性的 ID**。这些 Pod 是基于相同的规约来创建的， 但是不能相互替换：无论怎么调度，每个 Pod 都有一个永久不变的 ID。

如果希望使用存储卷为工作负载提供持久存储，可以使用 StatefulSet 作为解决方案的一部分。 尽管 StatefulSet 中的单个 Pod 仍可能出现故障， 但持久的 Pod 标识符使得将现有卷与替换已失败 Pod 的新 Pod 相匹配变得更加容易。

##### 使用 StatefulSet

StatefulSet 对于需要满足以下一个或多个需求的应用程序很有价值：

- 稳定的、唯一的网络标识符。
- 稳定的、持久的存储。
- 有序的、优雅的部署和扩缩。
- 有序的、自动的滚动更新。

在上面描述中，“稳定的”意味着 Pod 调度或重调度的整个过程是有持久性的。 如果应用程序不需要任何稳定的标识符或有序的部署、删除或扩缩， 则应该使用由一组无状态的副本控制器提供的工作负载来部署应用程序，比如 [Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/) 或者 [ReplicaSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/replicaset/) 可能更适用于你的无状态应用部署需要。

##### 限制

- 给定 Pod 的存储必须由 [PersistentVolume Provisioner](https://github.com/kubernetes/examples/tree/master/staging/persistent-volume-provisioning/README.md) 基于所请求的 `storage class` 来制备，或者由管理员预先制备。
- 删除或者扩缩 StatefulSet 并**不会**删除它关联的存储卷。 这样做是为了保证数据安全，它通常比自动清除 StatefulSet 所有相关的资源更有价值。
- StatefulSet 当前需要[无头服务](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#headless-services)来负责 Pod 的网络标识。你需要负责创建此服务。
- 当删除一个 StatefulSet 时，该 StatefulSet 不提供任何终止 Pod 的保证。 为了实现 StatefulSet 中的 Pod 可以有序且体面地终止，可以在删除之前将 StatefulSet 缩容到 0。
- 在默认 [Pod 管理策略](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/#pod-management-policies)(`OrderedReady`) 时使用[滚动更新](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/#rolling-updates)， 可能进入需要[人工干预](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/#forced-rollback)才能修复的损坏状态。

##### 组件

下面的示例演示了 StatefulSet 的组件。

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # 必须匹配 .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # 默认值是 1
  minReadySeconds: 10 # 默认值是 0
  template:
    metadata:
      labels:
        app: nginx # 必须匹配 .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: registry.k8s.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```

上述例子中：

- 名为 `nginx` 的 Headless Service 用来控制网络域名。
- 名为 `web` 的 StatefulSet 有一个 Spec，它表明将在独立的 3 个 Pod 副本中启动 nginx 容器。
- `volumeClaimTemplates` 将通过 PersistentVolume 制备程序所准备的 [PersistentVolumes](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/) 来提供稳定的存储。

StatefulSet 的命名需要遵循 [DNS 子域名](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/names#dns-subdomain-names)规范。

###### Pod 选择算符

你必须设置 StatefulSet 的 `.spec.selector` 字段，使之匹配其在 `.spec.template.metadata.labels` 中设置的标签。 未指定匹配的 Pod 选择算符将在创建 StatefulSet 期间导致验证错误。

###### 卷申领模板

你可以设置 `.spec.volumeClaimTemplates`， 它可以使用 PersistentVolume 制备程序所准备的 [PersistentVolumes](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/) 来提供稳定的存储。

###### 最短就绪秒数

**特性状态：** `Kubernetes v1.25 [stable]`

`.spec.minReadySeconds` 是一个可选字段。 它指定新创建的 Pod 应该在没有任何容器崩溃的情况下运行并准备就绪，才能被认为是可用的。 这用于在使用[滚动更新](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/#rolling-updates)策略时检查滚动的进度。 该字段默认为 0（Pod 准备就绪后将被视为可用）。 要了解有关何时认为 Pod 准备就绪的更多信息， 请参阅[容器探针](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)。

##### Pod 标识

StatefulSet Pod 具有唯一的标识，该标识包括顺序标识、稳定的网络标识和稳定的存储。 该标识和 Pod 是绑定的，与该 Pod 调度到哪个节点上无关。

##### 有序索引

对于具有 N 个副本的 StatefulSet，该 StatefulSet 中的每个 Pod 将被分配一个从 0 到 N-1 的整数序号，该序号在此 StatefulSet 上是唯一的。

##### 稳定的网络 ID

StatefulSet 中的每个 Pod 根据 StatefulSet 的名称和 Pod 的序号派生出它的主机名。 组合主机名的格式为`$(StatefulSet 名称)-$(序号)`。 上例将会创建三个名称分别为 `web-0、web-1、web-2` 的 Pod。 StatefulSet 可以使用[无头服务](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#headless-services)控制它的 Pod 的网络域。管理域的这个服务的格式为： `$(服务名称).$(名字空间).svc.cluster.local`，其中 `cluster.local` 是集群域。 一旦每个 Pod 创建成功，就会得到一个匹配的 DNS 子域，格式为： `$(pod 名称).$(所属服务的 DNS 域名)`，其中所属服务由 StatefulSet 的 `serviceName` 域来设定。

取决于集群域内部 DNS 的配置，有可能无法查询一个刚刚启动的 Pod 的 DNS 命名。 当集群内其他客户端在 Pod 创建完成前发出 Pod 主机名查询时，就会发生这种情况。 负缓存 (在 DNS 中较为常见) 意味着之前失败的查询结果会被记录和重用至少若干秒钟， 即使 Pod 已经正常运行了也是如此。

如果需要在 Pod 被创建之后及时发现它们，可使用以下选项：

- 直接查询 Kubernetes API（比如，利用 watch 机制）而不是依赖于 DNS 查询
- 缩短 Kubernetes DNS 驱动的缓存时长（通常这意味着修改 CoreDNS 的 ConfigMap，目前缓存时长为 30 秒）

正如[限制](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/#limitations)中所述， 你需要负责创建[无头服务](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/#headless-services)以便为 Pod 提供网络标识。

下面给出一些选择集群域、服务名、StatefulSet 名、及其怎样影响 StatefulSet 的 Pod 上的 DNS 名称的示例：

| 集群域名      | 服务（名字空间/名字） | StatefulSet（名字空间/名字） | StatefulSet 域名                | Pod DNS                                      | Pod 主机名   |
| ------------- | --------------------- | ---------------------------- | ------------------------------- | -------------------------------------------- | ------------ |
| cluster.local | default/nginx         | default/web                  | nginx.default.svc.cluster.local | web-{0..N-1}.nginx.default.svc.cluster.local | web-{0..N-1} |
| cluster.local | foo/nginx             | foo/web                      | nginx.foo.svc.cluster.local     | web-{0..N-1}.nginx.foo.svc.cluster.local     | web-{0..N-1} |
| kube.local    | foo/nginx             | foo/web                      | nginx.foo.svc.kube.local        | web-{0..N-1}.nginx.foo.svc.kube.local        | web-{0..N-1} |

**说明：** 集群域会被设置为 `cluster.local`，除非有[其他配置](https://kubernetes.io/zh-cn/docs/concepts/services-networking/dns-pod-service/)。

##### 稳定的存储

对于 StatefulSet 中定义的每个 VolumeClaimTemplate，每个 Pod 接收到一个 PersistentVolumeClaim。 在上面的 nginx 示例中，每个 Pod 将会得到基于 StorageClass `my-storage-class` 制备的 1 Gib 的 PersistentVolume。 如果没有声明 StorageClass，就会使用默认的 StorageClass。 当一个 Pod 被调度（重新调度）到节点上时，它的 `volumeMounts` 会挂载与其 PersistentVolumeClaims 相关联的 PersistentVolume。 请注意，当 Pod 或者 StatefulSet 被删除时，与 PersistentVolumeClaims 相关联的 PersistentVolume 并不会被删除。要删除它必须通过手动方式来完成。

##### Pod 名称标签

当 StatefulSet [控制器（Controller）](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/) 创建 Pod 时， 它会添加一个标签 `statefulset.kubernetes.io/pod-name`，该标签值设置为 Pod 名称。 这个标签允许你给 StatefulSet 中的特定 Pod 绑定一个 Service。

##### 部署和扩缩保证

- 对于包含 N 个 副本的 StatefulSet，当部署 Pod 时，它们是依次创建的，顺序为 `0..N-1`。
- 当删除 Pod 时，它们是逆序终止的，顺序为 `N-1..0`。
- 在将扩缩操作应用到 Pod 之前，它前面的所有 Pod 必须是 Running 和 Ready 状态。
- 在一个 Pod 终止之前，所有的继任者必须完全关闭。

StatefulSet 不应将 `pod.Spec.TerminationGracePeriodSeconds` 设置为 0。 这种做法是不安全的，要强烈阻止。 更多的解释请参考[强制删除 StatefulSet Pod](https://kubernetes.io/zh-cn/docs/tasks/run-application/force-delete-stateful-set-pod/)。

在上面的 nginx 示例被创建后，会按照 web-0、web-1、web-2 的顺序部署三个 Pod。 在 web-0 进入 [Running 和 Ready](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/pod-lifecycle/) 状态前不会部署 web-1。在 web-1 进入 Running 和 Ready 状态前不会部署 web-2。 如果 web-1 已经处于 Running 和 Ready 状态，而 web-2 尚未部署，在此期间发生了 web-0 运行失败，那么 web-2 将不会被部署，要等到 web-0 部署完成并进入 Running 和 Ready 状态后，才会部署 web-2。

如果用户想将示例中的 StatefulSet 扩缩为 `replicas=1`，首先被终止的是 web-2。 在 web-2 没有被完全停止和删除前，web-1 不会被终止。 当 web-2 已被终止和删除、web-1 尚未被终止，如果在此期间发生 web-0 运行失败， 那么就不会终止 web-1，必须等到 web-0 进入 Running 和 Ready 状态后才会终止 web-1。

##### Pod 管理策略

StatefulSet 允许你放宽其排序保证， 同时通过它的 `.spec.podManagementPolicy` 域保持其唯一性和身份保证。

##### OrderedReady Pod 管理

`OrderedReady` Pod 管理是 StatefulSet 的默认设置。 它实现了[上面](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/#deployment-and-scaling-guarantees)描述的功能。

##### 并行 Pod 管理

`Parallel` Pod 管理让 StatefulSet 控制器并行的启动或终止所有的 Pod， 启动或者终止其他 Pod 前，无需等待 Pod 进入 Running 和 ready 或者完全停止状态。 这个选项只会影响扩缩操作的行为，更新则不会被影响。

##### 更新策略

StatefulSet 的 `.spec.updateStrategy` 字段让你可以配置和禁用掉自动滚动更新 Pod 的容器、标签、资源请求或限制、以及注解。有两个允许的值：

- `OnDelete`

  当 StatefulSet 的 `.spec.updateStrategy.type` 设置为 `OnDelete` 时， 它的控制器将不会自动更新 StatefulSet 中的 Pod。 用户必须手动删除 Pod 以便让控制器创建新的 Pod，以此来对 StatefulSet 的 `.spec.template` 的变动作出反应。

- `RollingUpdate`

  `RollingUpdate` 更新策略对 StatefulSet 中的 Pod 执行自动的滚动更新。这是默认的更新策略。

##### 滚动更新

当 StatefulSet 的 `.spec.updateStrategy.type` 被设置为 `RollingUpdate` 时， StatefulSet 控制器会删除和重建 StatefulSet 中的每个 Pod。 它将按照与 Pod 终止相同的顺序（从最大序号到最小序号）进行，每次更新一个 Pod。

Kubernetes 控制平面会等到被更新的 Pod 进入 Running 和 Ready 状态，然后再更新其前身。 如果你设置了 `.spec.minReadySeconds`（查看[最短就绪秒数](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/#minimum-ready-seconds)）， 控制平面在 Pod 就绪后会额外等待一定的时间再执行下一步。

###### 分区滚动更新

通过声明 `.spec.updateStrategy.rollingUpdate.partition` 的方式，`RollingUpdate` 更新策略可以实现分区。 如果声明了一个分区，当 StatefulSet 的 `.spec.template` 被更新时， 所有序号大于等于该分区序号的 Pod 都会被更新。 所有序号小于该分区序号的 Pod 都不会被更新，并且，即使它们被删除也会依据之前的版本进行重建。 如果 StatefulSet 的 `.spec.updateStrategy.rollingUpdate.partition` 大于它的 `.spec.replicas`，则对它的 `.spec.template` 的更新将不会传递到它的 Pod。 在大多数情况下，你不需要使用分区，但如果你希望进行阶段更新、执行金丝雀或执行分阶段上线，则这些分区会非常有用。

###### 最大不可用 Pod

**特性状态：** `Kubernetes v1.24 [alpha]`

你可以通过指定 `.spec.updateStrategy.rollingUpdate.maxUnavailable` 字段来控制更新期间不可用的 Pod 的最大数量。 该值可以是绝对值（例如，“5”）或者是期望 Pod 个数的百分比（例如，`10%`）。 绝对值是根据百分比值四舍五入计算的。 该字段不能为 0。默认设置为 1。

该字段适用于 `0` 到 `replicas - 1` 范围内的所有 Pod。 如果在 `0` 到 `replicas - 1` 范围内存在不可用 Pod，这类 Pod 将被计入 `maxUnavailable` 值。

**说明：**

`maxUnavailable` 字段处于 Alpha 阶段，仅当 API 服务器启用了 `MaxUnavailableStatefulSet` [特性门控](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/feature-gates/)时才起作用。

###### 强制回滚

在默认 [Pod 管理策略](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/#pod-management-policies)(`OrderedReady`) 下使用[滚动更新](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/#rolling-updates)， 可能进入需要人工干预才能修复的损坏状态。

如果更新后 Pod 模板配置进入无法运行或就绪的状态（例如， 由于错误的二进制文件或应用程序级配置错误），StatefulSet 将停止回滚并等待。

在这种状态下，仅将 Pod 模板还原为正确的配置是不够的。 由于[已知问题](https://github.com/kubernetes/kubernetes/issues/67250)，StatefulSet 将继续等待损坏状态的 Pod 准备就绪（永远不会发生），然后再尝试将其恢复为正常工作配置。

恢复模板后，还必须删除 StatefulSet 尝试使用错误的配置来运行的 Pod。这样， StatefulSet 才会开始使用被还原的模板来重新创建 Pod。

##### PersistentVolumeClaim 保留

**特性状态：** `Kubernetes v1.23 [alpha]`

在 StatefulSet 的生命周期中，可选字段 `.spec.persistentVolumeClaimRetentionPolicy` 控制是否删除以及如何删除 PVC。 使用该字段，你必须启用 `StatefulSetAutoDeletePVC` [特性门控](https://kubernetes.io/zh-cn/docs/reference/command-line-tools-reference/feature-gates/)。 启用后，你可以为每个 StatefulSet 配置两个策略：

- `whenDeleted`

  配置删除 StatefulSet 时应用的卷保留行为。

- `whenScaled`

  配置当 StatefulSet 的副本数减少时应用的卷保留行为；例如，缩小集合时。

对于你可以配置的每个策略，你可以将值设置为 `Delete` 或 `Retain`。

- `Delete`

  对于受策略影响的每个 Pod，基于 StatefulSet 的 `volumeClaimTemplate` 字段创建的 PVC 都会被删除。 使用 `whenDeleted` 策略，所有来自 `volumeClaimTemplate` 的 PVC 在其 Pod 被删除后都会被删除。 使用 `whenScaled` 策略，只有与被缩减的 Pod 副本对应的 PVC 在其 Pod 被删除后才会被删除。

- `Retain`（默认）

  来自 `volumeClaimTemplate` 的 PVC 在 Pod 被删除时不受影响。这是此新功能之前的行为。

请记住，这些策略**仅**适用于由于 StatefulSet 被删除或被缩小而被删除的 Pod。 例如，如果与 StatefulSet 关联的 Pod 由于节点故障而失败， 并且控制平面创建了替换 Pod，则 StatefulSet 保留现有的 PVC。 现有卷不受影响，集群会将其附加到新 Pod 即将启动的节点上。

策略的默认值为 `Retain`，与此新功能之前的 StatefulSet 行为相匹配。

这是一个示例策略。

```yaml
apiVersion: apps/v1
kind: StatefulSet
...
spec:
  persistentVolumeClaimRetentionPolicy:
    whenDeleted: Retain
    whenScaled: Delete
...
```

StatefulSet [控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/)为其 PVC 添加了[属主引用](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/owners-dependents/#owner-references-in-object-specifications)， 这些 PVC 在 Pod 终止后被[垃圾回收器](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/garbage-collection/)删除。 这使 Pod 能够在删除 PVC 之前（以及在删除后备 PV 和卷之前，取决于保留策略）干净地卸载所有卷。 当你设置 `whenDeleted` 删除策略，对 StatefulSet 实例的属主引用放置在与该 StatefulSet 关联的所有 PVC 上。

`whenScaled` 策略必须仅在 Pod 缩减时删除 PVC，而不是在 Pod 因其他原因被删除时删除。 执行协调操作时，StatefulSet 控制器将其所需的副本数与集群上实际存在的 Pod 进行比较。 对于 StatefulSet 中的所有 Pod 而言，如果其 ID 大于副本数，则将被废弃并标记为需要删除。 如果 `whenScaled` 策略是 `Delete`，则在删除 Pod 之前， 首先将已销毁的 Pod 设置为与 StatefulSet 模板对应的 PVC 的属主。 这会导致 PVC 仅在已废弃的 Pod 终止后被垃圾收集。

这意味着如果控制器崩溃并重新启动，在其属主引用更新到适合策略的 Pod 之前，不会删除任何 Pod。 如果在控制器关闭时强制删除了已废弃的 Pod，则属主引用可能已被设置，也可能未被设置，具体取决于控制器何时崩溃。 更新属主引用可能需要几个协调循环，因此一些已废弃的 Pod 可能已经被设置了属主引用，而其他可能没有。 出于这个原因，我们建议等待控制器恢复，控制器将在终止 Pod 之前验证属主引用。 如果这不可行，则操作员应验证 PVC 上的属主引用，以确保在强制删除 Pod 时删除预期的对象。

##### 副本数

`.spec.replicas` 是一个可选字段，用于指定所需 Pod 的数量。它的默认值为 1。

如果你手动扩缩已部署的负载，例如通过 `kubectl scale statefulset statefulset --replicas=X`， 然后根据清单更新 StatefulSet（例如：通过运行 `kubectl apply -f statefulset.yaml`）， 那么应用该清单的操作会覆盖你之前所做的手动扩缩。

如果 [HorizontalPodAutoscaler](https://kubernetes.io/zh-cn/docs/tasks/run-application/horizontal-pod-autoscale/) （或任何类似的水平扩缩 API）正在管理 StatefulSet 的扩缩， 请不要设置 `.spec.replicas`。 相反，允许 Kubernetes 控制平面自动管理 `.spec.replicas` 字段。

## 排错

### 网络排错

[参考](https://mp.weixin.qq.com/s/r-AwtyWsb0aI4Ws5AjvaXg)
