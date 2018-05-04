---
show: step
version: 0.1
enable_checker: true
---

##Linux Capability探索实验

###一、实验描述

本实验中，学生将感受到linux capability功能在访问控制上的优势，掌握使用Capability达到遵守最小权限原则的目的，并分析linux中基于Capability访问控制的设计。

###二、环境搭建

在进行实验之前，首先需要做一些准备工作。

####2.1 下载Libcap

`libcap` 库能够使用户级别的程序与 `capability` 特性做交互，一些linux发行版不包括这个库，在环境中已经有 `/usr/include/sys/capability.h` 这个文件，为了避免老版本的影响，我们还是删掉以前的，然后重新下载一个。

```bash
$ cd
$ wget http://labfile.oss.aliyuncs.com/libcap-2.21.tar.gz
$ tar xvf libcap-2.21.tar.gz
$ sudo rm /usr/include/sys/capability.h
$ sudo rm /lib/libcap.so*
$ sudo make
$ sudo make install
```

在本实验中，你需要熟悉以下命令：

+ `setcap`: 给一个文件分配capabilities
+ `getcap`: 显示文件所带的capabilities
+ `getpcaps`: 显示线程所在的capabilities

### 三、 实验内容
在一个capability系统中，当一个程序运行时，对应的线程会初始化一系列capabilities（令牌）。当线程尝试访问某个对象时，操作系统会检查该线程的capabilities，并决定是否授权访问。

####3.1 实验1: 感受一下Capabilities
在操作系统中，有许多只允许超级用户使用的操作，比如配置网卡，备份所有用户文件，关闭计算机等，但如果要进行这些操作就必须先成为超级用户的话，那就违背了最小权限原则。

Set－UID程序允许用户暂时以root权限进行操作，即使程序中所进行的权限操作用不到root权限的所有权利，这很危险：因为如果程序被入侵了的话，攻击者可能得到root权限。

Capabilities将root权限分割成了权利更小的权限。小权限被称作capability。如果使用capabilities，那么攻击者最多只能得到小权限，无法得到root权限。这样，风险就被降低了。

在kernel版本2.6.24之后，Capabilities可以分配给文件（比如程序文件），线程会自带程序文件被分配到的capabilities。

下面这个例子演示capabilities如何移除root特权程序中的不必要的权利。

首先，以普通用户登录并运行以下命令：

	$ ping -c 3 www.baidu.com

