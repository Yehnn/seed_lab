---
show: step
version: 0.1
enable_checker: true
---

# Grep 文本查找和 Sed 文本编辑

## 1. 实验介绍

#### 1.1 实验内容

本节实验带领大家学习正则表达式，以及 grep 和 sed 的初级使用。

#### 1.2 实验知识点

- 正则表达式
- 字符
- grep 匹配字符
- sed 匹配字符

#### 1.3 推荐阅读

这里推荐用于测试正则表达式的在线工具，帮助初始学习的验证：

- http://tools.jb51.net/regex/create_reg 正则表达式在线生成工具
- http://tool.oschina.net/regex/ 正则表达式在线测试工具

sed 的推荐阅读：

- [sed简明教程](http://coolshell.cn/articles/9104.html)
- [sed单行脚本快速参考](http://sed.sourceforge.net/sed1line_zh-CN.html)
- [sed完全手册](http://www.gnu.org/software/sed/manual/sed.html)

## 2. 正则表达式

#### 2.1 什么是正则表达式

简单来说，正则表达式就是处理字符串的方法，以字符、行、单词、行列等为单位进行字符串的处理，通过一些特殊符号和支持的工具程序来辅助，就可以让用户轻松搜索、替换、过滤、剥离某些特定的字符串。

#### 2.2 基本语法

正则表达式通常被称为一个模式（**pattern**），用来描述或者匹配一系列符合某个句法规则的字符串。

#### 语系对正则表达式的影响

由于不同语系的编码数据不同，所以造成不同语系的数据选取结果有所差异。以英文大小写为例，zh_CN.big5 及 C 这两种语系差异如下：

- LANG=C 时： 0 1 2 3 4....ABCDE...Zabcde...z
- LANG=zh_CN 时：0 1 2 3 4...aAbBcCdD.....zZ

在使用正则表达式 [A-Z]， LANG=C 的情况时，找到的仅仅是大写字符 ABCD..Z。而在 LANG=zh_CN 情况下，会选取到 AbBcCdD.....zZ 字符。因此在使用正则表达式时要特别留意语系。

由于我们一般使用的兼容与 POSIX 的标准，因此使用 C 语系。

#### 2.3 正则表达式分类

1、基本的正则表达式（`Basic Regular Expression` 或叫 `Basic RegEx` ，简称 `BREs`）

2、扩展的正则表达式（`Extended Regular Expression` 或叫 `Extended RegEx` ，简称 `EREs`）

3、Perl 的正则表达式（`Perl Regular Expression` 或叫 `Perl RegEx` ，简称 `PREs`）

## 3. 字符

#### 3.1 普通字符

普通字符就是大部分比较简单的字符串构成，由数字（0-9），字母（a-z, A-Z）等组成。如：shiyanlou001。

我们并不需要去记忆哪些字符是普通字符，只需要知道哪些字符是特殊字符就可以，除了特殊字符之外的所有字符都是普通字符。

#### 3.2 特殊字符

正则表达式除了进行字符自身值的匹配外，还可以基于指定的规则进行模糊匹配。这就意味着需要一些特殊字符来表示这样模糊的匹配规则，因此这些特殊字符默认情况下并不能匹配到它们自身的字面值，而是表示某些特殊的功能。在后面我们将学习到特殊字符的具体使用形式。

#### 3.3 优先级

优先级为从上到下从左到右，依次降低：

| 运算符                      | 说明         |
| --------------------------- | ------------ |
| ` \ `                       | 转义符       |
| `(), (?:), (?=), [] `       | 括号和中括号 |
| `*、+、?、{n}、{n,}、{n,m}` | 限定符       |
| ` ^、$、\任何元字符 `       | 定位点和序列 |
| `｜ `                       | 选择         |

## 4. grep 匹配字符

正则表达式是处理字符串的一种规则的表述，需要通过一些支持的工具程序来辅助才能看到规则的效果，而 `grep` 就是常见的一种支持工具。下面我们一起来学习如何使用 grep 完成字符的匹配。

### 4.1 grep 基本操作


`grep` 命令用于打印输出文本中匹配的模式串，使用正则表达式作为匹配的条件。其中，`grep` 支持三种正则表达式引擎，分别用三个参数指定：

| 参数 | 说明                      |
| ---- | ------------------------- |
| `-G` | POSIX 基本正则表达式，BRE |
| `-E` | POSIX 扩展正则表达式，ERE |
| `-P` | Perl 正则表达式，PCRE     |

> 在没学过 perl 语言的情况下，大部分情况只使用 `ERE` 和 `BRE`，所以接下来的内容将不会讨论 PCRE 中特有的正则表达式语法。

语法

```
grep [参数] PATTERN [file]
```
在每个 file 或标准输入中查找 pattern 。默认的 pattern 是一个基本正则表达式（BRE）。

常用参数

| 参数   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| `-c`   | 只打印每个 file 中的匹配行数目                               |
| `-i`   | 忽略大小写                                                   |
| `-n`   | 输出的同时打印行号                                           |
| `-v`   | 反选，输出不匹配行的内容                                     |
| `-r`   | 递归匹配查找                                                 |
| `-R`   | 递归匹配查找，但会遍历所有符号链接                           |
| `-A n` | n 为正整数，表示 after ，除了列出匹配行之外，还列出后面的 n 行 |
| `-B n` | n 为正整数，表示 before ，除了列出匹配行之外，还列出前面的 n 行 |


既然 `grep` 作为支持正则表达式进行字符串操作的工具之一，那么下面我们通过几个 `grep` 常见的参数操作来熟悉下这个工具，之后再对正则表达式结合 grep 工具进行字符匹配进行探讨。

**实例**

eg 1：使用参数 `-R` 递归查找满足条件的文件

```
$ grep -R sh /etc/passwd
```

eg 2:使用参数 `-n` 输出查找内容与其行号

```
$ grep -n sh /etc/passwd
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512836788424.png/wm)

eg 3：使用参数 `-i` 忽略大小写输出内容

```
$ grep -i SH /etc/passwd
```

`/etc/passwd` 这个文件中只有小写的 `sh` ，但是输入参数 `-i`自动忽略了大小写。

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512837160944.png/wm)

eg 4：使用参数 `-v` 屏蔽匹配的相关行，输出内容

```
$ grep -v usr /etc/passwd
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512837168848.png/wm)

eg 5：使用参数 `-B n` 输出满足条件的内容前 5 行内容：

```
$ grep -B 5 shiyanlou /etc/passwd
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512837621130.png/wm)


### 4.2 grep 匹配字符

进行匹配前我们先建立一个 txt 测试文档，内容如下，可以使用实验桌面右边工具栏的剪切板拷贝到实验环境中：

```
$ vim blankspace.txt
```
```
Nice to meet you p2p
Where you've been?
I can show you incredible things
Magic, madness, heaven, (sins)
Saw you there and I thought oh my_god
Look at that face, you look like my next mistake404
Love's a game, wanna play
New money, suit and tie
I can read you like a magazine
Ain't it funny rumors fly
And I know you've heard about me
So hey, let's be friends
And I'm dying to see how this one ends
Grab your passport and my hand
I can make the bad guys good for a weekend

```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/blankspace.txt
  error: /home/shiyanlou 目录下没有 blankspace.txt
```

**特殊字符**

前面我们知道在正则表达式中主要就是通过对定义的字符进行逻辑过滤，下面我们通过实践的形式来学习正则表达式中使用特殊字符的规则编写。（对于普通字符，我们只需写出所需的字符即可做到匹配），对于特殊字符我们将其分为 5 大类来分别讲解：

**1.字符匹配**

| 字符     | 说明                                                         | 实例                                                      |
| -------- | ------------------------------------------------------------ | --------------------------------------------------------- |
| ` . `    | **匹配除“\n”以外的任何单个字符**。若要匹配包括“\n”在内的任何字符，使用“(.｜\n)”的模式 | 例如：`l.u`可以匹配：lou、liu、l7u 等                     |
| ` \   `  | **转义字符，将下一个字符标记为特殊字符、或原义字符。**       | 例如：`\\`匹配`\`，`\(` 匹配 `(`                          |
| `[...]`  | **匹配指定范围内的任意字符。**                               | 例如，`[a-z]`可以匹配 `a` 到 `z` 范围内的任意小写字母字符 |
| `[^...]` | **匹配任何不在指定范围内的任意字符**                         | 例如，`[^a-z]` 匹配任何不在 `a` 到 `z` 范围内的任意字符。 |
| ` \d `   | **匹配 0-9 的数字**，相当于`[0-9]`                           | 例如，`a\df` 可以匹配：a1f、a2f、a7f                      |
| `\D`     | **匹配除了 0-9 的任意字符**                                  | 例如，`a\df` 可以匹配：aaf、asf、adf                      |
| ` \w  `  | **匹配单个数字或字符或下划线（\_）,相当于 [0-9a-zA-Z_]**     | 例如，`a\wf` 可以匹配：aaf、a_f、a3f                      |
| ` \W  `  | **匹配非数字或字符或下划线（\_）,相当于[^0-9a-zA-Z_]**       | 例如，`a\Wf` 可以匹配：a f、a&f、a@f                      |
| `\s  `   | **匹配一个空白字符**                                         | 例如，`a\sf` 可以匹配：a f                                |
| ` \S  `  | **匹配非空字符,相当于[^\s]**                                 | 例如，`a\Sf` 可以匹配：aaf、a1f、a@f                      |

**实例**

eg 1：匹配字符 `.`

```
$ grep -n 'c.n' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512843788769.png/wm)

eg 2：匹配字符 `[...]`

```
$ grep -n '[vr]' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512843808961.png/wm)

eg 3：匹配字符 `\d`  

```
$ grep -n 'p\dp' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512843824702.png/wm)

这里无论是 `\d` 还是 `\D` 都不会有输出，这是因为 BRE(以及 ERE)均不支持。

eg 4：匹配字符 `\w`

```
$ grep -n 'y\wg' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512843978092.png/wm)

eg 5：匹配字符 `\s ` 

```
$ grep -n 'n\sr' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512843991221.png/wm)

**2.数量匹配**

| 量词     | 说明                                                         | 实例                                                         |
| -------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `{n}`    | **匹配确定的 n 次**，其中 n 为非负整数                       | 例如，`o{2}`不能匹配 `Bob`，但能匹配 `food`                  |
| `*`      | **匹配前面的子表达式零次或多次**                             | 例如，`zo*` 匹配 `z`、`zo`、`zoo`。等价于 {0,}               |
| `+`      | **匹配前面的子表达式一次或多次**                             | 例如，`zo+` 能匹配 `zo` 、`zoo`，但不能匹配 `z`。等价于 {1,}。 |
| `?`      | **匹配前面的子表达式零次或一次**                             | 例如，`do(es)?` 可以匹配 `do` 或 `does`。等价于 {0,1}        |
| ` {n,} ` | **至少匹配n次**，其中 n 为非负整数                           | 例如，`o{2,}`不能匹配 `Bob` 中的 `o`，但能匹配 `foooood` 中所有的 `o`。`o{1,}` 等价于 `o+`。`o{0,}`则等价于`o*` |
| `{n,m} ` | **最少匹配 n 次且最多匹配 m 次。** 其中，m 和 n 均为非负整数，且 `n<=m ` | 例如，`o{1,3}` 将匹配 `fooooood`中的前三个 o。`o{0,1}`等价于 ` o?`。**注意在逗号和两个数之间不能有空格** |


**实例**

>注意：数量匹配的功能在 BRE 中是不支持的，所以在下列实例中看不到效果，但是在 ERE 中是支持的，我们会在下文中详细讲解 ERE 中得到验证。

eg 1：匹配字符 `*` 

```
$ grep -n 'lo*' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512844252912.png/wm)

eg 2：匹配字符 `+` 

```
$ grep -n 'lo+' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512844233316.png/wm)

