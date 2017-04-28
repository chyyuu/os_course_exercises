# 同步互斥(lec 18) spoc 思考题

- 有"spoc"标记的题是要求拿清华学分的同学在实体课上完成的，对于学堂在线的选课同学是可选题目。

## 视频相关思考题

### 18.1 基本理解

1. 什么是信号量？它与软件同步方法的区别在什么地方？

 > 信号量是由操作系统提供的一种协调共享资源访问的方法。信号量是一种抽象数据类，由一个补保护的整形 (sem)变量和P()和V()两个原子操作组成，表示系统资源的数量。

 > 区别：

 > 软件同步是平等线程间的一种同步协商机制；

 > 信号量是由地位高于进程的管理者OS协调的同步机制；

2. 自旋锁为什么无法按先来先服务方式使用资源？

 > 原因：自旋锁是由TS指令实现的临界区申请操作，第一个检测到临界区空闲的申请者而不是第一个开始检测的申请者进入；

3. 下面是一种P操作的实现伪码。它能按FIFO顺序进行信号量申请吗？
```
 while (s.count == 0) {  //没有可用资源时，进入挂起状态；
        调用进程进入等待队列s.queue;
        阻塞调用进程;
}
s.count--;              //有可用资源，占用该资源； 
```

> 参考回答： 它的问题是，不能按FIFO进行信号量申请。

> 它的一种出错的情况
```
一个线程A调用P()原语时，由于线程B正在使用该信号量而进入阻塞状态；注意，这时value的值为0。
线程B放弃信号量的使用，线程A被唤醒而进入就绪状态，但没有立即进入运行状态；注意，这里value为1。
在线程A处于就绪状态时，处理机正在执行线程C的代码；线程C这时也正好调用P()原语访问同一个信号量，并得到使用权。注意，这时value又变回0。
线程A进入运行状态后，重新检查value的值，条件不成立，又一次进入阻塞状态。
至此，线程C比线程A后调用P原语，但线程C比线程A先得到信号量。
```

### 18.2 信号量使用

1. 什么是条件同步？如何使用信号量来实现条件同步？

 > 条件同步是指线程间的事件等待；

 > 条件同步的实现方法：

 > 定义初始值为零的信号量，事件触发进程使用V()操作表示事件出现，事件等待进程使用P()操作表示开始等待事件；

2. 什么是生产者-消费者问题？

 > 生产者生成数据，并放入缓冲区；

 > 消费者从缓冲区取出数据，进行处理；

 > 任何时间只有进程访问缓冲区；

3. 为什么在生产者-消费者问题中先申请互斥信号量会导致死锁？


 > 在两种情况下会出现循环等待：

 > 缓冲区空时，生产者等待缓冲区的互斥访问，以便放入数据；消费者占有缓冲区访问权，等待生产者放入的数据；

 > 缓冲区满时，生产者占有缓冲区访问权，等待空的缓冲块；消费者等待缓冲区的互斥访问，以便取出数据；

4. 为什么互斥信号量的实现比资源信号量的实现要简单？请说明．

 > 信号量中的整形变量的取值不同，互斥信号量的最大取值为1；而资源信号量的最大取值为资源总数；
 
### 18.3 管程

1. 管程的组成包括哪几部分？入口队列和条件变量等待队列的作用是什么？

 > 管程是一种并发程序的编程方法，由一个与入口队列对应的锁和若干个与共享数据访问的等待队列对应的条件变量组成，从而实现在任一时刻最多只有一个线程执行管程代码；

2. 管程与临界区有什么异同？

 > 相同点：在任一时刻最多只有一个线程执行管程代码或临界区代码；

 > 不同：正在管程中的线程可临时放弃管程的互斥访问，等待事件出现时恢复；而临界区不支持临时退出；

2. 为什么用管程实现的生产者-消费者问题中，可以在进入管程后才判断缓冲区的状态？

 > 管程允许临时放弃管程的互斥访问，而信号量并不支持临时放弃互斥访问权；

