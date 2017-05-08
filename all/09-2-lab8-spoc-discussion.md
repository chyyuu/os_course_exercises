# lab8 文件系统 (lec 22) spoc 思考题

- 有"spoc"标记的题是要求拿清华学分的同学在实体课上完成的，对于学堂在线的选课同学是可选题目。
## 视频相关思考题

### 22.1 总体介绍

 1. 与文件系统相关的系统调用接口、虚拟文件系统VFS、简单文件系统SFS和设备I/O等四个部分各实现什么功能？

 > 系统调用接口：向应用进程提供文件访问的系统调用；

 > 虚拟文件系统VFS：屏蔽具体文件系统差异，对上提供一个统一的文件系统访问接口；

 > 简单文件系统SFS：解析和读写磁盘数据块中具体的SFS文件系统存储结构

 > 设备I/O：完成实际磁盘设备上数据块的访问；

 2. 文件系统中的文件、目录、索引节点(inode)和安装点(挂载点)这几种数据结构分别支持些什么操作？

 > 文件：open/close, read/write

 > 目录：open/close, read

 > 索引节点：lookup, read/write

 > 挂载点：mount/unmount

### 22.2 ucore 文件系统架构

 1. 请简要阐述ucore 文件系统架构的四个组成部分

 > 系统调用接口：用户应用使用封装后的libc库函数，文件访问的libc库函数利用文件访问系统调用来实现；

 > VFS：内核的系统调用（文件、目录接口）会转换成对VFS抽象的文件访问接口（索引节点、文件卷、设备等接口）的调用，VFS再把抽象的VFS接口转换成具体的文件系统SFS的访问接口；

 > SFS：利用具体文件系统存储结构的解析，把SFS对上接口（索引节点、文件卷、设备等接口）的访问请求转换成设备数据块的访问；

 > I/O接口：不同具体设备上的数据块访问控制；

 2. 请简要说明进程proc_struct、文件file、inode之间的关系。 

 > 进程控制块数据结构proc_struct中，struct files_struct *filesp指向进程的打开文件表；

 > 进程打开文件表中struct file *file指向系统打开文件中的相应文件状态数据；

 > VFS中的系统打开文件表中struct inode *inode维护打开文件的状态信息，并最终对应到磁盘上的存储数据块；

 3. ucore中的进程打开文件表和系统打开文件表对应到具体的哪个数据结构上？

 > kern-ucore/fs/vfs/inode.h

 > struct inode: union in_info

### 22.3 Simple File System分析

 1. SFS在硬盘上的四大部分主要是什么，有何作用？
 
 > superblock：数据块大小、文件卷名字等文件卷信息
 
 > root-dir inode：根目录的inode信息（存储位置等）
 
 > freemap：数据块占用状态信息
 
 > data block：目录和文件的信息

 2. 硬盘上的SFS是如何加载到ucore中并初始化的？
 
 > sfs_do_mount()

 3. 硬盘上的inode和内存中的inode的关系和区别是什么?
 
 > 内存中的inode数据结构sfs_inode中有一个字段sfs_disk_inode，它对应磁盘上的inode；

 4. 描述file, dir, inode在内存和磁盘上的格式和相关操作。
 
 > 每一种类型的数据块都在SFS层中有对应的操作函数指针和数据结构定义；

### 22.4 Virtual File System分析

 1. file数据结构的主要内容是什么？与进程的关系是什么？

 > struct file数据结构的内容：文件状态、文件操作类型、文件指针、对应系统打开文件表项指针等；

 > struct file就是进程打开文件表对应的数据结构；

 2. inode数据结构的主要内容是什么？与file的数据结构的关系是什么？

 > struct inode数据结构的内容：文件类型、引用计数、对下具体文件操作函数指针、对上inode操作函数指针

 > struct inode就是系统打开文件表对应的数据结构；

 3. inode_ops包含哪些与文件相关的操作？

 > open, close, read, write, getdirentry, create, lookup

 4. VFS是如何把键盘、显示输出和磁盘文件统一到一个系统调用访问框架下的？ 

 > VFS把键盘、显示和磁盘文件都视为文件，VFS对上提供的访问接口都是文件访问接口；

 > VFS通过区别文件类型、文件操作类型、设备类型等来区别同类操作在不同设备的不同实现；

### 22.5 I/O 设备接口分析

 1. device数据结构的主要内容是什么？与fs的关系是什么？与inode的关系是什么？

 > struct device数据结构的内容：数据块大小、设备操作函数指针（d_open, d_close, d_io, d_ioctl）

 > fs和inode通过device数据结构中的设备操作函数指针实现对设备数据块的访问；

 2. 比较ucore中I/O接口、SFS文件系统接口和文件系统的系统调用接口的操作函数有什么异同？

 > 文件系统的系统调用接口：sys_open, sys_close, sys_read, sys_write, sys_seek, sys_fstat, sys_fsync, sys_chdir, sys_getcwd, sys_mkdir, sys_link, sys_rename, sys_unlink, sys_getdirentry, sys_dup, sys_pipe, sys_mkfifo, sys_mount, sys_umount, sys_ioctl

 > VFS文件系统接口：vfs_open, vfs_close, vfs_link, vfs_symlink, vfs_readlink, vfs_mkdir, vfs_unlink, vfs_rename, vfs_chdir, vfs_getcwd

 > SFS文件系统接口：sfs_rblock, sfs_wblock, sfs_rbuf, sfs_wbuf, sfs_sync_super, sfs_sync_freemap, sfs_clear_block, sfs_load_inode

 > I/O接口：d_open, d_close, d_io, d_ioctl

### 22.6 执行流程分析

1. (spoc) 理解文件访问的执行过程，即在ucore运行过程中通过`cprintf`函数来完整地展现出来读一个文件在ucore中的整个执行过程，(越全面细致越好)
完成代码填写，并形成spoc练习报告，需写练习报告和简单编码，完成后放到git server 对应的git repo中。

## 小组思考题

2. （spoc） 在下面的实验代码的基础上，实现基于文件系统的pipe IPC机制

### 练习用的[lab8 spoc exercise project source code](https://github.com/chyyuu/ucore_lab/tree/master/labcodes_answer/lab8_result)
