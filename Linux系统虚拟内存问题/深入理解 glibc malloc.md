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

**多线程:** 在Linux的早期，dlmalloc被用作默认内存分配器。但后来由于ptmalloc2对线程的支持，使得它成为linux的默认内存分配器。对多线程的支持能提高分配器的性能，同时也提高了应该程序的性能。




鉴于篇幅，本文就不加以详细说明了，只是为了方便后面对堆内存管理的理解，截取其中函数调用关系图：

