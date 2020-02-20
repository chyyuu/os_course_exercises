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

- 你理解的x86的实模式和保护模式有什么区别？你认为从实模式切换到保护模式需要注意那些方面？

- 物理地址、线性地址、逻辑地址的含义分别是什么？它们之间有什么联系？

- 你理解的risc-v的特权模式有什么区别？不同 模式在地址访问方面有何特征？

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

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。


## 问答题

### 第一节
#### Q1：应用程序能直接调用内核的函数吗？说明原因，举个例子。
A：应用程序不能直接调用内核的函数。因为这会破坏内核必须提供的隔离性、可用性。比如：在内核态，所有用户程序的地址空间均可访问。一些别有用心的程序可能会修改或查看其他程序的内存，导致对应的程序无法正常执行或机密数据被窃取，这都违反了隔离性。又或者，一些程序可能会通过这种方法干扰内核正常的进程调度，使得其他程序无法正常运行，这使得内核的可用性降低。


#### Q2：内核能直接调用应用程序的函数吗？说明原因，举个例子。
A：理论上来说，在内核态允许访问用户态代码与数据的情况下，内核是可以调用应用程序的函数的。但是这种“调用”并非函数调用，而是通过二进制接口 ABI(Application Binary Interface)，对于内核而言，由于与应用程序的抽象层级不同，缺乏相应的上下文环境，在它看来只是执行了若干条意义不明的指令而已，并没有什么意义。举个例子，用户态程序看库函数，看到的是 printf, stdin, stdout 等等，而系统态看库函数，看到的是一堆数据：printf 的地址，stdin 的地址，stdout 的地址。因此，不应该允许内核态执行用户态代码。

况且，这样还有安全问题。请问是什么安全问题？

答案：一个基本假设是：OS 认为应用程序的代码是不可信的，要么有 bug 要么是恶意代码。只不过他们运行在用户态，硬件提供的隔离机制来限制应用程序代码的 bug / 恶意行为造成的破坏不会干扰到整个系统执行。如果在拥有极高特权的内核态执行有 bug / 恶意的用户态代码，这些代码造成破坏将会波及整个系统。

#### Q3：请说出你知道的特权指令或特权寄存器（x86/arm/mips/risc-v）
A:     寄存器：在 RISC-V 中，Supervisor 态中断相关的寄存器如 *sstatus,scause,stvec,sscratch,sepc *，以及与虚拟存储相关的寄存器如 *satp *。在 Machine 态则有与时钟中断相关的寄存器 *mtime,mtimecmp* 等等。MIPS 的 CP0 寄存器。

指令：RISC-V ： csr 读写，{m,s}ret。ARM：MSR，MRS。MIPS：mfc0，mtc0。

#### Q4: 除了基于页表的地址空间隔离，请问还能想到其他的地址空间隔离方式吗？
A：如 x86 的段式管理、甚至物理隔离等。MIPS 的 Joint TLB。

#### Q5：中断/异常有何区别？如果没有中断机制，应用程序/操作系统要如何与外设进行交互？
A：*异常*是在执行一条指令的过程中**同步**触发，如执行超出当前特权级的指令、访问更高特权级的寄存器、缺页异常、除零等错误会触发异常；同时，在 RISC-V 中，低特权级也可以主动通过 *ecall(Environment Call) *指令进入更高的特权级完成在低特权级所不允许的任务，**系统调用**便依赖这个机制完成 。与之相比，*中断*何时发生则与指令的执行无关，它来源于外设、计时器等生成的中断信号，因此是**异步**的。但是，尽管来源不同，CPU 却通过相同的机制来处理它们，仅仅通过中断号加以区分。

外设通常与 CPU 约定一系列**状态寄存器**，CPU 可以通过它们来查看外设目前的状态。如果没有中断机制，就只能在一个 while 循环中通过访问对应的状态寄存器确定交给外设的任务完成，才能进行接下来的工作，此举浪费了 CPU 资源。如果依赖中断的话，CPU 在这段等待过程可以去完成其他任务(切换到其他进程)，在外设完成任务后，它会通过中断告诉 CPU 任务已完成，那么 CPU 可以继续进行之前未完成的任务。

#### Q6：在计算机系统中实现隔离的方法有哪几种？
A：

2. 1. 地址空间：限制程序在指定状态下，可以访问的地址空间范围；
2. 特权模式/中断机制：限制程序在指定状态下可以执行指令类型；在程序执行非法指令时触发异常，由高级别的监管程序进行处置；

