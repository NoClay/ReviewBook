### 3. 选择排序

**思想：**每一次从后边未排序的部分中选择最小的放到前边已经排序的部分

**最好情况时间：**O\(n^2\)。

**最坏情况时间：**O\(n^2\)。

**具体实现：**

```java
   /**
     * 选择排序
     *
     * @param data
     * @return
     */
    static int[] selectSort(int[] data) {
        if (data == null || data.length <= 0) {
            return null;
        }
        int[] result = new int[data.length];
        result = Arrays.copyOf(data, data.length);
        for (int i = 0; i < result.length; i++) {
            int min = i;
            for (int j = i + 1; j < result.length; j++) {
                if (result[min] > result[j]) {
                    min = j;
                }
            }
            swap(result, i, min);
        }
        return result;
    }
```



