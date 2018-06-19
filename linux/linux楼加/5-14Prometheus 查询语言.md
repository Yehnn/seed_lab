---
show: step
version: 1.0
enable_checker: true
---
# Prometheus 查询语言

## 1. 实验介绍

PromQL（Prometheus Query Language）是 Prometheus 自己开发的表达式语言，语言表现力很丰富，内置函数也很多。使用它可以对时序数据进行筛选和聚合。本次实验我们将来学习它。

## 2. 实验知识点

- PromQL 语法
- PromQL 操作符
- PromQL 函数

## 3. PromQL 语法

下面我们将会学习PromQL 语法。

### 3.1. 数据类型

PromQL 表达式计算出来的值有以下几种类型：

- 瞬时向量 (Instant vector): 一组时序，每个时序只有一个采样值
- 区间向量 (Range vector): 一组时序，每个时序包含一段时间内的多个采样值
- 标量数据 (Scalar): 一个浮点数
- 字符串 (String): 一个字符串，暂时未用

### 3.2. 时序选择器

#### 3.2.1. 瞬时向量选择器

瞬时向量选择器用来选择一组时序在某个采样点的采样值。

最简单的情况就是指定一个度量指标，选择出所有属于该度量指标的时序的当前采样值。比如下面的表达式：

```
http_requests_total
```

可以通过在后面添加用大括号包围起来的一组标签键值对来对时序进行过滤。比如下面的表达式筛选出了 job 为 prometheus，并且 group 为 canary 的时序：

```
http_requests_total{job="prometheus", group="canary"}
```

匹配标签值时可以是等于，也可以使用正则表达式。总共有下面几种匹配操作符：

- =：完全相等
- !=: 不相等
- =~: 正则表达式匹配
- !~: 正则表达式不匹配

下面的表达式筛选出了 environment 为 staging 或 testing 或 development，并且 method 不是 GET 的时序：

```
http_requests_total{environment=~"staging|testing|development",method!="GET"}
```

度量指标名可以使用内部标签 `__name__` 来匹配，表达式 `http_requests_total` 也可以写成 `{__name__="http_requests_total"}`。表达式 `{__name__=~"job:.*"}` 匹配所有度量指标名称以 `job:` 打头的时序。

#### 3.2.2. 区间向量选择器

区间向量选择器类似于瞬时向量选择器，不同的是它选择的是过去一段时间的采样值。可以通过在瞬时向量选择器后面添加包含在 `[]` 里的时长来得到区间向量选择器。比如下面的表达式选出了所有度量指标为 http_requests_total 且 job 为 prometheus 的时序在过去 5 分钟的采样值。

```
http_requests_total{job="prometheus"}[5m]
```

时长的单位可以是下面几种之一：

- s：seconds
- m：minutes
- h：hours
- d：days
- w：weeks
- y：years

#### 3.2.3. 偏移修饰器

前面介绍的选择器默认都是以当前时间为基准时间，偏移修饰器用来调整基准时间，使其往前偏移一段时间。偏移修饰器紧跟在选择器后面，使用 `offset` 来指定要偏移的量。比如下面的表达式选择度量名称为 http_requests_total 的所有时序在 5 分钟前的采样值。

```
http_requests_total offset 5m
```

下面的表达式选择 http_requests_total 度量指标在 1 周前的这个时间点过去 5 分钟的采样值。

```
http_requests_total[5m] offset 1w
```

## 4. PromQL 操作符

下面我们将会学习PromQL 操作符。

### 4.1. 二元操作符

PromQL 的二元操作符支持基本的逻辑和算术运算，包含算术类、比较类和逻辑类三大类。

#### 4.1.1. 算术类二元操作符

算术类二元操作符有以下几种：

- \+：加
- \-：减
- \*：乘
- /：除
- %：求余
- ^：乘方

算术类二元操作符可以使用在标量与标量、向量与标量，以及向量与向量之间。

> 二元操作符上下文里的向量特指瞬时向量，不包括区间向量。

- 标量与标量之间，结果很明显，跟通常的算术运算一致。
- 向量与标量之间，相当于把标量跟向量里的每一个标量进行运算，这些计算结果组成了一个新的向量。
- 向量与向量之间，会稍微麻烦一些。运算的时候首先会为左边向量里的每一个元素在右边向量里去寻找一个匹配元素（匹配规则后面会讲），然后对这两个匹配元素执行计算，这样每对匹配元素的计算结果组成了一个新的向量。如果没有找到匹配元素，则该元素丢弃。

#### 4.1.2. 比较类二元操作符

比较类二元操作符有以下几种：

- == (equal)
- != (not-equal)
- \> (greater-than)
- < (less-than)
- \>= (greater-or-equal)
- <= (less-or-equal)

比较类二元操作符同样可以使用在标量与标量、向量与标量，以及向量与向量之间。默认执行的是过滤，也就是保留值。可以通过在运算符后面跟 `bool` 修饰符来使得返回值 0 和 1，而不是过滤。

- 标量与标量之间，必须跟 bool 修饰符，因此结果只可能是 0（false） 或 1（true）。
- 向量与标量之间，相当于把向量里的每一个标量跟标量进行比较，结果为真则保留，否则丢弃。如果后面跟了 bool 修饰符，则结果分别为 1 和 0。
- 向量与向量之间，运算过程类似于算术类操作符，只不过如果比较结果为真则保留左边的值（包括度量指标和标签这些属性），否则丢弃，没找到匹配也是丢弃。如果后面跟了 bool 修饰符，则保留和丢弃时结果相应为 1 和 0。

#### 4.1.3. 逻辑类二元操作符

