# lec1: 操作系统概述

---

## **提前准备**

（请在上课前完成）

* 完成lec1的视频学习和提交对应的在线练习
* git pull ucore\_os\_lab, ucore\_os\_docs, os\_tutorial\_lab, os\_course\_exercises in github repos。这样可以在本机上完成课堂练习。
* 知道OS课程的入口网址，会使用在线视频平台，在线练习/实验平台，在线提问平台\(piazza\)
  * [http://os.cs.tsinghua.edu.cn/oscourse/OS2019spring](http://os.cs.tsinghua.edu.cn/oscourse/OS2019spring)


* 会使用linux shell命令，如ls, rm, mkdir, cat, less, more, gcc等，也会使用linux系统的基本操作。
* 在piazza上就学习中不理解问题进行提问。



# 思考题

## 填空题

* 当前常见的操作系统主要用_C和汇编_编程语言编写。
* "Operating system"这个单词起源于_操作员 operator_ 。
* 在计算机系统中，控制和管理_软硬件资源_ 、有效地组织_应用程序_运行的系统软件称作_操作系统_ 。
* 允许多用户将若干个作业提交给计算机系统集中处理的操作系统称为_批处理_操作系统
* 你了解的当前世界上使用最多的操作系统是_Windows_ 。
* 应用程序通过_系统调用_接口获得操作系统的服务。
* 现代操作系统的特征包括_并发_ ， _共享_ ， _虚拟_ ，_异步_ 。
* 操作系统内核的架构包括_微内核_ ， _宏内核_ ， _混合内核_ 。


## 问答题

- 请总结你认为操作系统应该具有的特征有什么？并对其特征进行简要阐述。
  - 封装底层硬件的细节，提供网络、文件等功能，以 syscall 的方式提供给用户程序
  - 提供进程机制，可以创建用户进程/线程
  - 提供隔离机制，隔离用户进程和内核，隔离用户进程和用户进程
  - 提供同步互斥机制，包括线程间/进程间


- 为什么现在的操作系统基本上用C语言来实现？为什么没有人用python，java来实现操作系统？

  - C 语言比较底层，可以写跨平台的代码并且效率较高，没有运行时，和硬件较为接近
  - python 和 java 等语言有很重的运行时，并且有 gc ，这些可能增加操作系统实现的复杂度和运行时间

---

## 可选练习题

---

- 请分析并理解[v9\-computer](https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/docs/v9_computer.md)以及模拟v9\-computer的em.c。理解：在v9\-computer中如何实现时钟中断的；v9 computer的CPU指令，关键变量描述有误或不全的情况；在v9\-computer中的跳转相关操作是如何实现的；在v9\-computer中如何设计相应指令，可有效实现函数调用与返回；OS程序被加载到内存的哪个位置,其堆栈是如何设置的；在v9\-computer中如何完成一次内存地址的读写的；在v9\-computer中如何实现分页机制。


- 请编写一个小程序，在v9-cpu下，能够输出字符


- 输入的字符并输出你输入的字符


- 请编写一个小程序，在v9-cpu下，能够产生各种异常/中断


- 请编写一个小程序，在v9-cpu下，能够统计并显示内存大小



- 请分析并理解[RISC-V CPU](http://www.riscvbook.com/chinese/)以及会使用模拟RISC\-V(简称RV)的qemu工具。理解：RV的特权指令，CSR寄存器和在RV中如何实现时钟中断和IO操作；OS程序如何被加载运行的；在RV中如何实现分页机制。
  - 特权指令：一系列特殊的寄存器和指令（mret，sret，ecall等）
  - 时钟中断：通过 sbi 的 set_timer 进行
  - IO 操作：一般是 MMIO ，通过读写内存进行 IO ，串口可以通过 sbi 的两个 call
  - OS 如何被加载运行的：采用一个 bootloader ，如 bbl 把内核加载到内存后在 s 态跑内核
  - 分页机制：m 态有 PMP ，s 态有 Sv32, Sv39, Sv48 等
  - 请编写一个小程序，在RV下，能够输出字符
    - s 态中可以 sbi 调用 console_putchar 即可
    - m 态则取决于具体硬件，例如 ns16550 串口
  - 输入的字符并输出你输入的字符
    - s 态 sbi 调用 console_getchar/console_putchar 即可
    - m 态同上
  - 请编写一个小程序，在RV下，能够产生各种异常/中断
    - 产生异常： ecall ，除 0 ，访问内存错误
    - 产生中断： sbi 设置时间中断/IPI
  - 请编写一个小程序，在RV下，能够统计并显示内存大小
    - 启动时会获取到 dts 地址，解析 dts 地址，就可以知道系统的内存的分布情况
