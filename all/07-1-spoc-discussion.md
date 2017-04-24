# 同步互斥(lec 17) spoc 思考题


- 有"spoc"标记的题是要求拿清华学分的同学在实体课上完成的，对于学堂在线的选课同学是可选题目。

## 视频相关思考题

### 17.1 背景

1. 请给出程序正确性的定义或解释。

 > 程序执行的结果是实现预期的功能，并且是确定的和可重现的；

 > 我们只能在确定的限制条件下来讨论“正确性”的定义或解释，没有无条件的“正确”。

2. 在一个新运行环境中程序行为与原来的预期不一致，是错误吗？

 > 如果预期行为包括明确的运行环境的限定，这就不是错误；否则就是错误；

3. 程序并发执行有什么好处和障碍？

 > 共享资源

 > 加速处理

 > 方便管理：模块化

4. 什么是原子操作？

 > 原子操作是指一次不存在任何中断或失败的操作，不会出现部分执行的状态。

### 17.2 现实生活中的同步问题

1. 家庭采购中的同步问题的5种解决方案的核心思路是什么？举例描述可能的漏洞。

 > 思路的关键点：有人去买，不会重复买；合理的枚举分类；

 > 方案一：先检查，后加相同的标签；漏洞是可能重复采购；

 > 方案二：先加相同的标签，后检查；漏洞是会出现没人去买的情况；

 > 方案三：先加不同的标签，后检查；漏洞是会出现没人去买的情况；

 > 方案四：先加不同的标签，后进行非对称检查；经枚举判断，结果是不会出现漏洞；

 > 方案五：原子操作

2. 家庭采购中的同步问题与操作系统中进程同步有什么区别？

 > 操作系统的进程切换比人的操作切换快；

 > 操作系统的进程对外界环境的感知手段比人少；

3. 如何通过枚举和分类方法检查同步算法的正确性？

 > 合理定义枚举分类标准是检查同步算法正确性的关键；

4. 尝试描述方案四的正确性。

 > 依据谁先加标签和加第二个标签的时间进行情况划分

5. 举例说明，互斥、死锁和饥饿的定义是什么？

 > 互斥：一个进程占用资源，其它进程不能使用；

 > 死锁：多个进程各占用部分资源，形成循环等待；

 > 饥饿：其他进程可能轮流占用资源，一个进程一直得不到资源；

### 17.3 临界区，禁用硬件中断,硬件原子操作同步方法

1. 什么是临界区？

 > 操作临界资源的互斥执行代码片段

2. 临界区的访问规则是什么？

 > 空闲则入、忙则等待、有限等待、让权等待

3. 禁用中断是如何实现对临界区的访问控制的？有什么优缺点？

 > 禁用中断可以阻止其他进程对临界区访问进程的打断；

 > 优点是简单；缺点是无法中断，临界区代码故障可以导致系统崩溃，其他进程可能出现饥饿；

4. Test&Set原子操作是否可以实现Exchange原子操作? Exchange原子操作是否可以实现Test&Set原子操作? 

 > 这两条指令在实现原子操作的功能是等价的。

### 17.4 基于软件的同步方法

1. 软件同步方法中的5种解决方案（三种尝试方案、Peterson算法和Eisenberg算法）的核心思路是什么？举例描述可能的漏洞。

 > 方案一：turn 表示允许进入临界区的线程标识；
方案漏洞：交替进入临界区；

 > 方案二：flag[i] 表示线程i是否在临界区；
先判断，后修改变量；
方案漏洞：并发判断后，可能出现同时进入临界区；

 > 方案二：flag[i] 表示线程i想要进入临界区；
先修改变量，后判断；
方案漏洞：并发修改变量后，可能出现都无法进入临界区的情况；

 > Peterson算法：turn 表示进入临界区的线程标识，flag[i] 表示线程i想要进入临界区；
先修改变量，后判断；后修改者等待；
只适用于两个进程；
方案正确性枚举判断：按写变量的顺序进行情况分类

 > Eisenberg算法：flag[i] 表示线程i想要进入临界区，turn 表示进入临界区的线程标识（有多个想进入时）；
