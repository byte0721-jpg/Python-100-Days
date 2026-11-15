## C 到 Python 转换指南

很多朋友之前学过 C 语言，第一次接触 Python 时总觉得"这一行怎么少了一半"。其实 C 和 Python 的差异主要集中在**书写形式**和**内存观念**这两件事上，理解了这两点，剩下的知识点几乎都能一一对上。这份指南就用"C 代码 vs Python 代码"对照的方式，帮大家把已有的 C 语言知识平移到 Python 里来。

### 1. 程序结构：从 main 函数到模块

C 程序必须有 `main` 函数作为入口，编译前还要用 `#include` 引入头文件。Python 是**解释执行**的脚本语言，代码从上到下逐行运行，不需要编译和链接，也没有必须存在的 `main` 函数。如果希望某段代码只在"直接运行本文件"时执行、被别人 `import` 时不执行，就用惯用的 `if __name__ == '__main__'` 判断。

**C 的写法：**

```c
#include <stdio.h>

int main(void) {
    printf("hello\n");
    return 0;
}
```

**Python 的写法：**

```python
def main():
    print('hello')  # 打印一行文字，print 自带换行

if __name__ == '__main__':  # 直接运行本文件时条件成立
    main()  # 调用主函数
```

两者的结构差异可以用表格来对照：

| C 语言 | Python | 说明 |
| --- | --- | --- |
| `#include <stdio.h>` | `import math` | 引入标准库，Python 引入的是模块 |
| `int main(void) {...}` | 顶层代码或 `if __name__ == '__main__':` | Python 没有强制入口函数 |
| 编译（gcc）+ 链接 | 直接运行 `python xxx.py` | 解释执行，没有编译环节 |
| `{ }` 界定代码块 | 缩进（通常 4 个空格）+ 冒号 | 缩进是语法的一部分 |
| 每条语句以 `;` 结尾 | 不需要分号，一行一条语句 | 一般不写分号 |

Python 标准库里常用模块和 C 标准库的对应关系，在后面的"标准库对照"一节会专门整理。

### 2. 变量与类型：从声明到动态类型

C 语言里变量必须**先声明类型再使用**，Python 是**动态类型**语言，变量不需要声明，赋值即创建，而且同一个变量可以随时指向不同类型的对象。

**C 的写法：**

```c
int a = 10;        // 声明整型变量
double price = 5.5;
char grade = 'A';  // 单个字符
```

**Python 的写法：**

```python
a = 10            # 变量无需声明类型，赋什么就是什么
price = 5.5       # 浮点数
grade = 'A'       # Python 没有 char 类型，单个字符就是长度为 1 的字符串
a = 'hello'       # 同一个变量可以改指另一种类型的对象
```

基本类型的对照关系如下表：

| C 语言类型 | Python 类型 | 差异说明 |
| --- | --- | --- |
| `int` / `long` | `int` | Python 的整数**没有溢出**，可以任意大 |
| `float` / `double` | `float` | Python 的 float 相当于 C 的 double |
| `char` | 没有对应类型 | 长度为 1 的 `str` 就是"字符" |
| `char s[]` / `char*` | `str` | Python 的字符串**不可变** |
| `int arr[10]` | `list` | Python 列表长度可变、可混装类型 |
| `struct` | `class` / `dict` | 见第 7 节 |

> **提示**：Python 的 `int` 不会溢出这一点对 C 程序员很友好，再也不用担心 `int` 乘法越界了，超大数直接写就行，例如 `print(2 ** 100)` 会输出一个 31 位的数。

类型转换方面，C 支持隐式转换（如 `int` 自动转 `double`），Python 中**不同类型之间不能隐式运算**，最典型的是从键盘读到的都是字符串，必须显式转换：

```python
s = '123'
x = int(s)       # str 转 int，必须显式调用 int()
y = float('3.5') # str 转 float
z = str(100)     # int 转 str
print(x + 1, y, z)  # 结果：124 3.5 100
# print(s + 1)   # 错误！str 和 int 不能相加，会报 TypeError
```

