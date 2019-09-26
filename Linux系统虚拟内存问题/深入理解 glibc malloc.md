# Understanding glibc malloc
我总是着迷于堆内存，问题包括：
- 如何从内核获取堆内存？
- 如何有效地管理内存？
- 它是由内核、库还是由应用程序本身管理的？
- 可以利用堆内存吗？
这些问题在脑海里已经有一段时间了，但是直到最近我才有时间了解它，在这里分享一下对这些知识的学习和总结。

当前针对各大平台主要有如下几种堆内存管理机制：
- dlmalloc – General purpose allocator
- ptmalloc2 – glibc
- jemalloc – FreeBSD and Firefox
- tcmalloc – Google
- libumem – Solaris
- …
本文主要学习介绍在linux glibc使用的ptmalloc2实现原理。本来linux默认的是dlmalloc，但是由于其不支持多线程堆管理，所以后来被支持多线程的ptmalloc2代替了。

当然在linux平台*malloc本质上都是通过系统调用brk或者mmap实现的。关于这部分内容，一定要学习下面这篇文章：[Syscalls used by malloc](https://github.com/Huoke/jottings/blob/master/Linux系统虚拟内存问题/Syscalls%20used%20by%20malloc.md#malloc使用的系统调用)

每一个内存分配器都声称它们是快速的，可伸缩的和内存效率高的！！但并不是所有的分配器都适合我们的应用程序。内存密集型应用程序的性能在很大程度上取决于内存分配器的性能。

在这篇文章中，我只讨论“glibc malloc”内存分配器。将来，我希望能覆盖其他内存分配器。在这篇文章中，为了更好地理解glibc malloc，我将链接它最近的源代码。所以系好安全带，让我们从glibc malloc开始吧！！
**历史:** ptmalloc2是从dlmalloc派生的。之后支持多谢程被添加到其中，并于2006年发布。在正式发布之后，ptmalloc2被集成到glibc源代码中。集成后，直接修改的是glibc malloc源代码本身。因此，ptmalloc2和glibc的malloc实现之间可能会有很多变化。

**系统调用:** 如本文所示，malloc调用的是brk或者mmap系统调用。

**多线程:** 在Linux的早期，dlmalloc被用作默认内存分配器。但后来由于ptmalloc2对线程的支持，使得它成为linux的默认内存分配器。对多线程的支持能提高分配器的性能，同时也提高了应该程序的性能。在dlmalloc体系中，当两个线程同时调用malloc时，只有一个线程可以进入临界区，因为内存空闲链表freelist数据结构是被所有线程共享的。因此，在多线程场景中，内存分配需要时间，从而导致性能下降。在ptmalloc2中，当两个线程同时调用malloc时，会立即分配内存，因为每个线程都维护一个单独的堆栈，因此维护这些堆的空闲链表数据结构也是独立的。为每个线程维护单独的堆和freelist数据结构的行为成为per thread arena。

#### Example:
```C++
/* per thread arena example */
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>
#include <sys/types.h>

void* threadFunc(void* arg)
{
    printf("Before malloc in thread 1 \n");
    getchar();
    char* addr = (char*)malloc(1000);
    printf("After malloc and befor free in thread 1 \n");
    getchar();
    free(addr);
    printf("After free in thread 1\n");
    getchar();
}

int main()
{
    pthread_t t1;
    void* s;
    int ret;
    char* addr;
    
    printf("Welcome to per thread arena example::%d\n",getpid());
    printf("Before malloc in main thread\n");
    
    getchar();
    
    add = (char*)malloc(1000);
    printf(After malloc and before free in main thread\n);
    getchar();
    free(add);
    printf(After free in main thread\n);
    getchar();
    ret = pthread_create(&t1, NULL, threadFunc, NULL);
    if(ret)
    {
        printf("Thread creation error\n");
        return -1;
    }
    ret = pthread_join(t1, &s);
    if(ret)
    {
        printf("Thread join error\n");
        return -1;
    }
    return 0;
}
```
#### Output Analysis:
"Before malloc in main thread":在下面输出中我们可以看到还没有堆段，也没有每个线程堆栈，因为thread1还没有创建。
```Shell
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/mthread$ ./mthread 
Welcome to per thread arena example::6501
Before malloc in main thread
...
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/mthread$ cat /proc/6501/maps
08048000-08049000 r-xp 00000000 08:01 539625     /home/sploitfun/ptmalloc.ppt/mthread/mthread
08049000-0804a000 r--p 00000000 08:01 539625     /home/sploitfun/ptmalloc.ppt/mthread/mthread
0804a000-0804b000 rw-p 00001000 08:01 539625     /home/sploitfun/ptmalloc.ppt/mthread/mthread
b7e05000-b7e07000 rw-p 00000000 00:00 0 
...
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/mthread$
```
"After malloc in main thread": 主线程调用 malloc 之后，在下面的输出中，我们可以看到堆段已经创建，它就在数据段（0804b000-0806c000）的正上方，通过使用brk syscall 来增加程序中断位置来创建堆内存。同样需要注意，即使分配了1000字节的内存，malloc已经创建堆出132 KB的内存。这个连续的内存空间就称为**arena**。因为这个arena是被主线程创建的所以称为**main arena**。之后申请内存将继续使用这个arena直到arena的空闲空间使用完毕。When arena runs out of free space, it can grow by increasing program break location (After growing top chunk’s size is adjusted to include the extra space). Similarly arena can also shrink when there is lot of free space on top chunk.



**NOTE**: 顶部内存块是arena最顶部的内存块。详细的内容请查看"Top Chunk"部分。
Top chunk is the top most chunk of an arena. For further details about it, see “Top Chunk” section below.
```Shell
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/mthread$ ./mthread 
Welcome to per thread arena example::6501
Before malloc in main thread
After malloc and before free in main thread
...
sploitfun@sploitfun-VirtualBox:~/lsploits/hof/ptmalloc.ppt/mthread$ cat /proc/6501/maps
08048000-08049000 r-xp 00000000 08:01 539625     /home/sploitfun/ptmalloc.ppt/mthread/mthread
08049000-0804a000 r--p 00000000 08:01 539625     /home/sploitfun/ptmalloc.ppt/mthread/mthread
0804a000-0804b000 rw-p 00001000 08:01 539625     /home/sploitfun/ptmalloc.ppt/mthread/mthread
0804b000-0806c000 rw-p 00000000 00:00 0          [heap]
b7e05000-b7e07000 rw-p 00000000 00:00 0 
...
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/mthread$
```
"After free in main thread": 在下面的输出中我们可以看到当分配的内存被释放后，在它后面的内存不会立刻被操作系统回收。已经分配的内存区域(1000 bytes)被释放通过"glibc malloc"库，它把空闲的内存区域加到 main arena bin(在glibc malloc中, freelists(空闲链表)就是内存块容器(bins))。 
Later when user requests memory, ‘glibc malloc’ doesnt get new heap memory from kernel, instead it will try to find a free block in bin. And only when no free block exists, it obtains memory from kernel.
```C++
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/mthread$ ./mthread 
Welcome to per thread arena example::6501
Before malloc in main thread
After malloc and before free in main thread
After free in main thread
...
sploitfun@sploitfun-VirtualBox:~/lsploits/hof/ptmalloc.ppt/mthread$ cat /proc/6501/maps
08048000-08049000 r-xp 00000000 08:01 539625     /home/sploitfun/ptmalloc.ppt/mthread/mthread
08049000-0804a000 r--p 00000000 08:01 539625     /home/sploitfun/ptmalloc.ppt/mthread/mthread
0804a000-0804b000 rw-p 00001000 08:01 539625     /home/sploitfun/ptmalloc.ppt/mthread/mthread
0804b000-0806c000 rw-p 00000000 00:00 0          [heap]
b7e05000-b7e07000 rw-p 00000000 00:00 0 
...
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/mthread$
```
"Before malloc in thread1": 在下面的输出中，我们可以看到没有thread1堆段，但是现在创建了thread1的线程栈。
```C++
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/mthread$ ./mthread 
Welcome to per thread arena example::6501
Before malloc in main thread
After malloc and before free in main thread
After free in main thread
Before malloc in thread 1
...
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/mthread$ cat /proc/6501/maps
08048000-08049000 r-xp 00000000 08:01 539625     /home/sploitfun/ptmalloc.ppt/mthread/mthread
08049000-0804a000 r--p 00000000 08:01 539625     /home/sploitfun/ptmalloc.ppt/mthread/mthread
0804a000-0804b000 rw-p 00001000 08:01 539625     /home/sploitfun/ptmalloc.ppt/mthread/mthread
0804b000-0806c000 rw-p 00000000 00:00 0          [heap]
b7604000-b7605000 ---p 00000000 00:00 0 
b7605000-b7e07000 rw-p 00000000 00:00 0          [stack:6594]
...
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/mthread$
```
"After malloc in thread1": 下面的输出我们可以看到thread1的堆段被创建出来。并且映射到的内存段(b7500000-b7521000 whose size is 132 KB) 因此，这显示堆内存是使用mmap syscall创建的，而不是主线程（它使用sbrk）。在这里，尽管用户只请求1000字节，但大小为1 MB的堆内存被映射到进程地址空间。在这1MB里面，因为只有132KB的内存被设置了读写权限，这将称为此线程的堆内存。这个连续的内存区域(132KB)被称为线程的 arena。

**注意**: 当用户请求大小超过128kb(比如malloc(132*1024))并且arena中没有足够的空间满足用户请求时，不管请求是从主arena还是线程arena发出，都会使用mmap syscall（而不是sbrk）分配内存。
```Shell
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/mthread$ ./mthread 
Welcome to per thread arena example::6501
Before malloc in main thread
After malloc and before free in main thread
After free in main thread
Before malloc in thread 1
After malloc and before free in thread 1
...
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/mthread$ cat /proc/6501/maps
08048000-08049000 r-xp 00000000 08:01 539625     /home/sploitfun/ptmalloc.ppt/mthread/mthread
08049000-0804a000 r--p 00000000 08:01 539625     /home/sploitfun/ptmalloc.ppt/mthread/mthread
0804a000-0804b000 rw-p 00001000 08:01 539625     /home/sploitfun/ptmalloc.ppt/mthread/mthread
0804b000-0806c000 rw-p 00000000 00:00 0          [heap]
b7500000-b7521000 rw-p 00000000 00:00 0 
b7521000-b7600000 ---p 00000000 00:00 0 
b7604000-b7605000 ---p 00000000 00:00 0 
b7605000-b7e07000 rw-p 00000000 00:00 0          [stack:6594]
...
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/mthread$
```
"After free in thread1": 下面的输出中我们看到释放已经分配的内存空间但是堆内存不会被操作系统回收。相反，分配的内存区域（大小为1000字节）被释放到“glibc malloc”，它将这个释放的块添加到其线程arenas bin中。
```Shell
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/mthread$ ./mthread 
Welcome to per thread arena example::6501
Before malloc in main thread
After malloc and before free in main thread
After free in main thread
Before malloc in thread 1
After malloc and before free in thread 1
After free in thread 1
...
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/mthread$ cat /proc/6501/maps
08048000-08049000 r-xp 00000000 08:01 539625     /home/sploitfun/ptmalloc.ppt/mthread/mthread
08049000-0804a000 r--p 00000000 08:01 539625     /home/sploitfun/ptmalloc.ppt/mthread/mthread
0804a000-0804b000 rw-p 00001000 08:01 539625     /home/sploitfun/ptmalloc.ppt/mthread/mthread
0804b000-0806c000 rw-p 00000000 00:00 0          [heap]
b7500000-b7521000 rw-p 00000000 00:00 0 
b7521000-b7600000 ---p 00000000 00:00 0 
b7604000-b7605000 ---p 00000000 00:00 0 
b7605000-b7e07000 rw-p 00000000 00:00 0          [stack:6594]
...
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/mthread$
```
# Arena：
arena个数:在上面的例子中，我们看到主线程包含主arena，thread1有自己独立的thread arena。所以线程和arena之间是不是一对一的关系而不考虑线程的数量呢？当然不是。一个变态的应用程序可以包含很多的线程数（而不是核心数），在这种情况下，每个线程有一个arena变得有点昂贵和无用。因此，出于这个原因，应用程序的arena数量是基于系统中存在的核心数。
```Shell
For 32 bit systems:
     Number of arena = 2 * number of cores.
For 64 bit systems:
     Number of arena = 8 * number of cores.

```
## Multiple Arena:
举个例子: 一个多线程APP(4个线程—— 一个主线程 + 3个用户线程)运行在单核的32bit的操作系统机器上。这种情况下线程数thread(4)>核心数。因此, 在这种情况下，“glibc malloc” 确保所有可用线程共享多个arena。但它是如何分享的呢？
- When main thread, calls malloc for the first time already created main arena is used without any contention.
- When thread 1 and thread 2 calls malloc for the first time, a new arena is created for them and its used without any contention. Until this point threads and arena have one-to-one mapping.
- 当第3个线程第一次调用malloc时，计算arena的限制数量，发现线程数量超过了计算的arena数量，所以尝试使用已经存在的arena(主线程arena、或者thread1 arena1、或者thread2的arena2)。
- 重复使用arena:
  - 一点轮询到可用的arenas，就尝试锁定arena。
  - 如果加锁成功 (lets say main arena is locked successfully), 返回成功加锁的arena给用户程序。
  - 如果当前的arena没有空闲，就在下一个arena中继续尝试。
- 现在第三个线程第二次调用malloc时, malloc 将尝试使用第一次调用malloc时使用的arena(也就是主线程的arena)。 主arena是空闲的则使用否则线程3将被阻塞直到主arena被释放。因此主arena是被主线程和thread3共享的。
## Multiple Heaps(多堆)：
“glibc malloc”源代码中主要有以下三种数据结构:
heap_info(堆头) ———— 单个线程arena可以有多个 
# Chunk(内存块)：
## 分配内存块：
#### 注意：
## 释放内存块：
# Bins
## Fast Bin：
## Unsort Bin:
## Small Bin:
## Large Bin:
## Top Chunk:
## Last Remainder Chunk:





鉴于篇幅，本文就不加以详细说明了，只是为了方便后面对堆内存管理的理解，截取其中函数调用关系图：

