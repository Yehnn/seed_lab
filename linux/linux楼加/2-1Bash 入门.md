---
show: step
version: 0.1
enable_checker: true
---

# Bash 入门

## 1. 实验介绍

#### 1.1 实验内容

本节实验带领大家初步了解 `shell` 和 `bash` 的概念。

**在后面的 `bash` 编程中，我们将会使用到第一周学习的知识，并对一些常用的内容进行复习**。

#### 1.2 实验知识点

- shell 的介绍
- 常用的 shell
- bash 的介绍
- bash 脚本的执行方式

## 2. shell 介绍

操作系统与用户之间有一个界面，来实现操作系统与用户的交互。用户可以通过一些命令实现对系统的操作，比如复制、删除文件，查看网络等等。在文本环境中，就是由 shell 实现两者的沟通。它是命令语言、命令解释程序及程序设计语言的统称。

shell，用户及 linux 系统内核之间的关系如下图：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1509516681464.png)

## 3. 常用的 shell

Linux 中的 shell 有多种类型，其中最常见的有 `Bourne Shell（sh）`、`C Shell（csh）`、`Korn Shell（ksh）`。
`Bourne Shell` 是 Unix 最初始的 Shell，并且在每种 Unix 上都可以使用，在 Shell 编程方面相当优秀，但在处理与用户交互上不如其他几种。
`Bash（Bourne Again Shell）` 是 Bourne Shell 的扩展，与 Bourne Shell 完全兼容，并且增加了许多特性，还包含了很多 C Shell 和 Korn Shell 中的优点，有灵活和强大的编程接口，同时又有友好的用户界面。

实验楼环境中还安装了 zsh，也是一种 shell ，它在补全功能、内容显示、主题等做了更多的功能强化。

我们该如何查看自己目前所在的 shell 版本，以及怎么切换 shell 呢？

**查看当前使用的 shell**

```bash
$ echo $0
```

**切换到 bash 交互式下**

```bash
$ bash

$ echo $0
```

系统中有着一个变量 `$SHELL` 记录着当前用户默认使用的 shell：

```bash
echo $SHELL
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1512668694960.png/wm)

退出当前的 `shell` 可以使用 `ctrl + D` 或者输入 `exit`。

对于使用上述的方式进入到 `bash` 中，其实是开启了一个新的 `bash` 的进程，如何验证呢？在第一周进程实验中我们讲解到进程之间的父子关系，我们可以利用这一点来验证，上述我们切换到 bash 之后我们通过这样的命令来查看:

```bash
#这里 -A 的参数标示显示匹配内容相关的行数
ps auxf |grep zsh -A 2
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1512669171831.png/wm)

从图中我们可以明确地查看到 `bash` 是由 `zsh` `fork` 出来的子进程。

而上述使用 `exit` 命令退出的只是当前的 `shell`，也就是 `bash`，然后会到了 `zsh` 的交互环境中。

