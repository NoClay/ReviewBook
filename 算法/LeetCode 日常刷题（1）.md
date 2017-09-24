# 1. Two Sum

Given an array of integers, return **indices** of the two numbers such that they add up to a specific target.

You may assume that each input would have **exactly** one solution, and you may not use the *same* element twice.

**Example:**

```
Given nums = [2, 7, 11, 15], target = 9,

Because nums[0] + nums[1] = 2 + 7 = 9,
return [0, 1].
```

算法：简单的穷举法即可

代码：

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int [] result = new int[2];
        for(int i = 0; i < nums.length; i ++){
            for(int j = i + 1; j < nums.length; j ++){
                if(nums[j] == target - nums[i]){
                    return new int[]{i, j};
                }
            }
        }
        return result;
    }
}
```

# 2. Add Two Numbers

You are given two **non-empty** linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

**Input:** (2 -> 4 -> 3) + (5 -> 6 -> 4)
**Output:** 7 -> 0 -> 8

算法：注意考虑加法的进位，然后按照常规方法即可

代码：

```java

    public static ListNode addTwoNumbers(ListNode l1, ListNode l2) {
        if (l1 == null) {
            return l2;
        } else if (l2 == null) {
            return l1;
        }
        ListNode result = null;
        ListNode end = null;
        int cal = 0;
        while (l1 != null && l2 != null) {
            int value = l1.val + l2.val + cal;
            cal = value / 10;
            value = value % 10;
            ListNode newNode = new ListNode(value);
            if (result == null) {
                result = end = newNode;
            } else {
                end.next = newNode;
                end = newNode;
            }
            l1 = l1.next;
            l2 = l2.next;
        }
        while (l1 != null) {
            int value = l1.val + cal;
            cal = value / 10;
            value = value % 10;
            ListNode newNode = new ListNode(value);
            end.next = newNode;
            end = newNode;
            l1 = l1.next;

        }
        while (l2 != null) {
            int value = l2.val + cal;
            cal = value / 10;
            value = value % 10;
            ListNode newNode = new ListNode(value);
            end.next = newNode;
            end = newNode;
            l2 = l2.next;
        }
        while (cal != 0) {
            int value = cal;
            cal = value / 10;
            value = value % 10;
            ListNode newNode = new ListNode(value);
            end.next = newNode;
            end = newNode;
        }
        return result;
    }
```

# 3. Longest Substring Without Repeating Characters

Given a string, find the length of the **longest substring** without repeating characters.

**Examples:**

Given `"abcabcbb"`, the answer is `"abc"`, which the length is 3.

Given `"bbbbb"`, the answer is `"b"`, with the length of 1.

Given `"pwwkew"`, the answer is `"wke"`, with the length of 3. Note that the answer must be a **substring**, `"pwke"` is a *subsequence* and not a substring.

算法：遍历整个字符串，如果遇到重复字符，则计算之前的子串长度，并更新最大值

代码：

```java
    public static int lengthOfLongestSubstring(String s){
        int max = 0;
        int []charMap = new int[256];
        int start = -1;
        for (int i = 0; i < s.length(); i++) {
            if (charMap[s.charAt(i) + 127] - 1 > start)
                start = charMap[s.charAt(i) + 127] - 1;
            charMap[s.charAt(i) + 127] = i + 1;
            max = max > i - start ? max : i - start;
        }
        return max;
    }
```

