---
show: step
version: 0.1
enable_checker: true
---

# Bash 流程控制

## 1. 实验介绍

#### 1.1 实验内容

本实验带领大家学习 shell 脚本中的流程控制，其中包括了条件判断，分支结构以及循环结构的流程控制语句。

#### 1.2 实验知识点

- 条件判断
- 分支结构
- 循环结构
- 循环控制

## 2. 条件判断

条件判断是在程序执行时，根据不同的条件，选择执行不同的程序语句。通常包括单分支结构和多分支结构。单分支结构就是只判断一种情况，多分支结构就是判断多种情况。

### 2.1 单分支

针对单分支的情况，我们可以使用两种方式进行条件判断。

#### 1. if 语句

最常见的条件判断语句就是：`if...then`，当满足条件就执行操作命令，`fi` 表示结束。

语法：

```bash
if 表达式 ;then

操作

fi
```

比如我们要实现判断从控制台输入的数字是否大于 7 ，如果大于 7 就在控制台打印出 you're right! 。

```bash
$ vim test.sh
```

```bash
#!/bin/bash

read -p "Input num :" a 

if [ $a -gt 7 ];then
   echo "you're right!"
fi
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/test.sh
  error: /home/shiyanlou 目录下没有 test.sh 文件
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4156timestamp1512714565920.png/wm)

#### 2. case 语句

通常单分支的情况都是使用 if 语句，但我们也可以使用 case 语句来判断单分支的情况。

单分支的 case 语句语法结构如下：

```bash
case 变量值 in
值)
操作
esac
```

> 判断当变量值等于某值的时候，就执行包含在结构中的语句。 用 `esac` 结束。

比如我们要实现当从控制台输入的数字等于 7 时，输出 you're right ，除了使用 if 语句还可以使用 case 语句。

```bash
$ vim testcase.sh
```

输入如下内容：

```bash
#!/bin/bash

read -p "Input num :" a 

case $a in 
7)
   echo "you're right!"
esac
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/testcase.sh
  error: /home/shiyanlou 目录下没有 testcase.sh 文件
```
运行结果如下：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/596222/1512827097446.png-wm)

### 2.2 多分支

上面的例子只是当我们输入的数字大于 7 才会输出一些内容，当我们想要再输入的数字等于 7 或者小于 7 时，也能给出一些提示信息的话，就需要对多种情况进行判断，此时我们就需要使用下面讲解的两种结构。

#### 1. elif/else

多分支的 if 语句结构如下：

```bash
if 表达式1 ;then
操作1
elif 表达式2 ;then
操作2
elif 表达式3 ;then
操作3
...
else
操作4
fi
```

> 如果表达式1的结果为真，则执行操作1，如果表达式2的结果为真，则执行操作2，依次类推，如果所有条件都不符合，则执行 `else` 后面的操作4 。同样也是以 `fi` 结束。

多分支 if 判断流程图：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4156timestamp1512754274229.png)

中间红色虚线框可以为零个或多个分支。

**实例**

我们想要实现在等于 7 和 小于 7 的时候也能打印出一些提示，就可以按照如下操作。

新建一个 test2.sh 

```bash
$ vim test2.sh
```

```bash
#!/bin/bash

read -p "Input num :" a 

if [ $a -gt 7 ];then
   echo "you're right!"
elif [ $a -eq 7 ];then
	echo "This number is equal to 7"
else
	echo "This number is less than 7"
fi
```
```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/test2.sh
  error: /home/shiyanlou 目录下没有 test2.sh 文件
```
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4156timestamp1512720453204.png/wm)

#### 2. case 

条件分支中如果需要判断的条件过多，那么使用 `if/elif` 来处理就很显得很麻烦，因此在处理选择条件较多的情况时就可以利用 `case` 来完成，它类似于编程语言中的 `switch/case` ，但是在用法上有所不同。

**语法**：

```bash
case 变量值 in
值1）
操作1
;;  
值2）
操作2
;;
...
*）
操作3
;;
esac
```

>  如果变量值等于值1，则执行操作1，如果变量值为值2，则执行操作2，依次类推，如果所有的条件都不满足，则执行 `*）` 后的操作3 。
>
>  - 每个判断条件需要使用 `;;` 双分号来结束。
>  - 值不一定是数字，也可以为字符串。
>  - 最后需要用 `esac` 来结束本次条件判断。

case 判断流程图

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4156timestamp1512753717805.png/wm)

**实例**

如下实例实现判断输入月份属于哪个季度。我们新建一个 case.sh

```bash
$ vim case.sh
```

```bash
#!/bin/bash

read -p "Input the month :" a

case $a in
1|2|3)
        echo "This is the first quarter";;
4|5|6)
        echo "This is the second quarter";;
7|8|9)
    echo "This is the third quarter";;
10|11|12)
    echo "This is the fourth quarter";;
*)
        echo "Input error";;
esac
```

运行

```bash
$ bash case.sh
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/case.sh
  error: /home/shiyanlou 目录下没有 case.sh 文件
