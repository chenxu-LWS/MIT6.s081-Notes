# Organization and System Calls-PART1⃣️



### 操作系统隔离性

> 为什么操作系统隔离性很重要？

* 隔离不同的应用程序

  * 防止应用程序之间产生影响

* 隔离操作系统和应用程序

  * 应用程序崩溃时，不会影响操作系统

  ---

> 没有OS会怎样？

假设OS并不存在，只是一些简单的库文件。如果有一个shell应用程序和一个echo应用程序。

那么应用程序会直接与硬件交互，比如应用程序可以直接看到CPU的核、磁盘、内存，没有额外的抽象层。

* 假设在shell程序中获取CPU的永久控制权，例如一个死循环，那么整个CPU资源都会被其占用，其他程序永远也得不到这个CPU，即无法得到**多进程分时复用**(**multiplexing**)的性质。
* 假设shell和echo程序都直接操作了内存的某个部分，其代码、数据等都直接保存到物理内存中。假设两个程序可以任意使用和篡改程序，那么可能发生内存覆盖等问题，导致崩溃，即无法得到**隔离性**。

**总结：在大部分分时OS中，将OS设计成一个库并不是一种常见的设计，需要通过提供抽象的硬件资源接口，强制实现硬件资源的隔离；但在实时OS中，应用程序彼此之间互相信任，则可以这样进行设计。**

---

> 举例-fork

例如unix常用的fork接口，创建了进程。进程本身不是CPU但对应了CPU资源，使得我们可以在CPU上运行计算任务。所以可以看到，我们的应用程序不能与CPU交互，只能和"抽象的进程"交互。OS会完成不同进程在CPU上的切换。

> 举例-exec

exec可以认为其抽象了内存。在运行exec时，会传入一个文件名，对应一个应用程序的**内存镜像**。内存镜像包含了程序对应的指令、全局数据。所以应用程序并不能直接访问物理内存，只能通过操作系统的内存隔离+内存控制，进行内存的隔层访问。

> 举例-file system

文件系统是OS的重要部分。files本质上抽象了硬盘，应用程序依然不可以直接访问硬盘资源，而是通过OS的抽象访问。通过这层抽象，我们只需对文件命名、读写，不需关心文件具体存在物理内存的哪里。OS会决定如何将文件与磁盘块对应，确保一个磁盘块只出现在一个文件中，且确保用户间不可以互相操作文件。

---

> CPU核数与进程

假设一个4核心CPU，我们有4个应用程序在运行，那么每个应用程序可以占有一个核心；

而如果此时有8个应用程序运行，则会分时复用这四个核心。
