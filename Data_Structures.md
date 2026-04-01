### 10.1-7 用两个stack实现一个queue  
```
Initialize: S1 = empty stack, S2 = empty stack

ENQUEUE(x):
    PUSH(S1, x)

DEQUEUE():
    if EMPTY(S2):
        if EMPTY(S1):
            error "queue underflow"
        while not EMPTY(S1):
            PUSH(S2, POP(S1))
    return POP(S2)
```

### 10.1-8 用两个queue实现一个stack  
```
Initialize: Q1 = empty queue, Q2 = empty queue

PUSH(x):
    ENQUEUE(Q2, x)        // 新元素入 Q2
    while not EMPTY(Q1):
        // 将 Q1 的所有元素移到 Q2
        ENQUEUE(Q2, DEQUEUE(Q1))
    swap(Q1, Q2)                  
    // 交换 Q1 和 Q2，Q1 现在包含正确顺序的栈

POP():
    if EMPTY(Q1): error "stack underflow"
    return DEQUEUE(Q1)            
    // 从 Q1 队头弹出
```  
### 10.2-5反转单向链表  
三指针  

没问题，针对 **CLRS 10.2-6 异或链表 (XOR Linked List)** 的专项分析，这里为你整理了一份详细的 Markdown 笔记。

---

# 10.2-6：异或双向链表 (XOR Linked List)

## 1. 核心概念
传统的双向链表每个节点需要两个指针（`prev` 和 `next`）。异或链表通过位运算，将这两个指针压缩为一个值 `np`：
$$x.np = x.prev \oplus x.next$$

### 数学基础
利用异或运算（XOR）的对称性：
1. $A \oplus A = 0$
2. $A \oplus 0 = A$
3. $A \oplus (A \oplus B) = B$
4. $B \oplus (A \oplus B) = A$



---

## 2. 访问与遍历 (SEARCH)
由于每个节点只存了异或后的值，我们无法直接通过一个节点得到它的邻居。
**要求**：遍历时必须始终持有**两个连续节点**的地址。

* **已知条件**：当前节点 `curr`，前驱节点 `prev`。
* **计算后继**：$next = curr.np \oplus prev$
* **计算前驱**（反向遍历时）：$prev = curr.np \oplus next$

**注意**：访问表头时，由于 `head.prev` 为 `NIL` (0)，所以 `head.next = head.np \oplus 0 = head.np`。

---

## 3. 插入操作 (INSERT)
在表头插入新节点 `x`：

1. **初始化新节点**：`x.np = 0 ^ L.head`（即 `x` 的前驱是 `NIL`，后继是原头节点）。
2. **更新原头节点**：
   * 原头节点原本存的是 `0 ^ head.next`。
   * 现在它的前驱变成了 `x`，所以新的 `head.np = x ^ head.next`。
   * 计算方法：`head.np = x ^ (0 ^ head.np)`。
3. **更新链表属性**：`L.head = x`。

---

## 4. 删除操作 (DELETE)
假设要删除节点 $x$，通过遍历我们已知其前驱 $A$ 和后继 $B$。

**基本逻辑**：在邻居的 `np` 中，用“对方”替换掉“待删节点 $x$”。

### A. 修改前驱 A 的 `np`
* **原状态**：$A.np = A.prev \oplus x$
* **新状态**：$A.np = A.prev \oplus B$
* **计算公式**：$A.np = (A.np \oplus x) \oplus B$

### B. 修改后继 B 的 `np`
* **原状态**：$B.np = x \oplus B.next$
* **新状态**：$B.np = A \oplus B.next$
* **计算公式**：$B.np = (B.np \oplus x) \oplus A$



---

## 5. 逆置操作 (REVERSE)
这是异或链表最强大的地方。在普通双链表中，逆置需要 $O(n)$ 时间修改所有指针；而在异或链表中：

* **时间复杂度**：$O(1)$
* **操作**：仅需交换链表的 `head` 指针和 `tail` 指针。
* **原理**：由于 $prev \oplus next$ 与 $next \oplus prev$ 的结果完全相同，节点的存储信息对于两个方向是天然对称的。  

---

### 10.3-3: nonrecursive 历遍二叉树  
* 思路：用一个stack辅助
```
ITERATIVE-TREE-TRAVERSAL(T)
    if T.root == NIL
        return
    
    S = PUSH(EMPTY-STACK(), T.root)  // 初始化栈并压入根节点
    
    while STACK-EMPTY(S) == FALSE
        x = POP(S)                   // 弹出当前待处理节点
        print x.key                  // 访问该节点
        
        // 注意：先压右再压左，保证左孩子先出栈
        if x.right != NIL
            PUSH(S, x.right)
            
        if x.left != NIL
            PUSH(S, x.left)
```  
没问题，已经为你整理好了这两道题的详细思路和伪代码。你可以直接复制到你的笔记工具中。

