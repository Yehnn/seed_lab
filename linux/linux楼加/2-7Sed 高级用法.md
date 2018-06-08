---
show: step
version: 0.1
enable_checker: true
---

# Sed 高级用法

## 1. 实验介绍

#### 1.1 实验内容

本实验我们将深入讲解 sed 的一些高级用法，编写 sed 脚本，完成比较复杂的文本修改操作。

#### 1.2 实验知识点

- 多重命令
- 缓冲空间
- 使用寻址
- 多行模式空间
- 保持空间
- 高级的流控制命令

#### 1.3 推荐阅读

- [sed 简明教程](http://coolshell.cn/articles/9104.html)
- [sed 单行脚本快速参考](http://sed.sourceforge.net/sed1line_zh-CN.html)
- [GNU 官方文档](https://www.gnu.org/software/sed/manual/sed.html)

## 2. 多重命令

在上一章节中我们简单的使用了 `sed`，帮助我们对文本文件做添加、删除、修改等操作。但都是单一命令的简单操作，如果我们需要同时执行多个操作，就要利用 `sed` 的多重命令特性了。

顾名思义，多重命令就是多个命令，它有如下几种使用方式。

我们先来新建一个实验文本 `words`，在里面输入如下内容，可以使用实验操作界面右边的工具栏中的剪切板将下面的文本内容贴入到实验环境中：

```bash
syl
I hate shiyanlou
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/words
  error: /home/shiyanlou 目录下没有 words 文件
```

### 2.1 使用分号分隔命令

我们想把文本的 `syl` 修改成 `shiyanlou`，同时把 `hate` 修改成 `love` ，可以使用分号分隔命令：

```bash
$ sed 's/syl/shiyanlou/; s/hate/love/' words
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1513179676364.png/wm)

如上所示，我们使用分号对命令进行了分隔。

### 2.2 使用 `-e` 参数

在 `sed` 命令中，还可以使用 `-e` 参数来执行多个命令：

```bash
$ sed -e 's/syl/shiyanlou/' -e 's/hate/love/' words
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1513179839795.png/wm)

如上所示，多次使用 `-e` 参数我们可以引用多个执行命令，从而达到我们的目的。

### 2.3. 分行

除了使用分号，以及 `-e` 参数之外，还可以通过分行来编写多个命令，达到多重命令的效果，如下所示：

```bash
$ sed '                  #按enter键                                         
quote> s/syl/shiyanlou/
quote> s/hate/love/' words
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1513180389075.png/wm)

## 3. 脚本文件

在命令行输入较长的命令是不切实际的，我们可以通过在脚本文件中写入多个 `sed` 命令。最后再使用 `sed` 的 `-f` 参数去执行脚本文件中的命令，对目标文件进行处理。这其实也属于多重命令的一种应用方式。

### 3.1 语法

```bash
$ sed -f <scriptfile> <file>
```

> - `scriptfile` ：脚本文件的名称。
> - `file` ：待处理文件的名称。

### 3.2 应用

与在 `多重命令` 中的示例类似，可以通过编写脚本文件来达到同样的效果。

如下所示的 `sed-script` 脚本文件中的内容：

```bash
s/syl/shiyanlou/
s/hate/love/
```

使用 `sed` 命令的 `-f` 参数执行，结果如下。 

```bash
sed -f sed-script words
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1513180711980.png/wm)

除此之外，还可以像写 `bash` 脚本的格式去书写 `sed` 脚本，这时不需要使用 `sed` 命令，直接赋予脚本权限，执行脚本即可。

例如如下所示的 `replace.sed` 脚本文件的内容：

```bash
#!/bin/sed -f

s/syl/shiyanlou/
s/hate/love/
```

然后通过 `chmod` 命令为其添加权限后，运行结果如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1513181173697.png/wm)

## 4. 寻址

`sed` 修改文件，有时候我们并不会对全文的内容进行操作，只对符合条件的行或者内容进行操作，这时候我们就会用到寻址了。而这里所谓的条件有两种：

- 指定的某一行
- 利用正则表达式，符合规则的内容

### 4.1 使用寻址

对于修改文件的内容，可以只对符合条件的行或者内容进行操作，也可以针对全文，大致的分类如下：

- 没有指定地址：应用于每一行
- 只有一个地址：应用于这个地址匹配的任意行
- 指定了由逗号分隔的两个地址：匹配第一个地址的第一行于匹配第二个地址之间的行
- 地址后跟有感叹号：应用于不匹配该地址的所有的行

我们将在以下应用实例中逐一展示：

新建文件 `xunzhi` ，输入如下内容：

