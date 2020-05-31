---
title: 剑指Offer(11-20)
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-03-26 23:20:11
categories: 算法
tags: 剑指Offer
---
关于剑指Offer的一些解题思路

![](http://img4.duitang.com/uploads/blog/201410/16/20141016153644_i3Ajf.gif)
<!--more-->
# 题11：数值的整数次方
实现函数double Power(double base, int exponent)，求base的exponent次方。 
不得使用库函数，同时不需要考虑大数问题。 
```
import java.util.Scanner;

public class Main {
    public static void main(String[] args)throws Exception {
        Scanner sc=new Scanner(System.in);
        double n=sc.nextDouble();
        int exp=sc.nextInt();
    System.out.println(power(n,exp));
    }

    public static double power(double base ,int exponent)throws Exception{

        if(equal(base,0.0) && exponent<0){
            throw new Exception("0的负次数幂无意义");   //  1/0  没意义
        }
        if(exponent<0){
            return  1.0/powerWithExponent(base,-exponent);
        }else {
            return powerWithExponent(base,exponent);
        }
    }

    public static double powerWithExponent(double base , int exponent){
        if(base==0)
            return 1;
        if(exponent==1)
            return base;
        double res =1;
        for(int i=0;i<exponent;i++){
            res=res*base;
        }
        return res;
    }

    public static boolean equal(double base , double none){
        if((base-none>-0.0000001)&&base-none<0.0000001){      //在0附近，误差0.0000001
            return true;
        }else {
            return false;
        }
    }
}
```

<br/> 
# 题12：打印1到最大的n位数
输入数字n，按顺序打印出从1最大的n位十进制数。比如输入3，则打印出1、2、3 一直到最大的3位数即999。 
```



```


<br/> 
# 题13：


<br/> 
# 题14：调整数组顺序使奇数位于偶数前面
输入一个整数数组，实现一个函数来调整该数组中数字的顺序，使得所有奇数位于数组的前半部分，所有偶数位予数组的后半部分。
```
public class Main {
    public static void main(String[] args)throws Exception {
        int[] array = {1, 4, 2, 5, 21, 67, 21, 66, 23, 77, 68};

        for (int i = 0; i < array.length; i++) {
            System.out.println(order(array)[i]);
        }
    }

    public static int [] order(int [] array){
    if(array.length==0||array==null)
        return array;
    int start=0;
    int end=array.length-1;
    while (start<end){
        while (start<end && !isEven(array[start])){
            start++;
        }
        while (start<end&& isEven(array[end])){
            end--;
        }
        if(start<end){
            int temp=array[start];
            array[start]=array[end];
            array[end]=temp;
        }
    }
  return array;
    }

    public static boolean isEven(int n){
            return  n%2==0;
    }
}
```
<br/> 
# 题15：链表中倒数第k个结点
题目：输入一个链表，输出该链表中倒数第k 个结点．为了符合大多数人的习惯，本题从 1 开始计数，即链表的尾结点是倒数第 1 个结点．例如一个链表有 6 个结点，从头结点开始它们的值依次是 1 、2、3、4、5 、6。这个个链表的倒数第 3 个结点是值为 4 的结点。
```
public class Main {
    public static void main(String[] args){
       ListNode head=new ListNode();
       ListNode second=new ListNode();
       ListNode third= new ListNode();
       ListNode forth=new ListNode();
       head.next=second;
       second.next=third;
       third.next=forth;
       head.val=1;
       second.val=2;
       third.val=3;
       forth.val=4;
       System.out.println(findKToTail(head,3).val);
    }

    public static ListNode findKToTail(ListNode head , int k){
        if (head==null&&k==0){
            return null;
        }
        int count=0;
        ListNode resNode = new ListNode();
        ListNode headNode=head;
        while (headNode!=null){
            headNode=headNode.next;
            count++;
        }
    for(int i=0;i<=count-k;i++){
            head=head.next;
         }
    return head.next;
    }

    public static class ListNode{
        int val;
        ListNode next;
    }
}
```



<br/> 
# 题16：反转链表



<br/> 
# 题17：合并两个排序的链表
题目：输入两个递增排序的链表，合并这两个链表并使新链表中的结点仍然是按照递增排序的
`递归的方法真是好用`
```
public class Main {
    public static void main(String[] args){
       ListNode head1=new ListNode();
        ListNode second1=new ListNode();
        ListNode third1= new ListNode();
        ListNode forth1=new ListNode();
       head1.next=second1;
       second1.next=third1;
       third1.next=forth1;
       head1.val=1;
       second1.val=5;
       third1.val=8;
       forth1.val=13;
        ListNode head2=new ListNode();
        ListNode second2=new ListNode();
        ListNode third2= new ListNode();
        ListNode forth2=new ListNode();
        head2.next=second2;
        second2.next=third2;
        third2.next=forth2;
        head2.val=2;
        second2.val=4;
        third2.val=7;
        forth2.val=9;
        ListNode resNode=mergeList(head1,head2);
        while (resNode!=null) {
            System.out.println(resNode.val);
            resNode = resNode.next;
        }
    }

    public static ListNode mergeList(ListNode head1 , ListNode head2){
       if(head1==null){
           return head2;
       }
       if(head2==null){
           return head1;
       }
       ListNode mergeNode=null;
           if(head1.val<head2.val){
               mergeNode=head1;
               mergeNode.next=mergeList(head1.next,head2);
           }else {
               mergeNode=head2;
               mergeNode.next=mergeList(head1,head2.next);
           }
       return mergeNode;
    }

    public static class ListNode{
        int val;
        ListNode next;
    }
}
```

<br/> 
# 题18：树的子结构


<br/> 
# 题19：二叉树的镜像

题目描述

操作给定的二叉树，将其变换为源二叉树的镜像。 
输入描述:
 二叉树的镜像定义：源二叉树 
            8
           /  \
          6   10
         / \  / \
        5  7 9 11
        镜像二叉树
            8
           /  \
          10   6
         / \  / \
        11 9 7  5
```
public class TreeNode { 
    int val = 0; 
    TreeNode left = null; 
    TreeNode right = null; 
 
    public TreeNode(int val) { 
        this.val = val; 
    } 
} 
*/  
public class Solution {
    public void Mirror(TreeNode root) {
        if(root==null ) return ;
        TreeNode temp;
        temp=root.left;
        root.left=root.right;
        root.right=temp;
         Mirror(root.left);
        Mirror(root.right);
        
    }
}
```

<br/> 
# 题20：顺时针打印矩阵
输入一个矩阵，按照从外向里以顺时针的顺序依次打印出每一个数字，例如，如果输入如下矩阵：

1 2 3 4
5 6 7 8
9 10 11 12
13 14 15 16

则依次打印出数字1,2,3,4,8,12,16,15,14,13,9,5,6,7,11,10.
```
import java.util.ArrayList;  
public class Solution {  
    public ArrayList<Integer> printMatrix(int [][] matrix) {  
        if (matrix == null)  
            return null;  
        ArrayList<Integer> result = new ArrayList<Integer>();  
        int start = 0;  
        while(matrix[0].length > start *2 && matrix.length > start *2){  
            printMatrixInCircle(matrix,result,start);  
            start++;  
        }  
        return  result;  
    }  
   
    public static void printMatrixInCircle(int [][]matix,ArrayList<Integer>result,int start){  
        int endX = matix[0].length - start -1;  
        int endY = matix.length - start -1;  
       //从左向右打印一行  
        for(int i = start;i <=endX;i++){  
            result.add(matix[start][i]);  
        }  
        //从上到下  
        for(int i  = start+1; i <=endY;i++)  
            result.add(matix[i][endX]);  
        //从右到左  
        if(start < endX &&start < endY)  
            for(int i = endX -1;i>= start;i--)  
                result.add(matix[endY][i]);  
        //从下到上  
        if(start < endX && start < endY-1)  
            for(int i = endY - 1;i >=start+1;i--)  
                result.add(matix[i][start]);  
    }  
}  
```
![](http://img.hb.aicdn.com/402338e39ee91c9f2f55116942593e37d009fd15418553-ulMGQY_fw658)