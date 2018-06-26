---
show: step
version: 1.0
enable_checker: true
---
# Ansible 调试技巧

## 一、实验介绍
#### 1.1 实验介绍
通过以上两个实验大家会发现，在执行 playbook 的时候成功的执行，没有信息的反馈，失败的信息也很杂乱，`本节实验`将带领大家使用更多的是用`调试技巧`

#### 1.2 实验知识点

- register 与 debug 的使用
- plugins 的使用


#### 1.3 适合人群
本节实验适合更深入学习ansible的人群，通过返回信息，调试分析判断
## 二、实验步骤

接下来正式进入实验。

### 2.1 register 与 debug 的使用

虽说我们在使用 ansible-playbook 的时候可以通过 `-vvv` 参数来查看一些调试信息但是那样的数据都是糅杂在一起的并不人性化。

![2.1-1](https://dn-simplecloud.shiyanlou.com/1135081470735181317-wm)

有时候我们需要看到 playbook 执行之后的一些输出信息，或者就是任务中的一个 `echo` 或者是 `ll` 这样的一些信息来帮助我们了解是否达到了预期的效果，但是它的成功返回只有 change，这时候我们就可以使用 register 与 debug 模块了

```
---
- hosts: test

  tasks:
  
     - name: test test register
       shell: echo $PATH
       
...
```

![2.1-2](https://dn-simplecloud.shiyanlou.com/1135081470730770813-wm)

若是我们使用 debug 模块的效果是这样的：

```
---
- hosts: test

  tasks:
  
     - name: test test register
       shell: echo $PATH
       register: test
       
       
     - debug: msg={{test}}
...
```

![2.1-3](https://dn-simplecloud.shiyanlou.com/1135081470731061330-wm)

### 2.2 plugins 的使用

通过以前的实验，我们可以发现成功的执行时，并没有修改时返回的是 `ok: [host ip]`，若是成功的执行，并且做出了修改，返回的是 `changed: [host ip]`,而当错误发生的时候则是返回一系列的信息，糅杂在一起。

![2.2-1](https://dn-simplecloud.shiyanlou.com/1135081470729915971-wm)

最主要的便是，若是我们需要获取一些标准输出的信息，或者是 echo 一些信息来判断时，是看不见的。正如上面的例子我们想查看 `$PATH` 的值的时候，只有一个 change 的返回，虽说可以使用 register 与 debug 模块来解决，但是我们不可能注册这么多的变量来查看呀。

这时我们可以使用这样的一个 callback 插件 human，便可以帮我们将本应该在执行命令后输出的一些信息显示出来，当然我们可以自己编写一个。

- 这个插件在 github 中公开出来了，我们可以通过 git 下载下来。

```
git clone https://github.com/n0ts/ansible-human_log.git
```

- 然后放在我们配置文件的预定义好的插件默认位置 `/usr/share/ansible/plugins/callback`,默认并没有创建这个文件夹，我们可以自己创建

![2.2-2](https://dn-simplecloud.shiyanlou.com/1135081470728889705-wm)

```
sudo mkdir -p /usr/share/ansible/plugins/callback

sudo mv /home/shiyanlou/ansible-human_log/human_log.py /usr/share/ansible/plugins/callback/human_log.py
```

这样当我们再执行 playbook 时就可以看到，部分的结果在 msg 中有返回

![2.2-3](https://dn-simplecloud.shiyanlou.com/1135081470731300639-wm)

![2.2-4](https://dn-simplecloud.shiyanlou.com/1135081470731389992-wm)

在安装软件包的时候，也不再是单一的一个change了

![2.2-5](https://dn-simplecloud.shiyanlou.com/1135081470732788861-wm)

同样若是遇到错误的信息也会这样的显示，不用只是看糅杂在一起的信息了

![2.2-6](https://dn-simplecloud.shiyanlou.com/1135081470732486609-wm)

我们可以通过 human 的源码了解到这个由 python 写出来的插件会把得到的信息这样的分类

```
# Fields to reformat output for
FIELDS = ['cmd', 'command', 'start', 'end', 'delta', 'msg', 'stdout','stderr', 'results']
```

并且我们可以看到并不是所有的情况都有详细的信息输出,如当是略过的情况是便不会输出，当然也没有什么信息可以输入。

```
def runner_on_failed(self, host, res, ignore_errors=False):
    self.human_log(res)

def runner_on_ok(self, host, res):
    self.human_log(res)

def runner_on_skipped(self, host, item=None):
    pass
```

这里只是想说这个 python 的插件并不是完美的，当我们看到有我们需要的信息却没有输出的时候不需要奇怪，会 python 的也可以根据自己的需求修改这个脚本

当然Ansible 宣称可以接受任何语言的模块与插件，所以不仅仅是会 python，会C、shell 等等只要能够达到自己的需求，都可以写一个插件为自己服务。

## 三、实验总结

通过本实验我们更加深入的了解 Ansible 的用法，在调试的时候我们可以使用 register 与 debug 模块来帮助我们获取一些信息，还有就是通过使用自定义的插件来帮助我们获取有用的输出信息。

## 四、参考资料

[1] Ansible 的 Developing Plugins：<http://docs.ansible.com/ansible/developing_plugins.html>

[2] github 的开源插件：<https://github.com/n0ts/ansible-human_log/blob/master/human_log.py>