Shell 简介及常用 Shell 操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week2/1-1.mp4
@`

## 4. Bash 介绍

首先介绍一下 bash 相关知识。

### 4.1 什么是 Bash

`Bash` 是 `GNU Bourne-Again Shell` 的缩写。因为 `Bourne-Again Shell` 功能的增强，所以对于 `Bourne shell（sh）`是一个双关语（`Bourne again / born again`）。

`Bourne shell` 是一个早期的重要 `shell`，由史蒂夫·伯恩在 `1978` 年前后编写，并同 `Version 7 Unix` 一起发布。`Bash` 则是在 1987 年由布莱恩·福克斯创造。在 1990 年，`Chet Ramey` 成为了主要的维护者。

### 4.2 为什么要学习 Bash

`Bash` 是每个 `Linux` 发行版都带有的一个标准基础软件，通过 `bash` 脚本可以批量完成一些任务，方便快捷。在 `Bash` 中也可以使用 `Linux` 命令，实现类似 `Windows` 下的批处理文件的功能。

### 4.3 Bash 特性

#### 4.3.1 自动补全（TAB）

`TAB` 键可以自动补全命令或者补全路径。

例如我们要打开当前目录下的 `Code` 文件夹

```bash
$ cd C
```

然后按下 `TAB` 键会自动补全为 `cd Code/` ，再按 `enter` 键就到 `Code` 目录下。

比如我们要执行清屏命令：

```bash
$ cle
```

然后按下 `TAB` 键会自动补全为 `clear` ，再按 `enter` 键就清除了屏幕的内容。

#### 4.3.2 命令历史记录（history）

查看命令历史记录的命令为 `history`

```bash
$ history
```

该命令会列出该用户针对当前种类 `shell` 使用过的命令（例如所有在 `bash` 下使用过的命令，例如在 `zsh` 下所有使用过的命令），对于 `bash` 而言，这些使用过的命令其实是保存在用户目录下的 `.bash_history` 文件中，即完整路径为 `/home/shiyanlou/.bash_history`。

除此之外，还可以通过 `ctrl + r` 再输入关键词，找出包含关键词的历史命令。

清除历史记录我们可以使用 `history -c` 命令（即清空 `history` 命令中的缓存，当然该命令在 `zsh` 中并没有 `-c` 参数）：

```bash
$ history -c
```

#### 4.3.3 命令别名（alias）

对于很多命令来说，我们可以使用简写等方式来达到同样的效果，这种方法被称为别名。

如下示例，对于 `ls` 命令，我们经常会使用到的 `-a` 和 `-l` 参数：

```bash
$ ls -al
```

针对长命令使用别名的这种方式。就像是实验楼的全拼 `shiyanlou` 一样，很多时候我们更愿意使用简写 `syl` 去代替 `shiyanlou`。

例如上面的命令 `ls -al` 我们就可以使用 `alias` 命令给它起一个别名叫 `la`：

```bash
$ alias la='ls -al'
```

然后我们输入 `la`，就可以起到 `ls -al` 命令一样的作用，除此之外，`alias` 命令还可以用于覆盖和修改原有的别名。例如当我们使用 ls 的时候命令是单纯的使用 `ls` 命令，他其实是 `ls --color=tty` 命令的别名。

>注意：查看的方法，我们在讲解内外键命令的时候有提及如何查看（`type` 命令）。

对应的删除别名使用 `unalias` 命令：

```bash
$ unalias la
```

现在再输入 `la` 执行就会出错。**但是这种暂时性的修改和删除只在当前 `shell` 中起作用。** 对于 `bash` 而言，我们有针对当前用户的配置文件 `/home/shiyanlou/.bashrc` 和系统的配置文件 `/etc/bash.bashrc`。可以通过修改配置文件来修改 `bash` 在启动时的一些默认项。

#### 4.3.4 通配符

在查找文件和字符的时候可以使用通配符，这里说的通配符并不是正则表达式，在第一周文件查找一节中，我们有学习过使用通配符匹配文件名进行查找的操作。

#### 4.3.5 管道符以及重定向标准输入输出

在第一周的 `Linux 管道符` 我们学习过相关内容，常用的操作如下：

+ 输出重定向，使用 `>` 符号

+ 追加内容可以使用 `>>`

+ 输入重定向，使用 `<` 符号

#### 4.3.6 两级提示符

`bash` 有两级用户提示符。

+ 第一级是用户经常看到的 `bash` 在等待命令输入时的提示符。默认的一级提示符是字符 `$` （如果是超级用户，就是 `#` 号）。可以通过更改 `PS1` 环境变量的值来设置提示符。

+ 二级提示符为提示多行输入或显示的提示符（非换行符），默认的二级提示符是 `>` ，可以通过更改 `PS2` 环境变量进行设置。例如，我们直接输入斜杠 `\`，就会出现二级提示符 `>`。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1512629202201.png/wm)

我们尝试修改一级提示符：

```bash
$ export PS1="XD >"
XD >
XD >
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1512628419558.png/wm)

输入命令后，我们可以看见命令提示符被更改。如果想将这种更改固定下来，可以把这个命令放到 `~/.bashrc` 中。

我们还可以改变颜色为绿色

