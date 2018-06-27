---
show: step
version: 1.0
enable_checker: true
---
# Kubernetes 介绍与部署

## 1. 实验介绍

从本次实验开始我们将陆续学习 [Kubernetes](https://kubernetes.io/)（简称 K8S）集群的部署、运维和管理。由于 Kubernetes 是一个非常庞大和复杂的系统，很难通过短短的几个实验顾及到方方面面，所以只能选择其中基础部分来讲解，更深入的使用大家可以在实际使用过程中去逐步了解。本次实验我们先来了解一下什么是 Kubernetes，以及如何快速部署一个 Kubernetes 集群。

> Kubernetes 相关实验部分内容和图片来源于网上，如有侵权请联系实验楼。

## 2. 简介

Kubernetes 最初源于谷歌内部的 Borg，Borg 是一个面向应用的容器集群部署和管理系统。Kubernetes 的目标旨在消除编排物理机和虚拟机计算、网络和存储基础设施管理这些负担，并使应用程序运营商和开发人员完全将重点放在以容器为中心的原语上来实现自动化运维。Kubernetes 还提供了稳定、兼容的基础平台，用于构建定制化的工作流程和更高级的自动化任务。 Kubernetes 同时具备完善的集群管理能力，包括多层次的安全防护和准入机制、多租户应用支撑能力、透明的服务注册和发现机制、内建负载均衡器、故障发现和自我修复能力、服务滚动升级和在线扩容、可扩展的资源自动调度机制、多粒度的资源配额管理能力。Kubernetes 还提供了完善的管理工具，涵盖开发、测试、运维、监控等各个环节。

## 3. 架构

#### 3.1. 系统架构

Kubernetes 集群里的节点分为 Master 和 Node 两种，其中 Master 负责管理集群，Node 负责运行应用容器。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5982timestamp1529486442885.png/wm)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5982timestamp1529486442256.png/wm)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5982timestamp1529486442576.png/wm)

Master 上运行的核心组件如下：

- API Server 资源操作入口，提供认证、授权、访问控制、API注册和发现等功能
- Scheduler 资源调度，按照预定的调度策略将 Pod（可认为是应用容器）调度到相应的机器上
- Controller 维护集群状态，比如故障检测、自动扩展、滚动更新等
- etcd 保存集群状态

Node 上运行的核心组件如下：

- Docker 容器引擎，负责镜像管理以及运行容器，也可使用其它容器运行时（Container Runtime）
- kubelet 管理容器生命周期，同时也负责存储卷和网络的管理
- kube-proxy 通过维护主机网络规则和连接转发来支持集群里的服务实现

除了核心组件，还有一些推荐的 Add-ons：

- kube-dns 为集群内部提供 DNS 服务
- Ingress Controller 为服务提供外网入口
- Heapster 资源监控
- Fluentd 日志采集
- Dashboard 图形管理界面
- Federation 管理多个集群

#### 3.2. 组件架构

下图是从组件角度看到的 Kubernetes 架构：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5982timestamp1529486481126.png/wm)

Kubernetes 作为一个平台，很多组件都是可以替换的，只要替换组件实现了相关接口就行。

- CNI Container Network Interface，容器网络接口
- CRI Container Runtime Interface，容器运行时接口
- CSI Container Storage Interface，容器存储接口
- OCI Open Container Initiative，开放容器标准，定义了容器跟底层 OS 之间的接口标准

## 4. 核心概念

Kubernetes 引入了下面这些核心概念，只有理解了这些核心概念才有可能真正理解和掌握 Kubernetes。

### 4.1. Pod

Kubernetes 有很多技术概念，同时也对应了很多资源对象，其中最重要也是最基础的是 Pod。Pod 是在 Kubernetes 集群中运行应用或服务的最小单元，它可以支持多容器，也就是一个 Pod 里包含多个容器。Pod 的设计理念是支持多个容器在一个 Pod 中共享网络地址和文件系统，可以通过进程间通信和文件共享这些简单高效的通信方式来组合成为服务。Pod 对多容器的支持是 Kubernetes 最基础的设计理念。比如要提供一个操作系统发行版的软件仓库服务，可以使用一个 Nginx 容器来发布软件，另一个容器用来从源仓库做同步，这两个容器的镜像不太可能是一个团队开发的，但它们一起工作才能提供一个服务。这种情况下，不同的团队各自开发构建自己的容器镜像，在部署的时候组合成为一个服务来对外提供服务。

