---
show: step
version: 1.0
enable_checker: true
---
# Kubernetes 服务

## 1. 实验介绍

前面的实验我们将应用成功的在 Kubernetes 集群里运行了起来，但这些应用服务怎样才能被访问到了。Kubernetes 里通过 Service 对象来描述由一组 Pod 提供的服务。服务默认只能在集群内访问，如果要服务对外提供服务，则需要发布服务。

## 2. 实验知识点

- 定义服务
- 发布服务
- 实例操作

## 3. 简介

Kubernetes Pod 是有生命周期的，它们可以被创建，也可以被销毁，一旦被销毁生命就永远结束。通过 ReplicationSet 能够动态地创建和销毁 Pod（例如，需要进行扩缩容，或者执行滚动升级）。 每个 Pod 都会获取它自己的 IP 地址，但这些 IP 地址不是稳定可依赖的。这会导致一个问题：在 Kubernetes 集群中，如果一组 Pod（称为 backend）为其它 Pod （称为 frontend）提供服务，那么 frontend 该如何发现，并连接到 backend 中的某个 Pod 呢？

### 3.1. 关于 Service

Kubernetes Service 定义了这样一种抽象：一个 Pod 的逻辑分组，一种可以访问它们的策略，通常称为微服务。 这一组 Pod 能够被 Service 访问到，通常是通过 Label Selector（但有时也需要没有 selector 的 Service）来实现的。

举个例子，考虑一个图片处理 backend，它运行了 3 个副本。这些副本是可互换的，frontend 不需要关心它们具体调用了哪个 backend 副本。然而组成这一组 backend 服务的 Pod 实际上可能会发生变化，frontend 客户端不应该也没必要知道这一组 backend 的状态。Service 定义的抽象能够解耦这种关联。

## 4. 定义 Service

一个 Service 在 Kubernetes 中是一个 REST 对象，和 Pod 类似。像所有的 REST 对象一样，可以使用 POST 方式来请求 API Server 创建 Service 对象。例如，假定有一组 Pod，它们对外暴露了 9376 端口，同时还被打上 app=MyApp 标签。

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

上述配置将创建一个名称为 my-service 的 Service 对象，它会将请求代理到使用 TCP 端口 9376，并且具有标签 app=MyApp 的 Pod 上。这个 Service 将被指派一个集群 IP（Cluster IP），它会被服务的代理使用。该 Service 的 selector 将会持续评估，处理结果将被 POST 到一个名称为 my-service 的 Endpoints 对象上。

需要注意的是，Service 能够将一个接收端口映射到任意的 targetPort。默认情况下，targetPort 将被设置为与 port 相同的值。另外，targetPort 还可以是一个字符串，引用了 backend Pod 的一个端口名称。但实际指派给该端口名称的端口号，在每个 backend Pod 中可能并不相同。对于部署和设计 Service，这种方式会提供更大的灵活性。 例如，可以在 backend 软件下一个版本中，修改 Pod 暴露的端口，却并不会中断客户端的调用。
 
Kubernetes Service 支持 TCP 和 UDP 协议，默认 TCP 协议。

### 4.1. 没有 selector 的 Service

Servcie 抽象了该如何访问 Kubernetes Pod，但也能够抽象其它类型的 backend，例如：

- 希望在生产环境中使用外部的数据库集群，但测试环境使用自己的数据库。
- 希望服务指向另一个 Namespace 中或其它集群中的服务。
- 正在将工作负载转移到 Kubernetes 集群，需要访问运行在 Kubernetes 集群之外的 backend。

在任何这些场景中，都能够定义没有 selector 的 Service：

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

由于这个 Service 没有 selector，就不会创建相关的 Endpoints 对象。可以手动将 Service 映射到指定的 Endpoints：

```yaml
kind: Endpoints
apiVersion: v1
metadata:
  name: my-service
subsets:
  - addresses:
      - ip: 1.2.3.4
    ports:
      - port: 9376
```

> 注意，Endpoint IP 地址不能是 loopback（127.0.0.0/8）、link-local（169.254.0.0/16）、或者 link-local 多播（224.0.0.0/24）。

访问没有 selector 的 Service，与有 selector 的 Service 的原理相同。请求将被路由到用户定义的 Endpoint（该示例中为 1.2.3.4:9376）。

ExternalName Service 是 Service 的特例，它没有 selector，也没有定义任何的端口和 Endpoint。相反地，对于运行在集群外部的服务，它通过返回该外部服务的别名这种方式来提供服务。

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
  namespace: prod
spec:
  type: ExternalName
  externalName: my.database.example.com
