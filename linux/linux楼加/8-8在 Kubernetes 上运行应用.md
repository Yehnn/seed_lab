---
show: step
version: 1.0
enable_checker: true
---
# 在 Kubernetes 上运行应用

## 1. 实验介绍

Kubernetes 存在的目的就是为了方便、稳定、可靠的运行应用。本次实验我们来学习如何在 Kubernetes 集群里运行简单的无状态应用。无状态应用没有状态（或者说状态保存在应用之外，比如数据库），可以随时在不同的 Node 之间迁移运行实例。

> 实验环境已经部署好 Kubernetes，进入实验环境后在 `HOME` 目录下执行 `./dind-cluster-v1.10.sh up` 即可启动集群。

## 2. Pod

Pod 是应用在 Kubernetes 集群里的运行实例，一个应用可以在多个 Node 上运行多个 Pod 来扩展性能或者保障高可用性。

### 2.1. 理解 Pod

Pod 是 Kubernetes 中你可以创建和部署的最小也是最简的单位。一个 Pod 代表着集群中运行的一个进程。Pod 中封装着应用的容器（有的情况下是好几个容器）、存储、网络，管理着容器如何运行的策略选项。Pod 代表着一个部署单位，也就是 Kubernetes 中应用的一个运行实例，可能由一个或者多个共享资源的容器组成。Docker 是 Kubernetes 中最常用的容器运行时，但是 Pod 也支持其他容器运行时。

在 Kubrenetes 集群中 Pod 有如下两种使用方式：

- 一个 Pod 中运行一个容器。每个 Pod 中一个容器的模式是最常见的用法。在这种使用方式中，你可以把 Pod 想象成是单个容器的封装，Kuberentes 管理的是 Pod 而不是直接管理容器。
- 在一个 Pod 中同时运行多个容器。一个 Pod 中也可以同时封装几个需要紧密耦合互相协作的容器，它们之间共享资源。这些在同一个 Pod 中的容器可以互相协作成为一个服务单位。

每个 Pod 都是应用的一个实例。如果你想平行扩展应用的话（运行多个实例），你应该运行多个 Pod，每个 Pod 都是一个应用实例。在 Kubernetes 中，这通常被称为复制（replication）。

### 2.2. Pod 中如何管理多个容器

Pod 中可以同时运行多个进程（作为容器运行）协同工作。同一个 Pod 中的容器会自动的分配到同一个 node 上。同一个 Pod 中的容器共享资源、网络和依赖，它们总是被同时调度。

注意在一个 Pod 中同时运行多个容器是一种比较高级的用法。只有当你的容器需要紧密配合协作的时候才考虑用这种模式。例如，你有一个容器作为 Web 服务器运行，需要用到共享的 Volume，还有另一个 Sidecar 容器来从远端获取资源更新这些文件。如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5984timestamp1529486681731.png/wm)

Pod中可以共享两种资源：

#### 2.2.1. 网络

每个 Pod 都会被分配一个唯一的 IP 地址。Pod 中的所有容器共享网络空间，包括 IP 地址和端口。Pod 内部的容器可以使用 localhost 互相通信。Pod 中的容器与外界通信时，必须分配共享网络资源（例如使用宿主机的端口映射）。

#### 2.2.2. 存储

可以为 Pod 指定多个共享的 Volume。Pod 中的所有容器都可以访问这些共享的 Volume。Volume 也可以用来持久化 Pod 中的存储资源，以防容器重启后文件丢失。

### 2.3. 使用 Pod

你很少会直接在 Kubernetes 中创建单个 Pod。因为 Pod 的生命周期是短暂的，用后即焚的实体。当 Pod 被创建后（不论是由你直接创建还是被其它 Controller），都会被Kuberentes 调度到集群的 Node 上。直到 Pod 的进程终止、被删掉、因为缺少资源而被驱逐、或者 Node 故障之前，这个 Pod 都会一直保持在那个 Node 上。

