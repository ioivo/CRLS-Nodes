# **Quicksort**
## 7.1
```
PARTITION(A, p, r)
1 x = A[r]
2 i = p – 1
3 for j = p to r – 1
4 if A[j] ≤ x
5   i = i + 1
6   exchange A[i] with A[j]
7 exchange A[i + 1] with A[r]
8 return i + 1  
```
### 7.1-2
What value of q does PARTITION return when all elements in the
subarray A[p : r] have the same value? Modify PARTITION so that q =
⌊
(p + r)/2
⌋
 when all elements in the subarray A[p : r] have the same
value  
ans : 碰到A[j] == x的情况，利用计数器在只有一半情况下执行`i += 1`并交换

### 7.1-4
Modify QUICKSORT to sort into monotonically decreasing order.

A[j] <= x 改为 A[j] <= x即可

## 7.2
the running time of
quicksort, when levels alternate between good and bad splits, is like the
running time for good splits alone: still O(n lg n), but with a slightly
larger constant hidden by the O-notation.
（平均时间接近best case）

对于已排序的，倒序的，元素全部一样的，快排的时间复杂度退化至$\Theta(n^2)$
