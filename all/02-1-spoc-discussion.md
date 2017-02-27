#lec 3 SPOC Discussion

##**提前准备**
（请在上课前完成）


 - 完成lec3的视频学习和提交对应的在线练习
 - git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises  　in github repos。这样可以在本机上完成课堂练习。
 - 仔细观察自己使用的计算机的启动过程和linux/ucore操作系统运行后的情况。搜索“80386　开机　启动”
 - 了解控制流，异常控制流，函数调用,中断，异常(故障)，系统调用（陷阱）,切换，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在linux, ucore, v9-cpu中的os*.c中是如何具体体现的。
 - 思考为什么操作系统需要处理中断，异常，系统调用。这些是必须要有的吗？有哪些好处？有哪些不好的地方？
 - 了解在PC机上有啥中断和异常。搜索“80386　中断　异常”
 - 安装好ucore实验环境，能够编译运行lab8的answer
 - 了解Linux和ucore有哪些系统调用。搜索“linux 系统调用", 搜索lab8中的syscall关键字相关内容。在linux下执行命令: ```man syscalls```
 - 会使用linux中的命令:objdump，nm，file, strace，man, 了解这些命令的用途。
 - 了解如何OS是如何实现中断，异常，或系统调用的。会使用v9-cpu的dis,xc, xem命令（包括启动参数），分析v9-cpu中的os0.c, os2.c，了解与异常，中断，系统调用相关的os设计实现。阅读v9-cpu中的cpu.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
 - 在piazza上就lec3学习中不理解问题进行提问。

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
-  BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？

{%s%}
BIOS完成硬件初始化和自检后，会根据CMOS中设置的启动顺序启动相应的设备，这里假定按顺序系统要启动硬盘。但此时，文件系统并没有建立，BIOS也不知道硬盘里存放的是什么，所以BIOS是无法直接启动操作系统。另外一个硬盘可以有多个分区，每个分区都有可能包括一个不同的操作系统，BIOS也无从判断应该从哪个分区启动，所以对待硬盘，所有的BIOS都是读取硬盘的0磁头、0柱面、1扇区的内容，然后把控制权交给这里面的MBR(Main Boot Record）。

MBR由两个部分组成：即主引导记录MBR和硬盘分区表DPT。在总共512字节的主引导分区里其中MBR占446个字节（偏移0--偏移1BDH)，一般是一段引导程序，其主要是用来在系统硬件自检完后引导具有激活标志的分区上的操作系统。DPT占64个字节（偏移1BEH--偏移1FDH),一般可放4个16字节的分区信息表。最后两个字节“55，AA”（偏移1FEH，偏移1FFH)是分区的结束标志。
{%ends%}
  
- 比较UEFI和BIOS的区别。
 
 {%s%}
 UEFI，全称Unified Extensible Firmware Interface，即“统一的可扩展固件接口”, 是适用于电脑的标准固件接口，旨在代替BIOS（基本输入/输出系统）。此标准由UEFI联盟中的140多个技术公司共同创建，其中包括微软公司。UEFI旨在提高软件互操作性和解决BIOS的局限性。 
 
 UEFI启动对比BIOS启动的优势有三点：
 
   - 安全性更强：UEFI启动需要一个独立的分区，它将系统启动文件和操作系统本身隔离，可以更好的保护系统的启动。
   - 启动配置更灵活：EFI启动和GRUB启动类似，在启动的时候可以调用EFIShell，在此可以加载指定硬件驱动，选择启动文件。比如默认启动失败，在EFIShell加载U盘上的启动文件继续启动系统。
   - 支持容量更大:传统的BIOS启动由于MBR的限制，默认是无法引导超过2TB以上的硬盘的。随着硬盘价格的不断走低，2TB以上的硬盘会逐渐普及，因此UEFI启动也是今后主流的启动方式。
 
 {%ends%}

 
- 描述PXE的大致启动流程。

{%s%}
   - 客户端电脑开机后，BIOS设置从网络启动
   - 网卡中的PXE Boot ROM自测，获得控制权，然后发送一个动态获得IP地址的广播包。
   - DHCP服务器在收到该广播包后，发送给客户端分配IP地址的DHCP回应包。内容包括客户端的IP地址，TFTP服务器的IP地址（DHCP服务器的066选项），预设通讯通道，及开机启动文件（DHCP服务器的067选项）。该文件应该是一种由PXE启动规范规定的固定格式的可执行文件，类似于Windows XP正常启动时显示多重启动菜单之前的启动代码。 
   - 客户面收到DHCP回应后，则会响应一个FRAME，以请求传送启动文件。之后，服务端将和客户机再进行一系列应答，决定启动参数。
   - 客户端通过TFTP通讯协议从服务器下载开机启动文件。启动文件接收完成后，将控制权转交给启动块，完成PXE启动。客户端通过这个开机影像文件开机，这个开机文件可以只是单纯的开机程式也可以是操作系统。
{%ends%}

