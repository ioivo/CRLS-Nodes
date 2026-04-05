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
      - 得到哈希值 `h_a(k)  


---

## 11.3-2：长字符串的除法散列法

### 1. 题目背景
* **输入：** 一个长度为 $r$ 的字符串。
* **进制：** 将字符串视为 **128 进制**（Radix-128）的数字 $k$。
* **目标：** 计算 $h(k) = k \bmod m$。
* **约束：** * $m$ 是一个 32 位计算机字（32-bit word）。
    * 字符串 $k$ 可能非常长，远远超过一个字的存储范围。
    * 要求：**除了字符串本身，只能使用常数个字（$O(1)$ 额外空间）**。

---

### 2. 核心痛点：整型溢出
如果直接根据 128 进制定义计算 $k$：
$$k = \sum_{i=0}^{r-1} c_i \cdot 128^{r-1-i}$$
当 $r$ 较大时，$128^{r-1}$ 会迅速爆炸（例如 $128^{10} = 2^{70}$），导致计算机无法直接存储和计算这个超大整数 $k$。

---

### 3. 解题思路：霍纳法则 + 模运算性质

为了在 $O(1)$ 空间内完成计算，我们需要应用两个数学工具：

#### A. 模运算的分配律
模运算（Modulo）满足以下性质，允许我们在加法和乘法过程中**提前取模**：
1.  $(A + B) \bmod m = ((A \bmod m) + (B \bmod m)) \bmod m$
2.  $(A \cdot B) \bmod m = ((A \bmod m) \cdot (B \bmod m)) \bmod m$

#### B. 霍纳法则 (Horner's Rule)
霍纳法则是一种计算多项式的高效算法。对于 128 进制数，我们可以将其改写为**嵌套求和**的形式：
$$k = (\dots((c_0 \cdot 128 + c_1) \cdot 128 + c_2) \dots ) \cdot 128 + c_{r-1}$$



---

### 4. 算法实现步骤
结合上述原理，我们可以通过一个简单的循环，在读取字符的同时不断取模：

1.  初始化 `hash_val = 0`。
2.  遍历字符串中的每一个字符 $c_i$：
    * 将当前的 `hash_val` 乘以基数 $128$。
    * 加上当前字符的数值 $c_i$。
    * **关键：** 对结果立即执行 `% m` 操作。
3.  遍历结束后的 `hash_val` 即为最终结果。

#### 伪代码实现：
```cpp
// 假设 m 可以放入 32-bit word
int computeHash(string s, int m) {
    long long hash_val = 0; // 使用 64 位中间变量防止乘法瞬时溢出
    for (int i = 0; i < s.length(); i++) {
        // 核心递归式：h = (h * radix + char) mod m
        hash_val = (hash_val * 128 + (unsigned char)s[i]) % m;
    }
    return (int)hash_val;
}
```

---
Gemini 说
这张图片是《算法导论》（CLRS）第11章“哈希表”中关于**散列函数（Hash Functions）**的课后习题（11.3节）。

因为你没有具体指定哪一题，我把这 6 道题的核心**解题思路（Thought Process）**逐一梳理给你：

11.3-1
题目大意： 在链表中搜索带有长字符串键 k 和哈希值 h(k) 的元素。如何利用哈希值加速搜索？
核心思路：过滤（Filtering）

痛点： 长字符串的比较非常耗时，时间复杂度与字符串长度成正比 O(L)。

解法： 整数比较非常快 O(1)。当你寻找目标键 k 
target
​
  时，先计算它的哈希值 h(k 
target
​
 )。在遍历链表时，先比较哈希值。如果哈希值不同，直接跳过；只有当哈希值相同时，才去真正比较那两个长字符串。这可以极大地减少字符串比较的次数。

11.3-2
题目大意： 把一个超长字符串当作 128 进制数，用除法散列法 h(k)=kmodm。m 能装进一个 32 位字中，但字符串超长。如何在不使用大数存储的情况下计算哈希值？
核心思路：霍纳法则（Horner's Rule） + 模运算性质

数学原理： (A⋅B+C)modm=((Amodm)⋅B+C)modm。这意味着你不需要把整个大数算出来再取模，你可以边乘边加边取模。

解法： 从字符串的最高位（或者最低位，取决于定义）开始遍历每个字符 c 
i
​
 。维护一个累加器 hash_val：
hash_val = (hash_val * 128 + c_i) mod m
这样 hash_val 永远不会超过 m（即可以用 32 位存下），空间复杂度只有 O(1)。


---

## 11.3-3：特定 $m$ 值下的哈希冲突
**问题描述**：若 $m = 2^p - 1$，且字符串按 $2^p$ 进制解释。证明字符的任意排列（Permutation）都会产生相同的哈希值。

* **证明要点**：
    1.  利用同余性质：由于 $2^p \equiv 1 \pmod{2^p - 1}$，因此对于任何幂次 $i$，恒有 $(2^p)^i \equiv 1^i \equiv 1 \pmod{2^p - 1}$。
    2.  多项式展开：$h(k) = (\sum c_i (2^p)^i) \bmod m = (\sum c_i \cdot 1) \bmod m$。
    3.  **结论**：哈希值退化成了字符数值的简单求和。因为加法满足交换律，字符顺序改变不影响和，所以会发生严重冲突。
* **应用局限**：不适用于易位构词（如 "stop" 和 "pots"）较多的场景。

---

## 11.3-4：乘法散列法计算实例
**问题描述**：$m=1000$，$A = (\sqrt{5}-1)/2 \approx 0.618033$。计算 $k \in \{61, 62, 63, 64, 65\}$ 的映射位置。

* **计算公式**：$h(k) = \lfloor m (k A \bmod 1) \rfloor$
* **步骤演示 (以 k=61 为例)**：
    1.  $61 \times 0.61803398... = 37.70007...$
    2.  取小数部分：$0.70007...$
    3.  乘以 $m$：$700.07...$
    4.  向下取整：$h(61) = 700$。


---

## ★ 11.3-5：$\epsilon$-全域哈希的下界
**问题描述**：证明任何从 $U$ 到 $Q$ 的 $\epsilon$-全域哈希函数族，其碰撞率 $\epsilon \ge 1/|Q| - 1/|U|$。

* **核心逻辑**：
    * 这是一个关于“平均分配”的数学证明。
    * 即使一个哈希函数族表现得再完美，由于“鸽巢原理”，当元素个数 $|U|$ 超过槽位个数 $|Q|$ 时，必然存在碰撞。
    * 该不等式给出了碰撞概率的理论极限，说明没有任何函数族能完全消除碰撞。

---

## ★ 11.3-6：基于多项式的 $\epsilon$-全域函数族
**问题描述**：定义 $h_b(a) = (\sum a_j b^j) \bmod p$，证明其为 $\epsilon = (d-1)/p$ 的全域哈希。

* **核心思路：利用域上的多项式性质**
    1.  **冲突条件**：$h_b(a) = h_b(a')$ 等价于 $\sum (a_j - a'_j) b^j \equiv 0 \pmod p$。
    2.  **多项式根的个数**：上述等式可以看作是一个关于 $b$ 的 $d-1$ 次多项式。在素数域 $\mathbb{Z}_p$ 中，非零的 $n$ 次多项式最多有 $n$ 个根。
    3.  **概率计算**：在 $p$ 个可能的 $b$ 中，最多只有 $d-1$ 个 $b$ 会导致冲突。因此碰撞概率 $\epsilon \le (d-1)/p$。

这是一份为你整理好的《算法导论》（CLRS）第 11.4 节“开放定址法”练习题的 Markdown 学习笔记，包含了核心推导过程和最终结论，非常适合复习。

---

## 11.4-1：线性探测与双重散列模拟
**问题描述**：将键序列 `10, 22, 31, 4, 15, 28, 17, 88, 59` 插入大小为 $m = 11$ 的哈希表。

### 1. 线性探测 (Linear Probing)
* **哈希函数**：$h(k, i) = (k \bmod 11 + i) \bmod 11$
* **插入过程**：
  * 10 $\rightarrow$ `T[10]`
  * 22 $\rightarrow$ `T[0]`
  * 31 $\rightarrow$ `T[9]`
  * 4 $\rightarrow$ `T[4]`
  * 15 $\rightarrow$ 冲突于 4，顺延至 `T[5]`
  * 28 $\rightarrow$ `T[6]`
  * 17 $\rightarrow$ 冲突于 6，顺延至 `T[7]`
  * 88 $\rightarrow$ 冲突于 0，顺延至 `T[1]`
  * 59 $\rightarrow$ 冲突于 4, 5, 6, 7，顺延至 `T[8]`

**最终哈希表**：
| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 22 | 88 | | | 4 | 15 | 28 | 17 | 59 | 31 | 10 |

### 2. 双重散列 (Double Hashing)
* **哈希函数**：$h(k, i) = (h_1(k) + i \cdot h_2(k)) \bmod 11$
  * $h_1(k) = k \bmod 11$
  * $h_2(k) = 1 + (k \bmod 10)$
* **复杂冲突处理示例 (键 15)**：
  * $h_1(15)=4, h_2(15)=6$。
  * $i=0$: 4 (被占)。
  * $i=1$: $(4+6) \bmod 11 = 10$ (被占)。
  * $i=2$: $(4+12) \bmod 11 = 5 \rightarrow$ 填入 `T[5]`。

**最终哈希表**：
| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 22 | | 59 | 17 | 4 | 15 | 28 | 88 | | 31 | 10 |

---

## 11.4-2：处理 DELETED 标记的伪代码
**核心逻辑**：`DELETED` 标记在搜索时被视为“被占用的跳板”以继续向后查找，在插入时被视为“可用的空槽”。

```text
HASH-DELETE(T, k)
    i = HASH-SEARCH(T, k)
    if i != NIL
        T[i] = DELETED

