# 同步互斥(lec 18) spoc 思考题


- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 18.1 基本理解
1. 什么是信号量？它与软件同步方法的区别在什么地方？
2. 什么是自旋锁？它为什么无法按先来先服务方式使用资源？
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
一个线程A调用P原语时，由于线程B正在使用该信号量而进入阻塞状态；注意，这时value的值为0。
线程B放弃信号量的使用，线程A被唤醒而进入就绪状态，但没有立即进入运行状态；注意，这里value为1。
在线程A处于就绪状态时，处理机正在执行线程C的代码；线程C这时也正好调用P原语访问同一个信号量，并得到使用权。注意，这时value又变回0。
线程A进入运行状态后，重新检查value的值，条件不成立，又一次进入阻塞状态。
至此，线程C比线程A后调用P原语，但线程C比线程A先得到信号量。
```

### 18.2 信号量使用

1. 什么是条件同步？如何使用信号量来实现条件同步？
2. 什么是生产者-消费者问题？
3. 为什么在生产者-消费者问题中先申请互斥信息量会导致死锁？
4. 为何二值信号量的实现比计数信号量的实现要简单？请说明．

### 18.3 管程

1. 管程的组成包括哪几部分？入口队列和条件变量等待队列的作用是什么？
2. 为什么用管程实现的生产者-消费者问题中，可以在进入管程后才判断缓冲区的状态？
3. 请描述管程条件变量的两种释放处理方式的区别是什么？条件判断中while和if是如何影响释放处理中的顺序的？

### 18.4 哲学家就餐问题

1. 哲学家就餐问题的方案2和方案3的性能有什么区别？可以进一步提高效率吗？

### 18.5 读者-写者问题

1. 在读者-写者问题的读者优先和写者优先在行为上有什么不同？
2. 在读者-写者问题的读者优先实现中优先于读者到达的写者在什么地方等待？


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
