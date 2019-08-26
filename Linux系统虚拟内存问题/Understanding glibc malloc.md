# Understanding glibc malloc
我总是着迷于堆内存，问题包括：
- 如何从内核获取堆内存？
- 如何有效地管理内存？
- 它是由内核、库还是由应用程序本身管理的？
- 可以利用堆内存吗？
这些问题在脑海里已经有一段时间了，但是直到最近我才有时间了解它，在这里分享一下对这些知识的学习和总结。

在现实生活中(Out there in the wild), 有许多内存分配器可用：
- dlmalloc – General purpose allocator
- ptmalloc2 – glibc
- jemalloc – FreeBSD and Firefox
- tcmalloc – Google
- libumem – Solaris
- …
