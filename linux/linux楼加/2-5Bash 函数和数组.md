---
show: step
version: 0.1
enable_checker: true
---

# Bash 函数和数组

## 1. 实验介绍

#### 1.1 实验内容

本节实验带领大家学习 bash 函数和数组。

#### 1.2 实验知识点

- 数组定义
- 数组赋值
- 数组删除
- 数组元素替换
- 函数定义
- 函数调用

## 2. 数组

在前面的内容中，我们有提到变量为单个元素，对应的有代表多个元素的数组。

### 2.1 数组的定义和赋值

`bash` 提供对于一维数组的支持，需要注意的是，它并不支持多维数组。通常情况下，数组的索引为一个整数，从 `0` 开始计算。但是我们也可以使用字符串作为数组的索引，这样的数组被称为 **关联数组**。

在 `bash` 中，变量其实可以理解为只有一个元素的索引数组。如下示例：

```bash
# 定义一个变量 var1，此时 var1 为只有一个元素的数组
var1="shiyanlou001"

# 查看 var1 的值
echo $var1

# 查看 var1 数组中索引为 0 的值，即第一个元素，关于引用数组中元素的内容我们在下面的内容将会学习到
echo ${var1[0]}

# 查看 var1 数组中索引为 1 的值，结果为空
echo ${var1[1]}
```

上述命令的运行截图如下：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1512809460913.png/wm)

即上述定义的变量 `var1` 可以理解为一个只有一个元素的数组。

对于索引数组的定义，我们只需给对应的索引分配一个值，即会自动创建一个索引数组，如下所示：

```bash
# 给索引 0 分配一个值，即会自动创建索引数组 var2
var2[0]=shiyanlou001

var2[1]=shiyanlou002

# 查看数组 var2 索引 `0` 对应的值，使用下述两种方式
echo $var2

echo ${var2[0]}

# 查看索引为 `1` 的值
echo ${var2[1]}
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1512810635379.png/wm)

除了上述介绍的定义索引数组的方式，我们还可以使用 `var=(value1 value2 value3)` 的方式进行赋值，并且可以再赋值的时候指定索引。

如下示例：

```bash
# 定义数组 var3
var3=(a b c)

# 查看数组中所有的元素
echo ${var3[*]}

# 查看数组中指定索引的值
echo ${var3[0]}
echo ${var3[1]}

# 定义数组 var4，使用指定索引定义
var4=([0]=a b [1]=c [9]=d)

# 查看相应的值
echo ${var4[*]}
echo ${var4[1]}
echo ${var4[9]}
```

运行结果如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1512811395907.png/wm)

在给索引数组分配值时，如果指定了索引，则将该索引分配给指定的值。

#### 使用 read 定义

使用 `read` 命令接受输入，并保存为一个数组，需要使用到 `-a` 参数，即`array`。

如下示例：

```
$ read -a var5

# 查看数组 var5 的所有值
echo ${var5[*]}
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1512898715387.png/wm)

#### 使用 declare 定义

`declare` 命令除了可以定义索引数组之外（使用 `-a` 参数），还可以定义关联数组（使用 `-A` 参数）。

如下示例，我们使用 `declare` 声明一个索引数组：

```bash
$ declare -a var6=(a b c d)

# 查看 var6 的全部元素
$ echo ${var6[*]}
```
![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1512899028387.png/wm)

声明关联数组需要使用 `-A` 参数，如下示例：

```bash
# 分别使用 a b c 字母作为索引
$ declare -A var7=([a]=1 [b]=2 [c]=3)

# 分别查看索引为 a，b 的值
$ echo ${var7[a]} ${var7[b]}
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1512899294254.png/wm)

### 2.2 引用

对于数组的引用来说，我们有以下几种方式：

| 方式                   | 含义                                 |
| ---------------------- | ------------------------------------ |
| `${array_name[n]}`     | 查看指定索引的元素，`n` 为数组的索引 |
| `${array_name[*]}`     | 所有的数组元素的值                   |
| `${array_name[@]}`     | 所有的数组元素的值                   |
| `${!array_name[@]}`    | 所有的索引                           |
| `${!array_name[*]}`    | 所有的索引                           |
| `${array_name[*]:m}`   | 从索引`m` 开始，后面的所有元素       |
| `${array_name[*]:m:n}` | 从索引 `m` 开始，后面的 `n` 个元素   |
| `${#array_name[*]}`    | 显示数组元素个数                     |

如下示例，我们定义一个数组，分别使用下述方式：

```bash
$ var8=(a b c d e f)

# 查看指定索引的元素
$ echo ${var8[1]}  ${var8[4]}

# 查看所有的元素
$ echo ${var8[*]}

# 查看所有的索引
$ echo ${!var8[*]}

# 查看从下标 2 开始，后面的所有元素
$ echo ${var8[*]:2}

# 显示数组元素的个数
$ echo ${#var8[*]}
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1512899729827.png/wm)

### 2.3 数组删除

删除数组也是使用 `unset` 命令，但是我们可以仅删除数组中的某个元素，也可以删除整个数组。

如下示例

```bash
# 定义数组
$ array=(1 2 3)

# 打印数组中的所有元素
$ echo ${array[*]}
1 2 3
#
$ unset array[1]    # 删除索引为1的元素，注意是从0开始
$ echo $array[*]
1 3
$ unset array    # 删除整个数组
$ echo ${array[*]}
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1512900571147.png/wm)

