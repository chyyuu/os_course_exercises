# lec5 SPOC思考题

##**提前准备**
（请在上课前完成）

- 完成lec５的视频学习和提交对应的在线练习
- git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises in github repos。这样可以在本机上完成课堂练习。
- 理解连续内存动态分配算法的实现（主要自学和网上查找）

NOTICE
- 有"w3l1"标记的题是助教要提交到学堂在线上的。
- 有"w3l1"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。


## 思考题
---

请简要分析最优匹配，最差匹配，最先匹配，buddy systemm分配算法的优势和劣势，并尝试提出一种更有效的连续内存分配算法 (w3l1)


## 小组思考题

请参考xv6（umalloc.c），ucore lab2代码，选择四种（0:最优匹配，1:最差匹配，2:最先匹配，3:buddy systemm）分配算法中的一种或多种，在Linux应用程序/库层面，用C、C++或python来实现malloc/free，给出你的设计思路，并给出可以在Linux上运行的malloc/free实现和测试用例。 (spoc)

---

## 扩展思考题

阅读[slab分配算法](http://en.wikipedia.org/wiki/Slab_allocation)，尝试在应用程序中实现slab分配算法，给出设计方案和测试用例。

## “连续内存分配”与视频相关的课堂练习

### 5.1 计算机体系结构和内存层次

1.操作系统中存储管理的目标是什么？
2.MMU的工作机理？
3.L1、L2和L3高速缓存有什么区别？


### 5.2 地址空间和地址生成
1.描述编译、汇编、链接和加载的过程是什么？
2.动态链接如何使用？尝试在Linux平台上使用LD_DEBUG查看一个C语言Hello world的启动流程。



### 5.3 连续内存分配
1.什么是内碎片、外碎片？
2.为什么最先匹配会越用越慢？
3.为什么最差匹配会的外碎片少？
4.在几种算法中分区释放后的合并处理如何做？
>  

### 5.4 碎片整理
1.对换和紧凑都是碎片整理技术，它们的主要区别是什么？为什么在早期的操作系统中采用对换技术？  
2.一个处于等待状态的进程被对换到外存（对换等待状态）后，等待事件出现了。操作系统需要如何响应？


### 5.5 伙伴系统
1.伙伴系统的空闲块如何组织？

2.伙伴系统的内存分配流程？伙伴系统的内存回收流程？

struct list_entry是如何把数据元素组织成链表的？  



## v9-cpu相关

[challenge]在v9-cpu上完成基于method-0/1/2/buddy system（选一个方法）with first/best/worst-fit(选一个fit)用户态动态内存分配函数malloc/free，要求在内核态实现sbrk系统调用以支持用户态动态内存分配方法。基于os4.c实现，可参考v9-cpu git repo的testing分支中的os.c和mem.h


[challenge]在v9-cpu上完成基GC算法，要求在内核态实现sbrk系统调用以支持用户态动态内存分配方法。基于os4.c实现，可参考v9-cpu git repo的testing分支中的os.c和mem.h

>

## 其它
- 静态分配内存与动态分配内存的区别是啥？

- 在隐式链表结构中，如何实现向前的合并操作？

- 如何判断一个函数是库函数还是一个系统调用？

- 为何要在OS内部和用户态中实现两层动态内存分配？

- 为何在OS内部没有采用GC分配方式？

## 课堂实践

观察最先匹配、最佳匹配和最差匹配这几种动态分区分配算法的工作过程，并选择一个例子进行分析分析整个工作过程中的分配和释放操作对维护数据结构的影响和原因。

  * [算法演示脚本的使用说明](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep3-malloc.md)
  * [算法演示脚本](https://github.com/chyyuu/os_tutorial_lab/blob/master/ostep/ostep3-malloc.py)

例如：
```
python ./ostep3-malloc.py -S 100 -b 1000 -H 4 -a 4 -l ADDRSORT -p BEST -n 5 -c
python ./ostep3-malloc.py -S 100 -b 1000 -H 4 -a 4 -l ADDRSORT -p FIRST -n 5 -c
python ./ostep3-malloc.py -S 100 -b 1000 -H 4 -a 4 -l ADDRSORT -p WORST -n 5 -c
```