C 用 `const` 定义常量，Python **没有真正的常量**，习惯上用全大写命名表示"请大家不要改"：

```python
PI = 3.1415926      # 全大写命名，约定俗成的"常量"
MAX_SIZE = 1000
```

### 3. 运算符：长得像但脾气不同

大部分算术运算符 C 和 Python 一致，但除法差异最大：C 中两个 `int` 相除还是 `int`（向零取整），Python 的 `/` 永远返回 `float`，想取整要使用 `//`（向下取整，也叫地板除）。

```python
print(7 / 2)    # 结果：3.5（Python 的 / 总是返回浮点数）
print(7 // 2)   # 结果：3（整除，相当于 C 的整数除法）
print(-7 // 2)  # 结果：-4（// 是向下取整，注意和 C 向零取整不同）
print(7 % 2)    # 结果：1（取余）
print(2 ** 10)  # 结果：1024（** 是幂运算，C 里要调用 pow 函数）
```

C 语言的 `i++`、`i--` 在 Python 里**不存在**。原因很简单：Python 中的整数是**不可变对象**，`i = i + 1` 是让名字 `i` 指向一个新的整数对象，而不是"修改 i 的值"，所以 `++` 这种"原地自增并返回旧值"的语义在这里讲不通。自增请老老实实写 `i += 1`。

```python
i = 0
i += 1   # 正确的自增写法，相当于 C 的 i++（或 ++i 的取值部分）
i += 2   # 相当于 i = i + 2
print(i) # 结果：3
```

逻辑运算符也不一样：C 用 `&&`、`||`、`!`，Python 用 `and`、`or`、`not`；而且 Python 的逻辑运算**返回的是操作数本身**，不是 0 和 1：

```python
print(3 and 5)    # 结果：5（and 返回最后一个为真的操作数）
print(0 or 'hi')  # 结果：hi（or 返回第一个为真的操作数）
print(not 0)      # 结果：True
```

这带来一个 C 程序员熟悉的技巧的 Python 版本——给变量设默认值：

```python
name = ''            # 假设输入为空
name = name or '匿名'  # 为空时取后面的默认值
print(name)          # 结果：匿名
```

三目运算符的写法也不同，C 是 `条件 ? 值1 : 值2`，Python 是 `值1 if 条件 else 值2`：

```python
age = 20
status = '成年' if age >= 18 else '未成年'  # 条件成立取前面的值
print(status)      # 结果：成年
```

最后是 Python 独有的**链式比较**，可以写出和数学一模一样的判断：

```python
score = 85
print(80 <= score < 90)          # 结果：True，相当于 80 <= score and score < 90
print(0 < score <= 100)          # 结果：True
```

### 4. 控制流：缩进替代大括号

#### if-else 对照

条件**不需要括号**，语句块用冒号加缩进表示：

```c
// C 的写法
if (score >= 90) {
    printf("优秀\n");
} else if (score >= 60) {
    printf("及格\n");
} else {
    printf("不及格\n");
}
```

```python
# Python 的写法
score = 85
if score >= 90:          # 注意：条件不写括号，行尾有冒号
    print('优秀')
elif score >= 60:        # C 的 else if 在 Python 里写作 elif
    print('及格')
else:
    print('不及格')       # 输出：及格
```

#### for 循环对照

这是思维转换最大的一处。C 的 `for` 是"初始化; 条件; 更新"三段式，本质上是计数循环；Python 的 `for` 是**for-in**结构，遍历一个可迭代对象。想按次数循环就用 `range(start, stop, step)`，注意 `stop` **不包含在范围内**：

```c
// C 的写法
for (int i = 0; i < 5; i++) {
    printf("%d ", i);
}
// 输出：0 1 2 3 4
```

```python
# Python 的写法
for i in range(5):       # range(5) 产生 0,1,2,3,4，不含 5
    print(i, end=' ')    # end=' ' 让 print 不换行，用空格分隔
# 输出：0 1 2 3 4

for i in range(2, 10, 3):  # 从 2 开始到 10 结束（不含），步长 3
    print(i, end=' ')
# 输出：2 5 8

for ch in 'abc':           # 直接遍历字符串，不用下标
    print(ch, end=' ')
# 输出：a b c
```

