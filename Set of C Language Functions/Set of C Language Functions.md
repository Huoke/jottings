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
