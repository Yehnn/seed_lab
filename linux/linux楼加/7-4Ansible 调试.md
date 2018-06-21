# Ansible 调试

## 1. 实验介绍

### 1.1 实验内容

在前面的实验中我们可以发现在执行 playbook 成功后并没有详细的信息反馈，而输出的失败信息也很杂乱。因此，这节实验将带着大家一起来学习如何进行调试（Debug）。

### 1.2 实验知识点

+ ansible-playbook 参数使用

+ Debug 的使用

+ Ansible 的安装

+ Developing Plugins

### 1.3 推荐阅读

## 2. 参数

对于 playbook 的调试技巧有多种，这里我们先使用参数这种比较简单的方式来协助 debug。

我们可以通过 `ansible-playbook -h` 或者 `man` 命令来查看详细的参数列表。

```bash
$ ansible-playbook -h
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516357176185.png-wm)

参数的使用格式如下：

```bash
ansible-playbook [options] playbook.yml
```

可以看到有很多的参数可以来协助我们分析，这里我们举一个比较常用的参数 `-v, --verbose`。（`-vvv` 是更详细的模式，`-vvvv` 启用连接调试）

选择 `verbose` 模式时，会打印出所有模块运行后的变量。

```bash
# -v 和 -verbose 参数的效果相同
$ ansible-playbook -v test_apache.yaml
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516357288337.png-wm)

可以看到输出了每个模块运行后的一些变量，这样就方便我们遇到实际问题时进一步查验信息。

```bash
$ ansible-playbook -vvv test_apache.yaml
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516357322411.png-wm)

这个参数输出的信息十分的详细，缺点就是不易于阅读和分析。

还有其他参数提供的模式来 debug 一些问题，这里不做进一步讲解，感兴趣的同学可以多多尝试。

## 3. Debug 模块的使用

在前面的学习中我们知道 Ansible 有众多的模块调用来实现某些功能，而对于像 playbook 这样执行模块后输出的信息往往返回的只有 change 这一类的信息。为了让我们能通过某些信息来判断是否达到预期效果时，我们就可以调用 Ansible 中的模块来实现。例如我们要讲的 debug 模块在执行过程中会打印出详细信息，同时还可以用在调试变量或表达式中。

`debug 模块`的使用相对简单，主要参数是 `msg` （用于调试输出信息）和 `var` 变量（需要调试的变量名称，即将任务执行的输出结果作为一个变量传递给 debug 模块）。

**补充：在 playbook 中，我们往往会用某个变量来储存某个命令的结果，以备日后访问的使用。在后面我们会用到 `register` 这个关键词（注册变量，Registered Variables）来决定将结果存储在哪个变量中。**

下面我们通过一个实际例子来学习下如何运用这个模块。

首先，我们重新建立一个 playbook 的文件。

```bash
$ sudo vim testdebug.yaml
```

然后，编写 playbook 的内容，这里我们先不调用模块，以便和后面对比。

```bash
---
- hosts: test

  tasks:

     - name: test register
       shell: echo $PATH  # 调用 shell 模块输出路径
...
```

接着保存退出，执行文件

```bash
$ ansible-playbook testdebug.yaml
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516357725157.png-wm)

从输出信息看到，只是返回了 `ok`、`changed` 这类的信息，具体的效果我们并不得知。

然后，这时我们再次编辑 `testdebug.yaml` 文件，添加相应的模块。

```bash
---
- hosts: test

  tasks:

     - name: test register
       shell: echo $PATH
       register: test

     - debug: msg={{test}}
...
```

保存退出，再次使用 `ansible-playbook` 执行 playbook。

