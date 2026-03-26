# Selection problem  
### Input : n 个 不同元素的集合 A 和一个 整数 i ($1 \le i \le n$)  
### Output : 第 i 大的元素  

## 9.1 如何通过比较找出集合中的最大和最小值？  
**Ans :  Processing elements in pairs.**  
选取第一个元素为max和min (n is odd) 或第一第二个元素为max和
min (n is even), 然后每次处理两个元素，其中较大数和max比较，较小的数和min比较，at a cost of 3 comparisons for every 2 elements.  
**The total number of comparisons is at most 3 ⌊n/2⌋**

### 9.1-1 : 找到第二小元素需要的比较次数？  
**Ans : $(n - 1) + (\lceil \lg n \rceil - 1) = n + \lceil \lg n \rceil - 2$**  
建立败者数，叶节点数量为n  

### 9.1-4：同时找最大和最小值的下界

证明在最坏情况下，同时找到 $n$ 个数的最大值和最小值至少需要 $\lceil 3n/2 \rceil - 2$ 次比较。

## 证明思路：状态追踪法

每个元素在比较过程中可以处于以下四种状态之一：

*   **U (Unknown)**：尚未参加过比较。
*   **M (Potential Min)**：只输过，没赢过（可能是最小值）。
*   **X (Potential Max)**：只赢过，没输过（可能是最大值）。
*   **N (Neither)**：既赢过也输过（绝不可能是最值）。

**目标**：初始时所有 $n$ 个元素都在 U 状态。最终必须达到只有 1 个元素在 M 状态，1 个元素在 X 状态，其余都在 N 状态。

## 比较的效率

1.  **U vs U（最划算）**：1 次比较让一个进入 M，一个进入 X。这能处理 2 个元素。
2.  **M vs M**：1 次比较将其中一个移出 M 进入 N。
3.  **X vs X**：1 次比较将其中一个移出 X 进入 N。

## 计算（以 $n$ 为偶数为例）

*   首先进行 $n/2$ 次 U vs U 比较，得到 $n/2$ 个 M 和 $n/2$ 个 X。
*   为了从 $n/2$ 个 M 中筛选出唯一的最小值，还需要 $(n/2) - 1$ 次比较。
*   为了从 $n/2$ 个 X 中筛选出最大值，还需要 $(n/2) - 1$ 次比较。
*   总计：$(n/2) + (n/2 - 1) + (n/2 - 1) = 3n/2 - 2$。