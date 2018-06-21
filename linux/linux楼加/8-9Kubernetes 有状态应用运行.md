# Kubernetes 有状态应用运行

## 实验介绍

有状态应用的状态需要在 Pod 销毁和重建之间保持，那么这些状态需要在应用运行过程中持久化到存储里。Kubernetes 里持久化存储通过 Volume 抽象来实现，Volume 底层实现可以是云存储或本地存储。Kubernetes 里使用 PV（PersistentVolume）来表示存储资源，通过 PVC（PersistentVolumeClaim）来声明存储资源需求。PV/PVC 的关系类似于 Node/Pod，只不过一个描述的是存储资源，另一个描述的是计算资源。有状态应用的 Pod 需要绑定 PV，在 Pod 销毁和重建之间要保证绑定到之前的 PV，另外很多时候 Pod 的启动顺序有要求。Kubernetes 里提供了 StatefulSet 对象来控制有状态应用的运行。本次实验我们先学习 Kubernetes 里跟存储相关的对象，然后再学习如何使用 StatefulSet 来控制有状态应用的运行。

## 实验知识点

- Volume
- PersistentVolume 和 PersistentVolumeClaim
- StatefulSet

## Volume（卷）

容器里文件的生命周期是短暂的，这使得在容器中运行某些应用时会出现问题。比如当容器崩溃时，kubelet 会重启它，但是容器中的文件将丢失，因为容器会以干净的状态（镜像最初状态）重新启动。另外在 Pod 中同时运行多个容器时，这些容器之间通常需要共享文件。Kubernetes 使用卷抽象很好的解决了这些问题。

### 背景

Docker 中也有卷的概念，但它比较宽松，功能也比较简单。在 Docker 中，卷是磁盘或其它容器中的一个目录，它的生命周期不受管理。即便 Docker 现在提供了卷驱动程序，但功能还是比较有限。

卷的生命比 Pod 中的容器长，当容器重启时数据仍然得以保存。Kubernetes 支持多种类型的卷，Pod 可以同时使用任意数量的卷。要使用卷，需要为 Pod 指定卷（spec.volumes）以及将它挂载到容器的位置（spec.containers.volumeMounts）。

容器中的进程看到的是由其 Docker 镜像和卷组成的文件系统。Docker 镜像位于文件系统层次结构的根目录，任何卷都被挂载在镜像的指定路径中。卷无法挂载到其他卷上或与其它卷有硬连接。Pod 中的每个容器都必须独立指定每个卷的挂载位置。

### 卷的类型

Kubernetes 支持以下类型的卷：

- awsElasticBlockStore
- azureDisk
- azureFile
- cephfs
- csi
- downwardAPI
- emptyDir
- fc (fibre channel)
- flocker
- gcePersistentDisk
- gitRepo
- glusterfs
- hostPath
- iscsi
- local
- nfs
- persistentVolumeClaim
- projected
- portworxVolume
- quobyte
- rbd
- scaleIO
- secret
- storageos
- vsphereVolume

Kubernetes 是开放的，如果没有适合的，可以自己去实现一种新的卷。下面我们选两种典型的卷来了解一下其使用方法，一种是云存储卷，另一种是本地存储卷。

### awsElasticBlockStore

awsElasticBlockStore 卷将 Amazon Web Services（AWS）的 EBS 卷挂载到容器中。即便 Pod 被删除，awsElasticBlockStore 卷也只是被卸载，其内容会保留下来。

使用 awsElasticBlockStore 卷有一些限制：

- 运行 Pod 的节点必须是 AWS EC2 实例
- 这些实例需要与 EBS 卷位于相同的区域和可用区
- EBS 仅支持卷和 EC2 实例的一对一挂载

在 Pod 中使用 awsElasticBlockStore 卷之前，需要先创建它。可以使用如下的命令来创建：

```bash
aws ec2 create-volume --availability-zone=eu-west-1a --size=10 --volume-type=gp2
```

awsElasticBlockStore 卷配置示例：

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-ebs
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-ebs
      name: test-volume
  volumes:
  - name: test-volume
    awsElasticBlockStore:
      volumeID: <volume-id>
      fsType: ext4
