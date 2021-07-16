# 排序

## 基本概念

### 稳定性

稳定性是指当待排序序列中有两个或两个以上相同的关键字时，排序前和排序后这些关键字的相对位置，如果没有发生变化就是稳定的，否则就是不稳定的。

### 排序算法分类

#### 插入类排序

在一个已经有序的序列中，插入一个新的关键字，就是“插入”类的排序。属于这类排序的有：

* 直接插入排序
* 折半插入排序
* 希尔排序

#### 交换类排序

交换类排序的核心是“交换”，即每一趟排序，都通过一系列的“交换”动作，让一个关键字排到它最终的位置上。属于这类排序的有：

* 冒泡排序
* [快速排序](algorithm/datastructure/排序/?id=快速排序)

#### 选择类排序

选择类排序的核心是“选择”，即每一趟排序都选出一个最小（或最大）的关键字，把它和序列中的第一个（或最后一个）关键字交换，这样最小（或最大）的关键字到位。属于这类排序的有：

* 简单选择排序
* [堆排序](algorithm/datastructure/排序/?id=堆排序)

#### 归并类排序

归并就是将两个或两个以上的有序序列合并成一个新的有序序列，归并类排序就是基于这种思想。属于这类排序的有：

* 二路归并排序

#### 基数类排序

基数类排序是最特别的一类，它基于多关键字排序思想，把一个逻辑关键字拆分成多个关键字。



----

## 交换类排序

### 快速排序

#### 算法说明

快速排序通过多次划分操作实现排序。以升序为例，其执行流程可以概括为：每一趟选择当前所有子序列中的一个关键字（通常是第一个）作为枢轴，将子序列中比枢轴小的移到枢轴前面，比枢轴大的移到枢轴后面；当本趟所有子序列都被枢轴以上述规则划分完毕后会得到新的一组更短的子序列，它们称为下一趟划分的初始序列集。

**快排优化：**

* 根据上述思路可以得知，若数组本来就有序，则每次划分都只能使得待排序的子序列数量-1（枢轴最终总是落在数组一端），这个时候快排就会变成冒泡排序，时间复杂度为 `O(n^2)`；
* 因此每次对子序列进行划分时，可以从子序列中随机选取一个枢轴（而不是第一个元素），由此引入的随机化可以使得时间复杂度降为 `O(nlogn)`



#### 代码

```java
class QuickSort {
    public int[] sortArray(int[] nums) {
        partition(nums, 0, nums.length - 1);
        return nums;
    }

    private void partition(int[] nums, int low, int high) {
        if (low >= high) {
            return;
        }
        // 随机选择基准值
        randomLow(nums, low, high);
        int p = nums[low];
        int i = low, j = high;
        while (i < j) {
            while (i < j && nums[j] >= p) {
                --j;
            }
            if (i < j) {
                nums[i] = nums[j];
                ++i;
            }
            while (i < j && nums[i] <= p) {
                ++i;
            }
            if (i < j) {
                nums[j] = nums[i];
                --j;
            }
        }
        nums[i] = p;
        partition(nums, low, i - 1);
        partition(nums, i + 1, high);
    }

    private void randomLow(int[] nums, int low, int high) {
        Random random = new Random();
        int pos = random.nextInt(high - low + 1);
        swap(nums, low, low + pos);
    } 

    private void swap(int[] nums, int i, int j) {
        int tmp = nums[i];
        nums[i] = nums[j];
        nums[j] = tmp;
    }
}
```





----

## 选择类排序

### 堆排序

#### 算法说明

堆排序是对简单选择排序的改进。

简单选择排序是从n个记录中找出一个最小的记录，需要比较n-1次。但是这样的操作并没有把每一趟的比较结果保存下来，在后一趟的比较中，有许多比较在前一趟已经做过了，但由于前一趟排序时未保存这些比较结果，所以后一趟排序时又重复执行了这些比较操作，因而记录的比较次数较多。

堆是具有下列性质的完全二叉树：每个结点的值都大于或等于其左右孩子结点的值，称为大顶堆；或者每个结点的值都小于或等于其左右孩子结点的值，称为小顶堆。

**算法思想：**

将待排序的序列构造成一个大顶堆。此时，整个序列的最大值就是堆顶的根节点。将它移走(其实就是将其与堆数组的末尾元素交换，此时末尾元素就是最大值)，然后将剩余的n-1个序列重新构造成一个堆，这样就会得到n个元素中的次最大值。如此反复执行，就能得到一个有序序列了。

[图解堆排序](https://www.cnblogs.com/chengxiao/p/6129630.html)

![](http://images.yingwai.top/picgo/20210716205345.gif)



#### 代码

```java
public class HeapSort {
	public static void main(String[] args) {
		int[] arr = { 50, 10, 90, 30, 70, 40, 80, 60, 20 };
		System.out.println("排序之前：");
		for (int i = 0; i < arr.length; i++) {
			System.out.print(arr[i] + " ");
		}
 
		// 堆排序
		heapSort(arr);
 
		System.out.println();
		System.out.println("排序之后：");
		for (int i = 0; i < arr.length; i++) {
			System.out.print(arr[i] + " ");
		}
	}
 
	/**
	 * 堆排序
	 */
	private static void heapSort(int[] arr) { 
		// 将待排序的序列构建成一个大顶堆
		for (int i = arr.length / 2; i >= 0; i--){ 
			heapAdjust(arr, i, arr.length); 
		}
		
		// 逐步将每个最大值的根节点与末尾元素交换，并且再调整二叉树，使其成为大顶堆
		for (int i = arr.length - 1; i > 0; i--) { 
			swap(arr, 0, i); // 将堆顶记录和当前未经排序子序列的最后一个记录交换
			heapAdjust(arr, 0, i); // 交换之后，需要重新检查堆是否符合大顶堆，不符合则要调整
		}
	}
 
	/**
	 * 构建堆的过程
	 * @param arr 需要排序的数组
	 * @param i 需要构建堆的根节点的序号
	 * @param n 数组的长度
	 */
	private static void heapAdjust(int[] arr, int i, int n) {
		int child;
		int father; 
		for (father = arr[i]; leftChild(i) < n; i = child) {
			child = leftChild(i);
			
			// 如果左子树小于右子树，则需要比较右子树和父节点
			if (child != n - 1 && arr[child] < arr[child + 1]) {
				child++; // 序号增1，指向右子树
			}
			
			// 如果父节点小于孩子结点，则需要交换
			if (father < arr[child]) {
				arr[i] = arr[child];
			} else {
				break; // 大顶堆结构未被破坏，不需要调整
			}
		}
		arr[i] = father;
	}
 
	// 获取到左孩子结点
	private static int leftChild(int i) {
		return 2 * i + 1;
	}
	
	// 交换元素位置
	private static void swap(int[] arr, int index1, int index2) {
		int tmp = arr[index1];
		arr[index1] = arr[index2];
		arr[index2] = tmp;
	}
}
```

#### 复杂度分析

**时间复杂度：**构建初始堆经推导复杂度为 `O(n)`，在交换并重建堆的过程中，需交换n-1次，而重建堆的过程中，根据完全二叉树的性质，`[log2(n-1),log2(n-2)...1]`逐步递减，近似为`O(nlogn)`。

**空间复杂度：**原地修改，`O(1)`。

