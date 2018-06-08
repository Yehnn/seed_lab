---
show: step
version: 0.1
enable_checker: true
---

# Awk 文本处理

## 1. 实验介绍

#### 1.1 实验内容

本实验将带领大家简单入门 awk 的使用。

#### 1.2 实验知识点

- awk 介绍
- awk 的模式
- awk 命令基本选项
- awk 变量
- 格式化打印 printf

#### 1.3 推荐阅读

- [awk 程序设计语言](http://awk.readthedocs.org/en/latest/chapter-one.html)
- [awk 简明教程](http://coolshell.cn/articles/9070.html)
- [awk 用户指南](http://www.gnu.org/software/gawk/manual/gawk.html)

## 2. awk 介绍

> `AWK` 是一种优良的文本处理工具，Linux 及 Unix 环境中现有的功能最强大的数据处理引擎之一.其名称得自于它的创始人 Alfred Aho（阿尔佛雷德·艾侯）、Peter Jay Weinberger（彼得·温伯格）和 Brian Wilson Kernighan（布莱恩·柯林汉) 姓氏的首个字母。AWK 程序设计语言，三位创建者已将它正式定义为“样式扫描和处理语言”。它允许您创建简短的程序，这些程序读取输入文件、为数据排序、处理数据、对输入执行计算以及生成报表，还有无数其他的功能。最简单地说，AWK 是一种用于处理文本的编程语言工具。（此段引用于[维基百科](https://zh.wikipedia.org/wiki/Awk)）

在大多数 Linux 发行版上面，实际我们使用的是 `gawk`（GNU awk，awk 的 GNU 版本），在我们的环境中 `ubuntu` 上，默认提供的是 `mawk` ，不过我们通常可以直接使用 `awk` 命令（awk语言的解释器），因为系统已经为我们创建好了`awk` 指向 `mawk` 的符号链接。

```bash
$ ll /usr/bin/awk
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1510568906467.png-wm)

> - `nawk`： 在 20 世纪 80 年代中期，对 awk 语言进行了更新，并不同程度地使用一种称为 `nawk` (new awk) 的增强版本对其进行了替换。许多系统中仍然存在着旧的 awk 解释器，但通常将其安装为 `oawk` (old awk) 命令，而 `nawk` 解释器则安装为主要的 `awk` 命令，也可以使用 `nawk` 命令。Dr. Kernighan 仍然在对 `nawk` 进行维护，与 `gawk` 一样，它也是开放源代码的，并且可以免费获得。
> - `gawk`： 是 `GNU Project` 的 `awk` 解释器实现。尽管早期的 `GAWK` 发行版是旧的 `AWK` 的替代程序，但不断地对其进行了更新，已包含 `NAWK` 的特性。
> - `mawk` ：也是 `awk` 编程语言的一种解释器，`mawk` 遵循 `POSIX 1003.2` （草案 11.3）定义的 `AWK` 语言，包含了一些没有在 `AWK` 手册中提到的特色，同时 `mawk` 提供一小部分扩展,另外据说 `mawk` 是实现最快的 `awk` 。

awk 处理的对象可以是一个文本文件，亦或者是通过管道符传过来的内容。无论什么形式的内容其本质上都是通过一个 for 循环处理，每次读入一行处理，然后转而执行下一行，直到整个文件的每一行都被执行完毕。

## 3. awk 的模式

下面学习 awk 模式相关内容。

### 3.1 记录和字段

在学习 awk 模式之前，我们首先来了解两个概念：记录，字段。

例如我们现在有这样一个表格，

| 张三   | a    | b    | c    |
| :----- | :--- | :--- | :--- |
| 李四   | d    | e    | f    |
| 王麻子 | g    | h    | i    |

记录：指这个表中的每一行。这个表有 3 行记录。第一条记录是 “张三 a b c” ，依次类推。

字段：指这个表中的每一列。也就是由字段分隔符所切分出来的每个部分，例如该表若以空格分割，每行有 4 个字段。例如第一行的第一个字段是 “张三” ，依次类推。

### 3.2 什么是 awk 的模式

`awk` 所有的操作都是基于 `pattern(模式)—action (操作)` 对来完成的，如下面的形式：

```bash
pattern {action}
```

你可以看到就如同很多编程语言一样，它将所有的动作操作用一对`{}`花括号包围起来。

> - 模式 `Pattern` 用于筛选记录，操作 `Action` 用于处理字段。
> - 其中 `pattern` 通常是表示用于匹配输入的文本的“关系式”或“正则表达式”（若是正则表达式需要使用 `//` 括住）
> - `action` 则是表示匹配后将执行的操作。
> - 对于 `awk` 读取的每条记录，如果一个记录与指定模式 `Pattern` 相匹配，或包含与该模式匹配的字段，那么执行相应的操作 `Action` 。
> - 在一个完整 `awk` 操作中，这两者可以只有其中一个，如果没有 `pattern` 则默认匹配输入的全部文本，操作会被应用到每条输入记录。如果没有 `action` 则默认为打印匹配内容到屏幕。

#### 应用实例

下面的实例带领大家更好地理解 awk 的模式以及省略模式和省略操作的区别。

我们先来查看 `Desktop` 目录下文件详细信息：

```bash
$ ll Desktop/
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/596222/1512925261620.png-wm)

> 输出的结果共有 5 行，每行被分割为 9 个部分。对 awk 来说，这就是 5 条`记录`（行），每条记录包含 9 个`字段`（列）。

如果我们想查看包含 firefox 的字段，则可以执行如下命令：

```bash
$ ll Desktop|awk '/firefox/{print $0}'
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/596222/1512925550489.png-wm)

> 上面的 `/firefox/` 即为模式，`print $0` 即为操作。
>
> `print $0` 意为打印当前行。
>
> **上面的命令可以这样理解：**
>
> `awk` 读取 `ll Desktop` 命令的每条输出作为记录，然后将模式与记录进行匹配（查找记录中是否包含 `firefox` 这个词），如果匹配，则执行操作（打印整条记录到屏幕）。

接下来我们分别省略模式和操作，看看输出的结果有什么不同：

省略操作：

```bash
$ ll Desktop|awk '/firefox/'
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/596222/1512926289911.png-wm)

>  上面是省略操作后的输出，可以看出默认的操作就是 `print $0`，也就是打印整条记录

省略模式：

```bash
$ ll Desktop|awk '{print $3,$9}' 
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/596222/1512926533833.png-wm)

> 上面是省略模式后的输出，可以看出 `awk` 对于每条记录都执行了对应的操作（`print $3,$9` 意为打印第 3 个和第 9 个字段）
>
> 有一个空行的原因是因为 `ll Desktop` 输出的统计结果，并没有第三个和第九个字段，所以占了一行。

### 3.2 模式类型

在 awk 中有下列几种模式：

> 1. 正则表达式
> 2. 关系表达式
> 3. 组合的 Pattern
> 4. Pattern1,Pattern2
> 5. BEGIN
> 6. END

#### 1. 正则表达式

模式的类型可以为正则表达式，正则表达式的规则需要写在 `//` 中。

比如我们要查询 `Desktop` 目录下包含 `ge` 或者 `gv` 的记录，可以使用如下命令：

```bash
$ ll Desktop|awk '/g[ev]/{print $0}' 
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/596222/1512927220377.png-wm)

> `/g[ev]/` 为 awk 程序指令中的 pattern（模式），这里的模式为正则表达式。

`~` 表示与正则表达式匹配（可以省略 `~` 号），`!~` 表示与正则表达式不匹配。

```bash
#查询 Desktop 目录下文件名包含 ge 和 gv 的记录
$ ll Desktop|awk '$9~/g[ev]/{print $0}' 

#查询 Desktop 目录下文件名不包含 ge 和 gv 的记录
$ ll Desktop|awk '$9!~/g[ev]/{print $0}'
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/596222/1512932338182.png-wm)

