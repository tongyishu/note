# nng_lmq消息队列

nng_lmq是一个轻量级的环形消息队列（FIFO）。之所以说它轻量级，是因为nng_lmq仅暴露消息队列的必要操作，不包含多线程相关的锁逻辑处理，调用者需自行处理该部分逻辑。

# nng_lmq结构体

```c
typedef struct nni_lmq {
	size_t lmq_cap;	     // 队列容量（由用户参数传入）
	size_t lmq_alloc;    // 实际分配的内存大小（向上取整为2的幂次，便于位运算）
	size_t lmq_mask;     // 位掩码(lmq_alloc - 1)，用于快速取模: x % lmq_alloc <=> x & lmq_mask
	size_t lmq_len;	     // 当前队列中消息数量
	size_t lmq_get;	     // 读索引，相当于队列头（值递增，实际索引为 lmg_get & lmq_mask）
	size_t lmq_put;	     // 写索引，相当于队列尾（值递增，实际索引为 lmg_put & lmq_mask）
	nng_msg **lmq_msgs;  // 消息指针数组（动态分配，容量>2时使用）
	nng_msg *lmq_buf[2]; // 消息指针数组（静态分配，容量<2时使用）
} nni_lmq;
```

# nng_lmq接口

```c
/* 初始化lmq，容量为cap */
void nni_lmq_init(nni_lmq *, size_t cap);

/* 销毁lmq */
void nni_lmq_fini(nni_lmq *);

/* 清空lmq */
void nni_lmq_flush(nni_lmq *);

/* 获取当前lmq队列中的消息数量 */
size_t nni_lmq_len(nni_lmq *);

/* 获取lmq的容量 */
size_t nni_lmq_cap(nni_lmq *);

/* 队列尾插入消息 */
int nni_lmq_put(nni_lmq *lmq, nng_msg *msg);

/* 队列头取出消息 */
int nni_lmq_get(nni_lmq *lmq, nng_msg **mp);

/* 重新调整队列容量为cap */
int nni_lmq_resize(nni_lmq *, size_t cap);

/* 判断队列是否满 */
bool nni_lmq_full(nni_lmq *);

/* 判断队列是否空 */
bool nni_lmq_empty(nni_lmq *);
```
