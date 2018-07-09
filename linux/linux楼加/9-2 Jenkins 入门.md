# Jenkins 入门

## 实验介绍

通过前面的实验了解到了什么是持续集成，以及它的重要性。那么如何在开发流程中去实现持续集成了？这需要借助于持续集成工具。Jenkins 作为持续集成工具的领头羊，我们肯定需要去了解它。

## 实验知识点

- Jenkins 介绍
- 安装 Jenkins

## Jenkins 介绍

[Jenkins](https://jenkins.io/) 是一个独立的开源自动化服务器，可以用来自动化一些任务的执行，例如构建、测试和部署软件等。Jenkins 可以通过本地系统包或者 Docker 来安装，可以运行在任意安装了 Java 环境的机器上。

Jenkins 来源于 [Hudson](http://hudson-ci.org/)。在 2009 年，Oracle 收购了 Sun 并继承了 Hudson 的基础代码。在 2011 年，Oracle 和开源社区关系紧张，Hudson 分化成了两个项目：

- Jenkins：被大多数 Hudson 开发者运营
- Hudson：被 Oracle 控制

Jenkins 在持续集成领域市场份额中居于主导地位，被各种大小规模的团队用于用各种语言的项目中，包括 .NET、Java、Ruby、Groovy、Grails、PHP 等。选择 Jenkins 的理由如下：

- 易于使用 Jenkins 的用户界面简单、直观、友好，发布工作人员只需要通过简单的 UI 操作就可以替代原来繁琐的发布工作。
- 拥有良好的扩展性 提供数以百计的开源插件可供使用，而且几乎每周会有新的开源插件贡献进来，这些插件的安装都十分快捷和简单。
- 发展良好 Jenkins 开源社区的规模变得越来越大、活跃度也变得越来越高，发展速度非常快。

使用 Jenkins 可带来如下价值：

- 减少发布工作人员的大量日常工作量，大大提高项目的发布效率。
- 不容易出错，降低人工发布带来的风险。
- 可 24 小时随时发布。
- 方便紧急修复或回滚操作 Rollback。
- 方便对发布流程进行控制、标准化。
- 方便发布统计、历史版本可追溯。

## Jenkins 词汇表

### Agent

代理通常是一台机器或容器，它连接到 Jenkins Master，并在 Master 指示下执行任务。

### Artifact

一个在构建或 Pipeline 运行期间生成的不可变文件，该文件存档到 Jenkins Master 中供用户后续使用。

### Build

项目单次执行结果

### Cloud

提供动态代理生成和分配的系统配置，例如由 Azure VM Agents 或 Amazon EC2 插件提供的系统配置。

### Core

主要的 Jenkins 应用程序（jenkins.war），提供了可以构建插件的基本环境，包括 Web UI、配置和基础。

### Downstream

一个配置好的 Pipeline 或 Project，作为另一个 Pipeline 或 Project 执行的一部分而被触发。

### Executor

执行节点上的 Pipeline 或 Project 的槽。一个节点可以配置零个或多个 Executor，具体多少跟该节点上能够并行执行的 Pipeline 或 Project 数量有关。

### Fingerprint

散列是全局唯一的，用于跟踪跨多个 Pipeline 或 Project 的 Artifact 或其他实体的使用情况。

### Folder

Pipeline 或 Project 的组织容器，类似于文件系统上的文件夹。

### Item

Web UI 中的实体，可以是 Folder、Pipeline 或 Project。

### Job

不再使用的术语，与 Project 同义。

### Label

用户定义的用于分组代理的文本，通常按类似功能或能力来分组。例如，对于基于 Linux 的代理打上 linux 标签，对具有 Docker 功能的代理打上 docker 标签。

### Master

处于中央，进行协调的进程，负责存储配置、加载插件以及生成各种用户界面。

### Node

作为 Jenkins 环境的一部分，用于执行 Pipeline 或 Project 的机器。无论 Master，还是 Agent 都可认为是 Node。

### Project

用户定义的由 Jenkins 来执行的定义，例如构建软件等。

### Pipeline

用户定义的连续执行一系列任务的流程。

### Plugin

功能的扩展，与核心分开提供。

### Publisher

构建的一部分，在所有步骤完成后执行，比如发布报告、发送通知等。

### Stage

Stage 用于定义 Pipeline 里的各个阶段。

### Step

Step 用于定义 Pipeline 或 Project 里的具体操作。

### Trigger

触发运行 Pipeline 的条件。

### Update Center

管理插件和插件元数据，以便在 Jenkins 中安装插件。

### Upstream

一个配置好的 Pipeline 或 Project，它的执行会触发其它 Pipeline 或 Project 执行。

### Workspace

节点上的 Pipeline 或 Project 运行时的工作目录。除非在 Master 上设置了特定的 Workspace 清理策略，否则 Workspace 在 Pipeline 或 Project 运行结束后会保留。

## 安装 Jenkins

Jenkins 的安装比较简单，其依赖只有一个 Java 运行时。具体安装过程如下：

1\. 在实验环境中可以通过 `apt` 来完成，首先添加相应的源。

```bash
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
```

2\. 除此之外，还需要安装 Java 运行时。

> 使用 `java -version` 来确认是否已有 Java，如有可跳过此步骤

```bash
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
```

3\. 更新索引库并安装。

```bash
sudo apt-get update
sudo apt-get install jenkins
```

4\. 启动服务。

```bash
sudo service jenkins start
```

5\. 服务启动起来之后，就可以打开浏览器访问 `http://localhost:8080`，页面会提示输入自动生成的密码。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517800624647.png/wm)

6\. 到提示的密码保存文件中查找密码。

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517800732741.png/wm)

7\. 将获得的密码填入浏览器中的输入框中，点击 `Continue` 继续。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517453845217.png/wm)

8\. 输入密码后会弹出一个窗口，提示安装插件，选择建议安装即可。

9\. 插件安装完成之后，需要创建一个管理员用户。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517454971463.png/wm)

10\. 安装完成，可以开始使用 Jenkins 了。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517455022891.png/wm)

Jenkins 安装操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week11/6-1.mp4
@`

## 实验总结

通过本次实验的学习我们对 Jenkins 有了基本的了解，也学会了如何安装它，接下来将学习如何使用它。