## 3.2 系统启动流程

- 分区引导扇区的结束标志是什么？{%s%}0X55AA{%ends%}
- 如何指定内核加载的参数？{%s%}加载程序的启动菜单{%ends%}
- BIOS、BIOS-MBR、BIOS-GPT的区别是什么？{%s%}只支持一个分区、支持4个启动分区、支持更多启动分区{%ends%}
- 在UEFI中的可信启动有什么作用？{%s%}通过启动前的数字签名检查来保证启动介质的安全性{%ends%}
- 了解NTLDR的启动流程。
 
 {%s%}
   - 访问启动盘的文件系统 (FAT 或者 NTFS)
   - 如果 Windows 处于冬眠模式，将 hiberfil.sys 载入内存，系统恢复至冬眠前的状况
   - 否则读取 boot.ini，显示启动菜单提示用户
   - 如果用户选择了非 NT-based 的系统，NTLDR 加载 boot.ini 内列出的文件，交出控制权
   - 如果选择 NT-based 的系统，NTLDR 运行 ｀ntdetect.com｀，获取电脑硬件信息
   - 运行 Ntoskrnl.exe，把上面获得的信息提交给它
{%ends%}

 
-  了解GRUB的启动流程。

{%s%}
   - BIOS加载MBR里面的GRUB，由于只有GRUB只占用446字节所以不能实现太多的功能，所以就有此阶段里面的文件来加载下一阶段的文件（/boot/grub下的文件）
   - 加载识别文件系统的程序，来识别文件系统，不加载就无法识别文件系统，进而就找不到boot目录，由于GRUB是无法识别LVM，所以不能把/boot分区设置为LVM，所以必须要把/boot单独分区
   - 开始寻找内核，启动内核
{%ends%}
 
 
- 比较NTLDR和GRUB的功能有差异。
 
{%s%} 
   - ntldr功能很少，只能引导win，只能装在硬盘；
   - grub是第三方操作系统引导器，从多种设备引导，也可以引导多种操作系统
{%ends%}

 
- 了解u-boot的功能。

{%s%} 
 u-boot 是一个主要用于嵌入式系统的引导程序，可以支持多种不同的计算机系统结构，包括 PPC、ARM、AVR32、MIPS、x86、68k、Nios 与 MicroBlaz,有丰富的设备驱动源码，如串口、以太网、SDRAM、FLASH、LCD、NVRAM、EEPROM、RTC、键盘等。u-boot不仅仅支持嵌入式Linux系统的引导，它还支持NetBSD, VxWorks, QNX, RTEMS, ARTOS, LynxOS, android嵌入式操作系统。
{%ends%}



## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？
   1. {%s%}中断：外部意外的响应；{%ends%}
   1. {%s%}异常：指令执行意外的响应；{%ends%}
   1. {%s%}系统调用：系统调用指令的响应；{%ends%}
-  中断、异常和系统调用的处理流程有什么异同？
   1. {%s%}源头、响应方式、处理机制{%ends%}
   1. {%s%}一个网的参考： http://blog.csdn.net/rainlight/article/details/636089{%ends%}
- 如何理解系统调用的同步和异步响应？

      {%s%}同步响应：有些系统调用要等系统服务完成后才返回；异步响应：有些系统调用在系统服务没完成就返回了；{%ends%}
      
- 系统调用表的内容是什么？
    - {%s%}系统调用的功能编号和入口地址{%ends%}
  
- Linux的系统调用有哪些？大致的功能分类有哪些？  (w2l1)

{%s%}
Linux的系统调用大概有250个左右，其继承了UNIX系统调用中最基本和最有用的部分，按照功能逻辑大致可以分为如下几类：

1. 进程控制：创建、终止、挂起进程等基本操作以及进程信息的查询
2. 文件系统控制：打开、关闭、读写文件等基本操作以及更改文件属性等系统操作
3. 系统控制：系统内核相关信息的查询以及一些系统级别的控制操作
4. 内存管理：分配、释放内存、地址映射、缓冲写回等内存相关的操作
5. 网络管理：域名的查询和设置以及主机名称、标识号的查询与设置
6. socket控制：建立连接、发送消息、断开连接等一系列套接字操作
7. 用户管理：用户和组相关信息的设置和查询
8. 进程间通信：进程间的消息、信号、管道、共享内存等通信操作
{%ends%}

