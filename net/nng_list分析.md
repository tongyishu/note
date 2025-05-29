# nng_list结构体

```c
/* 链表结点 */
typedef struct nni_list_node {
	struct nni_list_node *ln_next; // 记录下一个node
	struct nni_list_node *ln_prev; // 记录上一个node
} nni_list_node;

/* 双向链表 */
typedef struct nni_list {
	struct nni_list_node ll_head; // 头结点（哨兵结点）
	size_t ll_offset; // 用户结构体中nni_list_node成员的偏移量（类似内核的container_of宏）
} nni_list;
```

# nng_list及接口

```c

/* NODE宏通过 item_pointer 找到 node_pointer */
#define NODE(list, item) (nni_list_node *)(void *)(((char *)item) + list->ll_offset)

/* ITEM宏通过 node_pointer 找到 item_pointer */
#define ITEM(list, node) (void *)(((char *)node) - list->ll_offset)

/* 宏封装初始化函数，通过offsetof(type, field)自动计算偏移量
 * type是用户结构体类型，field是用户结构体中nni_list_node成员的名字 */
void nni_list_init_offset(nni_list *list, size_t offset);
#define NNI_LIST_INIT(list, type, field) \
    nni_list_init_offset(list, offsetof(type, field))

/* 静态初始化宏 */
#define NNI_LIST_INITIALIZER(list, type, field)          \
    {                                                    \
        .ll_head.ln_next = &(list).ll_head,              \
        .ll_head.ln_prev = &(list).ll_head,              \
        .ll_offset       = offsetof(type, field)         \
    }

/* 链表结点初始化宏，将前后指针置空表示节点未被插入链表 */
#define NNI_LIST_NODE_INIT(node)                       \
	do {                                           \
		(node)->ln_prev = (node)->ln_next = 0; \
	} while (0)

/* 获取链表第一个有效元素（头节点的下一个节点） */
void *nni_list_first(const nni_list *list);

/**/
void *nni_list_last(const nni_list *list);

/* 将 item 追加到链表尾部 */
void nni_list_append(nni_list *list, void *item);

/* 将 item 插入到链表头部 */
void nni_list_prepend(nni_list *list, void *item);

/* 在参考节点 before 前插入 item */
void nni_list_insert_before(nni_list *list, void *item, void *before);

/* 在参考节点 after 后插入 item */
void nni_list_insert_after(nni_list *list, void *item, void *after);

/* 获取 item 的下一个元素 */
void *nni_list_next(const nni_list *list, void *item);

/* 获取 item 的上一个元素 */
void *nni_list_prev(const nni_list *list, void *item);

/* 从链表中移除 item */
void nni_list_remove(nni_list *list, void *item);

/* 检查 item 是否在链表 list 中 */
int nni_list_active(nni_list *list, void *item);

/* 检查链表是否为空 */
int nni_list_empty(nni_list *list);

/* 检查 node 是否在任一链表中 */
int nni_list_node_active(nni_list_node *node);

/* 直接通过 node 指针将 node 从链表中移除 */
void nni_list_node_remove(nni_list_node *node);

/* 提供类似循环的语法，简化链表遍历操作（it 是用户结构体指针） */
#define NNI_LIST_FOREACH(l, it) \
	for (it = nni_list_first(l); it != NULL; it = nni_list_next(l, it))
```
