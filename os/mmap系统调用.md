# 一、mmap核心功能

`mmap`（Memory Map）是Unix/Linux系统中的重要系统调用，用于将**文件或设备映射到进程的虚拟内存空间** ，实现高效的文件I/O或进程间共享内存。以下是其核心用法和原理详解：

1. **文件映射** ：将文件内容直接映射到内存，读写内存即读写文件。
2. **匿名映射** ：创建进程间共享的纯内存区域（不关联文件）。
3. **高效I/O** ：避免`read/write` 的系统调用开销和用户态-内核态数据拷贝。
4. **共享内存** ：多个进程映射同一文件/区域可实现共享内存通信。

# 二、系统调用原型

```c
#include <sys/mman.h>

void *mmap(void *addr, size_t length, int prot, int flags, int fd, off_t offset);
int munmap(void *addr, size_t length);
```

参数说明：

| 参数       | 说明                                                                            |
| ---------- | ------------------------------------------------------------------------------- |
| `addr`   | 映射的起始地址（通常传NULL，由内核自动选择）                                    |
| `length` | 映射区域的长度（需 >0，且按页大小对齐）                                         |
| `prot`   | 内存保护标志（如可读、可写、可执行）                                            |
| `flags`  | 映射类型（共享/私有、匿名/文件等）                                              |
| `fd`     | 文件描述符（匿名映射时设为 `-1`）                                             |
| `offset` | 文件映射的起始偏移量（需按页大小对齐，通常为0）                                 |
| `return` | mmap成功时返回映射的起始地址，失败时返回-1<br />munmap成功时返回0，失败时返回-1 |

# 三、关键参数详解

- 内存保护标志`prot`（按位或组合）

| 标志           | 说明                   |
| -------------- | ---------------------- |
| `PROT_READ`  | 可读                   |
| `PROT_WRITE` | 可写                   |
| `PROT_EXEC`  | 可执行                 |
| `PROT_NONE`  | 禁止访问（用于哨兵页） |

- 映射类型`flags`（按位或组合）

| 标志              | 说明                                                   |
| ----------------- | ------------------------------------------------------ |
| `MAP_SHARED`    | 修改会写回文件，其他进程可见（用于文件映射或共享内存） |
| `MAP_PRIVATE`   | 修改仅对本进程有效（写时复制，用于进程私有映射）       |
| `MAP_ANONYMOUS` | 匿名映射（不关联文件，`fd` 需为 `-1`）             |
| `MAP_FIXED`     | 强制使用指定 `addr`（需谨慎，可能覆盖现有映射）      |
| `MAP_LOCKED`    | 锁定内存防止被交换到磁盘（需 `CAP_IPC_LOCK` 权限）   |

---

# 四、使用示例

文件映射（读写文件）

```c
#include <fcntl.h>
#include <stdio.h>
#include <sys/mman.h>
#include <unistd.h>

int main()
{
	int fd = open("test.txt", O_RDWR | O_CREAT, 0644);
	if (fd == -1) {
		perror("open");
		return -1;
	}

	// 获取文件大小
	int sz = 1024;
	if (ftruncate(fd, sz) == -1) {
		perror("ftruncate");
		close(fd);
		return -1;
	}

	// 映射文件到内存
	char *mapped = mmap(NULL, sz, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
	if (mapped == MAP_FAILED) {
		perror("mmap");
		close(fd);
		return -1;
	}

	// 通过内存修改文件内容
	sprintf(mapped, "Hello from mmap!\n");

	// 解除映射并关闭文件
	munmap(mapped, sz);
	close(fd);
	return 0;
}
```