3. 请描述管程条件变量的三种释放处理方式的区别是什么？条件判断中while和if是如何影响释放处理中的顺序的？

 > Mesa管程：占用管程的当前进程可在任何时刻释放占用资源并唤醒相应的等待进程，当前进程继续执行，被唤醒放回入口队列队首等待当前进程释放管程访问权；

 > Hoare管程：占用管程的当前进程可在任何时刻释放占用资源并唤醒相应的等待进程，当前进程进入唤醒队列等待，被唤醒进程继续执行直到释放管程访问权；管程空闲时，优先查看唤醒队列中的等待进程，唤醒队列中没有等待进程时再查看入口队列；

 > Hansen管程：占用管程的当前进程只在退出管程时释放占用资源并唤醒相应的等待进程，被唤醒进程继续执行直到释放管程访问权；

 > 条件判断中while和if对释放处理中的执行顺序影响：

 > 在Hansen和Mesa管程中，由于条件变量释放操作signal时并没有立即放弃管程访问权，资源的可用状态可能变化，需使用while()进行重新检查；

 > 在Hoare管程中，由于条件变量释放操作signal同时表示立即放弃管程访问权，资源的可用状态保持不变，可使用if判断，不需要再次检查。
 
 > Ref: https://piazza.com/class/i5j09fnsl7k5x0?cid=894
 
 > Ref: https://www.andrew.cmu.edu/course/15-440-kesden/applications/ln/lecture6.html

### 18.4 哲学家就餐问题

1. 哲学家就餐问题的方案2和方案3的性能有什么区别？

 > 方案2对应为只有一个哲学家在吃饭；
 
 > 方案3可以有两个哲学家在吃饭

### 18.5 读者-写者问题

1. 在读者-写者问题的读者优先和写者优先在行为上有什么不同？

 > 读优先：有读者正在读，且有写者等待时，后到的读者可开始读；

 > 写优先：有读者正在读，且有写者等待时，后到的读者不能开始读；

2. 在读者-写者问题的读者优先实现中优先于读者到达的写者在什么地方等待？

 > 互斥信号WriteMutex

### 18.6 信号量，管程，条件变量 

如果能用管程实现信号量，也能用信号量实现管程，则我们可以称二者在功能上等价．

1. 请用管程with条件变量来实现信号量

2. 请用信号量来实现管程with条件变量

3. 请评价如下的实现(用信号量来实现管程with条件变量)是否合理？简要说明理由．

```
Implementing a Monitor

CONTROL VARIABLES:

	mutex: semaphore, initial value 1 (FREE)
	next: record, with 2 fields:
		next.sem: semaphore, initial value 0
		next.count: counter, initial value 0

	FOR EACH CONDITION x:
	
	x: record, with 2 fields:
		x.sem: semaphore, initial value 0
		x.count: counter, initial value 0
ENTRY PROTOCOL (at the beginning of each monitor function):

	/* wait for exclusive access to the monitor */
	P(mutex);
EXIT PROTOCOL (at the end of each monitor function):

	/* if there are processes in the "next" queue, release one */
	if (next.count > 0) V(next.sem);

	/* otherwise, release the monitor */
	else V(mutex);
WAIT ON CONDITION x (x.wait):

	/* first perform the exit protocol */
	if (next.count > 0) V(next.sem);
	else V(mutex);

	/* now wait on the condition queue */
	x.count++;
	P(x.sem);
	x.count--;
SIGNAL CONDITION x (x.signal):

	/* do nothing unless a process is waiting */
	if (x.count > 0) {

		/* release the next waiting process */
		V(x.sem);

		/* wait on the "next" queue */
		next.count++;
		P(next.sem);
		next.count--;
	}
```


4. 

## 小组思考题

1. （spoc） 每人使用C++或python语言用信号量和条件变量两种手段分别实现[40个同步互斥问题](07-2-spoc-pv-problems.md)中的一题。请先理解[python threading 机制的介绍和实例](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab7/semaphore_condition)

 > 参考：[2015年操作系统课的信号量问题回答](https://piazza.com/class/i5j09fnsl7k5x0?cid=391)
 
 > 建议参考梁锡豪同学的输出信息显示方式，这种方式的可读性很好。
 
 > 建议重视测试用例的设计，以检查自己的实现是否有错。

2. (spoc)设计某个方法，能够动态检查出对于两个或多个进程的同步互斥问题执行中，没有互斥问题，能够同步等，以说明实现的正确性。
3. (spoc challenge)管程和信号量在解决同步互斥问题上是否等价？请证明/说明你的结论．
