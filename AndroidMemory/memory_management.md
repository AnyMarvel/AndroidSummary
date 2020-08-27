#内存管理基础

![](https://pic1.zhimg.com/80/8bf6ef80ddbcd9ed5984fe9a51bb5b78_720w.jpg?source=1940ef5c)

所以我们只需要了解上面主机部分的内容(上图主机部分)

##概述

CPU只能访问其寄存器（Register）和内存（Memory）， 无法直接访问硬盘（Disk）。 存储在硬盘上的数据必须首先传输到内存中才能被CPU访问。从访问速度来看，对寄存器的访问非常快，通常为1纳秒； 对内存的访问相对较慢，通常为100纳秒（使用缓存加速的情况下）；而对硬盘驱动器的访问速度最慢，通常为10毫秒。

>寄存器（Register）：CPU内部的高速存储区域

当一个程序加载到内存中时，它由四个内存区域组成：

堆栈（Stack）：存储由该程序的每个函数创建的临时变量
堆（Heap）：该区域特别适用于动态内存分配
数据（Data）：存储该程序的全局变量和静态变量
代码（Code）：存储该程序的指令

主要的内存管理技术

- Base and limit registers（基址寄存器和界限寄存器）
- Virtual memory（虚拟内存）
- Swapping（交换）
- Segmentation（分段）
- Paging（分页）

###Base and limit registers（基址寄存器和界限寄存器）

必须限制进程，以便它们只能访问属于该特定进程的内存位置。

每个进程都有一个基址寄存器和限制寄存器：

- 基址寄存器保存最小的有效存储器地址
- 限制寄存器指定范围的大小

例如，process 2的有效内存地址是300040到420940

![](https://upload-images.jianshu.io/upload_images/6549967-466df45e458f71b1.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/516/format/webp)

那么每个来自用户进程的内存访问都将首先针对这两个寄存器进行一次检查：

![](https://upload-images.jianshu.io/upload_images/6549967-cfec4c5f79771ca4.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/733/format/webp)

>操作系统内核可以访问所有内存位置，因为它需要管理整个内存。


###Virtual memory（虚拟内存）

虚拟内存（VM）是OS为内存管理提供的基本抽象。

- 所有程序都使用虚拟内存地址
- 虚拟地址会被转换为物理地址
- 物理地址表示数据的实际物理位置
- 物理位置可以是内存或磁盘

![](https://upload-images.jianshu.io/upload_images/6549967-c2d15cacda22d25b.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/672/format/webp)
![](https://upload-images.jianshu.io/upload_images/6549967-cc2b0e2e1acdfdef.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/477/format/webp)
![](https://upload-images.jianshu.io/upload_images/6549967-d3b9ba141fd5cec1.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/605/format/webp)

虚拟地址到物理地址的转换由存储器管理单元（MMU - Memory Management Unit）处理。MMU使用重定位寄存器（relocation register），其值在硬件级别上被添加到每个内存请求中。

![](https://upload-images.jianshu.io/upload_images/6549967-61a9e6be8ea382cd.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/550/format/webp)

###Swapping（交换）
交换是一种可以暂时将进程从内存交换到后备存储，而之后又可以将其返回内存以继续执行的技术。

后备存储通常是一个硬盘驱动器，其访问速度快，且大小足以存储内存映像的副本。

如果没有足够的可用内存来同时保留内存中的所有正在运行的进程，则某些当前未使用CPU的进程可能会被交换到后备存储中。

![](https://upload-images.jianshu.io/upload_images/6549967-c93a195e63813c60.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/762/format/webp)

交换是一个非常缓慢的过程。 主要耗时部分是数据传输。例如，如果进程占用10MB内存并且后备存储的传输速率为40MB/秒，则需要0.25秒来进行数据传输。 再加上将数据交换回内存的时间，总传输时间可能是半秒，这是一个巨大的延迟，因此，有些操作系统已经不再使用交换了。



###Segmentation（分段）

分段是一种将内存分解为逻辑片段的技术，其中每个片段代表一组相关信息。 例如，将每个进程按照堆栈，堆，数据以及代码分为不同的段，还有OS内核的数据段等。

将内存分解成较小的段会增加寻找空闲内存的机会。

![](https://upload-images.jianshu.io/upload_images/6549967-6ddbf62da7655b34.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/487/format/webp)

每个段都有一对寄存器：

- 基址寄存器：包含段驻留在内存中的起始物理地址
- 限制寄存器：指定段的长度

**段表（Segment table）** 存储每个段的基址和限制寄存器信息。

![](https://upload-images.jianshu.io/upload_images/6549967-bf03002fc3624e5a.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/493/format/webp)

使用分段时，虚拟内存地址是一对：<段号，偏移量>

段号（Segment Number）：用作段表的索引以查找特定条目
偏移量（Offset）：首先与限制寄存器进行比较，然后与基址结合以计算物理内存地址

![](https://upload-images.jianshu.io/upload_images/6549967-0a11546f7ed4ec37.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/961/format/webp)

###Paging（分页）

有时可用内存被分成许多小块，其中没有一块足够大以满足下一个内存需求，然而他们的总和却可以。这个问题被称为**碎片（Fragmentation）** ，许多内存分配策略都会受其影响。

分页是一种内存管理技术，它允许进程的物理内存**不连续**。它通过在称为页面（Page）的相同大小的块中分配内存来消除碎片问题，是目前比较优秀的内存管理技术。

分页将物理内存划分为多个大小相等的块，称为**帧（Frame）** 。并将进程的逻辑内存空间也划分为大小相等的块，称为**页面（Page）**。

**任何进程中的任何页面都可以放入任何可用的帧中。**

**页表（Page Table）** 用于查找此刻存储特定页面的帧。

![](https://upload-images.jianshu.io/upload_images/6549967-ed5d51147455c7b4.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/961/format/webp)

使用分页时，虚拟内存地址是一对：**<页码，偏移量>**

- 页码（Page Number）：用作页表的索引，以查找此页面的条目
- 移量（Offset）：与基址相结合，以定义物理内存地址

![](https://upload-images.jianshu.io/upload_images/6549967-fa33b90a8c2f27c9.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/965/format/webp)

举一个分页地址转换的例子：

> 虚拟内存地址为0x13325328，页表项0x13325包含的值是0x03004，那么物理地址是什么？

> 答案：
物理地址是0x03004328
页码为0x13325，偏移量为0x328
相应的帧号是0x03004