```

### local

local 卷表示本地存储设备，如磁盘、分区或目录。本地卷只能用作静态创建的 PV，系统会通过查看 Pod 使用的 PV 的关联节点来得到节点约束，从而将 Pod 调度到正确的节点上。注意 local 卷受底层节点可用性影响。

以下是使用 local 卷的 PV 示例：

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: example-pv
  annotations:
        "volume.alpha.kubernetes.io/node-affinity": '{
            "requiredDuringSchedulingIgnoredDuringExecution": {
                "nodeSelectorTerms": [
                    { "matchExpressions": [
                        { "key": "kubernetes.io/hostname",
                          "operator": "In",
                          "values": ["example-node"]
                        }
                    ]}
                 ]}
              }'
spec:
    capacity:
      storage: 100Gi
    accessModes:
    - ReadWriteOnce
    persistentVolumeReclaimPolicy: Delete
    storageClassName: local-storage
    local:
      path: /mnt/disks/ssd1
```

## PersistentVolume（持久卷）和 PersistentVolumeClaim（持久卷声明）

### 介绍

卷子系统将如何提供存储的细节抽象了出来。为此，Kubernetes 引入了两个新的 API 资源，PersistentVolume（PV） 和 PersistentVolumeClaim（PVC）。

PV 是由管理员设置的存储，它是集群的一部分。就像 Node 是集群中的资源一样，PV 也是集群中的资源。PVC 是用户的存储需求，它与 Pod 相似。Pod 消耗 Node 资源，PVC 消耗 PV 资源。Pod 可以请求特定级别的计算资源（CPU 和内存），PVC 可以请求特定大小和访问模式的存储资源。

### PV 类型

各种类型的 PV 以插件形式来实现。Kubernetes 目前支持以下类型：

- GCEPersistentDisk
- AWSElasticBlockStore
- AzureFile
- AzureDisk
- FC (Fibre Channel)
- FlexVolume
- Flocker
- NFS
- iSCSI
- RBD (Ceph Block Device)
- CephFS
- Cinder (OpenStack block storage)
- Glusterfs
- VsphereVolume
- Quobyte Volumes
- HostPath
- VMware Photon
- Portworx Volumes
- ScaleIO Volumes
- StorageOS

### PV 配置

每个 PV 配置中都包含一个 sepc 规格字段和一个 status 状态字段。

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```

#### 容量（capacity）

通常，PV 将具有特定的存储容量，使用 PV 的 `capacity` 属性来设置。目前，存储大小是可以设置的唯一属性，未来可能支持 IOPS、吞吐量等。

#### 卷模式（volumeMode）

卷模式通过 `volumeMode` 属性来设置，除了文件系统，还支持原始块设备。volumeMode 的有效值可以是 Filesystem 或 Block，默认为 Filesystem。

#### 访问模式（accessModes）

PV 可以以资源提供者支持的任何方式挂载到主机上。每个供应商支持不同的功能，每个 PV 的访问模式只能被设置为供应商支持的模式。例如 NFS 支持多个客户端同时读/写，但特定的 NFS PV 可能是以只读方式挂载到服务器上。

访问模式包括：

- ReadWriteOnce 可以被单个节点以读/写模式挂载
- ReadOnlyMany 可以被多个节点以只读模式挂载
- ReadWriteMany 可以被多个节点以读/写模式挂载

在命令行中访问模式缩写为：

- RWO - ReadWriteOnce
- ROX - ReadOnlyMany
- RWX - ReadWriteMany

#### 类（storageClassName）

PV 可以具有一个类，通过 `storageClassName` 属性来设置。一个特定类别的 PV 只能绑定到请求该类别的 PVC。没有设置 storageClassName 属性的 PV 就没有类，它只能绑定到不需要特定类的 PVC。

#### 状态（status）

PV 可以处于以下某种状态：

- Available（可用） 空闲资源还没有被任何 PVC 绑定
- Bound（已绑定） 已经被 PVC 绑定
- Released（已释放） PVC 已被删除，但是 PV 还未被回收
- Failed（失败） 回收失败

### PVC 配置

每个 PVC 中都包含一个 spec 规格字段和一个 status 状态字段。

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
      - {key: environment, operator: In, values: [dev]}
```

#### 访问模式（accessModes）

在请求具有特定访问模式的存储时，PVC 使用与 PV 相同的设置。

