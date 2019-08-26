# malloc使用的系统调用
登录到这个页面后，您应该知道malloc使用syscalls从操作系统获取内存。如下图所示，malloc调用brk或mmap 系统调用以获取内存。
## 一、brk
brk() 通过增加程序中断位置从内核中获取内存（初始化非零）。最开始堆的开头(start_brk)和结尾部分都指向相同的部分。
- 当ASLR关闭时，start_k和brk将指向data/bss段的末尾（end_data）。
- 当ASLR打开时，start_brk和brk将等于data/BSS段的结束（结束BRK数据）加上随机brk偏移量。
![](http://static.duartes.org/img/blogPosts/linuxFlexibleAddressSpaceLayout.png)

上面的“进程虚拟内存布局”图片显示start_brk是堆段的开始，brk（程序中断）是堆段的结束。
### 1.1、示例:
```Shell
/* sbrk and brk example */
#include <stdio.h>
#include <unistd.h>
#include <sys/types.h>

int main()
{
        void *curr_brk, *tmp_brk = NULL;

        printf("Welcome to sbrk example:%d\n", getpid());

        /* sbrk(0) gives current program break location */
        tmp_brk = curr_brk = sbrk(0);
        printf("Program Break Location1:%p\n", curr_brk);
        getchar();

        /* brk(addr) increments/decrements program break location */
        brk(curr_brk+4096);

        curr_brk = sbrk(0);
        printf("Program break Location2:%p\n", curr_brk);
        getchar();

        brk(tmp_brk);

        curr_brk = sbrk(0);
        printf("Program Break Location3:%p\n", curr_brk);
        getchar();

        return 0;
}
```
### 1.2 分析输出：
```Shell
Welcome to sbrk example:28478
Program Break Location1:0x804b000

Program break Location2:0x804c000

Program Break Location3:0x804b000
```
在映射物理内存的虚拟内存地址位置之前：在下面的输出中，我们可以观察到没有堆段。因此
- start_brk = brk = end_data = 0x804b000.
```Shell
/opt # ./a.out
Welcome to sbrk example:28478
Program Break Location1:0x804b000
...

 /opt # cat /proc/28479/maps
0804a000-0804b000 rw-p 00001000 08:01 539624                     /opt/test/a.out
b7e21000-b7e22000 rw-p 00000000 00:00 0 
7f7811850000-7f78119e9000 r-xp 00000000 00:00 188103             /lib64/libc-2.22.so
7f78119e9000-7f7811a26000 ---p 00199000 00:00 188103             /lib64/libc-2.22.so
```
增加程序需要映射物理内存的虚拟内存地址位置后：在下面的输出中我们可以观察到堆段。因此
- start_brk = end_data = 0x804b000
- brk = 0x804c000.
```Shell
Welcome to sbrk example:6141
Program Break Location1:0x804b000

Program Break Location2:0x804c000
...

0804a000-0804b000 rw-p 00001000 08:01 539624     /opt/test/a.out
0804b000-0804c000 rw-p 00000000 00:00 0          [heap]
b7e21000-b7e22000 rw-p 00000000 00:00 0 
```
其中:
- 0804b000-0804c000是此段的虚拟地址范围
- rw-p是标志（读、写、不执行、私有或者专用）
- 00000000 是文件偏移量，因为它没有从任何文件映射，所以这里为零。
- 00:00是主要/次要设备号——因为它没有从任何文件映射，所以这里是零。
- 0是inode编号-因为它没有从任何文件映射，所以这里是零
- [heap] 是堆段
# 二、mmap
malloc使用mmap创建私有匿名映射段。私有匿名映射的主要目的是分配新内存（零填充），而这个新内存将被调用进程独占使用。
## 2.1 示例：
```Shell
/* Private anonymous mapping example using mmap syscall */
#include <stdio.h>
#include <sys/mman.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdlib.h>

void static inline errExit(const char* msg)
{
        printf("%s failed. Exiting the process\n", msg);
        exit(-1);
}

int main()
{
        int ret = -1;
        printf("Welcome to private anonymous mapping example::PID:%d\n", getpid());
        printf("Before mmap\n");
        getchar();
        char* addr = NULL;
        addr = mmap(NULL, (size_t)132*1024, PROT_READ|PROT_WRITE, MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);
        if (addr == MAP_FAILED)
                errExit("mmap");
        printf("After mmap\n");
        getchar();

        /* Unmap mapped region. */
        ret = munmap(addr, (size_t)132*1024);
        if(ret == -1)
                errExit("munmap");
        printf("After munmap\n");
        getchar();
        return 0;
}
```
## 2.2 输出分析
1. 内存映射前(映射物理内存)：
在下面的输出中，我们只能看到属于共享库libc.so和ld-linux的内存映射段。
```Shell
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/syscalls$ cat /proc/6067/maps
08048000-08049000 r-xp 00000000 08:01 539691     /home/sploitfun/ptmalloc.ppt/syscalls/mmap
08049000-0804a000 r--p 00000000 08:01 539691     /home/sploitfun/ptmalloc.ppt/syscalls/mmap
0804a000-0804b000 rw-p 00001000 08:01 539691     /home/sploitfun/ptmalloc.ppt/syscalls/mmap
b7e21000-b7e22000 rw-p 00000000 00:00 0 
...
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/syscalls$
```
2. 内存映射后:
```Shell
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/syscalls$ cat /proc/6067/maps
08048000-08049000 r-xp 00000000 08:01 539691     /home/sploitfun/ptmalloc.ppt/syscalls/mmap
08049000-0804a000 r--p 00000000 08:01 539691     /home/sploitfun/ptmalloc.ppt/syscalls/mmap
0804a000-0804b000 rw-p 00001000 08:01 539691     /home/sploitfun/ptmalloc.ppt/syscalls/mmap
b7e00000-b7e22000 rw-p 00000000 00:00 0 
...
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/syscalls$
```
在下面的输出中，我们可以观察到我们的内存映射段（b7e0000–b7e2100，大小为132kb）与已经存在的内存映射段（b7e1000–b7e2000）相结合。
- 其中b7e0000-b7e2000是此段的虚拟地址范围
- rw-p是标志（读、写、不执行、私有）
- 00000000是文件偏移量–因为它没有从任何文件映射，所以这里是零
- 00:00是主要/次要设备号——因为它没有从任何文件映射，所以这里是零。
- 0是inode编号-因为它没有从任何文件映射，所以这里是零

3. munmap之后：
```Shell
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/syscalls$ cat /proc/6067/maps
08048000-08049000 r-xp 00000000 08:01 539691     /home/sploitfun/ptmalloc.ppt/syscalls/mmap
08049000-0804a000 r--p 00000000 08:01 539691     /home/sploitfun/ptmalloc.ppt/syscalls/mmap
0804a000-0804b000 rw-p 00001000 08:01 539691     /home/sploitfun/ptmalloc.ppt/syscalls/mmap
b7e21000-b7e22000 rw-p 00000000 00:00 0 
...
sploitfun@sploitfun-VirtualBox:~/ptmalloc.ppt/syscalls$
```
在下面的输出中，我们可以看到内存映射段是未映射的，即其相应的内存被释放到操作系统。

>注意：在我们的示例程序中，ASLR被关闭。



