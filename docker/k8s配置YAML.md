# YAML文件

[参考](http://victorfengming.gitee.io/kubernetes/7_Kubernetes%E9%9B%86%E7%BE%A4YAML%E6%96%87%E4%BB%B6%E8%AF%A6%E8%A7%A3/#kubernetes%E9%9B%86%E7%BE%A4yaml%E6%96%87%E4%BB%B6%E8%AF%A6%E8%A7%A3)、
[参考](https://juejin.cn/post/7107251448034885639#heading-3)
[Kubernetes指南](https://feisky.gitbooks.io/kubernetes/content/)

## 概述

k8s 集群中对资源管理和资源对象编排部署都可以通过声明样式（YAML）文件来解决，也就是可以把需要对资源对象操作编辑到YAML 格式文件中，我们把这种文件叫做资源清单文件，通过kubectl 命令直接使用资源清单文件就可以实现对大量的资源对象进行编排部署了。一般在我们开发的时候，都是通过配置YAML文件来部署集群的。

YAML文件：就是资源清单文件，用于资源编排

## 属性说明

|   属性名称     |   介绍     |
|    :---:      |   :---:    |
|apiVersion     |   API版本  |
|kind|资源类型|
|metadata|资源元数据|
|spec|资源规格|
|replicas|副本数量|
|selector|标签选择器|
|template|Pod模板|
|metadata|Pod元数据|
|spec|Pod规格|
|containers|容器配置|
|label|标签|
|namespace|命名空间|
|selector|选择器|
|strategy|策略|
|template|模板|
|volumeMounts|数据卷|

### apiVersion

只要记住6个常用的apiversion一般就够用了。

- v1： Kubernetes API的稳定版本，包含很多核心对象：pod、service等。
- apps/v1： 包含一些通用的应用层的api组合，如：Deployments, RollingUpdates, and ReplicaSets。
- `batch/v1`： 包含与批处理和类似作业的任务相关的对象，如：job、cronjob。
- `autoscaling/v1`： 允许根据不同的资源使用指标自动调整容器。
- `networking.k8s.io/v1`： 用于Ingress。
- `rbac.authorization.k8s.io/v1`：用于RBAC。

### kind

kind指定这个资源对象的类型，如 pod、deployment、statefulset、job、cronjob。

### metadata

metadata常用的配置项有 name,namespace,即配置其显示的名字与归属的命名空间。

### spec

一个嵌套字典与列表的配置项，也是主要的配置项，支持的子项非常多，根据资源对象的不同，子项会有不同的配置。期望pod的状态

### Namespace

名字空间（Namespace） 提供一种机制，将同一集群中的资源划分为相互隔离的组。 同一名字空间内的资源名称要唯一，但跨名字空间时没有这个要求。 名字空间作用域仅针对带有名字空间的对象， （例如 Deployment、Service 等），这种作用域对集群范围的对象 （例如 StorageClass、Node、PersistentVolume 等）不适用。

### Labels

标签（Labels） 是附加到 Kubernetes 对象（比如 Pod）上的键值对。 标签旨在用于指定对用户有意义且相关的对象的标识属性，但不直接对核心系统有语义含义。 标签可以用于组织和选择对象的子集。标签可以在创建时附加到对象，随后可以随时添加和修改。 每个对象都可以定义一组键/值标签。每个键对于给定对象必须是唯一的。



## 示例

deployment、pod、service 这三种资源的例子

### Deployment

```yaml
apiVersion: apps/v1        # 指定api版本，此值必须在kubectl api-versions中。业务场景一般首选”apps/v1“
kind: Deployment        # 指定创建资源的类型   
metadata:          # 资源的元数据/属性 
  name: demo      # 资源的名字，在同一个namespace中必须唯一
  namespace: default     # 部署在哪个namespace中。不指定时默认为default命名空间
  labels:          # 自定义资源的标签
    app: demo
    version: stable
  annotations:  # 自定义注释列表
    name: string
spec:     # 资源规范字段，定义deployment资源需要的参数属性，诸如是否在容器失败时重新启动容器的属性
  replicas: 1     # 声明副本数目
  revisionHistoryLimit: 3     # 保留历史版本
  selector:     # 标签选择器
    matchLabels:     # 匹配标签，需与上面的标签定义的app保持一致
      app: demo
      version: stable
  strategy:     # 策略
    type: RollingUpdate     # 滚动更新策略
                            # ecreate：删除所有已存在的pod,重新创建新的
                            # RollingUpdate：滚动升级，逐步替换的策略，同时滚动升级时，支持更多的附加参数，
                            # 例如设置最大不可用pod数量，最小升级间隔时间等等
    rollingUpdate:          # 滚动更新
      maxSurge: 1           # 滚动升级时最大额外可以存在的副本数，可以为百分比，也可以为整数
      maxUnavailable: 0     # 在更新过程中进入不可用状态的 Pod 的最大值，可以为百分比，也可以为整数
  template:     # 定义业务模板，如果有多个副本，所有副本的属性会按照模板的相关配置进行匹配
    metadata:     # 资源的元数据/属性 
      annotations:         # 自定义注解列表
        sidecar.istio.io/inject: "false"     # 自定义注解名字
      labels:    # 自定义资源的标签
        app: demo                       # 模板名称必填
        version: stable
    spec:   # 资源规范字段
      restartPolicy: Always             # Pod的重启策略。[Always | OnFailure | Nerver]
                                        # Always ：在任何情况下，只要容器不在运行状态，就自动重启容器。默认
                                        # OnFailure ：只在容器异常时才自动重启容器。
                                        # 对于包含多个容器的pod，只有它里面所有的容器都进入异常状态后，pod才会进入Failed状态
                                        # Nerver ：从来不重启容器
      nodeSelector:                     # 将该Pod调度到包含这个label的node上，仅能基于简单的等值关系定义标签选择器
        caas_cluster: work-node
      containers:        # Pod中容器列表
      - name: demo                      # 容器的名字   
        image: demo:v1                  # 容器使用的镜像地址   
        imagePullPolicy: IfNotPresent   # 每次Pod启动拉取镜像策略
                                        # IfNotPresent ：如果本地有就不检查，如果没有就拉取。默认 
                                        # Always : 每次都检查
                                        # Never : 每次都不检查（不管本地是否有）
        command: [string]     # 容器的启动命令列表，如不指定，使用打包时使用的启动命令
        args: [string]         # 容器的启动命令参数列表
            # 如果command和args均没有写，那么用Docker默认的配置
            # 如果command写了，但args没有写，那么Docker默认的配置会被忽略而且仅仅执行.yaml文件的command（不带任何参数的）
            # 如果command没写，但args写了，那么Docker默认配置的ENTRYPOINT的命令行会被执行，但是调用的参数是.yaml中的args
            # 如果如果command和args都写了，那么Docker默认的配置被忽略，使用.yaml的配置
        workingDir: string      # 容器的工作目录
        volumeMounts:        # 挂载到容器内部的存储卷配置
        - name: string         # 引用pod定义的共享存储卷的名称，需用volumes[]部分定义的的卷名
          mountPath: string        # 存储卷在容器内mount的绝对路径，应少于512字符
          readOnly: boolean        # 是否为只读模式
        - name: string
          configMap:         # 类型为configMap的存储卷，挂载预定义的configMap对象到容器内部
            name: string
            items:
            - key: string
              path: string
        ports:    # 需要暴露的端口库号列表
          - name: http     # 端口号名称
            containerPort: 8080     # 容器开放对外的端口 
          # hostPort: 8080    # 容器所在主机需要监听的端口号，默认与Container相同
            protocol: TCP     # 端口协议，支持TCP和UDP，默认TCP
        env:    # 容器运行前需设置的环境变量列表
        - name: string     # 环境变量名称
          value: string    # 环境变量的值
        resources:     # 资源管理。资源限制和请求的设置
          limits:     # 资源限制的设置，最大使用
            cpu: "1"         # CPU，"1"(1核心) = 1000m。将用于docker run --cpu-shares参数
            memory: 500Mi     # 内存，1G = 1024Mi。将用于docker run --memory参数
          requests:  # 资源请求的设置。容器运行时，最低资源需求，也就是说最少需要多少资源容器才能正常运行
            cpu: 100m
            memory: 100Mi
        livenessProbe:     # pod内部的容器的健康检查的设置。当探测无响应几次后将自动重启该容器
                          # 检查方法有exec、httpGet和tcpSocket，对一个容器只需设置其中一种方法即可
          httpGet: # 通过httpget检查健康，返回200-399之间，则认为容器正常
            path: /healthCheck     # URI地址。如果没有心跳检测接口就为/
            port: 8089         # 端口
            scheme: HTTP     # 协议
            # host: 127.0.0.1     # 主机地址
        # 也可以用这两种方法进行pod内容器的健康检查
        # exec:         # 在容器内执行任意命令，并检查命令退出状态码，如果状态码为0，则探测成功，否则探测失败容器重启
        #   command:   
        #     - cat   
        #     - /tmp/health   
        # 也可以用这种方法   
        # tcpSocket: # 对Pod内容器健康检查方式设置为tcpSocket方式
        #   port: number 
          initialDelaySeconds: 30     # 容器启动完成后首次探测的时间，单位为秒
          timeoutSeconds: 5     # 对容器健康检查等待响应的超时时间，单位秒，默认1秒
          periodSeconds: 30     # 对容器监控检查的定期探测间隔时间设置，单位秒，默认10秒一次
          successThreshold: 1     # 成功门槛
          failureThreshold: 5     # 失败门槛，连接失败5次，pod杀掉，重启一个新的pod
        readinessProbe:         # Pod准备服务健康检查设置
          httpGet:
            path: /healthCheck    # 如果没有心跳检测接口就为/
            port: 8089
            scheme: HTTP
          initialDelaySeconds: 30
          timeoutSeconds: 5
          periodSeconds: 10
          successThreshold: 1
          failureThreshold: 5
        lifecycle:        # 生命周期管理  
          postStart:    # 容器运行之前运行的任务  
            exec:  
              command:  
                - 'sh'  
                - 'yum upgrade -y'  
          preStop:        # 容器关闭之前运行的任务  
            exec:  
              command: ['service httpd stop']
      initContainers:        # 初始化容器
      - command:
        - sh
        - -c
        - sleep 10; mkdir /wls/logs/nacos-0
        env:
        image: {{ .Values.busyboxImage }}
        imagePullPolicy: IfNotPresent
        name: init
        volumeMounts:
        - mountPath: /wls/logs/
          name: logs
      volumes:
      - name: logs
        hostPath:
          path: {{ .Values.nfsPath }}/logs
      volumes:     # 在该pod上定义共享存储卷列表
      - name: string         # 共享存储卷名称 （volumes类型有很多种）
        emptyDir: {}         # 类型为emtyDir的存储卷，与Pod同生命周期的一个临时目录。为空值
      - name: string
        hostPath:          # 类型为hostPath的存储卷，表示挂载Pod所在宿主机的目录
          path: string    # Pod所在宿主机的目录，将被用于同期中mount的目录
      - name: string
        secret:             # 类型为secret的存储卷，挂载集群与定义的secre对象到容器内部
          scretname: string  
          items:     
          - key: string
            path: string
      imagePullSecrets:     # 镜像仓库拉取镜像时使用的密钥，以key：secretkey格式指定
      - name: harbor-certification
      hostNetwork: false    # 是否使用主机网络模式，默认为false，如果设置为true，表示使用宿主机网络
      terminationGracePeriodSeconds: 30     # 优雅关闭时间，这个时间内优雅关闭未结束，k8s 强制 kill
      dnsPolicy: ClusterFirst    # 设置Pod的DNS的策略。默认ClusterFirst
              # 支持的策略：[Default | ClusterFirst | ClusterFirstWithHostNet | None]
              # Default : Pod继承所在宿主机的设置，也就是直接将宿主机的/etc/resolv.conf内容挂载到容器中
              # ClusterFirst : 默认的配置，所有请求会优先在集群所在域查询，如果没有才会转发到上游DNS
              # ClusterFirstWithHostNet : 和ClusterFirst一样，不过是Pod运行在hostNetwork:true的情况下强制指定的
              # None : 1.9版本引入的一个新值，这个配置忽略所有配置，以Pod的dnsConfig字段为准
      affinity:  # 亲和性调试
        nodeAffinity:     # 节点亲和性。满足特定条件的pod对象运行在同一个node上。效果同nodeSelector，但功能更强大
          requiredDuringSchedulingIgnoredDuringExecution:     # 硬亲和性
            nodeSelectorTerms:         # 节点满足任何一个条件就可以
            - matchExpressions:     # 有多个选项时，则只有同时满足这些逻辑选项的节点才能运行 pod
              - key: beta.kubernetes.io/arch
                operator: In
                values:
                - amd64
        podAffinity:     # pod亲和性。满足特定条件的pod对象运行在同一个node上
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector: 
              matchExpressions: 
              - key: app
                operator: In
                values: 
                - nginx
            topologyKey: kubernetes.io/hostname
        podAntiAffinity:     # pod反亲和性。满足特定条件（相同pod标签）的pod对象不能运行在同一个node上
          requiredDuringSchedulingIgnoredDuringExecution: 
          - labelSelector: 
              matchExpressions: 
              - key: app
                operator: In
                values: 
                - nginx
            topologyKey: kubernetes.io/hostname        # 反亲和性的节点标签 key
      tolerations:        # 污点容忍度。允许pod可以运行在匹配的污点上
      - operator: "Equal"        # 匹配类型。支持[Exists | Equal(默认值)]。Exists为容忍所有污点
        key: "key1"
        value: "value1"
        effect: "NoSchedule"        # 污点类型：[NoSchedule | PreferNoSchedule | NoExecute]
                                        # NoSchedule ：不会被调度 
                                        # PreferNoSchedule：尽量不调度
                                        # NoExecute：驱逐节点
```

### pod

```yaml
apiVersion: v1
kind: Pod  
metadata:  
  name: pod-redis
  labels:
    name: redis
spec: 
  containers:
  - name: pod-redis
    image: docker.io/redis  
    ports:
    - containerPort: 80 #容器对外的端口
```

### Service

```yaml
apiVersion: v1     # 指定api版本，此值必须在kubectl api-versions中 
kind: Service     # 指定创建资源的角色/类型 
metadata:     # 资源的元数据/属性
  name: demo     # 资源的名字，在同一个namespace中必须唯一
  namespace: default     # 部署在哪个namespace中。不指定时默认为default命名空间
  labels:         # 设定资源的标签
  - app: demo
  annotations:  # 自定义注解属性列表
  - name: string
spec:     # 资源规范字段
  type: ClusterIP     # service的类型，指定service的访问方式，默认ClusterIP。
      # ClusterIP类型：虚拟的服务ip地址，用于k8s集群内部的pod访问，在Node上kube-porxy通过设置的iptables规则进行转发
      # NodePort类型：使用宿主机端口，能够访问各个Node的外部客户端通过Node的IP和端口就能访问服务器
      # LoadBalancer类型：使用外部负载均衡器完成到服务器的负载分发，需要在spec.status.loadBalancer字段指定外部负载均衡服务器的IP，并同时定义nodePort和clusterIP用于公有云环境。
  clusterIP: string        #虚拟服务IP地址，当type=ClusterIP时，如不指定，则系统会自动进行分配，也可以手动指定。当type=loadBalancer，需要指定
  sessionAffinity: string    #是否支持session，可选值为ClietIP，默认值为空。ClientIP表示将同一个客户端(根据客户端IP地址决定)的访问请求都转发到同一个后端Pod
  ports:
    - port: 8080     # 服务监听的端口号
      targetPort: 8080     # 容器暴露的端口
      nodePort: int        # 当type=NodePort时，指定映射到物理机的端口号
      protocol: TCP     # 端口协议，支持TCP或UDP，默认TCP
      name: http     # 端口名称
  selector:     # 选择器。选择具有指定label标签的pod作为管理范围
    app: demo
status:    # 当type=LoadBalancer时，设置外部负载均衡的地址，用于公有云环境    
  loadBalancer:    # 外部负载均衡器    
    ingress:
      ip: string    # 外部负载均衡器的IP地址
      hostname: string    # 外部负载均衡器的主机名
```

### ingress

```yaml
apiVersion: extensions/v1beta1         # 创建该对象所使用的 Kubernetes API 的版本     
kind: Ingress         # 想要创建的对象的类别，这里为Ingress
metadata:
  name: showdoc        # 资源名字，同一个namespace中必须唯一
  namespace: op     # 定义资源所在命名空间
  annotations:         # 自定义注解
    kubernetes.io/ingress.class: nginx        # 声明使用的ingress控制器
spec:
  rules:
  - host: showdoc.example.cn     # 服务的域名
    http:
      paths:
      - path: /      # 路由路径
        backend:     # 后端Service
          serviceName: showdoc        # 对应Service的名字
          servicePort: 80           # 对应Service的端口
```