---
title: 剑指Offer(31-40)
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-07-05 14:33:53
categories: 算法
tags: 剑指Offer
---
剑指Offer
![](http://img.zcool.cn/community/0199a2554443f50000019ae97b5161.jpg)
<!--more-->
# 题31：连续子数组的最大和
题目：输入一个整型数组，数组里有正数也有负数。数组中一个或连续的多个整数组成一个子数组。求所有子数组的和的最大值。要求时间复杂度为 O(n)。
例子说明：
例如输入的数组为{1, -2, 3, 10, -4, 7, 2, -5}，和最大的子数组为｛3, 10, -4, 7, 2}。因此输出为该子数组的和 18 。
```
public class Main{
    public static void main(String[] args){
       int [] arr={1, -2, 3, 10, -4, 7, 2, -5};
        System.out.println(bigsum(arr));
    }
    private static boolean flag=true;
    public static int    bigsum(int [] arr){

     if(arr==null || arr.length<=0){
     flag=false;
     return 0;
     }
     int bigsum=0;
     int sum=0;
     for(int i=0;i<arr.length;i++){
         if(sum<=0){
             sum=arr[i];
         }else {
             sum+=arr[i];
         }
         if(bigsum<sum){
             bigsum=sum;
         }
     }
     return bigsum;
    }
}
```
<br/>

# 题32：求从 1 到 n 的整数中 1 出现的次数
题目：输入一个整数 n 求从 1 到 n 这 n 个整数的十进制表示中 1 出现的次数。
举例说明：

例如输入 12 ，从 1 到 12 这些整数中包含 1 的数字有 1、10、11 和 12，1 一共出现了 5 次。
```

```
<br/>

# 题33：
```

```
<br/>
# 题34：
```

```
<br/>
# 题35：
```

```
<br/>
# 题36：
```

```
<br/>
# 题37：
```

```
<br/>
# 题38：
```

```
<br/>
# 题39：
```

```
<br/>
# 题40：
```

```


![]()