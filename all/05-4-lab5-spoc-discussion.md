# lab5 spoc 思考题

- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。


## 个人思考题

### 总体介绍

 - 第一个用户进程创建有什么特殊的？
 - 系统调用的参数传递过程？
 - getpid的返回值放在什么地方了？

### 进程的内存布局

 - 尝试在进程运行过程中获取内核堆栈和用户堆栈的调用栈？
 - 尝试在进程运行过程中获取内核空间中各进程相同的页表项（代码段）和不同的页表项（内核堆栈）？

### 执行ELF格式的二进制代码-do_execve的实现

 - 在do_execve中进程清空父进程时，当前进程是哪一个？在什么时候开始使用新加载进程的地址空间？
 - 新加载进程的第一级页表的建立代码在哪？

### 执行ELF格式的二进制代码-load_icode的实现

 - 第一个内核线程和第一个用户进程的创建有什么不同？
 - 尝试跟踪分析新创建的用户进程的开始执行过程？

### 进程复制

 - 为什么新进程的内核堆栈可以先于进程地址空间复制进行创建？
 - 进程复制的代码在哪？复制了哪些内容？
 - 进程复制过程中有哪些修改？为什么要修改？

### 内存管理的copy-on-write机制
 - 什么是写时复制？
 - 写时复制的页表在什么时候进行复制？共享地址空间和写时复制有什么不同？

## 小组练习与思考题

### (1)(spoc) 在真实机器的u盘上启动并运行ucore lab,

请准备一个空闲u盘，然后请参考如下网址完成练习

https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-boot-with-grub2-in-udisk.md

(报告可课后完成)请理解grub multiboot spec的含义，并分析ucore_lab是如何实现符合grub multiboot spec的，并形成spoc练习报告。

### (2)(spoc) 理解用户程的生命周期。

> 需写练习报告和简单编码，完成后放到git server 对应的git repo中

### 练习用的[lab5 spoc exercise project source code](https://github.com/chyyuu/ucore_lab/tree/master/related_info/lab5/lab5-spoc-discuss)

#### 掌握知识点
1. 用户进程的启动、运行、就绪、等待、退出
2. 用户进程的管理与简单调度
3. 用户进程的上下文切换过程
4. 用户进程的特权级切换过程
5. 用户进程的创建过程并完成资源占用
6. 用户进程的退出过程并完成资源回收

> 注意，请关注：内核如何创建用户进程的？用户进程是如何在用户态开始执行的？用户态的堆栈是保存在哪里的？

阅读代码，在现有基础上再增加一个用户进程，并通过增加cprintf函数到ucore代码中，
能够把上述知识点中的动态执行过程通过`cprintf`函数来完整地展现出来

请完成如下练习，完成代码填写，并形成spoc练习报告

```
内核创建用户进程的方法就是调用user_main函数来创建用户进程，user_main函数调用kernel_execve函数使用int指令发出一个软中断，接着就进行系统调用do_execve处理，在这个函数中会设置好进程的cr3寄存器与内存空间，然后调用load_icode函数载入ELF格式的二进制用户程序文件，而在load_icode中初始化trapframe的代码为

tf->tf_cs = USER_CS;
tf->tf_ds = tf->tf_es = tf->tf_ss = USER_DS;
tf->tf_esp = USTACKTOP;
tf->tf_eip = elf->e_entry;
tf->tf_eflags = FL_IF;


将相应的段寄存器都设为了USER_CS或USER_DS，esp设为了USTACKTOP，这样表明这个进程运行在用户态。
切换到用户态的过程是在trap中实现的，在do_execve结束后此进程处于RUNNABLE状态等待调度，在下一次系统因为时钟中断调用schedule函数时，proc_run函数将调用switch_to将相应的段寄存器设置好。然后在trapentry.S的尾部有一段代码


# call trap(tf), where tf=%esp
    call trap

    # pop the pushed stack pointer
    popl %esp

    # return falls through to trapret...
.globl __trapret
__trapret:
    # restore registers from stack
    popal

    # restore %ds, %es, %fs and %gs
    popl %gs
    popl %fs
    popl %es
    popl %ds

    # get rid of the trap number and error code
    addl $0x8, %esp
    iret

.globl forkrets
forkrets:
    # set stack to this new process's trapframe
    movl 4(%esp), %esp
    jmp __trapret

调用完trap后，连续使用6个popl语句恢复了当前程序执行的现场，最后用一条iret指令切换到用户态。

而在memlayout.h中有宏定义

#define USERTOP             0xB0000000
#define USTACKTOP           USERTOP
#define USTACKPAGE          256                         // # of pages in user stack
#define USTACKSIZE          (USTACKPAGE * PGSIZE)       // sizeof user stack

规定了用户态的堆栈顶在0xB000000处。

用户进程相关函数
1. 启动、运行：do_execve, proc_run; 等待: do_wait; 退出: do_exit
2. 管理与调度: schedule
3. 上下文切换: switch_to
4. 特权级切换: trapentry.S
5. 创建过程并完成资源占用: alloc_proc -> do_fork -> do_execve -> load_icode
6. 退出并完成资源回收: do_exit -> init_main （由initproc内核线程回收）
```