> **说明**：确实需要下标时（比如 C 里惯用的 `arr[i]` 遍历），用 `enumerate` 更地道，它会同时给出序号和元素，比 `for i in range(len(arr))` 更推荐。

#### for-else

Python 的循环可以接一个 `else` 块，循环**没有被 break 打断**（正常跑完）时执行，常用来做"找没找到"的判断：

```python
nums = [2, 4, 6, 8]
for n in nums:
    if n % 2 != 0:       # 找奇数
        print('列表中有奇数')
        break
else:                    # 没有 break 才会走到这里
    print('全是偶数')     # 输出：全是偶数
```

#### while 对照

while 几乎一模一样，只是不要括号和大括号：

```python
count = 3
while count > 0:         # 条件不写括号
    print(count)
    count -= 1           # 没有 count--，用 -= 1
# 输出：3 2 1
```

#### switch-case 的替代

C 的 `switch-case` 在 Python 中没有直接对应，常见的替代方案有三种：`if-elif` 链、字典映射函数表，以及 Python 3.10 新增的 `match` 语句。

**字典映射函数表**（函数作为值存进字典，这是 Python 很常用的手法）：

```python
def add(a, b):
    return a + b        # 返回两数之和

def sub(a, b):
    return a - b        # 返回两数之差

ops = {'+': add, '-': sub}        # 字典把运算符映射到函数
print(ops['+'](3, 5))             # 结果：8，取出函数并调用
```

**match 语句**（Python 3.10 及以上版本可用，写法和 C 的 switch 很像）：

```python
command = 'stop'
match command:           # 结构上类似 switch
    case 'start':
        print('启动')     # 匹配到就走对应分支
    case 'stop':
        print('停止')     # 输出：停止
    case _:              # 下划线相当于 C 的 default
        print('未知命令')
```

#### 其他控制流

| C 语言 | Python | 说明 |
| --- | --- | --- |
| `break` | `break` | 用法完全相同，跳出本层循环 |
| `continue` | `continue` | 用法完全相同，跳过本次循环 |
| `goto` | **没有** | Python 不提供 goto，用函数、循环和异常来组织跳转逻辑 |

### 5. 函数：从指针传参到一切皆对象

定义函数用 `def`，不需要写返回类型和参数类型（Python 3 之后可以选择性地写类型提示）。C 的 `void` 函数其实返回 `None`，这和 C 里的"无返回值"概念要区分开：

```python
def show():              # 没写 return 的函数返回 None
    print('你好')

result = show()          # 调用函数
print(result)            # 结果：None（相当于 C 的 void，但有一个真实的返回值）
```

**默认参数**是 C 语言没有的便利特性，调用时可以省略参数：

```python
def greet(name, message='你好'):  # message 有默认值
    print(f'{message}, {name}!')

greet('骆昊')               # 结果：你好, 骆昊!（省略了 message）
greet('骆昊', '早上好')      # 结果：早上好, 骆昊!（覆盖默认值）
```

C 用 `...` 和 `va_list` 实现可变参数，Python 直接用 `*args` 和 `**kwargs`：

```python
def total(*args):                # args 是一个元组，收集所有位置参数
    return sum(args)

print(total(1, 2, 3, 4))         # 结果：10，想传几个就传几个

def info(**kwargs):              # kwargs 是一个字典，收集所有关键字参数
    for key, value in kwargs.items():
        print(f'{key}={value}')

info(name='骆昊', age=40)        # 输出：name=骆昊 和 age=40 两行
```

C 语言函数只能返回一个值，想"返回多个"要靠指针参数或结构体；Python 直接返回一个**元组**，多个值一次搞定：

```python
def min_max(nums):
    return min(nums), max(nums)   # 返回两个值，其实是一个元组

low, high = min_max([3, 1, 4, 1, 5])  # 元组解包，一次接收两个值
print(low, high)                 # 结果：1 5
```