#### 2. 关系表达式

模式的类型可以为关系表达式。

下表列出了关系运算符

| 运算符 | 含义     |
| ------ | -------- |
| <      | 小于     |
| <=     | 小于等于 |
| ==     | 等于     |
| !=     | 不等于   |
| >=     | 大于等于 |
| >      | 大于     |

比如我们要查询 `Desktop` 目录下大于 800 字节的文件或者目录，可以使用如下命令：

```bash
$ ls -l Desktop|awk '$5>800{print $0}'
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/596222/1512929988428.png-wm)

> 这里的模式 `$5>800` 为关系表达式。

#### 3. 组合的模式

逻辑运算符 `||`（或）、`&&`（与）以及 `!`（非）将模式组合，组合后如果求值为真则模式匹配，否则不匹配。

比如我们要查询 `Desktop` 目录下大于 800 字节小于 8000 字节的文件或者目录，可以使用如下命令：

```bash
$ ls -l Desktop|awk '$5 > 800 && $5 < 8000{print $0}' 
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/596222/1512930151218.png-wm)

#### 4. pattern1，pattern2

以 `,`（逗号）隔开的两个 Pattern（模式），为匹配位置指定了一个范围，对从匹配第一个 Pattern 的记录开始，到匹配第二个 Pattern 结束的所有记录执行 Action（操作）。

