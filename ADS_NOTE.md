# 第 17 章 扩充数据结构 (Augmenting Data Structures)

## 核心思想
大多数情况下不需要发明全新的数据结构，而是在**教科书级数据结构**（如红黑树）上**存储额外信息**来支持新的操作。关键挑战：额外信息必须在普通操作（插入、删除）中高效维护。

---

## 17.1 动态顺序统计 (Dynamic Order Statistics)

### 问题
在**动态集合**上快速查询：
- 第 i 小的元素 (OS-SELECT)
- 给定元素的排名 (OS-RANK)

目标：O(lg n) 时间。

### 数据结构：顺序统计树 (Order-Statistic Tree)

**底层**：红黑树

**额外信息**：`x.size` = 以 x 为根的子树中**内部节点**的数量（不含哨兵 T.nil）

```
x.size = x.left.size + x.right.size + 1
T.nil.size = 0
```

**秩的定义**：当存在重复键值时，秩定义为**中序遍历**中的位置，消除歧义。

### OS-SELECT(x, i) — 找第 i 小元素

```pseudocode
OS-SELECT(x, i)
1   r = x.left.size + 1        // x 在自己子树中的排名
2   if i == r
3       return x
4   elseif i < r
5       return OS-SELECT(x.left, i)
6   else return OS-SELECT(x.right, i - r)
```

- 第 1 行：计算 x 的局部排名 r
- 若 i == r：找到，返回 x
- 若 i < r：目标在左子树
- 若 i > r：目标在右子树，且是右子树中第 (i − r) 小的元素

**时间复杂度**：O(lg n)，每次递归下降一层。

### OS-RANK(T, x) — 求元素 x 的排名

```pseudocode
OS-RANK(T, x)
1   r = x.left.size + 1        // x 在自己子树中的局部排名
2   y = x
3   while y ≠ T.root
4       if y == y.p.right      // 如果 y 是右孩子
5           r = r + y.p.left.size + 1   // 加上父节点及左子树
6       y = y.p
7   return r
```

**直觉**：从 x 向上爬回根。
- 若当前节点 y 是**右孩子**，说明父节点和父节点的整个左子树都在 y 前面，把它们都加进 r。
- 若 y 是**左孩子**，父节点和右子树都在后面，不用加。

**时间复杂度**：O(lg n)，每次迭代上移一层。

### 维护 size 属性

**插入**：
- 第一阶段（向下走）：沿路径每经过一个节点，`x.size++`，O(lg n)。
- 第二阶段（旋转）：旋转后只影响两个节点，在 LEFT-ROTATE 末尾加上：
  ```
  y.size = x.size
  x.size = x.left.size + x.right.size + 1
  ```
  每次旋转 O(1)，总共最多 2 次旋转，所以 O(1)。

**删除**：
- 第一阶段：从被移动的最底层节点向上走到根，沿途 `size--`，O(lg n)。
- 第二阶段：处理旋转同上，O(1)。

**结论**：维护 size 不改变插入/删除的 O(lg n) 渐近时间。

---

## 17.2 如何扩充数据结构 (How to Augment a Data Structure)

### 四步法

1. **选择底层数据结构**
   - 通常选红黑树（支持全序操作：MIN, MAX, SUCCESSOR, PREDECESSOR）

2. **确定额外信息**
   - 使新操作更高效的信息
   - 例：顺序统计树的 `size`；区间树的 `max`

3. **验证额外信息可维护**
   - 插入/删除时能否高效更新额外信息
   - 理想情况：只有 O(lg n) 个节点的信息需要更新

4. **开发新操作**
   - 这是扩充数据结构的最终目的

实际设计过程通常是迭代的（4 步并行推进）。

### 定理 17.1（扩充红黑树定理）

> 设 f 是红黑树 T 中每个节点上的一个属性，且对于任意节点 x，`x.f` 只依赖于 x 自身、x.left 和 x.right 的信息（包括它们的 f 值），且 `x.f` 可在 O(1) 时间内算出。则插入和删除操作可以在不改变 O(lg n) 运行时间的前提下，维护 T 中所有节点的 f 值。

**证明要点**：
- 改变一个节点的 f 只会影响其**祖先节点**
- 红黑树高 O(lg n)，传播代价 O(lg n)
- 插入/删除中的 2~3 次旋转，每次旋转传播代价 O(lg n)
- 总时间仍为 O(lg n)

**重要**：很多情况下（如 size, max）旋转后更新是 O(1) 而非 O(lg n)。但如果每次旋转都需要 O(lg n)，就需要保证旋转次数是常数（红黑树满足这一点；部分其他平衡树不满足）。

---

## 17.3 区间树 (Interval Trees)

### 问题
维护一组**动态区间**，支持：
- INTERVAL-INSERT(T, x)
- INTERVAL-DELETE(T, x)
- INTERVAL-SEARCH(T, i)：返回树中任意一个与区间 i 重叠的区间，或 T.nil

### 区间基础知识

- 区间 i = [t1, t2]，`i.low = t1`，`i.high = t2`
- 两个区间 i 和 i' **重叠** ⟺ `i.low ≤ i'.high` 且 `i'.low ≤ i.high`

**区间三分法 (Interval Trichotomy)**：任意两个区间 i 和 i' 满足恰好下列之一：
a. 重叠 (overlap)
b. i 在 i' 左侧：`i.high < i'.low`
c. i 在 i' 右侧：`i'.high < i.low`

### 设计区间树（四步法）

#### 第 1 步：底层数据结构
- 红黑树
- **键 = 区间的低端点** `x.int.low`（中序遍历 = 按低端点排序）

#### 第 2 步：附加信息
- `x.max =` 以 x 为根的子树中**所有区间端点的最大值**

