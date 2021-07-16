# 剑指 Offer 53 - II. 0~n-1中缺失的数字

https://leetcode-cn.com/problems/que-shi-de-shu-zi-lcof/

## 题目描述

一个长度为n-1的递增排序数组中的所有数字都是唯一的，并且每个数字都在范围0～n-1之内。在范围0～n-1内的n个数字中有且只有一个数字不在该数组中，请找出这个数字。

 

**示例 1:**

```
输入: [0,1,3]
输出: 2
```

**示例 2:**

```
输入: [0,1,2,3,4,5,6,7,9]
输出: 8
```



**限制：**

`1 <= 数组长度 <= 10000`



## 思路

二分查找，当搜索到的元素与下标相等时，排除左边元素，反之排除右边元素；当 `left > right` 时，`left` 指向的就为第一个无序元素的下标。



## 代码

```java
class Solution {
    public int missingNumber(int[] nums) {
        int left = 0, right = nums.length - 1;
        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (mid == nums[mid]) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        return left;
    }
}
```

