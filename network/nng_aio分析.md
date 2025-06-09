# nng_aio是什么

nng_aio是Nanomsg Asynchronous IO的缩写。在nng中，每个nng_aio实例都表示一个异步IO操作。发送/接收的消息nng_msg通过nng_aio传递，nng_aio内部的nng_task封装了异步操作的回调函数，由任务队列调度执行。

# nng_aio结构体

```c
struct nng_aio {
	size_t a_count;		// 记录 IO 操作中已传输的字节数
	nni_time a_expire;	// 绝对超时时间（用于超时队列的调度）
	nni_duration a_timeout; // 相对超时时间（可转换为绝对时间）

	int a_result; // 异步操作的结果码（0 表示成功，其他为 NNG 错误码，如 NNG_ETIMEDOUT）
	bool a_stop;  // 标记 aio 是否处于停止状态（阻止新操作启动）
	bool a_sleep;
	bool a_expire_ok;
	bool a_expiring;
	bool a_use_expire;

	nni_task a_task;  // 任务结构体，封装异步操作的回调函数及参数，由任务队列调度执行
	nni_iov a_iov[8]; // IO 向量数组（最多 8 个），用于支持Scatter/Gather IO
	unsigned a_nio;	  // 有效 IO 向量的数量（a_iov 中实际使用的元素数）
	nni_msg *a_msg;	  // 关联的消息对象

	void *a_inputs[4];
	void *a_outputs[4];

	nni_aio_cancel_fn a_cancel_fn; // 取消回调函数（当操作超时或被取消时触发）
	void *a_cancel_arg;	       // 取消回调函数的参数
	void *a_prov_data;
	nni_list_node a_prov_node;
	nni_aio_expire_q *a_expire_q; // 所属的超时队列
	nni_list_node a_expire_node;
	nni_reap_node a_reap_node; // 延迟回收节点（用于异步释放 aio 资源）
};
```

# nng_aio接口

```c
/* 异步操作初始化:
 *   1、初始化或分配一个 nni_aio 对象，设置回调函数 cb 和参数 arg
 *   2、将 aio 随机分配到一个超时队列，由 a_expire_q 指针记录，避免所有操作集中在一个队列
 */
void nni_aio_init(nni_aio *aio, nni_cb cb, void *arg);

/* 异步操作资源回收:
 *   1、通过 nni_reap_list 将 aio 加入延迟回收队列，避免在回调中直接 free 导致的竞态条件
 */
void nni_aio_reap(nni_aio *aio);

/* 异步操作终止:
 *   1、等待当前操作完成并阻止新操作（阻塞直到回调执行完毕）
 */
void nni_aio_stop(nni_aio *aio);

/* 异步操作关闭:
 *   1、不等待当前操作完成（非阻塞，适合快速释放资源）
 */
void nni_aio_close(nni_aio *aio);

/* 异步操作完成，通过任务队列调度回调:
 *   1、从超时队列中移除 aio（nni_aio_expire_rm）
 *   2、设置操作结果（a_result）和传输字节数（a_count）
 *   3、通过 nni_task_dispatch（异步）或 nni_task_exec（同步）触发回调
 */
void nni_aio_finish(nni_aio *aio, int result, size_t count);

/* 同步操作完成，直接执行回调 */
void nni_aio_finish_sync(nni_aio *aio, int result, size_t count);

/* 异步操作失败，返回错误码 */
void nni_aio_finish_error(nni_aio *aio, int result);

/* 完成消息传递操作，设置 a_msg 并触发回调 */
void nni_aio_finish_msg(nni_aio *aio, nni_msg *msg);

/* 异步操作强制终止:
 *   1、调用取消回调 a_cancel_fn 函数，并标记操作结果为指定错误码（如 NNG_ETIMEDOUT）
 */
void nni_aio_abort(nni_aio *, int rv);

/* 异步操作开始前的检查:
 *   1、检查 aio 是否处于停止状态（a_stop），若已停止则返回 NNG_ECANCELED，阻止无效操作
 *   2、操作前需确保 aio 未加入其它链表（nng_list），即未被其他操作占用
 */
int nni_aio_begin(nni_aio *aio);

/* 异步操作调度，注册超时回调并加入超时队列:
 *   1、将相对超时时间 a_timeout 转换为绝对时间 a_expire（若未显式设置 a_use_expire）
 *   2、调用 nni_aio_expire_add(aio) 将 aio 加入超时队列 a_expire_q->eq_list
 *   3、若操作已停止（a_stop），直接终止任务并返回 NNG_ECLOSED
 */
int nni_aio_schedule(nni_aio *aio, nni_aio_cancel_fn cancel, void *data);

```
