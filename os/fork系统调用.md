fork() 的核心作用是复制当前进程（父进程），生成一个几乎完全相同的子进程。调用 fork() 后，父进程和子进程会从 fork() 返回的位置开始，各自独立执行后续的代码。

# 返回值区分身份

父进程中：fork() 返回子进程的进程 pid
子进程中：fork() 返回 0
fork失败时：返回 -1

# 内存与资源复制

子进程会复制父进程的内存空间（代码、数据、堆、栈等）和文件描述符（如打开的文件、网络连接）。现代系统通常采用 "写时复制（Copy-On-Write, COW)" 优化：仅当父子进程修改内存时才实际复制，避免无意义的资源消耗。

# 执行顺序不确定

父子进程的执行顺序由操作系统调度决定，无法保证谁先运行。

# 代码示例

- 源代码（hahaha.c）

```c
#include <stdio.h>
#include <sys/wait.h>
#include <unistd.h>

int main()
{
	pid_t pid = fork();

	/* 父子进程从此处开始执行各自的代码 */

	if (pid == 0) { // 子进程
		pid_t child_pid = getpid();
		printf("child_pid: %u\n", child_pid);
		execl("/bin/sleep", "sleep", "300", NULL);
	} else if (pid > 0) { // 父进程
		pid_t parent_pid = getpid();
		printf("parent_pid: %u\n", parent_pid);
		waitpid(pid, NULL, 0);
		printf("子进程执行完成\n");
	}
	return 0;
}
```

- 执行结果

```bash
# gcc -o hahaha hahaha.c && ./hahaha
parent_pid: 41452
child_pid: 41453
```

- ps命令查看，其中`hahaha`是父进程，`sleep`是子进程

```bash
# ps -ef --forest | grep hahaha -C1
root    64574 19627  0 15:31 pts/101  00:00:00  |       |   \_ ps -ef --forest
root    64578 19627  0 15:31 pts/101  00:00:00  |       |   \_ grep --color=auto hahaha -C1
root    62441 77970  0 15:31 pts/173  00:00:00  |       \_ -bash
root    54583 77970  0 14:43 pts/179  00:00:00  |       \_ -bash
root    41452 54583  0 15:30 pts/179  00:00:00  |           \_ ./hahaha
root    41453 41452  0 15:30 pts/179  00:00:00  |               \_ sleep 300
```