eg 3：匹配字符 `?`

```
$ grep -n 'lo(ok)?' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512844310712.png/wm)

eg 4：匹配字符 `{n} ` 

```
$ grep -n 'o{2}' blankspace.txt
```

eg 5：匹配字符 `{n,m} `

```
$ grep -n 'o{1,3}' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512844325766.png/wm)


上面四种情况不会有输出，也是因为 BRE 不支持。

**3.位置匹配**

| 限定符 | 说明                           | 实例                                                         |
| ------ | ------------------------------ | ------------------------------------------------------------ |
| ` ^ `  | **匹配输入字符串的开始位置。** | 例如，`^shi` 可以匹配 `shi`、 `shiyan`、`shiyanloulou`       |
| `$`    | **匹配输入字符串的结束位置。** | 例如，`shi$` 可以匹配 `shi`、 `yanshi`、`yanloushi`          |
| `\b`   | **匹配单词的边界位置。**       | 例如，`er\b` 可以匹配 `never` 中的 `er`，但不能匹配 `verb` 中的 `er` |
| `\B`   | **匹配不是单词边界的位置**     | 例如，`er\B` 能匹配 `verb` 中的`er`，但不能匹配 `never`      |


**实例**

eg 1：匹配字符 ` ^ `

```
$ grep -n '^Lo' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512844570456.png/wm)

eg 2：匹配字符 `$`

```
$ grep -n 'd$' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512844580293.png/wm)

