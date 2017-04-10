# lab4 spoc 思考题

- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 视频相关思考题

### 13.1 总体介绍

1. 为什么讲基本原理时先讲进程后讲线程，而在做实验时先做线程后做进程？

 > 实现会容易：只需要实现PCB中的子集TCB，不用关注PCB中的资源管理部分；

2. ucore的线程控制块数据结构是什么？

 > 在ucore中只有一个PCB数据结构，进程和线程都使用这一个数据结构；

### 13.2 关键数据结构

1. 分析proc_struct数据结构，说明每个字段的用途，是线程控制块或进程控制块的，会在哪些函数中修改。

 > 寄存器状态、堆栈、当前指令指针等信息是线程控制块的；
 
 > mm, vma等内存管理字段是进程控制块的；

1. 如何知道ucore的两个线程同在一个进程？

 > 查看线程控制块中cr3是否一致；

1. context和trapframe分别在什么时候用到？

 > trapframe在中断响应时用到；context在线程切换时用到；

1. 用户态或内核态下的中断处理有什么区别？在trapframe中有什么体现？

 > 在用户态中断响应时，要切换到内核态；而在内核态中断响应时，没有这种切换；
 
 > tf_esp, tf_ss字段

1. 分析trapframe数据结构，说明每个字段的用途，是由硬件或软件保存的，在内核态中断响应时是否会保存。

 > tf_eip, tf_cs等由硬件保存；tf_esp, tf_ss等在用户态响应时由硬件保存；
 
 > 通用寄存器由软件保存；( lab4/kern/trap/trapentry.S )

### 13.3 执行流程

1. kernel_thread创建的内核线程执行的第一条指令是什么？它是如何过渡到内核线程对应的函数的？

 > tf.tf_eip = (uint32_t) kernel_thread_entry;

 > /kern-ucore/arch/i386/init/entry.S

 > /kern/process/entry.S

2. 内核线程的堆栈初始化在哪？

 > setup_stack

 > tf和context中的esp

3. fork()父子进程的返回值是不同的。这在源代码中的体现中哪？

4. 内核线程initproc的第一次执行流程是什么样的？能跟踪出来吗？

5. 分析线程切换流程，找到内核堆栈、页表、寄存器切换的代码位置。

6. 分析C语言中调用汇编函数switch_to()的参数传递位置。

7. 分析内核线程idleproc的创建流程，说明线程切换后执行的第一条是什么。

 > proc->context.eip = forkret;

 > tf.tf_eip = (uint32_t) kernel_thread_entry;

8. 分析内核线程initproc的创建流程，说明线程切换后执行的第一条是什么。

 > 
 
## 小组练习与思考题

(1)(spoc) 理解内核线程的生命周期。

> 需写练习报告和简单编码，完成后放到git server 对应的git repo中

### 掌握知识点
1. 内核线程的启动、运行、就绪、等待、退出
2. 内核线程的管理与简单调度
3. 内核线程的切换过程

### 练习用的[lab4 spoc exercise project source code](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab4/lab4-spoc-discuss)


请完成如下练习，完成代码填写，并形成spoc练习报告

### 1. 分析并描述创建分配进程的过程

> 注意 state、pid、cr3，context，trapframe的含义

### 练习2：分析并描述新创建的内核线程是如何分配资源的

> 注意 理解对kstack, trapframe, context等的初始化


当前进程中唯一，操作系统的整个生命周期不唯一，在get_pid中会循环使用pid，耗尽会等待

### 练习3：

阅读代码，在现有基础上再增加一个内核线程，并通过增加cprintf函数到ucore代码中
能够把内核线程的生命周期和调度动态执行过程完整地展现出来

### 练习4 （非必须，有空就做）：

增加可以睡眠的内核线程，睡眠的条件和唤醒的条件可自行设计，并给出测试用例，并在spoc练习报告中给出设计实现说明

### 扩展练习1: 

进一步裁剪本练习中的代码，比如去掉页表的管理，只保留段机制，中断，内核线程切换，print功能。看看代码规模会小到什么程度。

### 思考：

switch_to函数的汇编码与编译器生成的C函数的汇编码区别是什么？能否用C语言实现switch_to函数？为什么？