---



### 10.3-5：$O(1)$ 额外空间的非递归遍历

**题目要求：** 写出一个 $O(n)$ 时间的非递归过程，打印二叉树每个节点的键值。要求只能使用 $O(1)$ 的额外空间，且过程中不得修改树的结构。

**核心思路：状态机思想**
在不使用辅助栈且不修改树（如 Morris 遍历）的情况下，要实现回溯，前提是节点必须包含**父节点指针**（即节点结构为 `left`, `right`, `p`）。
我们通过维护两个变量 `curr`（当前节点）和 `prev`（上一个访问的节点）来模拟递归栈的行为。通过比较两者关系，判断当前的“移动方向”：

1.  **向下移动**：`prev` 是 `curr.p`。说明刚进入子树，优先访问左孩子。
2.  **向右/向上移动**：`prev` 是 `curr.left`。说明左子树已处理完，开始访问右孩子；若无右孩子，则向上回溯。
3.  **向上移动**：`prev` 是 `curr.right`。说明左右子树均已处理完，直接向上回溯。



**伪代码实现（以先序遍历为例）：**

```markdown
NONRECURSIVE-TRAVERSAL(T)
    curr = T.root
    prev = NIL
    
    while curr != NIL
        // 情况 A：从父节点进入当前节点（初次到达）
        if prev == curr.p
            print curr.key            // 在此处打印即为先序
            if curr.left != NIL
                next = curr.left
            else if curr.right != NIL
                next = curr.right
            else
                next = curr.p
                
        // 情况 B：从左孩子回溯上来
        else if prev == curr.left
            if curr.right != NIL
                next = curr.right
            else
                next = curr.p
                
        // 情况 C：从右孩子回溯上来
        else
            next = curr.p
        
        // 状态更新：准备下一次循环
        prev = curr
        curr = next
```

---

### 10.3-6：空间的极致压缩（左孩子右兄弟表示法）

**题目要求：** 原始的“左孩子右兄弟”表示法每个节点需三个指针。请设计一种方案，仅使用**两个指针**和**一个布尔值**，使得在 $O(\text{degree})$ 时间内能找到父节点或所有孩子。

**设计方案：指针复用与标记位**
我们将每个节点的两个指针定义为 `p1` 和 `p2`，布尔值定义为 `is_last`。

* **`p1` (left-child)**：始终指向该节点的第一个孩子。如果没有孩子，则为 `NIL`。
* **`p2` (sibling-or-parent)**：
    * 若 `is_last == False`：`p2` 指向下一个**兄弟节点**（正常的 `right-sibling`）。
    * 若 `is_last == True`：`p2` 指向该节点的**父节点**。
* **`is_last`**：布尔值，标识该节点是否为父节点的最后一个孩子。



**操作逻辑实现：**

1.  **获取所有孩子 (Find All Children)**：
    * **步骤**：从当前节点 `x` 的 `p1`（第一个孩子）开始，通过 `p2` 不断向右遍历兄弟。
    * **终止条件**：直到遇到一个节点其 `is_last == True`。
    * **时间复杂度**：$O(k)$，其中 $k$ 为孩子数量。

2.  **获取父节点 (Find Parent)**：
    * **步骤**：
        1.  检查当前节点 `x`。若 `x.is_last == True`，则其 `p2` 直接就是父节点。
        2.  若 `x.is_last == False`，则沿着 `p2` 向右遍历其后续兄弟。
        3.  直到找到某个兄弟 `y` 满足 `y.is_last == True`，此时 `y.p2` 即为父节点。
    * **时间复杂度**：$O(k)$，在最坏情况下（访问第一个孩子时找父节点）需要走完所有兄弟。

**总结比较：**

| 属性 | 原始表示法 | 压缩表示法 |
| :--- | :--- | :--- |
| **空间消耗** | 3 指针 | 2 指针 + 1 bit |
| **找孩子时间** | $O(k)$ | $O(k)$ |
| **找父节点时间** | $O(1)$ | $O(k)$ |

---

这两道题的精髓都在于：**如果你想省空间，就得用时间（遍历）去换；如果你想省递归栈，就得在数据结构里保留足够的路径信息（父指针）。**

你在实现 10.3-5 的逻辑时，有没有遇到过 `prev` 和 `curr` 状态判断死循环的情况？如果有，通常是因为 `NIL` 的处理边界没对齐。