传参语义是最容易误解的地方。Python **没有指针传参**，一切传参都是"把对象的引用传进去"，但效果类似 C 的"传值传引用混合体"：传入**不可变对象**（int、str、tuple）时函数内改不动外面的变量；传入**可变对象**（list、dict）时函数内的修改会影响外面：

```python
def modify(num, lst):
    num = 100            # 只是让局部变量 num 指向新对象，不影响外面
    lst.append(4)        # lst 和外面是同一个列表对象，修改会反映到外面

a = 1
b = [1, 2, 3]
modify(a, b)
print(a, b)              # 结果：1 [1, 2, 3, 4]，a 没变，b 被修改了
```

> **提示**：C 程序员可以把可变对象理解为"自动帮你传了指针"，把不可变对象理解为"传值"。想避免函数修改调用者的列表，传 `b[:]` 或 `b.copy()` 这样的副本进去。

作用域方面，函数内部想修改**模块级的全局变量**要声明 `global`（这一点和 C 的全局变量直接可改不同）：

```python
count = 0

def increase():
    global count         # 声明使用全局变量 count
    count += 1           # 修改的是全局变量

increase()
increase()
print(count)             # 结果：2
```

### 6. 数组与字符串：list 和 str 的全套功夫

C 的定长数组在 Python 里对应 `list`，创建、长度、下标访问都很直接，还支持负下标（从后往前数）：

```python
arr = [10, 20, 30, 40]   # 创建列表，相当于 C 的数组
print(len(arr))          # 结果：4（len 相当于 C 里自己记的数组长度）
print(arr[0], arr[-1])   # 结果：10 40（-1 表示最后一个元素）
arr.append(50)           # 追加元素，C 数组做不到，长度会变
print(arr)               # 结果：[10, 20, 30, 40, 50]
```

**二维数组**对应"列表的列表"，注意初始化方式，不要用 `[[0] * 3] * 3` 这种写法（三个行会是同一个列表的引用）：

```python
matrix = [[0] * 3 for _ in range(3)]  # 3 行 3 列的二维列表，每行是独立的列表
matrix[1][2] = 9                      # 下标从 0 开始，和 C 一致
print(matrix)                         # 结果：[[0, 0, 0], [0, 0, 9], [0, 0, 0]]
```

Python 的字符串和 C 的 `char[]` 最大的区别是**不可变**：不能通过下标修改某个字符，所有"修改"操作都是生成新字符串：

```python
s = 'hello'
# s[0] = 'H'            # 错误！字符串不可变，会报 TypeError
s = 'H' + s[1:]          # 正确做法：拼一个新字符串
print(s)                 # 结果：Hello
print(s + ' World')      # 结果：Hello World（+ 拼接，不用 strcat）
print(s * 2)             # 结果：HelloHello（乘法重复）
```

遍历字符串用 for-in，比 C 的 `for (i = 0; s[i] != '\0'; i++)` 简单得多，因为 Python 字符串不需要 `\0` 结尾符，长度由对象自己记录：

```python
for ch in 'abc':         # 逐字符遍历
    print(ch, end=' ')
# 输出：a b c
```

C 里要引入 `string.h` 才能做的事，Python 用字符串方法直接完成：

| string.h 函数 | Python 方法 | 示例 |
| --- | --- | --- |
| `strlen(s)` | `len(s)` | `len('hello')` 得 `5` |
| `strstr(s, t)` | `s.find(t)` | `'hello'.find('ll')` 得 `2`，找不到返回 `-1` |
| `strtok` / 拆分 | `s.split(sep)` | `'a,b,c'.split(',')` 得 `['a', 'b', 'c']` |
| 手写修剪循环 | `s.strip()` | `'  hi  '.strip()` 得 `'hi'` |
| 逐字符替换循环 | `s.replace(a, b)` | `'a-b'.replace('-', '+')` 得 `'a+b'` |
| `strcpy` + `strcat` | `+` 和 `join` | `''.join(['a', 'b'])` 得 `'ab'` |

**切片**是 Python 的招牌操作，C 里要手写循环才能完成的"取子串"，Python 一个表达式搞定：

