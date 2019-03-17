# lec6 SPOC思考题


NOTICE
- 有"w3l2"标记的题是助教要提交到学堂在线上的。
- 有"w3l2"和"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的git repo上。
- 有"hard"标记的题有一定难度，鼓励实现。
- 有"easy"标记的题很容易实现，鼓励实现。
- 有"midd"标记的题是一般水平，鼓励实现。

## 与视频相关思考题

### 6.1	非连续内存分配的需求背景
  1. 为什么要设计非连续内存分配机制？

为了防止内存分配中出现碎片问题、更灵活高效地利用内存，方便程序之间共享内存数据。


  1. 非连续内存分配中内存分块大小有哪些可能的选择？大小与大小是否可变?

可选择各种大小，大块方便内存管理，小块分配过程更加灵活。大小可变。


  1. 为什么在大块时要设计大小可变，而在小块时要设计成固定大小？小块时的固定大小可以提供多种选择吗？

大块时设计可变大小可使内存分配更加灵活、减少内存空间的浪费；小块时设计为固定大小可简化内存的管理、提高效率。可提供多种选择，这样可更高效地利用内存。

### 6.2	段式存储管理
  1. 什么是段、段基址和段内偏移？

段表示访问方式和存储数据等属性相同的一段地址空间。段基址是段的起始地址，段内偏移是一个数据在相应段内相对段基址的偏移量。


  1. 段式存储管理机制的地址转换流程是什么？为什么在段式存储管理中，各段的存储位置可以不连续？这种做法有什么好处和麻烦？

物理地址为段基址加段内偏移。段与段之间是分离的，不能连续访问，因此可以不连续存储。好处在于方便内存管理、提高数据安全性，缺点在于需要进行地址转换。


### 6.3	页式存储管理
 1. 什么是页（page）、帧（frame）、页表（page table）、存储管理单元（MMU）、快表（TLB, Translation Lookaside Buffer）和高速缓存（cache）？

    页是将逻辑地址划分成的相同大小的基本分配空间，页帧是将物理地址分配成的相同大小的基本分配空间，页表是逻辑地址到物理地址转换的关系表，MMU是处理CPU对内存访问的硬件，TLB是CPU内对页表的缓存，cache是CPU对内存的缓存。

 1. 页式存储管理机制的地址转换流程是什么？为什么在页式存储管理中，各页的存储位置可以不连续？这种做法有什么好处和麻烦？

物理地址是页号加页内偏移。连续的逻辑页号不一定对应连续的物理页号，因此各页储存位置可以不连续。好处在于方便内存管理、方便页面共享，缺点在于访问页表带来性能损失。


### 6.4	页表概述
 1. 每个页表项有些什么内容？有哪些标志位？它们起什么作用？

包含帧号、存在位、修改位、引用位。帧号为该页的物理帧号，存在位表示该页是否可用，修改位表示该页是否已被修改，引用位表示该页是否被别的进程引用。

 1. 页表大小受哪些因素影响？

地址空间大小、页大小、进程数。


### 6.5	快表和多级页表
 1. 快表（TLB）与高速缓存（cache）有什么不同？

快表储存页表项，高速缓存储存内存中之前访问的内容。cache对软件透明，TLB可能对操作系统不透明。

 1. 为什么快表中查找物理地址的速度非常快？它是如何实现的？为什么它的的容量很小？

因为TLB使用硬件实现。通过物理电路实现。因为受到芯片面积、功耗、成本等因素的影响。

 1. 什么是多级页表？多级页表中的地址转换流程是什么？多级页表有什么好处和麻烦？

多级页表是通过间接引用将页号分为多级，从而建立多级的页表。查询前一级的页号、找到相应的页表项，并找到下一级相应的页表，依次查找直到找到物理地址。好处在于减小了页表的大小，缺点在于查找过程较为复杂、耗时较长。


### 6.6	反置页表
 1. 页寄存器机制的地址转换流程是什么？

根据逻辑地址中的逻辑页号，在页寄存器中搜索相应的物理帧号，并与页偏移量组成物理地址。将逻辑地址进行hash。然后查找页寄存器。

 1. 反置页表机制的地址转换流程是什么？

将运行进程的pid与逻辑页号组合进行hash，然后在反置页表中查询相应的页表项，比较pid和虚拟页号是否相同并找到相应的物理帧号。

 1. 反置页表项有些什么内容？

pid、虚拟页号、下一条hash值相同的页表项的索引。标志位

### 6.7	段页式存储管理
 1. 段页式存储管理机制的地址转换流程是什么？这种做法有什么好处和麻烦？

先根据段号找到相应段表项转换地址、在根据页号查找页表转换地址。好处在于同时利用到了两种机制的优点，缺点在于转换机制更加复杂、开销高。

 1. 如何实现基于段式存储管理的内存共享？

