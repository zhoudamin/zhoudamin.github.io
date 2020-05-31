---
title: 剑指Offer(41-50)
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-07-11 15:33:07
categories: 算法
tags: 剑指Offer
---

![](http://image54.360doc.com/DownloadImg/2012/08/2512/26388884_11.gif)
<!--more-->

# 题46：求1+2+3+...+n
【题目描述】求1+2+3+…+n，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）
```
import java.util.Scanner;

public class Main{
    public static void main(String[] args){
        Scanner sc=new Scanner(System.in);
        int  num1=sc.nextInt();
        int num2=sc.nextInt();
        System.out.println( add(num1,num2));
    }

    public static int  add(int num1 ,int num2){
      int sum ,carry;
      do {
          sum=num1^num2;
          carry=(num1&num2)<<1;
          num1=sum;
          num2=carry;
      }while (carry!=0);
      return num1;
    }
}

```
<br/>

![](http://image54.360doc.com/DownloadImg/2012/08/2512/26415405_9.gif)