```python
s = 'hello world'
print(s[0:5])     # 结果：hello（下标 0 到 4，不含 5）
print(s[:5])      # 结果：hello（省略起点表示从 0 开始）
print(s[6:])      # 结果：world（省略终点表示取到末尾）
print(s[::-1])    # 结果：dlrow olleh（步长 -1，即反转字符串）
print(s[::2])     # 结果：hlowrd（每隔一个字符取一个）
```

`printf` 在 Python 里对应 f-string（格式化字符串字面量），变量直接写进 `{}`：

```c
// C 的写法
printf("name=%s, age=%d\n", "骆昊", 40);
printf("pi=%.2f\n", 3.14159);
```

```python
# Python 的写法
name, age, pi = '骆昊', 40, 3.14159
print(f'name={name}, age={age}')  # 结果：name=骆昊, age=40
print(f'pi={pi:.2f}')             # 结果：pi=3.14（:.2f 和 C 的 %.2f 含义相同）
```

### 7. 结构体与对象：从 struct 到 class

C 的 `struct` 只是把数据捆在一起，函数要单独写在结构体外面；Python 的 `class` 把数据（属性）和函数（方法）放在一起。C 里的 `typedef struct {...} Student;` 在 Python 中**不需要**，定义类之后类名直接可用：

```c
/* C 的写法 */
typedef struct {
    char name[20];
    int age;
} Student;

void show(Student *s) { printf("%s %d\n", s->name, s->age); }
```

```python
# Python 的写法
class Student:                      # 类名习惯用大驼峰命名
    """学生类"""

    def __init__(self, name, age):  # 构造方法，创建对象时自动调用
        self.name = name            # self 相当于 C 里的结构体指针 s
        self.age = age              # 直接赋值即创建属性，不用提前声明字段

    def show(self):                 # 方法的第一个参数固定是 self
        print(f'{self.name} {self.age}')

s1 = Student('骆昊', 40)            # 创建对象，不需要像 C 那样 malloc
s1.show()                           # 输出：骆昊 40，用 . 访问成员（不是 ->）
print(s1.name)                      # 结果：骆昊
```

几个关键差异：`__init__` 相当于 C 里手写的初始化函数，创建对象时自动执行；每个方法的第一个参数 `self` 代表当前对象自己，相当于 C 里 `void show(Student *s)` 的那个 `s`，只是 Python 把它放到参数列表里显式写出来（调用时不用传）；成员访问用 `.` 而不是 `->`，因为 Python 中根本没有指针。

### 8. 指针与内存：一切皆对象

Python **没有指针**，也不需要 `malloc` / `free`。每个对象都由解释器自动管理，通过**垃圾回收**机制回收不再使用的内存。虽然拿不到地址，但可以用 `id()` 查看对象的身份（可以理解为简化版的"地址"）：

```python
a = [1, 2, 3]
b = a                # b 和 a 指向同一个列表对象，没有发生复制
print(id(a) == id(b))  # 结果：True（同一个对象）
print(a is b)        # 结果：True（is 判断是不是同一个对象）
print(a == b)        # 结果：True（== 判断值是否相等）
```

`is` 和 `==` 的区别是 C 程序员特别要留意的：`==` 相当于比较"指针指向的内容"（按值比较），`is` 相当于比较"指针本身"（是否同一对象）：

```python
x = [1, 2, 3]
y = [1, 2, 3]        # 值相同但两个独立的列表
print(x == y)        # 结果：True（值相等）
print(x is y)        # 结果：False（不是同一个对象）
```

C 的 `NULL` 对应 Python 的 `None`，判断时推荐用 `is`：

```python
result = None                 # None 表示"没有值"，类似 NULL
if result is None:            # 惯用写法：is None
    print('还没有结果')        # 输出：还没有结果
```

内存管理对照表：

| C 语言 | Python | 说明 |
| --- | --- | --- |
| `malloc` / `calloc` | 创建对象即可 | 解释器自动分配 |
| `free` / `realloc` | 不需要 | 垃圾回收自动释放 |
| `sizeof(x)` | `sys.getsizeof(x)` | 查看对象占用内存 |
| `&x`（取地址） | `id(x)` | 获取对象身份标识 |
| `NULL` | `None` | 判断用 `is None` |

