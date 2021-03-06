# **lec5: SPOC思考题**

**##提前准备**
**（请在上课前完成）**

- **完成lec５的视频学习和提交对应的在线练习**
- **git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises in github repos。这样可以在本机上完成课堂练习。**
- **理解连续内存动态分配算法的实现（主要自学和网上查找）**

**NOTICE**
- **有"hard"标记的题有一定难度，鼓励实现。**
- **有"easy"标记的题很容易实现，鼓励实现。**
- **有"midd"标记的题是一般水平，鼓励实现。**


## **思考题**
---

## **“连续内存分配”与视频相关的课堂练习**

### **5.1 计算机体系结构和内存层次**

**1.操作系统中存储管理的目标是什么？**

- 抽象：抽象出逻辑地址空间，程序不需要考虑实际物理地址

- 虚拟化：使得程序“感觉”专享一块空间
- 共享化：不同程序可以共享一块空间
- 安全保护：控制应用程序对于内存的访问

### **5.2 地址空间和地址生成**

**1.描述编译、汇编、链接和加载的过程是什么？**

- 编译：从高级语言生成汇编代码
- 汇编：将汇编代码转换成为目标机器代码
- 链接：处理多个机器代码、标记等关系，生成可执行代码
- 加载：将可执行代码load到内存中去，修改程序中绝对地址以适应新的起始内存地址，保证程序运行正确

**2.动态链接如何使用？尝试在Linux平台上使用LD_DEBUG查看一个C语言Hello world的启动流程。  (optional)**

- .dynamic 保存了动态链接的一些基本信息，动态链接器“自举”，完成重定位和初始化，便完成动态链接。

### **5.3 连续内存分配**
**1.什么是内碎片、外碎片？**

- 内碎片：分配给程序的内存大小比实际要用的大小大，因此产生的无法使用的空间
- 外碎片：分配给程序内存之间的无法使用的内存空间

**2.最先匹配会越用越慢吗？请说明理由（可来源于猜想或具体的实验）？**

- 如果一直不释放，那么相当于用于寻找的链表越来越长，且越多排在前面的不满足条件，因此会越来越慢
- 如果随机使用，释放，感觉很难得出“是否越来越慢”的结论

**3.最差匹配的外碎片会比最优适配算法少吗？请说明理由（可来源于猜想或具体的实验）？**

- 是的。以为每次都是寻找最大的内存，分割后还可以装其他的程序。

**4.理解0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm算法中分区释放后的合并处理过程？ (optional)**

- 前三者：在释放的时候，查看上下是否也是空闲的，如果是，那么就合并，插入空闲空间的链表中
- 伙伴系统：释放的时候，查看上下是否空闲，而后进一步查看空闲的是不是与释放的**同样**大小，如果是则合并，循环这样的过程直到找不到相邻的同大小的空闲块为止。




### **5.4 碎片整理**
**1.对换和紧凑都是碎片整理技术，它们的主要区别是什么？为什么在早期的操作系统中采用对换技术？  **

- 紧凑：将程序在内存空间中移动，减少使得多个外碎片可以组合成为大块可用内存
- 对换：将位于等待状态的进程放入外存中，腾出的内存空间用于保存即将运行的程序
- 采取对换是因为处理比较简单，而紧凑则要考虑copy代价、地址变换、程序状态保存等问题比较复杂。

**2.一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？**

- 如果进程在队列中优先级较高，将该进程从外存中放入内存，执行该进程

### **5.5 伙伴系统**
**1.伙伴系统的空闲块如何组织？**

- 按$2^n$ 组织

**2.伙伴系统的内存分配流程？伙伴系统的内存回收流程？**

- 内存分配：找到第一个不小于需求的内存块（$2^n $）,如果该内存块的一半还满足需求，分为一半，直到即将不满足需求，将可切分满足需求的最小块分配给该进程。实际上，不同内存块($2^n$ )通过不同的链表来维护，每个链表记录同等大小的内存块的首地址，在分割的过程中，将大的从大的链表中删除，并且加入小的链表。 
- 内存回收：释放该内存，并且看相邻是否有相同大小的空闲空间，是则合并，循环该过程直到不可合并为止。

## **课堂实践**

**观察最先匹配、最佳匹配和最差匹配这几种动态分区分配算法的工作过程，并选择一个例子进行分析分析整个工作过程中的分配和释放操作对维护数据结构的影响和原因。**

  * **[算法演示脚本的使用说明](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep3-malloc.md)**
  * **[算法演示脚本](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep3-malloc.py)**

**例如：**
```
python ./ostep3-malloc.py -S 100 -b 1000 -H 4 -a 4 -l ADDRSORT -p BEST -n 5 -c
python ./ostep3-malloc.py -S 100 -b 1000 -H 4 -a 4 -l ADDRSORT -p FIRST -n 5 -c
python ./ostep3-malloc.py -S 100 -b 1000 -H 4 -a 4 -l ADDRSORT -p WORST -n 5 -c
```

### **扩展思考题 (optional)**

1. **请参考xv6（umalloc.c），ucore lab2代码，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在Linux应用程序/库层面，用C、C++或python来实现malloc/free，给出你的设计思路，并给出可以在Linux上运行的malloc/free实现和测试用例。**


2. **阅读[slab分配算法](http://en.wikipedia.org/wiki/Slab_allocation)，尝试在应用程序中实现slab分配算法，给出设计方案和测试用例。**