进入区：先修改flag，后判断是否有多个想进入；后修改者等待；
退出区：修改turn；
适用于多个进程；
方案正确性枚举判断：按写变量flag[i]的顺序和变量turn当前值进行情况分类

2. 尝试通过枚举和分类方法检查Peterson算法的正确性。

3. 尝试准确描述Eisenberg同步算法，并通过枚举和分类方法检查其正确性。

4.下列二线程同步机制是否有误？请给出分析．

```
CONCEPT: A shared variable named turn is used to keep track of whose turn it is to enter the critical section.
INITIALIZATION:

	shared int turn;
	...
	turn = i ;
ENTRY PROTOCOL (for Process i ):
	/* wait until it's our turn */
	while (turn != i ) {
	}
EXIT PROTOCOL (for Process i ):
	/* pass the turn on */
	turn = j ;
```


5.下列二线程同步机制是否有误？请给出分析．

```
CONCEPT: A shared Boolean array named flags contains a flag for each process. The flag values are BUSY when the process is in its critical section (using the resource), or FREE when it is not.
INITIALIZATION:

	typedef char boolean;
	...
	shared boolean flags[n - 1];
	...
	flags[i ] = FREE;
	...
	flags[j ] = FREE;
	...
ENTRY PROTOCOL (for Process i ):
	/* wait while the other process is in its CS */
	while (flags[j ] == BUSY) {
	}
-->
	/* claim the resource */
	flags[i ] = BUSY;
EXIT PROTOCOL (for Process i ):
	/* release the resource */
	flags[i ] = FREE;

```

6.下列二线程同步机制是否有误？请给出分析．

```
CONCEPT: Again we use a shared Boolean array as in Algorithm 2. Each process sets its flag before  testing the other flag, thus avoiding the problem of violating mutual exclusion.
INITIALIZATION:

	typedef char boolean;
	...
	shared boolean flags[n -1];
	...
	flags[i ] = FREE;
	...
	flags[j ] = FREE;
	...
ENTRY PROTOCOL (for Process i ):
	/* claim the resource */
	flags[i ] = BUSY;
-->
	/* wait if the other process is using the resource */
	while (flags[j ] == BUSY) {
	}
EXIT PROTOCOL (for Process i ):
	/* release the resource */
	flags[i ] = FREE;
```

7.下列二线程同步机制是否有误？请给出分析．

```
CONCEPT: To avoid the deadlock problem of Algorithm 3, we periodically clear and reset our own flag while waiting for the other one.
INITIALIZATION:

	typedef char boolean;
	...
	shared boolean flags[n -1];
	...
	flags[i ] = FREE;
	...
	flags[j ] = FREE;
	...
ENTRY PROTOCOL (for Process i ):
	/* claim the resource */
	flags[i ] = BUSY;
-->
	/* wait if the other process is using the resource */
	while (flags[j ] == BUSY) {
		flags[i ] = FREE;
		delay a while ;
		flags[i ] = BUSY;
	}
EXIT PROTOCOL (for Process i ):
	/* release the resource */
	flags[i ] = FREE;

```

8.下列二线程同步机制是否有误？请给出分析．

```
CONCEPT: Both the turn variable and the status flags are combined in a way which we (the requesting process) set our flag and then check our neighbor's flag. 

INITIALIZATION:

	typedef char boolean;
	...
	shared boolean flags[n -1];
	shared int turn;
	...
	turn = i ;
	...
	flags[i ] = FREE;
	...
	flags[j ] = FREE;
	...
ENTRY PROTOCOL (for Process i ):
	/* claim the resource */
	flags[i ] = BUSY;

	/* wait if the other process is using the resource */
	while (flags[j ] == BUSY) {

		/* if waiting for the resource, also wait our turn */
		if (turn != i ) {
		
			/* but release the resource while waiting */
			flags[i ] = FREE;
			while (turn != i ) {
			}
			flags[i ] = BUSY;
		}

	}
EXIT PROTOCOL (for Process i ):
	/* pass the turn on, and release the resource */
	turn = j ;
	flags[i ] = FREE;

```

