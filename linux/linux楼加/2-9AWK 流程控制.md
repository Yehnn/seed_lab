---
show: step
version: 1.0
enable_checker: true
---

# AWK 流程控制

## 1. 实验介绍

#### 1.1 实验内容

本节实验主要介绍 awk 中的流程控制，包括条件操作符，条件语句，循环语句等，以及数组的应用。

#### 1.2 实验知识点

- 条件语句
- 循环
- 数组

#### 1.3 推荐阅读

- http://bbs.chinaunix.net/forum.php?mod=viewthread&tid=1790335

## 2. 条件语句

#### 2.1 语法

awk 中的流程控制语句与其他高级编程语言（C、Java 等）相差不大，其格式为：

```c
if(expression)
  action1
[else action2]
```

如果条件表达式 expression 的值为真，就执行 action1 。当存在 else 语句时，如果条件表达式为假，就执行 action2 。

如果操作是多个语句组成，就要用大括号括起来：

```c
if(expression){
  statement1
  statement2
    ...
}
else if(expression){
  statement1
  statement2
    ...
}
else{
  statement1
  statement2
    ...
}
```

#### 2.2 操作符

#### 1. 关系操作符

| 操作符 | 描述       |
| ------ | ---------- |
| <      | 小于       |
| >      | 大于       |
| <=     | 小于或等于 |
| >=     | 大于或等于 |
| ==     | 相等的     |
| !=     | 不等的     |
| ~      | 匹配       |
| !~     | 不匹配     |

#### 2. 逻辑操作符

| 操作符   | 描述                         |
| -------- | ---------------------------- |
| 两个竖线 | 逻辑或，左右有一个为真就为真 |
| &&       | 逻辑与，左右都为真才为真     |
| !        | 逻辑非，表达式为假时才为真   |

#### 2.3 应用实例

新建一个测试文本 testavg，输入如下内容：

```
john 85 92 78 94 88
andrea 89 90 75 90 86
jasper 84 88 80 92 84
tom 60 55 70 65 60
bob 99 90 87 93 96
jim 76 75 83 65 66
```

我们将通过脚本来计算学生平均成绩：

新建一个脚本文件 avg ，输入如下内容：

```c
{
	total=$2+$3+$4+$5+$6
	avg=total/5
	print $1,avg
}
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/avg
  error: /home/shiyanluo 目录下没有 avg 文件
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/testavg
  error: /home/shiyanluo 目录下没有 testavg 文件
```

运行

```bash
$ awk -f avg testavg
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1510731337654.png-wm)

如果平均分为大于等于 65 就为及格的话，显示及格不及格情况。我们来修改 avg 如下

```c
{
	total=$2+$3+$4+$5+$6
	avg=total/5
	if(avg >= 65)
		grade="Pass"
	else
		grade="Fail"
	print $1,avg,grade
}
```
![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1510731513399.png-wm)

如果对成绩划分等级呢，大于等于 90 的为 A ，大于等于 80 的为 B ，大于等于 70 的为 C ，大于等于 60 的为 D ，其他的就是不及格 F 。

修改 avg 为如下：

```c
{
	total=$2+$3+$4+$5+$6
	avg=total/5
	if(avg >= 90) grade="A"
	else if (avg >= 80) grade="B"
	else if (avg >= 70) grade="C"
	else if (avg >= 60) grade="D"
	else grade="F"
	print $1,avg,grade
}
```

执行输出如下：

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1510732003029.png-wm)

你还可以自己试试利用前面 awk 入门讲到的格式化，对这个输出排版，让它更整齐。

awk 条件语句操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week2/9-1.mp4
@`

## 3. 循环

接下来将要学习有关 bash 中的循环。

### 3.1 while 循环

#### 1. 语法

```c
while(表达式){
  操作
}
```

如果括号里的表达式为真，就执行花括号里面的操作，一直循环，直到表达式为假。