```
x.max = max(x.int.high, x.left.max, x.right.max)
```

#### 第 3 步：维护信息
- `x.max` 只依赖自身及左右孩子 → O(1) 可算
- 由定理 17.1，插入/删除 O(lg n) 可维护
- 旋转后只需 O(1) 更新两个受影响节点的 max（练习 17.3-1）

#### 第 4 步：INTERVAL-SEARCH(T, i)

```pseudocode
INTERVAL-SEARCH(T, i)
1   x = T.root
2   while x ≠ T.nil and i does not overlap x.int
3       if x.left ≠ T.nil and x.left.max ≥ i.low
4           x = x.left
5       else x = x.right
6   return x
```

**解释**：
- 第 2 行：循环直到找到重叠或走到哨兵
- 第 3 行：条件 `x.left.max ≥ i.low` 意味着「左子树中存在某个区间的高端点 ≥ i.low」
  - 若成立 → 走左边（安全方向，即使左边没有重叠，右边也肯定没有）
  - 若失败 → 走右边（此时左子树所有区间高 < i.low，不可能重叠）
- 搜索只沿**一条**从根到叶的路径，O(lg n) 时间

### 定理 17.2（正确性证明）

两种情况：

**A. 搜索向右**（`x.left == nil` 或 `x.left.max < i.low`）
- 左子树中任意区间 i' 满足 `i'.high ≤ x.left.max < i.low`
- 所以 i' 完全在 i 左边，不重叠 ✓

**B. 搜索向左**（`x.left ≠ nil` 且 `x.left.max ≥ i.low`）
- 左子树中存在区间 i' 满足 `i'.high = x.left.max ≥ i.low`
- 若 i' 不与 i 重叠 → 由三分法知 `i.high < i'.low`
- 又因 i' 在左子树：`i'.low ≤ x.int.low`
- 对右子树任意 i''：`x.int.low ≤ i''.low`
- 连起来：`i.high < i'.low ≤ x.int.low ≤ i''.low` → i 在 i'' 左边，不重叠 ✓

无论走左还是走右，都不会错过另一侧可能的重叠区间。

---

## 精选练习题

### 17.1-3 非递归版 OS-SELECT

```pseudocode
OS-SELECT-ITERATIVE(T, i)
    x = T.root
    while x ≠ T.nil
        r = x.left.size + 1
        if i == r
            return x
        elseif i < r
            x = x.left
        else
            x = x.right
            i = i - r
    return T.nil
```

### 17.1-4 OS-KEY-RANK(T, k)
在键值不重复的顺序统计树中，根据键值 k 找到其排名：

```
OS-KEY-RANK(T, k)
    x = T.root
    rank = 0
    while x ≠ T.nil
        if k == x.key
            return rank + x.left.size + 1
        elseif k < x.key
            x = x.left
        else
            rank = rank + x.left.size + 1
            x = x.right
    return -1   // 未找到
```

### 17.1-7 用顺序统计树数逆序对 O(n lg n)

逆序对定义：i < j 但 A[i] > A[j]。算法：
1. 初始化空顺序统计树
2. 从后向前遍历数组（j = n down to 1）：
   - 插入 A[j] 到树中
   - A[j] 的逆序对贡献 = 当前树中所有严格小于 A[j] 的元素个数 = OS-RANK 相关
3. 更常见做法：从前向后遍历，对每个元素 A[i]，查询已插入元素中有多少个大于它 → `i - 1 - (rank of A[i] - 1)`，或者直接用 `x.size - rank`

总时间：n 次插入 + n 次排名查询 = O(n lg n)。

### 17.2-3 旋转后 O(1) 更新子树的聚合值
若每个节点 x 的 `x.f` 是中序遍历子树节点的聚合（用结合算子 ⊗），则：
- 旋转后只需重算 x 和 y 的 f 值
- `x.f = x.left.f ⊗ x.a ⊗ x.right.f`（O(1)）
- 因为只有受影响的两个节点的子节点变了

### 17.3-1 LEFT-ROTATE 更新 max（区间树）

```pseudocode
LEFT-ROTATE(T, x)
    y = x.right
    x.right = y.left
    if y.left ≠ T.nil
        y.left.p = x
    y.p = x.p
    if x.p == T.nil
        T.root = y
    elseif x == x.p.left
        x.p.left = y
    else x.p.right = y
    y.left = x
    x.p = y
    // 更新 max（顺序不能反：先孩子后父亲）
    x.max = max(x.int.high, x.left.max, x.right.max)
    y.max = max(y.int.high, y.left.max, y.right.max)
```

### 17.3-5 MIN-GAP 问题

用红黑树维护动态数集 Q，额外属性：
- `x.min_val`：子树最小值
- `x.max_val`：子树最大值
- `x.min_gap`：子树内相邻元素的最小差值

递归计算：
```
x.min_gap = min(
    x.left.min_gap,
    x.right.min_gap,
    x.key - x.left.max_val       (如果左子树非空),
    x.right.min_val - x.key      (如果右子树非空)
)
```

所有操作 O(lg n)，MIN-GAP 直接读根节点的 `min_gap`，O(1)。

---

## 本章核心总结

| 结构 | 底层 | 额外信息 | 支持的新操作 | 时间 |
|------|------|----------|-------------|------|
| 顺序统计树 | 红黑树 | x.size | OS-SELECT, OS-RANK | O(lg n) |
| 区间树 | 红黑树（按 low 排序） | x.max | INTERVAL-SEARCH | O(lg n) |

**四步扩充法** + **定理 17.1** = 在红黑树上扩充信息的通用方法论。

**关键直觉**：
- size 让「第几小」变成二分搜索
- max 让「可能重叠」变成单路径搜索
- 两者都只需维护 O(lg n) 个节点就能在动态变化中保持正确