通过段表指向相同的页表基址，实现进程间的段共享。

 1. 如何实现基于页式存储管理的内存共享？

通过页表指向相同的物理帧，实现内存共享。

## 个人思考题
（1） (w3l2) 请简要分析64bit CPU体系结构下的分页机制是如何实现的

x64中只有48位虚拟地址，页大小为4KB，分为四级页表，每级页表寻址长度为9位。

## 小组思考题
（1）(spoc) 某系统使用请求分页存储管理，若页在内存中，满足一个内存请求需要150ns (10^-9s)。若缺页率是10%，为使有效访问时间达到0.5us(10^-6s),求不在内存的页面的平均访问时间。请给出计算步骤。

$90\% \times 150 + 10\% \times x = 500 $，得到$x=3650ns$。

（2）(spoc) 有一台假想\的计算机，页大小（page size）为32 Bytes，支持32KB的虚拟地址空间（virtual address space）,有4KB的物理内存空间（physical memory），采用二级页表，一个页目录项（page directory entry ，PDE）大小为1 Byte,一个页表项（page-table entries
PTEs）大小为1 Byte，1个页目录表大小为32 Bytes，1个页表大小为32 Bytes。页目录基址寄存器（page directory base register，PDBR）保存了页目录表的物理地址（按页对齐）。

PTE格式（8 bit） :
```
  VALID | PFN6 ... PFN0
```
PDE格式（8 bit） :
```
  VALID | PT6 ... PT0
```
其
```
VALID==1表示，表示映射存在；VALID==0表示，表示映射不存在。
PFN6..0:页帧号
PT6..0:页表的物理基址>>5
```
在[物理内存模拟数据文件](./03-2-spoc-testdata.md)中，给出了4KB物理内存空间的值，请回答下列虚地址是否有合法对应的物理内存，请给出对应的pde index, pde contents, pte index, pte contents。
```
1) Virtual Address 6c74
   Virtual Address 6b22
2) Virtual Address 03df
   Virtual Address 69dc
3) Virtual Address 317a
   Virtual Address 4546
4) Virtual Address 2c03
   Virtual Address 7fd7
5) Virtual Address 390e
   Virtual Address 748b
```

比如答案可以如下表示： (注意：下面的结果是错的，你需要关注的是如何表示)
```
Virtual Address 7570:
  --> pde index:0x1d  pde contents:(valid 1, pfn 0x33)
    --> pte index:0xb  pte contents:(valid 0, pfn 0x7f)
      --> Fault (page table entry not valid)

Virtual Address 21e1:
  --> pde index:0x8  pde contents:(valid 0, pfn 0x7f)
      --> Fault (page directory entry not valid)

Virtual Address 7268:
  --> pde index:0x1c  pde contents:(valid 1, pfn 0x5e)
    --> pte index:0x13  pte contents:(valid 1, pfn 0x65)
      --> Translates to Physical Address 0xca8 --> Value: 16
```

[链接](https://piazza.com/class/i5j09fnsl7k5x0?cid=664)有上面链接的参考答案。请比较你的结果与参考答案是否一致。如果不一致，请说明原因。

（3）请基于你对原理课二级页表的理解，并参考Lab2建页表的过程，设计一个应用程序（可基于python、ruby、C、C++、LISP、JavaScript等）可模拟实现(2)题中描述的抽象OS，可正确完成二级页表转换。

[链接](https://piazza.com/class/i5j09fnsl7k5x0?cid=664)有上面链接的参考答案。请比较你的结果与参考答案是否一致。如果不一致，提交你的实现，并说明区别。

（4）假设你有一台支持[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)的机器，请问你如何设计操作系统支持这种类型计算机？请给出设计方案。

 (5)[X86的页面结构](http://os.cs.tsinghua.edu.cn/oscourse/OS2019spring/lecture06)
---

## 扩展思考题

阅读64bit IBM Powerpc CPU架构是如何实现[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)，给出分析报告。


## interactive　understand VM

[Virtual Memory with 256 Bytes of RAM](http://blog.robertelder.org/virtual-memory-with-256-bytes-of-ram/)：这是一个只有256字节内存的一个极小计算机系统。按作者的[特征描述](https://github.com/RobertElderSoftware/recc#what-can-this-project-do)，它具备如下的功能。

 - CPU的实现代码不多于500行；
 - 支持14条指令、进程切换、虚拟存储和中断；
 - 用C实现了一个小的操作系统微内核可以在这个CPU上正常运行；
 - 实现了一个ANSI C89编译器，可生成在该CPU上运行代码；
 - 该编译器支持链接功能；
 - 用C89, Python, Java, Javascript这4种语言实现了该CPU的模拟器；
 - 支持交叉编译；
 - 所有这些只依赖标准C库。

针对op-cpu的特征描述，请同学们通过代码阅读和执行对自己有兴趣的部分进行分析，给出你的分析结果和评价。
