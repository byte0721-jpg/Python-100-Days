## turtle绘图

**turtle（海龟）库**是 Python 的**标准库**，不需要额外安装，直接 `import turtle` 就可以使用。它通过一只"小海龟"在窗口中爬行来作图：海龟的坐标就是画笔的位置，海龟的朝向决定线条的方向，海龟爬过的轨迹就是画出来的线条。这种"跟着小海龟一步步画图"的方式非常直观，特别适合初学者理解程序与图形之间的关系。

### 基础函数

在使用 turtle 之前，先通过一个表格认识最常用的函数。

| 函数 | 作用 |
| --- | --- |
| `turtle.penup()` | 抬起画笔，之后移动海龟不会画线 |
| `turtle.pendown()` | 放下画笔，之后移动海龟会画线 |
| `turtle.forward(d)` | 沿当前方向前进 `d` 像素（别名 `turtle.fd(d)`） |
| `turtle.right(angle)` | 右转（顺时针）`angle` 度 |
| `turtle.left(angle)` | 左转（逆时针）`angle` 度 |
| `turtle.goto(x, y)` | 让海龟直接移动到坐标 `(x, y)` 处 |
| `turtle.pensize(width)` | 设置画笔的粗细 |
| `turtle.pencolor(color)` | 设置画笔的颜色，可用字符串如 `"red"` |
| `turtle.speed(v)` | 设置绘图速度，`v` 取 1~10 的整数，`0` 表示最快 |
| `turtle.hideturtle()` | 隐藏海龟图标，让画面更干净 |
| `turtle.done()` | 保持绘图窗口打开，等待用户关闭 |

> **提示**：默认情况下画笔是放下（pendown）状态，海龟移动就会留下痕迹。如果想让海龟"跳跃"到某个位置而不画线，就要先 `penup()`，到达后再 `pendown()`。

#### 画一个正方形

```python
import turtle

turtle.pensize(3)          # 设置画笔粗细为3像素
turtle.pencolor("red")     # 设置画笔颜色为红色
for _ in range(4):         # 循环4次，对应正方形的4条边
    turtle.forward(100)    # 前进100像素，画出一条边
    turtle.right(90)       # 右转90度，准备画下一条边
turtle.done()              # 保持窗口打开
```

#### 用 goto 画折线

```python
import turtle

turtle.penup()             # 先抬笔，避免从原点画出一条多余的线
turtle.goto(-100, 50)      # 海龟直接跳到坐标(-100, 50)处
turtle.pendown()           # 放下画笔，开始作图
turtle.goto(100, 50)       # 画线到(100, 50)
turtle.goto(0, -50)        # 再画线到(0, -50)，形成一个三角形
turtle.done()              # 保持窗口打开
```

> **说明**：`speed()` 的取值范围是 1（最慢）到 10（较快），传入 `0` 不代表不动，反而是最快的速度。调试代码时可以把速度调慢，方便观察海龟的运动过程。

### 实例：七段数码管绘制

数码管是一种常见的显示器件，比如老式的电子时钟就用它显示数字。一个数字由 7 段线条组成，习惯上给每段编号为 1 到 7：

```
    7
   ---
 6|   |1
   -5-
 4|   |2
   ---
    3
```

#### drawDigit 的核心思路

`drawDigit(digit)` 函数的思路是：根据要画的数字 `digit`，决定 7 段中哪些段要画、哪些段不画。海龟从数字**中间线段的左端**出发，面向右侧，每画完一段就右转 90 度调整方向；画完左下段后，先**左转 90 度**改为向上行进，再画剩下的三段。行进顺序与角度安排如下：

1. 中间段（5 号）：面向右侧，水平向右画线；
2. 右下段（2 号）：右转 90 度，竖直向下画线；
3. 底段（3 号）：右转 90 度，水平向左画线；
4. 左下段（4 号）：右转 90 度，竖直向上画线；
5. 左转 90 度，面向上方，画左上段（6 号）；
6. 右转 90 度，面向右侧，画顶段（7 号）；
7. 右转 90 度，面向下方，画右上段（1 号）。