### 2.4 替换

对于数组中每个元素，我们都可以对其进行替换操作。在进行替换时，可以指定仅操作数组中的某个元素，也可以针对整个数组。

如下示例:

```bash
# 定义数组
$ array=(a1 a2 b3 c4 d5)
$ echo ${array[*]}

# 将索引为 3 的元素中的字母 c 替换成字母 h
$ echo ${array[3]/c/h}

# 针对整个数组，进行操作，将字母 a 替换成 m
$ echo ${array[*]/a/m}

# 上述操作仅仅对输出进行替换，并未对数组中的元素进行重新赋值，我们查看数组中元素，还是与原来的内容一致
$ echo ${array[*]}
```

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1512901018242.png/wm)

Bash 数组的定义及操作视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week2/5-1.mp4
@`

## 3. 函数

在 bash 中也有函数的概念，我们接下来就来学习有关 bash 函数的内容。

### 3.1 定义

`bash` 也有自定义函数的功能，当脚本变得很大时，可将脚本文件中常用的功能写成函数，这样可以提高程序的复用性，更易维护。定义函数语法如下：

```bash
函数名(){
  函数体
}

或者

function 函数名(){
  函数体
}

或者

function 函数名 {
  函数体
}
```

对于函数定义有一些注意项，简单列举如下

+ 函数定义要在执行之前
+ bash 执行顺序是：系统别名-》函数-》系统命令-》可执行文件
+ 如果将函数放在独立的文件中，脚本使用的时候，需要使用 `source` 或 `.` 来加载

### 3.2 位置参数

函数的调用方式为 : `函数名 参数列表`

对于函数的参数与参数之间，函数名和参数之间都使用空格进行分隔。因此，在函数之中，我们引用函数的参数也是使用位置参数的方式进行使用。在函数中使用位置参数指代的是函数的参数，而不是脚本执行时使用的位置参数。

如下示例， `test3.sh` 脚本文件中的内容：

```bash
#!/bin/bash

# 定义函数
func1(){
	# 打印函数的参数
	echo $*
}

# 调用函数，参数为 a b c
func1 a b c

# 打印脚本的参数
echo $*
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/test3.sh
  error: /home/shiyanlou 目录下没有 test3.sh
```

调用脚本的执行方式为 `bash test3.sh 1 2 3`，执行结果如下图：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1512901414310.png/wm)

### 3.3 作用域

对于 `bash` 中函数中变量的作用域与其它高级编程语言不同。

如果我们在函数中有一行定义变量的语句，在我们调用函数后，该定义被执行，这时定义的变量并不属于函数内部的局部变量，在函数外部也可以使用。而定义属于函数内部的变量我们需要使用 `local` 命令。

如下示例 `test4.sh` 文件中的内容：

```bash
#!/bin/bash

func(){
	# 此时定义一个变量
	var1=shiyanlou001

	# 使用 local 定义一个局部变量
	local var2=shiyanlou002
}

# 执行函数
func

# 执行后，我们可以使用 var1 的值，但是不能使用 var2 的值
echo $var1 $var2
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/test4.sh
  error: /home/shiyanlou 目录下没有 test4.sh
