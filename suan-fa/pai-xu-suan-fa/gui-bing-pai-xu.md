### 4. 归并排序

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



