# **Sorting in Linear Time**
## 8.1  
### What is the smallest possible depth of a leaf in a decision tree for a comparison sort?
*Ans: n - 1*  
## 8.2 Counting sort

```
COUNTING-SORT(A, n, k)
1  let B[1 : n] and C[0 : k] be new arrays
2  for i = 0 to k
3      C[i] = 0
4  for j = 1 to n
5      C[A[j]] = C[A[j]] + 1
6  // C[i] now contains the number of elements equal to i.
7  for i = 1 to k
8      C[i] = C[i] + C[i - 1]
9  // C[i] now contains the number of elements less than or equal to i.
10 // Copy A to B, starting from the end of A.
11 for j = n downto 1
12     B[C[A[j]]] = A[j]
13     C[A[j]] = C[A[j]] - 1    // to handle duplicate values
14 return B
```
*Tips: 用数组计数并用了前缀和, 最后的倒序历遍保证了稳定性*

### 8.2-7  
Counting sort can also work efficiently if the input values have fractional parts, but the number of digits in the fractional part is small. Suppose that you are given n numbers in the range 0 to k, each with at most d decimal (base 10) digits to the right of the decimal point. Modify
counting sort to run in Θ (n + 10d k) time.  
*Ans: 将数组元素放大*  

### 8.4-5
给定单位圆盘（$x_i^2 + y_i^2 \le 1$）内均匀分布的 $n$ 个点，设计一个平均时间为 $\Theta(n)$ 的算法，按点到原点的距离 $d_i = \sqrt{x_i^2 + y_i^2}$ 进行排序。  

*映射：计算每个点的距离平方 $val_i = x_i^2 + y_i^2$。  
证明：$P(D \le r) = \frac{\pi r^2}{\pi (1)^2} = r^2$。令 $U = D^2$，则 $P(U \le u) = u$，符合 $[0, 1]$ 均匀分布。  
分桶：将点放入第 $\lfloor n \cdot (x_i^2 + y_i^2) \rfloor$ 个桶中。  
排序：对每个桶进行插入排序并合并。*

### 8.4-6
通过 $P(x)$ 映射后的值 $y_i$ 都会被“拉平”成 $[0, 1]$ 上的均匀分布。(数学通用版本)