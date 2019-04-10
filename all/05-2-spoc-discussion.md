# lec12: 进程／线程控制spoc练习

## 视频相关思考题
### 12.1 进程切换

1. 进程切换的可能时机有哪些？

> 时间片用完、被高优先级进程抢先、进入等待状态、进程结束

2. 分析ucore的进程切换代码，说明ucore的进程切换触发时机和进程切换的判断时机都有哪些。

> schedule
>
> proc_run
>
> switch_to

3. ucore的进程控制块数据结构是如何组织的？主要字段分别表示什么？

> 参见 [05-1-spoc-discussion.md](05-1-spoc-discussion.md) 中习题 11.2.3

### 12.2 进程创建

1. fork()的返回值是唯一的吗？父进程和子进程的返回值是不同的。请找到相应的赋值代码。

> 父进程中, fork 返回子进程的 pid: 见 `proc.c:do_fork()`. 注意这是 lab6 引入的代码.
>
> * 其中 `ret = proc->pid; fork_out: return ret;` 即返回子进程 pid.
>
> 子进程中 fork 返回 0: 见 `proc.c:copy_thread(struct proc_struct *dst_proc, ..., trapframe *src_tf)`. (参数名有变化)
>
> * 其中先 `*(dst_proc->tf) = *src_tf;` 完全复制一份 trap frame,
> * 然后 `dst_proc->tf->tf_regs.reg_eax = 0;` 使得在 destination trap frame 中的 eax 为 0.
> * 如果你忘记了, 请让我提醒你: x86 下面, 返回值放在 eax 中.
>   开始从子进程执行时, 从 trap frame eax 读到的就是返回值.

2. 新进程创建时的进程标识是如何设置的？请指明相关代码。

> 通过设置子进程的 PCB 中的 pid 来完成. 由父进程设置.
> 
> 见 `proc.c:do_fork()` 中 `proc->pid = get_pid();`

3. 请通过fork()的例子中进程标识的赋值顺序说明进程的执行顺序。

> 略

4. 请在ucore启动时显示空闲进程（idleproc）和初始进程（initproc）的进程标识。

> 略

5. 请在ucore启动时显示空闲线程（idleproc）和初始进程(initproc)的进程控制块中的 `pde_t *pgdir` 的内容。它们是否一致？为什么？

> 在 `proc_init()` 中加入 `cprintf("initproc->cr3: %p. idleproc->cr3: %p\n", initproc->cr3,  idleproc->cr3);` 即可.
>
> 一致的.
>
> 均为 `boot_pgdir`.
>
> 它们均共享内核地址空间.

### 12.3 进程加载

1. 加载进程后，新进程进入就绪状态，它开始执行时的第一条 **用户** 指令的位置，在elf中保存在什么地方？在加载后，保存在什么地方？

> 这里需要说明, 所谓 "新进程开始执行", 是指第一条用户指令开始执行, 也就是 `switch_to` 再之后 `iret` 之后第一条指令.
>
> 第一条用户指令保存在 elf 的 `e_entry` 中, 在 ELF 文件的第 24-27 共 4 字节. 可参考资源 `info elf`
>
> 加载后, 保存在内核分配的地址中. 参见 `proc.c:load_icode()` 中注释 3.6.1 的部分.

2. 第一个用户进程执行的代码在哪里？它是什么时候加载到内存中的？

> 第一个问题事实上问得有点没意义. 要么就在磁盘上, 要么加载后变到内存中.
>
> 被 `user_main()` 加载入内存的.

### 12.4 进程等待与退出

1. 试分析wait()和exit()的结果放在什么地方？exit()是在什么时候放进去的？wait()在什么地方取到出的？

> 略

2. 试分析ucore操作系统内核是如何把子进程exit()的返回值传递给父进程wait()的？

> 参见 `proc.c:do_wait()` 中
> ```
> if (code_store != NULL) {
>   *code_store = proc->exit_code;
> }
> ```

3. 什么是僵尸进程和孤儿进程？

