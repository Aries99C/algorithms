# 剑指Offer51.数组中的逆序对

## 题目

在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组，求出这个数组中的逆序对的总数。

## 解题思路

简单的题目往往拥有着较高的难度。既然这是一道困难题，那么肯定不是双for循环怎么简单的算法，那么基本思路应该是先做排序，然后用排序得到每个数在整个数组中是第k大。

例如，[1, 2, ..., 10]中10的实际位置是5，则说明10的右边有10-5=5个数与其构成逆序对，即n-k-i个逆序对。但是这一规律并不是对所有数都成立的，因为我们不清楚一个数左边是否还存在比他更大的数，而要做到这一点要花费更多的时间。因此要考虑在不改变数在数组内位置前就能得知其对逆序对总数的贡献。

一个想法是使用修改后的归并排序来计数。首先，假设归并排序函数能统计左右两个区间内的逆序对总数，那么只需要在归并过程中统计左右区间中交叉的逆序对总数即可，而这是可以在归并的同时实现的，当左区间指针对应的数大于右区间指针对应的数时，就说明左区间剩余的所有数都比右区间的这个数大，则该数对逆序对总数的贡献就应该为左区间剩余数的数量。由此就可以在归并排序的同时实现对逆序对数的统计。

## 代码

```java
public class Solution {
    public int reversePairs(int[] nums) {
        int len = nums.length;

        // 数组长度不构成逆序对
        if (len < 2) {
            return 0;
        }

        int[] copy = new int[len];
        for (int i = 0; i < len; i++) {
            copy[i] = nums[i];
        }

        // 用于归并的临时空间
        int[] temp = new int[len];
        return reversePairs(copy, 0, len - 1, temp);
    }

    private int reversePairs(int[] nums, int left, int right, int[] temp) {
        if (left == right) {
            return 0;
        }

        int mid = left + (right - left) / 2;
        int leftPairs = reversePairs(nums, left, mid, temp);
        int rightPairs = reversePairs(nums, mid + 1, right, temp);

        // 左右区间可以直接合并，不贡献逆序对
        if (nums[mid] <= nums[mid + 1]) {
            return leftPairs + rightPairs;
        }

        // 左右区间存在交叉，需要计算逆序对贡献
        int crossPairs = mergeAndCount(nums, left, mid, right, temp);
        return leftPairs + rightPairs + crossPairs;
    }

    private int mergeAndCount(int[] nums, int left, int mid, int right, int[] temp) {
        for (int i = left; i <= right; i++) {
            temp[i] = nums[i];
        }

        int i = left;
        int j = mid + 1;

        int count = 0;
        for (int k = left; k <= right; k++) {
            // 左右某指针已移动到区间末端
            if (i == mid + 1) {
                nums[k] = temp[j];
                j++;
            } else if (j == right + 1) {
                nums[k] = temp[i];
                i++;
            } 
            // 左区间小于右区间，无逆序对
            else if (temp[i] <= temp[j]) {
                nums[k] = temp[i];
                i++;
            } 
            // 左区间大于右区间，左区间后续的数都与右区间该数构成逆序对
            else {
                nums[k] = temp[j];
                j++;
                count += (mid - i + 1);
            }
        }
        return count;
    }
}
```

## 结果

时间复杂度击败$91.41%$，接近$100%$。