比如匹配 `Desktop` 目录下等于 179 字节的文件和目录，直到等于 767 字节的文件和目录。可以使用如下命令：

```bash
$ ls -l Desktop|awk '$5==179,$5==767{print $0}' 
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/596222/1512932703116.png-wm)

#### 5. BEGIN

`BEGIN` 模式是 `awk` 的一种特殊的 `Pattern`，`BEGIN` 模式指定的操作是在读取任何输入之前执行，且只执行一次。使用 `BEGIN` 模式甚至不需要指定输入文件，一般把与数据文件内容无关以及只需要执行一次的部分置于以 `BEGIN` 模式为 `Pattern` 的 `Action` 中。

例如：

```bash
$ ls -l Desktop|awk 'BEGIN{print "1 2 3 4 5 6 7 8 9"}{print}' 
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/596222/1512934541100.png-wm)

> 在打印命令 `ls -l Desktop` 的结果之前先打印出了 “1 2 3 4 5 6 7 8 9”。
>
> 通常可以用来给输出结果添加表头。

#### 6. END

`END` 模式也是 `awk` 中一种特殊的 `Pattern`，`END` 模式指定的操作是在读取所有的输入后执行。

例如：

```bash
$ ls -l Desktop|awk '{print}END{print "finish---------"}' 
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/596222/1512934908870.png-wm)

> 在打印命令 `ls -l Desktop` 的结果之后打印出了 ”finish---------“。

awk 的模式操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week2/8-1.mp4
@`

## 4. awk 命令基本选项

```bash
awk [-F fs] [-v var=value] [-f prog-file | 'program text'] [file...]
```

| 选项 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| `-F` | 预先指定字段分隔符（默认字段分割符是空格）                   |
| `-f` | 指定 awk 命令要执行的程序文件，或者在不加 -f 参数的情况直接将程序语句放在这里。 |
| `-v` | 预先为 awk 程序指定变量                                      |

应用实例：

- `-F`

`awk` 处理文本的方式，是读取每一行，然后将记录分割成一些`字段`，然后再对这些字段进行处理，默认情况下，`awk` 以空格作为一个字段的分割符，不过这不是固定的，你可以使用 `-F ` 任意指定分隔符，下面将告诉你如何做到这一点。

比如我们要以 `:`（冒号）作为一个字段的分隔符。则可以进行如下操作：

```bash
#以冒号为分割符
$ awk -F: '{print $1}' /etc/passwd
 
或者
$ awk 'BEGIN{FS=":"}{print $1}' /etc/passwd
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1513067277892.png/wm)

- `-f`

新建一个名叫 testf 的文件，实现用 awk 打印出 hello shiyanlou：

```bash
$ vim testf
```

```
BEGIN{print "hello shiyanlou"}
```

我们要执行这个 awk 命令，就要加上 `-f` 选项，后面跟上文件名：

```bash
$ awk -f testf
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/testf
  error: /home/shiyanlou 下没有 testf 文件
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/596222/1512935977696.png-wm)

> 实际上就是和 `awk 'BEGIN{print "hello shiyanlou"}'` 的执行结果是一样。只是把单引号里面的代码放入了一个文件中。
>
> 当我们的 awk 操作过多的时候，就可以把所有的操作都写在一个文件中。然后用 `awk -f 文件名` 的格式去执行。

- `-v`

