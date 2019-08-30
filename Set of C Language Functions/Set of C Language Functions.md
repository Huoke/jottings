# C语言函数小集合
## strdup（）
strdup（）函数是c语言中常用的一种字符串拷贝库函数，一般和free（）函数成对出现。

原型： extern char *strdup(char *s);

头文件：string.h
```Shell
char *strdup(const char *s)
{
        char *t = NULL;
        if (s && (t = (char*)malloc(strlen(s) + 1)))
        strcpy(t, s);
        return t;
}  
```
说明：

功 能: 将串拷贝到新建的位置处
strdup()在内部调用了malloc()为变量分配内存，不需要使用返回的字符串时，需要用free()释放相应的内存空间，否则会造成内存泄漏。

返回值：
返回一个指针,指向为复制字符串分配的空间;如果分配空间失败,则返回NULL值。

xstrdup 是在分配内存时使用的xmalloc，它的区别就是如果没有足够的剩余内存分配会在系统日志中记录，并且程序以EXIT_FAILURE退出。
## strcpy()
strcpy() 把从src地址开始且含有NULL结束符的字符串复制到以dest开始的地址空间.

原型：char *strcpy(char* dest, const char *src);

头文件：#include <string.h>和 #include <stdio.h>

说明：strcpy是标准的C语言标准库函数。src和dest所指内存区域不可以重叠且 dest必须有足够的空间来容纳src的字符串。

返回值：返回指向dest的指针。
```Shell
 // strcpy.     
#include <syslib.h>
#include <string.h>     
main() {        
    char *s="Golden Global View";
    char d[20]; 
    strcpy(d, s);
    printf("%s",d);       
    return 0;    
}
 ```
## 对比
1. strdup 不是标准的c函数，strcpy是标准的c函数，使用时注意场合。
2. strdup 可以直接把要复制的内容复制给**没有初始化的指针**，因为它会自动分配空间给目的指针，strcpy的目的指针一定是已经分配内存的指针。
3. strdup用完要free()函数释放内存，否则内存泄露 。
4. 使用strcpy必须事先确定src大小，可以先strlen判断src的大小，之后为dest申请空间，之后再strcpy就不会有问题了。
## xcalloc / xmalloc
alloc.c包含一个名为xcalloc的函数，它与xmalloc类似，只是它将内存初始化为零。
通常情况下我更喜欢新的xcalloc。比如：xmalloc（n_things*sizeof*things）; xcalloc（n_things，sizeof*things）；

前者增加了整数溢出的可能性，因此产生了缓冲区超支。在后一种情况下，我们有更多的机会进行边界检查。

malloc函数的实质体现在，它有一个将可用的内存块连接为一个长长的列表的所谓空闲链表。调用malloc函数时，它沿连接表寻找一个大到足以满足用户请求所需要的内存块。然后，将该内存块一分为二（一块的大小与用户请求的大小相等，另一块的大小就是剩下的字节）。接下来，将分配给用户的那块内存传给用户，并将剩下的那块（如果有的话）返回到连接表上。调用free函数时，它将用户释放的内存块连接到空闲链上。到最后，空闲链会被切成很多的小内存片段，如果这时用户申请一个大的内存片段，那么空闲链上可能没有可以满足用户要求的片段了。于是，malloc函数请求延时，并开始在空闲链上翻箱倒柜地检查各内存片段，对它们进行整理，将相邻的小空闲块合并成较大的内存块。如果无法获得符合要求的内存块，malloc函数会返回NULL指针，因此在调用malloc动态申请内存块时，一定要进行返回值的判断。
Linux Libc6采用的机制是在free的时候试图整合相邻的碎片，使其合并成为一个较大的free空间。

## 与new的区别编辑
从本质上来说，malloc（Linux上具体实现可以参考man malloc，glibc通过brk()&mmap()实现）是libc里面实现的一个函数，如果在source code中没有直接或者间接include过stdlib.h，那么gcc就会报出error：‘malloc’ was not declared in this scope。如果生成了目标文件（假定动态链接malloc），如果运行平台上没有libc（Linux平台，手动指定LD_LIBRARY_PATH到一个空目录即可），或者libc中没有malloc函数，那么会在运行时（Run-time）出错。new则不然，是c++的关键字，它本身不是函数。new不依赖于头文件，c++编译器就可以把new编译成目标代码（g++4.6.3会向目标中插入_Znwm这个函数，另外，编译器还会根据参数的类型，插入相应的构造函数）。
在使用上，malloc 和 new 至少有两个不同: new 返回指定类型的指针，并且可以自动计算所需要大小。而 malloc 则必须要由我们计算字节数，并且在返回后强行转换为实际类型的指针。另外有一点不能直接看出的区别是，malloc 只管分配内存，并不能对所得的内存进行初始化，所以得到的一片新内存中，其值将是随机的。除了分配及最后释放的方法不一样以外，通过malloc或new得到指针，在其它操作上保持一致。 [2] 