```
执行脚本后，我们并不能打印出 `var2` 的值，执行结果如下图所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1512902271760.png/wm)

### 3.4 返回值

每一个脚本或者命令执行完成之后都会有一个退出的状态，系统中一般只会记录上一个指令执行之后的退出状态，而同执行脚本一样，函数执行完成后也会有一个返回的状态值，默认为函数体中最后一行命令执行的状态。

除此之外，也可以显示的使用 `return` 语句给函数返回一个状态值。`return` 语句的使用方式如下：

```bash
return [n]
```

该语句会导致函数停止执行，并且将 `n` 值返回给调用者，如果未提供 `n`，则返回值是在该函数中执行的最后一个命令的退出状态。

如下示例 `test5.sh` 文件中的内容：

```bash
#!/bin/bash

func(){
	echo "start function"
	return 123
	echo "function end"
}

func

# 查看函数的返回值，使用 $?
echo "return: $?"
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/test5.sh
  error: /home/shiyanlou 目录下没有 test5.sh
```
在执行 `return` 语句后，函数就停止执行了，所以 `echo "function end"` 并不会被执行，脚本 `test5.sh` 运行结果如下所示：

![此处输入图片的描述](https://doc.shiyanlou.com/document-uid377240labid4104timestamp1512903039686.png/wm)


Bash 函数实现及使用视频：

`@
http://labfile.oss.aliyuncs.com/courses/980/week2/5-2.mp4
@`


### 3.5 其它示例

#### **1.求输入的数值中最大的数**

新建 `maxvalue.sh`

```bash
$ vim maxvalue.sh
```

```bash
#!/bin/bash

max(){
	while test $1 #这里$1代表传进函数的第一个参数
	do
		if test $maxvalue;then
			if test $1 -gt $maxvalue;then
				maxvalue=$1
			fi
		else
			maxvalue=$1
		fi
		shift     # 函数参数左移一位的意思
	done
	return $maxvalue  #返回值
}

max $*    #调用函数
echo "max value is $maxvalue"
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/maxvalue.sh
  error: /home/shiyanlou 目录下没有 maxvalue.sh
```
执行

```bash
$ bash maxvalue.sh 1 2 3 8 4
max value is 8
```
`shift [n]`表示把第 `n+1`个参数移到第1个参数, 即命令结束后 `$1` 的值等于 `$n+1` 的值,若 n 没有给定则默认为 `1`. 当 n 小于 0 或者大于参数个数 $# 时 `shift` 命令的返回值大于 `0`, 否则返回 `0`.

#### **2. 调用其它脚本**

新建一个 `beidy.sh`，被调用的脚本

```bash
$ vim beidy.sh
```

```bash
#!/bin/bash

echo "who you are: $USER"
```

新建一个 `dy.sh`。调用的脚本

```bash
$ vim dy.sh
```

```bash
#!/bin/bash

echo "you locate $0"

./beidy.sh
echo "first arg is $1"
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/dy.sh
  error: /home/shiyanlou 目录下没有 dy.sh
```
执行

```bash
$ bash dy.sh hello
you locate dy.sh
who you are: shiyanlou
first arg is hello
```

#### 3. 测试uRL

对一组 url 测试能否访问成功

```bash
#!/bin/bash


url_list=(  #定义一个包含三个网址的数组 url_list
http://www.baidu.com
http://www.shiyanlou.com
http://www.google.com
)

wait(){   #定义倒计时函数 wait
	echo -n 'wait 3 second...'
	for ((i=0;i<3;i++))
	do
		echo -n ".";sleep 1  #每隔1秒打印一个点
	done
	echo
}

check_url(){
	wait   # 调用已经定义的 wait 函数
	
	#循环遍历 url_list 中的地址
	for ((i=0;i<`echo ${#url_list[*]}`;i++))
	do
	    
	    #检测是否可以访问数组元素中的地址
		wget -o /dev/null -T 3 --tries=1 --spider ${url_list[$i]} >/dev/null 2>&1  #--tries是设置尝试次数，--spider检查网址，后面的>/dev/null 2>&1是不保留任何输出

		if [ $? -eq 0 ];then   #如果返回值为0则表示访问成功
			echo "${url_list[$i]} success"
		else
			echo "${url_list[$i]} false"
		fi
	done
}

main(){  #定义主函数，即入口函数，应用程序运行时首先执行的代码
		check_url   #调用定义的 check_url 函数
}


main	#调用主函数	
```

```checker
- name: check file
  script: |
    #!/bin/bash
    ls /home/shiyanlou/array.sh
  error: /home/shiyanlou 目录下没有 array.sh
```
执行

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/8797/1510215598431.png-wm)

## 4. 总结

本节实验中我们学习了以下内容，任何不清楚的地方欢迎与我们交流：

- 怎么用函数对代码进行优化
- 数组的定义，个数，删除，替换怎么操作