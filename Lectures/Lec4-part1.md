# Page Table-PART1⃣️

> 这部分主题是页表，对于虚拟内存的其他内容会在后面详细介绍。

首先明确，虚拟内存的主要目的是实现**内存的强隔离性**。如果正确的设置page table并对其进行管控，即可实现强隔离。这个强隔离是指，每个程序的内存独立，互不影响，且用户空间的程序不会影响内核空间的程序。

## 地址空间（Address Spaces）

要实现一种机制，将不同程序内存隔离起来，一种方式就是**地址空间**。

<img src="/Users/liuwenshuo/Documents/Notes/6.s801/Lectures/image-20220415220028104.png" alt="image-20220415220028104" style="zoom:33%;" />

我们给**包括内核在内**的所有程序专属、独立的地址空间。这样如果shell向地址1000写入数据，只会写入自己的地址1000，而不是公用的地址1000。

但问题在于**如何在一个物理内存上，创建不同的地址空间**？

## 页表

最灵活的方式就是实现**页表**。页表是在硬件中通过处理器和MMU（Memory Management Unit 内存管理单元）实现。

<img src="/Users/liuwenshuo/Documents/Notes/6.s801/Lectures/image-20220415221017342.png" alt="image-20220415221017342" style="zoom: 33%;" />

实现页表后，在所有的程序中涉及到的地址，均为**虚拟地址**。例如sd $7, (a0)，其中a0中地址为0x1000，这个就是当前程序的0x1000。需要通过MMU进行虚拟地址到物理地址的映射。

注意，**MMU并不保存虚拟地址-物理地址 的映射**，这个映射保存在内存中。因此CPU需要有寄存器用来存放**当前运行的进程的页表在物理内存中的首地址**。RISC-V中的SATP寄存器就保存了这个地址的首地址。

<img src="/Users/liuwenshuo/Documents/Notes/6.s801/Lectures/image-20220416144157015.png" alt="image-20220416144157015" style="zoom: 33%;" />

当OS的CPU从一个进程切换到另一个进程时，SATP寄存器的值也要改变，即更换到不同的页表的首地址。

> 既然SATP的寄存器根据进程修改，每个进程的SATP值由谁来保存和修改呢？
>
> A：**由kernel中的程序保存和修改**，内核在切换进程时写SATP，因此写SATP是一条特殊权限指令。因此用户应用程序不能通过更新SATP寄存器来更换当前进程的页表首地址，否则隔离性就被破坏了。

细节问题没有解决：

> 页表具体如何记录？如果每个虚拟地址都会有一个物理地址对应记录，假设寄存器是64bit，那么将会有2^64条记录，页表将大到内存无法存储。
>
> A：**分块**记录，不为每个地址都创建一条记录，而是以一个较大的粒度进行一次记录。

在RISC-V中，每次地址翻译针对一个page，一个page是4KB（=2^12bit），以一个page为基本粒度进行页表记录。

那么如果按照这种模式如何设计虚拟地址到物理地址的解析呢？

对于虚拟内存地址，将其划分为两个部分，index和offset，index用来查找page首地址，offset用来查找是page中的具体哪个字节。由于一个page是4KB（2^12bit），因此offset需要12位。

在RISC-V中：

* 虚拟地址是64bit，但高25bit没有使用，只有39bit用于表示虚拟地址。因此虚拟地址空间最大为2^39bit，即512GB的空间。由于offset必须为12位，index剩下27位。

<img src="/Users/liuwenshuo/Documents/Notes/6.s801/Lectures/image-20220416152412931.png" alt="image-20220416152412931" style="zoom:33%;" />

* 物理地址是56bit，其中44bit是物理地址的page页号，剩下12bit是offset，与虚拟地址的offset一致，用于寻找一个page中的具体地址。

可以看到，物理地址的page号由44位表达，虚拟地址的page（index）由27位表达。因此只需将一个具体的虚拟地址的page号（index）翻译成物理地址的page号（以下简称PPN-Physical Page Number），再将其index直接拼接，即可得到具体的物理地址。

![image-20220416182630147](/Users/liuwenshuo/Documents/Notes/6.s801/Lectures/image-20220416182630147.png)

> 这就完美的解决问题了吗？

至此，已经可以实现了以page为粒度的页表，也设计了如何将虚拟地址转换为物理地址的方式。但**此时的page table有多大呢**？由于虚拟地址的index有27bits，对应2^27个page，以page为粒度，那么每个页表最多有2^27个条目，这也是相当大的一个页表，消耗了大量的内存！

实际上这27bits是由三个9bit的数字组成（L2、L1、L0）。前9个bit用来索引最高级的page directory，中间9个用来索引第二级的page directory，最后9个用来索引第三级。

一个Page Directory是4096Bytes，和page大小相同。Directory的每个条目成为Page Table Entry，是64bit，和寄存器大小相同。因此一个Page Directory有512个条目。

SATP寄存器指向的是最高一级的page directory的首地址，之后用虚拟内存中的高9bit（L2）索引最高一级的page directory，这样就得到了一个PPN，将这个PPN的末位都补成0（12bit的0），即可得到其“指向的”第二级的page directory的首地址。然后用L1索引第二级，得到指向最后一级的PPN。通过L0索引最后一级，完成索引。即可得到物理内存地址的PPN。

![image-20220416170628425](/Users/liuwenshuo/Documents/Notes/6.s801/Lectures/image-20220416170628425.png)

这样做的优点是，我们可以针对性的占用内存，而不是一次性占用2^27个条目的位置。例如假设我们现在只需要一个page的虚拟内存，那么一共只需要创建3*512个条目的位置，这相比2^27少了太多。

> Flags是什么？
>
> A：标志位
>
> * 第一个标志位是Valid。如果valid位1，则表明当前是一个可用的条目，可以进行地址翻译。
> * 下两个标志位是Readable、Writable，表示是否可读/写这个page
> * Executable表示可以执行指令
> * User表示这个page可以在用户空间被访问