#### Q7：隔离的角度看，操作系统中的用户态和内核态有什么区别？
A：

1. 用户态程序只能访问本进程的用户地址空间的内容，只能执行普通指令。
2. 内核态程序可以访问本进程的内核地址空间和用户地址空间的内容，可以执行普通和特权指令。

### 第二节
#### Q1：你是如何理解和描述OS与APP的交互接口？
A：从应用程序自身的角度看，它的运行离不开内核通过系统调用提供的服务，但内核提供的是二进制接口(ABI)，与函数调用(高级语言如 C/C++)不同，它与平台相关，需要与汇编指令、寄存器打交道。而我们现在使用高级语言编程，之所以可以屏蔽这些底层细节，是因为我们并不直接使用系统调用，而是调用由下层函数库如 libc 等对系统调用进行封装之后的方便易用的用户态接口(API)。如果上过‘汇编语言编程’(汇编小学期)，大家应该已经体会过使用裸的系统调用是怎么回事。

然而，从内核的角度看，它是为了允许多个程序高效运行、相互隔离，才抽象出了系统调用接口，由此利用中断/特权级、虚拟存储机制实现了它的抽象。

#### Q2：你是如何理解和描述OS与CPU的交互接口？
A：从 OS 的角度来看，它负责管理一系列的硬件资源，向 OS 提供的接口是：

CPU : OS 使用 CPU 提供的指令集与寄存器；

内存：OS 管理物理内存；

I/O 外设：与 CPU 之间约定若干寄存器(状态、数据、控制)进行交互。

这三者不是独立的。比如 CPU 内部也有控制内存和外设的功能，为我们开发内核提供了更方便的接口。

#### Q3：我们理解的RISC-V的Supervisor特权模式适合运行哪种类型的软件？
A: 相比 M 态， S 态更适合运行如 OS 这种允许多个应用同时运行的软件，因为它支持虚拟存储，使得隔离可以完成。而 M 态主要用于嵌入式开发领域，它只运行单应用，可以得到更高的性能。

#### Q4: 如果让一个通用操作系统（如Linux）运行在RISC-V的User模式，请问会发生什么现象？是否有相应的解决方法？
A：由于不能访问高特权级指令、寄存器，通用操作系统在 U 态不能正常运行。也就是说根本不能直接运行。

因此，应该使用虚拟化技术来欺骗 OS，让它以为自己还运行在 S 态。比如模拟器 Qemu ，虚拟机 Virtualbox/VMWare。它们并不是将 CPU/内存/外设 均使用代码模拟，也可以利用真实的硬件资源来完成虚拟化任务。

#### Q5：RISC-V的中断产生后，CPU首先会在哪种特权模式下运行？
A：默认情况下，所有的中断/异常都会在最高的特权级 Machine 态处理。但是，对于内核开发而言，除了极少数因机制所限不得不在 M 态执行之外，其他的在 S 态处理也可以满足内核的需求。这样降低了开发复杂度，也减少了特权级切换的开销。

为了达到目的，需要在 Machine 态修改中断/异常代理寄存器 *mideleg,medeleg *，来将特定的中断、异常从 M 态代理到更低的特权级进行处理。详情参考特权级文档 3.1.13 小节。值得一提的是，架构保证中断不会使得特权级降低。

#### Q6：RISC-V有哪几个特权级？
 1. U: User
 2. S: Supervisor
 3. H: Hypervisor
 4. M: Machine

### 第三节 
#### Q1：
## Q1：在你熟悉的操作系统上用你熟悉的语言写一个跟踪功能函数，可以获取当前的函数调用栈和函数调用参数信息。
没有参考答案。

#### Q2：高级语言的编译到机器指令的调用约定都约定什么内容？
1. 函数调用参数保存位置和顺序
2. 函数返回值位置
3. 寄存器保存和使用约定

### 第四节
#### Q1：根据课程介绍，安装qemu-4.1+，通过qemu的monitor，查看ROM的指令是哪些？
A：https://rcore-os.github.io/rCore_tutorial_doc/chapter2/part5.html



## 参考资料
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
 - [v9 cpu architecture](https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/docs/v9_computer.md)
 - [RISC-V cpu architecture](http://www.riscvbook.com/chinese/)
 - [OS相关经典论文](https://github.com/chyyuu/aos_course_info/blob/master/readinglist.md)