HASH-SEARCH(T, k)
    i = 0
    repeat
        j = h(k, i)
        if T[j] == k
            return j
        i = i + 1
    until T[j] == NIL or i == m
    return NIL

HASH-INSERT(T, k)
    i = 0
    insert_pos = NIL
    repeat
        j = h(k, i)
        // 记录遇到的第一个 DELETED 位置
        if T[j] == DELETED and insert_pos == NIL
            insert_pos = j
        // 防止插入重复键
        if T[j] == k
            return error "key already exists" 
        i = i + 1
    until T[j] == NIL or i == m
    
    // 优先填入第一个 DELETED 位置
    if insert_pos != NIL
        T[insert_pos] = k
        return insert_pos
    elseif T[j] == NIL
        T[j] = k
        return j
    else
        return error "hash table overflow"
```

---

## 11.4-3：期望探测次数计算
**理论公式**：
* 查找失败（Unsuccessful）上界：$U = \frac{1}{1 - \alpha}$
* 查找成功（Successful）上界：$S = \frac{1}{\alpha} \ln \frac{1}{1 - \alpha}$

**代入计算**：
* **当 $\alpha = 3/4$ 时**：
  * 失败探测：$1 / (1 - 0.75) = \mathbf{4}$
  * 成功探测：$(4/3) \ln(4) \approx \mathbf{1.848}$
* **当 $\alpha = 7/8$ 时**：
  * 失败探测：$1 / (1 - 0.875) = \mathbf{8}$
  * 成功探测：$(8/7) \ln(8) \approx \mathbf{2.376}$

---

## 11.4-4：满载 ($\alpha=1$) 时的成功查找期望
**证明目标**：当 $n = m$ 时，成功查找的期望探测次数为第 $m$ 个调和数 $H_m$。

**证明过程**：
1. 哈希表中第 $k$ 个插入的元素，插入时的装载因子为 $\alpha_k = (k-1)/m$。
2. 查找成功相当于重演插入过程，第 $k$ 个元素的探测期望为：
   $$E_k = \frac{1}{1 - \alpha_k} = \frac{1}{1 - \frac{k-1}{m}} = \frac{m}{m - k + 1}$$
3. 所有元素的平均探测次数为：
   $$E = \frac{1}{m} \sum_{k=1}^{m} \frac{m}{m - k + 1}$$
4. 提取常数 $m$，并令 $j = m - k + 1$（反转求和顺序）：
   $$E = \sum_{j=1}^{m} \frac{1}{j} = H_m$$

---

## ★ 11.4-5：双重散列的循环周期证明
**问题本质**：证明当 $m$ 和步长 $h_2(k)$ 的最大公约数为 $d$ 时，探测序列只能覆盖哈希表的 $1/d$。

**证明逻辑**：
1. 探测序列回到起点，意味着总偏移量是表大小 $m$ 的倍数：
   $$i \cdot h_2(k) \equiv 0 \pmod m$$
2. 两边同除以最大公约数 $d = \gcd(m, h_2(k))$：
   $$i \cdot \frac{h_2(k)}{d} \equiv 0 \pmod{\frac{m}{d}}$$
3. 由于 $\frac{h_2(k)}{d}$ 与 $\frac{m}{d}$ 互质，$\frac{m}{d}$ 必须整除 $i$。
4. 满足条件的最小正整数 $i$ 为 $\frac{m}{d}$。这表明循环周期长度为 $m/d$，即探测序列只检查了整张表的 $1/d$。
> **推论**：为了能遍历整张表，必须保证 $d=1$，即 $m$ 和 $h_2(k)$ 必须**互质 (Relatively Prime)**。

---

## ★ 11.4-6：寻找 $U = 2S$ 的临界装载因子
**问题描述**：求满足“查找失败探测次数是查找成功两倍”的装载因子 $\alpha$。

**方程求解**：
$$\frac{1}{1 - \alpha} = 2 \left( \frac{1}{\alpha} \ln \frac{1}{1 - \alpha} \right)$$
化简得超越方程：
$$\frac{\alpha}{2(1 - \alpha)} + \ln(1 - \alpha) = 0$$

通过数值方法（如二分法）逼近：
* $\alpha = 0.7$ 时，结果 $\approx -0.038$
* $\alpha = 0.72$ 时，结果 $\approx +0.013$
* 精确近似解为 **$\alpha \approx 0.7153$**。

**工程启示**：当哈希表装载率达到约 71.5% 时，查找不存在元素的代价将是查找存在元素的严格两倍。这也是许多工业级哈希表（如 Java `HashMap`）将扩容阈值设定在 0.75 附近的理论基础之一。