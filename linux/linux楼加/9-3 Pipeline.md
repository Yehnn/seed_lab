# Pipeline

## 实验介绍

Pipeline 是 Jenkins 里最核心的概念，Jenkins 里持续集成都是以 Pipeline 来组织的。

## 实验知识点

- Pipeline 简介
- Jenkinsfile 基本语法
- 通过经典 UI 来定义 Pipeline

## 简介

### Pipeline 是什么？

Pipeline 是一套插件，支持在 Jenkins 中实施持续交付。

一个持续交付 Pipeline 是对软件获取过程的自动化表达，从源代码直通到用户和客户需要的软件。软件的每一次改变（在源代码仓库中提交）都会在发布过程中经历一个复杂的过程。这个过程包括以可靠和可重复的方式构建软件，以及通过测试和部署等阶段来推进软件的构建。

Pipeline 提供了一套可扩展的工具，通过使用 Pipeline DSL（领域特定语言）来对从简单到复杂的持续交付过程进行代码建模。

Pipeline 的定义被写入到一个文本文件（Jenkinsfile）中，该文件可以提交到项目的源代码仓库。这是 Pipeline-as-code 的基础，将持续交付 Pipeline 作为应用程序的一部分进行版本控制，并像任何其他代码一样进行审查。

创建 Jenkinsfile 并提交到源代码仓库提供了以下好处：

- 为所有分支和请求自动创建一个 Pipeline 构建过程。
- Pipeline 上的代码审查/迭代（以及源代码）。
- 审核追踪 Pipeline 。
- Pipeline 的单一真实来源，可由项目的多个成员查看和编辑。

可以通过 Web UI 或编写 Jenkinsfile 来定义 Pipeline，虽然它们使用的语法相同，但最佳实践还是编写 Jenkinsfile 并提交到源代码仓库。

### 声明式（Declarative）与脚本式（Scripted）Pipeline 语法

Jenkinsfile 可以使用两种语法来编写，声明式和脚本式。

声明式和脚本式 Pipeline 的构造完全不同。声明式 Pipeline 是 Jenkins 在脚本式后提供的新功能。相比脚本式它有如下优点：

- 提供了比脚本式 Pipeline 更丰富的语法功能
- 写和读 Pipeline 代码更容易

虽然两者构造完全不同，但许多语法组件对声明式和脚本式是通用的。

### 为什么要 Pipeline？

Jenkins 从根本上说是一个支持多种自动化模式的自动化引擎。Pipeline 在Jenkins 上增加了一套强大的自动化工具，支持从简单的持续集成到全面的持续交付等使用场景。Pipeline 提供了以下功能：

- 代码：Pipeline 是在代码中实现的，并且通常会作为源代码来管理，从而使团队能够编辑、审阅和迭代 Pipeline。
- 耐久性：Pipeline 可以在 Jenkins 计划内和计划外重启情况下保留下来。
- 可暂停性：在继续进行 Pipeline 运行之前， Pipeline 可以选择停止并等待用户输入或批准。
- 多用途：Pipeline 支持复杂的现实持续交付需求，包括分叉/连接，循环和并行执行。
- 可扩展性：Pipeline 插件支持定制扩展，以及与其他插件集成。

虽然 Jenkins 一直支持简单的将各种工作串联起来完成顺序性任务，但 Pipeline 使这一概念成为 Jenkins 的一等公民。基于 Jenkins 的可扩展性核心价值，Pipeline 也支持扩展。

下面的流程图是一个在 Pipeline 中能轻松建模出来的持续交付场景示例：

![](./images/jenkins-pipeline-flow.png)

### Pipeline 概念

以下概念是理解 Pipeline 的关键，它与 Pipeline 语法紧密相关。

#### Pipeline

Pipeline 是持续交付的用户定义模型。Pipeline 的代码定义了整个构建过程，通常包括构建应用程序、测试和交付应用程序等阶段。`pipeline` 块是声明式 Pipeline 语法的核心部分。

#### Node

Node 是属于 Jenkins 环境并且能够执行 Pipeline 的机器。`node` 块是脚本式 Pipeline 语法的关键部分。

#### Stage

`stage` 块代表 Pipeline 的一个阶段。

#### Step

`step` 表示一个任务。从根本上说，一个 step 告诉 Jenkins 在特定的时间做什么。例如要执行 shell 命令 `make`，可以使用 `sh` step：`sh 'make'`。

### Pipeline 语法概述

下面的 Pipeline 代码框架说明了声明式 Pipeline 语法和脚本式 Pipeline 语法之间的基本区别。`stages` 和 `steps` 在声明式和脚本式语法里都是常见元素。

#### 声明式 Pipeline

在声明式 Pipeline 语法中，`pipeline` 块定义了整个 Pipeline 要做的所有工作。

```groovy
pipeline {
    // 在任何可用的代理上执行此 Pipeline 或其任何阶段。
    agent any
    stages {
        // 定义“构建”阶段。
        stage('Build') {
            steps {
                // 执行一些与“构建”阶段相关的步骤。
            }
        }
        // 定义“测试”阶段。
        stage('Test') {
            steps {
                // 执行与“测试”阶段相关的一些步骤。
            }
        }
        // 定义“部署”阶段。
        stage('Deploy') {
            steps {
                // 执行与“部署”阶段相关的一些步骤。
            }
        }
    }
}
```

