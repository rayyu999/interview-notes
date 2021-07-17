# 二分查找框架

二分查找主要有三种模板，除了常规的找指定值，还有找某个值第一次出现的位置或最后一次出现的位置。



## 常规

```java
public int binsearch(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left < right) {
        int mid = left + (right - left >> 1);
        if (nums[mid] == target) {
            return mid;
        } else if (nums[mid] > target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return nums[left] == target ? left : -1;
}
```



## 找左边界（第一次出现的位置）

搜索区间为 `[left, right]`，中间元素小于目标时，收缩左边界；否则收缩右边界。这样能保证每次搜索的区间内都是大于等于目标的元素。

```java
public int binsearch(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left < right) {
        int mid = left + (right - left >> 1);
        if (nums[mid] >= target) {
            right = mid;
        } else {
            left = mid + 1;
        }
    }
    return nums[left] == target ? left : -1;
}
```



## 找右边界（最后一次出现的位置）

搜索区间为 `[left, right]`，中间元素大于目标时，收缩右边界；否则收缩左边界。这样能保证每次搜索的区间内都是小于等于目标的元素。

```java
public int binsearch(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left < right) {
        int mid = left + (right - left + 1 >> 1);
        if (nums[mid] <= target) {
            left = mid;
        } else {
            right = mid - 1;
        }
    }
    return nums[left] == target ? left : -1;
}
```