```bash
$ var=100
$ echo |awk -v variable=$var '{print variable}'

# 输出结果为：100
```

> 将外部变量 `var`  的值传入了 awk 里的 `varable` 变量。

awk 命令基本选项操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week2/8-2.mp4
@`

## 5. awk 内置变量

`awk` 中有许多系统变量或内置变量。`awk` 有两种类型的系统变量：
    - 第一种内置变量的类型用于 awk 的控制，例如分隔符等。
    - 第二种内置变量的类型用于信息的传达，例如当前记录中字段的数量，当前记录的数量等。

### 5.1 awk 常用的内置变量

| 变量名     | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| `FILENAME` | 当前输入文件名，若有多个文件，则只表示第一个。如果输入是来自标准输入，则为空字符串 |
| `$0`       | 当前记录的内容                                               |
| `NF`       | 当前记录字段数，列数                                         |
| `$N`       | N 表示字段号，最大值为 `NF` 变量的值                         |
| `FS`       | 字段分隔符，默认为" "空格                                    |
| `RS`       | 输入记录分隔符，默认为"\n"，即一行为一个记录                 |
| `NR`       | 已经读入的记录数，行数                                       |
| `FNR`      | 当前输入文件的记录数，请注意它与 NR 的区别。 对于 NR，读取不同文件，NR 是一直累计的。 但是对于 FNR，读取不同文件，开始下一个文件的时候 FNR 又从 1 开始了。在流程控制一节我们以实例讲解它们之间的区别 |
| `OFS`      | 输出字段分隔符，默认为" "空格                                |
| `ORS`      | 输出记录分隔符，默认为"\n"                                   |
| `ARGC`     | 命令行参数的数目                                             |
| `ARGIND`   | 命令行中当前文件的位置（从0到ARGC-1）                        |
| `ARGV`     | 命令行参数的数组                                             |

我们来通过下面的方法来输出常用的内置变量的值，更好地理解常用的几个内置变量的含义。

新建一个测试文本 test.txt，输入如下内容：

```
hello this is louplus
welcome to shiyanlou
this's a test file
would you like something to drink coffee,tea,or coco-cola
```

新建一个脚本文件 test.sh，输入如下内容：

```bash

# 使用默认的分隔符切分，打印各个变量的值
awk 'BEGIN{printf "%4s %4s %4s %4s %4s %4s %4s %4s %4s\n","FILENAME","ARGC","FNR","FS","NF","NR","OFS","ORS","RS";printf "---------------------------------------------\n"} {printf "%4s %4s %4s %4s %4s %4s %4s %4s %4s\n",FILENAME,ARGC,FNR,FS,NF,NR,OFS,ORS,RS}'  test.txt

# 使用 ' 单引号作为分隔符切分，打印各变量的值
awk -F\' 'BEGIN{printf "%4s %4s %4s %4s %4s %4s %4s %4s %4s\n","FILENAME","ARGC","FNR","FS","NF","NR","OFS","ORS","RS";printf "---------------------------------------------\n"} {printf "%4s %4s %4s %4s %4s %4s %4s %4s %4s\n",FILENAME,ARGC,FNR,FS,NF,NR,OFS,ORS,RS}'  test.txt

awk '{print NR,FNR,$1,$2,$3}' test.txt

# 修改分隔符
awk '{print $1,$2,$5}' OFS=" $ "  test.txt
```

> 将会在后面深入讲解 printf 格式化输出的使用方式

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/test.txt
  error: /home/shiyanlou 下没有 test.txt 文件
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/test.sh
  error: /home/shiyanlou 下没有 test.sh 文件
```
执行

```bash
$ bash test.sh
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971511330473007-wm)

## 6. awk 应用实例

学习了 awk 的基础内容后，我们下面再来做几个应用实例。

### 6.1 分隔符的修改

awk 可以使用 `print` 打印文件内容。

先用 vim 新建一个文本文档

```bash
$ vim test
```

包含如下内容：

```
I love linux
www.shiyanlou.com
```

- 将 test 的每个字段单独显示为一行(前面的 > 不用输入)

```bash
$ awk '{
> print $1 "\n" $2 "\n" $3
> }' test

# 或者
$ awk '{
> OFS="\n"
> print $1, $2, $3
> }' test
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/test
  error: /home/shiyanlou 下没有 test 文件
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1513071257406.png/wm)

