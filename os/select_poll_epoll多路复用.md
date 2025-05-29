# IO多路复用

IO 多路复用（I/O Multiplexing）是一种高效的 I/O 管理机制，用于同时监听多个文件描述符（如套接字、文件等），当其中任意一个或多个文件描述符处于可读、可写或异常状态时，通知应用程序进行处理。

# 核心原理

通过一个系统调用（如 `select`/`poll`/`epoll`），监听多个文件描述符的状态变化（可读/可写/异常）。应用程序只需阻塞在这个系统调用上，无需为每个连接单独创建线程/进程，从而减少资源消耗并提升并发性能。

# select系统调用

- 接口说明

```c
#include <sys/select.h>

/* 功能: 监听文件描述符集合的状态变化
 * 参数:
 *    nfds: 最大文件描述符的值 + 1（用于限定检查范围）
 *    readfds: 关注「可读」事件的描述符集合（传入传出参数）
 *    writefds: 关注「可写」事件的描述符集合
 *    exceptfds: 关注「异常」事件的描述符集合
 *    timeout: 超时时间（NULL 表示永久阻塞）
 * 返回: >0（就绪描述符数量）；0（超时）；-1（错误）
 */
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);

/* 辅助宏, 操作fd_set集合 */
int  FD_ISSET(int fd, fd_set *set); // 检查fd是否在集合中
void FD_ZERO(fd_set *set);	    // 清空集合
void FD_SET(int fd, fd_set *set);   // 将fd加入集合
void FD_CLR(int fd, fd_set *set);   // 将fd从集合移除
```

- 使用示例

```c
fd_set read_fds;
FD_ZERO(&read_fds);
FD_SET(socket_fd, &read_fds); // socket_fd为要监听的描述符

struct timeval timeout = {5, 0}; // 5秒超时
int nfds = select(socket_fd + 1, &read_fds, NULL, NULL, &timeout); // 注意第一个参数为socket_fd + 1
if (nfds > 0) {
	if (FD_ISSET(socket_fd, &read_fds)) {
		// 处理读事件
	}
}
```

# poll系统调用

- 接口说明

```c
#include <poll.h>

/* 结构体: 描述一个待监听的文件描述符及其事件 */
struct pollfd {
	int fd;	       // 待监听的文件描述符（-1 表示忽略）
	short events;  // 关注的事件（如 POLLIN: 可读；POLLOUT: 可写）
	short revents; // 实际发生的事件（由内核填充）
};

/* 功能: 监听pollfd数组中的描述符状态变化
 * 参数:
 *    fds: pollfd数组（传入传出参数）
 *    nfds: 数组长度（最大文件描述符的数量）
 *    timeout: 超时时间（-1 表示永久阻塞，单位: 毫秒）
 * 返回: >0（就绪描述符数量）；0（超时）；-1（错误）
 */
int poll(struct pollfd *fds, nfds_t nfds, int timeout);
```

- 使用示例

```c
struct pollfd fds[1];
fds[0].fd = socket_fd; // socket_fd为要监听的描述符
fds[0].events = POLLIN;

int nfds = poll(fds, 1, 5000); // 5秒超时
if (nfds > 0 && (fds[0].revents & POLLIN)) {
	// 处理读事件
}
```

# epoll系统调用

- 接口说明

```c
#include <sys/epoll.h>

/* 结构体: 描述epoll事件 */
struct epoll_event {
	uint32_t events; // 事件类型, EPOLLIN: 可读; EPOLLOUT: 可写
	void *data;	 // 自定义数据, 通常存储fd或指针
};

/* 功能: 创建epoll实例（内核2.6.8后size参数被忽略, 建议设为1）
 * 返回: >0（epoll实例的文件描述符）, -1（错误）
 */
int epoll_create(int size);

/* 功能: 控制epoll事件, 添加/修改/删除要监听的描述符
 * 参数:
 *    epfd: epoll实例的文件描述符
 *    op: 操作类型, EPOLL_CTL_ADD: 添加; EPOLL_CTL_MOD: 修改; EPOLL_CTL_DEL: 删除
 *    fd: 待监听的文件描述符
 *    event: 关联的事件（epoll_event结构体）
 * 返回值: 0（成功）; -1（错误）
 */
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);

/* 功能: 等待epoll事件就绪
 * 参数:
 *    epfd: epoll 实例的文件描述符
 *    events: 存储就绪事件的数组（由内核填充）
 *    maxevents: 数组最大长度（需<=预先分配的大小）
 *    timeout: 超时时间（-1 表示永久阻塞, 单位: 毫秒）
 * 返回: >0（就绪事件数量）; 0（超时）; -1（错误）
 */
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout);
```

- 使用示例

```c
int epoll_fd = epoll_create1(0);
struct epoll_event event = {
    .events = EPOLLIN | EPOLLET, // 边缘触发模式
    .data.fd = socket_fd,	 // socket_fd为要监听的描述符
};
epoll_ctl(epoll_fd, EPOLL_CTL_ADD, socket_fd, &event);

struct epoll_event events[10];
int nfds = epoll_wait(epoll_fd, events, 10, 5000); // 5秒超时
for (int i = 0; i < nfds; i++) {
	if (events[i].events & EPOLLIN) {
		// 处理读事件（需循环读取直到EAGAIN）
	}
}
```