9.下列二线程同步机制是否有误？请给出分析． 

```
CONCEPT: Both the turn variable and the status flags are used.

INITIALIZATION:

	typedef char boolean;
	...
	shared boolean flags[n -1];
	shared int turn;
	...
	turn = i ;
	...
	flags[i ] = FREE;
	...
	flags[j ] = FREE;
	...
ENTRY PROTOCOL (for Process i ):
	/* claim the resource */
	flags[i ] = BUSY;

	/* give away the turn */
	turn = j ;
	/* wait while the other process is using the resource *and* has the turn */
	while ((flags[j ] == BUSY) && (turn != i )) {
	}
EXIT PROTOCOL (for Process i ):
	/* release the resource */
	flags[i ] = FREE;

```

10.下列N线程同步机制是否有误？请给出分析． 

```
CONCEPT: The turn variable and status flags are used as in Dekker's algorithm for the 2-process case. The flags now have three possible values: WAITING for a process in the entry protocol, waiting for the resource' ACTIVE for a process in the critical section, using the resource; and IDLE for other cases.

Process priority is maintained in circular order beginning with the one holding the turn. Each process begins the entry protocol by scanning all processes from the one with the turn up to itself. These are the only processes that might have to go first if there is competition.

If the scan finds all processes idle, the process advances tentatively to the ACTIVE state. However, it is still possible that another process which started scanning later but belongs before us will also reach this state. We check one more time to be sure there are no active processes.

INITIALIZATION:

	shared enum states {IDLE, WAITING, ACTIVE} flags[n -1];
	shared int turn;
	int index;	/* not shared! */
	...
	turn = 0;
	...
	for (index=0; index<n; index++) {
		flags[index] = IDLE;
	}
ENTRY PROTOCOL (for Process i ):

	repeat {

		/* announce that we need the resource */
		flags[i] = WAITING;

		/* scan processes from the one with the turn up to ourselves. */
		/* repeat if necessary until the scan finds all processes idle */
		index = turn;
		while (index != i) {
			if (flag[index] != IDLE) index = turn;
			else index = index+1 mod n;
		}

		/* now tentatively claim the resource */
		flags[i] = ACTIVE;

		/* find the first active process besides ourselves, if any */
		index = 0;
		while ((index < n) && ((index == i) || (flags[index] != ACTIVE))) {
			index = index+1;
		}

	/* if there were no other active processes, AND if we have the turn
	   or else whoever has it is idle, then proceed.  Otherwise, repeat
	   the whole sequence. */
	} until ((index >= n) && ((turn == i) || (flags[turn] == IDLE)));

	/* claim the turn and proceed */
	turn = i;
EXIT PROTOCOL (for Process i ):

	/* find a process which is not IDLE */
	/* (if there are no others, we will find ourselves) */
	index = turn+1 mod n;
	while (flags[index] == IDLE) {
		index = index+1 mod n;
	}

	/* give the turn to someone that needs it, or keep it */
	turn = index;

	/* we're finished now */
	flag[i] = IDLE;

```


11.下列N线程同步机制是否有误？请给出分析． 

```
CONCEPT: Both status values and turn values are used. The status array is expanded to an integer value for each process, which is used to track that process' progress in scanning the status of other processes. The turn value is also expanded to an integer array. Its values represent the relative ordering for each pair of processes.


INITIALIZATION:

shared int flags[NUMPROCS];
shared int turn[NUMPROCS - 1];
int index;

for (index = 0; index < (NUMPROCS); index++) {

	flags[index] = -1
}


for (index = 0; index < (NUMPROCS-1); index++) {
	turn[index] = 0;
}

ENTRY PROTOCOL (for Process i):
/* repeat for all partners */
for (count = 0; count < (NUMPROCS-1); count++) {

	flags[i] = count;
	turn[count] = i;

	"wait until (for all k != i, flag[k]<count) or (turn[count] != i)"

}

EXIT PROTOCOL (for Process i):
/* tell everyone we are finished */
flags[i] = -1;
```

12.下列N线程同步机制是否有误？请给出分析． 