>  注意 `OFS` 是 `awk` 内建的变量。
>  - `OFS` 表示输出时的字段分隔符，默认为`" "`（空格）。
>
>  如上图所见，我们将字段分隔符设置为`\n`换行符，所以第一行原本以空格为字段分隔的内容就分别输出到单独一行了。
>
>  然后是`$N` 其中 `N` 为相应的字段号，这也是 `awk` 的内建变量，它表示引用相应的字段，因为我们这里第一行只有三个字段，所以只引用到了`$3`。除此之外另一个这里没有出现的`$0`，它表示引用当前记录（当前行）的全部内容。

- 将 test 的第二行的记录以点为分段的字段换成以空格为分隔

```bash
$ awk -F'.' '{
> print $1 "\t" $2 "\t" $3
> }' test

# 或者
$ awk '
> BEGIN{
> FS="."
> OFS="\t"  # 如果写为一行，两个动作语句之间应该以";"号分开  
> }{
> print $1, $2, $3
> }' test
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1513071536492.png/wm)

> 说明：这里的 `-F` 参数，前面已经介绍过，它是用来预先指定待处理记录的字段分隔符。
>
> 我们需要注意的是除了指定 `OFS ` 我们还可以在 `print`  语句中直接打印特殊符号,如这里的 `\t`。
>
> 在第二种方法中展示了预先指定分隔符的另一种方式，在所有的操作之前,设置了 `FS` 以 "." 代替默认的" "空格

**注意：**首先说明一点，我们在学习和使用 awk 的时候应该尽可能将其作为一门程序语言来理解，这样将会使你学习起来更容易，分多行、有一定的缩进输入，而不是全部写到一行，这样易于调试与查找问题。


### 6.2 计算文件总大小

下面的实验带领大家用 `awk` 实现统计 `/home/shiyanlou` 目录下文件的总大小。

我们先新建一个 num 文件

```bash
$ touch num
```

查看一下当前目录下的内容

```bash
$ ls -l
```

我们来统计文件数量，修改 num 为如下：

```bash
BEGIN{
    print "BYTES","\t","FILE"
}   #增加列标题
{
    sum+=$5
    filenum++
    print $5,"\t",$9
}
END{
    print "Total:",sum,"bytes("filenum-1 "files)"
}  #打印总数
```

> `sum` 是文件总大小，`filenum` 是文件个数。
> filenum 需要减去 1，是因为 `ls -l` 的时候会有一行是显示统计信息的，并不需要算进去。

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/num
  error: /home/shiyanlou 下没有 num 文件
```
执行

```bash
$ ls -l | awk -f num
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1513072877758.png/wm)

我们修改一下 `num` 使它只统计文件的数量和总大小（开头为 `-` 的就是文件），不包含目录，以及能统计目录下的子文件的数量和总大小。

```bash
BEGIN{
    print "BYTES","\t","FILE"
}

# 测试9个字段，文件类型是文件并且以 - 开头）
NF == 9 && /^-/{
    sum+=$5   #累计文件大小
    filenum++  #系统文件个数
    print $5,"\t",$9
}

# 测试9个字段，文件类型是目录（以 d 开头）
NF == 9 && /^d/{
    print "<dir>","\t",$9
}

