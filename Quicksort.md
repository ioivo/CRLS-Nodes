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
subarray A[p : r] have the same value? Modify PARTITION so that q =⌊(p + r)/2⌋ when all elements in the subarray A[p : r] have the same value?  
*Ans : 碰到 `A[j] == x` 的情况，利用计数器在只有一半情况下执行`i += 1`并交换*

### 7.1-4
Modify QUICKSORT to sort into monotonically decreasing order.

`A[j] <= x` 改为 `A[j] >= x` 即可

## 7.2
the running time of quicksort, when levels alternate between good and bad splits, is like the running time for good splits alone: still O(n lg n), but with a slightly larger constant hidden by the O-notation.（平均时间接近best case）

对于已排序的，倒序的，元素全部一样的array，快排的时间复杂度退化至$\Theta(n^2)$

## 7.3

### RANDOMIZED-PARTITION(A, p, r)
1  $i = \text{RANDOM}(p, r)$  
2  exchange $A[r]$ with $A[i]$  
3  **return** $\text{PARTITION}(A, p, r)$

### RANDOMIZED-QUICKSORT(A, p, r)
1  **if** $p < r$  
2      $q = \text{RANDOMIZED-PARTITION}(A, p, r)$  
3      $\text{RANDOMIZED-QUICKSORT}(A, p, q - 1)$  
4      $\text{RANDOMIZED-QUICKSORT}(A, q + 1, r)$

### 7.3-2  
When RANDOMIZED-QUICKSORT runs, how many calls are made
to the random-number generator RANDOM in the worst case? How
about in the best case? Give your answer in terms of 
Θ-notation.  
*Ans : 都是$\Theta(n)$. 因为最好情况是一颗平衡二叉树，但是非叶node也是$\Theta(n)$.* 
