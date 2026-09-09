# 识舟的学习笔记——2015 CMU 15-213 CSAPP 深入理解计算机系统01——Bits, Bytes, and Integers

一个计算机新生的学习笔记，简单记录一下学习进度...

## 目录/框架

* Representing information as bits
* Bit-level manipulations
* Integers
* * Representation: unsigned and signed
  * Conversion, casting
  * Expanding, truncating
  * Addition, negation, multiplication, shifting
* Representations in memory, pointers, strings
* Summary

## 课程内容

### Representing information as bits

#### Everything is bits

简单讲了一下二进制的优点，主要是因为二进制容易量化模拟信号表示的信息。

### Bit-level manipulations

#### Encoding Byte Values

1 Byte = 8 bits 一个简单的换算，1 字节（Byte）= 8 位（bit）

二进制是$ 00000000_2 到 11111111_2 $，十进制是$ 0_{10}到255_{10} $，十六进制是$00_{16}到FF_{16}$，这三者表示的范围是一样的。

#### Boolean Algebra

布尔代数，其运算我最初接触时是用门电路来理解的。

或门（|）：二者有一个1，结果就为1

与门（&）：二者都为1，结果才为1

非门（~）：1变0，0变1

异或门（^）：二者不一样结果为1，二者相同结果为0

用PPT中的表格来表示：

And:

| &    | 0    | 1    |
| :--- | ---- | ---- |
| 0    | 0    | 0    |
| 1    | 0    | 1    |

Or:
| \|   | 0    | 1    |
| ---- | ---- | ---- |
| 0    | 0    | 1    |
| 1    | 1    | 1    |

Not:

| ~    |      |
| ---- | ---- |
| 0    | 1    |
| 1    | 0    |

Xor:

| ^    | 0    | 1    |
| ---- | :--- | ---- |
| 0    | 0    | 1    |
| 1    | 1    | 0    |

PS:类似的还有与非门（NAND）和或非门（NOR），这两个是通用门，可以表示其他的所有门。

#### Operate on Bit Vectors

逐位对每一位进行运算，比如 ~(11010111) = 00101000

课程中有一个很好的比喻：
And(&)操作就像集合的交，Or(|)操作就像集合的并，异或操作就是所谓的对称的差异(symmetric difference)。

#### Contrast: Logical operaters

C语言中的&&，||，！用于逻辑运算，要避免与&，|，~等位运算混淆。

#### Shift Operations

介绍了算术移位和逻辑移位两种移位操作。对于同一个数字，左移的差别不大。右移的区别在于，算术移位会填充符号位，而逻辑移位只会无脑补零。实际区别详见下表：

| Argument x  | 01100010 |
| ----------- | -------- |
| << 3        | 00010000 |
| Log. >> 2   | 00011000 |
| Arith. >> 2 | 00011000 |

| Argument x  | 10100010 |
| ----------- | -------- |
| << 3        | 00010000 |
| Log. >> 2   | 00101000 |
| Arith. >> 2 | 11101000 |

同时，x << 8 这个运算有时候是未定义操作，在不同的平台或者编译器可能有不同的结果，因此要小心陷阱，避免未定义操作。例如课程中提到有些平台计算x << 8 = x << (8 mod 8) = x << 0 = x

### Integers

#### Numeric Ranges

##### 无符号数(Unsigned Values)

10110表示$ 16 × 1 + 8 × 0 + 4 × 1 + 2 × 1 + 1 × 0 = 22$​

* UMin = $0$
  000...0
* UMax = $2^w - 1$​
  111...1

##### 有符号数补码(Two’s Complement Values)

00110表示$ (-16) × 0 + 8 × 0 + 4 × 1 + 2 × 1 + 1 × 0 = 6$

10110表示$ (-16) × 1 + 8 × 0 + 4 × 1 + 2 × 1 + 1 × 0 = -10$​

*PS*:将首位加负号用于计算补码，相较于原码取反+1的计算方法，或许更便于理解，但本质都是用$ 原码数 - 2^w $来表示负数的做法。

