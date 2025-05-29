# 树的遍历

树的遍历也叫树的搜索，是指按照某种规则，对树中的所有结点进行有且仅有一次的访问。

最常见的是二叉树的遍历，共有四种遍历的方式，分别是前序遍历、中序遍历、后序遍历、层次遍历。其中前、中、后序遍历为深度优先搜索，层次遍历为广度优先搜索。

- 前序遍历（深度优先）（访问顺序：根左右）（使用栈）
- 中序遍历（深度优先）（访问顺序：左根右）（使用栈）
- 后序遍历（深度优先）（访问顺序：左右根）（使用栈）
- 层次遍历（广度优先）（使用队列）

# 递归实现

栈帧的局部变量和返回值由系统栈自动保存（递归的优点是实现简单，但效率不高，且容易栈溢出）

```c++
// 前序遍历
void walk_tree(tree_node *node) {
	if (node == nullptr) {
		// do something
		return;
	}

	walk_tree(node->left);
	walk_tree(node->right);
}

// 中序遍历
void walk_tree(tree_node *node) {
	walk_tree(node->left);

	if (node == nullptr) {
		// do something
		return;
	}

	walk_tree(node->right);
}

// 后序遍历
void walk_tree(tree_node *node) {
	walk_tree(node->left);
	walk_tree(node->right);

	if (node == nullptr) {
		// do something
		return;
	}
}
```

# 非递归实现

栈帧的局部变量使用tuple保存（非递归的实现优先点效率高，但实现复杂，不容易调试和理解）

```c++
void walk_tree(tree_node *root) {
	using frame = tuple<tree_node *, bool>;
	stack<frame> st;

	st.push(frame<root, false>);
	while(!st.empty()) {
		frame fr = st.top();
		st.pop();

		tree_node *none = get<0>(fr);
		bool visited = get<1>(fr);

		if (node == nullptr)
			continue;

		if (visited == false) {
			// st.push(frame<node, true>); // 该语句置于此为后序
			st.push(frame<node->right, false>);
			// st.push(frame<node, true>); // 该语句置于此为中序
			st.push(frame<node->left, false>);
			// st.push(frame<node, true>); // 该语句置于此为前序
		} else {
			// do something
		}
	}
}
```
