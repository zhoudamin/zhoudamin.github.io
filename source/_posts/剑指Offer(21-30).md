---
title: 剑指Offer(21-30)
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-07-03 11:23:05
categories: 算法
tags: 剑指Offer
---

![](http://img1.imgtn.bdimg.com/it/u=1822661550,696742345&fm=11&gp=0.jpg)
<!--more-->
# 题21：包含min函数的栈
题目：定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的min函数。在该栈中，调用min，push及pop的时间复杂度都是O(1).

```
public class Main{
    private MyStack<Integer> minStack = new MyStack<>();
    private MyStack<Integer> dataStack = new MyStack<>();

    public void push(Integer item){
        dataStack.push(item);
        if(minStack.length == 0 || item<=minStack.head.data){
            minStack.push(item);
        }else{
            minStack.push(minStack.head.data);
        }
    }
    public Integer pop(){
        if(dataStack.length == 0 || minStack.length == 0){
            return null;
        }
        minStack.pop();
        return dataStack.pop();
    }
    public Integer min(){
        if(minStack.length == 0){
            return null;
        }
        return minStack.head.data;
    }
    public static void main(String[] args){
        E21MinInStack test = new E21MinInStack();
        test.push(3);
        test.push(2);
        test.push(1);
        System.out.println(test.pop());
        System.out.println(test.min());
    }
}
```
<br/>
# 题28：字符串的排列
题目：输入一个字符串，打印出该字符串中字符的所有排列。例如输入字符串abc，则打印出由字符a、b、c所能排列出来的所有字符串abc、acb、bac、bca、cab和cba。
```
public class Main {
    public static void main(String[] args){
    permutation("abc");
    }

    public static void permutation(String str){
   int count=0;
   if(str==null){
       return;
   }
   char []  chs=str.toCharArray();//将字符串转换为字符数组
   int point=0;
   System.out.print(chs);
   System.out.print(" ");
   count++;
   char temp1=chs[point];
   chs[point]=chs[++point];
   chs[point]=temp1;
   while (!String.valueOf(chs).equals(str)){
       System.out.print(chs);
       System.out.print(" ");
       count++;
       if(point==chs.length-1){
           char temp=chs[point];
           chs[point]=chs[0];
           chs[0]=temp;
           point=0;
       }else {
           char temp=chs[point];
           chs[point]=chs[++point];
           chs[point]=temp;
       }
   }
    System.out.println(count);
    }
}
```

<br/>
# 题29：数组中出现次数超过一半的数组
题目描述
数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个长度为9的数组{1,2,3,2,2,2,5,4,2}。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。如果不存在则输出null。
```
public class Main {
    public static void main(String[] args){
    int [] array={1,2,3,2,2,2,5,4,2};
    System.out.println(morethanhalfnumber(array));
    }

    public static Integer  morethanhalfnumber(int [] array){
        int count=1;
        int temp=array[0];
        for(int i=1;i<array.length;i++){
            if(temp==array[i]){
                count++;
            }else {
                count--;
                if(count==0){
                    temp=array[i];
                    count++;
                }
            }
        }
        int count1=0;
        for(int i=0;i<array.length;i++){
            if(array[i]==temp){
                count1++;
            }
        }
        return count1>=array.length/2 ? temp:null;
    }
}
```
<br/>
# 题30：最小的k个数
题目描述
输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4。
```
import java.util.ArrayList;
import java.util.TreeSet;

public class Main {
    public static void main(String[] args){
    int [] array={4,5,1,6,2,7,3,8};
    int k=4;
    System.out.println(MinKnumber(array,k));

    }

    public static ArrayList<Integer> MinKnumber(int [] array, int k){
    if(array==null)
        return null;
    ArrayList<Integer> list = new ArrayList<Integer>(k);

    if(k>array.length)
        return list;
        TreeSet<Integer> tree=new TreeSet<Integer>();
        for(int i=0;i<array.length;i++){
            tree.add(array[i]);
        }
        int i=0;
        for (Integer elem:tree){
            if(i>k-1)
                break;
            list.add(elem);
            i++;
        }
        return list;
    }
}


```


![](http://qq.yh31.com/tp/zjbq/201706091131061993.gif)