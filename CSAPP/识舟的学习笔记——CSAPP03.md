# 识舟的学习笔记——2015 CMU 15-213 CSAPP 深入理解计算机系统03——Maching-Level Programming

## 目录/框架

## 课程内容

### Basics

#### History of Intel processors and architextures

目前主流有三大CPU架构：x86，ARM，RISC-V，我们先说x86。

对于英特尔而言，x86一开始是一个口头上的称呼，原因是历史上第一个芯片是8086，后来跳过81，出现了8286，8386等，这些都有86，所以人们把这称作x86。

x86早期也被叫做CISC，主要是受到80年代早期，CISC和RISC大战的影响。当时风靡一时的是RISC（Reduced Instruction Set Computer），也就是精简指令集计算机。其实之前没有人给x86取名为CSIC，但是许多用RISC的人把x86的处理器叫做CISC（Complex Instruction Set Computer），也就是复杂指令集计算机，这种说法其实是带有贬义的。

PS：目前处理器常用的另一大类是ARM（Acorn RISC Machine），其中Acorn是一家英国公司（已倒闭且被英伟达收购）。他们提出了一种高效且简单的指令集，可以放在芯片上，最重要的是可以定制。这家公司实质上并没有直接销售处理器，只是向公司出售他们的设计的使用权（也就是知识产权）。

#### C, assembly, maching code

关于指令集：编译器的目标是通过提供的指令，告诉机器要干什么，人们针对设计的大量指令设计了“指令集”架构的抽象。

对于程序员而言，编写汇编代码和写C代码有所不同。首先，像是C代码之类的编程语言，可以方便地通过各种方式检查，测试和操作。其次，在编程时用到的内存，实则是不同方式实现的虚构对象，通过虚拟内存实现看起来很大的内存空间。

#### Turning C into Object Code

在运行C代码时，首先会把*.c文件通过编译器（Compiler）生成汇编代码，通过汇编编译器（Assembler）生成字节形式的程序。

#### Compiling Into Assembly

以下这张图简单展示了编译的过程：

