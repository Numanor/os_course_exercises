# lec2：lab0 SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises  in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在不同操作系统（如linux, ucore,etc.)与不同硬件（如 x86, riscv, v9-cpu,etc.)中是如何相互配合来体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9\-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

  - 进程：至少需要支持时钟中断；至少需要提供中断、异常、系统服务、调整特权级管理指令
  - 虚存：至少需要支持MMU；至少需要提供MMU、分段(分页)管理指令
  - 文件系统：至少需要稳定存储硬件；至少需要提供I/O操作指令

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？

  - 实模式&保护模式

    实模式下物理内存地址能直接被程序访问，而保护模式下程序内部的地址（虚拟地址）要由操作系统转化为物理地址去访问，程序对此一无所知

  - 物理地址

    CPU在地址总线上给出地直接读写硬件存储器的地址

  - 线性地址

    在段机制下逻辑地址和物理地址之间变换的中间层

  - 逻辑地址

    应用程序的访存指令直接给出的虚拟地址

- 你理解的risc-v的特权模式有什么区别？不同模式在地址访问方面有何特征？

  risc-v架构定义了三种工作模式，又称特权模式（Privileged Mode）：

  - Machine Mode：机器模式，简称M Mode。
  - Supervisor Mode：监督模式，简称S Mode。
  - User Mode：用户模式，简称U Mode。

  risc-v架构定义M Mode为必选模式，另外两种为可选模式。通过不同的模式组合可以实现不同的系统。

  risc-v架构也支持几种不同的存储器地址管理机制，包括对于物理地址和虚拟地址的管理机制，使得risc-v架构能够支持从简单的嵌入式系统（直接操作物理地址 M Mode, PMP unit）到复杂的操作系统（直接操作虚拟地址 S Mode）的各种系统。

  https://content.riscv.org/wp-content/uploads/2017/05/riscv-privileged-v1.10.pdf

- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```
  “:”后的数字表示每一个域在结构体中所占的位数

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？

运行测试[代码](https://github.com/Numanor/os_tutorial_lab/blob/master/x86-32/userapp_related/lab0_ex3.c)得到：

```
intr is 0x20003
```

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

#### reference

 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
