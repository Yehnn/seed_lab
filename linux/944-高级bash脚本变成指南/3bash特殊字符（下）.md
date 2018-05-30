---
show: step
version: 1.0
enable_checker: true
---

# bash特殊字符（下）

## 一、小括号（( )）

下面是全部特殊字符的作用的列表：

![3-1-1](https://i.imgur.com/Grtt2Yk.png)
#### 1.命令组

在括号中的命令列表，将会作为一个子 shell 来运行。

在括号中的变量，由于是在子shell中，所以对于脚本剩下的部分是不可用的。父进程，也就是脚本本身，将不能够读取在子进程中创建的变量，也就是在子shell 中创建的变量。如：
```
$ vim test20.sh
```
输入代码：
```
#!/bin/bash

a=123
( a=321; )

echo "$a" #a的值为123而不是321，因为括号将判断为局部变量
```
运行代码：
```
$ bash test20.sh
a = 123
```
在圆括号中 a 变量，更像是一个局部变量。

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/test20.sh
  error: /home/shiyanlou 目录下没有 test20.sh 文件
- name: check result
  script: |
    #!/bin/bash
    bash /home/shiyanlou/test20.sh|grep 123
  error: test20.sh 执行结果不对
```

#### 2.初始化数组

创建数组
```
$ vim test21.sh
```
输入代码：
```
#!/bin/bash

arr=(1 4 5 7 9 21)
echo ${arr[3]} # get a value of arr
```
运行代码：
```
$ bash test21.sh
7
```
```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/test21.sh
  error: /home/shiyanlou 目录下没有 test21.sh 文件
- name: check result
  script: |
    #!/bin/bash
    bash /home/shiyanlou/test21.sh|grep 7
  error: test21.sh 执行结果不对
```

## 二、大括号（{ }）

#### 1.文件名扩展

复制 t.txt 的内容到 t.back 中
```
$ vim test22.sh
```
输入代码：
```
#!/bin/bash

if [ ! -w 't.txt' ];
then
    touch t.txt
fi
echo 'test text' >> t.txt
cp t.{txt,back}
```
运行代码：
```
$ bash test22.sh
```
查看运行结果：
```
$ ls
$ cat t.txt
$ cat t.back
```
注意： 在大括号中，不允许有空白，除非这个空白被引用或转义。

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/test22.sh
  error: /home/shiyanlou 目录下没有 test22.sh 文件
- name: check result
  script: |
    #!/bin/bash
    ls|grep t.txt|grep t.back
  error: test22.sh 执行结果不对
```

#### 2.代码块

代码块，又被称为内部组，这个结构事实上创建了一个匿名函数（一个没有名字的函数）。然而，与“标准”函数不同的是，在其中声明的变量，对于脚本其他部分的代码来说还是可见的。
```
$ vim test23.sh
```
输入代码：
```
#!/bin/bash

a=123
{ a=321; }
echo "a = $a"
```
运行代码：
```
$ bash test23.sh
a = 321
```
变量 a 的值被更改了。

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/test23.sh
  error: /home/shiyanlou 目录下没有 test23.sh 文件
- name: check result
  script: |
    #!/bin/bash
    bash /home/shiyanlou/test23.sh|grep 321
  error: test23.sh 执行结果不对
```

## 三、中括号（[ ]）

#### 1.条件测试

条件测试表达式放在[ ]中。下列练习中的-lt (less than)表示小于号。
```
$ vim test24.sh
```
输入代码：
```
#!/bin/bash

a=5
if [ $a -lt 10 ]
then
    echo "a: $a"
else
    echo 'a>10'
fi
```
运行代码：
```
$ bash test24.sh
a: 5
```
双中括号（[[ ]]）也用作条件测试（判断），后面的实验会详细讲解。

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/test24.sh
  error: /home/shiyanlou 目录下没有 test24.sh 文件
- name: check result
  script: |
    #!/bin/bash
    bash /home/shiyanlou/test24.sh|grep 5
  error: test24.sh 执行结果不对
```

#### 2.数组元素

在一个array结构的上下文中，中括号用来引用数组中每个元素的编号。
```
$ vim test25.sh
```
输入代码：
```
#!/bin/bash

arr=(12 22 32)
arr[0]=10
echo ${arr[0]}
```
运行代码：
```
$ bash test25.sh
10
```
```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/test25.sh
  error: /home/shiyanlou 目录下没有 test25.sh 文件
- name: check result
  script: |
    #!/bin/bash
    bash /home/shiyanlou/test25.sh|grep 10
  error: test25.sh 执行结果不对
```

## 四、尖括号（< 和 >）

#### 重定向

test.sh > filename：重定向test.sh的输出到文件 filename 中。如果 filename 存在的话，那么将会被覆盖。

test.sh &> filename：重定向 test.sh 的 stdout（标准输出）和 stderr（标准错误）到 filename 中。

test.sh >&2：重定向 test.sh 的 stdout 到 stderr 中。

test.sh >> filename：把 test.sh 的输出追加到文件 filename 中。如果filename 不存在的话，将会被创建。

## 五、竖线（|）

#### 管道

分析前边命令的输出，并将输出作为后边命令的输入。这是一种产生命令链的好方法。
```
$ vim test26.sh
```
输入代码：
```
#!/bin/bash

tr 'a-z' 'A-Z'
exit 0
```
现在让我们输送ls -l的输出到一个脚本中：
```
$ chmod 755 test26.sh
$ ls -l | ./test26.sh
```
输出的内容均变为了大写字母。

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/test26.sh
  error: /home/shiyanlou 目录下没有 test26.sh 文件
- name: check result
  script: |
    #!/bin/bash
    grep tr /home/shiyanlou/test26.sh
  error: test26.sh 代码不对
```

## 六、破折号（-）

#### 1.选项，前缀

在所有的命令内如果想使用选项参数的话,前边都要加上“-”。
```
$ vim test27.sh
```
输入代码：
```
#!/bin/bash

a=5
b=5
if [ "$a" -eq "$b" ]
then
    echo "a is equal to b."
fi
```
运行代码：
```
$ bash test27.sh
a is equal to b.
```
```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/test27.sh
  error: /home/shiyanlou 目录下没有 test27.sh 文件
- name: check result
  script: |
    #!/bin/bash
    bash /home/shiyanlou/test27.sh|grep "is"
  error: test27.sh 运行结果不对 
```

#### 2.用于重定向stdin或stdout

下面脚本用于备份最后24小时当前目录下所有修改的文件.
```
$ vim test28.sh
```
输入代码：
```
#!/bin/bash

BACKUPFILE=backup-$(date +%m-%d-%Y)
# 在备份文件中嵌入时间.
archive=${1:-$BACKUPFILE}
#  如果在命令行中没有指定备份文件的文件名,
#  那么将默认使用"backup-MM-DD-YYYY.tar.gz".

tar cvf - `find . -mtime -1 -type f -print` > $archive.tar
gzip $archive.tar
echo "Directory $PWD backed up in archive file \"$archive.tar.gz\"."

exit 0
```
运行代码：
```
$ bash test28.sh
$ ls
```
```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/test28.sh
  error: /home/shiyanlou 目录下没有 test28.sh 文件
- name: check result
  script: |
    #!/bin/bash
    ls /home/shiyanlou/*.tar.gz
  error: test28.sh 运行结果不对 
```

## 七、波浪号（~）

#### 目录

~ 表示 home 目录。