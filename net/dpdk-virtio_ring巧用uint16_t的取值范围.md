# 巧用uint16_t无符号类型的取值范围

```c
/* virtqueue_nused has load-acquire or rte_io_rmb insed */
static inline uint16_t
virtqueue_nused(const struct virtqueue *vq)
{
	uint16_t idx;

	if (vq->hw->weak_barriers) {
	/**
	 * x86 prefers to using rte_smp_rmb over rte_atomic_load_explicit as it
	 * reports a slightly better perf, which comes from the saved
	 * branch by the compiler.
	 * The if and else branches are identical with the smp and io
	 * barriers both defined as compiler barriers on x86.
	 */
#ifdef RTE_ARCH_X86_64
		idx = vq->vq_split.ring.used->idx;
		rte_smp_rmb();
#else
		idx = rte_atomic_load_explicit(&(vq)->vq_split.ring.used->idx,
				rte_memory_order_acquire);
#endif
	} else {
		idx = vq->vq_split.ring.used->idx;
		rte_io_rmb();
	}
	return idx - vq->vq_used_cons_idx;
}
```

virtio在判断使用的vring_used_elem的个数的时候，直接使用 idx – vq_used_cons_idx，其中idx是当前device消费的位置，vq_used_cons_idx是driver记录的上次消费位置。

这里我有个疑问：如果idx比vq_used_cons_idx小怎么办？

由于idx、vq_used_cons_idx及其运算结果idx – vq_used_cons_idx都是uint16_t类型，因此不会存在负数的情况。

无符号数（小 – 大）减法运算得到是两个数之间的距离，比如0 – 65535 = 1，因为 (65535 + 1) % (2^16) = 0，再比如 1 – 65535 = 2，因为(65535 + 2) % (2^16) = 1。

因此，直接相减可以得到环形队列中device已经used的元素个数。