画完右上段后海龟面向左侧，只要**右转 180 度**再前进 20 像素，就恰好站到下一个数字的起始位置，方向也回到了面向右侧。

哪些段要画？最容易想到的办法是给每个数字列出它的线段编号，例如数字 2 要画 1、3、4、5、7 这五段（跳过右下第 2 段和左上第 6 段）。下面直接给出完整可运行的代码。

```python
import turtle
import time

# 0~9每个数字对应的线段编号，例如数字2要画1、3、4、5、7段
DIGIT_SEGS = {
    0: (1, 2, 3, 4, 6, 7),
    1: (1, 2),
    2: (1, 3, 4, 5, 7),
    3: (1, 2, 3, 5, 7),
    4: (1, 2, 5, 6),
    5: (3, 4, 5, 6, 7),
    6: (2, 3, 4, 5, 6, 7),
    7: (1, 2, 7),
    8: (1, 2, 3, 4, 5, 6, 7),
    9: (1, 2, 3, 5, 6, 7),
}

def drawGap():                       # 绘制数码管各段之间的间隔
    turtle.penup()                   # 抬笔
    turtle.forward(5)                # 空走5像素，形成间隔

def drawLine(draw):                  # 绘制单段数码管
    drawGap()                        # 先空走一小段
    turtle.pendown() if draw else turtle.penup()  # 该段要画就落笔，否则保持抬笔
    turtle.forward(40)               # 画一段40像素的线
    drawGap()                        # 再空走一小段
    turtle.right(90)                 # 右转90度，为下一段调整方向

def drawDigit(digit):                # 根据数字绘制对应的七段数码管
    for seg in (5, 2, 3, 4):         # 按行进顺序处理中、右下、底、左下四段
        drawLine(seg in DIGIT_SEGS[digit])  # 该段在编号表中就落笔画线，否则空走转向
    turtle.left(90)                  # 左转90度，改为向上行进
    for seg in (6, 7, 1):            # 再处理左上、顶、右上三段
        drawLine(seg in DIGIT_SEGS[digit])
    turtle.penup()                   # 抬笔
    turtle.right(180)                # 右转180度，重新面向右侧
    turtle.forward(20)               # 空走20像素，站到下一个数字的起始位置

def drawDate(date):                  # 绘制形如20261107的日期
    for i in date:                   # 逐个字符处理日期字符串
        drawDigit(int(i))            # 把字符转成整数后绘制成数码管样式

def main():
    turtle.setup(800, 300)           # 设置窗口大小为800x300像素
    turtle.penup()                   # 抬笔
    turtle.goto(-300, 0)             # 移动到左侧起始位置
    turtle.pendown()                 # 落笔开始绘图
    turtle.pensize(5)                # 画笔粗细为5像素
    turtle.speed(5)                  # 设置绘图速度
    date = time.localtime()          # 获取当前日期时间
    drawDate(f"{date.tm_year}{date.tm_mon:02d}{date.tm_mday:02d}")  # 绘制年月日
    turtle.hideturtle()              # 绘制完成后隐藏海龟
    turtle.done()                    # 保持窗口打开

main()                               # 调用主函数开始绘图
```

> **提示**：`time.localtime()` 返回一个 `struct_time` 对象，其中 `tm_year`、`tm_mon`、`tm_mday` 分别对应年、月、日。由于月和日可能是 1 位数，我们用 `{:02d}` 格式化成两位，比如 11 月 7 日会变成 `1107`。如果希望"月"用橙色、"日"用红色画出来，只需在 `drawDate` 中根据序号调用 `turtle.pencolor()` 即可。

### 实例：科赫曲线绘制

**科赫曲线**是一种经典的分形图形。它的思想非常简单：一条线段，分成三等份，中间的三分之一替换成两段呈 60 度角的线段（形成一个"凸起"），然后对每一段新线段重复同样的操作，无限次之后就成了科赫曲线。

#### 递归思想

