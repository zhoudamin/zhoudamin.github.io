---
title: 剑指Offer(51-67)
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-07-11 15:33:24
categories: 算法
tags: 剑指Offer
---
英文新增面试题！
![](http://wx2.sinaimg.cn/mw690/648ac377ly1fc9yud55n0g20c806uqrm.gif)
<!--more-->
# 题51：数组中重复的数字

题目：

在一个长度为n的数组里的所有数字都在0到n-1的范围内。 数组中某些数字是重复的，但不知道有几个数字是重复的。也不知道每个数字重复几次。请找出数组中任意一个重复的数字。

例如，如果输入长度为7的数组{2,3,1,0,2,5,3}，那么对应的输出是重复的数字2或者3。

思路：


`1、排序`

将数组排序，然后扫描排序后的数组即可。

时间复杂度：O(nlogn)，空间复杂度:O(1)

`2、哈希表`

从头到尾扫描数组，每扫描到一个数字，判断该数字是否在哈希表中，如果该哈希表还没有这个数字，那么加入哈希表，如果已经存在，则返回该数字；

时间复杂度：O(n)，空间复杂度:O(n)

`3、交换`

0~n-1正常的排序应该是A[i]=i；因此可以通过交换的方式，将它们都各自放回属于自己的位置；

从头到尾扫描数组A，当扫描到下标为i的数字m时，首先比较这个数字m是不是等于i，

如果是，则继续扫描下一个数字；

如果不是，则判断它和A[m]是否相等，如果是，则找到了第一个重复的数字（在下标为i和m的位置都出现了m）；如果不是，则把A[i]和A[m]交换，即把m放回属于它的位置；

重复上述过程，直至找到一个重复的数字；

时间复杂度：O(n)，空间复杂度：O(1)

(将每个数字放到属于自己的位置最多交换两次)
```
public class Main{
    public static void main(String[] args){
        int[] nums={2,3,1,0,2,5,3};
        duplicate(nums);
    }

    public static void duplicate(int [] nums){
        if(nums==null ||nums.length<=0)
            return;
        for(int i=0;i<nums.length;i++){
            if(nums[i]>nums.length)
                return ;
        }

        for(int i=0;i<nums.length;i++){
            while (nums[i]!=i){
                if(nums[i]==nums[nums[i]]){
                    System.out.println(nums[i]);
                    break;
                }
                int temp=nums[nums[i]];
                nums[nums[i]]=nums[i];
                nums[i]=temp;
            }
        }
    }
}

```
<br/>
# 题52：构建乘积数组
题目： 
给定一个数组A[0,1,...,n-1],请构建一个数组B[0,1,...,n-1],其中B中的元素B[i]=A[0]*A[1]*...*A[i-1]*A[i+1]*...*A[n-1]。不能使用除法
```
public class Main{
    public static void main(String[] args){
        int[] nums={1,2,3,4,5};
        ConstructAarry(nums);
    }

    public static void ConstructAarry(int [] nums){
        int [] A =new int [nums.length] ;
        int [] B =new int [nums.length] ;
        int [] C =new int [nums.length] ;
        B[0]=1;
        C[0]=1;
        for(int i=1;i<nums.length;i++){
            B[i]=B[i-1]*nums[i-1];
            C[i]=C[i-1]*nums[nums.length-i];
        }
        for(int i=0;i<nums.length;i++){
            A[i]=B[i]*C[nums.length-i-1];
            System.out.println(A[i]);
        }
    }
}
```

<br/>
# 题53：正则表达式匹配
题目描述
请实现一个函数用来匹配包括'.'和'*'的正则表达式。模式中的字符'.'表示任意一个字符，而'*'表示它前面的字符可以出现任意次（包含0次）。 在本题中，匹配是指字符串的所有字符匹配整个模式。例如，字符串"aaa"与模式"a.a"和"ab*ac*a"匹配，但是与"aa.a"和"ab*a"均不匹配
```

```

<br/>
# 题54：表示数值的字符串
题目：

请实现一个函数用来判断字符串是否表示数值（包括整数和小数）。例如，字符串"+100","5e2","-123","3.1416"和"-1E-16"都表示数值。

但是"12e","1a3.14","1.2.3","+-5"和"12e+4.3"都不是。
```
import java.util.Scanner;

public class Main{
    public static void main(String[] args){
        Scanner sc=new Scanner(System.in);
        String str=sc.next();
        char [] chr=str.toCharArray();
        System.out.println(isNum(chr));
    }

    public static boolean isNum(char [] chr){
        if(chr==null)
            return false;
        int index=0;
        if(chr[index]=='+'||chr[index]=='-'){
            index++;
        }
        if (index==chr.length)
            return false;
        boolean num=true;
        //数有多少个数字
        while (index<chr.length&&chr[index]>='0'&&chr[index]<='9'){
            index++;
        }
        if(index!=chr.length){
            if(chr[index]=='.'){
                index++;
                while (index<chr.length&&chr[index]>='0'&&chr[index]<='9'){
                    index++;
                }
                if(index<chr.length&&(chr[index]=='E'||chr[index]=='e')){
                    num=isExponential(chr,index);
                }else
                    num=false;
            }

        }
        return num&&index==chr.length;
    }

    public static boolean isExponential(char[]chr ,int index){
        index++;
        if(index==chr.length)
            return false;
        if(chr[index]=='+'||chr[index]=='-'){
            index++;
        }
        if(index==chr.length)
            return false;
        while (index<chr.length&&chr[index]>='0'&&chr[index]<='9'){
            index++;
        }
        return index==chr.length? true:false;
    }
}
```

<br/>
# 题55：字符流中第一个不重复的字符
请实现一个函数用来找出字符流中第一个只出现一次的字符。例如，当从字符流中只读出前两个字符"go"时，第一个只出现一次的字符是"g"。当从该字符流中读出前六个字符“google"时，第一个只出现一次的字符是"l"。
输出描述:
如果当前字符流没有存在出现一次的字符，返回#字符。
```
import java.util.Scanner;

public class Main{
    public static void main(String[] args){
        Scanner sc=new Scanner(System.in);
        String str=sc.next();

        System.out.println( FirstAppearingOnce(str));
    }

    public static char FirstAppearingOnce(String str){
        int [] hash=new int [256];
        for(int i=0;i<str.length();i++){
            hash[str.charAt(i)]++;
        }
        int j=0;
        while (hash[str.charAt(j)]!=1){
            j++;
        }
        return str.charAt(j);
    }
}
```



![](http://image54.360doc.com/DownloadImg/2012/08/2512/26388884_7.gif)