END{
    print "Total:",sum,"bytes("filenum "files)"
}
```

> 这里我们使用两个模式，一个是当前记录有 9 个字段，一个是正则表达式。通过两个模式来判断当前记录是否为有效记录和文件的类型
>
> 文件类型是目录的话就打印出 `<dir>` 以及目录名。
>
> 文件类型是文件的话就打印出文件大小和文件名。
>
> 将 `ls -lR` 改成 `ls -l` 就不统计子文件的。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1513074294850.png/wm)


## 7. 格式化打印

awk 可以使用 `printf` 对输出进行格式化，以整齐的样式打印输出的内容。

#### 1. 语法及字符

语法：

```bash
printf ( format-expression [, arguments])
```

其中圆括号是可选的。`format-expression` 是一个用来描述格式的表达式，通常是以引号括起的字符串常量的形式。方括号里是一个参数列表，例如变量名列表。

格式化规定字符以 `%` 开始，后跟一个或几个规定字符，用来确定输出内容格式。

格式说明符号如下表，最常用的是 `s` ，`d`，`f`：

| 字符 | 含义                                         |
| ---- | -------------------------------------------- |
| c    | ASCII字符                                    |
| d    | 十进制整数                                   |
| i    | 十进制整数（在POSIX中增加的）                |
| e    | 浮点格式                                     |
| E    | 浮点格式                                     |
| f    | 浮点格式                                     |
| g    | e 或 f 的转换形式，长度最短，末尾的 0 被去掉 |
| G    | E 或 f 的转换形式，长度最短，末尾的 0 被去掉 |
| O    | 无符号的八进制                               |
| s    | 字符串                                       |
| u    | 无符号的十进制                               |
| x    | 无符号的十六进制，用 a-f 表示 10-15          |
| X    | 无符号的十六进制，用 A-F 表示 10-15          |
| %    | 字面字符 %                                   |

#### 2. 格式化打印实例

我们在统计文件大小的实例中，并没有对输出的内容进行格式化，我们这里对输出内容进行格式化，以解决各个字段和标题的对齐问题：

把 num 修改为如下：

```bash
ls -l $*|awk '
BEGIN{
    print "BYTES","\t","FILE"
}

NF == 9 && /^-/{
    sum+=$5
    ++filenum
    printf("%d\t%s\n",$5,$9)   #修改的地方，修改成printf
}

NF == 9 && /^d/{
    printf("<dir>\t%s\n",$9)
}

END{
    print "Total:",sum,"bytes("filenum "files)"
}'
```

> `$5` 是第 5 列的内容，也就是文件大小，格式化为十进制整数，所以用 `%d` 。
>
> `$9` 是第 9 列的内容，也就是文件名，它是字符串，所以用 `%s` 。

```checker
- name: check file
  script: |
    #!/bin/bash
    grep filenum /home/shiyanlou/num
  error: 没有修改 num 文件
```
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1513075945683.png/wm)

#### 3. 宽度和对齐方式

还可以用 `printf` 规定输出域的宽度和对齐方式。

语法：

```
%-width.precision format-specifier

```

> - `width` 宽度是一个数值。默认是右对齐。指定 - 可以设置左对齐。 
>
>
> - `precisinon` 精度，用于十进制或浮点数，控制小数点右边的位数。对于字符串型值，用于控制要打印的字符的最大数量。默认为 `%.6g` ，表示只打印小数部分的前六位。可以通过设置系统变量 `OFMT` 改变这个默认值。

如下输入：

```bash
$ echo | awk '{printf("|%10s|\n","hello")}'
$ echo | awk '{printf("|%-10s|\n","hello")}'
```

> 第一行是设置宽度为 10，右对齐。
>
> 第二行是设置宽度为 10，左对齐。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1513076316594.png/wm)

对于之前 num 统计脚本我们想要实现如下功能：

- 让文件名在左，大小在右
- 文件名左对齐，宽度 8，大小右对齐，宽度 5
- 把 print 的打印方式改成 printf 形式

可以根据前面学的内容想想如何实现。具体实现步骤如下：

修改 num 的内容为如下：

```bash
BEGIN{
    printf("|%-8s|\t|%5s|\n","FILE","BYTES")
}  #修改的行

NF == 9 && /^-/{
    sum+=$5
    ++filenum
    printf("|%-8s|\t|%5d|\n",$9,$5)  #修改的行。文件名左对齐，宽度 8，大小右对齐，宽度 5
}

NF == 9 && /^d/{
    printf("|%-8s|\t|%5s|\n",$9,"<dir>")  #修改的行。让文件名在左，大小在右。
}

END{
    printf("Total: %d bytes(%d files)\n",sum,filenum)
}  #修改的行
```

```checker
- name: check file
  script: |
    #!/bin/bash
    grep "|%" /home/shiyanlou/num
  error: 没有修改 num 文件
```

执行输出如下:

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid113508labid1timestamp1513076593818.png/wm)

awk 格式化打印操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week2/8-3.mp4
@`

## 8. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎与我们交流：

- awk 的作用
- awk 的模式
- awk 基本选项
- awk 变量
- 格式化打印 printf