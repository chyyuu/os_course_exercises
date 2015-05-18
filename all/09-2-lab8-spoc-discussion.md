# lab8 文件系统 (lec 22) spoc 思考题


- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 总体介绍
 1. 文件系统中的文件、目录、索引节点和安装点这几种数据结构分别支持些什么操作？
 2. 请简要描述ucore文件系统支持的文件系统抽象

 > 文件、目录、索引节点和安装点

### ucore 文件系统架构

 1. 请简要阐述ucore 文件系统架构的四个组成部分

 > 系统调用接口、VFS、SFS和I/O接口

 2. 请简要说明进程proc_struct、文件file、inode之间的关系

### Simple File System分析

 - SFS在硬盘上的四大部分主要是什么，有何作用？
 - 硬盘上的SFS是如何加载到ucore中并初始化的？
 - 硬盘上的inode和内存中的inode的关系和区别是什么?


### Virtual File System分析

 - file数据结构的主要内容是什么？与进程的关系是什么？
 - inode数据结构的主要内容是什么？与file的数据结构的关系是什么？
 - inode_ops包含哪些与文件相关的操作？

### I/O 设备接口分析

 - device数据结构的主要内容是什么？与fs的关系是什么？与inode的关系是什么？
 
## 小组思考题

1. (spoc) 理解文件访问的执行过程，即在ucore运行过程中通过`cprintf`函数来完整地展现出来读一个文件在ucore中的整个执行过程，(越全面细致越好)
完成代码填写，并形成spoc练习报告，需写练习报告和简单编码，完成后放到git server 对应的git repo中

2. （spoc） 在下面的实验代码的基础上，实现基于文件系统的pipe IPC机制

### 练习用的[lab8 spoc exercise project source code](https://github.com/chyyuu/ucore_lab/tree/master/labcodes_answer/lab8_result)
