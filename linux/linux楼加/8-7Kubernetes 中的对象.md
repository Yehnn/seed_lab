---
show: step
version: 1.0
enable_checker: true
---
# Kubernetes 中的对象

## 1. 实验介绍

Kubernetes 里存在大量对象，基本上每个核心概念都对应有一个对象。这些对象除了在系统内部使用，也通过 API 开放给外部系统使用。本次实验我们先来学习一些基础对象，这些对象是后续实验的基础，更多的对象在后面实验中用到时再讲。

## 2. 对象清单

以下列举的都是 Kubernetes 中的对象，这些对象可以在 Kubernetes 的配置文件中作为一种 kind（类型）来配置。

- Pod
- Node
- Namespace
- Service
- Volume
- PersistentVolume
- Deployment
- Secret
- StatefulSet
- DaemonSet
- ServiceAccount
- ReplicationController
- ReplicaSet
- Job
- CronJob
- SecurityContext
- ResourceQuota
- LimitRange
- HorizontalPodAutoscaling
- Ingress
- ConfigMap
- Label
- ThirdPartyResources

上面的对象可简单分为以下几类：

| 类别     | 名称                                                         |
| -------- | ------------------------------------------------------------ |
| 资源对象 | Pod、ReplicaSet、ReplicationController、Deployment、StatefulSet、DaemonSet、Job、CronJob、HorizontalPodAutoscaling |
| 配置对象 | Node、Namespace、Service、Secret、ConfigMap、Ingress、Label、ThirdPartyResource、 ServiceAccount |
| 存储对象 | Volume、Persistent Volume                                    |
| 策略对象 | SecurityContext、ResourceQuota、LimitRange                   |

## 3. 理解 Kubernetes 中的对象

Kubernetes 中的对象是持久化的条目，Kubernetes 使用这些条目去表示整个集群的状态。它们能够描述如下信息：

- 什么容器化应用在运行（以及在哪个 Node 上）
- 可以被应用使用的资源
- 关于应用如何表现的策略，比如重启策略、升级策略，以及容错策略
- Kubernetes 对象是“目标性记录”。一旦创建对象，Kubernetes 系统将持续工作以确保对象存在。通过创建对象，可以有效地告知 Kubernetes 系统，所需要的集群工作负载看起来应该是什么样子的，这就是 Kubernetes 集群的 **期望状态**。