Pod 是 Kubernetes 集群中所有业务类型的基础，可以看作运行在集群中的小机器人，不同类型的业务需要不同类型的小机器人去执行。目前 Kubernetes 中的业务主要分为长期伺服型（long-running）、批处理型（batch）、节点后台服务型（node-daemon）和有状态应用型（stateful application），分别对应的小机器人控制器为 Deployment、Job、DaemonSet 和 PetSet。

### 4.2. 副本控制器（Replication Controller，RC）

RC 是 Kubernetes 集群中最早的保证 Pod 高可用的资源对象。通过监控运行中的 Pod 来保证集群中运行指定数目的 Pod 副本。指定的数目可以是多个也可以是 1 个，少于指定数目，RC 会启动运行新的 Pod 副本，多于指定数目，RC 就会杀死多余的 Pod 副本。即使在指定数目为1的情况下，通过 RC 运行 Pod 也比直接运行 Pod 更明智，因为 RC 也可以发挥它高可用的能力，保证永远有 1 个 Pod 在运行。RC 是 Kubernetes 较早期的技术概念，只适用于长期伺服型的业务类型，比如控制小机器人提供高可用的 Web 服务。

### 4.3. 副本集（Replica Set，RS）

RS 是新一代 RC，提供同样的高可用能力，区别主要在于 RS 后来居上，能支持更多种类的匹配模式。副本集对象一般不单独使用，而是作为 Deployment 的理想状态参数使用。

### 4.4. 部署（Deployment）

部署表示用户对 Kubernetes 集群的一次更新操作。部署是一个比 RS 应用模式跟广泛的资源对象，可以是创建一个新的服务，更新一个服务，也可以是滚动升级一个服务。滚动升级一个服务，实际上是创建一个新的 RS，然后逐渐将新 RS 中副本数增加到理想状态，同时将旧 RS 中的副本数减小到 0 的复合操作。这样一个复合操作用一个 RS 是不太好描述的，所以用一个更通用的 Deployment 来描述。以 Kubernetes 的发展方向，未来对所有长期伺服型的的业务的管理，都会通过 Deployment 来管理。

### 4.5. 服务（Service）

RC、RS 和 Deployment 只是保证了支撑服务的 Pod 数量，但是没有解决如何访问服务的问题。一个 Pod 只是运行服务的一个实例，随时可能在一个节点上停止，在另一个节点以一个新的 IP 启动一个新的 Pod，因此不能以确定的 IP 和端口号来提供服务。要稳定地提供服务需要服务发现和负载均衡能力。服务发现完成的工作是针对客户端访问的服务，找到对应的的后端服务实例。在 Kubernetes 集群中，客户端需要访问的服务就是 Service 对象。每个 Service 会对应一个集群内部有效的虚拟 IP，集群内部通过虚拟 IP 访问服务。在 Kubernetes 集群中服务的负载均衡是由 kube-proxy 实现的。kube-proxy 是 Kubernetes 集群内部的负载均衡器。它是一个分布式代理服务器，在 Kubernetes 的每个节点上都运行了一个。这一设计体现了它的伸缩性优势，需要访问服务的节点越多，提供负载均衡能力的 kube-proxy 就越多，高可用节点也随之增多。与之相比，我们平时在服务器端使用反向代理做负载均衡，还要进一步解决反向代理的负载均衡和高可用问题。

### 4.6. 任务（Job）

Job 是 Kubernetes 用来控制批处理型任务的资源对象。批处理业务与长期伺服业务的主要区别是批处理业务的运行有头有尾，而长期伺服业务在用户不指示停止时会一直运行。Job 的 Pod 在把任务成功完成后就自动退出了。成功完成的标志根据自定义策略而不同，单 Pod 型任务有一个 Pod 成功就标志完成，定数成功型任务保证有 N 个任务全部成功，工作队列型任务根据应用确认的全局成功而标志成功。

### 4.7. 后台支撑服务集（DaemonSet）

