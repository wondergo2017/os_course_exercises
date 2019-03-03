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

- **你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？**

  切换进程需要时钟中断的支持

  虚存实现，可能需要用到MMU、TLB

  文件系统，需要合适的存储介质和技术，比如RAID

  特权指令有中断处理流程的各项（开闭中断使能，转服务程序，保存恢复现场等），管理内存分配，切换工作模式等。

- **你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？**

  **实模式，地址空间与实际地址空间一样，无法有效地进行内存管理，同时没有内存保护机制。**

  保护模式支持内存分页机制，对虚拟内存有较好的支持，不同的应用程序可以运行在不同的优先级上，起到比较好的保护作用。

  物理地址：处理器提交到总线上用于访问实际内存和外设的最终地址

  线性地址：在虚存管理之下每个运行的应用程序访问的地址空间（程序感觉到的连续地址空间）

  逻辑地址：应用程序直接使用的地址空间

- **你理解的risc-v的特权模式有什么区别？不同 模式在地址访问方面有何特征？**

  一共有四种模式：

  1. Machine Mode: 能够访问所有的硬件资源，但是没有虚存支持
  2. Hypervisor Mode：用来实现虚拟化，但还没有具体细化
  3. Supervisor Mode：运行OS的核，实现了MMU，提供了各种各样的页表形式
  4. User Mode：运行User-level的代码。

  参考 https://genode.org/documentation/articles/riscv

- **理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）**

- **对于如下的代码段，请说明":"后面的数字是什么含义**

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

​	答：所使用的bit数

- **对于如下的代码段，**

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
**如果在其他代码段中有如下语句，**

```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
**请问执行上述指令后， intr的值是多少？**

答： 0x20003  (unsigned 只截取了低32位)

### 课堂实践练习

#### 练习一

1. **请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。**

   ![1551606160893](01-3-lab0-spoc-discussion.assets/1551606160893.png)

   比如此段直接翻译：

   38：从0x64端口读入一个byte到%al寄存器

   39：%al寄存器的值与0x2做AND操作，如果是0，那么ZF置1

   40：如果ZF不为0（就是%al 低第2位为1）那么继续循环

   功能是等待OUT端口空闲，这样就可以继续执行输出的代码。

2. **(option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。**

#### 练习二

**宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。**

答：

可以简化代码编写。

比如之前描述的SETGATE这个宏定义，就可以减少代码编写的工作量。

#### reference
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
