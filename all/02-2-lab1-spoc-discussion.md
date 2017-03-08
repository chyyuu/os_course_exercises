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

## 个人思考题

NOTICE
- 有"w2l2"标记的题是助教要提交到学堂在线上的。
- 有"w2l2"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。
---

- 请描述ucore OS配置和驱动外设时钟的准备工作包括哪些步骤？ (w2l2)

ANSWER:

```
  - 对IDT的初始化，包了针时钟中断的中断描述符的设置
  - 对8259中断控制器进行初始化
  - 对8253时钟外设的初始化
  - 对EFLAG操作使能中断
 ```

- lab1中完成了对哪些外设的访问？ (w2l2)

ANSWER:

```
- 时钟,串口,并口，CGA，键盘
```

- lab1中的cprintf函数最终通过哪些外设完成了对字符串的输出？

ANSWER:

```
- 串口 并口 CGA
```

---

## 小组思考题

---

- lab1中printfmt函数用到了可变参，请参考写一个小的linux应用程序，完成实现定义和调用一个可变参数的函数。(spoc)
 
  - C可变参数函数实现 http://blog.csdn.net/weiwangchao_/article/details/4857567 
  - va_list & va_start & va_arg & va_end http://blog.csdn.net/skymingst/article/details/36690081

```
#include <stdio.h>
#include <stdarg.h>
void simple_va_fun(int start, ...)
{
       va_list arg_ptr;
       int nArgValue =start;
       int nArgCout=0;     //可变参数的数目
       va_start(arg_ptr,start); //以固定参数的地址为起点确定变参的内存起始地址。
       do {
          ++nArgCout;
          printf("the %d th arg: %d\n",nArgCout,nArgValue);//输出各参数的值
          nArgValue = va_arg(arg_ptr,int);                 //得到下一个可变参数的值
       } while(nArgValue != -1);
       va_end(arg_ptr);     
       return;
}

int main(int argc, char* argv[])
{
       simple_va_fun(100,-1);
       simple_va_fun(100,200,-1);
       return 0;
}
```



- 如果让你来一个阶段一个阶段地从零开始完整实现lab1（不是现在的填空考方式），你的实现步骤是什么？（比如先实现一个可显示字符串的bootloader（描述一下要实现的关键步骤和需要注意的事项），再实现一个可加载ELF格式文件的bootloader（再描述一下进一步要实现的关键步骤和需要注意的事项）...） (spoc)


- 如何能获取一个系统调用的调用次数信息？如何可以获取所有系统调用的调用次数信息？请简要说明可能的思路。(spoc)

- 如何修改lab1, 实现一个可显示字符串"THU LAB1"且依然能够正确加载ucore OS的bootloader？如果不能完成实现，请说明理由。


- 对于ucore_lab中的labcodes/lab1，我们知道如果在qemu中执行，可能会出现各种稀奇古怪的问题，比如reboot，死机，黑屏等等。请通过qemu的分析功能来动态分析并回答lab1是如何执行并最终为什么会出现这种情况？

- 对于ucore_lab中的labcodes/lab1,如果出现了reboot，死机，黑屏等现象，请思考设计有效的调试方法来分析常在现背后的原因。


- 在X86中设置了4种特权级，ring0~ring3，用户态是ring3，内核态是ring0，对于这两种状态来说，切换是非常常见的。但是既然设置了ring1和ring2，那么肯定是可以切换到的。请给出一个例子来验证这个问题，包括4个特权级之间的切换，并显示。[challenge]

---

## 开放思考题

---

- 如何修改lab1, 实现在出现除零错误异常时显示一个字符串的异常服务例程的lab1？



- 在lab1/bin目录下，通过`objcopy -O binary kernel kernel.bin`可以把elf格式的ucore kernel转变成体积更小巧的binary格式的ucore kernel。为此，需要如何修改lab1的bootloader, 能够实现正确加载binary格式的ucore OS？ (hard)
 

- GRUB是一个通用的bootloader，被用于加载多种操作系统。如果放弃lab1的bootloader，采用GRUB来加载ucore OS，请问需要如何修改lab1, 能够实现此需求？ (hard)


- 如果没有中断，操作系统设计会有哪些问题或困难？在这种情况下，能否完成对外设驱动和对进程的切换等操作系统核心功能？

---

## 各知识点的小练习

### 4.1 启动顺序
---
- 读入ucore内核的代码？


- 跳转到ucore内核的代码？


- 段寄存器的字段含义和功能有哪些？

- 什么是全局描述符表GDT和局部描述符表LDT？

- 全局描述符表的[初始化代码]

