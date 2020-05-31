---
title: Java之排序算法
categories:
  - 算法与数学
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2018-08-06 23:52:53
tags: 算法与数学
---

排序，各种排序~
<!--more-->
# 桶排序
## First Missing Positive

Given an unsorted integer array, find the first missing positive integer.

For example, given [1,2,0] return 3 and [3,4,-1,1] return 2.

Your algorithm should run in O(n) time and uses constant space.

分析：
本质上是桶排序，每当A[i]!=i+1时，A[i]与A[A[i]-1]交换，终止条件是A[i]==A[A[i]-1].

```java
public int firstMissingPositive(int [] nums){
    bucket_sort(nums);

    for(int i=0;i<nums.length;++i){
        if(nums[i]!=(i+1)){
            return i+1;
        }
        return nums.length+1;
    }

    private static void bucket_sort(int [] A){
        final int n=A.length;
        for(int i=0;i<n :i++){
            while(A[i]!=i+1){
                
                if(A[i]<1 || A[i]>n || A[i]==A[A[i]-1])
                    break;

                int temp = A[i];
                A[i]=A[temp-1];
                A[temp-1]=temp;
            }
        }
    }
}
```