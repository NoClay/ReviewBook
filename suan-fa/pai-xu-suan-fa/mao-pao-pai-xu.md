### 2. 冒泡排序（改进版）

**思想：**加flag岗哨，每一次遍历，都会把前边未排序部分的最大值冒出来，放到最后

**最佳运行时间：**O\(n\)

**最坏运行时间：**O\(n^2\)

**具体实现：**

```java
/**
     * 冒泡排序，从前向后
     *
     * @param data
     * @return
     */
    static int[] bubbleSort(int[] data) {
        if (data == null || data.length <= 0) {
            return null;
        }
        int[] result = new int[data.length];
        result = Arrays.copyOf(data, data.length);
        for (int i = result.length; i > 0; i--) {
            boolean flag = false;
            for (int j = 0; j < i - 1; j++) {
                if (result[j] > result[j + 1]) {
                    flag = true;
                    swap(result, j, j + 1);
                }
            }
            if (!flag) {
                break;
            }
        }
        return result;
    }
```



