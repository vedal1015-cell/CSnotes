# 识舟的学习笔记——2015 CMU 15-213 CSAPP 深入理解计算机系统02——Floating Point

## 目录/框架

* Fractional Binary
* IEEE Floating Point
* Floating Point Representation
  * Form And Encoding
  * “Normalized” Values, Denormalized Values And Special Values
* Floating Point Operations

## 课程内容

### Fractional Binary Numbers

和十进制类似，十进制对于小数点后的数，分别乘$10^{-1}$，$10^{-2}$，$10^{-3}$...二进制在小数点之后的数分别乘$2^{-1}$ ，$2^{-2}$，$2^{-3}$...我们能够看出，这种二进制的小数只能精确表示形如$\frac{x}{2^k}$的数，其他的数就要用循环小数来表示了。

对于一个小数，如果小数点比较靠近低位，那么表示的小数的精确度可能不够大，如果小数点比较靠近高位，那么表示的数字的范围就小了，因此我们选择浮点作为折中的方式，对于不同的数，通过移动二进制小数点来兼顾大的取值范围和高的精度。

### IEEE Floating Point

早期的浮点数表示各厂商很不统一，不过电气电子工程师学会在1985年提出的IEEE754等几个标准提出之后有了较为统一的浮点数表示。

### Floating Point Representation

#### Numerical Form

浮点数用一种类似科学计数法类似的方式来表示数字，格式是*$(-1)^S\ M\ 2^E$​*的形式,其中S相当于符号位，M是尾数，乘上2的E次幂。

#### Encoding

一个浮点数由三个部分组成：符号位（S），指数字段（exp），尾数字段（frac）。在不同位数的编码中，各部分位数如下：

| S    | exp     | frac          |
| ---- | ------- | ------------- |
| 1    | 8-bits  | 23-bits       |
| 1    | 11-bits | 52-bits       |
| 1    | 15-bits | 63 or 64-bits |

#### “Normalized” Values

$exp \ \neq 000...0\ and \ exp \ \neq \ 111...1$

* $E = exp - bias$，其中exp是exp字段的值
* $bias = 2^{k\ -\ 1}\ -\ 1$​，其中k是exp字段的位数
* $M\ =\ 1.xxx...x$，其中$xxx...x$​就是frac字段

#### Denormalized values

$exp\ =000...0$

* $E\ =\ 1\ -\ bias$，其中exp是exp字段的值（注意不是$0-bias$​），bias同上
* $M\ =\ 0.xxx...x$，其中$xxx...x$​​就是frac字段

#### Special Values

$exp\ =\ 111...1$

* 如果$frac\ =\ 000...0$，那么规定值为$\infin$
* 如果$frac\ \neq\ 000...0$ ，那么规定值为NaN，简单理解就是乱码，一般在计算$\sqrt{-1}$，$\infin\ -\ \infin$，$\infin\ ×\ 0$​之类计算机不确定答案是什么的情况下会出现

#### Visualization: Floating Point Encodings

![可视化浮点数编码](https://cdn.jsdelivr.net/gh/vedal1015-cell/blog_resourses_pub@main/note_for_study/CSAPP/Bits,_Bytes,_and_Integers/CSAPP_Class2_Picture1.png)

### Floating Point Operations: Basic Idea

实际运算时可能会有溢出或者精度不够的情况，因此我们计算时需要将结果舍入到我们需要的精度。舍入的方式有许多，以下是一些示例：

|                       | \$1.40 | \$1.60 | \$1.50 | \$2.50 | -\$1.50 |
| --------------------- | ----- | ----- | ----- | ----- | ------ |
| Towards zero          | \$1    | \$1    | \$1    | \$2    | -\$1    |
| Round down($-\infin$) | \$1    | \$1    | \$1    | \$2    | -\$2    |
| Round up($+\infin$)   | \$2    | \$2    | \$2    | \$3    | -\$1    |
| Nearest Even(default) | \$1    | \$2    | \$2    | \$2    | -\$2    |

其中需要注意的是向偶数舍入（Nearest Even），一般情况下四舍六入，只有舍去位最高为5时向偶数舍，确保舍去后最后一位为偶数（四舍六入偶成双）

#### Closer Look at Round-To-Even

从统计学意义来讲，向偶数写入确保了向上和向下取整的概率都是50%

#### Floating Point Multiplication

对于乘法计算：$(-1)^{s1}\ M1\ 2^{E1}\ ×\ (-1)^{s2}\ M2\ 2^{E2}$，结果为$(-s)^s\ M\ 2^E$，其中：

$s=s1 \textasciicircum s2$​

$M=M1×M2$

$E=E1+E2$

然后，如果M发生了溢出，那么对M进行相应的移位操作，并且因此对于指数E进行处理。

#### Floating Point Addition

对于加法计算：$(-1)^{s1}\ M1\ 2^{E1}\ +\ (-1)^{s2}\ M2\ 2^{E2}$，假设$E1\ >\ E2$

对$(-1)^{s2}\ M2\ 2^{E2}$进行移位处理，相当于让E1和E2保持一致，提出公因数$2^{E1}$，然后进行正常加减

#### Mathematical Properties of FP Operation

应当注意，整型不完全等于整数，浮点型也不完全等于实数，不过二者有类似的数学特性，但不能将二者完全划等号，比如浮点数符合交换律但是不一定满足结合律。

例：$3.14 + 10^{10} - 10^{10} = 0,\ \ \ 3.14 + (10^{10} - 10^{10}) = 3.14$