```bash
$ ansible-playbook testdebug.yaml
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516357889837.png-wm)

这时，我们就可以看到详细的 debug 信息。

Ansible Debug 模块操作视频：


`@
http://labfile.oss.aliyuncs.com/courses/980/week10/3-1.mp4
@`

## 4. Plugins

通过之前的实验我们可以发现，当一个 playbook 执行成功时，并未修改会返回一个 `ok:[host ip]` ，若执行成功并进行了修改，就会返回一个 `changed:[host ip]` ，当发生错误时就会返回一些 `fail` 错误信息，但有时这些信息十分繁杂，不利于阅读分析。虽然我们上面讲了如何用 debug 模块来解决这样的麻烦，但是，在批量操作的时候我们不可能挨个去注册这么多变量和模块来查看结果。这时，我们就可以使用 Ansible 中另一个强大的功能——`Plugins`（插件）来帮助我们在执行命令后输出一些信息，其中，在 GitHub 上公开了很多插件的代码，不过相信通过学习你也可以自己编写出一个插件哟！

Plugins 是一种增强 Ansible 核心功能的一段代码，Ansible 提供了一些插件，如下所示：

+ `Action plugins`：是模块的前端，在调用模块之前在控制器上执行动作

+ `Cache plugins`：用于保存缓存

+ `Callback plugins`：能够 hook 到 ansible 的事件，然后用于显示或记录

+ `Connection plugins`：定义了如何与 inventory 的主机进行通信

+ `Filters plugins`：允许操作 ansible 中的数据，这是一个 jinja2 的功能

+ `Lookup plugins`：用于从外部来源获取数据

+ `Strategy plugins`：用于控制和执行逻辑的流程

+ `Shell plugins`：用于处理 ansible 在远程主机上遇到的不同 shell 的命令和格式

+ `Test plugins`：用于验证 ansible 中的数据

+ `Vars plugins`：将额外的变量数据用到运行的 ansible 中

这些插件我们还可以在配置文件 `/etc/ansible/ansible.cfg` 中查看到：

```bash
$ sudo vim /etc/ansible/ansible.cfg
```

![实验楼](https://dn-simplecloud.shiyanlou.com/2767331516353867691-wm)

Ansible 插件介绍视频：


`@
http://labfile.oss.aliyuncs.com/courses/980/week10/3-2.mp4
@`

这里我们举两个插件的用法为例，其他的插件大家可以下来研究。

**Callback plugins**

前面我们通过模块就可以解决详尽显示出执行后的信息的问题，同样我们也可以通过这个 `Callback` 插件来帮助我们显示信息。

这个插件在 GitHub 中是公开的，我们可以直接 git 下载下来。

```bash
$ git clone https://github.com/n0ts/ansible-human_log.git
```

然后将这个文件放置在预定义的插件位置（`/usr/share/ansible/plugins/callback`），这里默认没有创建这个文件夹，需要我们自行创建。

```bash
$ sudo mkdir -p /usr/share/ansible/plugins/callback

$ sudo mv /home/shiyanlou/ansible-human_log/human_log.py /usr/share/ansible/plugins/callback/human_log.py
```

这里我们看到会涉及到了 `.py` 文件，这是因为在 Ansible 的使用中是通过 `Python API` 来管理节点，再通过扩展 Ansible 来响应 python 事件，同时也可以通过相应插件（plugins）来调取数据源。

这里我们 git 下来的 `human_log.py` 文件正是插件使用的一个 python API。可以用 `cat` 命令来查看下里面的内容。

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516358253547.png-wm)

内容较多，大家可以自行阅读。

然后，我们就可以再次执行之前的 playbook 即可。

```bash
$ ansible-playbook testdebug.yaml
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516358457763.png-wm)

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516358463742.png-wm)

从输出的结果可以看到更多的信息在结果中的返回，从 `human` 的源代码中还可以了解到插件会把信息进行如下的分类：

```bash
# Fields to reformat output for
FIELDS = ['cmd', 'command', 'start', 'end', 'delta', 'msg', 'stdout','stderr', 'results']
```

但是也可以看出并不是所有情况都会有详细的信息输出，也就是说明 python 的插件也不是最完美的，不过 Ansile 宣称能够接受任何语言的模块或插件，所以像 C、shell 等只要能够达到需求的语言都可以用来编写一个插件。

**Lookup plugins**

Lookup plugins 可以用于从外部数据存储中获取数据。

下面用一个简单的例子来是实现查找插件的功能，需求是查找并返回一个文本文件的内容来作为变量。

首先，创建一个文本文件

```bash
$ sudo vim lookup.txt
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516358618589.png-wm)

然后，创建查找的插件 py 文件。

```bash
$ sudo vim lookup.py
```

```bash
from ansible.errors import AnsibleError, AnsibleParserError
from ansible.plugins.lookup import LookupBase

try:
    from __main__ import display
except ImportError:
    from ansible.utils.display import Display
    display = Display()


class LookupModule(LookupBase):

    def run(self, terms, variables=None, **kwargs):

        ret = []

        for term in terms:
            display.debug("File lookup term: %s" % term)

            # Find the file in the expected search path
            lookupfile = self.find_file_in_search_path(variables, 'files', term)
            display.vvvv(u"File lookup using %s as file" % lookupfile)
            try:
                if lookupfile:
                    contents, show_data = self._loader._get_file_contents(lookupfile)
                    ret.append(contents.rstrip())
                else:
                    raise AnsibleParserError()
            except AnsibleParserError:
                raise AnsibleError("could not locate file in lookup: %s" % term)

        return ret
```

接着就是创建一个 playbook 文件。

```bash
$ sudo vim lookup.yaml
```

```bash
---
- hosts: all
  vars:
     contents: "{{ lookup('file', '/home/shiyanlou/lookup.txt') }}"

  tasks:

     - debug: msg="the value of lookup.txt is {{ contents }} as seen today {{ lookup('pipe', 'date +"%Y-%m-%d"') }}"
```

最后执行命令即可看到返回信息。

```bash
$ ansible-playbook lookup.yaml
```

![图片描述](https://dn-simplecloud.shiyanlou.com/uid/276733/1516358758055.png-wm)


## 5.总结

本节实验主要讲解了 Ansible 中对于 playbook 的几种调试技巧，包括了参数的使用、debug 模块和插件（plugins）。