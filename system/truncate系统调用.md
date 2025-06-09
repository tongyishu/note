# 一、函数原型

`truncate/ftruncate` 是Unix/Linux系统中的一个重要系统调用，用于修改文件大小。它可以将文件截断到指定长度（扩大或缩小）。

```c
#include <sys/types.h>
#include <unistd.h>

/* return: 成功返回0，失败返回-1 */
int ftruncate(int fd, off_t length);	      // 通过文件描述符操作
int truncate(const char *path, off_t length); // 通过文件路径操作
```

# 二、核心功能

1. **扩大文件** ： 若 length > 原文件大小，文件末尾会用空字节（`\0`）填充到指定长度。
2. **缩小文件** ： 若 length < 原文件大小，超出部分的数据会被永久丢弃。
3. **清空文件** ： 若 length == 0，则清空文件。
