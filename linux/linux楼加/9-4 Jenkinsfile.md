# Jenkinsfile

## 实验介绍

本次实验在前面学习的 Pipeline 基础上，介绍更多有用的步骤（step）和常见模式，并给出一些比较复杂的 Jenkinsfile 示例。

## 实验知识点

- 字符串插值
- 使用环境变量
- 设置环境变量
- 处理凭证
- 处理参数
- 处理失败
- 使用多个 agent

## 为什么要编写 Jenkinsfile

Jenkinsfile 提交到源代码托管系统有以下好处：

- Pipeline 上的代码审查/迭代
- 审核追踪 Pipeline
- Pipeline 的单一真实来源，可以由项目的多个成员查看和编辑

Pipeline 支持两种语法，声明式（Declarative，在 Pipeline 2.5 中引入）和脚本式（Scripted），本文示例我们统一使用更易读易写的声明式语法。

## 创建 Jenkinsfile

Jenkinsfile 是一个文本文件，其中包含 Jenkins Pipeline 的定义并被签入源代码托管系统。下面的 Pipeline 实现了一个基本的三阶段（stage）持续交付 Pipeline。

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
```

不是所有的 Pipeline 都有这三个阶段，但是为大多数项目定义它们是一个很好的起点。

为了让 Pipeline 使用源代码托管系统里的 Jenkinsfile，请按照如下步骤定义 Pipeline：

1. 按照前面介绍的经典 UI 方式创建 Pipeline，直到进入到 Pipeline 配置页；
1. 在 Pipeline 节，选择“Pipeline script from SCM”方式；
1. 选择你使用的源代码托管系统类型（比如 Git），并填写仓库信息；
1. 指定仓库里 Jenkinsfile 的路径，默认为仓库根目录下名为 `Jenkinsfile` 的文件；

使用文本编辑器（支持 `Groovy` 语法高亮的更好）来创建 Jenkinsfile 并提交到 Pipeline 项目里指定的路径。

上面的声明式 Pipeline 示例包含实现持续交付 Pipeline 的最少必要结构。agent 指令指示 Jenkins 为 Pipeline 分配执行器和工作区。没有 agent 指令，声明式 Pipeline 不仅无效，而且无法做任何工作！

### Build

对于许多项目来说，Pipeline 中工作的开始是 Build 阶段。通常 Build 阶段是汇编、编译或打包源代码的地方。Jenkinsfile 不是用来取代现有的构建工具，比如 GNU/Make、Maven、Gradle 等，而是作为一个胶合层来把项目开发生命周期的各个阶段（构建、测试、部署等）粘合起来。

Jenkins 有大量的插件来支持调用几乎所有常用的构建工具。下面的这个例子简单地在 `sh` 步骤中调用 `make` 命令。假定系统是基于 Unix/Linux 的，如果是 Windows 系统，可使用 `bat`。

```groovy
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'make'
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }
    }
}
```

- `sh` 步骤调用 make 命令，并且只有在该命令退出码为零（亦即成功）时才会继续，任何非零的退出码都将使 Pipeline 失败。
- `archiveArtifacts` 捕获与模式 `**/target/*.jar` 匹配的文件，并将它们保存到 Jenkins Master 中以供日后使用。

### Test

运行自动化测试是任何成功的持续交付流程的关键组成部分。因此，Jenkins 拥有许多由插件提供的可记录、报告和可视化测试的工具。最基本的，当测试失败时，让 Jenkins 记录报告并在 Web UI 中展示出来就很有用。以下示例中使用的 `junit` 步骤由 `JUnit` 插件提供。

在下面的示例中，如果测试失败，则 Pipeline 标记为 `unstable`，在 Web UI 中以黄色球显示。根据记录的测试报告，Jenkins 还可以提供历史趋势分析和可视化。

```groovy
pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                // 使用 || true 来使得即便在 make check 失败时也继续执行
                sh 'make check || true'
                junit '**/target/*.xml'
            }
        }
    }
}
```

- 使用 `sh 'make || true'` 可确保 sh 步骤始终为零退出码，从而为 junit 步骤提供捕获和处理测试报告的机会。
- `junit` 捕获并关联与模式 `**/target/*.xml` 匹配的 JUnit XML 文件。

### Deploy

根据项目或组织的需求，部署步骤不尽相同，可能是将构建的工件发布到 Artifactory 服务器，或将代码推送到生产系统等等。

下面 Pipeline 示例的 Deploy 阶段，假设前面的 Build 和 Test 阶段都已成功执行。

```groovy
pipeline {
    agent any

    stages {
        stage('Deploy') {
            when {
              expression {
                currentBuild.result == null || currentBuild.result == 'SUCCESS' 
              }
            }
            steps {
                sh 'make publish'
            }
        }
    }
}
```

- 检查 `currentBuild.result` 变量来判断构建是否成功。如果失败，Pipeline 状态为 `UNSTABLE`。

每次成功的 Pipeline 运行都将存档相关的构建工件、测试结果报告以及 Jenkins 中完整的控制台输出。

## 编写 Jenkinsfile

### 字符串插值

Jenkins Pipeline 使用与 Groovy 相同的规则进行字符串插值。Groovy 的字符串插值支持可能会让许多语言新手感到困惑。虽然 Groovy 支持使用单引号或双引号声明一个字符串，例如：

```groovy
def singlyQuoted = 'Hello'
def doublyQuoted = "World"
```

但只有双引号字符串支持字符串插值，例如：

```groovy
def username = 'Jenkins'
echo 'Hello Mr. ${username}'
echo "I said, Hello Mr. ${username}"
```

输出：

```groovy
Hello Mr. ${username}
I said, Hello Mr. Jenkins
```

了解如何使用字符串插值对于使用 Pipeline 的某些更高级功能至关重要。

### 使用环境变量

Jenkins Pipeline 通过全局变量来暴露环境变量，全局变量 env 可在 Jenkinsfile 中的任何位置使用。在 Jenkins Pipeline 中可访问的环境变量的完整列表记录在 `localhost:8080/pipeline-syntax/globals#env,` ，假设 Jenkins Master 运行在 `localhost:8080`。下面是一些环境变量示例：

#### BUILD_ID

当前构建 ID，在 Jenkins 版本 1.597+ 里与 `BUILD_NUMBER` 相同。

#### JOB_NAME

此次构建的项目名称，例如“foo”或“foo/bar”。

#### JENKINS_URL

Jenkins 的完整 URL，例如 `example.com:port/jenkins/`（注意：只有在“System Configuration”中设置了 Jenkins URL 时才可用）。

引用或使用这些环境变量可以像访问 Groovy Map 中的任何键一样完成，例如：

```groovy
pipeline {
    agent any
    stages {
        stage('Example') {
            steps {
                echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
            }
        }
    }
}
```

### 设置环境变量

依据使用声明式 Pipeline 还是脚本式 Pipeline，在 Jenkins Pipeline 中设置环境变量的方式会有所不同。声明式 Pipeline 支持 `environment` 指令，而脚本式 Pipeline 必须使用 `withEnv` 步骤。

```groovy
pipeline {
    agent any
    environment {
        CC = 'clang'
    }
    stages {
        stage('Example') {
            environment {
                DEBUG_FLAGS = '-g'
            }
            steps {
                sh 'printenv'
            }
        }
    }
}
```

- environment 在顶层 pipeline 块中使用的指令将应用于 Pipeline 内的所有步骤。
- environment 在 stage 中定义的只能应用于该 stage。

### 处理凭证

在 Jenkins 中配置的凭证可以在 Pipelines 中进行处理以供使用。Jenkins 的声明式 Pipeline 语法具有 `credentials()` 辅助方法（在 `environment` 指令中使用），该方法支持秘密文本、用户名密码以及秘密文件这几种凭证类型。

#### 秘密文本（Secret text）

以下 Pipeline 代码显示了如何在 Pipeline 里通过环境变量来使用秘密文本凭证。

在此示例中，将两个秘密文本凭证分配给单独的环境变量以便访问 Amazon Web Services（AWS）。这些凭证在 Jenkins 中对应的凭证 ID 分别为`jenkins-aws-secret-key-id` 和 `jenkins-aws-secret-access-key`。

```groovy
pipeline {
    agent {
        // Define agent details here
    }
    environment {
        AWS_ACCESS_KEY_ID     = credentials('jenkins-aws-secret-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins-aws-secret-access-key')
    }
    stages {
        stage('Example stage 1') {
            steps {
                //
            }
        }
        stage('Example stage 2') {
            steps {
                //
            }
        }
    }
}
```

- 您可以使用语法 `$AWS_ACCESS_KEY_ID` 和 `$AWS_SECRET_ACCESS_KEY` 来在阶段的步骤里引用凭证。如果尝试在 Pipeline 中检索（例如 `echo $AWS_SECRET_ACCESS_KEY`）这些凭证变量的值，则 Jenkins 仅返回值“****”，以防止秘密信息被写入控制台输出或任何日志。
- 在这个 Pipeline 示例中，分配给这两个 `AWS_...` 环境变量的凭证作用域为整个 Pipeline，因此这些凭证变量可以在任何阶段的步骤中使用。

#### 用户名和密码（Usernames and Passwords）

以下 Pipeline 代码片段显示了如何使用用户名和密码凭证环境变量来创建 Pipeline。

在此示例中，用户名和密码凭证分配给环境变量，以便使用一个公共的账号来访问 Bitbucket 仓库。这些证书需要已在 Jenkins 中使用证书 ID `jenkins-bitbucket-common-creds` 配置过。

在 environment 指令中设置凭证环境变量时：

```groovy
environment {
    BITBUCKET_COMMON_CREDS = credentials('jenkins-bitbucket-common-creds')
}
```

实际上设置了以下三个环境变量：

- `BITBUCKET_COMMON_CREDS`：包含用分号分隔的用户名和密码 `username:password`。
- `BITBUCKET_COMMON_CREDS_USR`：仅包含用户名。
- `BITBUCKET_COMMON_CREDS_PSW`：仅包含密码。

以下代码片段显示了整个示例 Pipeline：

```groovy
pipeline {
    agent {
        // Define agent details here
    }
    stages {
        stage('Example stage 1') {
            environment {
                BITBUCKET_COMMON_CREDS = credentials('jenkins-bitbucket-common-creds')
            }
            steps {
                //
            }
        }
        stage('Example stage 2') {
            steps {
                //
            }
        }
    }
}
```

在此 Pipeline 示例中，分配给三个 `COMMON_BITBUCKET_CREDS...` 环境变量的凭证仅限于范围“Example stage 1”，因此这些凭证变量不可用于 “Example stage 2” 阶段的步骤。

#### 秘密文件（Secret files）

秘密文件的处理方式与前面的秘密文本完全相同。秘密文本和秘密文件凭证之间的唯一区别在于：对于秘密文本，凭证本身直接输入 Jenkins；而对于秘密文件，凭证存储在文件中，然后上传到 Jenkins。

与秘密文本不同，秘密文件具有以下特征：

- 文件太大，直接输入 Jenkins 太笨重
- 二进制格式，例如 GPG 文件

### 处理参数

声明式 Pipeline 支持开箱即用参数，允许 Pipeline 通过参数指令在运行时接受用户指定的参数。如果使用“Build with Parameters”选项将 Pipeline 配置为接受参数，则可以将参数作为 `params` 变量的成员进行访问。

假设名为“Greeting”的字符串参数已配置，则可以通过 `${params.Greeting}` 方式访问该参数：

```groovy
pipeline {
    agent any
    parameters {
        string(name: 'Greeting', defaultValue: 'Hello', description: 'How should I greet the world?')
    }
    stages {
        stage('Example') {
            steps {
                echo "${params.Greeting} World!"
            }
        }
    }
}
```

### 处理失败

声明式 Pipeline 支持多种失败处理方式，允许声明许多不同的“后置条件”来进行处理，例如 `always`、`unstable`、`success`、`failure` 和 `changed`。

```groovy
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'make check'
            }
        }
    }
    post {
        always {
            junit '**/target/*.xml'
        }
        failure {
            mail to: team@example.com, subject: 'The Pipeline failed :('
        }
    }
}
```

### 使用多个 agent

在前面的所有例子中，只使用了一个 agent。这意味着 Jenkins 会在任何可用的地方分配一个执行器，而不管执行器是如何标记或配置的。这种行为不仅可以被覆盖，并且 Pipeline 还允许使用 Jenkins 环境中的多个 agent 来满足更高级的使用场景，比如在多个平台执行构建或测试。

在下面的例子中，将在一个 agent 上执行构建阶段，测试阶段分别在两个标记为“linux”和“windows”的 agent 上执行，它们都会使用构建阶段的结果。

```groovy
pipeline {
    agent none
    stages {
        stage('Build') {
            agent any
            steps {
                checkout scm
                sh 'make'
                stash includes: '**/target/*.jar', name: 'app'
            }
        }
        stage('Test on Linux') {
            agent {
                label 'linux'
            }
            steps {
                unstash 'app'
                sh 'make check'
            }
            post {
                always {
                    junit '**/target/*.xml'
                }
            }
        }
        stage('Test on Windows') {
            agent {
                label 'windows'
            }
            steps {
                unstash 'app'
                bat 'make check'
            }
            post {
                always {
                    junit '**/target/*.xml'
                }
            }
        }
    }
}
```

- stash 步骤允许捕获匹配模式 `**/target/*.jar` 的文件以在 Pipeline 内重用。一旦 Pipeline 完成执行，stashed 的文件将从Jenkins Master 中删除。
- agent 参数允许任何有效的 Jenkins 标签表达式。
- unstash 将从 Jenkins Master 中检索指定名称的 stash 到 Pipeline 当前工作空间。
- bat 允许在 Windows 平台上执行批处理脚本。

### 可选的步骤参数

Pipeline 遵循 Groovy 语言约定，允许在方法参数周围省略括号。许多 Pipeline 步骤还使用命名参数语法来作为简化 Map 的创建，创建 Groovy Map 的语法为 `[key1: value1, key2: value2]`。以下两种写法效果一样：

```groovy
git url: 'git://example.com/amazing-project.git', branch: 'master'
git([url: 'git://example.com/amazing-project.git', branch: 'master'])
```

为方便起见，当调用只有一个参数的步骤（或者只有一个必需参数）时，参数名称可以省略，例如：

```groovy
sh 'echo hello' /* short form  */
sh([script: 'echo hello'])  /* long form */
```

## 4. 总结

本次实验我们深入学习了 Jenkinsfile 的声明式语法，包括使用环境变量、处理凭证、处理参数和处理失败等。
