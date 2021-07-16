# 剑指 Offer 57. 和为s的两个数字

https://leetcode-cn.com/problems/he-wei-sde-liang-ge-shu-zi-lcof/

## 题目描述

输入一个递增排序的数组和一个数字s，在数组中查找两个数，使得它们的和正好是s。如果有多对数字的和等于s，则输出任意一对即可。

 

**示例 1：**

```
输入：nums = [2,7,11,15], target = 9
输出：[2,7] 或者 [7,2]
```

**示例 2：**

```
输入：nums = [10,26,30,31,47,60], target = 40
输出：[10,30] 或者 [30,10]
```



**限制：**

* `1 <= nums.length <= 10^5`
* `1 <= nums[i] <= 10^6`



## 思路

遍历数组中的元素，计算与目标值的差，然后使用二分查找到数组的剩余部分寻找有无与差值相等的元素，若有则返回对应的下标，反之返回-1。当遍历到某一个元素寻找到符合条件的另一个元素时，跳出循环返回结果。



## 代码

```java
class Solution {
    public int[] twoSum(int[] nums, int target) {
        int[] res = new int[2];
        for (int i = 0; i < nums.length; ++i) {
            int sub = target - nums[i];
            int index = binsearch(nums, i+1, nums.length, sub);
            if (index != -1) {
                res[0] = nums[i];
                res[1] = nums[index];
                break;
            }
        }
        return res;
    }

    private int binsearch(int[] nums, int low, int high, int target) {
        if (low > high) {
            return -1;
        }
        int left = low, right = high - 1;
        while (left <= right) {
            int mid = left + (right - left) / 2;
            if (nums[mid] == target) {
                return mid;
            } else if (nums[mid] > target) {
                right = mid - 1;
            } else if (nums[mid] < target){
                left = mid + 1;
            }
        }
        return -1;
    }
}
```