### 9. 动态数据结构：容器开箱即用

C 里实现栈、队列、哈希表、链表都要手写结构体和指针操作，Python 的内置容器把这些全都包办了。

**列表当栈用**（后进先出）：

```python
stack = []
stack.append(1)      # 入栈，相当于 push
stack.append(2)
stack.append(3)
print(stack.pop())   # 结果：3（pop 弹出栈顶，后进先出）
print(stack)         # 结果：[1, 2]
```

**队列**用 `collections.deque`（列表开头插入 `insert(0, x)` 效率低，deque 两端操作都是高效 O(1)）：

```python
from collections import deque   # 引入双端队列

queue = deque()
queue.append('a')    # 队尾入队
queue.append('b')
print(queue.popleft())  # 结果：a（队头出队，先进先出）
```

**字典**就是现成的哈希表，相当于自己手写的 `hash table`：

```python
scores = {'骆昊': 100, 'Python': 99}  # 键值对创建
scores['C语言'] = 98                   # 插入键值对
print(scores['骆昊'])                  # 结果：100（按键取值）
print('C语言' in scores)               # 结果：True（O(1) 查找是否存在）
```

**集合**相当于自动去重的哈希集合，C 里要去重要写排序加遍历：

```python
nums = [1, 2, 2, 3, 3, 3]
unique = set(nums)          # 集合自动去重
print(unique)               # 结果：{1, 2, 3}
print(unique & {2, 3, 4})   # 结果：{2, 3}（交集运算）
```

**链表**在 Python 中通常不需要手写：随机访问用 `list`、头部增删用 `deque`、键值映射用 `dict`，绝大多数场景这些容器就够用了。确实需要链式结构时，用类把节点串起来即可，但一般轮不到自己写。

### 10. 文件操作：从 fopen 到 with open

C 用 `fopen` / `fclose` 和一堆 `fscanf` / `fgets` 函数，Python 把这些统一收敛到一个 `open` 函数上，配合 `with` 语句自动关闭文件（不用担心忘记 `fclose`）：

```python
# 写文件
with open('demo.txt', 'w', encoding='utf-8') as f:  # 打开文件用于写，with 结束自动关闭
    f.write('第一行\n')          # 写入一行
    f.writelines(['第二行\n', '第三行\n'])  # 把字符串列表批量写入

# 读文件
with open('demo.txt', 'r', encoding='utf-8') as f:
    print(f.read())              # read 一次性读出全部内容
# 输出：
# 第一行
# 第二行
# 第三行

with open('demo.txt', 'r', encoding='utf-8') as f:
    print(f.readline(), end='')  # readline 一次读一行（含行尾换行符）
# 输出：第一行

with open('demo.txt', 'r', encoding='utf-8') as f:
    lines = f.readlines()        # readlines 把所有行放进一个列表
print(len(lines))                # 结果：3
```

模式对照表（字母含义和 C 的 fopen 基本一致）：

| 模式 | C 含义 | Python 含义 | 说明 |
| --- | --- | --- | --- |
| `'r'` | `fopen(..., "r")` | 读文本 | 文件不存在会报错 |
| `'w'` | `fopen(..., "w")` | 写文本 | 覆盖已有内容，文件不存在则创建 |
| `'a'` | `fopen(..., "a")` | 追加文本 | 在文件末尾追加 |
| `'r+'` | `fopen(..., "r+")` | 读写 | 既可读又可写 |
| `'rb'` / `'wb'` / `'ab'` | `fopen(..., "rb")` 等 | 读/写/追加**二进制** | 加 `b` 处理字节串而非字符串 |

> **提示**：处理文本文件时建议显式指定 `encoding='utf-8'`，避免 Windows 下默认编码导致的乱码问题。

### 11. 标准库对照：头文件到模块

C 程序员熟悉的头文件，在 Python 里几乎都有对应的模块：

