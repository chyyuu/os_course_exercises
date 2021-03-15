# lec2：SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在不同操作系统（如linux, ucore,etc.)与不同硬件（如 x86, riscv, v9-cpu,etc.)中是如何相互配合来体现的。
- 安装好ucore/rcore实验环境，能够编译运行ucore/rcore labs中的源码。
- 会使用实验中的常用的工具和命令: objdump，nm，file, strace，gcc, gdb等，了解这些命令的用途。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

- 你理解的risc-v的特权模式有什么区别？不同 模式在地址访问方面有何特征？

  

### 课堂实践练习

#### 练习一

1. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。


#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。


## 问答题

#### Q1：应用程序能直接调用内核的函数吗？说明原因，举个例子。
A：

#### Q2：内核能直接调用应用程序的函数吗？说明原因，举个例子。
A：

#### Q3：请说出你知道的特权指令或特权寄存器（x86/arm/mips/risc-v）
A:

#### Q4: 除了基于页表的地址空间隔离，请问还能想到其他的地址空间隔离方式吗？
A：

#### Q5：中断/异常有何区别？如果没有中断机制，应用程序/操作系统要如何与外设进行交互？
A：

#### Q6：你是如何理解和描述OS与APP的交互接口？
A：

#### Q7：你是如何理解和描述OS与CPU的交互接口？
A：

#### Q8：我们理解的RISC-V的Supervisor特权模式适合运行哪种类型的软件？
A:

#### Q9: 如果让一个通用操作系统（如Linux）运行在RISC-V的User模式，请问会发生什么现象？是否有相应的解决方法？
A：

#### Q10：RISC-V的中断产生后，CPU首先会在哪种特权模式下运行？
A：

#### Q11：根据课程介绍，安装qemu-RV64  v5.0+，通过qemu的monitor，查看ROM的指令是哪些？
A：https://rcore-os.github.io/rCore_tutorial_doc/chapter2/part5.html


## 参考资料
 - [v9 cpu architecture](https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/docs/v9_computer.md)
 - [RISC-V cpu architecture](http://www.riscvbook.com/chinese/)
 - [OS相关经典论文](https://github.com/chyyuu/aos_course_info/blob/master/readinglist.md)
