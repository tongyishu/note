在分析uio驱动时，一直不明白uio是如何使用open系统调用打开/dev/uioX设备文件，并进行read/write操作的。

为什么read/write能够识别uio的操作？什么时候注册uio的read/write回调的？于是分析了一下open系统调用的实现，便于理解uio的实现。

```c
#include <error.h>
#include <fcntl.h>
#include <stdio.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
	int fd = open("a.txt", O_WRONLY | O_CREAT | S_IRWXU);
	if (fd < 0)
		return -1;
	close(fd);
	return 0;
}
```

open系统调用的函数原型：

**int open(const char *name, int flags, mode_t mode);**

与之对应的系统调用：

**SYSCALL_DEFINE3(open, const char __user*, filename, int flags, umode_t mode);**

```bash
open()
	do_sys_open()
		get_unused_fd_flags()  # 获取未使用的fd
			alloc_fd()
		do_file_open()  # 创建struct file结构
			set_nameidata()
			path_openat()
				alloc_empty_file()
				path_init()
				link_path_walk()
				do_last()
					vfs_open()
						do_dentry_open()
							file->f_inode = inode;
							file->f_mapping = inode->i_mapping;
							file->f_op = fops_get(inode->i_fop);

		fd_install()  # 将fd与struct关联, 对于struct file的操作直接转换为对fd的操作
```

open会调用do_sys_open函数来实现：

1、获取一个新的未使用的fd，get_unused_fd_flags，用于唯一标识此进程即将要打开的文件

2、根据文件名查找文件的inode，并根据inode创建进供程使用的struct file结构体

3、将fd与struct file结构关联，对于struct file的操作将直接转换为对fd的操作，如read()/write()

在创建struct file结构体的过程中，会使用struct nameidata结构体来记录文件名等相关信息（便于查找），并查找文件路径（link_path_walk），找到文件后会调用do_last来在vfs中创建一个struct file结构体，并将对于struct inode的ops操作传递给struct file。

因此，对于fd的操作会转化为对struct file的操作，进而转化为对struct inode的操作。
