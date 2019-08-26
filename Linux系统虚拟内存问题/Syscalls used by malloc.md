# malloc使用的系统调用
登录到这个页面后，您应该知道malloc使用syscalls从操作系统获取内存。如下图所示，malloc调用brk或mmap 系统调用以获取内存。
## 一、brk
brk() 通过增加程序中断位置从内核中获取内存（初始化非零）。最开始堆的开头(start_brk)和结尾部分都指向相同的部分。
- 当ASLR关闭时，start_k和brk将指向data/bss段的末尾（end_data）。
- 当ASLR打开时，start_brk和brk将等于data/BSS段的结束（结束BRK数据）加上随机brk偏移量。
![](http://static.duartes.org/img/blogPosts/linuxFlexibleAddressSpaceLayout.png)
上面的“进程虚拟内存布局”图片显示start_brk是堆段的开始，brk（程序中断）是堆段的结束。