逻辑操作符仅用于向量与向量之间。

- and：交集
- or：合集
- unless：补集

具体运算规则如下：

- `vector1 and vector2` 的结果由在 vector2 里有匹配（标签键值对组合相同）元素的 vector1 里的元素组成。
- `vector1 or vector2` 的结果由所有 vector1 里的元素加上在 vector1 里没有匹配（标签键值对组合相同）元素的 vector2 里的元素组成。
- `vector1 unless vector2` 的结果由在 vector2 里没有匹配（标签键值对组合相同）元素的 vector1 里的元素组成。

#### 4.1.4. 二元操作符优先级

PromQL 的各类二元操作符运算优先级如下：

1. ^
2. *, /, %
3. +, -
4. ==, !=, <=, <, >=, >
5. and, unless
6. or

### 4.2. 向量匹配

前面算术类和比较类操作符都需要在向量之间进行匹配。共有两种匹配类型，`one-to-one` 和 `many-to-one` / `one-to-many`。

#### 4.2.1. One-to-one 向量匹配

这种匹配模式下，两边向量里的元素如果其标签键值对组合相同则为匹配，并且只会有一个匹配元素。可以使用 `ignoring` 关键词来忽略不参与匹配的标签，或者使用 `on` 关键词来指定要参与匹配的标签。语法如下：

```
<vector expr> <bin-op> ignoring(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) <vector expr>
```

比如对于下面的输入：

```
method_code:http_errors:rate5m{method="get", code="500"}  24
method_code:http_errors:rate5m{method="get", code="404"}  30
method_code:http_errors:rate5m{method="put", code="501"}  3
method_code:http_errors:rate5m{method="post", code="500"} 6
method_code:http_errors:rate5m{method="post", code="404"} 21

method:http_requests:rate5m{method="get"}  600
method:http_requests:rate5m{method="del"}  34
method:http_requests:rate5m{method="post"} 120
```

执行下面的查询：

```
method_code:http_errors:rate5m{code="500"} / ignoring(code) method:http_requests:rate5m
```

得到的结果为：

```
{method="get"}  0.04            //  24 / 600
{method="post"} 0.05            //   6 / 120
```

也就是每一种 method 里 code 为 500 的请求数占总数的百分比。由于 method 为 put 和 del 的没有匹配元素所以没有出现在结果里。

#### 4.2.2. Many-to-one / one-to-many 向量匹配

这种匹配模式下，某一边会有多个元素跟另一边的元素匹配。这时就需要使用 `group_left` 或 `group_right` 组修饰符来指明哪边匹配元素较多，左边多则用 group_left，右边多则用 group_right。其语法如下：

```
<vector expr> <bin-op> ignoring(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> ignoring(<label list>) group_right(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_left(<label list>) <vector expr>
<vector expr> <bin-op> on(<label list>) group_right(<label list>) <vector expr>
```

> 组修饰符只适用于算术类和比较类操作符。

对于前面的输入，执行下面的查询：

```
method_code:http_errors:rate5m / ignoring(code) group_left method:http_requests:rate5m
```

将得到下面的结果：

```
{method="get", code="500"}  0.04            //  24 / 600
{method="get", code="404"}  0.05            //  30 / 600
{method="post", code="500"} 0.05            //   6 / 120
{method="post", code="404"} 0.175           //  21 / 120
```

也就是每种 method 的每种 code 错误次数占每种 method 请求数的比例。这里匹配的时候 ignoring 了 code，才使得两边可以形成 Many-to-one 形式的匹配。由于左边多，所以需要使用 group_left 来指明。

> Many-to-one / one-to-many 过于高级和复杂，要尽量避免使用。很多时候通过 ignoring 就可以解决问题。

### 4.3. 聚合操作符

PromQL 的聚合操作符用来将向量里的元素聚合得更少。总共有下面这些聚合操作符：

- sum：求和
- min：最小值
- max：最大值
- avg：平均值
- stddev：标准差
- stdvar：方差
- count：元素个数
- count_values：等于某值的元素个数
- bottomk：最小的 k 个元素
- topk：最大的 k 个元素
- quantile：分位数

聚合操作符语法如下：

```
<aggr-op>([parameter,] <vector expression>) [without|by (<label list>)]
```

其中 `without` 用来指定不需要保留的标签（也就是这些标签的多个值会被聚合），而 `by` 正好相反，用来指定需要保留的标签（也就是按这些标签来聚合）。

下面来看几个示例：

```
sum(http_requests_total) without (instance)
```

http_requests_total 度量指标带有 application、instance 和 group 三个标签。上面的表达式会得到每个 application 的每个 group 在所有 instance 上的请求总数。效果等同于下面的表达式：

```
sum(http_requests_total) by (application, group)
```

下面的表达式可以得到所有 application 的所有 group 的所有 instance 的请求总数。

```
sum(http_requests_total)
```

## 5. PromQL 函数

Prometheus 内置了一些函数来辅助计算，下面介绍一些典型的，完整的列表请参考 [官方文档](https://prometheus.io/docs/prometheus/latest/querying/functions/)。

- abs()：绝对值
- sqrt()：平方根
- exp()：指数计算
- ln()：自然对数
- ceil()：向上取整
- floor()：向下取整
- round()：四舍五入取整
- delta()：计算区间向量里每一个时序第一个和最后一个的差值
- sort()：排序

## 6. 实验总结

本次实验我们对 Prometheus 的查阅语言 PromQL 有了细致的学习，数据我们可以通过表达式查询得到了，那么怎么展示它们了？接下来我们将学习 Prometheus 的数据可视化。