```

当查询主机 my-service.prod.svc.CLUSTER 时，集群的 DNS 服务将返回一个值为 my.database.example.com 的 CNAME 记录。访问这个服务的工作方式与其它的相同，唯一不同的是重定向发生在 DNS 层，而且不会进行代理或转发。如果后续决定要将数据库迁移到 Kubernetes 集群中，可以启动对应的 Pod，增加合适的 Selector 或 Endpoint，修改 Service 的 type，但服务使用方不用做任何更改。

## 5. VIP 和 Service 代理

在 Kubernetes 集群中，每个 Node 运行一个 kube-proxy 进程。kube-proxy 为 Service 实现了一种 VIP（Virtual IP）。在 Kubernetes v1.0 版本，代理在 userspace。在 Kubernetes v1.1 版本，新增了 iptables 代理，但并不是默认的运行模式。从 Kubernetes v1.2 起，默认使用 iptables 代理。

在 Kubernetes v1.0 版本，Service 是 4 层（TCP/UDP over IP）概念。在 Kubernetes v1.1 版本，新增了 Ingress API，用来表示 7 层（HTTP）服务。

### 5.1. userspace 代理模式

这种模式，kube-proxy 会监视 Kubernetes master 对 Service 对象和 Endpoints 对象的添加和移除。对每个 Service，它会在本地 Node 上打开一个代理端口（随机选择）。任何连接到代理端口的请求，都会被代理到 Service 的 backend Pods 中的某个上面（从 Endpoints 中获知）。使用哪个 backend Pod，是基于 Service 的会话亲和性（SessionAffinity）来确定的。它会安装 iptables 规则，捕获到达 Service 的 clusterIP（Virtual IP）和 Port 的请求，并重定向到代理端口，代理端口再代理请求到 backend Pod。

结果是，任何到达 Service 的 IP:Port 的请求，都会被代理到一个合适的 backend，不需要客户端知道关于 Kubernetes、Service 或 Pod 的任何信息。

默认的策略是通过 round-robin 算法来选择 backend Pod。要实现基于客户端 IP 的会话亲和性，可以通过设置 `service.spec.sessionAffinity` 的值为 ClientIP （默认值为 None）来实现。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5987timestamp1529487428026.png/wm)

### 5.2. iptables 代理模式

这种模式，kube-proxy 会监视 Kubernetes master 对 Service 对象和 Endpoints 对象的添加和移除。对每个 Service，它会安装 iptables 规则，从而捕获到达该 Service 的 clusterIP 和端口的请求，进而将请求重定向到 Service 的一组 backend Pod 中的某个上面。对于每个 Endpoints 对象，它也会安装 iptables 规则，这个规则会选择一个 backend Pod。

同 userspace 代理模式，默认的策略是随机选择一个 backend。要实现基于客户端 IP 的会话亲和性，可以将 `service.spec.sessionAffinity` 的值设置为 ClientIP。

和 userspace 代理类似，结果是任何到达 Service 的 IP:Port 的请求，都会被代理到一个合适的 backend，不需要客户端知道关于 Kubernetes、Service 或 Pod 的任何信息。这种方式比 userspace 代理更快、更可靠。然而，不像 userspace 代理，如果初始选择的 Pod 没有响应，iptables 代理不会自动地重试另一个 Pod，所以它需要依赖 readiness probes。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5987timestamp1529487455535.png/wm)

### 5.3. ipvs 代理模式

这种模式是 Kubernetes v1.9 里新增的。在此模式下，kube-proxy 监视 Kubernetes Services 和 Endpoints，调用 `netlink` 接口来相应地创建 ipvs 规则，并定期与 Kubernetes Services 和 Endpoints 同步 ipvs 规则，以确保 ipvs 状态与预期一致。当访问服务时，流量将被重定向到其中一个后端 Pod。

与 iptables 类似，ipvs 基于 netfilter 钩子函数，但使用散列表作为基础数据结构并在内核空间中工作。这意味着 ipvs 可以更快地重定向流量，并且在同步代理规则时具有更好的性能。此外，ipvs 为负载均衡算法提供了更多选项，例如：

- rr：轮询
- lc：最少连接
- dh：目标哈希
- sh：来源哈希
- sed：最短的预期延迟
- nq：不排队

注意：在运行 kube-proxy 之前，ipvs 模式需要在节点上已安装 IPVS 内核模块。当 kube-proxy 以 ipvs 代理模式启动时，kube-proxy 会验证节点上是否安装了 IPVS 模块，如果未安装，kube-proxy 将回退到 iptables 代理模式。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5987timestamp1529487470646.png/wm)

在以上这些代理模式中，为服务的 IP:Port 绑定的流量都会代理到适当的后端，而客户端不知道任何关于 Kubernetes、Service 或 Pod 的信息。通过设置`service.spec.sessionAffinity` 为 ClientIP，可以选择基于客户端 IP 的会话关联，并且还可通过 `service.spec.sessionAffinityConfig.clientIP.timeoutSeconds` 字段来设置最大会话粘滞时间（默认为 10800）。

## 6. 多端口 Service

很多 Service 需要暴露多个端口。对于这种情况，Kubernetes 支持在 Service 对象中定义多个端口。当使用多个端口时，必须给出所有的端口的名称，这样 Endpoint 就不会产生歧义，例如：

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
    selector:
      app: MyApp
    ports:
      - name: http
        protocol: TCP
        port: 80
        targetPort: 9376
      - name: https
        protocol: TCP
        port: 443
        targetPort: 9377
```