长期伺服型和批处理型服务的核心在业务应用，可能有些节点运行多个同类业务的 Pod，有些节点上又没有这类 Pod 运行。而后台支撑型服务在 Kubernetes 集群中的指定节点群上每个节点都要运行一个此类 Pod。节点集群可能是所有集群节点，也可能是通过 nodeSelector 选定的一些特定节点。典型的后台支撑型服务包括存储、日志和监控等在每个节点上支持 Kubernetes 集群运行的服务。

### 4.8. 有状态服务集（PetSet）

在云原生应用的体系里，有两组近义词：第一组是无状态（stateless）、牲畜（cattle）、无名（nameless）、可丢弃（disposable），第二组是有状态（stateful）、宠物（pet）、有名（having name）、不可丢弃（non-disposable）。RC 和 RS 主要是控制提供无状态服务的，其控制的 Pod 的名字是随机设置的，一个 Pod 出故障了就被丢弃掉，在另一个节点重启一个新的 Pod，名字和启动在哪儿都不重要，重要的是 Pod 总数。而 PetSet 是用来控制有状态服务的，PetSet 中每个 Pod 的名字都是事先确定的，不能更改。PetSet 中 Pod 名字的作用是关联相关状态。

对于 RC 和 RS 中的 Pod，一般不挂载存储或者挂载共享存储，保存的是所有 Pod 共享的状态，Pod 像牲畜一样没有分别。而对于 PetSet 中的 Pod，每个 Pod 挂载自己独立的存储，如果一个 Pod 出现故障，从其他节点启动一个同样名字的 Pod，要挂载上原来 Pod 的存储以继续以之前的状态提供服务。

适合于 PetSet 的业务包括数据库服务 MySQL、PostgreSQL等，集群状态管理服务 Zookeeper、etcd 等有状态服务。PetSet 的另一种典型应用场景是作为一种比普通容器更稳定可靠的模拟虚拟机的机制。传统的虚拟机正是一种有状态的宠物，运维人员需要不断地维护它，容器刚开始流行时，我们用容器来模拟虚拟机使用，所有状态都保存在容器里，而这已被证明是非常不安全、不可靠的。使用 PetSet，Pod 仍然可以通过漂移到不同节点提供高可用，而存储也可以通过外挂的存储来提供高可靠性，PetSet 做的只是将确定的 Pod 与确定的存储关联起来保证状态的连续性。

### 4.9. 集群联邦（Federation）

在云计算环境中，服务的作用距离范围从近到远一般可以有：同主机（Host，Node）、跨主机同可用区（Available Zone）、跨可用区同地区（Region）、跨地区同服务商（Cloud Service Provider）、跨云平台。Kubernetes 的设计定位是单一集群在同一个地区内，因为同一个地区的网络性能才能满足 Kubernetes 的调度和计算存储连接要求。而集群联邦就是为提供跨 Region 跨服务商的 Kubernetes 集群服务而设计的。

每个 Federation 都有自己的分布式存储、API Server 和 Controller Manager。用户可以通过 Federation 的 API Server 注册该 Federation 的成员集群。当用户通过 Federation 的 API Server 创建、更改资源对象时，Federation API Server 会在自己所有注册的子集群里都创建一份对应的资源对象。在提供业务请求服务时，Federation 会先在自己的各个子集群之间做负载均衡，而对于发送到某个具体集群的业务请求，会依照这个集群独立提供服务时一样的调度模式去做集群内部的负载均衡。而集群之间的负载均衡是通过域名服务的负载均衡来实现的。

所有的设计都尽量不影响集群现有的工作机制，这样对于每个子集群来说，并不需要知晓外层还有一个 Federation，也就意味着所有现有的 Kubernetes 代码和机制不需要因为Federation 功能有任何变化。

### 4.10. 存储卷（Volume）

