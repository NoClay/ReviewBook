# 4. Median of Two Sorted Arrays

There are two sorted arrays **nums1** and **nums2** of size m and n respectively.

Find the median of the two sorted arrays. The overall run time complexity should be O(log (m+n)).

**Example 1:**

```
nums1 = [1, 3]
nums2 = [2]

The median is 2.0

```

**Example 2:**

```
nums1 = [1, 2]
nums2 = [3, 4]

The median is (2 + 3)/2 = 2.5
```

##解法

#### Approach #1 Recursive Approach [Accepted]

To solve this problem, we need to understand "What is the use of median". In statistics, the median is used for:

为了解决这个问题，我们需要理解“中位数的定义”，通常，中位数定义为：分割一个集合称为长度相等的两部分，其中一个部分的值都大于另一个集合。

> Dividing a set into two equal length subsets, that one subset is always greater than the other.

If we understand the use of median for dividing, we are very close to the answer.

First let's cut \text{A}A into two parts at a random position ii:

如果我们已经理解了中位数的定义，我们已经很接近了答案，首先我们将A从一个随机的位置划分为两个部分。

```
          left_A             |        right_A
    A[0], A[1], ..., A[i-1]  |  A[i], A[i+1], ..., A[m-1]

```

Since$ \text{A}$ has $m$ elements, so there are $m+1$ kinds of cutting ($i = 0 \sim m$).

既然A有m个元素，所以我们可以有m+1种方式将他们切成两部分

And we know:

    len(left_A)=i,len(right_A)=m−i.
    Note: when i = 0; left_A is empity, and when i = m, right_A is empty.
With the same way, cut $\text{B}$ into two parts at a random position $j$:

```
          left_B             |        right_B
    B[0], B[1], ..., B[j-1]  |  B[j], B[j+1], ..., B[n-1]
```

Put **left_A** and **left_B** into one set, and put **right_A** and **right_B** into another set. Let's name them **left_part** and **right_part**:

我们将left_A和left_B放到同一个集合，将right\_A和right\_B放到另一个集合，然后将他们命名为left\_part, right\_part。

```
          left_part          |        right_part
    A[0], A[1], ..., A[i-1]  |  A[i], A[i+1], ..., A[m-1]
    B[0], B[1], ..., B[j-1]  |  B[j], B[j+1], ..., B[n-1]
```

If we can ensure:

> 1. len(left\_part) = len(right\_part)
> 2. max(left\_part) <= min(right\_part)

then we divide all elements in **{A,B}** into two parts with equal length, and one part is always greater than the other. Then

**$\text{median} = \frac{\text{max}(\text{left}\_\text{part}) + \text{min}(\text{right}\_\text{part})}{2}$**

To ensure these two conditions, we just need to ensure:

> 1. **i + j = m - i + n - j (or: m - i + n - j + 1)**
>    if $n \geq m$, we just need to set: [Math Processing Error]
> 2. $\text{B}[j-1] \leq \text{A}[i]and \text{A}[i-1] \leq \text{B}[j]$

ps.1 For simplicity, I presume **A[i−1],B[j−1],A[i],B[j]** are always valid even if **i = 0, i = m, j = 0, j = m**. I will talk about how to deal with these edge values at last.

**注意：这里我假设 A[i - 1],B[j−1],A[i],B[j] 依然有效，即使i = 0, i = m, j = 0, j = m的情况下，之后会介绍如何解决这些异常值。**

ps.2 Why $n \geq m$? Because I have to make sure j is non-negative since **0≤i≤m** and $j = \frac{m + n + 1}{2} - i $. If **n < m**, then j may be negative, that will lead to wrong result.

**注意：为什么n>= m？因为我们必须保证j是一个非负数，因为$0 \leq i \leq m$ ，并且$j = \frac{m + n + 1}2 - i$， 如果$n < m$, 那么$j$可能就会成为一个负数，就会导致答案错误**

So, all we need to do is:

> Searching $i$ in $[0, m]$, to find an object $i$ such that:
>
> $\qquad \text{B}[j-1] \leq \text{A}[i]$  and $\ \text{A}[i-1] \leq \text{B}[j]$,  where  $j = \frac{m + n + 1}{2} - i$