| C 头文件 | Python 模块 / 方法 | 典型对照 |
| --- | --- | --- |
| `math.h` | `math` 模块 | `sqrt(x)` → `math.sqrt(x)`，`pow(x, y)` → `x ** y` |
| `string.h` | `str` 的内置方法 | `strlen` → `len`，`strstr` → `find`，见第 6 节 |
| `stdio.h` | 内置函数 `print` / `input` | `printf` → `print`，`scanf` → `input` |
| `time.h` | `time` / `datetime` 模块 | `time(NULL)` → `time.time()`，格式化时间用 `strftime` |
| `stdlib.h`（rand 部分） | `random` 模块 | `rand()` → `random.randint(a, b)` |
| `assert.h` | 内置 `assert` 语句 | 用法几乎一致 |

```python
import math                       # 对应 math.h
import time                       # 对应 time.h
from datetime import datetime     # 更好用的时间类型

print(math.sqrt(16))              # 结果：4.0
print(math.floor(3.7))            # 结果：3（向下取整，对应 floor 函数）
print(int(time.time()))           # 输出：当前 Unix 时间戳（秒），对应 time(NULL)
print(datetime.now().strftime('%Y-%m-%d %H:%M:%S'))  # 输出：格式化的当前时间
```

`assert` 用法与 C 一致，断言失败抛出 `AssertionError`：

```python
x = 10
assert x > 0, 'x 必须是正数'      # 条件成立则无事发生
assert isinstance(x, int), 'x 应该是整数'  # 常用于调试期检查
print('断言通过')                  # 输出：断言通过
```

### 12. 输入输出：scanf 与 input

C 的 `scanf` 需要写格式符、取地址，Python 的 `input()` 一次读一行并**永远返回字符串**，需要数字时记得显式转换（见第 2 节）：

```c
// C 的写法
int age;
scanf("%d", &age);
```

```python
# Python 的写法
age = int(input('请输入年龄：'))  # input 返回 str，必须 int() 转换
print(age + 1)                    # 转换后才能做加法
```

读多行或读到文件结束（EOF）时，用 `sys.stdin`，相当于 C 的 `while (scanf(...) != EOF)`：

```c
// C 的写法：读到 EOF 为止
int x;
while (scanf("%d", &x) != EOF) { printf("%d\n", x * 2); }
```

```python
# Python 的写法一：逐行读，直到输入结束
import sys                        # 引入 sys 模块获得标准输入流

for line in sys.stdin:            # 从标准输入逐行读取，Ctrl+D / Ctrl+Z 表示 EOF
    print(int(line) * 2)          # 每行一个数，输出它的两倍

# Python 的写法二：一次性读入所有内容
# data = sys.stdin.read()         # 读到 EOF 的全部文本
```

`printf` 格式符与 f-string 的对照表（f-string 是目前最推荐的写法）：

| C 格式符 | Python f-string 写法 | 示例（值） | 输出 |
| --- | --- | --- | --- |
| `%d` | `{x}` | `42` | `42` |
| `%5d` | `{x:5d}` | `42` | `   42`（宽度 5 右对齐） |
| `%05d` | `{x:05d}` | `42` | `00042` |
| `%f` | `{x:f}` | `3.14159` | `3.141590` |
| `%.2f` | `{x:.2f}` | `3.14159` | `3.14` |
| `%e` | `{x:e}` | `12345.6` | `1.234560e+04` |
| `%s` | `{s}` | `'hi'` | `hi` |
| `%10s` | `{s:>10}` | `'hi'` | `        hi`（宽度 10 右对齐） |
| `%-10s` | `{s:<10}` | `'hi'` | `hi        `（左对齐） |
| `%x` | `{x:x}` | `255` | `ff` |
| `%o` | `{x:o}` | `8` | `10` |
| `%%` | 直接写 `%` | — | `%`（f-string 中不需要转义） |

```python
x, s = 42, 'hi'
print(f'[{x:05d}]')      # 结果：[00042]
print(f'[{x:.2f}]')      # 结果：[42.00]，int 也能按浮点格式输出
print(f'[{s:>10}]')      # 结果：[        hi]
```