![一个高级语言转化为汇编语言的实例](https://cdn.jsdelivr.net/gh/vedal1015-cell/blog_resourses_pub@main/note_for_study/CSAPP/Machine_Level_Programing/CSAPP_Class3_Picture1.png)

左边是C代码，右边则是汇编代码。我们可以看到右边有很多%，这些%是寄存器是实际名称，而前面的内容则说明了要对寄存器执行的操作，比如pushq是把他推到栈上，movq是把他从一个地方复制到另外一个地方，call是调用一些方法，popq是出栈，ret是从特定函数返回。

我们可以输入一行代码来实现这种转换：

```
gcc -Og -S sum.c
```

这行命令行代码告诉编译器将C代码转换为汇编代码。gcc会调用一个程序序列来完成编译的不同阶段。后面的-S是停止（Stop）的意思，说明只完成gcc的第一阶段，即把C转化为汇编代码。-Og的作用是一种规范，告诉编译器做什么优化，这里表示按照原C代码生成汇编代码。

#### Assembly Characteristics

汇编代码有以下几个方面的特点：

##### Data Types

汇编代码有多种整数数据类型，大小为1字节，2字节，4字节和8字节，他们不区分符号与无符号的存储方式，指针和地址也以数字形式存储。

浮点数以不同的方式存储，有特定的寄存器组。

数组和其他一些数据结构在机器码级别不存在，一般由编译器人工构造。

##### Operations

汇编级程序每条指令能做到的事情其实相当有限，包括但不限于将数据移动到内存，进行加法和乘法运算等等，每次只能进行一些很简单的操作。

#### x86-64 Integer Registers

x86-64用下面这一套寄存器：

| 64-bit Register | 32-bit Register | 64-bit Register | 32-bit Register |
| :-------------- | :-------------- | :-------------- | :-------------- |
| `%rax`          | `%eax`          | `%r8`           | `%r8d`          |
| `%rbx`          | `%ebx`          | `%r9`           | `%r9d`          |
| `%rcx`          | `%ecx`          | `%r10`          | `%r10d`         |
| `%rdx`          | `%edx`          | `%r11`          | `%r11d`         |
| `%rsi`          | `%esi`          | `%r12`          | `%r12d`         |
| `%rdi`          | `%edi`          | `%r13`          | `%r13d`         |
| **`%rsp`**      | **`%esp`**      | `%r14`          | `%r14d`         |
| `%rbp`          | `%ebp`          | `%r15`          | `%r15d`         |

这里是16个寄存器，分别对应了寄存器的%r和%e版本，其中%r是64位的版本，%e是32位的版本，也就是%r的低32位版本。

PS：前8个寄存器有特定的名字，而后8个寄存器由数字表示，这是历史传统原因。

#### Moving Data

关于最常见的movq指令：

mov指令有三种，可以把源从一个数据复制到目标。源和目标可以是被编入程序的立即数（immediate），可以是寄存器或者内存。

这里的寄存器其实是一个特别命名的储存位置，而内存通常是一个字节数组，程序中给出的是地址。

一般而言movq指令一般是如下格式：

| 源            | 目标                      |
| ------------- | ------------------------- |
| 立即数（Imm） | 寄存器（Reg）/内存（Mem） |
| 寄存器（Reg） | 寄存器（Reg）/内存（Mem） |
| 内存（Mem）   | 寄存器（Reg）             |

指令的格式：`movq Src,Dest`

`movq $0x4,%rax` ⇒ `temp = 0x4;`
`movq $147,(%rax)` ⇒ `*p = -147;`
`movq %rax,%rdx` ⇒ `temp2 = temp1;`
`movq %rax,(%rdx)` ⇒ `*p = temp;`
`movq (%rax),%rdx` ⇒ `temp = *p;`

当寄存器之前加了括号时，寄存器的值其实表示的是地址，计算时使用的是内存中地址对应的位置所存储的值。例：

(%rcx)⇒ (R) ⇒ Mem[Reg[R]]

寄存器的括号之前还可以加数字，这表示将地址偏移。例：

8(%rbp) ⇒ D(R) ⇒ Mem[D + Reg[R]]

最常用的格式其实有四个参数，因为可以再加一个偏移的索引参数和比例。例：

D(Rb,Ri,S) ⇒ Mem[Reg[Rb] + Reg[Ri] * S + D]

PS：这个表示方法在数组中很常用。

以下是一些实际的例子：

**寄存器初始值：**

| Register | Value    |
| :------- | :------- |
| `%rdx`   | `0xf000` |
| `%rcx`   | `0x0100` |

**地址计算表：**

| Expression      | Address Computation | Address   |
| :-------------- | :------------------ | :-------- |
| `0x8(%rdx)`     | `0xf000 + 0x8`      | `0xf008`  |
| `(%rdx,%rcx)`   | `0xf000 + 0x100`    | `0xf100`  |
| `(%rdx,%rcx,4)` | `0xf000 + 4*0x100`  | `0xf400`  |
| `0x80(,%rdx,2)` | `2*0xf000 + 0x80`   | `0x1e080` |

#### Address computation Instruction

关于lea指令（load effective address）：

这个lea指令的目标是通过C的&符号来计算地址，他的格式看起来像一个有源和目的地的mov指令，但是目的地只能是寄存器，实现的效果看起来是从内存中读取数据并载入到寄存器中，实际上寄存器中存储的是地址值。。

举个例子：`leaq (%rdi,%rdi,2)` ⇒ %rax # t <- x + x*2

#### Some Arithmetic Operations

其他运算方面的指令看起来会相对简单一些，都是两个或者参数。

两个参数的指令格式是源（Src）放在目标（Dest）之前。各种指令见下表：

| Format  | Computation                        |
| :------ | :--------------------------------- |
| `addq`  | `Src,Dest`    `Dest = Dest + Src`  |
| `subq`  | `Src,Dest`    `Dest = Dest - Src`  |
| `imulq` | `Src,Dest`    `Dest = Dest * Src`  |
| `salq`  | `Src,Dest`    `Dest = Dest << Src` |
| `sarq`  | `Src,Dest`    `Dest = Dest >> Src` |
| `shrq`  | `Src,Dest`    `Dest = Dest >> Src` |
| `xorq`  | `Src,Dest`    `Dest = Dest ^ Src`  |
| `andq`  | `Src,Dest`    `Dest = Dest & Src`  |
| `orq`   | `Src,Dest`    `Dest = Dest | Src`  |

一个参数的指令就像C中的一样，用来执行一些简单的操作。指令见下表：

| Instruction | Operand | Computation       |
| :---------- | :------ | :---------------- |
| `incq`      | `Dest`  | `Dest = Dest + 1` |
| `decq`      | `Dest`  | `Dest = Dest - 1` |
| `negq`      | `Dest`  | `Dest = - Dest`   |
| `notq`      | `Dest`  | `Dest = ~Dest`    |

PS：注意notq是C中的**~**这个是位运算，不是**！**运算。
