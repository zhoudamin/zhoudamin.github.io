---
title: LeetCode解题思路(一)
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-06-26 22:48:16
categories: 算法
tags: LeetCode
---

做的一些题的解题思路
![面对疾风吧](http://upload-images.jianshu.io/upload_images/3876828-a4346506018aa44f.gif?imageMogr2/auto-orient/strip  "面对疾风吧")
<!--more-->
# Product of Array Except Self 
除本身之外的数组之积
Given an array of n integers where n > 1, nums, return an array output such that output[i] is equal to the product of all the elements of nums except nums[i].
Solve it without division and in O(n).
For example, given [1,2,3,4], return [24,12,8,6].

解题思路：拆分法
[A_234 , A_134 , A_124 , A_123 ]=
[1     , A_1   , A_12  , A_123 ]*
[A_234 , A_34  , A_4   , 1     ]
```
/**
 * Created by Administrator on 2017/5/8.
 */
public class LeetCode {

    public static void main(String[] args) {
        // int [] nums={5, 7, 1, 8,3, 10};  //测试
        int[] nums = {1, 3, 5, 6};
        int k = 5;
        int [] res = productExceptSelf(nums);
        for (int i=0;i<res.length;i++) {
            System.out.print(res[i]+" ");
        }
    }

    public static int[] productExceptSelf(int[] nums) {
        final int [] result = new int [nums.length];
        final int [] right = new int [nums.length];
        final int [] left = new int [nums.length];
        left[0]=1;
        for(int i=1;i<nums.length;i++){
            left[i]=left[i-1]*nums[i-1];
        }
        right[nums.length-1]=1;
        for(int i=nums.length-2;i>=0;i--){
            right[i]=right[i+1]*nums[i+1];
        }

        for (int i=0;i<nums.length;i++){
            result[i]=right[i]*left[i];
        }
        return  result;
    }
}
```
<br/>
# Increasing Triplet Subsequence

Given an unsorted array return whether an increasing subsequence of length 3 exists or not in the array.

Formally the function should:
Return true if there exists i, j, k 
such that arr[i] < arr[j] < arr[k] given 0 ≤ i < j < k ≤ n-1 else return false.
Your algorithm should run in O(n) time complexity and O(1) space complexity.

Examples:
Given [1, 2, 3, 4, 5],
return true.

Given [5, 4, 3, 2, 1],
return false.
``
用整数最大值去比较，x1记录第一个数，x2记录第二大的数，当出现第三大的数，则return true。
``

```
public class Solution {
    public boolean increasingTriplet(int[] nums) {
        int x1=Integer.MAX_VALUE;
        int x2=Integer.MAX_VALUE;
        
        for(int x: nums){
            if(x<=x1) x1=x;
            else if(x<=x2) x2=x;
            else return true;
        }
        return false;
    }
}
```

<br/>
# Contains Duplicate II
Given an array of integers and an integer k, find out whether there are two distinct indices i and j in the array such that nums[i] = nums[j] and the absolute difference between i and j is at most k.

``
维护一个HashMap,key为整数，value为下标，将数组中的元素不断添加到这个Hashmap中，遇到重复时，计算下标距离；
``
``
用Integer.MAX_VALUE 设置为比较的初始值；
``
``
学会用HashMap是非常关键的。
``

```
public class LeetCode {
    public static void main(String[] args) {
        int[] nums = {1, 3, 5, 1,6};
        int k=3;
        System.out.print(containsNearbyDuplicate(nums,k));
    }

    public static boolean containsNearbyDuplicate(int[] nums, int k) {
        final Map<Integer,Integer> map = new HashMap<>();
        int min=Integer.MAX_VALUE;

        for(int i=0;i<nums.length;i++){
            if(map.containsKey(nums[i])){
                final int preIndex=map.get(nums[i]);
                final int gap = i-preIndex;
                min = Math.min(min,gap);
            }
            map.put(nums[i],i);
        }
        return min<=k;
    }
}
```
<br/>
# Add Two Numbers
You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.

You may assume the two numbers do not contain any leading zero, except the number 0 itself.

Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
``
主要学习怎么创建链表，怎么定义链表
``
```
public class LeetCode {
    public static void main(String[] args) {
        int[] inputl1=new int[]{2,4,3};
        int[] inputl2=new int[]{5,6,4};
        ListNode l1=buildListNode(inputl1);
        ListNode l2=buildListNode(inputl2);
        ListNode listNode =addTwoNumbers(l1,l2);
        while(listNode!=null){
            System.out.println("val "+listNode.val);
            listNode=listNode.next;
        }
    }
    //定义链表
   public static class ListNode{
        int val;
        ListNode next;
        ListNode(int val){
            this.val=val;
            this.next=null;
        }
    }
    //创建链表
    private static ListNode buildListNode(int[] input){
        ListNode first = null,last = null,newNode;
        int num;
        if(input.length>0){
            for(int i=0;i<input.length;i++){
                newNode=new ListNode(input[i]);
                newNode.next=null;
                if(first==null){
                    first=newNode;
                    last=newNode;
                }
                else{
                    last.next=newNode;
                    last=newNode;
                }
            }
        }
        return first;
    }

    /**
     * Definition for singly-linked list.
     * public class ListNode {
     *     int val;
     *     ListNode next;
     *     ListNode(int x) { val = x; }
     * }
     */
        public static ListNode addTwoNumbers(ListNode l1, ListNode l2) {
           ListNode dummy =new ListNode(-1);
           int carry = 0;
           ListNode prev = dummy;
           for(ListNode pa=l1 ,  pb=l2 ; pa!=null || pb!=null ;
            pa=pa==null?null : pa.next,
            pb=pb==null ? null : pb.next,
            prev=prev.next){
               final int ai=pa==null?0:pa.val;
               final int bi=pb==null?0:pb.val;
               final int value=(ai+bi+carry)%10;
               carry=(ai+bi+carry)/10;
               prev.next=new ListNode (value);
            }

            if(carry>0)
                prev.next=new ListNode (carry);
            return dummy.next;
    }
}
```

# Evaluate Reverse Polish Notation

计算逆波兰表达式（又叫后缀表达式）的值

'' 2 '','' 1 '','' + '', ''3'', ''* ''     -->(2+1)*3-->9

用堆栈遇到运算符则把前面两个拿出来运算

```java
public class Main {
    public static void main(String[] args) {
       String []tokens={"2", "1", "+", "3", "*"};
        System.out.print(evalRPN(tokens));
    }

    public static int evalRPN(String [] tokens){
        Stack<String> s = new Stack<>();
        if(tokens.length==1){
            return Integer.parseInt(tokens[0]);
        }
        for(String token:tokens){
            if(!isOperator(token)){
                s.push(token);
            }else {
                int y=Integer.parseInt(s.pop());
                int x=Integer.parseInt(s.pop());
                switch (token.charAt(0)){
                    case '+':x+=y;break;
                    case '-':x-=y;break;
                    case '*':x*=y;break;
                    case '/':x/=y;break;
                }
                s.push(String.valueOf(x));
            }
        }
        return Integer.parseInt(s.peek());
    }

    private static boolean isOperator(final String op){
        return op.length() == 1 && OPS.indexOf(op)!=-1;
    }

    private static String OPS = new String("+-*/");
}
```











































![](http://cdn01.wallconvert.com/_media/wp_400x250/1/4/37803.jpg)