转义字符两者相同：`\n` 换行、`\t` 制表符、`\\` 反斜杠、`\'` `\"` 引号。另外 Python 还有一个 C 没有的便利写法——在字符串前加 `r` 表示原始字符串，反斜杠不做转义，写 Windows 路径和正则表达式特别省事：

```python
print('a\tb')            # 结果：a	b（\t 生效，制表符）
print(r'a\tb')           # 结果：a\tb（原始字符串，\t 原样输出）
print(r'C:\new\test')    # 结果：C:\new\test（不必写成 C:\\new\\test）
```

### 13. 常见坑速查

从 C 过来写 Python，下面这些坑几乎人人都会踩一遍，先列出来避雷：

1. **IndentationError**：缩进是语法，混用 Tab 和空格、复制粘贴代码缩进错位都会直接报错。统一用 4 个空格。
2. **整数除法 `/` 返回 float**：`7 / 2` 是 `3.5` 不是 `3`，想要整数结果用 `7 // 2`。
3. **字符串不可变**：`s[0] = 'H'` 会报 `TypeError`，只能创建新字符串。
4. **列表赋值是引用**：`b = a` 不复制列表，`b` 和 `a` 是同一个对象，改 `b` 等于改 `a`。真正的复制用 `a[:]` 或 `a.copy()`（浅拷贝，嵌套列表的内层仍是引用）。
5. **默认参数可变对象陷阱**：默认参数只在函数定义时求值一次，用列表做默认参数会"记住"上次的修改。
6. **`range(5)` 不含 5**：产生 0 到 4，边界判断和 C 的 `i < 5` 一样是"左闭右开"。
7. **UTF-8 编码**：读写文件不指定 `encoding='utf-8'`，在 Windows 上可能乱码。
8. **解释执行无编译链接**：写错类型、拼错变量名只有**运行到那一行**才报错，不像 C 编译期就能发现，要多写多测。
9. **异常机制替代错误码**：C 用返回 `-1` / `NULL` 加 `errno` 表示出错，Python 用 `try` / `except` 捕获异常，错误码检查的习惯要换掉。

第 4、5、9 条的演示代码：

```python
# 坑 4：列表赋值是引用
a = [1, 2, 3]
b = a                    # b 和 a 指向同一个列表
b.append(4)
print(a)                 # 结果：[1, 2, 3, 4]（a 也"被改了"）
c = a[:]                 # 浅拷贝，创建新列表
c.append(5)
print(a)                 # 结果：[1, 2, 3, 4]（a 不受影响）

# 坑 5：默认参数可变对象陷阱
def add_item(item, basket=[]):   # 危险！默认列表在多次调用间共享
    basket.append(item)
    return basket

print(add_item(1))       # 结果：[1]
print(add_item(2))       # 结果：[1, 2]（上次的 1 还在！）

# 正确写法：默认值用 None，函数内再创建新列表
def add_item2(item, basket=None):
    if basket is None:   # 每次调用时新建列表
        basket = []
    basket.append(item)
    return basket

print(add_item2(1))      # 结果：[1]
print(add_item2(2))      # 结果：[2]
```

```python
# 坑 9：异常机制替代错误码
# C 的习惯：检查 atoi 返回值是否合法
# Python 的做法：try / except
s = 'abc'
try:
    x = int(s)                 # 尝试转换
    print(x)
except ValueError:             # 转换失败时走这里，程序不崩溃
    print('不是合法的整数')     # 输出：不是合法的整数
```

###  总结

C 语言训练出来的逻辑思维在 Python 里全部有用，需要重建的主要是两件事：一是**书写习惯**，用缩进代替大括号、用 for-in 代替三段式 for、用 f-string 代替 printf；二是**内存观念**，一切皆对象、赋值即引用、内存交给垃圾回收，从此不再和指针、malloc 打交道。把这份指南里的对照关系过一遍，再动手把以前用 C 写过的小程序（排序、文件读写、简单数据结构）用 Python 重写一遍，转换就基本完成了。
