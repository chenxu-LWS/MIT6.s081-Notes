# Organization and System Calls-PART2⃣️



## 操作系统防御性

OS的防御性是指，抵御来自**应用程序对OS**的攻击，同时要防止应用程序打破自身**与其他应用程序的隔离**。

实际上OS的高防御性很难保证，即使是Linux也经常有一些内核bug使得应用程序打破隔离域并控制内核。因此作为开发OS的人员，必须掌握防御的思想，要在**应用程序和OS之间建立一堵高墙**，且OS对这堵墙可以进行任何想执行的策略。

这种强隔离性，需要通过**硬件**实现。包含两个部分：

* **user/kernel mode**
* **虚拟内存Virtual Memory**

---

## 硬件对强隔离的支持

### user / kernel mode

这是指处理器的两种模式，user mode和kernel mode。

当运行在kernel mode时，CPU可以运行特定权限的指令（指**汇编指令**）；user mode时，只能运行普通权限的指令。这就保证了**应用程序和OS之间的隔离性**。

> CPU如何检查当前mode？
>
> A：处理器中有一个flag，是处理器中的一个bit。当处理器解析指令时，如果指令为特殊权限指令且该bit为1，则会拒绝这条指令。

<img src="/Users/liuwenshuo/Documents/Notes/6.s801/Lectures/image-20220414215946725.png" alt="image-20220414215946725" style="zoom:33%;" />

如上图所示，我们认为用户态/内核态就是分隔用户空间和内核空间的边界。用户空间运行的程序运行在user mode，内核的运行在kernel mode。OS位于内核空间，因此拥有kernel mode的控制权。

但往往一个用户态的程序需要调用系统调用，因此必须有一种方式使得应用程序可以将控制权以一种协同工作的方式转移给内核（**Entering Kernel**）。

在RISC-V中，有一个ECALL指令就做了这件事。**ECALL <SYSCALL CODE>**，就可以通过**系统调用**的方式进入内核。

> ECALL会跳转到内核中一个由内核控制的特定的位置。每次应用程序执行ECALL，都会通过这个**接入点**进入内核。例如，当用户空间执行fork时，并不是直接**~~调用~~**OS中的对应函数，而是通过ECALL指令，并将fork系统调用号（一个数字）传给ECALL，由ECALL跳转到内核执行函数。

这个接入点，实际上就是一个函数，在我们的系统中它是位于syscall.c中的syscall函数。每次应用程序发起的系统调用都会调用到这个函数。syscall会检查ECALL的参数，转移到对应的系统调用。（具体在后面的part有介绍如何进行的代码跟踪）

### 虚拟内存（简易介绍）

处理器中包含Page Table（页表），**页表**用于将虚拟内存地址映射到物理内存。**每个进程拥有自己独立的页表**，因此每个进程只能访问自己页表中的物理内存。

OS要保证每个进程拥有不重合的物理内存。这样就保证了**应用程序之间的内存强隔离性**。

---

## 宏内核与微内核

宏内核：例如Linux、XV6(Lab中的OS)以及大多数经典的OS都是宏内核。宏内核是所有操作系统服务都运行在kernel mode，kernel有大量的代码

* 优点：OS的各个模块都在同一个大程序中，可以紧密的集成在一起
* 缺点：考虑bug的话，任何一个OS的bug都可能成为漏洞，因为在内核运行了一个巨大的操作系统，出现bug可能性也较高。

微内核：让尽可能少的代码运行在kernel mode

* 优点：少bug少漏洞

* 缺点：

  * OS模块之间的通信时，由于这些模块的一部分是在用户态实现的，会导致多次的用户控件<->内核空间的切换。因此有**反复跳转带来的性能损耗**。（图中通过shell进行系统调用，值返回给FS的流程）

    ![assets%2F-MHZoT2b_bcLghjAOPsJ%2F-MJbSdGiMLB2VO1kFUtK%2F-MJbcOEEsVLZivNXiWaO%2Fimage](/Users/liuwenshuo/Documents/Notes/6.s801/Lectures/assets%252F-MHZoT2b_bcLghjAOPsJ%252F-MJbSdGiMLB2VO1kFUtK%252F-MJbcOEEsVLZivNXiWaO%252Fimage.png)

  * 宏内核的紧耦合系统下，各组成部分很容易互相共享page cache，性能较高；但微内核下由于各组成部分分离，很难得到高性能。