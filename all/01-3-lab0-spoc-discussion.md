# lec2：lab0 SPOC思考题

## **提前准备**
（请在上课前完成，option）

- 完成lec2的视频学习
- git pull ucore_os_lab, os_tutorial_lab, os_course_exercises  in github repos。这样可以在本机上完成课堂练习。
- 了解代码段，数据段，执行文件，执行文件格式，堆，栈，控制流，函数调用,函数参数传递，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在不同操作系统（如linux, ucore,etc.)与不同硬件（如 x86, riscv, v9-cpu,etc.)中是如何相互配合来体现的。
- 安装好ucore实验环境，能够编译运行ucore labs中的源码。
- 会使用linux中的shell命令:objdump，nm，file, strace，gdb等，了解这些命令的用途。
- 会编译，运行，使用v9-cpu的dis,xc, xem命令（包括启动参数），阅读v9-cpu中的v9\-computer.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
- 了解基于v9-cpu的执行文件的格式和内容，以及它是如何加载到v9-cpu的内存中的。
- 在piazza上就学习中不理解问题进行提问。

---

## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

页表、特权级、中断。需要提供软中断的指令（如 int，ecall），操作页表和 Flush TLB 的指令（lcr3、sfence.vma等）

- 你理解的x86的实模式和保护模式有什么区别？你认为从实模式切换到保护模式需要注意那些方面？

  实模式是 16 位，保护模式是 32 位，所以指令的长度、寄存器的位数等都不一样。实模式中地址通过段的方式进行计算，保护模式除了段以外还有页表。切换到保护模式，需要注意的是切换前后汇编的位数设置要不同，并且应该在切换前把相应的页表和段设置妥当，以便跳转后能继续运行 32 位代码。

- 物理地址、线性地址、逻辑地址的含义分别是什么？它们之间有什么联系？

  逻辑地址用段描述符计算后得到线性地址，线性地址用页表映射后得到物理地址。代码中访问的所有地址都是逻辑地址，而物理地址是实际上访问 RAM 的地址。

- 你理解的risc-v的特权模式有什么区别？不同模式在地址访问方面有何特征？

  有m态/s态和u态。区别就是可以访问的范围不同。u态和s态之间通过页表（Sv32等）进行特权级的区分，而m态也可以通过pmp限制u态和s态对物理地址的访问。它们都有各自的一些指令。

- 理解ucore中list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```

代表这一项的位数（bits）。8位相当于char，16位相当于short。

- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？

0b0000000000000011_0000000000000010_00000_000_1111_0_00_1_0000000000000011

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。
2. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

```asm
.macro SAVE_ALL
    # If coming from userspace, preserve the user stack pointer and load
    # the kernel stack pointer. If we came from the kernel, sscratch
    # will contain 0, and we should continue on the current stack.
    csrrw sp, (xscratch), sp
    bnez sp, _save_context
_restore_kernel_sp:
    csrr sp, (xscratch)
    # sscratch = previous-sp, sp = kernel-sp
_save_context:
    # provide room for trap frame
    addi sp, sp, -36 * XLENB
    # save x registers except x2 (sp)
    STORE x1, 1
    STORE x3, 3
    # tp(x4) = hartid. DON'T change.
    # STORE x4, 4
    STORE x5, 5
    STORE x6, 6
    STORE x7, 7
    STORE x8, 8
    STORE x9, 9
    STORE x10, 10
    STORE x11, 11
    STORE x12, 12
    STORE x13, 13
    STORE x14, 14
    STORE x15, 15
    STORE x16, 16
    STORE x17, 17
    STORE x18, 18
    STORE x19, 19
    STORE x20, 20
    STORE x21, 21
    STORE x22, 22
    STORE x23, 23
    STORE x24, 24
    STORE x25, 25
    STORE x26, 26
    STORE x27, 27
    STORE x28, 28
    STORE x29, 29
    STORE x30, 30
    STORE x31, 31

    # get sp, sstatus, sepc, stval, scause
    # set sscratch = 0
    csrrw s0, (xscratch), x0
    csrr s1, (xstatus)
    csrr s2, (xepc)
    csrr s3, (xtval)
    csrr s4, (xcause)
    # store sp, sstatus, sepc, sbadvaddr, scause
    STORE s0, 2
    STORE s1, 32
    STORE s2, 33
    STORE s3, 34
    STORE s4, 35
.endm
```

保存进程上下文，首先判断 sscratch 的内容，得知中断前特权是s还是u，然后配置内核态的sp，接着向栈上保存各个寄存器的内容，再读出几个csr：sscratch，sstatus，sepc，sbadvaddr，sscause然后也保存下来。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

上面练习一的代码段就是一段宏定义，里面又用到了 STORE 这个宏定义，这样可以减少代码重复，尤其是 rcore 为了支持不同的 riscv 平台，有一部分汇编指令不完全一样，这部分就可以通过宏来实现代码的复用。


## 问答题

#### 在配置实验环境时，你遇到了那些问题，是如何解决的。

## 参考资料
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
 - [v9 cpu architecture](https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/docs/v9_computer.md)
 - [RISC-V cpu architecture](http://www.riscvbook.com/chinese/)
 - [OS相关经典论文](https://github.com/chyyuu/aos_course_info/blob/master/readinglist.md)