* TMin = $-2^{w-1}$
  100...0
* TMax = $2^{w-1} - 1$
  011...1

#### Conversion Visualized

课程视频中的这张图片可以很好地体现补码的表示情况：

![视频链接：https://www.bilibili.com/video/BV1iW411d7hd/?spm_id_from=333.1391.0.0&p=2&vd_source=e7cfe7d51133ac7f7572686a6e387c16](https://cdn.jsdelivr.net/gh/vedal1015-cell/blog_resourses_pub@main/note_for_study/CSAPP/Bits,_Bytes,_and_Integers/CSAPP_Class1_Picture1.png)

##### Signed vs. Unsigned in C

C语言是少数Unsigned是一个明确的数据类型的语言之一，在Unsigned数据和Signed数据之间操作会发生类型转换。当我们比较两个数据类型不同的数时，C会对二者进行隐式的类型转换。比如，有符号和无符号的数在比较时，有符号数会被转化为无符号数进行运算。

课程中用一张表格来反映各种比较的情况：

| Constant₁    | Constant₂         | Relation | Evaluation |
| ------------ | ----------------- | -------- | ---------- |
| 0            | 0U                | ==       | unsigned   |
| -1           | 0                 | <        | signed     |
| -1           | 0U                | >        | unsigned   |
| 2147483647   | -2147483647-1     | >        | signed     |
| 2147483647U  | -2147483647-1     | <        | unsigned   |
| -1           | -2                | >        | signed     |
| (unsigned)-1 | -2                | >        | unsigned   |
| 2147483647   | 2147483648U       | <        | unsigned   |
| 2147483647   | (int) 2147483648U | >        | signed     |

因此我们可以总结出类型转换的规则：不改变数的比特表示，只更换数的解释方式。

#### Sign Extension

课程中讨论的符号扩展是指补码数，考虑的是在不改变数值的情况下增加数字的符号位。

无符号数不考虑符号位，只需要在最前面补0即可。对于有符号数，则需要填充与符号位相同的数，原因很简单，我们不妨看看正数和负数分别是什么情况。

正数和无符号数类似都要补0，由于符号位也是0，因此只需要在前面复制符号位0。

负数从取反加1的角度则更容易理解。一个负数的原码一般是$1XX...X$，取反加1之后是$1(XX...X)_{取反加1}$，若是符号扩展，原码变为$1 00...0XX...X$，取反加1之后是$1 111...1(XXX...X)_{取反加1}$​​，相当于复制符号位1（注意补码符号位不变）

教材中的说法大概是这样的：扩展 1 位后，新符号位在权 −2^w 的位置贡献 −2^w·s，原符号位退居 +2^{w−1}·s，合计仍为 −2^{w−1}·s，与原值一致。

类似的，还有截断（Truncating）操作，截断的结果有可能会改变原有的数值，其最终值是原数进行类似取模操作后的结果。

举个栗子：100101 = -27

截断两位：**~~10~~**0101 = 5

其中，5是(-27 mod 8)的结果，因为四位数0101不考虑符号位最大为7,所以要对8取余。

PS：对于负数的取模运算，可以这样简单理解：对于负数-x对p取模，求一个k$(0\ \leq\ k\ < p)$，使得(-x+k)可以整除p。

#### Addition, negation, multiplication, shifting

##### Unsigned Addition

关于加法，当我们使用两个w位的数字相加的时候，其结果可能最高有(w+1)位。

![视频链接：https://www.bilibili.com/video/BV1iW411d7hd/?spm_id_from=333.1391.0.0&p=3&vd_source=e7cfe7d51133ac7f7572686a6e387c16](https://cdn.jsdelivr.net/gh/vedal1015-cell/blog_resourses_pub@main/note_for_study/CSAPP/Bits,_Bytes,_and_Integers/CSAPP_Class1_Picture2.png)

因此，计算加法时我们会丢弃首位，最后的结果为$ ((u + v) mod \ 2^w)$，即取模后的和。这种在做加法时超出范围导致取模的情况就是溢出。

##### 2’s Complement Addition

对于有符号数的补码加法，两个绝对值较大的同号数相加时可能会超出表示范围，最终得到异号结果，这样就会造成上溢和下溢，形成类似取模的最终结果，实际情况如下图。（可以想象图中上溢和下溢的部分向上和向下平移会变成什么样子）

![视频链接：https://www.bilibili.com/video/BV1iW411d7hd/?spm_id_from=333.1391.0.0&p=3&vd_source=e7cfe7d51133ac7f7572686a6e387c16](https://cdn.jsdelivr.net/gh/vedal1015-cell/blog_resourses_pub@main/note_for_study/CSAPP/Bits,_Bytes,_and_Integers/CSAPP_Class1_Picture3.png)

##### Signed Multiplication in C

乘法与加法类似，同样会有溢出的情况，且最多可能会有2 * w位，同样会给结果造成影响。

![视频链接：https://www.bilibili.com/video/BV1iW411d7hd/?spm_id_from=333.1391.0.0&p=3&vd_source=e7cfe7d51133ac7f7572686a6e387c16](https://cdn.jsdelivr.net/gh/vedal1015-cell/blog_resourses_pub@main/note_for_study/CSAPP/Bits,_Bytes,_and_Integers/CSAPP_Class1_Picture4.png)

##### Power-of-2 Multiply with Shift

移位操作则较为简单，$u << k$​等价于$u * 2^k$​​，一个w位数最多有w + k位。

### Representations in memory, pointers, strings

#### Byte-Oriented Memory Organization

首先，由于$10^3$和$2^{10}$大约为一个数量级，$ 128TB = 2^{10}\ *\ 10^{12}bit\approx 10^{15}bit$，因此我们就能感受到128TB是一个很大的空间（快到千万亿bit了）。

一般来说我们很难获得这么多内存，但是当我们运行程序时，程序会认为我们有这么大的范围，这个范围其实不是真的。然而，事实上操作系统只能够使用内存中的某些区域，如果访问其他位置就会报错，这叫做分段错误(segmentation fault)，具体的操作涉及到虚拟内存有关的知识。

程序是一大堆软硬件的复杂组合，但其在电脑中只是一大串字节数组的空间。它通过在内存之间，或者在内存和磁盘驱动器之间移动来实现程序的复杂功能。

#### Machine Words

关于*字长*：字长没有明确的定义，更多是一种惯例性说法，通常指整数与指针数据的"标称"位宽（nominal size of integer and pointer data，CS:APP §2.1.3），用来判断数能表示的范围、指针（虚拟地址）上限以及算术单元按多少位工作。通俗地说，字长是一个标准，用于说明最大值和指针范围的标准，以及用于判断如何存储值和算术运算的标准。比如64位计算机，一般是说他擅长处理64位值和相关的算术运算，并且指针也可以有这么大。实际情况中，往往是硬件和编译器一起决定字长的大小。

#### Byte Ordering

对于同样一个数的存储：

大端字节序(Big Endian)：地址从小到大是从高位到低位的字节序。

小端字节序(Little Endian)：地址从小到大是从低位到高位的字节序。

现在一般以小端字节序为主，大端字节序主要应用场景是在网络传输上。

看起来，大端字节序因为都是从左到右，可读性更好，那么为什么小端字节序成为了主流呢？简单来说，小端字节序是低位存储在低地址，高位存储在高地址，因此从程序的角度，小端序可能更清晰好写，举个例子：一个十进制数654321，我们要判断奇偶性，就要找最后一位，假如他的首地址是0x00800000，一般就是0x00800000存最后几位（比如21），那么只要判断0x00800000位置的数的奇偶性了。更多大端序和小端序的区别可以看[这篇文章](https://www.ruanyifeng.com/blog/2022/06/endianness-analysis.html)。
