# **Heapsort**

## 6-1
```
BUILD-MAX-HEAP'
 (A, n)
1 A.heap-size = 1
2 for i = 2 to n
3 MAX-HEAP-INSERT(A, A[i], n)
```
### a. Do the procedures BUILD-MAX-HEAP and BUILD-MAX-HEAPalways create the same heap when run on the same input array? Provethat they do, or provide a counterexample.

#### Ans : 不一样，标准建堆是自底向上逐步调用MAX-HEAPIFY的（下沉node），而loop版本的INSERT是先在尾部放入node在上浮

