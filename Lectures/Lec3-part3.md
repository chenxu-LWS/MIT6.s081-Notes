# Organization and System Calls-PART3⃣️

> XV6、qemu、risc-v的关系
>
> * XV6是一个操作系统，是在x86处理器上实现的Unix V6版本
>
> * riscv是指令集
>
> * qemu是模拟器
>
> 这里，XV6是基于riscv指令集的；在实验中，QEMU作为模拟器仿真了RISC-V处理器，使得我们的简易xv6可以跑在QEMU上

## 编译kernel

<img src="/Users/liuwenshuo/Documents/Notes/6.s801/Lectures/image-20220415194849885.png" alt="image-20220415194849885" style="zoom: 50%;" />

查看代码结构，最重要的有三个部分：

* **kernel**：XV6是宏内核，这里包含了基本上所有的内核文件。编译时会将这些代码编译成二进制文件，运行在kernel mode。
* **mkfs**：这里会创建一个空的文件镜像，我们会将这个镜像存放到硬盘，这样就可以直接使用一个空的文件系统。
* **user**：这些是运行在user mode的程序。

> #### 内核具体如何进行编译起来的？
>
> 1. **Makefile**读取一个C文件，以proc.c为例
> 2. 调用gcc编译器生成一个**汇编**文件proc.s
> 3. 走汇编解析器生成**可执行文件**的proc.o
> 4. 系统加载器收集所有的.o文件，进行**链接**，生成内核文件

<img src="/Users/liuwenshuo/Documents/Notes/6.s801/Lectures/image-20220415200345442.png" alt="image-20220415200345442" style="zoom:50%;" />

可以看到内核编译结束后，使用了qemu-system-riscv64指令。参数：

* -kernel：内核文件路径，这是即将在QEMU中运行的程序文件
* -m：RISC-V虚拟机中会使用的内存数量
* -smp：虚拟机可使用的CPU核数
* -drive：虚拟机使用的磁盘驱动，这里传入的是mkfs中生成的fs.img镜像文件

**至此XV6系统就在QEMU中启动了**

## QEMU

QEMU实际上就是一个计算机，可以认为它就是一个真正的主板，方针。可以用来启动XV6系统。

> #### QEMU如何做到仿真了RISC-V处理器？
>
> QEMU是一个开源的C程序，在github开源。
>
> 实际上在QEMU的主循环中只是做了一件事情：
>
> **读取**4字节～8字节的RISC-V指令，**解析**、找出对应的操作码（op code），在软件中**执行**相应指令。
>
> 为了完成这些，QEMU还需要以C语言声明，模拟X0、X1寄存器。

## XV6解析