- 以ucore lab8的answer为例，uCore的系统调用有哪些？大致的功能分类有哪些？(w2l1)

ucore lab8的answer中共有22个系统调用，大致分为如下几类
{%s%}

1. 进程管理：包括 fork/exit/wait/exec/yield/kill/getpid/sleep
2. 文件操作：包括 open/close/read/write/seek/fstat/fsync/getcwd/getdirentry/dup
3. 内存管理：pgdir命令
4. 外设输出：putc命令
{%ends%}


 
## 3.4 linux系统调用分析
-  通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(w2l1)
 
{%s%}

objdump的作用是将一个文件反汇编得到其各个程序段的汇编码，以该文件为例，使用-S参数后得到代码段.text的全部反汇编内容；

nm的作用是显示特定文件的符号信息，该程序的符号信息输出包括 _start 的地址 0x080482f0 ， SYS_write 的值 0x04 等等

file的作用是显示指定文件的格式，该文件输出结果为：ELF格式32位小端序可执行文件，Intel 80386架构，动态链接等

Linux系统调用主要是通过 int 0x80 实现，通过寄存器 eax , ebx , ecx , edx 传入参数来，其中 eax 代表系统调用号，而 ebx 往后均为系统调用函数的参数
{%ends%}

 
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(w2l1)
 
{%s%}


strace可以跟踪进程执行时的系统调用和所接收的信号，加上-c参数后还可以知道每个调用所花费的时间。