![3.1-1](https://dn-simplecloud.shiyanlou.com/uid/8797/1524896224836.png-wm)

命令成功运行，如果你查看/bin/ping的属性会发现它是一个root所有的Set-UID程序。如果ping中包含漏洞，那么整个系统就可能被入侵。问题是我们是否能移除ping的这些权限。

让我们关闭程序的suid位：

	$ sudo su
	# chmod u-s /bin/ping

![3.1-2](https://dn-simplecloud.shiyanlou.com/uid/8797/1524899467538.png-wm)

**（注：1. 请先``which ping``确认ping的真正位置，2. ‘#’开头说明以root权限运行命令，普通用户下请自行在命令前加sudo）**  

现在再ping百度看会发生些什么：

```bash
$ ping www.baidu.com
ping: icmp open socket: Operation not permitted
```

它会提示你操作不被允许。这是因为ping命令需要打开RAW套接字，该操作需要root特权，这就是为什么ping是Set-UID程序了。但有了capability,我们就可以杯酒释兵权了，让我们分配cap_net_raw给ping，看看会发生什么：

	$ sudo su
	# setcap cap_net_raw=ep /bin/ping
	按ctrl + D 键回到普通用户
	$ ping -c 3 www.baidu.com

![3.1-3](https://dn-simplecloud.shiyanlou.com/uid/8797/1524900447039.png-wm)

**任务1:** 取消下列程序的Set－UID并不影响它的行为。

+ /usr/bin/passwd

  >  seed 用户的密码是 dees。

```
$ sudo su seed
$ sudo chmod u-s /usr/bin/passwd
$ passwd
$ sudo setcap cap_chown,cap_dac_override,cap_fowner=ep /usr/bin/passwd
```

效果截图如下：

![3.1-4](https://dn-simplecloud.shiyanlou.com/uid/8797/1524908149710.png-wm)

**任务2:** 现在我们来熟悉一下其它capability（下面简称cap），你需要做：

1. 解释该cap的功能
2. 找到合适的程序演示该cap的效果。

你可以通过阅读 /usr/include/linux/capability.h 文件了解这些cap。
以下是本任务用到的cap：

+ cap_dac_read_search
+ cap_dac_override
+ cap_chown
+ cap_setuid
+ cap_kill
+ cap_net_raw

(请同学独立搜索完成，这里就不做演示了哈:D)

```checker
- name: check libcap
  script: |
    #!/bin/bash
    ls /home/shiyanlou/libcap-2.21
  error: 没有下载libcap到/home/shiyanlou目录
- name: check old
  script: |
    #!/bin/bash
    ! ls /usr/include/sys/capability.h
    ! ls /lib/libcap.so*
  error: 没有删除以前的libcap
- name: check capability
  script: |
    #!/bin/bash
    /sbin/getcap /usr/bin/passwd|grep cap_chown|grep cap_dac_override|grep cap_fowner|grep ep
  error: 没有设置 passwd 的 capability
```

####3.2 实验2: 调整权限


跟使用ACL的访问控制相比，capabilities有其它优势：它可以动态调整大量线程的权限，这对于遵守最小权限原则是很有必要的。当线程中某个权限不再需要时，它应当移除所有相对应的capabilities。这样，即使线程被入侵了，攻击者也得不到已经被删除的capabilities。使用以下管理操作调整权限：

1. Deleting：线程永久删除某个capability
2. Disabling：线程会暂时停用某个capability。
3. Enabling：对应Disabling，启用capability。

为了支持动态的capability分配，Linux使用一个与Set－UID相近的机制。举个例子，一个线程具有3组capability设置：允许（permitted P），可继承（inheritable I），和有效（effective E）。允许组由允许线程使用的cap组成，但其中的cap可能还没有激活。有效组由线程当前可以使用的cap组成。有效组是允许组的子集。线程可以随时更改有效组的内容只要不越过允许组的范围。可继承组是在程序运行exec()调用后计算新子线程的cap组用的。

当一个线程fork新线程的时候，子线程的cap组从父线程拷贝。当在一个线程中运行一个新程序时，它的新cap组将根据以下公式计算：

	pI_new = pI
	pP_new = fP | (fI & pI)
	pE_new = pP_new if fE = true
	pE_new = empty if fE = false

new后缀指新计算值，p前缀指线程，f前缀指文件cap。I，P，E分别指代 inheritable，permitted，effective，是一个cap位一个cap位计算的。

切换到 /home/shiyanlou/libcap-2.21/libcap 目录下，编辑 cap_proc.c文件。

```bash
$ cd /home/shiyanlou/libcap-2.21/libcap
$ sudo vi cap_proc.c
```

为了让程序操作cap变得简单，添加以下三个函数到 /home/shiyanlou/libcap-2.21/libcap/cap_proc.c 中

```
/* Disable a cap on current process */
int cap_disable(cap_value_t capflag)
{
	cap_t mycaps;
	
	mycaps = cap_get_proc();
	if (mycaps == NULL)
		return -1;
	if (cap_set_flag(mycaps, CAP_EFFECTIVE, 1, &capflag, CAP_CLEAR) != 0)
		return -1;
	if (cap_set_proc(mycaps) != 0)
		return -1;
	return 0;
}
/* Enalbe a cap on current process */
int cap_enable(cap_value_t capflag)
{
	cap_t mycaps;
	
	mycaps = cap_get_proc();
	if (mycaps == NULL)
		return -1;
	if (cap_set_flag(mycaps, CAP_EFFECTIVE, 1, &capflag, CAP_SET) != 0)
		return -1;
	if (cap_set_proc(mycaps) != 0)
		return -1;
	return 0;
}
/* Drop a cap on current process */
int cap_drop(cap_value_t capflag)
{
	cap_t mycaps;
	
	mycaps = cap_get_proc();
	if (mycaps == NULL)
		return -1;
	if (cap_set_flag(mycaps, CAP_EFFECTIVE, 1, &capflag, CAP_CLEAR) != 0)
		return -1;
	if (cap_set_flag(mycaps, CAP_PERMITTED, 1, &capflag, CAP_CLEAR) != 0)
		return -1;
	if (cap_set_proc(mycaps) != 0)
		return -1;
	return 0;
}

```
运行以下命令编译安装libcap：

	$ sudo make
	$ sudo make install	

**任务3:** 在 /home/shiyanlou/libcap-2.21/libcap 目录下新建一个 use_cap.c 文件，并分配cap_dac_read_search给它。以普通用户登录并运行程序。描述并解释你的观察。

```
#include <fcntl.h>
#include <sys/types.h>
#include <errno.h>
#include <stdlib.h>
#include <stdio.h>
#include <linux/capability.h>
#include <sys/capability.h>
int main( void )
{
	if ( open( "/etc/shadow", O_RDONLY ) < 0 )
		printf( "(a) Open failed\n" );

	if ( cap_disable( CAP_DAC_READ_SEARCH ) < 0 )
		return(-1);
	if ( open( "/etc/shadow", O_RDONLY ) < 0 )
		printf( "(b) Open failed\n" );

	if ( cap_enable( CAP_DAC_READ_SEARCH ) < 0 )
		return(-1);
	if ( open( "/etc/shadow", O_RDONLY ) < 0 )
		printf( "(c) Open failed\n" );

	if ( cap_drop( CAP_DAC_READ_SEARCH ) < 0 )
		return(-1);
	if ( open( "/etc/shadow", O_RDONLY ) < 0 )
		printf( "(d) Open failed\n" );

	if ( cap_enable( CAP_DAC_READ_SEARCH ) == 0 )
		return(-1);
	if ( open( "/etc/shadow", O_RDONLY ) < 0 )
		printf( "(e) Open failed\n" );

}
```

使用以下命令编译运行：

	$ gcc -c use_cap.c
	$ gcc -o use_cap use_cap.o -lcap

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1525415551547.png-wm)

```checker
- name：check cap_proc
  script: |
    #!/bin/bash
    grep cap_disable /home/shiyanlou/libcap-2.21/libcap/cap_proc.c
  error: 没有修改 cap_proc.c 文件
- name: check use_cap
  script: |
    #!/bin/bash
    ls /home/shiyanlou/libcap-2.21/libcap/use_cap.c
  error: 没有 use_cap.c 文件
```

当你完成以上任务时，请回答下面几个问题：

**问题1:**
当我们想动态调整基于ACL访问控制权限的数量时，应该怎么做？与capabilities比较哪种更加便捷？

ACL的使用：http://vbird.dic.ksu.edu.tw/linux_basic/0410accountmanager_3.php

**问题2:**
当程序（以普通用户运行）禁用cap A时，它遭到了缓冲区溢出攻击。攻击者成功注入恶意代码并运行。他可以使用cap A么？ 如果线程删除了cap A呢，可以使用cap A么？

**问题3:**
问题如上，改用竞态条件攻击。他可以使用cap A么？ 如果线程删除了cap A呢，可以使用cap A么？


###四、 练习

在实验楼环境安步骤进行实验，并截图

回答实验中给出的问题。

您已经完成本课程的所有实验，**干的漂亮！**

### License

本课程所涉及的实验来自[Syracuse SEED labs](http://www.cis.syr.edu/~wedu/seed/)，并在此基础上为适配[实验楼](http://www.shiyanlou.com)网站环境进行修改，修改后的实验文档仍然遵循GNU Free Documentation License。

本课程文档github链接：[https://github.com/shiyanlou/seedlab](https://github.com/shiyanlou/seedlab)

附[Syracuse SEED labs](http://www.cis.syr.edu/~wedu/seed/)版权声明：

> Copyright Statement Copyright 2006 – 2009 Wenliang Du, Syracuse University. The development of this document is funded by the National Science Foundation’s Course, Curriculum, and Laboratory Improvement (CCLI) program under Award No. 0618680 and 0231122. Permission is granted to copy, distribute and/or modify this document under the terms of the GNU Free Documentation License, Version 1.2 or any later version published by the Free Software Foundation. A copy of the license can befound at http://www.gnu.org/licenses/fdl.html.