(https://github.com/chyyuu/ucore_os_lab/blob/master/labcodes_answer/lab1_result/boot/bootasm.S#L49)？


- GDT内容的设置格式？初始映射的基址和长度？特权级的设置位置？

- 描述符特权级DPL、当前特权级CPL和请求特权级RPL的含义是什么？在哪些寄存器中这些字段？对应的访问条件是什么？ (challenge)写出一些简单的小程序（c or asm）来体现这些特权级的区别和联系。


```
 - 建议参见链接“ http://blog.csdn.net/better0332/article/details/3416749 ”对特权级的表述，并查阅指令手册。 
```
 

- CR0控制寄存器的字段含义和功能有哪些？使能保护模式的控制如何实现？

```
CR0中PE标志位设置
 
 https://github.com/chyyuu/ucore_os_lab/blob/master/labcodes_answer/lab1_result/boot/bootasm.S#L52 
```

- 分析可执行文件格式elf的格式。

ANSWER:

 1. ELF header的格式；([文档：Header](http://wiki.osdev.org/ELF)、[代码](https://github.com/chyyuu/ucore_os_lab/blob/master/labcodes_answer/lab1_result/libs/elf.h#L9)) 
 1. proghdr的格式分析；（[文档：Program header](http://wiki.osdev.org/ELF)、[代码](https://github.com/chyyuu/ucore_os_lab/blob/master/labcodes_answer/lab1_result/libs/elf.h#L28)）


- ucore内核的elf是否要求连续存放？为什么？


---

### 4.2 C函数调用的实现
---

- 函数调用的stackframe结构？函数调用的参数传递方法有哪几种？
  1. [Understanding the Stack](http://www.cs.umd.edu/class/sum2003/cmsc311/Notes/Mips/stack.html)：函数调用时堆栈状态分析
  1. [print_stackframe](https://github.com/chyyuu/ucore_os_lab/blob/master/labcodes_answer/lab1_result/kern/debug/kdebug.c#L292)：ucore中输出当前堆栈状态的函数实现


- 系统调用的stackframe结构？系统调用的参数传递方法有哪几种？

 

- 分析ucore的系统调用实现代码；


- 分析Linux的系统调用实现代码；

- 比较不同特权级的中断切换时的堆栈变化差别；(challenge)写出一些简单的小程序（c or asm）来显示出不同特权级的的中断切换的堆栈变化情况。


---

### 4.3 GCC内联汇编
---

- 使用内联汇编的原因？

特权指令、性能优化

- 对ucore中的一段内联汇编进行完整的解释？
 
 * [GCC-Inline-Assembly-HOWTO](http://ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html)：有_syscall3的内联汇编说明；

---

### 4.4 x86中断处理过程
---

4.4 x86中断处理过程

- 中断描述符表IDT的结构？



- IDT表的格式和初始化代码在哪？



- 中断描述表到中断服务例程的地址计算过程？



- IDTR的初始化代码在哪？


- 中断处理中硬件压栈内容？用户态中断和内核态中断的硬件压栈有什么不同？


- 为什么在用户态的中断响应要使用内核堆栈？
  
  保护中断服务例程代码的安全；


- 中断处理中硬件保存了哪些寄存器？

- trap类型的中断门与interrupt类型的中断门有啥设置上的差别？如果在设置中断门上不做区分，会有什么可能的后果?

---

### 4.5 练习一 ucore编译过程
---

- gcc编译、ld链接和dd生成两个映像对应的makefile脚本行？


- 函数print_stackframe中要调用函数print_debuginfo(uintptr_t eip)来打印函数源码位置信息

```
  print_stackframe(void)
   eip = read_eip();
   #option 1
   print_debuginfo(eip - 1);
   #option 2
   print_debuginfo(eip );
```


- 对于如下5条语句执行后得到的5个eip（类型为uint32_t）的结果的数值关系是什么？
```
  eip = ((uint32_t *)ebp)[2];
  eip = ((uint32_t *)ebp)[1];
  eip = ((uint32_t *)(ebp+4);
  eip = ((uint32_t *)(ebp+2);
  eip = ((uint32_t *)(ebp+1);
```        

---

### 4.6 练习二 qemu和gdb的使用
---

- qemu的命令行参数含义解释？



- gdb命令格式？反汇编、运行、断点设置


### 4.7 练习三 加载程序
---

- A20的使能代码分析？


- 生成主引导扇区的过程分析？


- 保护模式的切换代码？


- 如何识别elf格式？对应代码分析？


- 跳转到elf的代码？


- 函数调用栈获取？


---

### 4.8 练习四和五 ucore内核映像加载和函数调用栈分析
---

- 如何识别elf格式？对应代码分析？


- 跳转到elf的代码？

- 函数调用栈获取？

- 函数read_ebp是inline的，而函数read_eip是__noinline的，能否正好相反设置，即设置函数read_ebp是_noinline的，而函数read_eip是inline的？为什么？

---


### 4.9 练习六 完善中断初始化和处理
---

- 各种设备的中断初始化？

- 中断描述符表IDT的排列顺序？


- CPU加电初始化后中断是使能的吗？为什么？

- 中断服务例程的入口地址在什么地方设置的？

- alltrap的中断号是在哪写入到trapframe结构中的？


- trapframe结构？

---

## v9-cpu相关题目
---

### 提前准备
```
download os_tutorial_lab
cd os_tutorial_lab/v9_computer/os_user_task_syscall
make
make run

```

### v9-cpu的执行过程(challenge)
  1. 参考os*.c，写一个小程序，能够显示在用户态无法执行的所有特权指令，能够显示出不同特权级的的中断切换的堆栈变化情况。
  1. 扩展em.c，可以打印v9-cpu执行的每一条指令和寄存器状态
  1. 扩展em.c，在产生fatal错误后，把这之前执行的n条指令和寄存器状态打印出来
  1. 扩展c.c和em.c，使得可以指定em.c能够跟踪并打印执行程序中对全局变量的读写或修改的指令，以及指令的执行情况
  1. 扩展em.c的debugger功能，可以设置断点
  1. 扩展em.c的debugger功能，可以任意打断当前执行程序的执行，回到debugger
