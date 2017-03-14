### 3. 快速排序

**思想：**快速排序是对冒泡排序的一种改进。它的基本思想是：通过一趟排序将要排序的数据分割成独立的两部分，其中一部分的所有数据都比另外一不部分的所有数据都要小，然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。（默认的分割值为当前数组的最后一个值，可以用随机化改进）

**最坏运行时间：**当输入数组已排序时，时间为O\(n^2\)，当然可以通过随机化来改进（shuffle array 或者 randomized select pivot）,使得期望运行时间为O\(nlgn\)。

**最佳运行时间：**O\(nlgn\)

**具体实现：**

```java
   /**
     * 快速排序
     * @param data
     * @return
     */
    static int[] quickSort(int[] data){
        if (data == null || data.length <= 0) {
            return null;
        }
        int[] result = new int[data.length];
        result = Arrays.copyOf(data, data.length);
        quickCore(result, 0, data.length - 1);
        return result;
    }

    static void quickCore(int []data, int start, int end){
        if (start < end){
            int pos = start;
            int key = data[end];
            for (int i = start; i < end; i++) {
                if (data[i] < key){
                    swap(data, pos, i);
                    pos ++;
                }
            }
            swap(data, pos, end);
            quickCore(data, start, pos - 1);
            quickCore(data, pos + 1, end);
        }
    }
```

**证明算法正确性：**

对partition函数证明循环不变式：A\[p...i\]的所有元素小于等于pivot，A\[i+1...j-1\]的所有元素大于pivot。

初始：i=p-1,j=p，因此A\[p...p-1\]=空，A\[p...p-1\]=空，因此成立。

保持：当循环开始前，已知A\[p...i\]的所有元素小于等于pivot，A\[i+1...j-1\]的所有元素大于pivot，在循环体中，

 - 如果A\[j\]&gt;pivot，那么不动，j++，此时A\[p...i\]的所有元素小于等于pivot，A\[i+1...j-1\]的所有元素大于pivot。

 - 如果A\[j\]&lt;=pivot，则i++，A\[i+1\]&gt;pivot，将A\[i+1\]和A\[j\]交换后，A\[P...i\]保持所有元素小于等于pivot，而A\[i+1...j-1\]的所有元素大于pivot。

终止：j=r，因此A\[p...i\]的所有元素小于等于pivot，A\[i+1...r-1\]的所有元素大于pivot。

