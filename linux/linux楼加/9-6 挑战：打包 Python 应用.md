# 挑战：打包 Python 应用

## 介绍

本次挑战将使用 Jenkins 来打包一个简单的 Python 计算器应用。应用代码下载地址 [jenkins-python-app](http://labfile.oss.aliyuncs.com/courses/980/09/assets/jenkins-python-app.tar.gz)。

打包出来的可执行程序在 Linux 下的运行效果如下：

![](./images/add2vals.png)

## 目标

- 创建一个 Jenkins Pipeline，Pipeline 里需包含构建（Build）、测试（Test）和分发（Delivery）三个阶段
- 为了避免 Pipeline 对节点运行环境的依赖，请使用 Docker 容器来运行 Pipeline 的各个阶段
- 运行 Pipeline 来生成工件（打包好的应用程序），下载该工件并保存到路径 `$HOME/add2vals`

## 提示语

### 创建 GitHub 仓库

在个人 GitHub 账号下新建一个仓库 `jenkins-python-app`，将应用代码提交到该仓库。

### 编写 Jenkinsfile 并提交到仓库

Jenkinsfile 放在仓库根目录下即可。Jenkinsfile 里需配置 `agent` 为 Docker 容器。由于每个阶段使用的镜像不一样，因此需要在每个阶段下单独配置 agent，同时需要把全局的 `agent` 设为 none（不需要全局运行环境）。关于如何配置 agent 为容器，可参考官方文档 [Pipeline Syntax](https://jenkins.io/doc/book/pipeline/syntax/#agent)。

Build、Test 和 Deliver 三个阶段使用的 Docker 镜像分别为 `python:2-alpine`、`qnib/pytest` 和 `cdrx/pyinstaller-linux:python2`。每个阶段下都包含一个 `sh` 步骤，该执行的 shell 命令分别为 `python -m py_compile sources/add2vals.py sources/calc.py`、`py.test --verbose --junit-xml test-reports/results.xml sources/test_calc.py` 和 `pyinstaller --onefile sources/add2vals.py`。

在 Test 阶段，无论前面步骤执行是否成功(`post` 状态为 `always`)，都使用 JUnit 插件来检查单元测试结果（`junit 'test-reports/results.xml'`）。

在最后一个 Deliver 阶段，还需要使用 `archiveArtifacts` 指令来在 Pipeline 执行成功（`post` 状态为 `success`）时生成工件，也就是打包应用。

### 创建 Pipeline 项目

使用“经典 UI”方式即可，不需要通过“Blue Ocean”的可视化编辑器来定义 Pipeline。注意，进行到定义 Pipeline 步骤时，选择“Pipeline script from SCM”方式，以便告诉 Jenkins 从 GitHub 仓库里获取定义。

### 运行 Pipeline

Pipeline 运行成功后，会生成工件（可执行程序 `add2vals`）。将该工件保存到实验环境的指定路径，以便检查。

> Jenkins 服务的运行身份为 `jenkins`，该用户默认没有访问 Docker 服务监听套接字文件的权限，因此无法执行 docker 指令。需要将 `jenkins` 用户加入到 `docker` 用户组，加入后还需重启 Jenkins 服务来使得修改生效。

## 知识点

- 手动编写 Jenkinsfile
- Jenkinsfile `docker` 指令使用
- Jenkins Pipeline 创建
- Jenkins Pipeline 运行
