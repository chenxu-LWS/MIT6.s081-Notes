# Lab2 System Calls

> Lab2主要目标是理解xv6的系统调用过程，并为xv6增加两个新的系统调用



### 增加trace系统调用

> trace <mask> <command>
>
> trace用来跟踪一个程序进行了哪些系统调用。
>
> 其中mask是一个整型的掩码，系统调用是用从1开始的整数来做标识的，因此为了表示多个系统调用，这里使用mask掩码来表示我们要追踪的是哪些系统调用。例如要追踪调用码为1和3的，则mask=1010
>
> command是被跟踪的程序指令



根据Hints开始实验

对于整个xv6系统调用的架构

1. 用户态系统调用在user/user.h，user/user.pl
2. 内核态系统调用相关在kernel/syscall.c，kernel/syscall.h
3. 进程相关代码在kernel/proc.h and kernel/proc.c.



* 在Makefile的UPROGS加入trace程序，使shell可以调用到trace函数
* 在user.h声明系统调用trace，使得在用户态可以调用到trace函数
* 在user.pl添加entry，使得trace可以通过makefile的语句被编译为汇编（risc-v实现）
* 为了实现trace，根据hint，我们需要在proc结构体（记录进程信息的结构体）中加入一个mask值，将命令中的mask记录在进程中；同时为了使子进程也能获取到这个mask，需要修改fork函数，将mask也传递到子进程中
* 在kernel/syscall.c实现打印功能，进行系统调用获取的返回值放到寄存器a0中，通过mask和当前系统调用值num的掩码相与，判断是否打印。

<img src="/Users/liuwenshuo/Documents/Notes/MIT6.s081/Labs/Trace.png" alt="Trace" style="zoom: 50%;" />



### 增加sysinfo系统调用

