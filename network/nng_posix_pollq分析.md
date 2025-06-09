# nng_posix_pollq是什么

nng_posix_pollq是IO多路复用模块，用于高效管理文件描述符（FD）的事件监听。

nng实现了4种IO多路复用：poll/epoll/kqueue/port，这里只提取公共部分，不深入实现的细节。

# nng_posix_pollq结构体

每种实现方式都包含两个结构体：nni_posix_pollq和nni_posix_pfd。

- nni_posiz_pollq全局只有一个事件管理实例：nni_posix_global_pollq。用来管理事件监听，管理线程创建销毁，管理文件描述符集合。
- nni_posix_pfd代表文件描述符，操作系统上一个int值可以表示一个文件描述符，这里nng对文件描述符做了封装。

以epoll的实现方式为例:

```c
struct nni_posix_pollq {
	nni_mtx mtx;
	int epfd;	// epoll创建的描述符，用于描述符的事件监听
	int evfd;	// eventfd创建的描述符，当有其它描述符需要回收时，通知线程回收
	bool close;	// pollq是否关闭
	nni_thr thr;	// 事件监听线程
	nni_list reapq; // 描述符回收队列
};

struct nni_posix_pfd {
	nni_list_node node;
	nni_posix_pollq *pq; // 描述符关联的pollq，指向全局唯一的nni_posix_global_pollq
	int fd;		     // 封装的操作系统描述符
	nni_posix_pfd_cb cb; // 描述符有事件到来时，执行的回调函数
	void *arg;	     // 回调函数的参数
	bool closed;	     // 描述符是否已从回收队列移除
	bool closing;	     // 描述符是否已关闭并移除监听
	bool reap;	     // 是否要回收该描述符
	unsigned events;     // 描述符监听的事件
	nni_mtx mtx;
	nni_cv cv;
};
```

# nng_posiz_pollq接口

```c
/* 创建描述符:
 *   1、使用 fd 初始化一个 nni_posix_pfd 描述符
 *   2、并将 nni_posix_pfd 描述符关联到全局管理实例 nni_posix_global_pollq
 */
int nni_posix_pfd_init(nni_posix_pfd **pfdp, int fd);

/* 销毁描述符:
 *   1、将描述符加入 reapq 队列
 *   2、向管理线程发送 event，回收该描述符
 *   3、在回收描述符之前，阻塞当前线程
 */
void nni_posix_pfd_fini(nni_posix_pfd *pfd);

/* 修改文件描述符的事件为 event */
int nni_posix_pfd_arm(nni_posix_pfd *pfd, unsigned event);

/* 返回nni_posix_pfd封装的操作系统的描述符 */
int nni_posix_pfd_fd(nni_posix_pfd *pfd);

/* 关闭描述符，并将其从监听的集合中删除 */
void nni_posix_pfd_close(nni_posix_pfd *pfd);

/* 设置监听到事件后的回调函数 cb 及参数 arg */
void nni_posix_pfd_set_cb(nni_posix_pfd *, nni_posix_pfd_cb cb, void *arg);
```
