# 识舟的学习笔记——2015 CMU 15-213 CSAPP 深入理解计算机系统03——Maching-Level Programming

## 目录/框架

## 课程内容

### Basics

#### History of Intel processors and architextures

目前主流有三大CPU架构：x86，ARM，RISC-V，我们先说x86。

对于英特尔而言，x86一开始是一个口头上的称呼，原因是历史上第一个芯片是8086，后来跳过81，出现了8286，8386等，这些都有86，所以人们把这称作x86。

x86早期也被叫做CISC，主要是受到80年代早期，CISC和RISC大战的影响。当时风靡一时的是RISC（Reduced Instruction Set Computer），也就是精简指令集计算机。其实之前没有人给x86取名为CSIC，但是许多用RISC的人把x86的处理器叫做CISC（Complex Instruction Set Computer），也就是复杂指令集计算机，这种说法其实是带有贬义的。