#### 卷模式（volumeMode）

PVC 使用与 PV 相同的设置，将卷用作文件系统或块设备。

#### 资源（resources）

像 Pod 一样，PVC 可以请求特定数量的资源。

#### 选择器（selector）

PVC 可以指定一个标签选择器来进一步过滤 PV。只有标签与选择器匹配的 PV 可以绑定到声明。选择器由两个字段组成：

- matchLabels PV 必须有具有该标签值
- matchExpressions 一个匹配要求列表，通过指定标签名、操作符和值列表来判断是否满足某条规则。

所有来自 matchLabels 和 matchExpressions 的要求需要全部满足才算匹配。

#### 类（storageClassName）

PVC 只能绑定到具有相同类的 PV，没有指定类的只能绑定到同样没有指定类的 PVC。

### 在 Pod 中使用卷

Pod 通过 PVC 来使用卷。PVC 必须与 Pod 位于相同的命名空间。集群在 Pod 的命名空间中查找 PVC，并使用它来获取支持 PVC 的 PV，然后挂载到 Pod 里。

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: dockerfile/nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

## StatefulSet

StatefulSet 是一种控制器（Controller），类似于 Deployment，都是用来控制应用的运行，只不过 StatefulSet 控制的是更为复杂的有状态应用。StatefulSet 为应用的每个 Pod 实例提供唯一的标识，并且它可以保证部署和扩容的顺序。其应用场景包括：

- 稳定的持久化存储，即 Pod 重新调度后还是能访问到相同的持久化数据，基于 PVC 来实现
- 稳定的网络标志，即 Pod 重新调度后其 Pod Name 和 Hostname 不变，基于 Headless Service（没有 Cluster IP 的 Service，后续实验会细讲）来实现
- 有序部署，有序扩展，即 Pod 是有顺序的，在部署或者扩展的时候要依据定义的顺序依次进行（从 0 到 N-1，在下一个 Pod 运行之前所有之前的 Pod 必须都是 Running 和 Ready 状态）
- 有序收缩，有序删除（从 N-1 到 0）

从上面的应用场景可以发现，StatefulSet 包含如下组件：

- 用于定义网络标志的 Headless Service
- 用于创建 PV 的 volumeClaimTemplates
- 定义具体应用的 StatefulSet

StatefulSet 中每个 Pod 的域名格式为 `statefulSetName-{0..N-1}.serviceName.namespace.svc.cluster.local`。跟通常域名一样，子域在前，主域在后，只不过节数比较多而已。具体含义如下：

- statefulSetName 为 StatefulSet 的名字
- 0..N-1 为 Pod 的序号，从 0 开始到 N-1
- serviceName 为 Headless Service 的名字
- namespace 为服务所在的 namespace，Headless Service 和 StatefulSet 必须在相同的 namespace 中
- .svc.cluster.local 为 Cluster Domain

### 使用 StatefulSet

StatefulSet 适用于有以下某个或多个需求的应用：

- 稳定，唯一的网络标志。
- 稳定，持久化存储。
- 有序，优雅地部署和扩容。
- 有序，优雅地删除和终止。
- 有序，自动的滚动升级。

其中稳定是 Pod 调度中持久性的代名词。如果应用程序不需要稳定调度，则应该使用提供一组无状态副本的控制器来部署应用程序，例如 Deployment。通过下面的StatefulSet 配置示例可以看到 StatefulSet 中的组件：

```yaml
---
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
      app: nginx
  serviceName: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: gcr.io/google_containers/nginx-slim:0.8
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
      accessModes:
      - ReadWriteOnce
      storageClassName: local-storage
      resources:
        requests:
          storage: 10Mi
```

其中包含了以下组件：

- 一个名为 nginx 的 Headless Service，用于控制网络域。
- 一个名为 web 的 StatefulSet，它的 spec 中指定在了 2 个运行 nginx 容器的 Pod。
- volumeClaimTemplates 使用 local-storage 提供的存储资源来创建 PV。

### Pod 身份

StatefulSet Pod 具有唯一的身份，包括序号、稳定的网络 ID 和稳定的存储。身份绑定到 Pod 上，不管它（重新）调度到哪个节点上。

#### 序号

对于一个有 N 个副本的 StatefulSet，每个副本都会被指定一个整数序号，在 [0,N) 之间，且唯一。

