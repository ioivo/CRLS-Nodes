### A red-black tree is a binary search tree that satisfies the following **red-black properties**:

1. Every node is either red or black.
2. The root is black.
3. Every leaf (NIL) is black.
4. If a node is red, then both its children are black.
5. For each node, all simple paths from the node to descendant leaves contain the same number of black nodes.  

## 左旋操作

```
LEFT-ROTATE(T, x)
1  y = x.right              // 设置 y
2  x.right = y.left         // 将 y 的左子树变为 x 的右子树
3  if y.left != T.nil       // 如果 y 的左子树不为空 ...
4      y.left.p = x         // ... 则将 x 设置为该子树根节点的父节点
5  y.p = x.p                // 将 x 的父节点变为 y 的父节点
6  if x.p == T.nil          // 如果 x 原本是根节点 ...
7      T.root = y           // ... 则将 y 设置为新的根节点
8  elseif x == x.p.left     // 否则，如果 x 原本是一个左子节点 ...
9      x.p.left = y         // ... 则将 y 设置为新的左子节点
10 else x.p.right = y       // 否则，x 原本是一个右子节点，现将 y 设置为右子节点
11 y.left = x               // 将 x 变为 y 的左子节点
12 x.p = y                  // 将 y 设置为 x 的父节点
```

### 13.2-5：二叉搜索树的“右转换” (Right-Converted)

**核心定义**
如果二叉搜索树 $T_1$ 可以仅通过一系列的 **右旋 (RIGHT-ROTATE)** 操作转变为 $T_2$，则称 $T_1$ 可以被“右转换”为 $T_2$。

---

#### 1. 无法进行右转换的反例
右旋操作的一个物理前提是：**执行旋转的节点必须拥有左子树**。
* **构造 $T_1$**：一棵纯粹向右生长的链表状 BST（例如：节点 1 的右子为 2，2 的右子为 3，没有任何左子节点）。
* **构造 $T_2$**：一棵包含相同节点，但具有左子树的合法 BST（例如：节点 3 的左子为 2，2 的左子为 1）。
* **结论**：因为 $T_1$ 中没有任何节点拥有左子树，所以在 $T_1$ 上连一次右旋都无法合法执行。因此，$T_1$ 永远无法被右转换为 $T_2$。

#### 2. 可转换情况下的 $O(n^2)$ 上界证明
如果题目已知 $T_1$ 确实可以右转换为 $T_2$，我们可以采用类似 **“冒泡排序”** 的自顶向下思想来证明其旋转次数的上限为 $O(n^2)$：

* **步骤一：定位并上浮根节点。** 首先观察目标树 $T_2$ 的根节点 $r$。在初始的 $T_1$ 中，找到节点 $r$ 的位置。通过对其一系列祖先节点不断执行右旋，可以将 $r$ 一层层“冒泡”上升，直到它成为整棵树的根。由于树中最多有 $n$ 个节点，最大深度为 $n-1$，因此把一个节点上浮到根部最多需要 $O(n)$ 次右旋。
* **步骤二：冻结与递归。** 当 $r$ 成为根节点后，它的位置就已经和 $T_2$ 一致了。此时将根节点 $r$ “冻结”（不再对其参与旋转），然后对 $r$ 的左子树和右子树递归执行“步骤一”，依次把目标子树的根节点冒泡上来。
* **推导总复杂度：** 整棵树共有 $n$ 个节点。每个节点为了到达其在目标结构中的相对正确位置，最多需要经历 $O(n)$ 次右旋上浮。因此，总的右旋次数绝对不会超过 $n \times O(n) = O(n^2)$。