Kubernetes 集群中的存储卷跟 Docker 的存储卷类似，只不过 Docker 的存储卷作用范围为一个容器，而 Kubernetes 的存储卷的生命周期和作用范围是一个 Pod。每个 Pod 中声明的存储卷由 Pod 中的所有容器共享。Kubernetes 支持非常多的存储卷类型，特别的，支持多种公有云平台的存储，包括 AWS、Google 和 Azure 云，支持多种分布式存储包括 GlusterFS 和 Ceph，也支持较容易使用的主机本地目录 hostPath 和 NFS。Kubernetes 还支持使用 Persistent Volume Claim（PVC）这种逻辑存储，使用这种存储，使得存储的使用者可以忽略后台的实际存储技术，而将有关存储实际技术的配置交给存储管理员通过 Persistent Volume 来配置。

### 4.11. 持久存储卷（Persistent Volume，PV）和持久存储卷声明（Persistent Volume Claim，PVC）

PV 和 PVC 使得 Kubernetes 集群具备了存储的逻辑抽象能力，使得在配置 Pod 的逻辑里可以忽略对实际后台存储技术的配置，而把这项配置的工作交给 PV 的配置者，即集群管理者。存储的 PV 和 PVC 的这种关系，跟计算的 Node 和 Pod 的关系是非常类似的，PV 和 Node 都是资源的提供者，根据集群的基础设施变化而变化，由 Kubernetes 集群管理员配置，而 PVC 和 Pod 是资源的使用者，根据业务服务的需求变化而变化，由 Kubernetes 集群的使用者即服务的管理员来配置。

### 4.12. 节点（Node）

Kubernetes 集群中的计算能力由 Node 提供，最初 Node 称为服务节点 Minion，后来改名为 Node。Kubernetes 集群中的 Node 也就等同于 Mesos 集群中的 Slave 节点，是所有 Pod 运行所在的工作主机，可以是物理机也可以是虚拟机。不论是物理机还是虚拟机，工作主机的统一特征是上面要运行 kubelet 管理节点上运行的容器。

### 4.13. 密钥对象（Secret）

Secret 是用来保存和传递密码、密钥、认证凭证这些敏感信息的对象。使用 Secret 的好处是可以避免把敏感信息明文写在配置文件里。在 Kubernetes 集群中配置和使用服务不可避免地要用到各种敏感信息实现登录、认证等功能，例如访问 AWS 存储的用户名密码。为了避免将类似的敏感信息明文写在所有需要使用的配置文件中，可以将这些信息存入一个Secret 对象，而在配置文件中通过 Secret 对象引用这些敏感信息。这种方式的好处包括：意图明确，避免重复，减少暴漏机会。

### 4.14. 用户帐户（User Account）和服务帐户（Service Account）

顾名思义，用户帐户为人提供账户标识，而服务账户为计算机进程和 Kubernetes 集群中运行的 Pod 提供账户标识。用户帐户和服务帐户的一个区别是作用范围：用户帐户对应的是人的身份，人的身份与服务的 namespace 无关，所以用户账户是跨 namespace 的，而服务帐户对应的是一个运行中程序的身份，与特定 namespace 是相关的。

### 4.15. 命名空间（Namespace）

命名空间为 Kubernetes 集群提供虚拟的隔离作用，Kubernetes 集群初始有两个命名空间，分别是默认命名空间 default 和系统命名空间 kube-system，除此以外，管理员可以创建新的命名空间以满足需要。

### 4.16. RBAC（Role-based Access Control，RBAC）访问授权

相对于基于属性的访问控制（Attribute-based Access Control，ABAC），RBAC 主要是引入了角色（Role）和角色绑定（RoleBinding）的抽象概念。在 ABAC 中，Kubernetes 集群中的访问策略只能跟用户直接关联，而在 RBAC 中，访问策略可以跟某个角色关联，具体的用户再跟一个或多个角色相关联。显然，RBAC 像其他新功能一样，每次引入新功能，都会引入新的资源对象，从而引入新的概念抽象，而这一新的概念抽象一定会使集群服务管理和使用更容易扩展和重用。

## 5. 部署

