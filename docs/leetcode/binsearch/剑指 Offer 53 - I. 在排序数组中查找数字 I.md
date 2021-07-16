# 剑指 Offer 53 - I. 在排序数组中查找数字 I

https://leetcode-cn.com/problems/zai-pai-xu-shu-zu-zhong-cha-zhao-shu-zi-lcof/

## 题目描述

统计一个数字在排序数组中出现的次数。

 

**示例 1:**

```
输入: nums = [5,7,7,8,8,10], target = 8
输出: 2
```



**示例 2:**

```
输入: nums = [5,7,7,8,8,10], target = 6
输出: 0
```



**限制：**

`0 <= 数组长度 <= 50000`



## 思路

因为数组有序，因此可以首先使用二分查找找到与 `target` 相等的元素在数组中第一次出现的下标，再往后搜索统计出现次数。



## 代码

```java
class Solution {
    public int search(int[] nums, int target) {
        int left = 0, right = nums.length - 1;
        // 二分查找找到target在数组中的起始下标
        while (left < right) {
            int mid = left + (right -left) / 2;
            // 因为要找起始下标，所以在相等时不跳过当前元素
            if (nums[mid] >= target) {
                right = mid;
            }
            if (nums[mid] < target) {
                left = mid + 1;
            }
        }
        int res = 0;
        for (int i = left; i < nums.length; ++i) {
            if (nums[i] == target) {
                ++res;
            }
        }
        return res;
    }
}
```

