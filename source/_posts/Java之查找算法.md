---
title: Java之查找算法
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-06-22 10:57:01
categories: 算法与数学
tags: Java
---

用Java实现一些基本查找算法。

![](http://cdn01.wallconvert.com/_media/wp_400x250/1/3/20606.jpg)
<!--more-->


## Search for a Range

描述
Given a sorted array of integers, find the starting and ending position of a given target value.
Your algorithm's runtime complexity must be in the order of O(log n) .
If the target is not found in the array, return [-1, -1] .
For example, Given [5, 7, 7, 8, 8, 10] and target value 8, return [3, 4] .
分析
已经排好了序，用二分查找。
重新实现 lower_bound 和 upper_bound
Search for a Range
```
// Search for a Range
// 重新实现 lower_bound 和 upper_bound
// 时间复杂度O(logn)，空间复杂度O(1)
public class Solution{
	private static int[] searchRange(int [] nums,int target){
        int lower=lower_bound(nums,0,nums.length,target);
        int high=upper_bound(nums,0,nums.length,target);

        if(lower == nums.length || nums[lower]!=target)
            return new int[]{-1,-1};
        else
            return new int[]{lower,high-1};
    }

  private static   int lower_bound(int [] A , int first ,int last ,int target){
        while(first!=last){
            int mid=(first+last)/2;
            if(target>A[mid])   first=++mid;
            else	last=mid;
        }
        return first;
    }

  private static   int upper_bound(int [] A , int first ,int last ,int target){
        while(first!=last){
            int mid=(first+last)/2;
            if(target>=A[mid])   first=++mid; //找到最远边界
            else	last=mid;
        }
        return first;
    }
}
```


## Search Insert Position

描述
Given a sorted array and a target value, return the index if the target is found. If not, return the index
where it would be if it were inserted in order.
You may assume no duplicates in the array.
Here are few examples.
[1,3,5,6], 5 → 2
[1,3,5,6], 2 → 1
[1,3,5,6], 7 → 4
[1,3,5,6], 0 → 0
分析
即 std::lower_bound() 。
```
  public static int searchInsert(int [] nums , int target){
            int first=0;
            int last=nums.length;
            while(first!=last){
                int mid=(first+last)/2;
                if(target>nums[mid])
                    first=mid+1;
                else last=mid;
            }
            return first;
       }
```

![](http://cdn01.wallconvert.com/_media/wp_400x250/1/2/19124.jpg)