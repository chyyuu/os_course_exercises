# lab1 SPOC思考题

##**提前准备**
（请在上课前完成）

 - 完成lec4的视频学习和提交对应的在线练习
 - git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises in github repos。这样可以在本机上完成课堂练习。
 - 了解x86的保护模式，段选择子，全局描述符，全局描述符表，中断描述符表等概念，以及如何读写，设置等操作
 - 了解Linux中的ELF执行文件格式
 - 了解外设:串口，并口，时钟，键盘,CGA，已经如何对这些外设进行编程
 - 了解x86架构中的mem地址空间和io地址空间
 - 了解x86的中断处理过程（包括硬件部分和软件部分）
 - 了解GCC内联汇编
 - 了解C语言的可函数变参数编程
 - 了解qemu的启动参数的含义
 - 在piazza上就lec3学习中不理解问题进行提问
 - 学会使用 qemu
 - 在linux系统中，看看 /proc/cpuinfo的内容

## 思考题

### 启动顺序

1. 段寄存器的字段含义和功能有哪些？
	1. 代码段寄存器 CS（Code Segment）存放当前正在运行的程序代码所在段的段基址，表示当前使用的指令代码可以从该段寄存器指定的存储器段中取得，相应的偏移量则由IP提供
	2. 数据段寄存器 DS（Data Segment）指出当前程序使用的数据所存放段的最低地址，即存放数据段的段基址
	3. 堆栈段寄存器 SS（Stack Segment）指出当前堆栈的底部地址，即存放堆栈段的段基址
	4. 附加段寄存器 ES（Extra Segment）指出当前程序使用附加数据段的段基址，该段是串操作指令中目的串所在的段
	5. 附加段寄存器 FS
	6. 附加段寄存器 GS
2. 描述符特权级DPL、当前特权级CPL和请求特权级RPL的含义是什么？在哪些寄存器中存在这些字段？对应的访问条件是什么？
	* 参见[DPL,RPL,CPL 之间的联系和区别](http://blog.csdn.net/better0332/article/details/3416749)对特权级的表述。
3. 分析可执行文件格式elf的格式（无需回答）
	* ELF header的格式
	  * [文档：Header](http://wiki.osdev.org/ELF)
	  * [代码](https://github.com/chyyuu/ucore_os_lab/blob/master/labcodes_answer/lab1_result/libs/elf.h#L9)
	* proghdr的格式分析
	  * [文档：Program header](http://wiki.osdev.org/ELF)
	  * [代码](https://github.com/chyyuu/ucore_os_lab/blob/master/labcodes_answer/lab1_result/libs/elf.h#L28)

### 4.4 x86中断处理过程

1. 中断处理中硬件压栈内容？用户态中断和内核态中断的硬件压栈有什么不同？
	* 参见实验指导书[中断与异常](https://objectkuan.gitbooks.io/ucore-docs/lab1/lab1_3_3_2_interrupt_exception.html)部分。
2. 为什么在用户态的中断响应要使用内核堆栈？
	* 保护中断服务例程代码的安全。
3. trap类型的中断门与interrupt类型的中断门有啥设置上的差别？如果在设置中断门上不做区分，会有什么可能的后果?
	* 调用Interrupt Gate时，Interrupt会被CPU自动禁止
	* 调用Trap Gate时，CPU则不会去禁止或打开中断，而是保留它原来的样子
	* 如果在设置上不做区分，会导致重复触发中断

### 4.8 练习四和五 ucore内核映像加载和函数调用栈分析

1. 在kdebug.c文件中用到的函数`read_ebp`是内联的，而函数`read_eip`不是内联的。为什么要设计成这样？
	* `ebp`可以直接获得，若不内联则会得到错误的`ebp`值
	* 而由于没有直接获取`eip`值的指令，我们需要利用`call`指令将`eip`压栈的特性，通过调用`read_eip`函数来读出压在栈上的`eip`的值。若将`read_eip`内联，则不会有函数调用存在，无法获得`eip`的值。
```c
static __noinline uint32_t
read_eip(void) {
    uint32_t eip;
    asm volatile("movl 4(%%ebp), %0" : "=r" (eip));
    return eip;
}
```

### 4.9 练习六 完善中断初始化和处理

1. CPU加电初始化后中断是使能的吗？为什么？
	* 不是。CPU启动后，BIOS会在POST自检完成后在内存中建立中断向量表和中断服务程序。

## 开放思考题

1. 如何修改lab1, 实现在出现除零异常时显示一个字符串的异常服务例程？
2. 在lab1/bin目录下，通过`objcopy -O binary kernel kernel.bin`可以把elf格式的ucore kernel转变成体积更小巧的binary格式的ucore kernel。为此，需要如何修改lab1的bootloader, 能够实现正确加载binary格式的ucore OS？ (hard)
3. GRUB是一个通用的bootloader，被用于加载多种操作系统。如果放弃lab1的bootloader，采用GRUB来加载ucore OS，请问需要如何修改lab1, 能够实现此需求？ (hard)
4. 如果没有中断，操作系统设计会有哪些问题或困难？在这种情况下，能否完成对外设驱动和对进程的切换等操作系统核心功能？

## 课堂实践
### 练习一
在Linux系统的应用程序中写一个函数print_stackframe()，用于获取当前位置的函数调用栈信息。实现如下一种或多种功能：函数入口地址、函数名信息、参数调用参数信息、返回值信息。

### 练习二
在ucore内核中写一个函数print_stackframe()，用于获取当前位置的函数调用栈信息。实现如下一种或多种功能：函数入口地址、函数名信息、参数调用参数信息、返回值信息。