```bash
a b
b
c

d b

e

f
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/xunzhi
  error: /home/shiyanlou 目录下没有 xunzhi 文件
```
把文件中的 `b` 字符替换成 `new`，使用如下操作：

```bash
$ nl xunzhi | sed 's/b/new/'
```

在下面的运行截图中可以看到所有的 `b` 都被替换成 `new`，这就是没有指定地址的方式。

如果我们想要将字符 `a` 所在行的字符 `b` 替换为 `new`，就需要使用寻址：

```bash
$ nl xunzhi | sed '/a/s/b/new/'
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971510383498454-wm)

若是我们想删除最后一行，可执行如下操作：

```bash
$ nl xunzhi | sed '$d'
```

>注意：`d` 是 sed 中的删除命令

利用正则表达式删除空行：

```bash
$ cat xunzhi | sed '/^$/d'
```

指定某个范围内的行，例如删除从第3行到最后一行的所有行：

```bash
$ nl xunzhi | sed '3,$d'
```

当然指定区间范围时，具体行数于正则也可以混合使用，例如删除第一行直到第一个空行的所有行：

```bash
$ cat xunzhi | sed '1,/^$/d'
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971510383399802-wm)

### 4.2 分组命令

在上面的操作中，我们只是对字符 `a` 所在的行执行**一个操作**，当我们想对 `a` 所在的行执行**多个操作**的时候就需要使用到分组命令。

sed 使用大括号 `{}` 将一个地址嵌套在另一个地址中，或者在相同的地址上应用多个命令。我们可以根据如下实践来更好的理解。

修改我们的 `xunzhi` 文件内容，如下所示：

```
a b hello world
b world
c b d hello world

d b world good

e hello

f
```

我们要把包含字符 `b` 又包含 `hello` 的行中的 `world` 改成 `shiyanlou`，再把第 5 行的 `world` 替换成 `louplus`。该怎么做呢？

我们发现第 `5` 行包含 `b` 和 `d`。我们把 `b` 作为一个共同的地址。找到有 `b` 的行，在有 `b` 的行中找到有 `hello` 的行，然后替换 `world` 为 `shiyanlou`，再在有 `b` 的行中找到有 `d` 的行，然后替换包含 `world` 的为 `louplus`。注意这个顺序，这里有两行都同时包含 `b`, `d`, `world` 三个字符串，如果先去替换成 `louplus` 的话，那么第三行的 `world` 也会被替换成 `louplus`，而后执行替换成 `shiyanlou` 的时候，第三行就没有 `world` 那个字符串了。

新建一个文件 `fenzu`，输入如下内容

```
# this is fenzu
/b/{
/hello/s/world/shiyanlou/
/d /s/world/louplus/
}
```

> 文件中的注释可以使用 `#` 开头，这里第一行就是注释。可以不写。
>
> 注意大括号后面不能有空格，并且右括号要单独在一行。
> 注意 `d` 后面有空格，如果没有空格的话，所有含有 `b` 字符的行的字符 `world` 都会被替换成 `louplus`。因为 `sed` 会把 `d` 认成一个特殊字符。采用转义符去转义 `d` 是不行的。

运行如下命令：

