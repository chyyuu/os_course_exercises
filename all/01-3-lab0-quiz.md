# Lec2 在线练习

## 单选题

---

清华大学目前的操作系统实验中采用的OS对象是()

- [ ] Linux
- [ ] ucore/rcore
- [ ] xv6
- [ ] Nachos

> 是参考了xv6, OS161, Linux的教学操作系统ucore/rcore OS

在ucore lab的实验环境搭建中，使用的**非开源**软件是()

- [ ] eclipse CDT
- [ ] Scitools Understand
- [ ] gcc
- [ ] qemu

>  Scitools Understand 是非开源软件，主要可以用于分析代码，可免费试用一段时间。

在ucore/rcore lab的实验环境搭建中，用来模拟x86-32 or RISC-V 64的软件是()

- [ ] apt
- [ ] git
- [ ] meld
- [ ] qemu

> qemu是一个支持模拟多种CPU的模拟软件


---

## 是非题

---

|                  | 是 | 否 |
| ---------------- | ---- | --- |
ucore lab实验中8个实验是否可以不按顺序完成|(x)|()|

> 每个实验i依赖前面所有的实验(0～i-1)，即完成了lab i，才能完成lab i+1

|                  | 是 | 否 |
| ---------------- | ---- | --- |
ucore lab实验中在C语言中采用了面向对象的编程思想，包括函指针表和通用链表结构|(x)|()|

> 是的，这使得可编出更加灵活的操作系统功能模块和数据结构

---

## 多选题

---

x86-32 CPU（即80386）有多种运行模式，ucore lab中碰到和需要处理哪些模式()

- [x] 实模式
- [x] 保护模式
- [ ] SMM模式
- [ ] 虚拟8086模式

> ucore需要碰到和处理16位的实模式和32位的保护模式

RISC-V CPU有多种特权模式，rcore lab中碰到和需要处理哪些特权模式()

- [x] M-mode
- [x] S-mode
- [ ] H-mode
- [x] U-mode

> rcore需要碰到和处理M/S/U-mode

---

