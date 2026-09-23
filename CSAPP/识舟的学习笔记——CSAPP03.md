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

汇编代码有多种整型数据类型大小为1字节，2字节，4字节和8字节