#### 脚本式 Pipeline

在脚本式 Pipeline 语法中，一个或多个 `node` 块在整个 Pipeline 中执行核心工作。虽然这不是脚本式 Pipeline 语法的强制性要求，但将 Pipeline 的工作限制在 `node` 块中会带来以下好处：

- 通过添加项目到 Jenkins 队列来安排块中包含的 `steps`。只要执行器在 Node 上空闲，这些 `steps` 就会运行。
- 创建一个工作空间（特定于该 Pipeline 的目录），在此空间中可以对获取到的源代码执行工作。

```groovy
// 在任何可用的代理上执行此 Pipeline 或其任何阶段。
node {  
    // 定义“构建”阶段。stage 块在脚本式 Pipeline 语法中是可选的。但使用 stage 块可以更清晰地在 Jenkins UI 中显示 stage 的每个子任务/步骤。
    stage('Build') {
        // 执行一些与“构建”阶段相关的步骤。
    }
    // 定义“测试”阶段。
    stage('Test') {
        // 执行与“测试”阶段相关的一些步骤。
    }
    // 定义“部署”阶段。
    stage('Deploy') {
        // 执行与“部署”阶段相关的一些步骤。
    }
}
```

#### Pipeline 示例

下面的 Jenkinsfile 示例，采用声明式 Pipeline 语法：

```groovy
// pipeline 是声明式 Pipeline 特定语法，它定义了一个包含执行整个 Pipeline 的所有内容和指令的“块”。
pipeline {
    // agent 是声明式 Pipeline 特定的语法，它指示 Jenkins 为整个 Pipeline 分配执行器和工作空间。
    agent any
    stages {
        // stage 是描述 Pipeline 阶段的语法块。
        stage('Build') {
            // steps 是声明式 Pipeline 特定语法，用于描述要在此 stage 中运行的 steps。
            steps {
                // sh 是一个 Pipeline step（由 Pipeline: Nodes and Processes 插件提供），用于执行 shell 命令。
                sh 'make'
            }
        }
        stage('Test') {
            steps {
                sh 'make check'
                // junit 是另一个 Pipeline step（由 JUnit 插件提供），用于汇总测试报告。
                junit 'reports/**/*.xml'
            }
        }
        stage('Deploy') {
            steps {
                sh 'make publish'
            }
        }
    }
}
```

换成脚本式 Pipeline 语法的写法如下：

```groovy
// node 是脚本式 Pipeline 的特定语法，指示 Jenkins 在任何可用的 agent/node 上执行此 Pipeline（以及其中包含的任何 stage）。
node {
    stage('Build') {
        sh 'make'
    }
    stage('Test') {
        sh 'make check'
        junit 'reports/**/*.xml'
    }
    stage('Deploy') {
        sh 'make publish'
    }
}
```

## 通过经典 UI 定义 Pipeline

Pipeline 可以通过以下方式之一来创建：

- 通过 Blue Ocean：在 Blue Ocean 中建立一个 Pipeline 项目后，Blue Ocean 界面可以帮助你编写 Pipeline Jenkinsfile，并将其提交到源代码仓库。
- 通过经典 UI：直接在 Jenkins 中创建基本的 Pipeline。
- 源代码仓库：可以手动编写 Jenkinsfile，然后将其提交到项目的源代码仓库。

本次实验我们先学习最简单的“经典 UI”方式，后面会有单独的实验来讲解“Blue Ocean”方式和“源代码仓库”方式。

通过经典 UI 创建的 Jenkinsfile 存储在 Jenkins 的主目录下。具体创建步骤如下：

1\. 登录 Jenkins，默认访问地址为 `http://localhost:8080/`。

2\. 在 Jenkins 主页中，单击左上角的“新建任务”。

![](./images/jenkins-home.png)

3\. 选择“流水线”，输入任务名称，然后确定。

![](./images/jenkins-new-item.png)

4\. 在 Pipeline 创建页面，向下滚动到“流水线”部分。

![](./images/jenkins-new-pipeline.png)

确保“定义”选中的是“Pipeline script”，然后输入下面的内容：

```groovy
pipeline {
    agent any
    stages {
        stage('Stage 1') {
            steps {
                echo 'Hello world!'
            }
        }
    }
}
```

5\. 保存后将打开 Pipeline 详情页面，单击左侧的“立即构建”以运行 Pipeline。

![](./images/jenkins-pipeline-detail.png)

6\. 构建结果如下，在左侧的构建历史记录下，单击 ＃1 以访问此次 Pipeline 运行的详细信息。

![](./images/jenkins-pipeline-build-result.png)

7\. 点击左侧的“控制台输出”来查看运行过程中的日志输出。

![](./images/jenkins-pipeline-build-detail.png)

8\. 运行过程的日志输出如下。

![](./images/jenkins-pipeline-build-console.png)

通过经典 UI 定义 Pipeline 可以方便的测试 Pipeline 代码片段，或者用于处理不需要提交到源代码仓库的简单 Pipeline。

## 实验总结

本次实验我们学习了 Pipeline 的概念及其重要性，以及如何通过经典 UI 来定义 Pipeline。