![5AVV8.png](https://t1.picb.cc/uploads/2017/09/24/5AVV8.png)

所以我们可以进行一个二分查找：

1. 设置iMin = 0, iMax = m，然后再区间$[iMin, iMax]$中开始查找

2. 设置$i = \frac{iMin + iMax} 2$， $j = \frac{m + n + 1}2 - i$

3. 现在我们已经有$len(left\_part) = len(right\_part)$。并且这里可能会产生三个分支解决

   +  $B[j - 1] \leq A[i]  and A[i - 1] \leq B[j]$

     这意味着我们找到了这样的一个$i$， 所以停止搜索

   + $B[j - 1] > A[i]$

     这意味着我们的$A[i]$太小了，**大概我们走远了**, 所以我们可以调整$i$，从而让$B[j - 1] \leq A[i]$

     所以我们增大$i$？

     ​	当然，因为当i增大的时候，j会相应的减小，所以B[j - 1]会减小, A[i]会增		大，最后$B[j - 1] \leq A[i]$

     所以我们必须增大i，也就是说，我们必须调整搜索的区间为$[i + 1, iMax]$。

     所以设置$iMin = i + 1$，然后到第2步

   + $A[i - 1] > B[j]$

     这意味着我们还是走远了，所以我们需要减小$i$，所以我们必须调整搜索区间为$[iMin, i - 1]$，

     所以设置$iMax = i - 1$, 然后到第2步

When the object $i$ is found, the median is:

> $\max(\text{A}[i-1], \text{B}[j-1]) $,  when m+n is odd 当m+n是奇数
>
> $\frac{\max(\text{A}[i-1], \text{B}[j-1]) + \min(\text{A}[i], \text{B}[j])}{2}$,   when m+n is even 当m+n是偶数

![5ACG7.png](https://t1.picb.cc/uploads/2017/09/24/5ACG7.png)

现在是时候考虑之前的异常值了，实际上解决办法比你想象的简单。

我们需要确保的是$max(left\_part) \leq min(right\_part)$。所以如果$i$ 和$j$不是那些边界值(0, m, n)，意味着$A[i - 1], B[j - 1], A[i], B[j]$都存在，然后我们才会检查是否$B[j - 1] \leq A[i]  and A[i - 1] \leq B[j]$，但是如果$A[i - 1], B[j - 1], A[i], B[j]$中的某个值不存在，那么我们就不需要检查这两项的正确性，如果$i = 0$，那么$A[i - 1]$不存在，所以我们不需要确保$A[i - 1] \leq B[j]$。 所以我们需要做的是：

> 在$[0, m]$的区间中搜索$i$，
>
> $({j == 0}\  {or}\ i == m \ or \ B[j - 1] \leq A[i]) \ and (i == 0 \ or j == n \ or A[i - 1] \leq B[j])$ 当$j = \frac{m + n + 1}2 - i$

所以在一个搜索循环中，我们需要确认的是这三个分支

> 1. $({j == 0}\  {or}\ i == m \ or \ B[j - 1] \leq A[i]) $and
>
>    $ (i == 0 \ or j == n \ or A[i - 1] \leq B[j])$ 
>
>    这意味着这个$i$符合要求，所以可以停止搜索
>
> 2. $j > 0 \ and i < m \ and\  B[j - 1] >  A[i]$
>
>    这意味着我们必须增大$i$
>
> 3. $i > 0 \ j < n \ and \ A[i - 1] > B[j]$
>
>    这意味着，我们需要减小$i$

##代码：

```java
class Solution {

    public static double findMedianSortedArrays(int[] nums1, int[] nums2) {
        if (nums1 == null && nums2 == null) {
            return 0;
        }
        if ((nums1 == null || nums1.length == 0)
                && (nums2 != null && nums2.length != 0)) {
            return nums2.length % 2 == 0 ?
                    (nums2[nums2.length / 2] + nums2[nums2.length / 2 - 1]) / 2.0f
                    : (nums2[nums2.length / 2]);
        }
        if ((nums2 == null || nums2.length == 0)
                && (nums1 != null && nums1.length != 0)) {
            return nums1.length % 2 == 0 ?
                    (nums1[nums1.length / 2] + nums1[nums1.length / 2 - 1]) / 2.0f
                    : (nums1[nums1.length / 2]);
        }
        assert nums1 != null;
        assert nums2 != null;
        int[] arrA;
        int[] arrB;
        if (nums1.length > nums2.length){
            arrA = nums2;
            arrB = nums1;
        }else{
            arrA = nums1;
            arrB = nums2;
        }
        int iMin = 0;
        int iMax = arrA.length;
        int m = arrA.length;
        int n = arrB.length;
        assert n >= m;
        int halfLen = (m + n + 1) / 2;
        while (iMin <= iMax){
            int i = (iMin + iMax) / 2;
            int j = halfLen - i;
            if (i < iMax && arrB[j - 1] > arrA[i]){
                iMin ++;
            }else if (i > iMin && arrA[i - 1] > arrB[j]){
                iMax --;
            }else {
                int maxLeft = 0;
                if (i == 0){
                    maxLeft = arrB[j - 1];
                }else if (j == 0){
                    maxLeft = arrA[i - 1];
                }else{
                    maxLeft = max(arrA[i - 1], arrB[j - 1]);
                }
                if ((m + n) % 2 != 0){
                    return maxLeft;
                }
                int minRight = 0;
                if (i == m){
                    minRight = arrB[j];
                }else if (j == n){
                    minRight = arrA[i];
                }else{
                    minRight = min(arrA[i], arrB[j]);
                }
                return (maxLeft + minRight) / 2.0;
            }
        }
        return 0.0;
    }

    public static int max(int a, int b) {
        return a > b ? a : b;
    }

    public static int min(int a, int b) {
        return a > b ? b : a;
    }
}
```

# 5. Longest Palindromic Substring

Given a string **s**, find the longest palindromic（回文） substring in **s**. You may assume that the maximum length of **s** is 1000.

**Example:**

```
Input: "babad"

Output: "bab"

Note: "aba" is also a valid answer.
```

**Example:**

```
Input: "cbbd"

Output: "bb"
```

## 解法：

### 1.暴力

这个方法没什么好讲的，我们一贯的算法是**避免使用暴力来制止暴力**，就是这样 ^_^^_^，这个暴力的解法时间复杂度贼高！

### 2.动态规划

+ 首先分解为小问题，单个字符肯定是回文的，那么我们考虑下对于一个$map[s.length()][s.length()]$，我们采用这个数组中的$map[i][j]$代表$s.subString(i, j + 1)$是一个回文串，则有基础公式

  $map[i][i] = true$

  $map[i][i + 1] = (s_i == s_j)$

+ 之后可以根据以下公式推出

  $map[i, j] = (map[i + 1][j - 1] \ and \ S_i == S_j)$

  最后选出最长的回文串即可。

```java
    public String longestPalindrome(String s) {
        int n = s.length();
        String res = null;
        boolean[][] dp = new boolean[n][n];
        for (int i = n - 1; i >= 0; i--) {
            for (int j = i; j < n; j++) {
                dp[i][j] = s.charAt(i) == s.charAt(j) && (j - i < 3 || dp[i + 1][j - 1]);
                if (dp[i][j] && (res == null || j - i + 1 > res.length())) {
                    res = s.substring(i, j + 1);
                }
            }
        }
        return res;
    }
```



###3. 中心延展法

一个回文串可以从他的中心拓展而出，所以因为整个数组的长度为$n$，所以这里会有$2n - 1$个中心点，比如字符串`abba`，这里的中心点就有7个。

所以我们可以有如下的算法：

```java
public String longestPalindrome(String s) {
    int start = 0, end = 0;
    for (int i = 0; i < s.length(); i++) {
        int len1 = expandAroundCenter(s, i, i);
        int len2 = expandAroundCenter(s, i, i + 1);
        int len = Math.max(len1, len2);
        if (len > end - start) {
            start = i - (len - 1) / 2;
            end = i + len / 2;
        }
    }
    return s.substring(start, end + 1);
}

private int expandAroundCenter(String s, int left, int right) {
    int L = left, R = right;
    while (L >= 0 && R < s.length() && s.charAt(L) == s.charAt(R)) {
        L--;
        R++;
    }
    return R - L - 1;
}
```

# 6.  ZigZag Conversion

The string `"PAYPALISHIRING"` is written in a zigzag pattern on a given number of rows like this: (you may want to display this pattern in a fixed font for better legibility)

```
P     L     I
A   A I   R N
Y P   S I   G
P	  H
```

And then read line by line: 

```
"PAHNAPLSIIGYIR"
```

Write the code that will take a string and make this conversion given a number of rows:

```
string convert(string text, int nRows);
```

**convert("PAYPALISHIRING", 4)**  should return **"PLIAAIRNYPSIGPH"**

```java


    public String convert(String s, int numRows) {
        if (numRows <= 1) return s;
        StringBuilder[] stringMap = new StringBuilder[numRows];
        for (int i = 0; i < stringMap.length; i++) {
            stringMap[i] = new StringBuilder("");
        }
        int increase = 1;
        int index = 0;
        for (int i = 0; i < s.length(); i++) {
            stringMap[index].append(s.charAt(i));
            if (index == 0) {
                increase = 1;
            }
            if (index == numRows - 1) {
                increase = -1;
            }
            index += increase;
        }
        StringBuilder builder = new StringBuilder();
        for (StringBuilder aStringMap : stringMap) {
            builder.append(aStringMap);
        }
        return builder.toString();
    }
```



.