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

为支持进程需要支持时钟中断；为支持虚存需要支持MMU；为支持文件系统需要支持存储器等外设。需要支持中断使能、设置内存寻址模式、设置页表、执行IO操纵等特权指令。

- 你理解的x86的实模式和保护模式有什么区别？你认为从实模式切换到保护模式需要注意那些方面？

实模式下内存不受保护，访存指令访问的逻辑地址即为物理地址；而保护模式下物理地址不能直接被访问，需要将虚拟地址由操作系统转换为物理地址。

需要注意建立描述符表，设置控制寄存器CR0的PE位。

- 物理地址、线性地址、逻辑地址的含义分别是什么？它们之间有什么联系？

物理地址：数据在内存中的实际地址

线性地址：逻辑地址到物理地址变换之间的中间层

逻辑地址：应用程序访存指令中的虚拟地址

线性地址是逻辑地址到物理地址之间转换的中间层，访存过程中，逻辑地址可变换为线性地址，查询页表变换为物理地址，才能在硬件上真正实现访存。

- 你理解的risc-v的特权模式有什么区别？不同 模式在地址访问方面有何特征？

RISC-V的特权模式与x86特权模式区别在于有多个，包括机器模式、Hypervisor模式、管理员模式、用户模式等级别。机器级是最高级特权、支持所有操作，较低等级的模式试图执行较高等级的操作时会产生异常。

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

代表每一个字段的实际位数。

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

0x20003

### 课堂实践练习

#### 练习一

1. 请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

代码为lab8_result下的/process/switch.S。

```assembly
.text
.globl switch_to
switch_to:                      # switch_to(from, to)

    movl 4(%esp), %eax          # eax points to from
    popl 0(%eax)                # save eip !popl
    movl %esp, 4(%eax)
    movl %ebx, 8(%eax)
    movl %ecx, 12(%eax)
    movl %edx, 16(%eax)
    movl %esi, 20(%eax)
    movl %edi, 24(%eax)
    movl %ebp, 28(%eax)

    movl 4(%esp), %eax          # not 8(%esp): popped return address already
                                # eax now points to to
    movl 28(%eax), %ebp
    movl 24(%eax), %edi
    movl 20(%eax), %esi
    movl 16(%eax), %edx
    movl 12(%eax), %ecx
    movl 8(%eax), %ebx
    movl 4(%eax), %esp

    pushl 0(%eax)               # push eip

    ret
```

可以猜测到这段代码的作用是保存当前各寄存器的值，并将各寄存器恢复为过去已保存的状态，两个参数（esp+4和esp+8）分别代表要将寄存器状态保存到的位置的指针和要恢复的寄存器状态的指针。代码首先将esp+4存入eax中，然后依次将eip、esp等寄存器存入eax指向的连续32字节中。之后取出esp+8（此时为esp+4），将其指向的数据恢复到各寄存器中，最后返回。

1. (option)请在rcore中找一段你认为难度适当的RV汇编代码，尝试解释其含义。

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore或rcore中宏定义的用途，并举例描述其含义。

数据类型转换，如to_struct宏将结构体字段指针转换为结构体的指针；定义常见操作以简化代码，如ROUNDDOWN(a, n)宏将a向下取整到n的最近的倍数；定义需要使用的特定常数，如mmu.h中定义的大量寄存器标志值。

进行复杂数据结构中的数据访问。


## 问答题

#### 在配置实验环境时，你遇到了那些问题，是如何解决的。

我开始时试图使用WSL进行试验，可以完成安装qemu、完成ucore的编译，但是无法在qemu中模拟运行，提示No available video device，最终还是选择了使用虚拟机进行配置。

## 参考资料
 - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)
 - [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)
 - [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)
 - [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)
 - [IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)
 - [v9 cpu architecture](https://github.com/chyyuu/os_tutorial_lab/blob/master/v9_computer/docs/v9_computer.md)
 - [RISC-V cpu architecture](http://www.riscvbook.com/chinese/)
 - [OS相关经典论文](https://github.com/chyyuu/aos_course_info/blob/master/readinglist.md)