```
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4156timestamp1512723242880.png/wm)

Bash 条件判断操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week2/4-1.mp4
@`

## 4. 循环结构

在编写脚本时会遇到要重复执行一段代码很多次或上百上千次，这种时候设定成循环语句来实现就会简单很多了。

常用的循环结构有：

+ 当型结构：**for 循环、while 循环**
+ 直到型结构：**until 循环**

> - 当型结构：先对控制条件进行判断，当条件满足时，再重复执行一些操作，直到条件不再满足。
>
> - 直到型循环：先在执行了一次循环体之后，再对控制条件进行判断，当条件不满足时执行循环体，满足时则停止。
### 4.1 for 循环

`for` 循环流程图如下：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1510035231810.png-wm)

`for` 循环有三种结构：

- 列表 for 循环
- 不带列表的 for 循环
- 类似 C 语言风格的 for 循环

>  注：列表是一种数据构成的有限序列，即按照一定的线性顺序，排列而成的数据的集合。比如 1，2，3，4，5

下面将一一介绍：

#### 1. 列表 for 循环

语法：

```bash
for 循环变量 in 列表
do
	操作
done
```

> - `do` 和 `done` 之间的命令称为循环体，执行次数和列表中常数或字符串的个数相同。
> - `for` 循环首先将 in 后列表的第一个常数或字符串赋值给循环变量，然后执行循环体，再取列表中第二个值赋值给循环变量，然后执行循环体，以此类推。
> - `bash` 支持列表 `for` 循环使用略写的计数方式，1～10 的范围用 `{1..10}` 表示（大括号不能去掉，否则会当作一个字符串处理）。 
> - `bash` 中还支持按规定的步数进行跳跃的方式实现列表 `for` 循环，例如计算1～100内所有的偶数之和。      

**实例1**：

用 for 循环的方式输出 1 到 10 ：

新建一个 `loop.sh`

```bash
$ vim loop.sh
```

```bash
#!/bin/bash

for i in {1..10}
do
echo $i
done
```
```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/loop.sh
  error: /home/shiyanlou 目录下没有 loop.sh 文件
```
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4156timestamp1512728175271.png/wm)

**实例2**：

我们使用 for 循环进行数值运算，计算出 `1~10` 的和。

```bash
$ vim sum.sh
```

```bash
#!/bin/bash

sum=0

for n in {1..10};do

  let "sum+=n" # 进行数值运算

done

echo "sum is : $sum"
  
```
```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/sum.sh
  error: /home/shiyanlou 目录下没有 sum.sh 文件
```
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4156timestamp1512728020875.png/wm)

#### 2. 不带列表的 for 循环

语法：

```bash
for 循环变量
do
操作
done
```

**实例：**

使用 for 循环打印出从控制台输入的参数。

```bash
$ vim nolist.sh
```

```bash
#!/bin/bash

for a
do
echo $a
done
```

执行：

```bash
$ bash nolist.sh 1 2 3
1
2
3
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/nolist.sh
  error: /home/shiyanlou 目录下没有 nolist.sh 文件
```
#### 3. 类似 C 语言风格的 for 循环

也被称为计数循环。

**实例：**

实现计算 1 到 10 的和。

```bash
#!/bin/bash

sum=0

for ((n=1;n<=10;n++));
do

  let "sum+=n"

done

echo "sum is : $sum"
  
```

> - `sum` 代表和，先给 `sum` 赋了一个初值 0 。
> - 在`for ` 后面的双括号中，给循环变量 `n` 赋了一个初值 1，当 n 小于或者等于 10 的时候，就把 sum 和 n 的和赋值给变量 sum ，然后 n 再加一。刚开始的时候，n=1，sum=0，这个时候 n 的值是小于 10 的，所以进入循环体，执行 sum+=n ，这个时候 sum 的值就变成了 1 ，然后执行 n++ ，这个时候 n 的值变为了 2，此时 n 的值还是小于 10 的，所以再次进入循环体，依次类推。直到 n 的值大于 10 时，就不再进入循环体。

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/sum.sh
  error: /home/shiyanlou 目录下没有 sum.sh 文件
```
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4156timestamp1512728011498.png/wm)

### 4.2 while 循环

while 循环利用一个条件来控制是否继续重复执行某个操作。为了避免死循环，必须保证循环体中包含退出循环的条件。

语法：

```bash
while 判断表达式
do
	操作
done
```

while 循环流程图如下：

在开始的时候就说明了 while 循环属于当型循环结构，即先对条件进行判断，然后再循环。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1510035239428.png-wm)

**实例：**

使用 while 循环计算出 `1~10` 的和。

```bash
$ vim sumwhile.sh
```

```bash
#!/bin/bash

sum=0
num=1
while (($num <= 10))
do
	let "sum+=num"
	let "num++"
done
echo "the sum is $sum"
```

```bash
$ bash sumwhile.sh
the sum is 55
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/sumwhile.sh
  error: /home/shiyanlou 目录下没有 sumwhile.sh 文件
