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
