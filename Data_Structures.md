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

**如果想省空间，就得用时间（遍历）去换；如果想省递归栈，就得在数据结构里保留足够的路径信息（父指针）。**  


---

# 11.1-4：巨大 未初始化数组 的直接寻址实现

## 1. 问题背景
在处理超大数组（如长度 $m$ 远大于元素个数 $n$）时，系统分配的物理内存往往包含随机的**垃圾数据（Garbage）**。由于数组极大，执行 $O(m)$ 的全量初始化（如 `memset`）在性能上是不可接受的。

我们需要设计一种方案，在不预先清零的情况下，能够以 **$O(1)$** 的时间复杂度判断数组中的某个槽位到底是“真实数据”还是“内存垃圾”。

---

## 2. 核心设计：双向校验机制 (Validation Ring)
为了实现这一目标，我们需要引入两个辅助数组（大小等于最大可能的元素个数 $n$）和一个计数器：

1.  **$T$ (Huge Array)**：原始的巨大数组，存储实际的卫星数据，初始充满垃圾值。
2.  **$S$ (Stack)**：一个栈，存储当前字典中**实际存在的键（Key）**。
3.  **$V$ (Validation)**：存储键在栈 $S$ 中的**索引（Index）**。即 $V[k]$ 存储键 $k$ 在 $S$ 中的位置。
4.  **$n$**：计数器，记录当前存储的元素总数，同时作为栈顶指针。



### 验证逻辑：如何区分真伪？
一个键 $k$ 被认为是有效的，必须**同时**满足以下两个条件：
1.  **范围校验**：$0 \le V[k] < n$（索引必须在当前栈的有效范围内）。
2.  **双向确认**：$S[V[k]] == k$（“回跳”确认：从 $V$ 查到的索引去 $S$ 里找，存的必须是 $k$ 自己）。

**原理**：即使 $V[k]$ 的内存垃圾值恰好落在 $[0, n-1]$ 之间，它指向的 $S[V[k]]$ 里的值也极难恰好等于 $k$。这种“闭环”设计封死了垃圾数据伪装成合法数据的可能性。

---

## 3. 操作实现 (伪代码)

### 初始化 (Initialize)
只需要重置计数器，无需理会巨大的内存空间。
```markdown
INITIALIZE()
    n = 0
```

### 查找 (Search)
```markdown
SEARCH(T, k)
    // 双向验证
    if 0 <= V[k] < n and S[V[k]] == k
        return T[k]
    else
        return NIL
```

### 插入 (Insert)
```markdown
INSERT(T, k, x)
    if SEARCH(T, k) != NIL
        T[k] = x  // 键已存在，直接覆盖数据
    else
        T[k] = x
        V[k] = n     // 在 V 中记录该键在栈中的位置
        S[n] = k     // 将键压入栈 S
        n = n + 1    // 元素总数加 1
```

### 删除 (Delete)
为了保持栈 $S$ 的连续性，删除时需要将待删除元素与**栈顶元素**交换。
```markdown
DELETE(T, k)
    if SEARCH(T, k) == NIL
        return
    
    idx = V[k]         // 获取 k 在栈中的位置
    last_key = S[n-1]  // 获取当前的栈顶元素
    
    // 1. 将栈顶元素搬到 idx 的位置，填补被删除元素的空缺
    S[idx] = last_key
    V[last_key] = idx
    
    // 2. 逻辑上减少元素个数
    n = n - 1
```

---

## 4. 复杂度分析
* **时间复杂度**：
    * `INITIALIZE`: $O(1)$
    * `SEARCH`: $O(1)$
    * `INSERT`: $O(1)$
    * `DELETE`: $O(1)$
* **空间复杂度**：$O(m)$。虽然使用了额外数组，但每个槽位只存储 $O(1)$ 大小的索引或键。

## 5. 总结
这个设计的精髓在于**不信任任何未经验证的内存内容**。它利用了“两个互补的随机值几乎不可能形成闭环”的概率学特征（或者说确定性的逻辑验证），实现了在巨大空间内的即时存取。  

---
### If collisions are resolved by chaining, search would take $\Theta$(1 + α), where α = n / m.  

---

### **11.2-4**  
Suggest how to allocate and deallocate storage for elements *within the hash table itself* by creating a **“free list”**: a linked list of all the unused slots.

Assume that one slot can store:
- a flag, **and**
- either  
  (i) one element plus one pointer, or  
  (ii) two pointers.

All dictionary operations (INSERT, DELETE, SEARCH) and free-list management operations (e.g., `allocate()`, `deallocate()`) should run in $ O(1) $ **expected** time.

> **Question**: Does the free list need to be *doubly linked*, or does a *singly linked* free list suffice?

---

### **11.2-5**  
You need to store a set of $ n $ keys in a hash table of size $ m $. Show that:

 the keys are drawn from a universe $ U $ with  
$$
| 1) \cdot m,
$$  
then $ U $ contains a subset of size $ n $ consisting of keys that **all hash to the same slot**, under *any* deterministic hash function $ h : U \to \{0, 1, \dots, m-1\} $.

Consequently, the worst-case searching time for hashing with chaining is $ \Theta(n) $.

*(Hint: Use the pigeonhole principle.)*

---

### **11.2-6**  
You have stored $ n $ keys in a hash table of size $ m $, with collisions resolved by **chaining**, and you know:
- the length $ \ell_i $ of each chain $ i $ (for $ i = 0, 1, \dots, m-1 $),
- in particular, the length $ L = \max_i \ell_i $ of the longest chain.

Let $ \alpha = n/m $ be the load factor.