eg 3：匹配字符 `\b`

```
$ grep -n 's\b' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512844589567.png/wm)

eg 4：匹配字符 `\B`

```
$ grep -n 's\B' blankspace.txt
```
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512844596175.png/wm)

**4.逻辑和分组匹配**

| 字符        | 说明                                                         | 实例                                                         |
| ----------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `x｜y `     | **逻辑或，表示任意匹配 x 或 y**                              | 例如， `F｜food ` 能匹配  `F `或 `food `。 `(F｜f)ood ` 则匹配 `Food ` 或 `food `。 |
| `()`        | **匹配一个子表达式，相当于一个分组，通常可以指定这个分组的重复次数** | 例如，`F(oo){2}d `，匹配的是 `Fooood `                       |
| `(?<NAME>)` | **同样是匹配一个子表达式，不过还指定了分组的名字**           | 例如，`F(?<ID>oo){2}d `，匹配的也是 `Fooood `                |

逻辑和分组这里的三个字符在 BRE 中都不支持。

**5.其他字符说明**

| 符号        | 说明                                           |
| ----------- | ---------------------------------------------- |
| `[:alnum:]` | 表示所有十进制数字和英文字母，即 0-9, A-Z, a-z |
| `[:alpha:]` | 表示所有英文字母，即 A-Z, a-z                  |
| `[:lower:]` | 表示所有小写字母，即 a-z                       |
| `[:upper:]` | 表示所有大写字母，即 A-Z                       |
| `[:digit:]` | 表示所有数字，即 0-9                           |
| `[:blank:]` | 表示空格键和 [Tab] 键                          |
| `[:cntrl:]` | 表示键盘上的控制键，即 Tab, Delete 等          |
| `[:graph:]`  | 包含 `[:alnum:]`,`[:punct:]` 
| `[:print:]`  | 表示任何可以被输出的字符                          |
| `[:punct:]`  | 表示标点符号，即：" ' ? ! ; : # `$`...              |
| `[:space:]`  | 表示任何会产生空白的字符，包括：空白键, [Tab]等   |
| `[:xdigit:]` | 表示 16 进制的数字，即 0-9, A-F, a-f 的数字与字节 |

**实例**

eg 1：匹配字符 `[:alnum:]`

```
$ grep -n '[[:alnum:]]r' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512845125956.png/wm)

