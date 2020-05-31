---
title: Java常用算法手册
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-07-10 16:55:35
categories: 算法
tags: 书籍
---
本书于黄埔图书馆（香雪馆）偶读之，算法不错！

![](http://image54.360doc.com/DownloadImg/2012/08/2512/26415405_22.gif)
<!--more-->
# 简单约瑟夫环算法

1.算法背景:
罗马人攻占了乔塔帕特，41人藏在一个山洞中躲过了这场浩劫。这41个人中，包括历史学家josephus和他的一个朋友。剩余的39个人为了表示不向罗马人屈服
，决定集体自杀。大家决定了一个自杀方案，所有这41人围城一个圆圈，由第一个人开始顺时针报数，没报数为3的人就立刻自杀，然后由下一个人重新开始报数
任然是每报数为3的人就立刻自杀,......,知道所有人都自杀死亡为止.
约瑟夫和他的朋友并不像自杀，于是约瑟夫想到了一个计策，他们两个同样参数到自杀方案中，但是最后却躲过了自杀。请问是怎么做到的

```
public class Main{
    static final int nums=41;//总人数
    static final int killMan = 3; //数到3则杀

    public static void main(String[] args){
        Joseph(2);
    }

    public static void Joseph(int alive){
        int []man =new int [nums]; // 存活者为0
        int pos=-1;  //数组角标
        int  i=0;  //循环，与3比较
        int count=1;  //杀到第几个计数
        while(count<=nums){
            do {
                pos=(pos+1)%nums;   //环状处理
                if(man[pos]==0)
                    i++;
                if(i==killMan){
                    i=0;
                    break;
                }
            }while (true);
                man[pos]=count;      //记录pos+1位置的是第几个挂
            count++;
        }
        alive=nums-alive;
        for(int j=0;j<nums;j++){
            if(man[j]>alive){
                System.out.println("不被杀的位置是："+(j+1)); //数组从0开始，所以位置是j+1
            }
        }

    }
}
```
``
0 = 14
1 = 36
2 = 1
3 = 38
4 = 15
5 = 2
6 = 24
7 = 30
8 = 3
9 = 16
10 = 34
11 = 4
12 = 25
13 = 17
14 = 5
15 = 40
16 = 31
17 = 6
18 = 18
19 = 26
20 = 7
21 = 37
22 = 19
23 = 8
24 = 35
25 = 27
26 = 9
27 = 20
28 = 32
29 = 10
30 = 41
31 = 21
32 = 11
33 = 28
34 = 39
35 = 12
36 = 22
37 = 33
38 = 13
39 = 29
40 = 23
``
<br/>
# 背包问题--动态规划
背包问题具体例子：假设现有容量10kg的背包，另外有3个物品，分别为a1，a2，a3。
物品a1重量为3kg，价值为4；
物品a2重量为4kg，价值为5；
物品a3重量为5kg，价值为6。
将哪些物品放入背包可使得背包中的总价值最大？



![](http://img.zcool.cn/community/01feca56d3c25f32f875520f2310a4.gif)