Describe a procedure that selects a key **uniformly at random** from among the $ n $ keys in the hash table and returns it in expected time  
$$
O\!\left(L \cdot \left(1 + \frac{1}{\alpha}\right)\right).
$$

Your algorithm may preprocess the chain lengths (e.g., build a cumulative array), but the *selection* step must meet the stated expected time bound.  
没问题，我为你整理了一份结构清晰的 Markdown 笔记。这份笔记涵盖了 **11.2-4** 到 **11.2-6** 的详细解析、数学证明以及复杂度推导。

---
# 下面是上述3题的解答

## 11.2-4 散列表内的存储管理（Free List）

**题目要求：** 设计一种在散列表数组内部管理空闲空间的方法，要求所有操作（分配、释放）均为 $O(1)$。

### 1. 核心设计
将散列表的数组 $T$ 视为内存池，每个槽位包含：
* **Key**：存储的数据。
* **Next**：指向散列链中下一个元素的指针。
* **Flag**：标志位，记录该槽位是“已占用”还是“空闲”。
* **自由表 (Free List)**：将所有 **Flag = 空闲** 的槽位串成一个单链表。

### 2. 操作逻辑
* **插入 (Insert)**：
    1. 从 `Free List` 的头部取出一个空闲槽位。
    2. 将数据填入该槽位，并将其挂载到对应的散列链中。
* **删除 (Delete)**：
    1. 将元素从散列链中移除。
    2. 将该槽位的 `Flag` 标为空闲，并将其插回 `Free List` 的头部。

### 3. 结论
**单链表足够。** 因为对自由表的操作仅限于“头插”和“头删”，单链表完全能以 $O(1)$ 实现。



---

## 11.2-5 最坏情况分析（鸽巢原理）

**题目要求：** 证明若全域 $|U| > (n-1)m$，则存在 $n$ 个键哈希到同一个槽位。

### 1. 证明步骤
1.  **已知条件**：散列表大小为 $m$，待存储键数为 $n$，全域大小为 $|U|$。
2.  **反证法假设**：假设对于任意槽位 $i$，映射到该槽位的键的数量 $n_i$ 都满足 $n_i \le n-1$。
3.  **求和分析**：那么全域 $U$ 中所有能被哈希处理的键的总数上限为：
    $$\sum_{i=0}^{m-1} n_i \le \sum_{i=0}^{m-1} (n-1) = m(n-1)$$
4.  **矛盾点**：题目给定 $|U| > m(n-1)$。根据**鸽巢原理**，如果我们要将超过 $m(n-1)$ 个键分配到 $m$ 个槽位中，必然至少有一个槽位分配到了至少 $n$ 个键。
5.  **结论**：在最坏情况下，链接法的搜索时间复杂度为 $\Theta(n)$。

---

## 11.2-6 均匀随机选择元素（拒绝采样法）

**题目要求：** 在已知最大链表长度 $L$ 的情况下，以 $O(L(1 + 1/\alpha))$ 的期望时间随机选出一个键。

### 1. 算法逻辑（投点法）
在一个大小为 $m \times L$ 的逻辑矩阵中均匀投点：
1.  **随机采样**：产生随机整数 $i \in [0, m-1]$（选槽位）和 $j \in [1, L]$（选高度）。
2.  **有效性检查**：
    * 若 $j \le \text{length}(T[i])$，则返回该链表的第 $j$ 个元素。
    * 若 $j > \text{length}(T[i])$，则舍弃并重试。



### 2. 复杂度推导
设定 $\alpha = n/m$（装载因子）。

* **成功概率 $P$**：
    $$P = \frac{n}{m \cdot L}$$
* **期望尝试次数**：
    $$E[\text{Attempts}] = \frac{1}{P} = \frac{mL}{n} = \frac{L}{n/m} = \frac{L}{\alpha}$$
* **单次尝试代价**：
    * 检查长度：$O(1)$。
    * 成功后定位元素：$O(L)$。
* **总期望时间 $E[T]$**：
    $$E[T] \approx (\text{期望失败次数}) \cdot O(1) + 1 \cdot O(L)$$
    $$E[T] = O\left( \frac{L}{\alpha} + L \right) = O\left( L \left( 1 + \frac{1}{\alpha} \right) \right)$$

### 3. 实例解析
若 $m=4, n=2, L=2$（2个元素全在同一个槽位）：
* $\alpha = 0.5$。
* 逻辑矩阵大小 $4 \times 2 = 8$。其中只有 2 个格子有元素。
* 成功概率 $P = 2/8 = 1/4$。
* 平均需要投点 4 次才能命中，命中后遍历链表耗时 $O(L)$。
* 总耗时正比于 $2 \times (1 + 1/0.5) = 2 \times 3 = 6$。  


# Knuth 乘法哈希法（Multiplication Method）

## 方法概述

乘法哈希法通过以下步骤计算哈希值：

1. **参数定义**：
   - `k`：待哈希的键值（key）
   - `w`：机器字长（word size，通常为32或64位）
   - `ℓ`：哈希表需要的位数
   - `m = 2^ℓ`：哈希表大小
   - `a`：一个精心选择的w位常数

2. **计算步骤**：

   a. **乘法运算**：计算 `k × a`
      - 结果是一个 `2w` 位的值
      - 表示为：`k × a = r₁ × 2^w + r₀`
      - 其中：
        - `r₁`：乘积的高w位（高位字）
        - `r₀`：乘积的低w位（低位字）

   b. **提取哈希值**：
      - 取 `r₀` 的 `ℓ` 个最高有效位（即右移`w` - `ℓ` 位后的数字）
      - 得到哈希值 `h_a(k)`