Pod 不会自愈。如果 Pod 运行的 Node 故障，或者是调度器本身故障，这个 Pod 就会被删除。同样的，如果 Pod 所在 Node 缺少资源或者 Pod 处于维护状态，Pod 也会被驱逐。Kubernetes 使用更高级的称为 Controller 的抽象层来管理 Pod 实例。虽然可以直接使用 Pod，但是在 Kubernetes 中通常是使用 Controller 来管理 Pod。

### 2.4. Pod 和 Controller

Controller 可以创建和管理多个 Pod，提供副本管理、滚动升级和集群级别的自愈能力。例如，如果一个 Node 故障，Controller 就能自动将该节点上的 Pod 调度到其他健康的 Node 上。通常，Controller 会用你提供的 Pod template 来创建相应的 Pod。

### 2.5. Pod Template

Pod 模版是包含了其他对象的 Pod 定义，例如 Replication Controllers，Jobs 和 DaemonSets。Controller 根据 Pod 模板来创建实际的 Pod。

## 3. Deployment

Deployment 为 Pod 和 ReplicaSet 提供声明式更新。只需要在 Deployment 中描述想要的目标状态是什么，Deployment controller 就会将 Pod 和 ReplicaSet 的实际状态改变到目标状态。可以定义一个全新的 Deployment 来创建 ReplicaSet 或者删除已有的 Deployment 并创建一个新的来替换。

### 3.1. 创建 Deployment

下面是一个 Deployment 示例，它创建了一个 ReplicaSet 来启动 3 个 nginx Pod。

下载 Deployment 示例文件并执行 `create` 命令：

```bash
$ kubectl create -f https://Kubernetes.io/docs/user-guide/nginx-deployment.yaml --record
deployment "nginx-deployment" created
```

也可手动创建包含如下内容的 Deployment 文件（注意相应替换上面命令里的 `-f` 选项的值为本地文件路径）：

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

将 kubectl 的 `--record` 的 flag 设置为 true 可以在 Annotation 中记录当前命令创建或升级了该资源。这在未来会很有用，例如查看每个 Deployment revision 中执行了哪些命令。

然后立即执行 `kubectl get deployments` 命令将获得如下结果：

```bash
$ kubectl get deployments
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         0         0            0           1s
```

输出结果表明我们希望的 repalica 数是 3，当前 replica 数是 0, 最新的 replica 数是 0，可用的 replica 数是 0。

过段时间后再执行 `kubectl get deployments` 命令，将获得如下输出：

```bash
$ kubectl get deployments
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           18s
```

可以看到 Deployment 已经创建了 3 个 replica，所有的 replica 都是最新的。

执行 `kubectl get rs` 和 `kubectl get pods` 会查看已创建的 RS 和 Pod。

```bash
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-2035384211   3         3         0       18s
$ kubectl get pods --show-labels
nginx-deployment-75675f5897-gsx7s   1/1       Running   0          6m        app=nginx,pod-template-hash=3123191453
nginx-deployment-75675f5897-m84fc   1/1       Running   0          6m        app=nginx,pod-template-hash=3123191453
nginx-deployment-75675f5897-z887c   1/1       Running   0          6m        app=nginx,pod-template-hash=3123191453
```

刚创建的 RS 将保证总是有 3 个 Pod 存在。RS 和 Pod 可以看做是 Deployment 在某一时刻的状态，每次 Deployment 更新，都会导致 RS 和 Pod 对象销毁和重建。

### 3.2. 更新 Deployment

假如我们现在想要让 Pod 使用 nginx:1.9.1 的镜像来代替原来的 nginx:1.7.9 的镜像。