#### 2. 应用实例

如下计算 1 到 100 的和：

```bash
$ awk '                                                            
> BEGIN{
> i=1;
> test=100;
> total=0;
> while(i<=test){
> total+=i;
> i++;
> }print total;}'
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1510734196827.png-wm)

### 3.2 do-while 循环

#### 1. 语法

```c
do{
  操作
}
while(表达式)
```

与 while 循环的不同之处：循环体至少执行一次无论表达式是否成立。

#### 2. 应用实例

新建一个脚本文件 dowhile ，输入如下内容

```c
BEGIN{
	do{
		print x=x+1

	}while(x<0)
}
```

执行之后我们会发现结果为 1，这就是因为：

> awk 中的变量被初始化为 0
>
> 在判断表达式之前会执行其中的语句块 `print x=x+1`
>
> 此时打印出 x 的值之后再做判断，发现不成立，所以不再执行

这就是 do/while 循环与 while 的区别。                                            

### 3.3 for 循环

#### 1. 语法

```c
for(set_counter;test_counter;increment_counter){
  操作
}
```

- set_counter：设置计数器变量的初值
- test_counter：描述在循环开始时要测试的条件
- increment_counter：每次在循环的数值变化器，且在重新测试测试条件之前

#### 2. 应用实例

还记得我们上面那个学生成绩的文本 testavg 吗。

我们用 for 循环打印输入行的每一个字段。新建一个 for.awk 文本，输入以下内容：

```c
{
	for (i=1;i<=NF;i++)
		print $i
}
```

>注意：for 与 if 一样当操作的语句只有一句的时候可以不使用大括号 `{}`，若是多操作还是需要。

![实验楼](https://dn-simplecloud.shiyanlou.com/87971510800986529-wm)

要反序打印怎么办？我们来修改 for.awk 文件为如下：

```c
{
	for(i=NF;i>=1;i--)
		print $i
}
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971510818669219-wm)

还记得上面我们上面求平均成绩吗，下面我们来用 for 循环的方式来求平均值，修改 avg 为如下内容：

```c
{
	total=0
	for(i=2;i<=NF;i++)
		total+=$i
	avg=total/(NF-1)
	print $1,avg
}
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971510820723936-wm)


awk 循环语句操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week2/9-2.mp4
@`

### 3.4 综合实例：求阶乘

一个数的阶乘是由该数累乘小于这个数的数得到。

我们下面来编写一个 awk 的程序来从控制台读取用户输入的数，输出它的阶乘。

新建一个 jiec ，输入如下内容

```c
awk '
BEGIN{
	printf("Enter number:")
}

$1 ~ /^[0-9]+$/{     //获取输入，匹配是不是为数字
	num=$1         //num是我们要计算阶乘的数
	if(num==0)
		fact=1    
	else
		fact=num
	for(x=num-1;x>1;x--)
		fact*=x     //计算阶乘
	printf("The factorial of %d is %g\n",num,fact)
	exit
}
{printf("\nInvalid entry.Enter a number:")}'
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/jiec
  error: /home/shiyanlou 目录下没有 jiec 文件
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971510822065931-wm)

> num 是我们要计算阶乘的那个数。fact 是得出的阶乘。


### 3.5 综合实例：统计特定文件中的词频

新建一个 words.txt 文本，输入如下内容：

```
hello world
hello shiyanlou
hello louplus
```

要统计 words.txt 文件中的词频。想一想我们应该怎么编写这个 bash 脚本。下面我们来新建一个 freq.sh 脚本文件：

```bash
#!/bin/bash

