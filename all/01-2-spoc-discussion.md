# 操作系统概述

## 思考题

### 填空题

* 当前常见的操作系统主要用<u>C,C++,ASM</u>编程语言编写。
* "Operating system"这个单词起源于<u>Operator</u>。
* 在计算机系统中，控制和管理<u>各种资源</u>、有效地组织<u>多道程序</u>运行的系统软件称作<u>操作系统</u>。
* 允许多用户将若干个作业提交给计算机系统集中处理的操作系统称为<u>批处理</u>操作系统
* 你了解的当前世界上使用最多的32bit CPU是<u>ARM</u>，其上运行最多的操作系统是<u>Android</u>。
* 应用程序通过<u>系统调用</u>接口获得操作系统的服务。
* 现代操作系统的特征包括<u>并发性</u>，<u>共享性</u>，<u>虚拟性</u>，<u>异步性</u>，<u>持久性</u>。
* 操作系统内核的架构包括<u>宏内核</u>，<u>微内核</u>，<u>外核</u>。


### 问答题

- 请总结你认为操作系统应该具有的特征有什么？并对其特征进行简要阐述。

操作系统应该具有的特征有：虚拟性、并发性、异步性、共享性和持久性。
* 虚拟性：虚拟是指把一个物理上的实体变为若干个逻辑上的对应物。在操作系统中利用了多种虚拟技术，分别用来实现虚拟处理器、虚拟内存和虚拟外部设备。
* 并发性：并发是指两个或多个事件在同一时间间隔内发生，在多道程序环境下，一段时间内宏观上有多个程序在同时执行，而在同一时刻，单处理器环境下实际上只有一个程序在执行，故微观上这些程序还是在分时的交替进行。操作系统的并发是通过分时得以实现的。操作系统的并发性是指计算机系统中同时存在多个运行着的程序，因此它具有处理和调度多个程序同时执行的能力。
* 异步性：在多道程序环境下，允许多个程序并发执行，但由于资源有限，进程的执行不是一贯到底，而是走走停停，以不可预知的速度向前推进，这就是进程的异步性。异步性使得操作系统运行在一种随机的环境下，可能导致进程产生于时间有关的错误。但是只要运行环境相同，操作系统必须保证多次运行进程，都获得相同的结果。
* 共享性：系统中的资源可供内存中多个并发执行的进程共同使用。
* 持久性：通过实现文件系统，操作系统可以将程序以及数据存储在磁盘等存储介质中。
  详细解释可以参考[操作系统的特征](https://wizardforcel.gitbooks.io/wangdaokaoyan-os/content/2.html)。

- 为什么现在的操作系统基本上用C语言来实现？为什么没有人用python，java来实现操作系统？

C语言是编译型语言，有良好的性能，能够直接嵌入汇编，可以方便地操作硬件；Python，Java无法保证性能，不能直接操作硬件。
* 用Java实现的操作系统：[JavaOS](https://en.wikipedia.org/wiki/JavaOS)
* 用Python实现的操作系统：[pycorn](https://github.com/tornewuff/pycorn)，[pythonix](https://github.com/wfxpanisa/pythonix)

## 可选练习题

- 请分析并理解[v9\-computer](https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/docs/v9_computer.md)以及模拟v9\-computer的em.c。理解：在v9\-computer中如何实现时钟中断的；v9 computer的CPU指令，关键变量描述有误或不全的情况；在v9\-computer中的跳转相关操作是如何实现的；在v9\-computer中如何设计相应指令，可有效实现函数调用与返回；OS程序被加载到内存的哪个位置,其堆栈是如何设置的；在v9\-computer中如何完成一次内存地址的读写的；在v9\-computer中如何实现分页机制；

https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/tools/em.c
https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/docs/v9_computer.md

- 请编写一个小程序，在v9-cpu下，能够输出字符

https://github.com/chyyuu/os_tutorial_lab/tree/master/v9_computer/os_helloworld

- 输入的字符并输出你输入的字符


- 请编写一个小程序，在v9-cpu下，能够产生各种异常/中断

https://github.com/chyyuu/os_tutorial_lab/tree/master/v9_computer/os_bad_phys_addr
https://github.com/chyyuu/os_tutorial_lab/tree/master/v9_computer/os_divid_by_zero
https://github.com/chyyuu/os_tutorial_lab/tree/master/v9_computer/os_invalid_intruction
https://github.com/chyyuu/os_tutorial_lab/tree/master/v9_computer/os_page_fault
https://github.com/chyyuu/os_tutorial_lab/tree/master/v9_computer/os_timer_interrupt

- 请编写一个小程序，在v9-cpu下，能够统计并显示内存大小

