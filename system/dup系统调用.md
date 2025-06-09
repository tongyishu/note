# 文件描述符的本质

在理解dup系列函数前，需先理解几个内核的概念：

```bash
/* 进程相关 */
struct task_struct // 每个task_struct为一个进程，包含一个files_struct指针
	struct files_struct // 进程的文件管理
		struct fdtable fdt[0]  // 标准输入，通常关联到终端设备/dev/pts/XXX
		struct fdtable fdt[1]  // 标准输出，通常关联到终端设备/dev/pts/XXX
		struct fdtable fdt[2]  // 标准错误，通常关联到终端设备/dev/pts/XXX
		struct fdtable fdt[fd] // fd为文件描述符，本质上为fdtable的下标索引，每个fdtable包含一个file指针
		...       |
/* 系统相关 */             v
struct file // 每个file为一个打开的文件实例，操作系统级别（进程无关），记录文件打开状态信息，每个file包含一个inode指针
	struct inode // 每个inode为一个磁盘文件，操作系统级别（进程无关），记录文件本身的信息
```

复制文件描述符（dup）的本质是在进程的文件描述符表（fdt）中新增一个fdtable条目，指向同一个 `struct file` 表项 。因此新旧描述符共享文件偏移量、打开标志等状态，但各自的描述符标志 `O_CLOEXEC`是独立的。

# dup/dup2/dup3系统调用

```c
/* oldfd: 待复制的原文件描述符，必须有效且已打开
 * return: 成功返回新的文件描述符（当前进程中最小未使用的文件描述符）；失败返回 -1
 */
int dup(int oldfd);

/* oldfd: 待复制的原描述符，必须有效且已打开
 * newfd: 要复制的目标描述符，即新的描述符数值
 * return: 成功返回 newfd；失败返回 -1
 */
int dup2(int oldfd, int newfd);

#define _XOPEN_SOURCE 700 // 需定义此宏以启用 dup3（POSIX.1-2008 标准）
/* oldfd: 待复制的原描述符，必须有效且已打开
 * newfd: 要复制的目标描述符，即新的描述符数值
 * flags: 控制复制行为，目前仅支持 O_CLOEXEC
 * return: 成功返回 newfd；失败返回 -1
 */
int dup3(int oldfd, int newfd, int flags);
```

# 三者对比

| 特性                   | dup                          | dup2                           | dup3                         |
| ---------------------- | ---------------------------- | ------------------------------ | ---------------------------- |
| **函数原型**     | dup(oldfd)                   | dup2(oldfd, newfd)             | dup3(oldfd, newfd, flags)    |
| **新描述符选择** | 内核自动分配（最小可用）     | 由 newfd 指定                  | 由 newfd 指定                |
| **覆盖旧描述符** | 不支持（需手动 close）       | 支持（自动关闭 newfd）         | 支持（自动关闭 newfd）       |
| **标志支持**     | 无                           | 无                             | 仅 O_CLOEXEC                 |
| **exec自动关闭** | 否（需手动 fcntl）           | 否（需手动 fcntl）             | 是（通过 O_CLOEXEC）         |
| **典型场景**     | 简单复制（不关心描述符编号） | I/O 重定向（如替换标准描述符） | 多进程场景（避免描述符泄漏） |
| **系统兼容性**   | 所有 Unix 系统               | 所有 Unix 系统                 | Linux 2.6.23+（需宏定义）    |

# 代码示例

- 使用dup2实现重定向功能（test.c）

```c
#include <fcntl.h>
#include <stdio.h>
#include <unistd.h>

int main()
{
	// 保存原标准输出描述符（用于恢复）
	int __stdout = dup(STDOUT_FILENO);
	// 打开目标文件（写模式，创建/截断）
	int redirect = open("output.txt", O_RDWR | O_CREAT, 0644);

	// 重定向标准输出到文件
	dup2(redirect, STDOUT_FILENO);
	close(redirect);

	// 输出到文件
	printf("重定向到文件的内容\r\n");
	fflush(stdout); // 立即清空缓冲区内容

	// 恢复标准输出到终端
	dup2(__stdout, STDOUT_FILENO);
	close(__stdout);

	// 输出到终端
	printf("标准输出已恢复\r\n");
	return 0;
}
```

- 运行结果

```bash
# gcc -o test test.c && ./test
标准输出已恢复
# cat output.txt 
重定向到文件的内容
```
