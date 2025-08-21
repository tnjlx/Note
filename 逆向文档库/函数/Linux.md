<!-- TOC -->

- [1. fork](#1-fork)
  - [1.1. fork入门知识](#11-fork入门知识)
  - [1.2. fork进阶知识](#12-fork进阶知识)
- [2. getrlimit、setrlimit](#2-getrlimitsetrlimit)
  - [2.1. 定义](#21-定义)
  - [2.2. 参数resource](#22-参数resource)
  - [2.3. 结构体rlimit](#23-结构体rlimit)
  - [2.4. SHELL内建命令ulimit](#24-shell内建命令ulimit)
- [3. dup、dup2](#3-dupdup2)
  - [3.1. 内核中的文件描述符](#31-内核中的文件描述符)
  - [3.2. dup和dup2详解](#32-dup和dup2详解)
- [4. FlushInstructionCache](#4-flushinstructioncache)
- [5. demon](#5-demon)

<!-- /TOC -->
# 1. fork
## 1.1. fork入门知识
一个进程调用fork（）函数后，系统先给新的进程分配资源，例如存储数据和代码的空间。然后把原来的进程的所有值都复制到新的新进程中，只有少数值与原来的进程的值不同。相当于克隆了一个自己。fork调用的一个奇妙之处就是它仅仅被调用一次，却能够返回两次，它可能有三种不同的返回值：
* 在父进程中，fork返回新创建子进程的进程ID；
* 在子进程中，fork返回0；
* 如果出现错误，fork返回一个负值；

fork出错可能有两种原因：
* 当前的进程数已经达到了系统规定的上限，这时errno的值被设置为EAGAIN。
* 系统内存不足，这时errno的值被设置为ENOMEM。

每个进程都有一个独特（互不相同）的进程标识符（process ID），可以通过getpid（）函数获得，还有一个记录父进程pid的变量，可以通过getppid（）函数获得变量的值。

## 1.2. fork进阶知识
* 当一个进程死亡后，其fork出的子进程的父进程将会被设置为1，防止出现进程不存在父进程的情况。
* 调用printf函数时，操作系统仅仅是把内容放到了stdout的缓冲队列里，并没有实际的写到屏幕上。但是，只要看到有/n则会立即刷新stdout，因此就马上能够打印了。

# 2. getrlimit、setrlimit
## 2.1. 定义
在linux系统中，每种资源都有相关的软硬限制，比如进程的core file的最大值，虚拟内存的最大值等。 getrlimit、setrlimit可以获取或设定资源使用限制。
```c
#include <sys/resource.h>
int getrlimit(int resource, struct rlimit *rlim);
int setrlimit(int resource, const struct rlimit *rlim);
```
成功执行返回0，失败返回-1。

## 2.2. 参数resource
* RLIMIT_AS //进程的最大虚内存空间，字节为单位。
* RLIMIT_CORE //内核转存文件的最大长度。
* RLIMIT_CPU //最大允许的CPU使用时间，秒为单位。当进程达到软限制，内核将给其发送SIGXCPU信号，这一信号的默认行为是终止进程的执行。然而，可以捕捉信号，处理句柄可将控制返回给主程序。如果进程继续耗费CPU时间，核心会以每秒一次的频率给其发送SIGXCPU信号，直到达到硬限制，那时将给进程发送 SIGKILL信号终止其执行。
* RLIMIT_DATA //进程数据段的最大值。
* RLIMIT_FSIZE //进程可建立的文件的最大长度。如果进程试图超出这一限制时，核心会给其发送SIGXFSZ信号，默认情况下将终止进程的执行。
* RLIMIT_LOCKS //进程可建立的锁和租赁的最大值。
* RLIMIT_MEMLOCK //进程可锁定在内存中的最大数据量，字节为单位。
* RLIMIT_MSGQUEUE //进程可为POSIX消息队列分配的最大字节数。
* RLIMIT_NICE //进程可通过setpriority() 或 nice()调用设置的最大完美值。
* RLIMIT_NOFILE //指定比进程可打开的最大文件描述词大一的值，超出此值，将会产生EMFILE错误。
* RLIMIT_NPROC //用户可拥有的最大进程数。
* RLIMIT_RTPRIO //进程可通过sched_setscheduler 和 sched_setparam设置的最大实时优先级。
* RLIMIT_SIGPENDING //用户可拥有的最大挂起信号数。
* RLIMIT_STACK //最大的进程堆栈，以字节为单位。

## 2.3. 结构体rlimit
```c
struct rlimit {
　　rlim_t rlim_cur;　　//soft limit，软限制，超过此限制系统会进行告警
　　rlim_t rlim_max;　　//hard limit，硬限制，严格不可超过此限制，soft limit必须小于hard limit。
};
```

## 2.4. SHELL内建命令ulimit
该命令和函数具有相同的功能。
```bash
usage：ulimit [-SHacdefilmnpqrstuvx [limit]]
```
* 当不指定limit的时候，该命令显示当前值。
* 注意，当你要修改limit的时候，如果不指定-S或者-H，默认是同时设置soft limit和hard limit。

# 3. dup、dup2
## 3.1. 内核中的文件描述符
一个进程的生存周期中，会有一些文件被打开，从而会返回一些文件描述符，从shell中运行一个进程，默认会有3个文件描述符存在(0、１、2),0与进程的标准输入相关联，１与进程的标准输出相关联，2与进程的标准错误输出相关联，一个进程当前有哪些打开的文件描述符可以通过`/proc/进程ID/fd`目录查看。文件表中包含：文件状态标志、当前文件偏移量、v节点指针，每个打开的文件描述符(fd标志)在进程表中都有自己的文件表项，由文件指针指向。

## 3.2. dup和dup2详解
dup和dup2都可以用来复制一个现存的文件描述符。经常用来重新定向进程的STDIN,STDOUT,STDERR。

dup函数定义在`<unistd.h>`中，函数原形为：`int dup(int filedes);`。函数返回一个新的描述符，这个新的描述符是传给它的描述符的拷贝，若出错则返回－1。由dup返回的新文件描述符一定是当前可用文件描述符中的最小数值。这函数返回的新文件描述符与参数filedes将会共享同一个文件数据结构。

dup2函数定义在`<unistd.h>`中，函数原形为：`int dup2(int filedes, int filedes2)`;。同样，函数返回一个新的文件描述符，若出错则返回－1。与dup不同的是，dup2可以用filedes2参数指定新描述符的数值。如果filedes2已经打开,则先将其关闭。如若filedes等于filedes2,则dup2返回filedes2,而不关闭它。同样，返回的新文件描述符与参数filedes共享同一个文件数据结构。

# 4. FlushInstructionCache
在对内存进行写操作后，视情况需要调用FlushInstructionCache函数刷新cache，否则可能cache中的数据为脏数据，导致出现错误，尤其是在多核处理器上，对于memcpy和rep这种指令

# 5. demon
该函数涉及到Linux操作系统的守护进程，详见[Linux守护进程](../Linux/Linux守护进程)。