与 Kubernetes 对象工作需要使用 Kubernetes API。当使用 kubectl 命令行接口时，它会调用 Kubernetes API。也可以在程序中直接调用 Kubernetes API。为了方便程序开发，Kubernetes 提供了多种语言的客户端库，包括 Go、Python、Java、dotnet 和 JavaScript。具体可参考 [官网](https://kubernetes.io/docs/reference/client-libraries/)。

### 3.1. Spec 与 Status

每个 Kubernetes 对象包含两个嵌套的对象字段，它们负责管理对象的配置，分别是 spec 和 status。spec 必须提供，它描述了对象的期望状态。status 描述了对象的实际状态，它由 Kubernetes 系统提供和更新。在任何时刻，Kubernetes 控制平面一直处于活跃状态，管理着对象的实际状态，使其与我们期望的状态相匹配。

例如，Kubernetes Deployment 对象表示运行在集群中的应用。当创建 Deployment 时，需要设置 Deployment 的 spec，以指定该应用需要有几个（比如 3 个）副本在运行。Kubernetes 系统读取 Deployment spec，启动我们所期望的该应用的 3 个实例。如果那些实例中有失败，Kubernetes 系统通过修正来响应 spec 和 status 的不一致。

### 3.2. 描述 Kubernetes 对象

当创建 Kubernetes 对象时，必须提供对象的 spec，用来描述该对象的期望状态，以及关于对象的一些基本信息（例如名称）。当使用 Kubernetes API 创建对象时（直接创建或通过 kubectl），API 请求必须在请求体中包含 JSON 格式的信息。更常用的做法是在 YAML 格式的文件中为 kubectl 提供这些信息，YAML 格式比 JSON 更易读。kubectl 在执行 API 请求时，会将这些信息转换成 JSON 格式。

这里有一个示例文件 `nginx-deployment.yaml`，展示了 Kubernetes Deployment 的必需字段：

```yaml
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

可以使用 `kubectl create` 命令来创建该 Deployment：

```bash
$ kubectl create -f nginx-deployment.yaml --record
deployment "nginx-deployment" created
```

### 3.3. 必需字段

在用于创建 Kubernetes 对象的 .yaml 文件中，需要配置如下的字段：

- apiVersion 创建该对象所使用的 Kubernetes API 版本
- kind 想要创建的对象的类型
- metadata 帮助识别对象唯一性的数据，包括 name 和 可选的 UID、namespace
- spec 精确格式随每个 Kubernetes 对象而不同，包含特定于该对象的嵌套字段

## 4. Node

Node 是 Kubernetes 集群的工作节点，可以是物理机也可以是虚拟机。

### 4.1. Node的状态

Node 包括如下状态信息：

- Address
  - HostName：可以被 kubelet 中的 `--hostname-override` 参数替代。
  - ExternalIP：可以被集群外部路由到的 IP 地址。
  - InternalIP：集群内部使用的 IP，集群外部无法访问。
- Condition
  - OutOfDisk：磁盘空间不足时为 True
  - Ready：Node controller 40 秒内没有收到 Node 状态报告为 Unknown，健康为 True，否则为 False。
  - MemoryPressure：没有内存压力时为 True，否则为 False。
  - DiskPressure：没有磁盘压力时为 True，否则为 False。
- Capacity
  - CPU
  - 内存
  - 可运行的最大 Pod 个数
- Info：节点的一些版本信息，如 OS、 Kubernetes、Docker 等

### 4.2. Node 管理

禁止 Pod 调度到该节点上：

```bash
kubectl cordon
```

驱逐该节点上的所有 Pod：

```bash
kubectl drain
```

该命令会删除该节点上的所有 Pod（DaemonSet除外），在其他 Node 上重新启动它们，通常在节点需要维护时使用该命令。使用该命令会自动调用 `kubectl cordon` 命令。当该节点维护完成，启动了 kubelet 后，再使用 `kubectl uncordon` 即可将该节点恢复。

## 5. Namespace

在一个 Kubernetes 集群中可以使用 namespace 创建多个“虚拟集群”，这些 namespace 之间可以完全隔离，也可以通过某种方式让一个 namespace 中的服务可以访问到其他的 namespace 中的服务，这需要通过 RBAC 定义集群级别的角色来实现。

### 5.1. 哪些情况适合使用多个 namespace

因为 namespace 可以提供独立的命名空间，因此可以实现部分的环境隔离。当你的项目和人员众多的时候可以考虑根据项目属性，例如生产、测试、开发划分不同的 namespace。

### 5.2. Namespace 使用

获取集群中有哪些 namespace：

```bash
kubectl get ns
```

集群中默认会有 `default` 和 `kube-system` 这两个 namespace。执行 kubectl 命令时可以使用 `-n` 指定要操作的 namespace。用户的普通应用默认是在 default 下，与集群管理相关的为整个集群提供服务的应用一般部署在 kube-system 下。另外，并不是所有的资源对象都会对应 namespace，node 和 persistentVolume 就不属于任何 namespace。

## 6. Label

Label 是附着到对象（例如 Pod）上的键值对。可以在创建对象的时候指定，也可以在对象创建后随时指定。Label 的值对系统本身并没有什么含义，只对用户有意义。

```json
"labels": {
  "key1" : "value1",
  "key2" : "value2"
}
```

Kubernetes 将对 Label 进行索引和反向索引用来优化查询和检测变化 ，在 UI 和命令行中会对它们排序。不要在 Label 中使用大型、非标识的结构化数据，记录这样的数据应该用 Annotation。

### 6.1. 动机

Label 能够将组织架构映射到系统架构上，这样更便于管理服务。比如你可以给对象打上如下这些类型的 Label：

- "release": "stable", "release": "canary"
- "environment": "dev", "environment": "qa", "environment": "production"
- "tier": "frontend", "tier": "backend", "tier": "cache"
- "partition": "customerA", "partition": "customerB"
- "track": "daily", "track": "weekly"
- "team": "teamA","team:": "teamB"

### 6.2. 语法和字符集

Label key 的组成：

- 不得超过 63 个字符
- 可以使用前缀，使用 / 分隔，前缀必须是 DNS 子域，不得超过 253 个字符，系统中的自动化组件创建的 Label 必须指定前缀，`kubernetes.io/` 由 Kubernetes 保留
- 起始必须是字母（大小写都可以）或数字，中间可以有连字符、下划线和点

Label value 的组成：

- 不得超过 63 个字符
- 起始必须是字母（大小写都可以）或数字，中间可以有连字符、下划线和点

### 6.3. Label selector

Label 不是唯一的，很多对象可能有相同的 Label。通过 Label selector，客户端/用户可以得到一个对象集合，然后对该集合进行操作。

Label selector 有两种类型：

- equality-based：可以使用 `=`、`==`、`!=` 操作符，使用逗号分隔多个表达式
- set-based：可以使用 `in`、`notin`、`!` 操作符

另外还可以没有操作符，直接写出某个 Label 的 key，表示过滤有某个 key 的对象而不管该 key 的 value 是何值，`!` 表示没有该 Label 的对象

### 6.4. 示例

```bash
kubectl get pods -l environment=production,tier=frontend
kubectl get pods -l 'environment in (production),tier in (frontend)'
kubectl get pods -l 'environment in (production, qa)'
kubectl get pods -l 'environment,environment notin (frontend)'
```

## 7. Annotation

Annotation，顾名思义，就是注解。Annotation 可以将 Kubernetes 资源对象关联到任意的非标识性元数据。使用客户端（工具和库）可以检索到这些元数据。

### 7.1. 关联元数据到对象

Label 和 Annotation 都可以将元数据关联到 Kubernetes 资源对象。Label 主要用于选择对象，可以挑选出满足特定条件的对象。相比之下，Annotation 不能用于标识及选择对象。Annotation 中的元数据可多可少，可以是结构化的或非结构化的，也可以包含 Label 中不允许出现的字符。

Annotation 和 Label 一样都是 key/value 键值对映射结构：

```json
"annotations": {
  "key1" : "value1",
  "key2" : "value2"
}
```

以下列出了一些可以记录在 Annotation 中的对象信息：

- 声明配置层管理的字段。使用 Annotation 关联这类字段可以用于区分不同的配置来源：客户端或服务器设置的默认值，自动生成的字段或自动生成的 auto-scaling 和 auto-sizing 系统配置的字段。
- 创建信息、版本信息或镜像信息。例如时间戳、版本号、git 分支、PR 序号、镜像哈希值以及仓库地址。
- 记录日志、监控、分析或审计存储仓库的指针。
- 可以用于 debug 的客户端（库或工具）信息，例如名称、版本和创建信息。
- 用户信息、工具或系统来源信息，例如来自非 Kubernetes 生态的相关对象的 URL 信息。
- 轻量级部署工具元数据，例如配置或检查点。
- 负责人的电话或联系方式，或能找到相关信息的目录条目信息，例如团队网站。

如果不使用 Annotation，您也可以将以上类型的信息存放在外部数据库或目录中，但这样做不利于创建用于部署、管理、内部检查的共享工具和客户端库。

## 8. 实验知识点

- Kubernetes 中有哪些对象
- Kubernetes 中对象的标准属性
- Node 对象
- Label 对象
- Annotation 对象

## 9. 实验总结

本次实验我们学习了 Kubernetes 中的一些基础对象，了解这些对象有助于理解后面的实验。