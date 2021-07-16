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

或者分别用二分查找找到第一次和最后一次出现的位置，然后根据两个值即可计算出结果。



## 代码

找左边界然后往后搜索：

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

分别找两个边界：

```java
class Solution {
    public int search(int[] nums, int target) {
        if (nums.length > 0) {
            int f = findFirst(nums, target);
            if (f == -1) {
                return 0;
            }
            int l = findLast(nums, target);
            return l - f + 1;
        }
        return 0;
    }

    private int findFirst(int[] nums, int target) {
        int l = 0, r = nums.length - 1;
        while (l < r) {
            int mid = l + (r - l >> 1);
            if (nums[mid] >= target) {
                r = mid;
            } else {
                l = mid + 1;
            }
        }
        return nums[l] == target ? l : -1;
    }

    private int findLast(int[] nums, int target) {
        int l = 0, r = nums.length - 1;
        while (l < r) {
            int mid = l + (r - l + 1 >> 1);
            if (nums[mid] <= target) {
                l = mid;
            } else {
                r = mid - 1;
            }
        }
        return nums[l] == target ? l : -1;
    }
}
```