用递归来描述这个过程非常自然：

- **递归出口**：当阶数 `n` 为 0 时，直接画一条直线；
- **递归公式**：当阶数 `n` 大于 0 时，画科赫曲线 = 画一条低一阶的科赫曲线 → 左转 60 度 → 再画一条低一阶的 → 右转 120 度 → 再画一条低一阶的 → 左转 60 度 → 最后画一条低一阶的。

#### 核心代码

```python
import turtle

def koch(n, size):                   # n为阶数，size为初始线段长度
    if n == 0:                       # 递归出口：0阶就是一条直线
        turtle.forward(size)         # 直接前进size像素
        return
    for angle in [60, -120, 60, 0]:  # 四段曲线之间的转向角度
        koch(n - 1, size / 3)        # 递归绘制低一阶、长度为三分之一的曲线
        turtle.left(angle)           # 每段画完后按角度转向

def main():
    turtle.setup(800, 400)           # 设置窗口大小
    turtle.penup()                   # 抬笔
    turtle.goto(-300, 100)           # 移动到起始位置
    turtle.pendown()                 # 落笔开始绘图
    turtle.pensize(2)                # 画笔粗细为2像素
    turtle.speed(8)                  # 加快绘图速度
    koch(3, 600)                     # 绘制3阶、初始长度600像素的科赫曲线
    turtle.hideturtle()              # 隐藏海龟
    turtle.done()                    # 保持窗口打开

main()                               # 调用主函数开始绘图
```

> **说明**：`for angle in [60, -120, 60, 0]` 配合 `turtle.left(angle)`，等价于"画一段后左转 60 度、画一段后右转 120 度、画一段后左转 60 度、最后一段画完不再转向"，四个转向角的和恰好为 0，所以画完曲线后海龟的朝向不变，方便连续绘制多条曲线。

#### 科赫雪花

把科赫曲线旋转三次，首尾相接，就得到了**科赫雪花**。做法是：循环调用三次 `koch()` 函数，每画完一条边就右转 120 度，让三条边围成一个三角形轮廓。

```python
import turtle

def koch(n, size):                   # 与上面相同的科赫曲线递归函数
    if n == 0:                       # 递归出口
        turtle.forward(size)         # 画一条直线
        return
    for angle in [0, 60, -120, 60]:  # 四段曲线的转向
        koch(n - 1, size / 3)        # 递归画低一阶曲线
        turtle.right(angle)          # 转向

def main():
    turtle.setup(800, 800)           # 设置窗口大小为800x800像素
    turtle.penup()                   # 抬笔
    turtle.goto(-250, 150)           # 移动到雪花顶点的起始位置
    turtle.pendown()                 # 落笔
    turtle.pensize(2)                # 画笔粗细
    turtle.pencolor("blue")          # 画笔颜色为蓝色
    turtle.speed(10)                 # 3阶雪花线条很多，把速度调快
    for _ in range(3):               # 雪花的三条边
        koch(3, 500)                 # 画一条3阶科赫曲线
        turtle.right(120)            # 右转120度，准备画下一条边
    turtle.hideturtle()              # 隐藏海龟
    turtle.done()                    # 保持窗口打开

main()                               # 调用主函数开始绘图
```

> **提示**：科赫曲线是典型的"指数爆炸"式递归，阶数每增加 1，线段数量就是上一阶的 4 倍。3 阶雪花一共要画 $3 \times 4^3 = 192$ 条最底层的小线段，绘图速度建议用 `speed(10)` 或 `speed(0)`，阶数不建议超过 4，否则要等很久。

###  总结

turtle 是 Python 标准库中最直观的绘图工具，用"小海龟"爬行的方式作图，核心是抬笔落笔（`penup`/`pendown`）、前进转向（`forward`/`right`/`left`）这几组基础函数；七段数码管通过控制 7 段线段的绘制与转向，配合 `time.localtime()` 就能画出当前日期，而科赫曲线则展示了递归的威力——一个简单的规则不断重复，就能画出复杂精美的分形图形。
