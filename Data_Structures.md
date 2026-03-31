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