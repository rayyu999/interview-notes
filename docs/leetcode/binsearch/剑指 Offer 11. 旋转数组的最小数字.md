# 剑指 Offer 11. 旋转数组的最小数字

https://leetcode-cn.com/problems/xuan-zhuan-shu-zu-de-zui-xiao-shu-zi-lcof/

## 题目描述

把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。输入一个递增排序的数组的一个旋转，输出旋转数组的最小元素。例如，数组 `[3,4,5,1,2]` 为 `[1,2,3,4,5]` 的一个旋转，该数组的最小值为1。  

示例 1：

```
输入：[3,4,5,1,2]
输出：1
```

示例 2：

```
输入：[2,2,2,0,1]
输出：0
```

注意：本题与主站 154 题相同：https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array-ii/



## 思路

最简单的思路是遍历数组，用一个变量 `min` 存储已扫描元素的最小值。每扫描到一个元素就与 `min` 比较，若小于 `min`，则更新 `min` 为当前元素，时间复杂度为 $O(N)$。

可以利用旋转的有序数组的性质，使用二分查找将时间复杂度降为 $O(\log N)$。

* 根据数组的定义，可以知道数组可以分成两个升序数组，其中前一个升序数组的元素全部大于等于后一个升序数组的元素；
* 可以定义两个指针 `left` 和 `right` 分别指向搜索区间的起始元素和终止元素，每次把区间中间的元素 `numbers[mid]` 与终止元素 `numbers[right]` 比较：
  * 若 `numbers[mid] > numbers[right]`，证明 `mid` 与 `right` 指向的元素分别在不同的有序数组中，可以排除左边一半的区间；
  * 若 `numbers[mid] < numbers[right]`，证明 `mid` 与 `right` 指向的元素在同一个有序数组中，可以排除右边一半的区间（此时 `numbers[mid]` 仍有可能为最小元素，因此排除的元素不包括其在内）；
  * 若 `numbers[mid] = numbers[right]`，可以排除终止元素，将右指针向前移动一个元素。
    * 因为此时无法判断 `mid` 和 `right` 分别指向的元素是否处于同一个有序数组中，因此只能排除最右边的一个元素。

最后 `left` 或 `right` 指向的元素即为结果。



## 代码

```java
class Solution {
    public int minArray(int[] numbers) {
        if (numbers.length > 0) {
            int left = 0, right = numbers.length - 1;
            while(left < right) {
                int mid = (left + right) / 2;
                if (numbers[mid] > numbers[right]) {
                    left = mid + 1;
                } else if (numbers[mid] == numbers[right]) {
                    right--;
                } else {
                    right = mid;
                }
            }
            return numbers[left];
        }
        throw new IllegalArgumentException("No min array solution");
    }
}
```

