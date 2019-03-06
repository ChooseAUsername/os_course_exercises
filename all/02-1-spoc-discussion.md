# lec 3 SPOC Discussion

## **提前准备**
（请在上课前完成）


 - 完成lec3的视频学习和提交对应的在线练习
 - git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises  　in github repos。这样可以在本机上完成课堂练习。
 - 仔细观察自己使用的计算机的启动过程和linux/ucore操作系统运行后的情况。搜索“80386　开机　启动”
 - 了解控制流，异常控制流，函数调用,中断，异常(故障)，系统调用（陷阱）,切换，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在linux, ucore, v9-cpu中的os*.c中是如何具体体现的。
 - 思考为什么操作系统需要处理中断，异常，系统调用。这些是必须要有的吗？有哪些好处？有哪些不好的地方？
 - 了解在PC机上有啥中断和异常。搜索“80386　中断　异常”
 - 安装好ucore实验环境，能够编译运行lab8的answer
 - 了解Linux和ucore有哪些系统调用。搜索“linux 系统调用", 搜索lab8中的syscall关键字相关内容。在linux下执行命令: ```man syscalls```
 - 会使用linux中的命令:objdump，nm，file, strace，man, 了解这些命令的用途。
 - 了解如何OS是如何实现中断，异常，或系统调用的。会使用v9-cpu的dis,xc, xem命令（包括启动参数），分析v9-cpu中的os0.c, os2.c，了解与异常，中断，系统调用相关的os设计实现。阅读v9-cpu中的cpu.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
 - 在piazza上就lec3学习中不理解问题进行提问。

## 第三讲 启动、中断、异常和系统调用-思考题

## 3.1 BIOS
-  请描述在“计算机组成原理课”上，同学们做的MIPS CPU是从按复位键开始到可以接收按键输入之间的启动过程。

在我们组设计的CPU中，按下复位键后，将会将所有寄存器全部清零、分支预测缓存清零并设为无效状态、VGA模块的显示缓存、MMU模块的缓存全部清零，此时PC被置为0，即为监控程序存储区域，因此从内存中地址为0处取指并开始执行监控程序。

-  x86中BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？

主引导记录。因为操作系统不一定存在于磁盘的第一个扇区，BIOS无法得知如何读入操作系统内核。（文件系统未建立，BIOS不知道磁盘里是什么；磁盘可以有多个分区，BIOS不知道由哪个分区启动。）

-  比较UEFI和BIOS的区别。

UEFI是一个接口标准，提供在所有平台上一致的操作系统启动服务；UEFI支持更大容量的硬盘；UEFI启动配置更灵活；UEFI安全性跟强。

-  理解rcore中的Berkeley BootLoader (BBL)的功能。

BBL主要部分代码如下：

```c
void boot_loader(uintptr_t dtb)
{
  filter_dtb(dtb);
#ifdef PK_ENABLE_LOGO
  print_logo();
#endif
#ifdef PK_PRINT_DEVICE_TREE
  fdt_print(dtb_output());
#endif
  mb();
  /* Use optional FDT preloaded external payload if present */
  entry_point = kernel_start ? kernel_start : &_payload_start;
#ifndef BBL_BOOT_MACHINE
#if __riscv_xlen == 64
#ifdef BBL_SV39
  setup_page_table_sv39();
#else
  setup_page_table_sv48();
#endif
  entry_point += 0xffffffff40000000;
#else
  setup_page_table_sv32();
  entry_point += 0x40000000;
#endif
#endif
  boot_other_hart(0);
}
```

功能包括过滤DTB、打印FDT信息、建立页表、加载内核等。

## 3.2 系统启动流程

- x86中分区引导扇区的结束标志是什么？

0x55AA

- x86中在UEFI中的可信启动有什么作用？

通过启动前的数字签名检查来保证启动介质的安全性。

- RV中BBL的启动过程大致包括哪些内容？

由代码可知过程包括过滤DTB、建立页表、进入machine mode或supervisor mode等过程。

## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？

中断：外设请求服务

异常：应用程序意想不到的行为

系统调用：应用程序请求操作提供服务

- 中断、异常和系统调用的处理流程有什么异同？

异：源头不同，中断来自外设、异常和系统调用来自应用程序；响应方式不同，中断异步、异常同步、系统调用同步和异步均可；处理机制不同，中断对应用程序透明，异常发生时杀死或重新执行意想不到的应用程序指令，系统调用等待和持续

同：都会保存现场，切换进内核态，启动相应的服务例程

- 以ucore/rcore lab8的answer为例，ucore的系统调用有哪些？大致的功能分类有哪些？

系统调用如下：

```c
    [SYS_exit]              sys_exit,
    [SYS_fork]              sys_fork,
    [SYS_wait]              sys_wait,
    [SYS_exec]              sys_exec,
    [SYS_yield]             sys_yield,
    [SYS_kill]              sys_kill,
    [SYS_getpid]            sys_getpid,
    [SYS_putc]              sys_putc,
    [SYS_pgdir]             sys_pgdir,
    [SYS_gettime]           sys_gettime,
    [SYS_lab6_set_priority] sys_lab6_set_priority,
    [SYS_sleep]             sys_sleep,
    [SYS_open]              sys_open,
    [SYS_close]             sys_close,
    [SYS_read]              sys_read,
    [SYS_write]             sys_write,
    [SYS_seek]              sys_seek,
    [SYS_fstat]             sys_fstat,
    [SYS_fsync]             sys_fsync,
    [SYS_getcwd]            sys_getcwd,
    [SYS_getdirentry]       sys_getdirentry,
    [SYS_dup]               sys_dup,
```

分为进程管理、文件操作、内存管理、外设输出类

## 3.4 linux系统调用分析
- 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(仅实践，不用回答)
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(仅实践，不用回答)


## 3.5 ucore/rcore系统调用分析 （扩展练习，可选）
-  基于实验八的代码分析ucore的系统调用实现，说明指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。
- 以ucore/rcore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
- 以ucore/rcore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。


## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？

系统调用时发生堆栈切换和特权级的切换，而函数调用中不存在；系统调用开销超过函数调用。

系统调用使用INT和IRET指令，函数调用使用CALL和RET指令。系统调用更安全。

- 通过分析x86中函数调用规范以及`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？

函数调用发生时，堆栈不切换，call和ret指令直接在当前堆栈上操作；系统调用发生时，假如处于内核态则无需切换堆栈，直接在当前堆栈上操作，假如处于用户态，则切换到内核态、切换堆栈并在栈上进行相应操作。

- 通过分析RV中函数调用规范以及`ecall`、`eret`、`jal`和`jalr`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？

RV函数调用与x86类似，堆栈不切换。系统调用通过ABI实现，与x86类似，需要进行堆栈和特权级切换，切换时会将相关的寄存器值保存在栈中，内核态下可以进行访问。


## 课堂实践 （在课堂上根据老师安排完成，课后不用做）
### 练习一
通过静态代码分析，举例描述ucore/rcore键盘输入中断的响应过程。

### 练习二
通过静态代码分析，举例描述ucore/rcore系统调用过程，及调用参数和返回值的传递方法。