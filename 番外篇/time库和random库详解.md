## time库和random库详解

在Python中，处理时间和生成随机数是最常见的两类需求：程序计时要用到**time库**，模拟、游戏、抽样等场景则离不开**random库**。这两个库都是Python的内置标准库，不需要额外安装，`import`之后就能直接使用。下面我们结合嵩天《Python语言程序设计基础（第3版）》中的内容，把这两个库的核心知识点梳理一遍。

### time库和程序计时

#### 时间戳的概念

**时间戳**（timestamp）是指从1970年1月1日00:00:00（格林威治时间，也就是所谓"Unix纪元"）到当前时刻的总秒数，通常是一个浮点数。计算机内部就用这样一个数字来表示"某一时刻"，它不受时区、格式的影响，非常适合用来做时间的计算和比较。

```python
import time

print(time.time())  # 获取当前时间的时间戳（浮点数，秒）
```

输出：

```text
1794028800.123456
```

> **说明**：每次运行得到的数值都不一样，因为它代表的是"当下"这个时刻。时间戳只在1970年之后才有意义，如果传入负数表示1970年之前的时间，部分平台可能不支持。

#### time库的三类函数

time库的功能可以归为三类：

| 类别 | 函数 | 作用 |
| --- | --- | --- |
| 时间获取 | `time()`、`ctime()`、`gmtime()`、`localtime()` | 获取时间戳或以不同形式表示时间 |
| 时间格式化 | `strftime()`、`strptime()` | 在"时间对象/时间戳"和"字符串"之间互相转换 |
| 程序计时 | `sleep()`、`perf_counter()` | 让程序暂停，或精确测量一段代码的耗时 |

#### 时间获取函数

- `time.time()`：获取当前时间戳，返回浮点数。
- `time.ctime([secs])`：把一个时间戳转换成**人类易读的字符串**；不传参数时默认取当前时间。
- `time.gmtime([secs])`：把时间戳转换成**格林威治时间**（UTC）的`struct_time`对象。
- `time.localtime([secs])`：把时间戳转换成**本地时间**的`struct_time`对象。

```python
import time

print(time.time())           # 当前时间戳
print(time.ctime())          # 当前时间的易读字符串
print(time.ctime(0))         # 时间戳0对应的时刻（Unix纪元）
t = time.localtime()         # 当前本地时间的struct_time对象
print(t.tm_year, t.tm_mon, t.tm_mday)  # 分别取年、月、日
print(time.gmtime(0))        # 时间戳0的UTC时间结构体
```

输出：

```text
1794028800.123456
Fri Nov  7 10:00:00 2026
Thu Jan  1 00:00:00 1970
2026 11 7
time.struct_time(tm_year=1970, tm_mon=1, tm_mday=1, tm_hour=0, tm_min=0, tm_sec=0, tm_wday=3, tm_yday=1, tm_isdst=0)
```

> **说明**：`struct_time`是一个带字段名的时间元组，常用的字段有`tm_year`（年）、`tm_mon`（月，1~12）、`tm_mday`（日）、`tm_hour`（时）、`tm_min`（分）、`tm_sec`（秒）、`tm_wday`（星期，0表示周一）、`tm_yday`（一年中的第几天）。`gmtime()`和`localtime()`的区别只在于时区：前者是UTC时间，后者是我们本地时区的时间。

#### 时间格式化：strftime和strptime

`time.strftime(format[, t])`把一个时间对象（`struct_time`）按照**格式控制符**格式化成字符串；`time.strptime(string, format)`则反过来，把字符串按照给定的格式**解析**成`struct_time`对象。两者是一对互逆操作。

常用格式控制符如下：

| 格式控制符 | 含义 | 示例 |
| --- | --- | --- |
| `%Y` | 四位年份 | 2026 |
| `%m` | 月份（01~12） | 11 |
| `%d` | 日期（01~31） | 07 |
| `%H` | 小时（24小时制，00~23） | 10 |
| `%M` | 分钟（00~59） | 30 |
| `%S` | 秒（00~59） | 45 |
| `%a` | 星期简称 | Sat |
| `%A` | 星期全称 | Saturday |
| `%b` | 月份简称 | Nov |
| `%B` | 月份全称 | November |
| `%p` | 上午/下午 | AM、PM |
| `%f` | 微秒（6位数字） | 123456 |

```python
import time

t = time.localtime()
# 把当前时间格式化成常见格式
print(time.strftime('%Y-%m-%d %H:%M:%S', t))   # 年-月-日 时:分:秒
print(time.strftime('%Y年%m月%d日 %H时%M分%S秒', t))  # 中文格式
print(time.strftime('%a, %b %d %Y', t))        # 星期简称, 月份简称 日 年

# 把字符串解析成struct_time对象
st = time.strptime('2026-11-07 10:30:45', '%Y-%m-%d %H:%M:%S')
print(st.tm_year, st.tm_mon, st.tm_mday)       # 从解析结果中取年、月、日
```