```
CONCEPT: A process waiting to enter its critical section chooses a number. This number must be greater than all other numbers currently in use. There is a global shared array of current numbers for each process. The entering process checks all other processes sequentially, and waits for each one which has a lower number. Ties are possible; these are resolved using process IDs.

INITIALIZATION:

	typedef char boolean;
	...
	shared boolean choosing[n]
	shared int num[n];
	...
	for (j=0; j < n; j++) {
		num[j] = 0;
	}
	...
ENTRY PROTOCOL (for Process i):
	/* choose a number */
	choosing[i] = TRUE;
	num[i] = max(num[0], ..., num[n-1]) + 1;
	choosing[i] = FALSE;
	
	/* for all other processes */
	for (j=0; j < n; j++) {
	
		/* wait if the process is currently choosing */
		while (choosing[j]) {}
		
		/* wait if the process has a number and comes ahead of us */
		if ((num[j] > 0) &&
		  ((num[j] < num[i]) ||
		  (num[j] == num[i]) && (j < i))) {
			while (num[j] > 0) {}
		}
	}
		
EXIT PROTOCOL (for Process i):
	/* clear our number */
	num[i] = 0;
```

13.(spoc)基于“python, ruby, C, C++，LISP、JavaScript”等语言模拟实现Eisenberg同步算法，并给出覆盖所有枚举分类的测试用例，在实现报告写出设计思路和测试结果分析。

### 17.5 高级抽象的同步方法

1. 如何证明TS指令和交换指令的等价性？

 > 利用一条指令来实现另一条指令的功能；

2. 自旋锁（spinlock）和无忙等待锁是如何实现同步的？它们有什么不同？

 > 自旋锁是基于TS指令实现同步的，进入区的等待是占用CPU的；

 > 无忙等待锁是在自旋锁的基础上加一个等待队列和进程切换，进入区的等待是不占用CPU的；

3. 为什么硬件原子操作指令能简化同步算法的实现？

 > 原子操作指令是硬件实现的，与进程数目无关；

 > 缺点是，等待时占用CPU，可能出现饥饿和死锁；

## 小组思考题

1. （spoc）阅读[简化x86计算机模拟器的使用说明](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab7/lab7-spoc-exercise.md)，理解基于简化x86计算机的汇编代码。

2. （spoc)了解race condition. 进入[race-condition代码目录](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab7/race-condition)。

 - 执行 `./x86.py -p loop.s -t 1 -i 100 -R dx`， 请问`dx`的值是什么？
 - 执行 `./x86.py -p loop.s -t 2 -i 100 -a dx=3,dx=3 -R dx` ， 请问`dx`的值是什么？
 - 执行 `./x86.py -p loop.s -t 2 -i 3 -r -a dx=3,dx=3 -R dx`， 请问`dx`的值是什么？
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 1 -M 2000`, 请问变量x的值是什么？
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 2 -a bx=3 -M 2000`, 请问变量x的值是什么？为何每个线程要循环3次？
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 0`， 请问变量x的值是什么？
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 1`， 请问变量x的值是什么？
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -t 2 -M 2000 -i 4 -r -s 2`， 请问变量x的值是什么？ 
 - 变量x的内存地址为2000, `./x86.py -p looping-race-nolock.s -a bx=1 -t 2 -M 2000 -i 1`， 请问变量x的值是什么？ 

3. （spoc） 了解software-based lock, hardware-based lock, [software-hardware-lock代码目录](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab7/software-hardware-locks)

  - 理解flag.s,peterson.s,test-and-set.s,ticket.s,test-and-test-and-set.s 请通过x86.py分析这些代码是否实现了锁机制？请给出你的实验过程和结论说明。能否设计新的硬件原子操作指令Compare-And-Swap,Fetch-And-Add？
```
Compare-And-Swap

int CompareAndSwap(int *ptr, int expected, int new) {
  int actual = *ptr;
  if (actual == expected)
    *ptr = new;
  return actual;
}
```

```
Fetch-And-Add

int FetchAndAdd(int *ptr) {
  int old = *ptr;
  *ptr = old + 1;
  return old;
}
```