if [ $# -ne 1 ];     #如果输入的参数不为1个
then
	echo "Usage:$0 filename";
	exit -1
fi
filename=$1

egrep -o "\b[[:alpha:]]+\b" $filename | \   #匹配单词
awk '

	{count[$0]++}       

	END{
		printf("%-14s%s\n","word","count");     #表头

		for(ind in count)
		{
			printf("%-14s%d\n",ind,count[ind]);
		}     #格式化打印单词以及对应的出现次数
	}       
'
```

执行

```bash
$ bash freq.sh words.txt
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/words.txt
  error: /home/shiyanlou 目录下没有 words.txt 文件
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/freq.sh
  error: /home/shiyanlou 目录下没有 freq.sh 文件
```

输出如下图

![实验楼](https://dn-simplecloud.shiyanlou.com/87971510822853344-wm)

## 4. 数组

数组可以用来存储一组数据的变量。数组中的每一个元素通过它们在数组中的下标访问。每个下标都用方括号括起来。

### 4.1 基本语法

#### 1. 建立数组

语法：

```c
array[index]=value
```

> 数组名 array，下标 index 以及相应的值 value

#### 2. 读取数组值

语法：

```c
{for(item in array) print array[item]}   #输出的顺序是随机的

{for(i=1;i<=len;i++) print array[i]}   #Len是数组的长度
```

#### 3. 多维数组

awk 支持线性数组，在这种数组中的每个元素的下标是单个下标。如果将线性数组看成是一行数组，那么两位数组将表示数据的行和列，这样的数组就是多维数组中的两维数组，依此类推。

例如二维数组语法：

```c
file_array[NR,i]=$i
```

>  file_array 就是数组名。
>
>  NR 是记录数，也就是行， i 是字段数，也就是列。下标是两个。
>
>  下标分隔符，默认是 `\034` ，可以用 `SUBSEP` 设置分隔符。

我们来看看下面

```bash
$ awk 'BEGIN{array["a","b"]=1;for(i in array) print i}'           
$ awk 'BEGIN{SUBSEP=":";array["a","b"]=1;for(i in array) print i}' 
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1510826667948.png-wm)

怎么向多维数组写入或读出元素?我们通过这样一个例子来了解：

我们新建一个宽高都为 6 的二维数组，每个位置上先用 O 填充，遍历 testdw 中的坐标，把相应坐标上的元素变为 X。

新建一个 testdw 文件，输入如下内容：

```
1,1
2,2
3,3
4,4
5,5
6,6
1,6
2,5
3,4
4,3
5,2
6,1
```

新建一个脚本文件 dw.awk ，输入如下内容：

```c
BEGIN{
FS=","
WIDTH=6
HEIGHT=6
for(i=1;i<=WIDTH;++i){
	for(j=1;j<=HEIGHT;++j){
		dw[i,j]="O"
	}
}
}
{
	dw[$1,$2]="X"
}
END{
for(i=1;i<=WIDTH;++i){
	for(j=1;j<=HEIGHT;++j)
		printf("%s",dw[i,j])
	printf("\n")
}
}
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/testdw
  error: /home/shiyanlou 目录下没有 testdw 文件
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/dw.awk
  error: /home/shiyanlou 目录下没有 dw.awk 文件
```
![实验楼](https://dn-simplecloud.shiyanlou.com/87971511345139450-wm)

#### 4. 删除数组

语法：

```bash
delete array  #删除整个数组
delete array[item]  #删除某个数组元素（item）
```

> array 为数组名

awk 数组操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week2/9-3.mp4
@`

### 4.2 应用实例

#### 1. 遍历

（1）for循环

新建一个 youxu1，输入如下内容：

```c
#!/bin/awk -f

BEGIN{
	a[1]="a"
	a[2]="b"
	a[3]="c"
	a[4]="d"
	a[5]="e"

	for(i=1;i<=length(a);i++){
		print i,a[i]
	}
}
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/youxu1
  error: /home/shiyanlou 目录下没有 youxu1 文件
```

我们执行的时候可能会在 length(a) 那行报一个 `illegal reference array a` 的错误，这是因为我们 awk 版本的问题，可以用 `man awk` 看到实验楼是 `mawk` 。我们安装 `gawk` 就行了。

```bash
$ sudo apt-get install gawk  #安装gawk
$ gawk -f youxu1  #执行
```

> 用 `length(a)` 函数得到数组 a 的长度，有关函数我们后面会有详细介绍
>
> 数组的下标是从1开始计算。注意和其他语言区分。
>
> 如果是 mawk 下，就把 `i<length(a)` 改成 `i in a` ，执行命令也可以是 `mawk -f youxu1`

（2）使用 in

新建一个 youxu2，输入如下内容：

```c
#!/bin/awk -f

BEGIN{
	a[1]="a"
	a[2]="b"
	a[3]="c"
	a[4]="d"
	a[5]="e"

	for(i in a){
		print i,a[i]
	}
}
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/youxu2
  error: /home/shiyanlou 目录下没有 youxu2 文件
```
执行

```bash
$ gawk -f youxu2
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971511257288308-wm)