```
### 4.3 until 循环

until 与 while 的不同是 until 在条件表达式不成立时，进入循环，条件成立，终止循环，与 while 刚好相反。

语法：

```bash
until 判断表达式
do
操作
done
```


until 循环流程图如下：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4156timestamp1512753735254.png/wm)

**实例：**

使用 until 循环计算出 `1~10` 的和。


```bash
$ vim sumuntil.sh
```

```bash
#!/bin/bash

sum=0
num=1
until (( $num > 10 ))
do
	let "sum+=num"
	let "num++"
done
echo "the sum is $sum"
```

运行

```bash
$ bash sumuntil.sh
the sum is 55
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/sumuntil.sh
  error: /home/shiyanlou 目录下没有 sumuntil.sh 文件
```
### 4.4 select 循环

除了前面三种比较常见的循环控制以外，还有一种叫 select 循环的控制语句。它主要是提供一种创建具有编号的菜单的方法。就像我们平时出去吃饭一样，店家制定一个菜单供我们点菜。

语法：

```bash
select 变量名 in [ 菜单取值列表 ]
do
	操作
done
```

流程图：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1510038932438.png-wm)

**实例1**

通过 `select` 简单创建一个菜单选项，实现打印出我们选择的字符串。

新建一个 select.sh 脚本文件：

```bash
$ vim select.sh
```

```bash
#!/bin/bash

select str in hello shiyanlou louplus
do
	echo $str
done
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/select.sh
  error: /home/shiyanlou 目录下没有 select.sh 文件
```
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4156timestamp1512728990386.png/wm)

> `#？` 是默认提示符
> 退出按 `ctrl + C` 

**实例2：**

打印出我们选择的在 /home/shiyanlou 下的文件名或目录名。

新建一个 select2.sh

```bash
vim select2.sh
```

```bash
#!/bin/bash

PS3="select a num:"
select dir in `ls /home/shiyanlou`
do
	echo -e "you seleted:\n  $dir"
done
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/select2.sh
  error: /home/shiyanlou 目录下没有 select2.sh 文件
```
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4156timestamp1512730540149.png/wm)

> 注
> - `\n`  是换行，echo 需要加上 `-e` 参数才可以识别这个转义符
> - `PS3` 是 `select` 循环的提示符，就是之前提到的默认提示符（`#？`），这里我们对其进行了修改

### 4.5 嵌套循环

主要是指一个循环语句中还可以包含另一个循环语句，其中，for、while、until 循环等都可以嵌套其他循环语句。

下面我们通过 while 循环里嵌套一个 for 循环来简单说明下嵌套循环的逻辑。建议先猜测运行结果，再看实际的运行结果。

新建一个脚本 loops.sh
```bash
$ vim loops.sh
```
```bash
#!/bin/bash

n=3
while [ $n -gt 0 ]
do
  echo "out loop $n"
  n=$[ $n-1 ]
  for (( i=2; i>0; i--))
  do
    echo "this is inside loop $i"
  done
done
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/loops.sh
  error: /home/shiyanlou 目录下没有 loops.sh 文件
```
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4156timestamp1512755191833.png/wm)



## 5. 循环控制

像其他编程语言一样，shell 有时也需要在没有到达循环条件结束时就强制跳出循环。常见的强制跳出循环的操作有：`break`、`continue` 等。

### 5.1 break

`break` 强制跳出所有循环，终止整个循环的执行。它可以跳出 for、while、until 等循环。

```bash
# 退出循环
break  

# 在嵌套循环中，退出第 n 层循环
break n 
```

**举例**

```bash
$ vim break.sh
```
```bash
#!/bin/sh

num=0

while [ $num -lt 10 ]
do
   echo $num
   
   if [ $num -eq 7 ]
   then
      break
   fi
   let num++
done
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/break.sh
  error: /home/shiyanlou 目录下没有 break.sh 文件
```
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4156timestamp1512749107308.png/wm)

> 可以看到 break 执行后，7 以后的数字就没有再输出了，也就是说跳出了循环。


### 5.2 continue 

`continue` 也是用于强制跳出循环，不过它主要是跳出当前循环，不会跳出所有循环，也就是后面的循环在正常情况下依旧能执行。

**举例**

```bash
$ vim continue.sh
```
```bash
#!/bin/sh

for num in {1..10}
do

   if [ $num -eq 7 ]
   then
      continue
   fi

   echo $num
done
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/continue.sh
  error: /home/shiyanlou 目录下没有 continue.sh 文件
```
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid276733labid4156timestamp1512750087342.png/wm)

> 运行到 continue 的时候就跳出了当前的循环，因此在输出时就没有输出数字 `7`。 

Bash 循环结构和循环控制操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week2/4-2.mp4
@`

## 6 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎与我们交流：

- 条件判断 if
- 分支结构 case、elif/else
- 循环结构 for、while、until、select，
- 循环控制 break、continue