输出：

```text
2026-11-07 10:30:45
2026年11月07日 10时30分45秒
Sat, Nov 07 2026
2026 11 7
```

> **提示**：`strftime`中的格式控制符和字符串里的普通字符（如`-`、`:`或中文）可以自由组合，普通字符会原样输出。解析时，`string`的写法必须和`format`严格对应，否则会抛出`ValueError`。

#### 程序计时

程序计时主要用到两个函数：

- `time.sleep(s)`：让程序**暂停**`s`秒，`s`可以是整数也可以是浮点数。
- `time.perf_counter()`：返回一个**性能计数器**的值，它以最高的精度计时，起点任意（通常是开机或进程启动时间），两次调用取差值就得到了中间这段代码的耗时。测量代码运行时间应该优先用它，而不是`time.time()`。

```python
import time

start = time.perf_counter()   # 记录起始时刻
for i in range(1000000):      # 模拟一段耗时操作
    pass
end = time.perf_counter()     # 记录结束时刻
print('耗时：{:.4f}秒'.format(end - start))  # 差值即代码耗时
```

输出：

```text
耗时：0.0213秒
```

> **说明**：测量耗时的关键代码就是一个模板——`start = time.perf_counter()`，执行被测代码，然后`end - start`就是耗时。`time.sleep()`则常用于模拟耗时操作（如下载）、控制循环节奏，或者让进度条刷新得慢一点方便观察。

#### 实例：文本进度条

文本进度条是time库的经典应用，我们分三步由简到繁来实现。

**第一步：简单地打印进度**

```python
import time

total = 10                     # 总共要完成的任务数
for i in range(total + 1):     # 从0%打印到100%
    percent = i / total * 100  # 计算完成百分比
    print('进度：{:.0f}%'.format(percent))  # 每次打印一行
    time.sleep(0.1)            # 休眠0.1秒模拟任务执行
```

输出（每次打印会换行，占用多行）：

```text
进度：0%
进度：10%
进度：20%
...
进度：100%
```

**第二步：用`\r`实现单行刷新**

`\r`是**回车符**，它的作用是把光标移回当前行的行首但不换行。配合`print()`的`end=''`参数（不自动换行）和`flush=True`参数（立即刷新输出缓冲区），就可以让进度信息在**同一行**上不断更新。

```python
import time

total = 10
for i in range(total + 1):
    percent = i / total * 100          # 完成百分比
    print('\r进度：{:.0f}%'.format(percent), end='', flush=True)  # \r回到行首，单行刷新
    time.sleep(0.1)
print()                                # 循环结束后换行，避免后面的输出接在同一行
```

输出（始终只占一行，不断被覆盖更新）：

```text
进度：100%
```

> **提示**：`flush=True`很关键。如果不刷新缓冲区，`\r`后面的内容可能会滞留在缓冲区中，导致进度条"卡住"不更新。另外`\r`只能覆盖当前行，新内容比旧内容短时行尾会留下残影，所以实际使用时常用`'{:=<20}'`这类格式控制把输出填充成固定宽度。

**第三步：完整版进度条（百分数+进度条+耗时+预计剩余时间）**

```python
import time

scale = 50                             # 进度条的总宽度（50个字符）
start = time.perf_counter()            # 记录开始时刻
for i in range(scale + 1):
    done = '*' * i                     # 已完成部分
    rest = '.' * (scale - i)           # 未完成部分
    percent = i / scale * 100          # 完成百分比
    dur = time.perf_counter() - start  # 已经消耗的时间
    # 预计剩余时间 = 已耗时 / 完成比例 * 剩余比例
    remain = dur * (scale - i) / i if i else 0
    print('\r{:^3.0f}%[{}->{}]{:.2f}s 剩余{:.2f}s'.format(
        percent, done, rest, dur, remain), end='', flush=True)
    time.sleep(0.1)                    # 模拟任务执行
print('\n任务完成！')                   # 结束后换行并提示
```

输出：

```text
                    100%[**************************************************->]5.10s 剩余0.00s
任务完成！
```

> **说明**：这个版本每次刷新都显示完成百分比、用`*`和`.`拼出的进度条图形、已消耗的时间以及根据"匀速假设"估算的剩余时间。`'{:^3.0f}%'`中的`^`表示居中对齐，让百分数看起来更整齐。预计剩余时间的公式是：已耗时 ÷ 完成比例 × 剩余比例，一开始`i`为0时不能直接除，所以做了特殊处理。

### random库和随机数运用

#### random库概述