> - 可以看到第一种方式是按原来的顺序打印出，而第二种方式是随机打印出。
> - 如果下标不是统一为数字，只能用 in 遍历。

#### 2. 顺序排列

我们先来新建一个测试文本 testbasic:

测试文本的每行都由一个行号开始，行号不是按照顺序排列的。

```bash
$ vim testbasic
```

```
2 hello world
1 hello shiyanlou
3 hello louplus
```

下面的程序实现把测试文件内容按行号顺序打印出来。此程序通过使用行号作为下标创建数组来排序行，然后打印按数字顺序排序的行。

新建一个脚本文件 basic：

```bash
$ vim basic
```

```c
{
    //用行号作为数组arr的下标，每行的内容作为对应的值
    if ($1 > max) 
        max = $1
    arr[$1] = $0
}

END {
  	//用for 循环遍历，打印出数组元素
    for (x = 1; x <= max; x++)
        print arr[x]
}
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/testbasic
  error: /home/shiyanlou 目录下没有 testbasic 文件
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/basic
  error: /home/shiyanlou 目录下没有 basic 文件
```
执行：

```bash
$ awk -f basic testbasic
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/596222/1512992451331.png-wm)

#### 3. 成绩例子

记得上面的成绩例子吗，我们按照平均成绩分了等级。现在我们要知道获得 A 的学生是多少，获得 B 的学生是多少怎么做。

我们可以为每个字母等级设置不同的变量并判断哪个计数器可以递增。我们可以定义一个数组 class_grade，用字母等级作为数组的下标。如果遇到该等级，就给对应的值加 1 。在 END 中用 for 循环遍历 letter_grade 制定数组 class_grade 的一个下标。输出被传送 sort 中，用于按正确的顺序输出等级。

我们修改 avg 内容为如下：

```c
BEGIN{OFS="\t"}
{
total=0
for(i=2;i<=NF;++i)
	total+=$i
avg=total/(NF-1)
student_avg[NR]=avg
if(avg >= 90) grade="A"
else if (avg >= 80) grade="B"
else if (avg >= 70) grade="C"
else if (avg >= 60) grade="D"
else grade="F"
++class_grade[grade]
print $1,avg,grade
}
END{
for(x=1;x<=NR;x++)
	class_avg_total+=student_avg[x]
class_average=class_avg_total/NR
for(x=1;x<=NR;x++)
	if(student_avg[x]>=class_average)
		++above_average
	else
		++below_average
print ""
print "class average: ",class_average
print "at or above average: ",above_average
print "below average: ",below_average
for(letter_grade in class_grade)
	print letter_grade ":",class_grade[letter_grade]|"sort"
}
```

执行

```bash
$ awk -f avg testavg
```

![实验楼](https://dn-simplecloud.shiyanlou.com/87971511342076034-wm)

## 5. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎与我们交流：

- 条件操作符有哪些
- do ,while,for 循环的应用
- 怎么统计词频
- 怎么建立，读取，删除数组
- 怎么遍历数组