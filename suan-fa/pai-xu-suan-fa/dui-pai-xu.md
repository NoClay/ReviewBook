### 5. 堆排序

**思想：**利用最大堆和最小堆的数据结构，最大堆（堆顶是整个堆的最大值，其子树符合条件），最小堆（堆顶是整个堆的最小值）利用大顶堆\(小顶堆\)堆顶记录的是最大关键字\(最小关键字\)这一特性，使得每次从无序中选择最大记录\(最小记录\)变得简单。

**其基本思想为\(大顶堆\)：**

1\)将初始待排序关键字序列\(R1,R2....Rn\)构建成大顶堆，此堆为初始的无序区；

2\)将堆顶元素R\[1\]与最后一个元素R\[n\]交换，此时得到新的无序区\(R1,R2,......Rn-1\)和新的有序区\(Rn\),且满足R\[1,2...n-1\]&lt;=R\[n\];

3\)由于交换后新的堆顶R\[1\]可能违反堆的性质，因此需要对当前无序区\(R1,R2,......Rn-1\)调整为新堆，然后再次将R\[1\]与无序区最后一个元素交换，得到新的无序区\(R1,R2....Rn-2\)和新的有序区\(Rn-1,Rn\)。不断重复此过程直到有序区的元素个数为n-1，则整个排序过程完成。

**如何创建最大堆：**先将初始按照从根节点到子节点的顺序创建完全二叉树，然后慢慢调整子树，使其子树称为最大堆，然后逐渐向上

**特性：**unstable sort、In-place sort。

**最优时间：**O\(nlgn\)

**最差时间：**O\(nlgn\)

**具体实现：**

![](http://img.my.csdn.net/uploads/201301/03/1357220958_2074.png)

**证明算法正确性：**

（1）证明build\_max\_heap的正确性：

循环不变式：每次循环开始前，A\[i+1\]、A\[i+2\]、...、A\[n\]分别为最大堆的根。

初始：i=floor\(n/2\)，则A\[i+1\]、...、A\[n\]都是叶子，因此成立。

保持：每次迭代开始前，已知A\[i+1\]、A\[i+2\]、...、A\[n\]分别为最大堆的根，在循环体中，因为A\[i\]的孩子的子树都是最大堆，因此执行完MAX\_HEAPIFY\(A,i\)后，A\[i\]也是最大堆的根，因此保持循环不变式。

终止：i=0，已知A\[1\]、...、A\[n\]都是最大堆的根，得到了A\[1\]是最大堆的根，因此证毕。

（2）证明heapsort的正确性：

循环不变式：每次迭代前，A\[i+1\]、...、A\[n\]包含了A中最大的n-i个元素，且A\[i+1\]&lt;=A\[i+2\]&lt;=...&lt;=A\[n\]，且A\[1\]是堆中最大的。

初始：i=n，A\[n+1\]...A\[n\]为空，成立。

保持：每次迭代开始前，A\[i+1\]、...、A\[n\]包含了A中最大的n-i个元素，且A\[i+1\]&lt;=A\[i+2\]&lt;=...&lt;=A\[n\]，循环体内将A\[1\]与A\[i\]交换，因为A\[1\]是堆中最大的，因此A\[i\]、...、A\[n\]包含了A中最大的n-i+1个元素且A\[i\]&lt;=A\[i+1\]&lt;=A\[i+2\]&lt;=...&lt;=A\[n\]，因此保持循环不变式。

终止：i=1，已知A\[2\]、...、A\[n\]包含了A中最大的n-1个元素，且A\[2\]&lt;=A\[3\]&lt;=...&lt;=A\[n\]，因此A\[1\]&lt;=A\[2\]&lt;=A\[3\]&lt;=...&lt;=A\[n\]，证毕。