- 通过 Bash，调用 execve 通知操作系统执行程序
- 调用 brk(0)，开始建立程序栈基址
- 调用 access，访问一些动态链接库
- 调用 open，read 和 close，读入文件
- 通过 arch_prctl 设定程序运行架构
- 使用 mprotect 设置内存保护属性
- 调用 write，将 Hello World 输出
- exit_group 退出程序
- 期间有多次调用 mmap，进行内存虚实转化
{%ends%}


 
## 3.5 ucore系统调用分析
-  基于实验八的代码分析ucore的系统调用实现，说明25个指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。
   1. [实验八系统调用编号头文件](https://github.com/chyyuu/ucore_os_lab/blob/master/labcodes_answer/lab8_result/libs/unistd.h)

ANSWER:

在 ucore 中，执行系统调用前，需要将系统调用的参数出存在寄存器中。eax 表示系统调用类型，其余参数依次存在 edx, ecx, ebx, edi, esi 中。

```c
	...
    int num = tf->tf_regs.reg_eax;
    ... 
    arg[0] = tf->tf_regs.reg_edx; 
    arg[1] = tf->tf_regs.reg_ecx;
    arg[2] = tf->tf_regs.reg_ebx;
    arg[3] = tf->tf_regs.reg_edi;
    arg[4] = tf->tf_regs.reg_esi;
    ...
```


 1. 以getpid为例，分析ucore的系统调用中返回结果的传递代码。

`syscalls` 是保存了各种系统调用的函数指针，进入 `sys_getpid`，直接返回当前进程的 pid。

```c
	sys_getpid(uint32_t arg[]) {
	    return current->pid;
	}
```

而各类系统调用返回结果统一存在eax中。

```c
	tf->tf_regs.reg_eax = syscalls[num](arg);
```



- 以ucore lab8的answer为例，分析ucore 应用的系统调用编写和含义。

ANSWER:

 
```c
syscall(int num, ...) {
    va_list ap;
    va_start(ap, num);
    uint32_t a[MAX_ARGS];
    int i, ret;
    for (i = 0; i < MAX_ARGS; i ++) {
        a[i] = va_arg(ap, uint32_t);
    }
    va_end(ap);

    asm volatile (
        "int %1;"
        : "=a" (ret)
        : "i" (T_SYSCALL),
          "a" (num),
          "d" (a[0]),
          "c" (a[1]),
          "b" (a[2]),
          "D" (a[3]),
          "S" (a[4])
        : "cc", "memory");
    return ret;
}
```

这段代码是用户态的syscall函数，其中num参数为系统调用号，该函数将参数准备好后，通过 `SYSCALL` 汇编指令进行系统调用，进入内核态，返回值放在 `eax` 寄存器，传入参数通过 `eax` ~ `esi` 依次传递进去。在内核态中，首先进入 `trap()` 函数，然后调用 ` trap_dispatch()` 进入中断分发，当系统得知该中断为系统调用后，OS调用如下的 `syscall` 函数

```c
void
syscall(void) {
    struct trapframe *tf = current->tf;
    uint32_t arg[5];
    int num = tf->tf_regs.reg_eax;
    if (num >= 0 && num < NUM_SYSCALLS) {
        if (syscalls[num] != NULL) {
            arg[0] = tf->tf_regs.reg_edx;
            arg[1] = tf->tf_regs.reg_ecx;
            arg[2] = tf->tf_regs.reg_ebx;
            arg[3] = tf->tf_regs.reg_edi;
            arg[4] = tf->tf_regs.reg_esi;
            tf->tf_regs.reg_eax = syscalls[num](arg);
            return ;
        }
    }
    print_trapframe(tf);
    panic("undefined syscall %d, pid = %d, name = %s.\n",
            num, current->pid, current->name);
}
```

该函数得到系统调用号 `num = tf->tf_regs.reg_eax;`，通过计算快速跳转到相应的 `sys_` 开头的函数，最终在内核态中，完成系统调用所需要的功能。
 
- 以ucore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。

 
ANSWER:
   
利用 `trap.c` 的 `trap_in_kernel()` 函数判断是否是用户态的系统调用，调用 `syscall()` 时传入此参数

```c
    case T_SYSCALL:
        syscall(trap_in_kernel(tf));
        break;
```
更改 `syscall()` 的函数原型为

```c
    void syscall(bool);
```

之后在 `syscall(bool)` 中加入输出即可
```c
    int num = tf->tf_regs.reg_eax;
    if (num >= 0 && num < NUM_SYSCALLS) {
        if (syscalls[num] != NULL) {
            arg[0] = tf->tf_regs.reg_edx;
            arg[1] = tf->tf_regs.reg_ecx;
            arg[2] = tf->tf_regs.reg_ebx;
            arg[3] = tf->tf_regs.reg_edi;
            arg[4] = tf->tf_regs.reg_esi;

	    if (!in_kernel) {
	    	cprintf("SYSCALL: %d\n", num);
	    }
            tf->tf_regs.reg_eax = syscalls[num](arg);
            return ;
        }
    }
```

下面是qemu运行的输出结果片段，可以看出在用户程序输出前调用了 `SYS_open`，输出 `sh is running` 的过程中调用了 `SYS_write`

``` 
Iter 1, No.0 philosopher_sema is thinking
kernel_execve: pid = 2, name = "sh".
SYSCALL: 100
SYSCALL: 100
SYSCALL: 103
....
```
 
## 3.6 请分析函数调用和系统调用的区别
1. 系统调用与函数调用的区别是什么？
   1. {%s%}指令不同{%ends%}
   1. {%s%}特权级不同{%ends%}
   1. {%s%}堆栈切换{%ends%}
 1. 通过分析`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较函数调用与系统调用的堆栈操作有什么不同？
   1. {%s%}SS:SP压栈{%ends%}

## v9-cpu相关题目
---

### 提前准备
```
download os_tutorial_lab
cd os_tutorial_lab/v9_computer/os_user_task_syscall
make
make run
```

### v9-cpu系统调用实现
- [os_user_task_syscall.c](https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/os_user_task_syscall/os_user_task_syscall.c)的系统调用中参数传递代码分析。

ANSWER:

用户态程序通过 `write()` 这个API函数来实现输出的操作，而这个函数是通过系统调用来实现的。

```c
  asm(LL,8); asm(LBL,16); asm(LCL,24); asm(TRAP,S_write);
```

`write` 函数将寄存器a,b,c分别设置为三个传入参数，然后触发系统调用，在 `alltraps` 函数中，做好堆栈的处理，进入中断向量 `trap` 函数，其中将这三个参数传递给 `sys_write` 这个内核函数，在内核态完成输出，最终返回用户态。

- [os_user_task_syscall.c](https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/os_user_task_syscall/os_user_task_syscall.c)的系统调用中返回结果的传递代码分析。

ANSWER:

由

```c
   case S_write: a = sys_write(a, b, c); break;
```
 可知 `sys_write`函数的返回值存放在了 `a` 这一个变量所在的位置，而这个变量在 `alltraps` 中通过倒数第二句 `POPA` 将寄存器a设置为a变量的值，即函数返回值是通过寄存器a实现的。
 
- 理解[os_user_task_syscall.c](https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/os_user_task_syscall/os_user_task_syscall.c)的的系统调用编写和含义。

ANSWER:

在程序中，用户态 `task0` 执行的时候，需要使用操作系统提供的输出接口，而这首先是通过一个用户态API函数调用 `write()` ；接着该函数进行系统调用，并传入必要的参数，进入内核态；在内核态中的中断处理程序过程中，系统得到 `write` 函数传入的参数，再通过内核态的函数 `sys_write` 来进行输出。
​    
总之，用户程序通过API函数调用、再到系统调用、再到内核态函数调用，完成了用户程序需要的输出功能。
