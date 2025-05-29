# eventfd接口函数

`eventfd`是linux系统提供的一个轻量级进程间通信（IPC）机制，用于事件通知。它通过文件描述符实现，通常与epoll、select或poll等多路复用技术结合使用，适用于高效的事件驱动编程。

```c
#include <sys/eventfd.h>

/* 功能: 创建一个事件文件描述符
 * 参数:
 *    initval: 初始计数器值（无符号64位整数）
 *    flags: EFD_CLOEXEC 执行exec时关闭文件描述符;
 *           EFD_NONBLOCK 非阻塞模式;
 *           EFD_SEMAPHORE 信号量模式, 逐次递减计数器;
 * 返回: >0（事件文件描述符）, -1（错误）
 */
int eventfd(unsigned int initval, int flags);
```

# eventfd使用示例

```c
int efd = eventfd(0, EFD_CLOEXEC | EFD_NONBLOCK);

uint64_t val = 1;
write(efd, &val, sizeof(val)); // 写线程触发事件

uint64_t buf;
ret = read(efd, &buf, sizeof(buf)); // 读线程读取事件

close(efd);
```

# eventfd和pipe对比

| **特性**         | **eventfd**                    | **pipe**                                 |
| ---------------------- | ------------------------------------ | ---------------------------------------------- |
| **本质**         | 内核维护的**64位无符号计数器** | 内核维护的**环形缓冲区（FIFO）**         |
| **数据类型**     | 仅支持 `uint64_t` 数值（事件计数） | 支持任意二进制数据（流式字节序列）             |
| **操作语义**     | 事件触发（增/减计数器）              | 数据读写（流式传输）                           |
| **描述符数量**   | 单描述符（读写共用）                 | 双描述符（读端 `pipe[0]`、写端 `pipe[1]`） |
| **内存占用**     | 极小（仅存储一个64位整数）           | 较大（默认缓冲区大小约64KB，可调整）           |
| **系统调用开销** | 低（仅操作计数器）                   | 较高（需操作环形缓冲区，涉及内存拷贝）         |
| **上下文切换**   | 少（轻量级事件通知）                 | 可能更多（流式数据需多次读写）                 |

eventfd通过 `read`/`write` 操作计数器：

* `write(fd, &val, 8)`：将`val` 加到计数器（若计数器溢出则阻塞，除非`EFD_NONBLOCK`）
* `read(fd, &val, 8)`：读取当前计数器值，并根据`EFD_SEMAPHORE` 标志决定重置（清零）或减 1
* **原子性** ：内核保证`read`/`write` 操作的原子性（无需额外同步）

pipe通过 `read(pipe[0], buf, len)` 和 `write(pipe[1], buf, len)` 操作缓冲区：

* 写操作：将数据追加到管道尾部（若缓冲区满则阻塞，除非设置`O_NONBLOCK`）
* 读操作：从管道头部读取数据（若缓冲区空则阻塞，除非设置`O_NONBLOCK`）
* **流式特性** ：数据按顺序读写，无消息边界（需应用层定义协议）
