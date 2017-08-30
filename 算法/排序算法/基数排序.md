### 8. 基数排序

**思想：**当d为常数、k=O\(n\)时，效率为O\(n\)我们也不一定要一位一位排序，我们可以多位多位排序，比如一共10位，我们可以先对低5位排序，再对高5位排序。

**引理：**假设n个b位数，将b位数分为多个单元，且每个单元为r位，那么基数排序的效率为O\[\(b/r\)\(n+2^r\)\]。当b=O\(nlgn\)，r=lgn时，基数排序效率O\(n\)

**特性：**stable sort、Out-place sort。

**最坏情况运行时间：**O\(\(n+k\)d\)

**最好情况运行时间：**O\(\(n+k\)d\)

**具体实现：**

```java
   /**
     * 基数排序
     * @param data
     * @return
     */
    static int[] radixSort(int [] data){
        if (data == null || data.length <= 0) {
            return null;
        }
        int[] result = new int[data.length];
        result = Arrays.copyOf(data, data.length);
        int pos = 0;
        while (radixCore(result, pos)){
            pos ++;
        }
        return result;
    }

    /**
     * 这里采用按照每一位进行排序
     * @param data
     * @param pos
     * @return
     */
    static boolean radixCore(int []data, int pos){
        int[] ballot = new int[10];
        int result[] = new int[data.length];
        for (int i = 0; i < data.length; i++) {
            int key = (int) ((data[i] / Math.pow(10, pos)) % 10);
            ballot[key] ++;
        }
        for (int i = 1; i < ballot.length; i++) {
            ballot[i] += ballot[i - 1];
        }
        //仍然有位参与了排序
        if (ballot[9] == ballot[0]){
            return false;
        }
        for (int i = data.length - 1; i >= 0 ; i--) {
            int value = data[i];
            int key = (int) ((data[i] / Math.pow(10, pos)) % 10);
            int position = ballot[key];
            result[position - 1] = value;
            ballot[key] --;
        }
        for (int i = 0; i < data.length; i++) {
            data[i] = result[i];
        }
        return true;
    }
```