> * 僵尸进程: 已经退出 (完成执行 `exit()`), 但是还没有被父进程回收 (通过 `wait()`).
>   这样的进程会占用系统资源.
>
> * 孤儿进程: 即使原来的父进程退出 (完成 `exit()`), 也保持运行的进程.
>   可以对比一般情况下, 一旦父进程退出, 所有子进程都会被操作系统回收, 无论是否完成其执行.

4. 试分析sleep()系统调用的实现。在什么地方设置的定时器？它对应的等待队列是哪个？它的唤醒操作在什么地方？

> 略.

5. 通常的函数调用和函数返回都是一一对应的。有不是一一对应的例外情况？如果有，请举例说明。

> fork()

## 小组思考题

(1) (spoc)设计一个简化的进程管理子系统，可以管理并调度支持“就绪”和“等待”状态的简化进程。给出了[参考代码](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab5/process-cpuio-homework.py)，请理解代码，并完成＂YOUR CODE"部分的内容．　可２个人一组

### 进程的状态 
```
 - RUNNING - 进程正在使用CPU
 - READY   - 进程可使用CPU
 - WAIT    - 进程等待I/O完成
 - DONE    - 进程结束
```

### 进程的行为
```
 - 使用CPU, 
 - 发出YIELD请求,放弃使用CPU
 - 发出I/O操作请求,放弃使用CPU
```

### 进程调度
 - 使用FIFO/FCFS：先来先服务, 只有进程done, yield, io时才会执行切换
   - 先查找位于proc_info队列的curr_proc元素(当前进程)之后的进程(curr_proc+1..end)是否处于READY态，
   - 再查找位于proc_info队列的curr_proc元素(当前进程)之前的进程(begin..curr_proc-1)是否处于READY态
   - 如都没有，继续执行curr_proc直到结束

### 关键模拟变量
 - io_length : IO操作的执行时间
 - 进程控制块
```
PROC_CODE = 'code_'
PROC_PC = 'pc_'
PROC_ID = 'pid_'
PROC_STATE = 'proc_state_'
```
 - 当前进程 curr_proc 
 - 进程列表：proc_info是就绪进程的队列（list），
 - 在命令行（如下所示）需要说明每进程的行为特征：（１）使用CPU ;(2)等待I/O
```
   -l PROCESS_LIST, --processlist= X1:Y1,X2:Y2,...
   X 是进程的执行指令数; 
   Ｙ是执行yield指令（进程放弃CPU,进入READY状态）的比例(0..100) 
   Ｚ是执行I/O请求指令（进程放弃CPU,进入WAIT状态）的比例(0..100)
```
 - 进程切换行为：系统决定何时(when)切换进程:进程结束或进程发出yield请求

### 进程执行
```
instruction_to_execute = self.proc_info[self.curr_proc][PROC_CODE].pop(0)
```

### 关键函数
 - 系统执行过程：run
 - 执行状态切换函数:　move_to_ready/running/done　
 - 调度函数：next_proc

### 执行实例
   
#### 例1
```
$./process-simulation.py  -l 5:30:30,5:40:30 -c
Produce a trace of what would happen when you run these processes:
Process 0
  io
  io
  yld
  cpu
  yld

Process 1
  yld
  io
  yld
  yld
  yld

Important behaviors:
  System will switch when the current process is FINISHED or ISSUES AN YIELD or IO
Time     PID: 0     PID: 1        CPU        IOs 
  1      RUN:io      READY          1            
  2     WAITING    RUN:yld          1          1 
  3     WAITING     RUN:io          1          1 
  4     WAITING    WAITING                     2 
  5     WAITING    WAITING                     2 
  6*     RUN:io    WAITING          1          1 
  7     WAITING    WAITING                     2 
  8*    WAITING    RUN:yld          1          1 
  9     WAITING    RUN:yld          1          1 
 10     WAITING    RUN:yld          1          1 
 11*    RUN:yld       DONE          1            
 12     RUN:cpu       DONE          1            
 13     RUN:yld       DONE          1            
```
