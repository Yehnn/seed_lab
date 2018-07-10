---
show: step
version: 1.0
enable_checker: true
---
# Keepalived 原理与工作机制

## 一、实验介绍

#### 1.1 内容简介

Keepalived 是 Linux 下的一个轻量级的高可用集群软件，可用于实现网络或服务器的高可用性。Keepalived 通过虚拟路由冗余协议（VRRP）实现高可用功能，它的部署和使用非常简单。

Keepalived 也是 LVS 的扩展项目，因此 LVS 与 Keepalived 之间具备良好的兼容性。

本节实验内容将会介绍 Keepalived 的相关理论知识，包括它的结构体系与工作原理，VRRP 协议介绍。

#### 1.2 实验知识点

* Keepalived 功能了解
* 虚拟路由冗余协议（VRRP）讲解
* Keepalived 体系结构
* Keepalived 集群监控

#### 1.3 推荐阅读

本文部分内容参考[ Keepalived 官方网站](http://keepalived.org/doc/introduction.html) 提供的使用文档。

## 二、实验内容

下面介绍实验基础知识。

### 2.1 Keepalived 用途

Keepalived 是一款优秀的高可用软件。最初是为 LVS 设计的，用来监控集群系统中各个服务节点的运行状态。Keepalived 是一个类似于 Layer3，Layer4，Layer5 交换机制的软件，工作在 TCP/IP 参考模型的 **第3层（网络层）、第4层（传输层）和第5层（应用层）**。它的作用是检测每个服务器节点的状态，如果其中某一个服务节点出现故障，Keepalived 将会检测到此故障节点，并将它从集群系统中删除。当故障节点恢复正常后，Keepalived 又会自动将此节点加入集群系统中。以上过程，不需要人工干预，需要人工操作的只是将故障节点修复正常即可。

Keepalived 一般用于负载均衡层，防止负载调度器的单点故障，实现负载均衡层的高可用特性。

### 2.2 VRRP  协议

Keepalived 主要通过虚拟路由冗余协议（VRRP）来实现高可用高能。

在现实网络环境中，不同网段主机之间的通信通过路由器完成，如果主机之间的路由器出现故障，主机间将会无法通信，此时，路由器的故障称为一个单点故障。VRRP 协议就是解决此类问题的。

VRRP（Virtual Router Redundancy Protocol）是由 IETF 提出的解决局域网中网关出现单点失效现象的路由协议。使得在发生故障而进行设备功能切换时可以不影响内外数据通信，不需要再修改内部网络的网络参数。VRRP 协议需要具有 IP 地址备份，优先路由选择，减少不必要的路由器间通信等功能。

在学习 VRRP 协议之前，需要弄清楚两个概念：物理路由器与虚拟路由器。物理路由器是具有路由功能的设备，可以是常见的路由器，也可以是一台服务器。虚拟路由器是一个抽象的概念。

VRRP 将局域网的一组物理路由器组织成一个虚拟路由器，称之为一个备份组，这个虚拟路由器通过虚拟 IP 地址对外提供服务。在虚拟路由器内部，多个物理路由器协同工作，同一时间内，只有一个物理路由器对外提供服务，它的角色为 **MASTER**，称为**主路由**，主路由是由选举算法决定的，它拥有对外服务的虚拟 IP 地址（VIP）。其他物理路由器处于备份状态，不对外提供服务，他们的角色为 **BACKUP**，称为**备份路由器**。

**VRRP 结构与工作流程图：**

![](https://dn-anything-about-doc.qbox.me/document-uid113508labid1timestamp1473319225612.png/wm)

**工作流程如下：**

* VRRP 备份组中的路由器会根据优先级选举出主路由 MASTER，MASTER 路由器通过发送报文，将虚拟 MAC 地址通知给与它相连的设备或主机，从而承担报文转发任务。
* MASTER 主路由会周期性的向备份组内所有备份路由器 BACKUP 发送 VRRP 通告报文，备份路由器根据报文信息监控 MASTER 的运行状态。
* 如果 MASTER 出现故障，BACKUP 在 3 秒内不能收到 MASTER 发送的 VRRP 通告报文，其他 BACKUP 会重新进行选举，其中优先级最高的 BACKUP 将成为新的 MASTER。
* VRRP 备份组进行状态切换时，新的 MASTER 会立即发送带有虚拟路由器的 MAC 地址和虚拟 IP 地址信息的报文，更新与它相连的设备中 ARP 表。从而将用户流量导入到新的 MASTER 路由器。整个切换过程非常快，对用户完全透明，保证了服务的持续可用性。
* 当原 MASTER 故障恢复后，在收到现 MASTER 发来的 VRRP 数据包时，从中解析出它的优先级，如果自己的优先级较高，则通知当前 MASTER，进行角色切换，现 MASTER 降级为 BACKUP，自己升级为 MASTER。如果两者优先级相同，则比较两者的 IP 地址，IP 地址越大，优先级越高。
* 当 BACKUP 路由器的优先级高于 MASTER 主路由时，主要由 BACKUP 路由器的工作方式（抢占或非抢占）来决定是否重新选举 MASTER 主路由。

当我们了解了 VRRP 的工作原理之后，我们会发现其实它与 LVS 的工作原理似乎差不多，都是虚拟出一个 IP 对外提供服务，然后自己在内部通过某种机制来进行切换，这应该便是为什么它们有这么高的契合度的原因.

所以我们换着方式让两个工具相互结合，让 keepalived 工作在 Load Balancer 上，做好出口路由的高可用，避免 LVS 负载均衡端的单点隐患，这样大大的提高了我们的服务质量，特别是在突然的大流量冲击下。

### 2.3 Keepalived 体系结构

**Keepalived 体系结构图：**

![](https://dn-anything-about-doc.qbox.me/document-uid113508labid1timestamp1473316871034.png/wm)

Keepalived 主要分为两层结构：**用户空间层**，**内核空间层**。

内核空间层位于最底层，包括了 IPVS 和 NETLINK 两个内核模块。

通过 IPVS 模块可以实现基于 IP 的负载均衡集群，LVS 集群就是主要通过 IPVS 模块来实现负载均衡。

NETLINK 模块主要提供一些高级路由及其一些相关的网络功能，完成用户空间层  Netlink Reflector 模块发来的各种网络请求。

在用户空间层，Keepalived 可以分为四个部分，分别是：**Scheduler I/O Multiplexer**、**Memory Mangement**、**Control Plane**  和 **Core components**。

* Scheduler I/O Multiplexer：一个 I/O 复用分发调度器，它负责安排 keepalived 所有内部的任务请求
* Memory Management：一个内存管理机制，这个框架提供的访问内存的一些通用方法。
* Control Plane 是 keepalived 的控制面板，可以实现对配置文件进行编译和解析，keepalived的配置文件解析比较特殊，它只有在用到某模块时才解析相应的配置。
* Core components 是 keepalived 要核心组件，包含了一系列的功能模块。

 **Core components** 部分主要包括以下一些功能模块：

* WatchDog：监控 checkers 和 VRRP 进程的状况；
* Checkers：真实服务器的健康检查 health checking，是 keepalived 最主要的功能；
* VRRP Stack：负载均衡器之间的切换；
* IPVS wrapper:设定规则到内核 ipvs 的接口；
* Netlink Reflector：设定 vrrp 的 vip 地址等路由相关的功能。

### 2.4 Keepalived 集群监控

通过 VRRP 协议，Keepalived 可以实现负载均衡服务器的高可用特性。同时，Keepalived 也可以实现对集群中服务器运行状态的监控和故障隔离。前面提到过，Keepalived 工作在 TCP/IP 参考模型的 **第3层（网络层）、第4层（传输层）和第5层（应用层）**，我们将分别从这几个层次来讲解 Keepalived 是如何对服务器进行状态监控与检测。

- **网络层**

  Keepalived 在网络层采用互联网控制消息协议（ICMP）向服务器集群中每个节点定时发送 ICMP 数据包，类似于 ping 命令的作用。如果某个节点没有返回响应数据包，则 Keepalived 认为此节点发生故障，并从服务器集群中删除该节点。

- **传输层**

  在传输层中，Keepalived 通过 TCP 的端口连接和扫描技术定期查看集群中的某个节点端口是否开放。如果 Keepalived 在传输层检测到某个节点的指定端口没有数据响应，则判断服务器节点的端口存在异常情况，将此节点从服务器集群中删除。

- **应用层**

  在应用层中，Keepalived 会根据用户设定检测服务器的相关服务和程序是否正常运行。若检测结果与用户设定不相符，则判断定对应的服务器节点为异常情况，将它从服务器集群中删除。

## 三、实验总结

本次实验中，我们对高可用集群软件 Keepalived 进行了深入学习。了解了什么是 VRRP 协议，以及它的工作流程；了解了 Keepalived 的体系结构，以及 Keepalived 如何对后端服务器集群做状态监控与故障隔离。

实验文档中，只是对 Keepalived 的工作原理和流程做了比较深入的理论介绍，远没有达到详细讲解的程度，如果你对 Keepalived 更多技术细节有兴趣，可以查看 Keepalived 官网的详细文档说明和查阅网络资料。

本文介绍内容，是比较基础也是比较重要的内容，掌握这些内容之后会对后续实验 Keepalived 的实际应用有很大帮助。