# 使用 Blue Ocean 创建 Pipeline

## 实验介绍

前面已经学习过“使用经典 UI”和“手动编写 Jenkinsfile”两种方式来创建 Pipeline，本次实验我们来学习第三种“使用 Blue Ocean”。

## 实验知识点

- 什么是 Blue Ocean
- 安装 Blue Ocean 插件
- 添加节点
- 使用 Blue Ocean 创建 Pipeline

## 什么是 Blue Ocean

Blue Ocean 重新思考了 Jenkins 的用户体验。它为 Jenkins Pipeline 从零开始设计，但仍与自由式作业兼容，Blue Ocean 为团队成员减少了混乱并增加了清晰度。Blue Ocean 的主要特点包括：

- 强大的 Pipeline 可视化，可以快速直观地了解 Pipeline 状态。
- Pipeline 编辑器：引导用户通过直观和可视化的过程来创建 Pipeline，从而使 Pipeline 的创建变得平易近人。
- 个性化以适应团队中每个成员的基于角色的需求。
- 在需要干预或出现问题时及时提醒，有助于加快异常处理和提高生产力。
- 原生支持分支和拉取，使得在与 GitHub 和 Bitbucket 中的其他人协作时能最大程度释放开发人员生产力。

Blue Ocean 默认没有启用，需要先安装对应插件。为了让创建的 Pipeline 有地方运行，还需要添加一个节点。接下来我们使用 Blue Ocean 来创建一个 Pipeline 来集成一个位于 GitHub 上的 Python Web 应用。

## 安装 Blue Ocean 插件

1\. 在 Jenkins 首页，点击左侧的“系统管理”，然后点击“管理插件”：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517815267870.png/wm)

2\. 选择“Blue Ocean”插件进行安装，安装完成后返回首页：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517815287098.png/wm)

> 如果安装完成后页面有报错，可能是插件之间的依赖关系导致，重启一下 Jenkins 服务即可解决。

## 添加节点

Pipeline 运行在节点上，如果系统里还没有节点，可按如下步骤添加：

1\. 在 Jenkins 首先点击“系统管理”，然后选择“管理节点”：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517816859810.png/wm)

2\. 点击左侧的“创建节点”：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517817253920.png/wm)

3\. 添加标签，选择启动方法，并在弹出框中添加证书：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517817659323.png/wm)

4\. 输入节点服务器的登录用户名和密码（可使用本地服务器）：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517817644938.png/wm)

5\. 证书创建完成后，在下拉框中选择刚才创建的证书：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517817937490.png/wm)

6\. 点击 `Save` 之后，该节点就添加成功了。

## 准备 GitHub 项目

把要集成的项目放到你的 GitHub 账号下，以便 Jenkins 拉取。该项目的目录结构如下所示（仓库名为 `test_git`）：

```text
test_git
|----app.py
|----requirements.txt
|----README.md
```

`requirements.txt` 内容如下:

```text
flask==0.10
```

`app.py` 内容如下:

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello():
    return 'Hello Shiyanlou002!'

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8001, debug=True)
```

为了让 Jenkins 能从 GitHub 上拉取项目，还需要创建一个 GitHub 的账号证书：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517819494670.png/wm)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517819600998.png/wm)

## 创建 Pipeline

1\. 打开 Jenkins 首页，选择左侧的“Open Blue Ocean”：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517463933648.png/wm)

2\. 点击 “创建新的 Pipeline”：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517465773008.png/wm)

3\. 选择 GitHub 代码仓库，并点击“Create an access key here”：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517465918693.png/wm)

4\. 创建 `token`：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517466199952.png/wm)

5\. Token 创建成功后，复制并粘贴到上一个页面的输入框中：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517466559684.png/wm)

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517466565068.png/wm)

6\. 选择 test_git 仓库，点击 “创建 Pipelines”，将进入到可视化的 Pipeline 编辑器：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517818872018.png/wm)

7\. 在可视化的 Pipeline 编辑器里，可添加阶段以及阶段下的步骤。在当前选中阶段下，点击右侧的“Add step”来给阶段添加步骤。依次创建一个“Change current directory”步骤和“Git”步骤。在“Git”步骤中，需要输入前面创建的 GitHub 账户证书 ID：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517819986853.png/wm)

8\. 继续添加两个“build”和“run”阶段，并在每个阶段下添加一个类型为“Shell Script”的步骤，对应的命令分别为 `sudo -H pip -r requirements.txt` 和 `python app.py`。

> 如果执行 `sudo -H pip -r requirements.txt` 报错，则可能用的是新版 `pip`，改成 `sudo -H pip install -r requirements.txt` 即可。

9\. 点击右上角的“Save”来提交：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517824244458.png/wm)

10\. 运行结果如下:

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517825222068.png/wm)

11\. 如果 Pipeline 执行成功，就可以通过地址 `http://localhost:8001` 来访问应用：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517825289349.png/wm)

Blue Ocean 会自动在 GitHub 仓库目录下添加一个 Jenkinsfile 文件：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1517907197961.png/wm)

当然也可以手动编写好 Jenkinsfile 文件并提交到仓库，在可视化 Pipeline 编辑器页面会自动加载该文件。

一旦创建好 Pipeline，后续每次提交代码到项目时，都可以手动运行 Pipeline 来测试代码的正确性，甚至可以设置在提交代码时自动运行 Pipeline。

## 总结

本次实验我们学习了 Blue Ocean 的用法，可以看到其体验很良好，它使得创建 Pipeline 变得更容易了。
