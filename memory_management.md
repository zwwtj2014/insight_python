# 内存管理


### 无存储器抽象存在的问题
1. 用户程序可以访问任意内存，容易破坏操作系统，造成崩溃
2. 同时运行多个程序特别困难

### 存储器抽象：地址空间
1.  **基址寄存器与界限寄存器**可以简单的动态重定位，每个内存地址送到内存之前，都会自动加上基址寄存器的内容。
2.  **交换技术**把一个进程完全调入内存，使该进程运行一段时间，然后把它存回磁盘。空闲进程主要存在磁盘上，所以当他们不运行时就不会占用内存。

### 空闲内存的管理
1. **使用位图的内存管理** 提供了一种简单的利用一块固定大小的内存区域能对内存使用情况进行记录的方法，但是问题在于当把一个占 k 个单元的进程调入内存时，存储管理器必须搜索位图，在位图中查找 k 个连续 0 的串>，这个查找过程是个耗时操作。
2. **使用链表的存储管理** 维护记录已分配内存段和空闲内存段的链表。为创建的进程分配内存时有以下算法
    *  首次适配算法
    *  下次适配算法
    *  最佳适配算法
    *  最差适配算法
    *  快速适配算法（为那些常用大小的空闲区维护单独的链表）
也可以为进程和空闲区维护各自的链表，这样能提高以上 4  个算法的速度，但是这样会增加复杂度，而且使内存释放速度变慢。
![](https://raw.githubusercontent.com/acmerfight/insight_python/master/images/system1.png)

### 虚拟内存

 1. 虚拟内存的基本思想是每个程序拥有自己的地址空间，这个空间可以被分割成多个块，每个块称为一页，每一页又连续的地址范围。这些页被映射到物理内存，但并不是所有的页都必须在内存中才能运行程序。当程序引用到一部分在物理内存中的地址空间时，由硬件立刻执行必要的映射。当程序引用到不在物理内存中的地址空间时，由操作系统负责将缺失的部分装入物理内存并重新执行失败的命令。
 2. 在使用虚拟内存的情况下，虚拟地址会被送到内存管理单元 MMU，MMU 把虚拟地址映射为物理内存地址。
 3. 虚拟内存的本质是创造一个新的抽象概念---**地址空间**，这个概念对应的是物理内存的抽象。虚拟内存的实现是将虚拟地址空间分解成页，并将每一页映射到物理内存的某个页框或者解除映射。
 4. 转换检测缓冲区（Translation Lookaside Buffer, TLB）又称为快表，提供中间由虚拟地址转换为物理地址时的缓存，可以直接将虚拟地址映射到物理地址，加速分页过程。
 5. 如何处理巨大的虚拟地址空间，
    * 多级页表，分级索引更多的地址空间
    * 倒排页表，使用页框号而不是虚拟页号来索引页表项，中间通过 TLB 加速

### 页面置换算法

 1. 最优页面置换算法，根据页面被访问前所需要的指令数作为标记，根据指令数的由多到少进行置换，这个方法对评价页面置换算法很有用，但它在实际系统中却不能使用，因为无法真正的实现。
 2. 最近未使用页面置换算法，NRU，在最近的一个时钟周期内，淘汰一个没有被访问的已修改页面，近似 LRU 算法，NRU 只是更粗略些。
 3. 先进先出的页面置换算法，FIFO，使用链表，可能抛弃重要页面。
 4. 第二次机会页面置换算法，比 FIFO 有重大改善。
 5. 时钟页面置换算法，现实。
 6. 最近最少使用页面置换算法，优秀但是难以实现。
 7. NFU（Not Frendly Used）最不常用算法。
 8. 工作集页面置换算法，设法跟踪页面的工作集（使用过的内存页面），以确保进程运行之前，他的工作集已经在内存中了，其目的是大大减少却也中断率。实现开销大。
 9. 工作集时钟页面置换算法，使用页框为元素的循环表。
 
**结合实现和理论最优，最好的两种是老化算法和工作集时钟算法，分别基于 Lru 和工作集**

### 分页系统中的设计问题

 1. **局部分分配策略与全局分配策略**，怎样在相互竞争的可运行进程之间分配内存.
    *  局部分配为每个进程分配固定的内存片段，即使有大量的空闲页框存在，工作集的增长也会颠簸。
    *  全局分配在进程间动态地分配页框，分配给各个进程的页框数是动态变化的。给每个进程分配一个最小的页框数使无论多么小的进程都可以运行，再需要更大的内存时去公共的内存池里去取。
    *  FIFO LRU 既适用于局部算法，也适用于全局算法。WSClock 工作集更适用于局部算法。
    
 2. **负载控制** ， 即使使用了最优的页面置换算法，最理想的全局分配。当进程组合的工作集超出内存容量时，就可能发生颠簸。这时只能根据进程的特性（IO 密集 or CPU 密集）将进程交换到磁盘上。
 3. **页面大小** 的确定不存在全局最优的结果，小页面减少页面内内存浪费，但是小页面，意味着更大的页表，更多的计算转换时间。现在一般的页面大小是 4KB 或 8KB
 4. 地址空间太小，所以**分离指令空间，数据空间**
 5. **共享页面**，在数据空间和指令空间分离的基础上很容易实现程序的共享，linux 采取了 copy on write 的方案，也有数据的共享。
 6. **共享库**，多个程序并发，如果有一个程序已经装在了共享库，其他程序就没有必要再进行装载，减少内存浪费。而且共享库并不会一次性的装入内存，而是根据需要以页面为单位进行装载的。共享时不使用绝对地址，使用相对偏移量的代码（位置无关代码 position-independent code）
 7. 共享库是**内存映射文件**的一种特例，核心思想是进程可以发起一个系统调用，将一个文件映射到其虚拟地址空间的一部分，在多数实现中在映射共享的页面时不会实际读入页面的内容，而是在访问时被每次一页的读入，磁盘文件被当作后背存储。当进程退出或显示的接触文件时，所有被改动的页面会被写入到文件中。
 8. **清除策略**，发生缺页中断时有大量的空闲页框，此时的分页系统在最佳状态，有一个分页守护（paging daemon）的后台进程，它在多数时候睡眠，但会被定期唤醒，如果空闲页框过少，分页守护进程通过预定的页面置换算法选择页面换出内存。
 9. **虚拟内存接口**，在一些高级系统中，程序员可以对内存映射进行控制，允许控制的原因是为了允许两个或多个进程共享一部分内存。页面共享可以用来实现高性能的消息传递系统。
 
### 分段
 1. 对于很多问题而言，有两个或多个独立的地址空间比只有一个好得多。所以有了分段，每个段构成一个独立的地址空间，段是一个逻辑实体，一个段能包括一个过程，一个数组，一个堆栈，但一般不会同时包含多种不同类型的内容。
 2. 保护程序和数据可以被划分为逻辑上独立的地址空间并且有助于共享和保护。

![](https://github.com/acmerfight/insight_python/blob/master/images/segment.png)

### FAQ

1.  为什么要有地址空间？

    首先直接把物理地址暴露给进程会带来严重问题
    1.  如果用户程序可以寻址内存的每个字节，就有很大的可能破坏操作系统，造成系统崩溃
    2.  同时运行多个程序十分困难
    地址空间创造了一个新的内存抽象，地址空间是一个进程可用于寻址内存的一套地址的集合。每个进程都有一个自己的地址空间，并且这个地址空间独立于其它进程的地址空间。使用基址寄存器和界限器可以实现。

2.  为什么要有虚拟内存？

    要运行的程序过大内存无法容纳，虚拟内存的基本思想是每个程序拥有自己的地址空间，这个空间被分割成多个块，每个块是一个页面，每一页有连续的地址范围。这些页被映射到物理内存。但并不是所有的页都必须在内存中才能运行程序。当程序引用到一部分在物理内存中的地址空间时，由硬件立刻执行必要的映射。当引用到不在物理内存中的地址空间时，由操作系统负责将缺失的部分装入物理内存并重新执行失败的命令。从某个角度讲虚拟内存时对基址寄存器和界限寄存器设计思想的综合。

### 参考资料

[虚拟内存](http://www.wikiwand.com/zh/%E8%99%9A%E6%8B%9F%E5%86%85%E5%AD%98)

《现代操作系统》