```bash
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
deployment "nginx-deployment" image updated
$ kubectl get pods --show-labels
nginx-deployment-75675f5897-gsx7s   1/1       Running             0          8m        app=nginx,pod-template-hash=3123191453
nginx-deployment-75675f5897-m84fc   1/1       Running             0          8m        app=nginx,pod-template-hash=3123191453
nginx-deployment-75675f5897-z887c   1/1       Running             0          8m        app=nginx,pod-template-hash=3123191453
nginx-deployment-c4747d96c-xg6f7    0/1       ContainerCreating   0          3s        app=nginx,pod-template-hash=703038527
$ kubectl get pods --show-labels
NAME                               READY     STATUS    RESTARTS   AGE       LABELS
nginx-deployment-c4747d96c-bhg6p   1/1       Running   0          7s        app=nginx,pod-template-hash=703038527
nginx-deployment-c4747d96c-m46p2   1/1       Running   0          36s       app=nginx,pod-template-hash=703038527
nginx-deployment-c4747d96c-xg6f7   1/1       Running   0          1m        app=nginx,pod-template-hash=703038527
```

第一次执行 `kubectl get pods --show-labels` 看到开始更新，原先的三个 Pod 都还在（通过 pod-template-hash 标签来区分）。经过一段时间后，出现了三个新的 Pod，原先的三个 Pod 都删除了。更新过程是逐个替换 Pod，尽量不影响服务的可用性。

也可执行 `kubectl edit deployment/nginx-deployment` 命令来更新 Deployment。该命令会在编辑器里打开 Deployment 文件，修改 `.spec.template.spec.containers[0].image` 节点的值为 `nginx:1.9.1`，以及其它任何需要修改的地方。保存退出后，Kubernetes 会检测到其中的变化，并调整集群中的应用来达到新的部署状态要求。

### 3.3. 回退 Deployment

有时候可能想回退一个 Deployment，例如当新的 Deployment 一直在循环崩溃和重启。默认情况下，Kubernetes 会在系统中保存前两次 Deployment 的滚动（rollout）历史记录，以便可以随时回退。只要 Deployment 的 rollout 被触发就会创建一个新的 revision。也就是说当且仅当 Deployment 的 Pod template 被更改，例如更新 template 中的 label 和容器镜像时，就会创建一个新的 revision。其他的更新，比如扩容 Deployment 不会创建 revision，因此我们可以很方便的手动或者自动扩容。这意味着当回退到历史 revision 时，只有 Deployment 中的 Pod template 部分才会回退。

假设我们在更新 Deployment 的时候犯了一个拼写错误，将镜像的名字写成了nginx:1.91，而正确的名字应该是 nginx:1.9.1：

```bash
$ kubectl set image deployment/nginx-deployment nginx=nginx:1.91
deployment "nginx-deployment" image updated
```

此时 rollout 将会卡住。执行下面的命令可以看到一直在等待 rollout 完成。

```bash
$ kubectl rollout status deployments nginx-deployment
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
```

按住 `Ctrl+C` 停止上面的 rollout 状态监控。然后执行下面的命令可以看到新的 Deployment 里 READY 的副本数一直是 0，而老的 Deployment 一直维持原样。这是因为没有新的副本启动成功，老的副本也就没法销毁。

```bash
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-595696685f   1         1         0         2m
nginx-deployment-75675f5897   3         3         3         32m
```

然后看下 Pod 状况，会看到新建的 Pod，状态一直是 ImagePullBackOff，也就是拉取镜像失败。

```bash
$ kubectl get pods
NAME                                READY     STATUS             RESTARTS   AGE
nginx-deployment-595696685f-z4lfr   0/1       ImagePullBackOff   0          5m
nginx-deployment-75675f5897-c47bc   1/1       Running            0          36m
nginx-deployment-75675f5897-kgnq7   1/1       Running            0          36m
nginx-deployment-75675f5897-pp89q   1/1       Running            0          36m
```

可以使用 `kubectl describe deployment` 来查看 Deployment 当前状态。从 Replicas 项可以看到，目标是 3 个，1 个在升级，3 个可用，1 个不可用。

