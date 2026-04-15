
# 12.1-3

## 题目要求
设计一个非递归算法实现二叉搜索树（BST）的中序遍历。
1. **基础版**：使用栈（Stack）作为辅助数据结构。
2. **进阶版**：不使用栈，空间复杂度为 $O(1)$。假设可以测试两个指针的相等性（通常暗示存在父节点指针 `parent`）。

---

## 方案一：辅助栈法（空间复杂度 $O(n)$）

### 核心思路
中序遍历的顺序是 **左 -> 根 -> 右**。非递归实现的关键在于：利用栈来模拟递归时的“回溯”过程。
1. 将当前节点及其所有左侧后代压入栈中。
2. 当左侧为空时，弹出栈顶元素（即当前最左节点），进行访问。
3. 转向该节点的右子树，重复上述过程。

### C++ 实现
```cpp
void inorderTraversalWithStack(TreeNode* root) {
    std::stack<TreeNode*> s;
    TreeNode* curr = root;

    while (curr != nullptr || !s.empty()) {
        // 1. 尽可能向左走
        while (curr != nullptr) {
            s.push(curr);
            curr = curr->left;
        }
        
        // 2. 弹出并访问
        curr = s.top();
        s.pop();
        std::cout << curr->key << " ";
        
        // 3. 转向右子树
        curr = curr->right;
    }
}
```

---

## 方案二：父指针法（空间复杂度 $O(1)$）

### 核心思路
这是题目中提到的“优雅解法”。当节点结构中包含 `parent` 指针时，我们可以通过比较“当前节点”与“前驱节点”的位置关系，判断遍历的方向，从而摆脱对栈的依赖。

**方向判定逻辑：**
- **从父节点进入：** 优先向左走；若左子树为空，则访问自己并尝试向右走。
- **从左子树返回：** 说明左侧已处理完，访问自己并尝试向右走。
- **从右子树返回：** 说明当前子树已全部处理完，向上回溯到父节点。

### C++ 实现
```cpp
void inorderTraversalWithParent(TreeNode* root) {
    TreeNode* curr = root;
    TreeNode* prev = nullptr;
    TreeNode* next = nullptr;

    while (curr != nullptr) {
        if (prev == curr->parent) {
            // 情况 A：从上方（父节点）下来
            if (curr->left != nullptr) {
                next = curr->left; // 优先往左走
            } else {
                std::cout << curr->key << " "; // 左边没了，访问自己
                next = (curr->right != nullptr) ? curr->right : curr->parent;
            }
        } else if (prev == curr->left) {
            // 情况 B：从左子树升上来
            std::cout << curr->key << " "; // 左边处理完了，访问自己
            next = (curr->right != nullptr) ? curr->right : curr->parent;
        } else {
            // 情况 C：从右子树升上来
            next = curr->parent; // 右边处理完了，继续往上走
        }

        prev = curr;
        curr = next;
    }
}
```

---

## 总结与对比

| 特性 | 辅助栈法 | 父指针法 | Morris 遍历 (扩展) |
| :--- | :--- | :--- | :--- |
| **空间复杂度** | $O(h)$，$h$ 为树高 | $O(1)$ | $O(1)$ |
| **前提条件** | 无需额外指针 | 节点需包含 `parent` | 无需父指针，但需修改/还原叶子节点指针 |
| **适用场景** | 通用 BST 遍历 | 系统底层或内存敏感型开发 | 竞赛算法或极致空间优化 |  


---  


# BST 节点前驱与后继 (Predecessor & Successor) 总结

在二叉搜索树（BST）中，前驱和后继定义了中序遍历序列中某个节点的相邻关系。

## 1. 核心定义
* **Successor (后继)**：中序遍历中紧跟在 $x$ 之后的节点。即：键值大于 $x$ 的所有节点中最小的一个。
* **Predecessor (前驱)**：中序遍历中排在 $x$ 之前的节点。即：键值小于 $x$ 的所有节点中最大的一个。

---

## 2. 寻找逻辑

### A. 寻找后继 (Successor)
1.  **存在右子树**：后继是其右子树中的**最小值**。
    * *操作*：`node = node->right`，然后一直向左走到底。
2.  **不存在右子树**：后继是其某个祖先。
    * *操作*：向上回溯，直到找到一个节点，它是其父节点的**左孩子**。该父节点即为后继。



### B. 寻找前驱 (Predecessor)
1.  **存在左子树**：前驱是其左子树中的**最大值**。
    * *操作*：`node = node->left`，然后一直向右走到底。
2.  **不存在左子树**：前驱是其某个祖先。
    * *操作*：向上回溯，直到找到一个节点，它是其父节点的**右孩子**。该父节点即为前驱。

---

## 3. 关键性质
* **复杂度**：单次寻找操作的时间复杂度为 $O(h)$，其中 $h$ 是树的高度。
* **全遍历性能**：从最小值开始连续调用 $n-1$ 次 Successor 遍历整棵树，总时间复杂度为 **$\Theta(n)$**（摊还分析：每条边最多被访问两次）。
* **叶子节点性质**：对于叶子节点，其父节点要么是它的前驱，要么是它的后继。
* **双分支节点性质**：如果一个节点有两个孩子，其后继一定没有左孩子，前驱一定没有右孩子。

---

## 4. C++ 代码参考 (带 Parent 指针)

```cpp
// 寻找后继节点
TreeNode* getSuccessor(TreeNode* x) {
    if (x == nullptr) return nullptr;

    // 情况 1: 有右子树
    if (x->right != nullptr) {
        x = x->right;
        while (x->left != nullptr) x = x->left;
        return x;
    }

    // 情况 2: 无右子树，向上找
    TreeNode* y = x->parent;
    while (y != nullptr && x == y->right) {
        x = y;
        y = y->parent;
    }
    return y;
}

// 寻找前驱节点 (对称逻辑)
TreeNode* getPredecessor(TreeNode* x) {
    if (x == nullptr) return nullptr;

    if (x->left != nullptr) {
        x = x->left;
        while (x->right != nullptr) x = x->right;
        return x;
    }

    TreeNode* y = x->parent;
    while (y != nullptr && x == y->left) {
        x = y;
        y = y->parent;
    }
    return y;
}
```

---

## 5. 常见面试考点
> **Q：如果在没有 `parent` 指针的情况下找后继怎么办？**
>
> **A**：需要从根节点（Root）开始搜索。
> 1. 如果目标值小于当前节点，当前节点可能是后继，记录之并向左找。
> 2. 如果目标值大于当前节点，向右找。
> 3. 直到找到目标节点，若它有右子树，则直接找右子树最小值；否则，之前记录的最后一个“向左转”时的节点就是后继。