# Page Table-PART2⃣️



## 页表缓存TLB

这里主要针对页表进行优化。在我们的三层页表结构下，每次从内存加载或写入数据时，都需要做三次内存查找（三层page directory）。所以这里可以加一层缓存层进行优化。这个缓存层称为TLB。

如果切换了页表，那么TLB也会被清空失效。

实际上**TLB和MMU**都并不是OS级别的实现，而是**硬件级别**的实现，因此我们在OS中并不需要考虑这个，只需要知道有这个缓存层即可。如下图是一个主板，中间是一个RISC-V处理器，旁边是处理器的4个核心，每个核有自己的MMU和TLB。处理器旁边就是DRAM(物理内存)芯片。

![img](/Users/liuwenshuo/Documents/Notes/6.s801/Lectures/assets%252F-MHZoT2b_bcLghjAOPsJ%252F-MK_UbCc81Y4Idzn55t8%252F-MKaZik5ig19xs1OK5WX%252Fimage.png)

这解释了为什么我们的OS启动是从0x80000000开始的。完成虚拟地址到物理地址的翻译后，如果物理地址大于0x80000000表示在DRAM物理内存中的某个地址；否则是其他的IO设备。（例如使用多路选择器进行分发）

## 内核页表

> **内核页表和用户页表**：
>
> * **每个用户进程有自己单独的用户页表；而内核页表是所有进程共享的。**这样做的目的是出于性能考虑，当不同进程从用户态切入内核态时，切入的都是一套页表，即内核态那张表的TLB是不太需要刷新的。
> * **内核页表的虚拟地址和物理地址是直接映射的，而用户页表则不是，用户页表的地址是从0开始的**
> * 
> * **内核页表指内核地址空间的页表，用户页表是用户某个进程的地址空间页表**
> * **内核页表和用户页表都是保存在内核中的，用户无法直接管理**
> * **当内核和用户之间需要传递某个地址（例如一个指针）时，无法使用相同的虚拟地址，因此需要先翻译成物理地址，再传递**

下图展示了**内核空间**虚拟地址-物理地址的映射关系（内核页表的结构）。

> 可以看到虚拟地址在KERNBASE到PHYSTOP的区间内，与物理地址是水平映射的，即虚拟地址=物理地址。

其中有几个可以注意的点：

* Guard page是什么？可以每个kernel stack有一个Guard page，如果kernel stack耗尽了，它会溢出到Guard page。但OS对guard page做了特殊处理，这个page对应的PTE没有设置valid，因此当解析道这段地址时会触发page fault，防止了kernel stack的溢出。

<img src="https://906337931-files.gitbook.io/~/files/v0/b/gitbook-legacy-files/o/assets%2F-MHZoT2b_bcLghjAOPsJ%2F-MK_UbCc81Y4Idzn55t8%2F-MKaY9xY8MaH5XTiwuBm%2Fimage.png?alt=media&token=3adbe628-da78-472f-8e7b-3d0b1d3177b5" alt="img"  />

如下图物理地址的分布。可以看到最下面是未被使用的地址，地址0x1000是boot ROM的物理地址。当我们对主板加电，主板首先做的事情就是运行存储在bootROM中的代码。boot完成后跳转到0x80000000。

<img src="/Users/liuwenshuo/Documents/Notes/6.s801/Lectures/assets%252F-MHZoT2b_bcLghjAOPsJ%252F-MK_UbCc81Y4Idzn55t8%252F-MKaeaT3eXOyG4jfKKU7%252Fimage.png" alt="assets%2F-MHZoT2b_bcLghjAOPsJ%2F-MK_UbCc81Y4Idzn55t8%2F-MKaeaT3eXOyG4jfKKU7%2Fimage" style="zoom: 67%;" />

还有一些其他的IO设备：

* PLIC：中断控制器
* UART0：负责与console和显示器交互
* VIRTIO disk：负责与磁盘交互