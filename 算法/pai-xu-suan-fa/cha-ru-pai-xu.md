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

**证明算法正确性：**

循环不变式：在每次循环开始前，A\[1...i-1\]包含了原来的A\[1...i-1\]的元素，并且已排序。

初始：i=2，A\[1...1\]已排序，成立。

保持：在迭代开始前，A\[1...i-1\]已排序，而循环体的目的是将A\[i\]插入A\[1...i-1\]中，使得A\[1...i\]排序，因此在下一轮迭代开       始前，i++，因此现在A\[1...i-1\]排好序了，因此保持循环不变式。

终止：最后i=n+1，并且A\[1...n\]已排序，而A\[1...n\]就是整个数组，因此证毕。

**而在算法导论2.3-6中还问是否能将伪代码第6-8行用二分法实现？**

实际上是不能的。因为第6-8行并不是单纯的线性查找，而是还要移出一个空位让A\[i\]插入，因此就算二分查找用O\(lgn\)查到了插入的位置，但是还是要用O\(n\)的时间移出一个空位。

**问：快速排序（不使用随机化）是否一定比插入排序快？**

答：不一定，当输入数组已经排好序时，插入排序需要O\(n\)时间，而快速排序需要O\(n^2\)时间。

# ~~优化~~

~~**折半插入排序：**基于折半查找（二分法查找）有序数组的方法，我们可以通过对前边已经排序的部分进行折半查找，找到记录应该插入的位置。这种方法改良了查找（比较）关键字的代价，但是并没有改良我们对与移动元素的时间代价，所以时间复杂度没有太大变化。已然和原来一样。~~

# 优化

**希尔排序：**先将整个带牌元素序列分割成若干个子序列，分别进行直接插入排序， 然后依次缩减增量再进行排序，待整个序列中的元素基本有序的时候（增量足够小的时候），再对全体元素进行一次直接插入排序，因为直接插入排序在元素基本有序的时候效率是很高的，因此希尔排序，在时间效率上要比前两种方法有较大提高，但是极不稳定。

具体实现：

```java
   /**
     * 希尔排序
     * @param data
     * @return
     */
    static int[] shellSort(int [] data){
        if (data == null || data.length <= 0) {
            return null;
        }
        int[] result = new int[data.length];
        result = Arrays.copyOf(data, data.length);
        int len = result.length / 2;
        while (len >= 1){
            for (int i = len; i < result.length ; i++) {
                int j = i;
                int key = result[i];
                while (j >= len && result[j - len] > key) {
                    result[j] = result[j - len];
                    j -= len;
                    count++;
                }
                result[j] = key;
            }
            len /= 2;
        }
        return result;
    }
```



