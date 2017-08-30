### 6. 归并排序

**思想：**运用分治的思想解决排序问题，将对一个数组的排序分解为将一个个小数组的排序，最后合并, 在合并的时候采用的算法思想是合并两个有序的数组。

**最坏情况运行时间：**O\(nlgn\)

**最佳运行时间：**O\(nlgn\)

**具体实现：**

```java
/**
     * 归并排序
     * @param data
     * @return
     */
    static int[] mergeSort(int[] data) {
        if (data == null || data.length <= 0) {
            return null;
        }
        int[] result = new int[data.length];
        result = Arrays.copyOf(data, data.length);
        mergeCore(result, 0, result.length - 1);
        return result;
    }

    static void mergeCore(int[] data, int start, int end) {
        if (start < end) {
            int middle = (start + end) / 2;
            mergeCore(data, start, middle);
            mergeCore(data, middle + 1, end);
            merge(data, start, middle, end);
        }
    }

    static void merge(int[] data, int start, int middle, int end) {
        int i = middle;
        int j = end;
        int k = end - start;
        int[] result = new int[k + 1];
        for (int l = 0; l < result.length; l++) {
            result[l] = data[start + l];
        }
        //归并
        while (i >= start && j > middle) {
            if (data[i] < data[j]) {
                result[k] = data[j];
                j--;
            } else {
                result[k] = data[i];
                i--;
            }
            k--;
        }
        while (i >= start) {
            result[k] = data[i];
            i--;
            k--;
        }
        while (j > middle) {
            result[k] = data[j];
            j--;
            k--;
        }
        for (int l = 0; l < result.length; l++) {
            data[start + l] = result[l];
        }
    }
```

**证明算法正确性：**

其实我们只要证明merge\(\)函数的正确性即可。

merge函数的主要步骤在第25~31行，可以看出是由一个循环构成。

循环不变式：每次循环之前，A\[p...k-1\]已排序，且L\[i\]和R\[j\]是L和R中剩下的元素中最小的两个元素。

初始：k=p，A\[p...p-1\]为空，因此已排序，成立。

保持：在第k次迭代之前，A\[p...k-1\]已经排序，而因为L\[i\]和R\[j\]是L和R中剩下的元素中最小的两个元素，因此只需要将L\[i\]和R\[j\]中最小的元素放到A\[k\]即可，在第k+1次迭代之前A\[p...k\]已排序，且L\[i\]和R\[j\]为剩下的最小的两个元素。

终止：k=q+1，且A\[p...q\]已排序，这就是我们想要的，因此证毕。

归并排序的例子：

![](http://img.my.csdn.net/uploads/201301/03/1357220947_3483.png)

**问：归并排序的缺点是什么？**

答：他是Out-place sort，因此相比快排，需要很多额外的空间。

**问：为什么归并排序比快速排序慢？**

答：虽然渐近复杂度一样，但是归并排序的系数比快排大。

**问：对于归并排序有什么改进？**

答：就是在数组长度为k时，用插入排序，因为插入排序适合对小数组排序。在算法导论思考题2-1中介绍了。复杂度为O\(nk+nlg\(n/k\)\) ，当k=O\(lgn\)时，复杂度为O\(nlgn\)

# 优化

**自然归并排序：**第一步合并相邻的长度为1的子序列，这是因为长度为1的子序列已经是排好序的，对序列的一次线性扫描即可找到这些已经排序好的子序列，然后将相邻的排好序的子序列两两合并，构成更大的排好序的子序列。直到整个数组已经排好序。

时间复杂度为O\(n\)

