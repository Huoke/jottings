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
本文主要学习介绍在linux glibc使用的ptmalloc2实现原理。本来linux默认的是dlmalloc，但是由于其不支持多线程堆管理，所以后来被支持多线程的prmalloc2代替了。

当然在linux平台*malloc本质上都是通过系统调用brk或者mmap实现的。关于这部分内容，一定要学习下面这篇文章：[Syscalls used by malloc](https://sploitfun.wordpress.com/2015/02/11/syscalls-used-by-malloc/)
