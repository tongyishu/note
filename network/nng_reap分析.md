# nng_reap是什么

nng_reap是延迟回收：当资源（如内存、文件描述符、对象实例）的释放可能被其他线程引用或依赖时，避免直接释放导致 `Use-After-Free`或死锁。通过将资源加入回收队列，由独立线程异步处理释放，确保所有引用操作完成后再执行释放。

nng全局有多个回收链表（nni_reap_list），通过rl_next字段链接在一起。比如: aio_reap_list、tcp_reap_list、ipc_reap_list等。
每个类型的nni_reap_list中又包含一个单向链表，存放待回收的目标结点。

示意图如下:

```bash
nni_reap_list(aio) -> nni_reap_node(aio) -> nni_reap_node(aio)
      |
      v
nni_reap_list(tcp) -> nni_reap_node(tcp) -> nni_reap_node(tcp)
      |
      v
nni_reap_list(ipc) -> nni_reap_node(ipc) -> nni_reap_node(ipc)
      |
      v
    ......
```

# nng_reap结构体

```c
struct nni_reap_node {
	nni_reap_node *rn_next; // 单向链表结点
};

struct nni_reap_list {
	nni_reap_list *rl_next;	 // 用于链接各个类型的nni_reap_list
	nni_reap_node *rl_nodes; // 用于链接当前类型的待回收对象nni_reap_node
	size_t rl_offset;	 // 在目标结构体中的偏移量
	nni_cb rl_func;		 // 回收时调用的回调函数
	bool rl_inited;		 // 是否已加入到全局回收列表
};
```

# nng_reap接口

```c
/* 
 *   1、将 rl 类型的链表加到全局链表（如果 rl_inited == false 的话）
 *   2、将 item 目标对象加入 rl_nodes 单向链表中 */
void nni_reap(nni_reap_list *rl, void *item);

/* 等待清空回收队列:
 *   1、阻塞当前线程，直到所有回收队列中的对象被处理完毕
 */
void nni_reap_drain(void);

/* reap子系统初始化:
 *   1、负责创建全局回收线程
 *   2、初始化全局回收列表
 */
int nni_reap_sys_init(void);

/* reap子系统退出:
 *   1、停止回收线程
 *   2、遍历所有子系统回收列表，强制释放剩余对象
 */
void nni_reap_sys_fini(void);
```