```bash
$ sed -f fenzu xunzhi
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971510392701235-wm)

## 5. 缓冲空间

#### 5.1 空间概念

通过上述内容我们了解多个 `sed` 操作可以写成一个脚本去执行，而 `sed` 是如何实现这样复杂的操作呢？这里将不得不提到 `sed` 维护的两个缓冲空间：

+ `pattern space`：模式空间，用于存放读取到的内容
+ `hold space`：保持空间，用于暂时存储模式空间传过来的内容

`sed` 简单的操作流程是这样处理文本文件的：

- 首先将文本内容的第一行读入模式空间中；
- 然后执行相关的处理命令；
- 紧接着将模式空间处理后的内容输出；
- 最后删除模式空间中的内容；

就这样不断的执行这样四个步骤直至将文本文件中所有的行都读取并操作完成。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1513185273450.png/wm)

而当我们需要做一些复杂一点的操作时，仅仅一个模式空间并不够，所以出现了保持空间：

- 首先将文本内容的第一行读入模式空间中；
- 然后执行相关的处理命令；
    + 将内容放置保持空间，暂存；
    + 执行相关处理；
- 紧接着将模式空间处理后的内容输出；
- 最后删除模式空间中的内容；

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1513186355252.png/wm)

>注意：两个空间在只能存储一行的内容，读取下一行内容时，若是空间中有内容只会被覆盖掉，要想存储两行的内容必须使用追加操作。追加操作的实质是在已有的内容后添加换行符，然后添加内容。

模式空间很好理解，在之前使用 `sed` 的例子中都是使用的模式空间处理相关的问题，那么什么时候我们会用到保持空间呢？

例如我们想将文章中的语句倒叙，也就是本是第一句的内容最后输出，本该最后一句输出的第一句输出.

在最开始的 `words` 文件，第一句是 `syl`，第二句是 `I love shiyanlou`，我们要的目的是第二句变成第一句，第一句变成第二句。我们可以这样处理：

- 首先 `sed` 将读取第一行放入模式空间
- 紧接着我们将模式空间的内容放入保持空间
- 读取下一行内容
- 然后我们将保持空间的的内容追加到模式空间
- 然后输出模式空间的内容

将在下文中具体实现该功能，这里只是为了介绍保持空间的某种使用方法。

### 5.2 多行模式空间

在上文的图示于注意中我们提到两个空间其实正常情况下只能存储单行内容，而在某种特殊操作下我们可以存储多行内容，此时我们变称为多行空间。此时的行我们是以换行符 `\n` 来分辨。

`sed` 允许匹配扩展到多行。包含 `3` 个多行命令 `(N,D,P)`

| 命令 | 意思                       |
| ---- | -------------------------- |
| N    | 追加下一行                 |
| D    | 删除多行模式空间的第一行   |
| P    | 打印多行模式空间中的第一行 |

#### 5.2.1 追加下一行

之前我们处理的字符串都在一行，当我们想要处理的字符串被断开了，比如我们想要将下面名叫 `zhuijia` 的实验文本中的 `modify that document` 替换成 `modify this document` ，但字符串不在一行上，`that` 被放在了下一行，这个时候我们就需要读取下一行的内容。

我们通过如下实践来理解，我们首先新建一个名叫 `sedn` 的脚本文件，实现把 `zuijia` 这个文本中的 `modify that document` 替换成 `modify this document`。

新建一个 `zuijia` ，内容如下：

```
Permission is granted to copy, distribute and/or modify
that document under the terms of the GNU Free Documentation License, Version 1.3 or any later version published by the Free Software Foundation; 
with no Invariant Sections, 
no Front-Cover Texts, and no Back-Cover Texts. A copy of the license is included in the section entitled “GNU Free Documentation License
```

新建一个 `sedn` ,内容如下：

```
N
s/modify\nthat/modify this/
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1513195784800.png/wm)

我们如何证明，这两行内容在模式空间中追加了呢？我们只需要删除 `N` 即可，就会发现达不到我们想要的效果了，而之所以这里能够成功是因为它的流程是这样的：

- 首先 `sed` 读取第一行内容放在模式空间中
- 然后 `N` 命令将下一行内容追加在模式空间中，而第一句与第二句之间就间隔了一个换行符
- 然后执行替换的命令

若是没有追加的话，这里正则表达式将无法匹配，也就没有办法做替换了。

#### 5.2.2 删除多行中第一行

通过 `D` 命令我们可以删除多行中的第一行，同样是上面的例子，我们若是在 `N` 命令与替换命令中间添加了 `D` 命令会是什么效果呢？可以思考一下。

#### 5.2.3 多行打印

打印命令 `p` 会打印模式空间中的所有内容，而打印命令 `P` 则输出多行模式空间的第一部分，也就是第一个换行符之前的内容。

我们通过以下实践来验证 `P` 的功能：

我们将匹配 `UNIX` 结尾的行，并且如果下一行是以 `System` 开始，那么在 `UNIX` 之后加上 `Operating`。

首先我们新建一个文本 `testprin`，内容如下：

```
Here are examples of the UNIX
System. Where UNIX
System appears, it should be the UNIX
Operating System.
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/testprin
  error: /home/shiyanlou 目录下没有 testprin 文件
```
我们新建一个脚本文件 prin 来替换 `System` 为 `Operating System`，内容如下：

```
/UNIX$/{
  N
  /\nSystem/{
    s// Operating &/
    P
    D
  }
}

```

> - `&` 意思是用分组中正则表达式的内容替换掉匹配的内容。
> - `//` 之间不加任何内容，表示的是上一次的匹配。在这例子中就是 `\nSystem`。
>
> 执行过程如下：
>
> 首先读入一行。如果结尾匹配 `UNIX`，那么执行 `N` 命令读入下一行。
>
> 1. 如果匹配到 `\nSystem`，那么将 `System` 替换成 `Operating \nSystem`，然后打印出替换之后的第一行，并把模式空间中的第一行删除。
> 2. 如果下一行不是 `System` 开头，那么完成匹配 `UNIX` 分组的操作，继续读取下一行进行匹配。
>

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/prin
  error: /home/shiyanlou 目录下没有 prin 文件