## 7. 选择自己的 IP 地址

在 Service 创建请求中，可以通过设置 spec.clusterIP 字段来指定自己的 Cluster IP 地址。比如，希望替换一个已经已存在的 DNS 条目，或者遗留系统已经配置了一个固定的 IP 且很难重新配置。用户选择的 IP 地址必须合法，并且这个 IP 地址在 service-cluster-ip-range CIDR 范围内，这在 API Server 里可通过一个标识来指定。如果 IP 地址不合法，API Server 会返回 HTTP 状态码 422，表示值不合法。

## 8. 服务发现

Kubernetes 支持 2 种基本的服务发现模式，环境变量和 DNS。

### 8.1. 环境变量

当 Pod 运行在 Node 上，kubelet 会为每个活跃的 Service 添加一组环境变量。它同时支持 Docker links 兼容变量、简单的 {SVCNAME}_SERVICE_HOST 和 {SVCNAME}_SERVICE_PORT 变量，这里 Service 的名称需大写，横线被转换成下划线。

举个例子，一个名称为 redis-master 的 Service 暴露了 TCP 端口 6379，同时给它分配了 Cluster IP 地址 10.0.0.11，这个 Service 生成了如下环境变量：

```text
REDIS_MASTER_SERVICE_HOST=10.0.0.11
REDIS_MASTER_SERVICE_PORT=6379
REDIS_MASTER_PORT=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP=tcp://10.0.0.11:6379
REDIS_MASTER_PORT_6379_TCP_PROTO=tcp
REDIS_MASTER_PORT_6379_TCP_PORT=6379
REDIS_MASTER_PORT_6379_TCP_ADDR=10.0.0.11
```

这种模式有顺序要求，Pod 想要访问的任何 Service 必须在 Pod 自己之前被创建，否则这些环境变量就不会被赋值。而 DNS 模式没有这个限制。

### 8.2. DNS

一个可选（强烈推荐）的集群插件是 DNS 服务器。DNS 服务器监视着创建新 Service 的 Kubernetes API，从而为每一个 Service 创建一组 DNS 记录。如果整个集群的 DNS 一直被启用，那么所有的 Pod 应该能够自动对 Service 进行名称解析。

例如，有一个名称为 my-service 的 Service，它在 Kubernetes 集群中名为 my-ns 的 Namespace 中，为 my-service.my-ns 创建了一条 DNS 记录。在名称为 my-ns 的 Namespace 中的 Pod 能够简单地通过服务名查询找到 my-service。在另一个 Namespace 中的 Pod 必须限定名称为 my-service.my-ns。这些名称查询的结果是 Cluster IP。

Kubernetes 也支持对端口名称的 DNS SRV（Service）记录。如果名称为 my-service.my-ns 的 Service 有一个名为 http 的 TCP 端口，可以对 _http._tcp.my-service.my-ns 执行 DNS SRV 查询，得到 http 的端口号。

Kubernetes DNS 服务器是唯一的一种能够访问 ExternalName 类型的 Service 的方式。

## 9. Headless Service

有时不需要或不想要负载均衡，以及单独的 Service IP。遇到这种情况，可以通过指定 Cluster IP 的值为 None 来创建 Headless Service。这个选项允许开发人员使用他们自己的方式来路由请求，从而降低与 Kubernetes 系统的耦合性，比如使用自己的服务注册和发现机制。

对这类 Service 并不会分配 Cluster IP，kube-proxy 不会处理它们，而且平台也不会为它们进行负载均衡和路由。DNS 是否实现自动配置，依赖于 Service 是否定义了 selector。

### 9.1. 配置 Selector

