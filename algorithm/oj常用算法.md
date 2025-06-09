# 回溯算法

回溯算法实际是一个类似枚举的搜索尝试过程，主要是在搜索尝试过程中寻找问题的解。当发现已不满足求解条件时，就回退一步重新选择，尝试别的路径。

```bash
backtrace()
{
	if (V is a feasible solution)			// V是一个可行解
		output V				// 输出V
	else
		for K in each possible value		// 对每个可能的取值K进行遍历
			if K satisfy constraints	// 如果K满足约束条件
				backtrace(K)		// 进行下一层递归
				recover context		// 恢复上下文并进行回溯
}
```

# 拓扑排序

拓扑排序主要用来解决具有依赖性关系的问题（先解决问题的依赖，再解决该问题）。

求解拓扑排序的一般步骤如下：

1. 找到所有入度为0的顶点
2. 删除这些顶点的出度
3. 重复以上步骤

```bash
topsort()
{
	queue<vertex> q;
	int counter = 0;

	q.make_empty();
	for each vertex v
		if (v.indegree == 0)
			q.enqueue(v);

	while (!q.empty()) {
		v = q.dequeue();
		counter++;

		for each vertex w adjacent to v
			if (--w.indegree == 0)
				q.enqueue();
	}

	if (counter != VERTEX_NUM)
		throw cycle_found_exception();
}


```

# 动态规划

动态规则的本质还是递归，只是记录了中间过程的计算结果，用空间换时间，其理论基础为数学归纳法。

解决问题的步骤如下：

1. 确定初始值
2. 寻找递推关系，建立 dp[i][j] 与 { dp[i-1][j]、dp[i][j-1]、dp[i-1][j-1] } 的关系
3. 存储中间结果以减少重复计算

一般采用数组存储中间结果，在填充数组的过程中，要注意必须使用现有的结果进行计算并填充。比如：

dp[i][j] 依赖于 dp[i-1][j]，则需要先计算 dp[i-1][j]，再计算 dp[i][j]