```bash
$ kubectl describe deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Thu, 24 May 2018 14:12:37 +0800
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision=2
                        kubernetes.io/change-cause=kubectl set image deployment/nginx-deployment nginx=nginx:1.91
Selector:               app=nginx
Replicas:               3 desired | 1 updated | 4 total | 3 available | 1 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.91
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  nginx-deployment-75675f5897 (3/3 replicas created)
NewReplicaSet:   nginx-deployment-595696685f (1/1 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  39m   deployment-controller  Scaled up replica set nginx-deployment-75675f5897 to 3
  Normal  ScalingReplicaSet  9m    deployment-controller  Scaled up replica set nginx-deployment-595696685f to 1
```

为了修复这个问题，我们需要回退到稳定的 Deployment revision。

#### 3.3.1. 查看 Deployment 升级历史

使用下面的命令来查看 Deployment 的 revision：

```bash
$ kubectl rollout history deployment/nginx-deployment
deployments "nginx-deployment"
REVISION  CHANGE-CAUSE
1         kubectl create --filename=nginx-deployment.yaml --record=true
2         kubectl set image deployment/nginx-deployment nginx=nginx:1.91
```

因为之前创建 Deployment 的时候使用了 `--record` 参数来记录，所以可以查看到每次 revision 的变化。

使用下面的命令来查看单个 revision 的详细信息：

```bash
$ kubectl rollout history deployment/nginx-deployment --revision=2
deployments "nginx-deployment" with revision #2
Pod Template:
  Labels: app=nginx
  pod-template-hash=1512522419
  Annotations: kubernetes.io/change-cause=kubectl set image deployment/nginx-deployment nginx=nginx:1.91
  Containers:
   nginx:
    Image: nginx:1.91
    Port: 80/TCP
    Host Port: 0/TCP
    Environment: 、<none>
    Mounts: 、<none>
  Volumes: 、<none>
```

#### 3.3.2. 回退到历史版本

使用下面的命令来回退到上一个版本：

```bash
$ kubectl rollout undo deployment/nginx-deployment
deployment.apps "nginx-deployment"
```

也可以使用 `--revision` 参数来回退到某个指定的历史版本：

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=1
```

查看 Deployment 和 RS 状态，可以看到均已恢复正常，新建的 RS 副本数已经为 0。

```bash
$ kubectl get deployment
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3         3         3            3           52m
$ kubectl get rs
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-595696685f   0         0         0         22m
nginx-deployment-75675f5897   3         3         3         52m
```

#### 3.3.3. 清理策略

你可以通过设置 `.spec.revisonHistoryLimit` 项来指定 Deployment 最多保留多少次 revision 历史记录。默认会保留所有，如果将该项设置为 0，Deployment 将不允许回退。

### 3.4. Deployment 扩容

可以使用下面的命令来扩容 Deployment：

```bash
$ kubectl scale deployment nginx-deployment --replicas 5
deployment.extensions "nginx-deployment" scaled
```

执行下面的命令来确认 Deployment 的副本数扩容到了 5 个。

```bash
$ kubectl get deployment
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   5         5         5            5           1h
```

还可使用 `autoscale` 命令来在适当的时候自动扩容。下面的命令表示尽量让 Deployment 的所有 Pod 的平均 CPU 使用率接近于 80%。如果高于此值，增加 Pod 个数，否则减少 Pod 个数。

```bash
$ kubectl autoscale deployment nginx-deployment --min=3 --max=10 --cpu-percent=80
deployment.apps "nginx-deployment" autoscaled
```

## 4. 实验知识点

- Pod 对象介绍
- 通过 Deployment 对象来运行应用
- 应用更新
- 应用回退

## 5. 实验总结

本次实验通过学习 Pod 和 Deployment 对象，我们掌握了如何在 Kubernetes 集群里运行应用，以及如何更新和回退应用版本。