```bash
export PS1="\[\e[32;1m\][\u@\h \W]$ > \[\e[0m\]"
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1512628770559.png/wm)

在上述修改命令提示符是所使用的变量的值，如 `\u`，`\h` 代表的含义如下：

| 字符   | 含义                | 字符 | 含义                          |
| :----- | :------------------ | :--- | :---------------------------- |
| `\u`   | 当前用户的用户名    | `\!` | 该命令的历史记录编号          |
| `\h`   | 主机名              | `\#` | 当前命令的命令编号            |
| `\d`   | 当前日期            | `\$` | 如果用户是 root，显示的是 `#` |
| `\t`   | 当前时间            | `\w` | 当前工作目录的的路径          |
| `\s`   | 当前运行的shell名字 | `\W` | 当前工作目录的名字            |
| `\nnn` | nnn的八进制         | `\\` | 显示反斜杠                    |
| `\n`   | 打印新行            |      |                               |

| 前景 | 背景 | 颜色   |
| :--- | :--- | :----- |
| 30   | 40   | 黑色   |
| 31   | 41   | 红色   |
| 32   | 42   | 绿色   |
| 33   | 43   | 黄色   |
| 34   | 44   | 蓝色   |
| 35   | 45   | 粉红色 |
| 36   | 46   | 青蓝色 |
| 37   | 47   | 白色   |

| 代码 | 意义     |
| :--- | :------- |
| 0    | off      |
| 1    | 高亮显示 |
| 4    | 下划线   |
| 5    | 闪烁     |
| 7    | 反白显示 |
| 8    | 不可见   |

Bash 常用命令操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week2/1-2.mp4
@`

## 5. 脚本执行的方式

在第一周的 `Linux 基本操作` 一节中，我们有写过一个非常简单的 `bash` 脚本 `test.sh`，对于 `Linux` 来说，并不根据文件的扩展名来识别文件类型，这里的 `.sh` 只是为了让使用该脚本的人进行一个区分，类似于 `Linux` 中目录与文件的颜色不一致。

我们将通过以下的例子为大家示范执行的方式：

如下所示 `test1.sh`，一个简单 bash 脚本中的内容：

```bash
#!/bin/bash

# I love shiyanlou

username="syl001"
echo "Hello syl001 !"
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/test1.sh
  error: /home/shiyanlou 目录下没有 test1.sh 文件
```

第一行的 `#!`是一个约定的标记，告诉系统这个脚本是用指定的解释程序来执行，这里使用的是 `/bin/bash`。

第二行为注释，以 `#` 开头的内容为注释，注释是记录方式不会被程序所执行，一般用于记录其中内容的特殊意义。

在脚本中我们为一个变量 `username` 赋值，并输入一个字符串，我们在后面会学习到有关于变量的内容。

而对于执行脚本，有四种方式：

```bash
source test1.sh

. test1.sh

./test1.sh

bash test1.sh 
```

使用 `source` 命令，只是简单的读取脚本当中的语句并在当前 `shell` 中执行。例如，我们执行上面的 `test1.sh` 文件，执行后文件中定义的 `$username` 变量可以在当前的 `shell` 中使用：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1512631132569.png/wm)

而直接使用文件的方式，需要 `test1.sh` 为一个可执行的文件，即拥有执行的权限，前面的 `./` 是指定路径：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1512632564469.png/wm)

对于在脚本中第一行指定 `#!/bin/bash`，与第三种直接使用 `bash` 执行的方式原理是一样的，都会打开一个新的 `bash` 的进程去执行脚本。两者之间的区别在于直接使用 `bash` 不需要该文件为可执行文件，仅需拥有读取文件内容的权限即可。

每一个脚本或者命令执行完成之后都会有一个退出的状态，系统中一般只会记录上一个指令执行之后的退出状态，我们可以通过 `$?` 变量来获取这个状态值，常见的值有：

- 0：表示上一个指令执行成功
- 1：表示未知的错误
- 126：表示命令无法执行
- 127：表示无效命令
- 130：表示通过 `ctrl+c` 退出命令

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1512728033493.png/wm)

当然我们在脚本中也可以控制脚本退出的状态，通过 `exit 状态值`即可。直接使用 `exit` 命令等同于 `exit 0`，即默认值为 `0`.

Bash 脚本执行操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week2/1-3.mp4
@`

## 6. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎与我们交流：

- shell 的介绍
- 常用的 shell
- bash 的介绍
- bash 脚本的执行方式