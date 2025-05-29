# nng_taskq结构

```c
struct nni_taskq {
	nni_list tq_tasks;	   // taskq的任务链表
	nni_mtx tq_mtx;		   // taskq的互斥锁，对taskq进行操作时需要加锁
	nni_cv tq_sched_cv;	   // taskq调度时的条件变量，用于唤醒tq_threads中的线程
	nni_cv tq_wait_cv;	   // 暂未使用
	nni_taskq_thr *tq_threads; // taskq拥有的线程链表，taskq初始化时一次性申请tq_nthreads个线程
	int tq_nthreads;	   // taskq拥有的线程个数
	bool tq_run;		   // taskq是否处于运行态
};
```

# nni_taskq调度

- 入队

```c
void nni_task_dispatch(nni_task *task)
{
	nni_taskq *tq = task->task_tq;

	/* 没有回调直接回收资源并返回 */
	if (task->task_cb == NULL) {
		nni_task_exec(task);
		return;
	}
	nni_mtx_lock(&task->task_mtx);
	if (task->task_prep) {
		task->task_prep = false;
	} else {
		task->task_busy++;
	}
	nni_mtx_unlock(&task->task_mtx);

	nni_mtx_lock(&tq->tq_mtx);
	/* 将task加入链表 */
	nni_list_append(&tq->tq_tasks, task);
	/* 唤醒一个线程来处理该task */
	nni_cv_wake1(&tq->tq_sched_cv);
	nni_mtx_unlock(&tq->tq_mtx);
}
```

- 出队

```c
static void nni_taskq_thread(void *self)
{
	nni_taskq_thr *thr = self;
	nni_taskq *tq = thr->tqt_tq;
	nni_task *task;

	/* 设置该线程的名称 */
	nni_thr_set_name(NULL, "nng:task");

	nni_mtx_lock(&tq->tq_mtx);
	for (;;) {
		if ((task = nni_list_first(&tq->tq_tasks)) != NULL) {
			/* 取出taskq链表的第一个task */
			nni_list_remove(&tq->tq_tasks, task);
			nni_mtx_unlock(&tq->tq_mtx);

			/* 运行该task */
			task->task_cb(task->task_arg);

			nni_mtx_lock(&task->task_mtx);
			task->task_busy--;
			if (task->task_busy == 0) {
				nni_cv_wake(&task->task_cv);
			}
			nni_mtx_unlock(&task->task_mtx);

			nni_mtx_lock(&tq->tq_mtx);
			continue;
		}

		/* 在下次循环前，先判断taskq是否处于运行态 */
		if (!tq->tq_run) {
			break;
		}

		/* 阻塞当前线程，等待唤醒 */
		nni_cv_wait(&tq->tq_sched_cv);
	}
	nni_mtx_unlock(&tq->tq_mtx);
}
```