random库是Python标准库中用于生成**随机数**的库。严格来说，计算机产生的随机数是**伪随机数**：它由确定的算法（梅森旋转算法）计算得出，只要算法和**随机数种子**确定，生成的随机数序列就是完全相同的。如果不手动设置种子，random库默认使用系统时间作为种子，所以每次运行的结果看起来"随机"；一旦用`random.seed(a)`设置了固定种子，每次运行程序都会得到同一批随机数，这在程序调试和结果复现时非常有用。

```python
import random

random.seed(10)               # 固定随机数种子
print(random.random())        # 产生0~1之间的随机小数
random.seed(10)               # 重新设置相同的种子
print(random.random())        # 得到和上面完全相同的值
```

输出：

```text
0.5714025946899135
0.5714025946899135
```

#### random库的常用函数

| 函数 | 作用 |
| --- | --- |
| `seed(a=None)` | 初始化随机数种子，默认用系统时间 |
| `random()` | 生成`[0.0, 1.0)`之间的随机浮点数 |
| `randint(a, b)` | 生成`[a, b]`之间的随机整数（两端都能取到） |
| `randrange(start, stop[, step])` | 从`range(start, stop, step)`中取一个随机整数（不含stop） |
| `uniform(a, b)` | 生成`[a, b]`之间的随机浮点数 |
| `choice(seq)` | 从序列中随机选取一个元素 |
| `shuffle(x)` | 将序列`x`原地打乱（随机排序） |
| `sample(pop, k)` | 从总体`pop`中随机抽取`k`个不重复的元素，返回列表 |

```python
import random

print(random.random())                 # [0.0, 1.0)的随机小数
print(random.randint(1, 6))            # 掷骰子：1~6的随机整数
print(random.randrange(0, 100, 5))     # 从0、5、10、...、95中随机取一个
print(random.uniform(2.5, 3.5))        # 2.5~3.5之间的随机浮点数
print(random.choice(['苹果', '香蕉', '橘子']))  # 从列表中随机取一个元素

cards = list(range(1, 11))             # 1~10的数字牌
random.shuffle(cards)                  # 原地打乱顺序，相当于洗牌
print(cards)
print(random.sample(range(1, 34), 5))  # 从1~33中不重复地抽5个数（如选球号码）
```

输出：

```text
0.5714025946899135
4
85
3.2379628402187853
香蕉
[3, 8, 1, 10, 5, 7, 2, 9, 4, 6]
[11, 27, 3, 19, 30]
```

> **说明**：注意`randint(a, b)`和`randrange(start, stop)`的边界区别——前者`b`可以取到，后者`stop`取不到，这是初学者最容易踩的坑。`shuffle()`是原地修改，返回值是`None`；`sample()`不会修改原序列，适合做"不放回抽样"。

#### 实例：蒙特卡洛方法求圆周率π

**问题分析**：蒙特卡洛方法是用随机模拟来求解数学问题的方法。求π可以用**撒点法**：在边长为1的正方形内随机撒点，正方形的一个顶点处有一个四分之一圆（半径为1）。落在四分之一圆内的点数与总点数之比，就近似等于两者的面积之比，即：

$$\frac{\pi}{4} \approx \frac{落在圆内的点数}{总点数}$$

所以 $\pi \approx 4 \times \frac{落在圆内的点数}{总点数}$，撒的点越多，结果越精确。

```python
import random

total = 1000000          # 撒点的总数量，越多越精确
inside = 0               # 落在四分之一圆内的点的数量
for _ in range(total):
    x = random.random()  # 在[0,1)x[0,1)的正方形内随机取一点
    y = random.random()
    if x * x + y * y <= 1:  # 到原点距离不超过1，即落在四分之一圆内
        inside += 1
pi = 4 * inside / total  # 用频率近似面积之比，估算圆周率
print('估算的圆周率：{:.6f}'.format(pi))
```

输出：

```text
估算的圆周率：3.142964
```

> **提示**：每次运行的结果会略有不同，因为撒点是随机的。想让它可复现，可以在循环前加一句`random.seed(1)`。把`total`增大到一千万甚至更多，估算值会更加接近3.141593，但耗时也会相应增加，这时可以配合上面学过的`time.perf_counter()`来测一测程序的耗时。

###  总结

time库负责时间处理：`time()`获取时间戳，`ctime()`得到易读字符串，`localtime()`/`gmtime()`得到时间结构体，`strftime()`/`strptime()`在字符串和时间对象之间互相转换，`sleep()`暂停程序，`perf_counter()`精确计时，用它`end - start`的差值就能测量代码耗时，文本进度条就是这些函数的典型组合应用。random库负责随机数：本质是伪随机数，种子相同则序列相同；`random()`生成0~1的小数，`randint()`/`randrange()`生成随机整数，`uniform()`生成区间内的随机浮点数，`choice()`随机取元素，`shuffle()`洗牌，`sample()`抽样；蒙特卡洛撒点法求π则展示了随机模拟解决数学问题的思路。