eg 2：匹配字符 `[:digit:]`

```
$ grep -n '[[:digit:]]' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512845143925.png/wm)

eg 3：匹配字符 `[:punct:]` 

```
$ grep -n '[[:alnum:]]?' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512845157582.png/wm)

eg 4：匹配字符 `[:upper:]`

```
$ grep -n '[[:upper:]]o' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512845168050.png/wm)

eg 5：匹配字符 `[:xdigit:]`

```
$ grep -n '[[:xdigit:]]b' blankspace.txt
```
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512845176406.png/wm)

grep 匹配字符操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week2/6-1.mp4
@`

### 4.3 扩展正则表达式（ERE）

前面我们学习了基础的正则表达式（BRE），但是在字符匹配的操作过程中我们发现有一些是 `BRE` 无法进行匹配操作的，这时我们可以尝试下扩展的正则表达式（ERE）能否实现。

通过 `grep` 使用扩展正则表达式需要加上 `-E` 参数，或使用 `egrep` 即可。

```
grep -E 

egrep 
```

**扩展规则**

#### 1. 量词匹配   

之前量词匹配的验证失败，我们将在这里得到验证：

eg 1：匹配字符 `+`
```
$ egrep -n 'lo+' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512847050280.png/wm)

eg 2：匹配字符 `?`

```
$ egrep -n 'an(n)?' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512847061014.png/wm)

eg 3：匹配字符 `{n} ` 

```
$ egrep -n 'o{2}' blankspace.txt
```
eg 4：匹配字符 `{n,m} `

```
$ egrep -n 'o{1,3}' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512847105624.png/wm)

#### 2. 逻辑与分组匹配

eg 1：匹配字符 `|`

```bash
$ egrep -n 'ss|oo' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512847117527.png/wm)