```
![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1510563155769.png-wm)

试试删除 `P` 或者 `D` 然后查看输出什么，或者更改成对应的`p` 与 `d` 命令。

### 5.3 保持空间

上文中我们对保持空间做了一定的介绍，以下是与保持空间相关的常用命令：

| 命令     | 缩写   | 功能                                 |
| -------- | ------ | ------------------------------------ |
| Hold     | h 或 H | 将模式空间的内容复制或追加到保持空间 |
| Get      | g 或 G | 将保持空间内容复制或追加到模式空间   |
| Exchange | x      | 交换保持空间和模式空间的内容         |

>注意：在以下例子中我们会用到 `!`，表示的是指定行不执行后续的操作，其他所有行都要执行。

我们通过上述的倒序实践来深入理解：

新建一个名为 `reverse.sed` 的脚本：

```bash
#!/bin/sed -nf

# 从第二行开始将保持空间的内容追加到模式空间中
1! G

# 直到最后一行才打印
$ p

# 将模式空间的内容复制到保持空间
h
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/reverse.sed
  error: /home/shiyanlou 目录下没有 reverse.sed 文件
```
然后为该脚本添加执行权限，然后我们执行：

```bash
./reverse.sed words
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1513197607326.png/wm)

我们来看其运行的流程：

- 首先 `sed` 读取第一行数据 `syl`
- 第一个命令 `1! G` 表示不是第一行的内容才会执行 `G` 命令，所以此时不操作
- 第二个命令 `$ p` 表示最后一行才打印，所以此时依旧不操作
- 第三个命令 `h` 表示将模式空间的内容复制到到保持空间中
    + 此时模式空间内容是：`syl`
    + 此时保持空间内容是：`syl`
- 第一行处理完毕
- 读取第二行内容 `I hate shiyanlou`
    + 此时模式空间内容是：`I hate shiyanlou`
    + 此时保持空间内容是：`syl`
- 第一个命令 `1! G` 表示不是第一行的内容才会执行 `G` 命令，所以此时会把保持空间的内容追加到模式空间
    + 此时模式空间内容是：`I hate shiyanlou`\n`syl`
    + 此时保持空间内容是：`syl`
- 第二个命令 `$ p` 表示最后一行才打印，所以此时便打印模式空间的内容
- 第三个命令 `h` 表示将模式空间的内容复制到到保持空间中
    + 此时模式空间内容是：`I hate shiyanlou`\n`syl`
    + 此时保持空间内容是：`I hate shiyanlou`\n`syl`

相信大家通过该例子能对 `h`、`H`、`g`、`G` 命令有一定的了解了，同时也对模式空间与保持空间有一定的认识了。

## 6. 高级的流控制命令

所谓的流控制，也就是像我们前面学习的流程控制一样，在 `sed` 中也有用来做分支和测试的命令。

我们将学习这两个命令，一个用于控制执行脚本的哪一部分，一个控制何时执行。他们分别是：

-分支（`b`）
-测试（`t`）

这两个命令可以将脚本中的运行转移到包含特殊标签的行，如果没有指定标签则将控制转移到脚本的结尾处。`分支命令`用于无条件转移，`测试命令`用于有条件转移。

定义标签的语法：

```
:mylabel
```

> - `mylabel` 标签是任意不多于 7 个字符
> - 标签本身占据一行并以冒号开始
> - 冒号与标签之间不允许有空格
> - 行结尾处的空格会被认为是标签的一部分，所以注意不要在标签后面插入空格
> - 当在分支命令或测试命令中指定标签时，命令和标签之间允许有空格。例如 `b mylabel`

将通过以下例子带领大家简单的了解一下分支命令 `b` 的作用：

```bash
$ printf '%s\n' a1 a2 a3|sed -e '/1/bx;s/a/z/;:x;y/123/456/'
```

> y 命令的格式是 `y/src/dst/`，其作用是在模式空间中匹配到任意 `src` 中的字符都会被替换成 `dst` 中的对应字符，例如在模式空间中匹配到 1，就会替换成相应位置的 4。

命令执行的流程是：

- `sed` 读取第一行内容 `a1`
- 第一个命令是 `/1/bx`，寻址到包含 1 的内容会跳转到设置的 `x` 标签处
- 执行后续命令 `y/123/456/`，此时模式空间内容是 `a1`，其中 `1` 得到匹配，被替换成 `4`
- 第一行执行完毕
- sed 读取第二行内容 `a2`
- 第一个命令是 `/1/bx`，此时是匹配不到 1，所以并不会跳转
- 第二个命令是 `s/a/z/` 加匹配到的 `a` 替换成 `z`
- 第三个命令是 `y/123/456/`，此时模式空间内容是 `z2`，其中 `2` 得到匹配，被替换成 `5`
- 第二行执行完毕
- 第三行执行与第二行类似。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1513199540858.png/wm)