对定义了 selector 的 Headless Service，Endpoint 控制器在 API 中创建了 Endpoints 记录，并且配置 DNS A 记录，通过这个地址直接到达 Service 的后端 Pod 上。

### 9.2. 不配置 Selector

对没有定义 selector 的 Headless Service，Endpoint 控制器不会创建 Endpoints 对象。对下面这些情况，DNS 系统会进行配置：

- ExternalName 类型 Service 的 CNAME 记录
- 与 Service 共享一个名称的任何 Endpoints

## 10. 发布服务

对一些应用（如 Frontend）的某些部分，希望通过外部（Kubernetes 集群外部）IP 地址暴露 Service。Kubernetes ServiceTypes 允许指定 Service 类型，默认是 ClusterIP。

Type 的取值以及行为如下：

- ClusterIP：通过集群的内部 IP 暴露服务，选择该值，服务只能够在集群内部可以访问，这也是默认的 ServiceType。
- NodePort：通过每个 Node 上的 IP 和端口（NodePort）暴露服务。NodePort 服务会路由到 ClusterIP 服务，这个 ClusterIP 服务会自动创建。通过请求 \<NodeIP>:\<NodePort>，可以从集群的外部访问一个 NodePort 服务。
- LoadBalancer：使用云提供商的负载均衡器，可以向外部暴露服务。外部的负载均衡器可以路由到 NodePort 服务和 ClusterIP 服务。
- ExternalName：通过返回 CNAME 和它的值，可以将服务映射到 externalName（例如，foo.bar.example.com）。没有任何类型代理被创建，只有 Kubernetes 1.7 或更高版本的 kube-dns 才支持。

### 10.1. NodePort 类型

如果设置 type 的值为 NodePort，Kubernetes master 将从给定的配置范围内（默认 30000-32767）分配端口，每个 Node 将从该端口代理 Service 请求。该端口将通过 Service 的 spec.ports[*].nodePort 字段被指定。如果需要指定的端口号，可以配置 nodePort 的值，系统将分配这个端口，如果端口分配失败则调用 API 也将失败。

这种类型下让开发人员可以自由地安装他们自己的负载均衡器，并配置 Kubernetes 不能完全支持的环境参数，或者直接暴露一个或多个 Node 的 IP 地址。需要注意的是，Service 可通过 <NodeIP>:spec.ports[*].nodePort 和 spec.clusterIp:spec.ports[*].port 访问到。

### 10.2. LoadBalancer 类型

使用支持外部负载均衡器的云提供商的服务，设置 type 的值为 LoadBalancer，将为 Service 提供负载均衡器。负载均衡器是异步创建的，关于负载均衡器的信息可通过 Service 的 status.loadBalancer 字段查看到。

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
      nodePort: 30061
  clusterIP: 10.0.171.239
  loadBalancerIP: 78.11.24.19
  type: LoadBalancer
status:
  loadBalancer:
    ingress:
      - ip: 146.148.47.155
```

来自外部负载均衡器的流量将直接导入到 backend Pod 上，不过实际它们是如何工作的，依赖于云提供商。某些云提供商允许指定 loadBalancerIP。如果没有设置 loadBalancerIP，将会给负载均衡器指派一个临时 IP。如果设置了 loadBalancerIP，但云提供商并不支持这种特性，那么设置的 loadBalancerIP 值将会被忽略掉。

### 10.3. 外部 IP

如果有外部 IP 路由到一个或多个群集节点，则 Kubernetes 服务可暴露在这些 externalIPs 上。通过外部IP（以及服务端口）进入群集的流量将被路由到服务端点中的一个。externalIPs 不受 Kubernetes 管理，由集群管理员负责。

在 ServiceSpec，externalIPs 可以与任何类型的服务一起指定。在下面的例子中，my-service 可以通过 80.11.12.10:80 地址访问到。

```yaml
kind: Service
apiVersion: v1
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
  externalIPs:
    - 80.11.12.10
