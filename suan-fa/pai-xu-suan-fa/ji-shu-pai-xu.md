### 7. 计数排序

**思想：**类似桶排，对于给定的输入序列中的每一个元素x，确定该序列中值小于x的元素的个数。一旦有了这个信息，就可以将x直接存放到最终的输出序列的正确位置上。例如，如果输入序列中只有17个元素的值小于x的值，则x可以直接存放在输出序列的第18个位置上。当然，如果有多个元素具有相同的值时，我们不能将这些元素放在输出序列的同一个位置上，因此，上述方案还要作适当的修改。

**特性：**stable sort、out-place sort。

**最坏情况运行时间：**O\(n+k\)

**最好情况运行时间：**O\(n+k\)

**具体实现：**

```java
 /**
     * 计数排序
     * @param data
     * @return
     */
    static int[] countSort(int[] data){
        if (data == null || data.length <= 0) {
            return null;
        }
        int[] result = new int[data.length];
        result = Arrays.copyOf(data, data.length);
        int max = result[0];
        int min = result[0];
        for (int i = 0; i < result.length; i++) {
            if (max < result[i]){
                max = result[i];
            }
            if (min > result[i]){
                min = result[i];
            }
        }
        int[] ballot = new int[max + 1 - min];
        for (int i = 0; i < result.length; i++) {
            ballot[result[i] - min] ++;
        }
        for (int i = 1; i < ballot.length; i++) {
            ballot[i] += ballot[i - 1];
        }
        for (int i = data.length - 1; i >= 0 ; i--) {
            int value = data[i];
            int pos = ballot[value - min];
            result[pos - 1] = value;
            ballot[value - min] --;
        }
        return result;
    }
```



