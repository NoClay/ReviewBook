### 1. 插入排序

**思想：**遍历未排序的数组，将当前便利元素插入到前边已经排好序的合适位置，可以考虑从已经排序的数组的后边向前插入，可以更简便。

**最优复杂度：**当输入数组就是排好序的时候，复杂度为O\(n\)，而快速排序在这种情况下会产生O\(n^2\)的复杂度。

**最差复杂度：**当输入数组为倒序时，复杂度为O\(n^2\)

**具体实现：**

```java
 /**
     * 插入排序
     *
     * @param data
     * @return
     */
    static int[] insertSort(int[] data) {
        if (data == null || data.length <= 0) {
            return null;
        }
        int[] result = new int[data.length];
        result = Arrays.copyOf(data, data.length);
        for (int i = 1; i < result.length; i++) {
            int j = i;
            int key = result[i];
            while (j > 0 && result[j - 1] > key) {
                result[j] = result[j - 1];
                j--;
            }
            result[j] = key;
        }
        return result;
    }
```