### 6.1 分支和周期

`b`，`t` 命令后可以跟一个标签，使命令跳转至定义处。而标签用一个冒号跟一个或多个字母（例如 `:x` ）。如果标签被省略，分支命令将重新开始循环。

分支到标签和启动循环之间的区别：循环重新启动时，`sed` 首先打印模式空间的当前内容，然后将下一个输入行读入模式空间。跳转到标签（即使在程序的开始处），不会打印模式空间，也不会读取下一个输入行。

下面是一个不带标签的 `b` 命令，并由此简单地重新开始该循环。在每个循环中，打印模式空间并读取下一个输入行:

```bash
$ seq 3|sed b
1
2
3
```

下面是一个无限循环，它不会终止，也不会打印任何东西。该 `b` 命令跳转到 `x` 标签，一个新的周期永远不会开始：

```bash
$ seq 3|sed ':x;bx'
```

分支通常用 `n` 和 `N` 命令补充：两条命令都将下一个输入行读入模式空间，而不用等待周期重新开始。在读取下一个输入行之前，`n` 命令会打印当前的模式空间，然后清空它。`N` 将一个换行符和下一个输入行附加到模式空间。

```bash
$ seq 3|sed ':x;n;bx'
1
2
3
$ seq 3|sed ':x;N;bx'
1
2
3
```

> - 这两个例子都不会循环读取文本内容
> - 在第一个例子中，`n ` 命令首先打印模式空间的内容，清空模式空间然后读取下一个输入行。
> - 在第二个示例中，`N`  命令将下一个输入行附加到模式空间。行在模式空间中累积，直到没有更多的输入行要读取，然后 `N` 命令终止 `sed` 程序。当程序终止时，执行循环结束操作，并打印整个模式空间。

或许上面你还看不出什么。再试试下面的命令：

```bash
$ printf '%s\n' aa bb cc dd|sed ':x;n;=;bx'

$ printf '%s\n' aa bb cc dd|sed ':x;N;=;bx'

$ printf '%s\n' aa bb cc dd|sed ':x;n;s/\n/***/;bx'

$ printf '%s\n' aa bb cc dd|sed ':x;N;s/\n/***/;bx'
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971511505075055-wm)

> 等号意为打印当前行号。

```bash
$ printf '%s\n' aa bb cc dd|sed '='
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1511506698771.png-wm)

### 6.2 分支示例

我们通过下面的实践来更深入地理解分支命令 `b`：

我们新建一个测试文本 line.txt 输入如下内容：

```
All the wor=
ld's a stag=
e,
And all the=
 men and wo=
men merely =
players:
They have t=
heir exits =
and their e=
ntrances;
And one man=
 in his tim=
e plays man=
y parts.
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/line.txt
  error: /home/shiyanlou 目录下没有 line.txt 文件
```
以下程序使用地址匹配 `/=$/` 作为条件，如果当前模式空间以 `=` 结尾，使用 `N` 读取下一个输入行，取代所有 `=` 以及后面的换行符，使用 `b` 分支命令无条件地跳转到程序开头，而不用重新开始一个新的循环。如果模式空间不是以 `=` 结尾，将打印模式空间并启动新的循环。

```bash
$ sed ':x;/=$/{N;s/=\n//g;bx}' line.txt
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971511508253828-wm)

下面是一个用 `t` 命令实现相同效果。`N` 命令将除了最后一行的所有行附加到模式空间中去。替代命令用一个空字符串替换末尾的 `=` 和换行符。如果替换成功，则条件分支命令 `t` 跳转到程序的开头，而不会完成或重新启动循环。如果替换失败，则该 `t` 命令不会分支。然后，`P` 将打印模式空间内容，直到第一个新行，`D` 将删除模式空间内容，直到一个新行。

```bash
$ sed ':x;$!N;s/=\n//;tx;P;D' line.txt
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971511511077363-wm)

## 7. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎与我们交流：

- 多重命令用什么符号分隔
- 怎么保存输出
- 模式空间是什么，执行命令的时候模式空间有什么变化
- 怎么使用寻址
- 分组命令是用来做什么的
- 多行模式空间是用来做什么的，有哪几个命令
- 保持空间的作用
- 高级流控制命令中分支命令 b ，和测试命令 t 的使用