```

## 11. 动手实验

下面我们来创建一个包含两个实例的“Hello World”应用，并创建一个 Service 来将它暴露给外部。

1、在集群里运行一个 Hello World 应用，包含两个实例。

```bash
$ kubectl run hello-world --replicas=2 --labels="run=load-balancer-example" --image=gcr.io/google-samples/node-hello:1.0  --port=8080
deployment.apps "hello-world" created
```

2、查看 Deployment 对象。

```bash
$ kubectl get deployments hello-world
NAME          DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
hello-world   2         2         2            2           9s
$ kubectl describe deployment hello-world
Name:                   hello-world
Namespace:              default
CreationTimestamp:      Thu, 31 May 2018 15:32:30 +0800
Labels:                 run=load-balancer-example
Annotations:            deployment.kubernetes.io/revision=1
Selector:               run=load-balancer-example
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  1 max unavailable, 1 max surge
Pod Template:
  Labels:  run=load-balancer-example
  Containers:
   hello-world:
    Image:        gcr.io/google-samples/node-hello:1.0
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   hello-world-5b446dd74b (2/2 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  18s   deployment-controller  Scaled up replica set hello-world-5b446dd74b to 2
```

3、查看 ReplicaSet 对象。

```bash
$ kubectl get replicasets
NAME                     DESIRED   CURRENT   READY     AGE
hello-world-5b446dd74b   2         2         2         2m
$ kubectl describe replicaset hello-world-5b446dd74b
Name:           hello-world-5b446dd74b
Namespace:      default
Selector:       pod-template-hash=1600288306,run=load-balancer-example
Labels:         pod-template-hash=1600288306
                run=load-balancer-example
Annotations:    deployment.kubernetes.io/desired-replicas=2
                deployment.kubernetes.io/max-replicas=3
                deployment.kubernetes.io/revision=1
Controlled By:  Deployment/hello-world
Replicas:       2 current / 2 desired
Pods Status:    2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  pod-template-hash=1600288306
           run=load-balancer-example
  Containers:
   hello-world:
    Image:        gcr.io/google-samples/node-hello:1.0
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  2m    replicaset-controller  Created pod: hello-world-5b446dd74b-bchvp
  Normal  SuccessfulCreate  2m    replicaset-controller  Created pod: hello-world-5b446dd74b-v9chh
```

4、将 Deployment 暴露为一个 NodePort 类型的 Service。

```bash
$ kubectl expose deployment hello-world --type=NodePort --name=example-service
service "example-service" exposed
```

5、查看 Service 对象。

```bash
$ kubectl describe service example-service
Name:                     example-service
Namespace:                default
Labels:                   run=load-balancer-example
Annotations:              <none>
Selector:                 run=load-balancer-example
Type:                     NodePort
IP:                       10.98.205.10
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30496/TCP
Endpoints:                10.244.2.4:8080,10.244.3.2:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

这里可以看到自动分配的 NodePort 是 30496，记住这个端口，后面会用到。

6、查看运行 Hello World 应用的 Pods。

```bash
$ kubectl get pods --selector="run=load-balancer-example" --output=wide
NAME                           READY     STATUS    RESTARTS   AGE       IP           NODE
hello-world-5b446dd74b-bchvp   1/1       Running   0          4m        10.244.3.2   kube-node-2
hello-world-5b446dd74b-v9chh   1/1       Running   0          4m        10.244.2.4   kube-node-1
```

7、查看 Node IP。

```bash
$ kubectl get nodes
NAME          STATUS    ROLES     AGE       VERSION
kube-master   Ready     master    13d       v1.10.3
kube-node-1   Ready     <none>    13d       v1.10.3
kube-node-2   Ready     <none>    13d       v1.10.3
$ docker exec kube-node-1 ifconfig eth0
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.192.0.3  netmask 255.255.0.0  broadcast 10.192.255.255
        ether 02:42:0a:c0:00:03  txqueuelen 0  (Ethernet)
        RX packets 1148  bytes 642950 (627.8 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1181  bytes 141275 (137.9 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

$ docker exec kube-node-2 ifconfig eth0
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.192.0.4  netmask 255.255.0.0  broadcast 10.192.255.255
        ether 02:42:0a:c0:00:04  txqueuelen 0  (Ethernet)
        RX packets 1238  bytes 754439 (736.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 1230  bytes 194255 (189.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

我们是以 NodePort 的模式创建的服务，外部需要通过 NodeIP:NodePort 地址来访问服务。我们的 Kubernetes 集群节点是用 Docker 容器模拟的，所以可以使用 `docker exec` 在容器内执行命令来查询 Node 的 IP。

8、访问服务。

```bash
$ curl http://10.192.0.3:30496/
Hello Kubernetes!
```

> 注意，如果使用某个节点 IP 无法访问，那么可以换另外一个节点的 IP。端口为前面查询到的在 Node 上的映射端口。

## 12. 实验总结

本次实验我们先学习了 Kubernetes 中服务相关的基础知识，然后实际动手来创建了一个服务。通过本次实验，大家能够基本掌握 Kubernetes 中服务的使用。实际应用中，每个云服务商支持的方式和细节会有所不同，需要查阅其文档来正确使用。
