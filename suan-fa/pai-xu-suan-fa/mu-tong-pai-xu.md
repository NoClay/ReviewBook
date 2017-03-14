### 9. 桶排

**思想：**基数排序是先映射到大小为10的计数数组中,然后再映射到大小等于待排序数组长度的临时数组中.而桶排序就是直接整个足够大的临时数组,把待排序的元素全部映射过来.其索引为待排序元素数值.所以为了确定临时数组的大小得先算出数组中最大数.

**特性：**out-place sort、stable sort。

**最坏情况运行时间：**当分布不均匀时，全部元素都分到一个桶中，则O\(n^2\)，当然\[算法导论8.4-2\]也可以将插入排序换成堆排序、快速排序等，这样最坏情况就是O\(nlgn\)。

**最好情况运行时间：**O\(n\)

![](http://img.my.csdn.net/uploads/201301/03/1357220969_9294.png)

**具体实现：**

```java
  /**
     * 木桶排序
     * @param data
     * @return
     */
    static int[] bucketSort(int data[]){
        if (data == null || data.length < 1){
            return null;
        }
        int [] results = new int[data.length];
        results = Arrays.copyOf(data, data.length);
        int max = results[0];
        int min = results[0];
        for (int i = 0; i < results.length; i++) {
            if (max < results[i]){
                max = results[i];
            }
            if (min > results[i]){
                min = results[i];
            }
        }
        int[] buckets = new int[max - min + 1];
        for (int i = 0; i < results.length; i++) {
            buckets[results[i] - min] ++;
        }
        int pos = 0;
        for (int i = 0; i < buckets.length; i++) {
            count ++;
            for (int j = buckets[i]; j > 0; j--) {
                results[pos] = i + min;
                pos ++;
            }
        }
        return results;
    }
```

**证明算法正确性：**

对于任意A\[i\]&lt;=A\[j\]，且A\[i\]落在B\[a\]，A\[j\]落在B\[b\]，我们可以看出a&lt;=b，因此得证。