eg 2：匹配字符 `()` 

```bash
$ egrep -n 'Lo(ve|ok)' blankspace.txt

```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512847125446.png/wm)


grep ERE 匹配操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week2/6-2.mp4
@`


## 5 sed 匹配字符

在正则表达式的支持工具中除了 `grep` 可以用来搜索字符，还有 `sed` 和 `awk` 也可以完成这项操作，这节实验我们先简单介绍下 `sed` 匹配字符的操作，在后续课程中将继续给大家讲解 `sed` 和 `awk` 的更多用法。

sed 的全称是 stream editor，也就是流编辑器。其作用便是将文本文件或来自于管道符传入的输入流做文本的处理，如替换、增加内容、删除内容等等。

sed 命令基本格式：

```bash
sed [参数]... [执行命令] [输入文件]...

```

常用的参数：

| 参数          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| `-n`          | 安静模式，只打印受影响的行，默认打印输入数据的全部内容       |
| `-e`          | 用于在脚本中添加多个执行命令一次执行，在命令行中执行多个命令通常不需要加该参数 |
| `-f filename` | 指定执行 filename 文件中的命令                               |
| `-r`          | 使用扩展正则表达式，默认为标准正则表达式                     |
| `-i`          | 将直接修改输入文件内容，而不是打印到标准输出设备             |

执行命令的格式：

```bash
[n1][,n2]command

[n1][~step]command

```

其中 n1,n2 表示输入内容的行号，它们之间为 `,`，如果为`～`波浪号则表示从 n1 开始以 step 为步进的所有行；command为执行动作，下面为一些常用动作指令：

| 命令 | 说明                                 |
| ---- | ------------------------------------ |
| `s`  | 行内替换                             |
| `c`  | 整行替换                             |
| `a`  | 插入到指定行的后面                   |
| `i`  | 插入到指定行的前面                   |
| `p`  | 打印指定行，通常与 `-n` 参数配合使用 |
| `d`  | 删除指定行                           |

### 5.1 sed 使用实例

#### 1. 删除指定行

为了让效果明显地看到，我们使用 `nl` 命令将 blankspace.txt 的内容与打印行号同时打印出来，同时利用 sed 将 2-5 行删除显示：


```bash
$ nl blankspace.txt | sed '2,5d'
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512849854464.png/wm)

>  命令解释：'2,5d' 表示 2~5 行，d 表示删除。

删除第13行到最后一行, `$` 定位到最后一行：

```bash
$ nl blankspace.txt | sed '13,$d'
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512849879730.png/wm)

当然之前的操作都没有加 `-i` 参数，所以只是展示了效果但是没有修改原文，若是要在原文件中删除第 1 行：

```bash
$ sed -i '1d' blankspace.txt
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512849892014.png/wm)

#### 2. 添加字符串

在第二行后添加 test 字符串，`a`表示在行后添加一行加上字符串，`i` 表示在行前加一行添加字符串

例如：

```bash
$ nl blankspace.txt | sed '2a test'
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512849910350.png/wm)

在第二行前添加一行插入 `test` 字符串：

```bash
$ nl blankspace.txt | sed '2i test'
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512849917735.png/wm)

#### 3. 替换字符串

将 2-5 行内容取代为 `blankspace`

c 为替换内容选项。

```bash
$ nl blankspace.txt | sed '2,5c blankspace'
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512849930522.png/wm)

#### 4. 列出匹配字符

列出 `blankspace.txt` 内第 5-7 行

sed 命令中 `-n` 为安静模式选项。以下两条命令执行结束后可对比结果。

```bash
$ nl blankspace.txt |sed -n '5,7p'

$ nl blankspace.txt |sed  '5,7p'
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4130timestamp1512849937815.png/wm)

我们将会在下一节中做 sed 的进一步扩展，当然如果希望了解更多 sed 的高级用法，推荐阅读如下指南：

- [sed简明教程](http://coolshell.cn/articles/9104.html)
- [sed单行脚本快速参考](http://sed.sourceforge.net/sed1line_zh-CN.html)
- [sed完全手册](http://www.gnu.org/software/sed/manual/sed.html)

sed 匹配字符操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week2/6-3.mp4
@`

## 6 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎与我们交流：

- 正则表达式的基本语法
- 正则表达式的特殊字符有哪些
- grep 常用的参数有哪些
- grep 怎么使用扩展表达式
- sed 常用的参数有哪些