#### 稳定的网络 ID

StatefulSet 中的每个 Pod 的主机名由 StatefulSet 的名称和 Pod 的序号构成。前面的示例会创建两个名为 web-0、web-1 的 Pod。StatefulSet 可以使用 Headless Service 来控制其 Pod 的域。在创建每个 Pod 时，它将获取一个匹配的 DNS 子域，形式为 $(Pod 名称).$(管理服务域)，其中管理服务域由 StatefulSet 上的 serviceName 字段定义。

#### 稳定存储

Kubernetes 为每个 volumeClaimTemplate 创建一个 PV。前面的 nginx 例子中，每个 Pod 将具有一个由 local-storage 存储类创建的 10MB 空间的 PV。当该 Pod（重新）调度到其它节点后，volumeMounts 将挂载之前的 PV。

### Pod 管理策略

#### OrderedReady Pod

StatefulSet 中默认使用的是这种策略。它实现了前面所述的行为。

#### Parallel Pod

Parallel Pod 策略告诉 StatefulSet 并行的启动和终止 Pod，在启动和终止其他 Pod 之前不会等待其它 Pod 变成运行/就绪或完全终止状态。

### 动手实验

下面我们通过实际运行一个有状态应用来实践一下前面学到的内容。

#### 创建 local PV

在创建 StatefulSet 之前我们先要准备好 PV 资源。在实验环境中我们只能创建 local（本地存储）类型的 PV。

首先，在 Kubernetes 里启用 local 类型的存储资源。将下面的配置保存到文件 `local-storage.yml`，然后执行 `kubectl create -f local-storage.yml` 即可启用。

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

```bash
$ kubectl create -f local-storage.yml
storageclass.storage.k8s.io "local-storage" created
```

然后，在每个 Node 上创建一个 PV。将下面的配置保存到文件 `local-pv.yml`，然后执行 `kubectl create -f local-pv.yml` 即可创建。

```yaml
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv1
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /tmp
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - kube-node-1
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv2
spec:
  capacity:
    storage: 1Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /tmp
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - kube-node-2
```

上面的配置会在 kube-node-1 和 kube-node-2 两个 Node 上分别创建一个 PV。其中，存储类 `spec.storageClassName` 用的是前面启用的 local-storage， `spec.capacity.storage` 指定了容量为 1G，挂载路径 `spec.local.path` 为 /tmp（该路径需要在 Node 上存在）。

```bash
$ kubectl create -f local-pv.yml
persistentvolume "local-pv1" created
persistentvolume "local-pv2" created
$ kubectl get pv
NAME        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM     STORAGECLASS    REASON    AGE
local-pv1   1Gi        RWO            Retain           Available             local-storage             13s
local-pv2   1Gi        RWO            Retain           Available             local-storage             13s
```

可以看到两个 PV 都已创建成功，其状态为可用（Available），底层资源提供者为 local-storage。

#### 创建 StatefulSet

将前面的示例文件保存为 `web.yml`，然后按照下面的命令来操作。

```bash
# 创建 StatefulSet
$ kubectl create -f web.yml
service "nginx" created
statefulset.apps "web" created

# 查看创建的 Headless Service 和 Statefulset
$ kubectl get service nginx
NAME      TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
nginx     ClusterIP   None         <none>        80/TCP    48s
$ kubectl get statefulset web
NAME      DESIRED   CURRENT   AGE
web       2         1         58s

# 根据 volumeClaimTemplates 自动创建的 PVC
$ kubectl get pvc
NAME        STATUS    VOLUME      CAPACITY   ACCESS MODES   STORAGECLASS    AGE
www-web-0   Bound     local-pv2   1Gi        RWO            local-storage   15s
www-web-1   Bound     local-pv1   1Gi        RWO            local-storage   10s

# 查看创建的 Pod，它们都是有序的
$ kubectl get pods
NAME      READY     STATUS    RESTARTS   AGE
web-0     1/1       Running   0          34s
web-1     1/1       Running   0          30s
```

## 实验总结

本次实验我们先学习了 Kubernetes 里有关存储的管理，包括各种存储对象，然后学习了如何使用 StatefulSet 来控制有状态应用的运行，最后实际运行了一个简单的有状态应用。