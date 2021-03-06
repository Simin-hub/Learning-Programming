# API

[参考地址](https://www.redhat.com/zh/topics/api/what-are-application-programming-interfaces)

**应用编程接口**（application programming interface, API）是一组**用于构建和集成应用软件的定义和协议**。

API 由一组定义和协议组合而成，可用于构建和集成应用软件。有时我们可以把它们当做信息提供者和信息用户之间的合同——**建立消费者（呼叫）所需的内容和制作者（响应）要求的内容**。例如，天气服务的 API 可指定用户提供邮编，制作者回复的答案由两部分组成，第一部分是最高温度，第二部分是最低温度。

换言之，如果您想与计算机或系统交互以检索信息或执行某项功能，API 可帮助您将您需要的信息传达给该系统，使其能够理解并满足您的请求。 

您可以把 API 看做是用户或客户端与他们想要的资源或 Web 服务之间的传递者。它也是企业在共享资源和信息的同时保障安全、控制和身份验证的一种方式，即确定哪些人可以访问什么内容。 

API 的另一个优势是您无需了解缓存的具体信息，即如何检索资源或资源来自哪里。

## API 的工作原理是什么？

通过 API，您无需了解实施原理，也能将您的产品或服务与其他的互通。这样可以简化应用的开发，节省时间和成本。在您开发新的工具和产品，或[管理](https://www.redhat.com/zh/topics/management)现有工具和产品时，强大灵活的 API 可以帮助您简化[设计](https://www.redhat.com/zh/topics/api/what-is-api-design)、管理和使用，并带来更多创新机遇。

API 有时被视为合同，而合同文本则代表了各方之间的协议：如果一方以特定方式发送远程请求，该协议规定了另一方的软件将如何做出响应。

由于 API 简化了开发人员将新应用组件[集成到](https://www.redhat.com/zh/topics/integration)现有[基础架构](https://www.redhat.com/zh/topics/cloud-native-apps/what-is-an-application-architecture)中的方式，继而也对业务和 IT 团队之间的协作提供了帮助。随着数字市场日新月异，业务需求通常也会出现迅速变化，新的竞争对手利用新的应用即可改变整个行业。为了保持竞争力，支持快速开发和部署创新服务尤为重要。[云原生应用](https://www.redhat.com/zh/topics/cloud-native-apps)开发是提高开发速度的一个明确办法，依赖于通过 API 连接 [微服务](https://www.redhat.com/zh/topics/microservices)应用架构。

API 是通过云原生应用开发来连接您自己的基础架构的一个简化方式，此外还支持您向客户和其他外部用户分享您的数据。公共 API 有着独特的商业价值，因其可以简化和扩展您与合作伙伴的联系方式，并有望支持您从数据中实现盈利（Google Maps API 便是其中一个广为人知的案例）。

![img](https://www.redhat.com/cms/managed-files/styles/wysiwyg_full_width/s3/API-page-graphic.png?itok=5zMemph9)

例如，我们假设有这样一家图书发行公司：该发行商可以提供一个[云应用](https://www.redhat.com/zh/topics/cloud-native-apps/what-are-cloud-applications)，供书店店员查看书的库存情况。这款应用的开发成本高昂、有平台限制、开发周期较长，还需进行日常维护。

或者，该发行商也可以提供一个 API 来查询库存情况。这一方法有以下几个好处：

- API 可以帮助他们将所有库存相关信息汇集到一处，供客户方便地访问数据。
- 只要 API 的行为不发生变化，该图书发行商就能在不影响客户的情况下更改内部系统。
- 借助公开可用的 API，开发人员可为该图书发行商、图书销售商或第三方开发一个应用，帮助客户查找所需的图书。这样不仅能提高销量，还能带来其他的商机。

简而言之，API 既能让您开放自己的资源访问权限，又能确保[安全性并让您继续握有控制权](https://www.redhat.com/zh/topics/security)。如何以及向谁开放访问权限由您自己决定。[API 安全防护](https://www.redhat.com/zh/topics/security/api-security)离不开良好的 API 管理，其中包括对 [API 网关](https://www.redhat.com/zh/topics/api/what-does-an-api-gateway-do)的使用。借助整合各种资源（包括传统系统和[物联网](https://www.redhat.com/zh/topics/internet-of-things/what-is-iot)（IoT））的分布式整合平台，可以连接至 API 并创建使用 API 所提供的数据或功能的应用。

### API 发行政策

#### 私有

这类 API 仅供内部使用，能让公司最大限度地控制自己的 API。

#### 合作伙伴

您会和特定的业务合作伙伴共享这类 API。它能在不影响质量的情况下带来额外收益。

#### 公共

这类 API 可供所有人使用。第三方可以使用这类 API 来开发应用，以便与您的 API 进行互动或实现某种创新。

## 利用 API 实现创新

通过向合作伙伴或公众提供您的 API，可以：

- 创造新的收入渠道，或拓展现有收入渠道。
- 扩大您的品牌覆盖范围。
- 通过外部开发和协作，推动开放创新或提高效率。

听上去还不错，对吗？但是，API 是如何做到上述几点的呢？

我们继续以刚才的那个图书发行公司为例。

假设该公司的某个合作伙伴开发了一个应用，可以帮助人们查找书店书架上的图书。这种体验的改进为书店带来了更多的客户（该发行商的客户），并拓展了现有的收入渠道。

或许会有第三方使用某个公共 API 来开发应用，以便人们直接从该发行商处（而非书店）购书。这样就能为该图书发行商打开新的收入渠道。

与特定合作伙伴或全世界共享 API 能带来积极的影响。除了自己公司的营销活动之外，每一个合作关系都能帮助您提高品牌辨识度。以公共 API 的形式向所有人开放技术，可以激励开发人员以您的 API 为中心构建应用生态系统。有更多的人使用您的技术，就意味着可能会有更多的人与您做生意。

公开技术可以带来意外之喜。有时，这些惊喜更会颠覆整个行业。对于这家图书发行公司而言，新形态的公司（比如图书出借服务）可能会完全改变他们的业务模式。借助合作伙伴和公共 API，您可以激发社区成员的创意（其人数远超您的内部开发团队）。各种奇思妙想会源源不断地涌现，公司只需明辨市场的变化情况并做好相应的准备。API 大有用处。

## API 简史

API 概念的出现，始于计算机时代的初期，远远早于个人电脑诞生之前。当时，API 常被当作操作系统的库，而且基本上都在本地系统上运行，仅偶尔用于大型机之间传递消息。将近 30 年后，API 走出了它们的本地环境。到了 21 世纪初，API 成为了用于实现数据远程集成的一种重要技术。

## 远程 API

远程 API 旨在通过通信网络进行互动。这里的*远程*是指 API 操控的资源不在提出请求的计算机上。由于互联网是应用最广泛的通信网络，所以大多数 API 都是基于 Web 标准来设计的。并非所有的远程 API 都是 Web API，但可以认为 Web API 都是远程 API。

Web API 通常会使用 HTTP 来传输请求消息，并提供响应消息的结构定义。这些响应消息通常都会以 XML 或 JSON 文件的形式来提供。XML 和 JSON 都是首选格式，因为它们会以易于其他应用操纵的方式来呈现数据。

## SOAP VS REST

随着 Web API 的不断普及，相应的协议规范也随之产生了，从而推动了信息交换的标准化：**简单对象访问协议，简称 SOAP**。使用 **SOAP 设计的 API 会使用 XML 格式来收发消息，并通过 HTTP 或 SMTP 来接收请求**。使用 SOAP 时，在不同环境中运行的应用或使用不同语言编写的应用能够更加轻松地共享信息。

相关的规范还有一个，即[表述性状态传递（REST）](https://www.redhat.com/zh/topics/api/what-is-a-rest-api)。**遵循 REST 架构约束的 Web API 被称为 RESTful API**。REST 与 SOAP 有着根本区别：SOAP 是一种协议，而 REST 是一种架构模式。这意味着 RESTful Web API 没有官方标准。正如 Roy Fielding 在论文"Architectural Styles and the Design of Network-based Software Architectures"（架构模式以及基于网络的软件架构的设计）中定义的那样，只要 API 符合 RESTful 系统的 6 个导向性约束，就算作 RESTful API：

- **客户端/服务器架构：**REST 架构由客户端、服务器和资源构成，通过 HTTP 来处理请求。
- **无状态：**请求所经过的服务器上不会存储任何客户端内容。与会话状态相关的信息会存储在客户端上。
- **可缓存性：**通过缓存，可免去客户端与服务器之间的某些交互。
- **分层系统：**客户端与服务器之间的交互可以通过额外的层来进行调解。这些层可以提供额外的功能，如负载均衡、共享缓存或安全防护。
- **按需代码（可选）：**服务器可通过传输可执行代码来扩展客户端的功能。
- **统一接口：**这项约束是 RESTful API 的设计核心，共涵盖 4 个层面：
  - **识别请求中的资源：**请求中的资源会被识别，并与返回给客户端的表示内容分离开来。
  - **通过不同的表示内容来操纵资源：**客户端会收到表示不同资源的文件。这些表示内容必须提供足够的信息，以便执行修改或删除操作。
  - **自描述消息：**返回给客户端的每个消息都包含充足的信息，用于指明客户端应该如何处理所收到的信息。
  - **将超媒体作为应用状态的引擎：**在访问某个资源后，REST 客户端应该能够通过超链接来发现当前可用的所有其他操作。

虽然看似有很多约束需要遵循，但是这些约束遵循起来要比遵循规定的协议容易得多。因此，RESTful API 现在变得比 SOAP 更为普及。

近年来，OpenAPI 规范已发展成为定义 REST API 的通用标准。OpenAPI 为开发人员提供了一种与语言无关的方式来构建 REST API 接口，从而最大程度减少不确定的因素，让用户安心工作。

另一个新兴的 API 标准是 [GraphQL](https://www.redhat.com/zh/topics/api/what-is-graphql)，这是一种可替代 REST 的查询语言和服务器端运行时。GraphQL 可优先让客户端准确地获得所需的数据，没有任何冗多余。作为 REST 的替代方案，GraphQL 允许开发人员构建相应的请求，从而通过单个 API 调用从多个数据源中提取数据。

[详细了解 SOAP 和REST](https://www.redhat.com/zh/topics/integration/whats-the-difference-between-soap-rest)

## SOA 与微服务架构

最常使用远程 API 的 2 种架构方案分别是：面向服务的架构（SOA）和[微服务](https://www.redhat.com/zh/topics/cloud-native-apps/what-is-service-oriented-architecture)架构。在这 2 种方案中，SOA 的历史更为久远一些。最初，它是在单体式应用的基础上经过改进而形成的。虽然单个单体式应用也可以完成各种操作，但通过某种集成模式（如企业服务总线（ESB））在不同应用间实现松散耦合后，即可获得某些功能。

从大多数层面来看，SOA 都要比单体式架构更简单；但是，如果无法明确理解各种组件交互，SOA 也可能会进一步加剧整个环境的复杂性。这种复杂性的加剧会重新引发 SOA 想要解决的某些问题。

对于专用松散耦合服务的使用，微服务架构与 SOA 模式类似。但是，微服务架构会对传统架构进行进一步细分。在微服务架构中，服务会采用通用消息传递框架，如 RESTful API。它们会使用 RESTful API 来实现相互通信，且无需执行繁琐的数据转换处理或使用其他的集成层。使用 RESTful API 可以加速新功能和新更新的交付；甚至还可以说，是这类 API 促进了这种速度的提升。该架构中的每一个服务都呈离散状态。一个服务可以被取代、增强或丢弃，而不会影响架构中的任何其他服务。这种轻量级架构有助于优化分布式资源或云资源，而且能够支持个别服务的动态扩展。

[详细了解 SOA](https://www.redhat.com/zh/topics/cloud-native-apps/what-is-service-oriented-architecture)

## [RESTful API](https://github.com/Simin-hub/Golang-Learning-and-Interview/blob/main/Go/%E5%AE%9E%E6%88%98/RESTFul%20JSON%20API.md)



## [RPC](https://github.com/Simin-hub/Golang-Learning-and-Interview/blob/main/Go/%E8%BF%9B%E9%98%B6/RPC.md)