Kubernetes 官方文档里提供了很多种 [部署方式](https://kubernetes.io/docs/setup/pick-right-solution/)，从源码编译，到本地开发环境部署，再到无需部署直接使用的在线服务，甚至可以只用几个命令就可以在各大 IaaS 平台上部署一个 Kubernetes 集群。本节部署我们重点关注本地开发环境部署，以便后续可以在此集群中进行试验。

从前面的介绍可以看到，Kubernetes 包含的组件比较多，完全手动在本地部署一个集群并不是一件容易的事情，好在有一些工具和脚本来简化这件事情。这些工具和脚本在安装过程中会下载 Kubernetes 各种组件的镜像，由于镜像仓库在国外，且集群启动过程中也会去请求一些信息。因此在本地部署之前请准备好代理，并且在集群部署和启动过程中设置系统全局代理，推荐使用 [Proxifier](https://www.proxifier.com/) 工具。Kubernetes 相关实验的实验环境我们提供的是大陆外的服务器，这样不用代理就可以正常部署和启动集群。考虑到连接速度原因，这些服务器只提供字符界面连接方式，图形方式的带宽要求比较高，体验不太好。如果不打算在本地实验，可跳过部署这部分内容，后面的实验我们会提供已经部署好 Kubernetes 集群的环境。

### 5.1. Minikube 方式

如果条件允许，推荐使用 [Minikube 方式](https://kubernetes.io/docs/getting-started-guides/minikube/)，部署简单且环境独立。Minikube 会在本地创建一个虚拟机，然后在这个虚拟机里面部署一个 Kubernetes 集群。如果你的“本地”本身已经是一个虚拟机，那么该虚拟机需要支持“虚拟机 in 虚拟机”技术，否则请使用 Kubeadm-dind 方式。

Minikube 方式部署的集群支持如下 Kubernetes 特性：

- DNS 集群内域名解析
- NodePorts 通过节点主机端口暴露其上容器服务
- ConfigMaps and Secrets 配置和密钥管理
- Dashboards Web 控制台
- Container Runtime 容器运行时，包括 Docker、rkt 和 CRI-O 几种
- Enabling CNI（Container Network Interface）启用容器网络接口
- Ingress 对外暴露集群内的服务

#### 5.1.1. 安装虚拟机软件

- 对于 macOS，可安装 [VirtualBox](https://www.virtualbox.org/wiki/Downloads)，[VMware Fusion](https://www.vmware.com/products/fusion) 或 [HyperKit](https://github.com/moby/hyperkit)
- 对于 Linux，可安装 [VirtualBox](https://www.virtualbox.org/wiki/Downloads) 或 [KVM](http://www.linux-kvm.org/)
- 对于 Windows，可安装 [VirtualBox](https://www.virtualbox.org/wiki/Downloads) 或 [Hyper-V](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install)

> 对于 Linux，Minikube 也支持通过 `–vm-driver=none` 选项支持在本地系统里运行 Kubernetes 各个组件，而不是在虚拟机里。这种方式只依赖 Docker，不需要安装虚拟机软件。

#### 5.1.2. 安装 kubectl

[kubectl](https://kubernetes.io/docs/user-guide/kubectl/) 是一个命令行工具，可以使用它来部署和管理 Kubernetes 集群。其安装方式也有很多种，这里只挑选几种典型的，完整的请参考 [官方文档](https://kubernetes.io/docs/tasks/tools/install-kubectl/)。

**Ubuntu 和 Debian**

```bash
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubectl
```

**CentOS 和 RHEL**

```bash
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
yum install -y kubectl
```

**macOS**

可通过 [Homebrew](https://brew.sh/) 工具来安装，如果没有该工具请先安装。

```bash
brew install kubectl
```

**Windows**

可通过 [Powershell Gallery](https://www.powershellgallery.com/) 工具来安装，如果没有该工具请先安装。

```
Install-Script -Name install-kubectl -Scope CurrentUser -Force
install-kubectl.ps1 [-DownloadLocation <path>]
```

如果 Downloadlocation 未指定，则会安装在用户的 `temp` 目录。配置文件存放在 `$HOME/.kube` 目录下。如果要更新重新运行 `install-kubectl.ps1` 脚本即可。

不管使用哪种安装方式，安装完成后请执行 `kubectl version` 来确认是否安装成功。

```bash
$ kubectl version
Client Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.1", GitCommit:"d4ab47518836c750f9949b9e0d387f20fb92260b", GitTreeState:"clean", BuildDate:"2018-04-12T14:26:04Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.1", GitCommit:"d4ab47518836c750f9949b9e0d387f20fb92260b", GitTreeState:"clean", BuildDate:"2018-04-12T14:14:26Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"linux/amd64"}
```

#### 5.1.3. 安装 Minikube

安装 Minikube 很简单，到 [官网](https://github.com/kubernetes/minikube/releases) 下载对应平台的安装包，按照一般软件的安装步骤执行即可。

#### 5.1.4. 部署

安装好 kubectl 和 Minikube 后，使用下面的命令来部署和启动一个 Kubernetes 集群。部署过程中会下载各个组件的镜像，由于镜像仓库在国外，请务必设置好系统全局代理，大约需要消耗 1~2 GB的流量。如果网速在 1MB/s 以上，部署过程大约 10~30 分钟可完成。

```bash
$ minikube start
Starting local Kubernetes cluster...
Running pre-create checks...
Creating machine...
Starting local Kubernetes cluster...
...
```

集群部署和启动成功后，执行下面的命令部署一个测试用的 echoserver 应用到集群里来测试集群的正确性。

```bash
$ kubectl run hello-minikube --image=k8s.gcr.io/echoserver:1.4 --port=8080
deployment "hello-minikube" created
$ kubectl expose deployment hello-minikube --type=NodePort
service "hello-minikube" exposed
$ kubectl get pod
NAME                              READY     STATUS              RESTARTS   AGE
hello-minikube-3383150820-vctvh   0/1       ContainerCreating   0          3s
$ kubectl get pod
NAME                              READY     STATUS    RESTARTS   AGE
hello-minikube-3383150820-vctvh   1/1       Running   0          13s
$ curl $(minikube service hello-minikube --url)
CLIENT VALUES:
client_address=192.168.99.1
command=GET
real path=/
...
```

在命令行输入 `minikube dashboard` 来打开 Kubernetes Web 控制台。在 Web 控制台里可查看集群状态和执行管理操作。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid606277labid5982timestamp1529486551749.png/wm)

测试没问题，删除服务和应用，并停止集群。

```bash
$ kubectl delete services hello-minikube
service "hello-minikube" deleted
$ kubectl delete deployment hello-minikube
deployment "hello-minikube" deleted
$ minikube stop
Stopping local Kubernetes cluster...
Stopping "minikube"...
```

### 5.2. Kubeadm-dind 方式

[Kubeadm-dind 方式](https://github.com/Mirantis/kubeadm-dind-cluster) 适合在虚拟机或云主机环境里部署 Kubernetes 集群，这些环境里不支持 Minikube 方式。这种方式没有 Minikube 方式成熟和完善，所以有条件的话尽量使用 Minikube 方式。DIND 代表 Docker in Docker，因为这种方式部署的集群里节点都是通过 Docker 容器模拟的，而部署到集群里的应用又以 Docker 容器运行在节点容器里。目前支持 Kubernetes 1.8.x，1.9.x 和 1.10.x 几个版本，下面我们以最新的 1.10.x 为例。

Kubeadm-dind 提供了一个预先配置好的脚本来部署集群，一键就可以完成集群的部署。请同样确保本地访问国外网络的畅通性。

```bash
wget https://cdn.rawgit.com/Mirantis/kubeadm-dind-cluster/master/fixed/dind-cluster-v1.10.sh
chmod +x dind-cluster-v1.10.sh
./dind-cluster-v1.10.sh up
```

如果本地先前没有安装 kubectl，可把 Kubeadm-dind 自带的 kubectl 命令添加到系统路径下。然后执行 `kubectl get nodes` 来查看集群里的所有节点。

```bash
export PATH="$HOME/.kubeadm-dind-cluster:$PATH"
kubectl get nodes
```

运行过程中可再次执行 `up` 指令来重启集群，执行 `down` 指令来停止集群，执行 `clean` 来销毁集群。

```bash
./dind-cluster-v1.10.sh up
./dind-cluster-v1.10.sh down
./dind-cluster-v1.10.sh clean
```

## 6. 实验知识点

- Kubernetes 简介和架构
- 采用 Minikube 方式来部署集群
- 采用 Kubeadm-dind 方式来部署集群

## 7. 实验总结

本地实验我们了解了什么是 Kubernetes，以及 Kubernetes 集群里包含的核心组件，并学会了如何在本地部署一个集群。
