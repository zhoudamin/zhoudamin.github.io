---
title: Java之数组算法
categories: 算法与数学
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2018-07-26 23:28:19
tags: 算法与数学
---

数组是常见的算法题，所以多做题，多总结，总是能找到idea的。
<!--more-->

## Product of Array Except Self

Given an array of n integers where n > 1, nums, return an array output such that output[i] is equal to the product of all the elements of nums except nums[i].

Solve it without division and in O(n).

For example, given [1,2,3,4], return [24,12,8,6].

Follow up:
Could you solve it with constant space complexity? (Note: The output array does not count as extra space for the purpose of space complexity analysis.)

思路：
首先，介绍方法： [1,a1,a12,a123].*[a234,a34,a4,1]=[a234,a134,a124,a123]
这个算法有个O(1)的思路:就是用常数的方法，从左边乘到右边，再取一个常数，从右边乘到左边

```java
//关键算法段

left[0]=1;

for(int i=1;i<num.length;++i){
    left[i]=left[i-1]*num[i-1];
}

int right=1;
for(int i=num.length-1;i>=0;i--){
    left[i]*=right;   //right 初始为1，每次乘完再迭代
    right*=num[i];
}
return left;
```


