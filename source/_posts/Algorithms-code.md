---
title: Algorithms code
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-04-03 11:37:50
categories: 算法与数学
tags: 算法
---
一些值得回看的小算法。
![](http://cdn01.wallconvert.com/_media/wp_400x250/1/2/16098.jpg)

<!--more-->



# 最长的连续子数组

子数组数字不重复

```java
int [] arr={1,2,3,4,7}; //输出4
int [] arr1={1,2,3,4,1,2,3,4,5,1}; //输出5
```



```java
package RecursiveAndDynamic;

/**
 * Created by zdmein on 2017/9/2.
 * 最长的连续子数组（子数组数字不重复）
 * int [] arr={1,2,3,4,7}; 输出4
 *  int [] arr1={1,2,3,4,1,2,3,4,5,1}; 输出5
 */
public class longestSubArr1 {
    public static void main(String [] args){
        int [] arr={1,2,3,4,7};
        int [] arr1={1,2,3,4,1,2,3,4,5,1};
        longestSubArr(arr1);

    }

    public static void longestSubArr(int [] arr){
        if(arr==null||arr.length==0){
            return;
        }

        int len=1;
        int maxlen=1;
        for(int i=1;i<arr.length;i++){
            if(arr[i]==arr[i-1]+1){
                len++;
                maxlen=Math.max(len,maxlen);
            }else {
                len=1;
            }
        }
        System.out.println(maxlen);
    }
}
```



---



# 删除链表中的元素

样例
给出链表 1->2->3->3->4->5->3, 和 val = 3, 你需要返回删除3之后的链表：1->2->4->5。

```
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode(int x) { val = x; }
 * }
 */
public class Solution {
    /**
     * @param head a ListNode
     * @param val an integer
     * @return a ListNode
     */
    public ListNode removeElements(ListNode head, int val) {
        // Write your code here
        if(head==null) return head;
        ListNode p=head,q=head.next;
        while(q!=null){
        if(q.val==val){
              p.next=q.next;
              q=q.next;
          }else{
        	  p=p.next;
        	  q=q.next;
          }
        }
        if(head.val==val) head=head.next;
        return head;
    }
}
```

# 求n的阶乘结果后有几个零

```
//结果后的零是由n中的5组成的，多少个5就有多少个0，所以只需要统计有多少5就可以了

public class Solution {
    public int trailingZeroes(int n) {
       int rs=0;
    	while(n!=0){
    		rs+=n/5;
    		n/=5;
    	}
          return rs;
    }
}

```
# 统计小于n的素数个数

我写的用 n/2;
改进为   Math.sqrt(n)；
复杂度还是高了点；
```
public class Solution {
    public int countPrimes(int n) {
        boolean [] notPrimes = new boolean[n];
       int count =0;
       for(int i=2;i<n;i++){
    	   if(notPrimes[i]==false){
    		   count++;
    		   for(int j=2;i*j<n;j++){
    			   notPrimes[i*j]=true;
    		   }
    	   }
       }
        return count;
    }
}
```
这个算法降低了复杂度，值得学习！！

# 同构字符串

Given "egg", "add", return true.
Given "foo", "bar", return false.
Given "paper", "title", return true.
即结构相同
我的
```
public class Solution {
    public boolean isIsomorphic(String s, String t) {
        int [] count=new int [s.length()];   
        for(int i=0;i<s.length();i++){
        	count[i]=s.charAt(i)-t.charAt(i);
        	 for(int j=0;j<i;j++){
      		   if(s.charAt(i)==s.charAt(j)){
      			   if(count[i]!=count[j]){
      				   return false;
      			   }
      		   }
      			   if(t.charAt(i)==t.charAt(j)){
          			   if(count[i]!=count[j]){
          				   return false;
          			   }
      		   }
        }
        }        
        return true;
    }
}
```
降低复杂度,用数组装字符，很重要的思想！
```
public class Solution {
    public boolean isIsomorphic(String s, String t) {
        int [] m=new int [256];
        int [] n=new int [256];      
        for(int i=0;i<s.length();i++){
            if(m[s.charAt(i)]!=n[t.charAt(i)]){
            	return false;
            }
            m[s.charAt(i)]=i+1;     //关键部分，通过i，实现错位
            n[t.charAt(i)]=i+1;     //不同位置加的值不同      
        }
        return true;
    }
}
```
# 组合成最大的数

given [3, 30, 34, 5, 9], the largest formed number is 9534330.
[0,0]，输出0.
```
public class Solution {
    public String largestNumber(int[] nums) {
        String [] rs=new String[nums.length];
        String sb=new String();
        for(int i=0;i<nums.length;i++){
        	rs[i]=String.valueOf(nums[i]);
        }
        for(int j=0;j<nums.length-1;j++){
    	for(int i=0;i<nums.length-j-1;i++){
    		String s1=rs[i]+rs[i+1];
    		String s2=rs[i+1]+rs[i];
    		if(s2.compareTo(s1)>0){
    			String temp=rs[i];
    			rs[i]=rs[i+1];
    			rs[i+1]=temp;
    		}
    	 }
    	}
        for(int i=0;i<nums.length;i++){
        	sb+=rs[i];
        }
         if(sb.charAt(0) == '0')
            return "0";
        return sb;
    }
}
```

# 判断n是不是2的幂

只需要将n转化为二进制，然后计算二进制位里面有多少个1，如果计数为1，那么n是2的幂



---



# 快乐数

写一个算法来判断一个数是不是"快乐数"。

一个数是不是快乐是这么定义的：对于一个正整数，每一次将该数替换为他每个位置上的数字的平方和，然后重复这个过程直到这个数变为1，或是无限循环但始终变不到1。如果可以变为1，那么这个数就是快乐数。

您在真实的面试中是否遇到过这个题？ Yes
样例
19 就是一个快乐数。

1^2 + 9^2 = 82
8^2 + 2^2 = 68
6^2 + 8^2 = 100
1^2 + 0^2 + 0^2 = 1

```
public class Solution {
    /**
     * @param n an integer
     * @return true if this is a happy number or false
     */
    public boolean isHappy(int n) {
        // Write your code here
      
        Set<Integer> inLoop = new HashSet<Integer>();
    int squareSum,remain;
  while (inLoop.add(n)) {
    squareSum = 0;
    while (n > 0) {
        remain = n%10;
      squareSum += remain*remain;
      n /= 10;
    }
    if (squareSum == 1)
      return true;
    else
      n = squareSum;

  }
  return false;

    }
}
```

# Add Binary

Given two binary strings, return their sum (also a binary string).

For example,
a = "11"
b = "1"
Return "100".
```
package leetCode;
public class Temp
{
	public static void main(String [] args)
	{	   
		String  a="11";
		String b="1";
		System.out.print(addBinary(a,b));
	}
    public static   String addBinary(String a, String b) {
    	  if(a == null || a.isEmpty()) 
              return b;
          if(b == null || b.isEmpty()) 
              return a;
             
          StringBuilder stb = new StringBuilder();
          int i = a.length() - 1;
          int j = b.length() - 1;
          int aBit;
          int bBit;
          int carry = 0;
          int result;

          while(i >= 0 || j >= 0 || carry == 1) {
              aBit = (i >= 0) ? Character.getNumericValue(a.charAt(i--)) : 0;
              bBit = (j >= 0) ? Character.getNumericValue(b.charAt(j--)) : 0;
              result = aBit ^ bBit ^ carry;
              carry = ((aBit + bBit + carry) >= 2) ? 1 : 0;
              stb.append(result);
          }
          return stb.reverse().toString();                
    }
}
```

---



# 徒手开平方

·牛顿法-直线逼近原理
```
package leetCode;

public class Temp
{
  public static void main(String [] args)
  {    
    int  a=7;
    System.out.print(mySqrt(a));
  }
  
    public static   int mySqrt(int x) {
   long rs=x;
   while(rs*rs>x){
     rs=(rs+x/rs)/2;
   }
   return (int)rs;
    }
}

# 数学之美 —— 0x5f375a86
```
float InvSqrt(float x)
{
    float xhalf = 0.5f*x;
    int i = *(int*)&x; // get bits for floating VALUE 
    i = 0x5f375a86- (i>>1); // gives initial guess y0
    x = *(float*)&i; // convert bits BACK to float
    x = x*(1.5f-xhalf*x*x); // Newton step, repeating increases accuracy
    return x;
}  
```
# 两数字字符串相加

Given two non-negative integers num1 and num2 represented as string, return the sum of num1 and num2.

Note:

The length of both num1 and num2 is < 5100.
Both num1 and num2 contains only digits 0-9.
Both num1 and num2 does not contain any leading zero.
You must not use any built-in BigInteger library or convert the inputs to integer directly.
```
package leetCode;

public class Temp
{
	public static void main(String [] args)
	{	   
		String a="1001";
		String b="1341";
		System.out.print(addStrings(a,b));
	}
	
	public static  String addStrings(String num1, String num2) {
		StringBuilder sb=new StringBuilder();
		int carry=0;
	    for(int i=num1.length()-1,j=num2.length()-1;i>=0||j>=0||carry==1;i--,j--){
	    	int x=i<0?0:num1.charAt(i)-'0';
	    	int y=j<0?0:num2.charAt(j)-'0';
	    	sb.append((x+y+carry)%10);
	    	carry=(x+y+carry)/10;
	    }
	    return sb.reverse().toString();
	}
}
```
# 字符串中出现多少字母

存一下有多少个非0次出现的
```
int nonZero = 0;
	for (int i = 0; i < lenb; ++i) 
		if (++num[b[i] – ‘a’] == 1) ++nonZero;
```
``
12.单词翻转
``
```
翻转句子中全部的单词，单词内容不变
例如I’m a student.  变为student. a I’m
 in-place翻转 字符串第i位到第j位
while (i < j) swap(s[i++], s[j--]);
有什么用？
翻转整个句子 ： .tneduts a m’I
每个单词单独翻转： student. a I’m
难点？ 如何区分单词？找空格，split
```
# HashSet

```
public static int[] intersection(int[] nums1, int [] nums2){
        Set<Integer> set= new HashSet<>();
        Set<Integer> interset=new HashSet<>();
        for(int i=0;i<nums1.length;i++){
        	set.add(nums1[i]);        //存入不重复的值   
        }
        for(int i=0;i<nums2.length;i++){
        	if(set.contains(nums2[i])){  //如果重复上个set数组里的值
        		interset.add(nums2[i]);   //添加上个数组里有的值
        	}
        }
        
        int [] res=new int [interset.size()];
        int i=0;
        for(Integer num:interset){
        	res[i++]=num;
        }
        return res;
    }
}
```
# 350. Intersection of Two Arrays II

Given two arrays, write a function to compute their intersection.

Example:
Given nums1 = [1, 2, 2, 1], nums2 = [2, 2], return [2, 2].

Note:
Each element in the result should appear as many times as it shows in both arrays.
The result can be in any order.

先排序，一个一个对比，相等的才存，不等就小的数组指针加1。

```
public static int[] intersection(int[] nums1, int [] nums2){
        Arrays.sort(nums1);
        Arrays.sort(nums2);
        int p1=0;
        int p2=0;
        ArrayList<Integer> myList= new ArrayList <Integer>();
        while((p1<nums1.length)&&(p2<nums2.length)){
          if(nums1[p1]<nums2[p2]){
            p1++;
          }else{
            if(nums1[p1]>nums2[p2]){
              p2++;
            }else{
              myList.add(nums1[p1]);
              p1++;
              p2++;
            }
          }
        }
           
        int[] res= new int [myList.size()];
        for(int i=0;i<res.length;i++){
          res[i]=myList.get(i);    //ArrayList取值方法   
        }
        return res;   //转化成正常数组输出
  }
```
# 将字符数组转化为字符串

```
public static  String reverseString(String s) {
        char [] rs= new char[s.length()];     
        for(int i=0;i<s.length();i++){
        	rs[i]=s.charAt(i);
        }
        for(int i=0;i<s.length()/2;i++){
        	char temp=rs[i];
        	 rs[i]=rs[s.length()-i-1];
        	rs[s.length()-i-1]=temp;
        }
        return new String(rs);  //转化
    }
```

# 不用加号实现两数相加
```
public int getSum(int a, int b) {
    return b == 0 ? a : getSum(a ^ b, (a & b) << 1);
}



---



# 合并排序数组

合并两个排序的整数数组A和B变成一个新的数组。

给出A=**[1,2,3,4]**，B=**[2,4,5,6]**，返回** [1,2,2,3,4,4,5,6]**

```java
public class Solution {
    /*
     * @param A: sorted integer array A
     * @param B: sorted integer array B
     * @return: A new sorted integer array
     */
    public int[] mergeSortedArray(int[] A, int[] B) {
        // write your code here
       int indexA=0;
        int indexB=0;
        int []C =new int [A.length+B.length];
        int indexC=0;
        while(indexA!=A.length && indexB!=B.length) {
            if (A[indexA] < B[indexB]) {
                C[indexC++]=A[indexA++];
            }else if (A[indexA] > B[indexB]) {
                C[indexC++]=B[indexB++];
        }else {
                C[indexC++]=A[indexA++];
                C[indexC++]=B[indexB++];
            }
        }
        
        if(indexA==A.length && indexB!=B.length){
            for(int i=indexB;i<B.length;i++){
                C[indexC++]=B[i];
            }
        }

        if(indexA!=A.length && indexB==B.length){
            for(int i=indexA;i<A.length;i++){
                C[indexC++]=A[i];
            }
        }
        return C;
    }
}
```



---

1. 写代码：多线程实现从A,B,C三个文件中读取文件放到D文件中，优化：如何同步实现。

---

# 写一个单例模式，什么时候用到

在某些情况下，有些对象只需要一个就可以了，即每个类只需要一个实例。
单例模式的作用就是保证在整个应用程序的生命周期中，任何一个时刻，单例类的实例都只存在一个。
实现一个单例模式：

```java
public class Test{
      private Test(){}
      private static Test uniqueInstance = new Test();
      public static Test getInstence(){
          return uniqueInstance;
      }
}
```

由于在使用前就创建好了对象，因此，可以在多线程环境下使用这种方法。

---

# 装饰者模式是什么，举例

装饰者模式通过组合的方式扩展对象的特性，这种方式允许我们在任何时候对对象的功能
进行扩展，甚至是运行时扩展，而若我们用继承来完成对类的扩展，则只能在编译阶段实现
，所以在某些时候装饰者模式比继承更灵活。

特征：
* 装饰者和被装饰者的对象有着共同的超类
* 我们可以用多个装饰者修饰一个对象
* 可以用修饰过的对象替代代码中的原对象
* 一个对象在任何时候都可以被装饰，甚至是运行时

应用场景：
一家奶茶店，一杯奶茶是10元；
如果加糖+2、加奶+3、加葡萄+5元...
如果写类去继承，会要写n多个
如果用装饰就直接在类里加价钱

```java
return mTea.getPrice()+2;
return mTea.getPrice()+15;
return mTea.getPrice()+20;

-----

mTea = new SimpleTea();
mTea = new SugarDecorator(mTea);
mTea = new MilkDecorator(mTea);

int price1 = mTea.getPrice();
System.out.println("price1="+price1);
```





---



# 写代码：找出字符数组中第一次只出现三次的字符

```java
package Array;

/**
 * Created by zdmein on 2017/9/6.
 * 找出字符数组中第一次只出现三次的字符
 思路：
 1、统计各字符出现次数
 2、遍历字符串，如果出现字符次数为3则输出该字符并跳出循环；否则如果不存在则输出" . "
 */
public class ThreeChar1 {

    public static void main(String [] args){
        String str="abesabjdwsjibuhfrsadfesdsferewh";
        char [] chars=str.toCharArray();
        ThreeChar(chars);
    }

    public  static void ThreeChar(char [] chars){
        if(chars==null||chars.length==0){
            System.out.println(".");
        }

        int [] res=new int[256];
        for(int i=0;i<chars.length;i++){
            res[chars[i]]++;
        }

        for(int i=0;i<chars.length;i++){
            if(res[chars[i]]==3){
                System.out.println(chars[i]);
                break;
            }
        }
    }
}
```



---



# 验证身份证是否正确 

```
身份证号码验证
 
 1、号码的结构    
  公民身份号码是特征组合码，由十七位数字本体码和一位校验码组成。排列顺序从左至右依次为：六位数字地址码，    
  八位数字出生日期码，三位数字顺序码和一位数字校验码。    
 
 2、地址码(前六位数）     
  表示编码对象常住户口所在县(市、旗、区)的行政区划代码，按GB/T2260的规定执行。     
 
 3、出生日期码（第七位至十四位）    
  表示编码对象出生的年、月、日，按GB/T7408的规定执行，年、月、日代码之间不用分隔符。     
 
 4、顺序码（第十五位至十七位）    
  表示在同一地址码所标识的区域范围内，对同年、同月、同日出生的人编定的顺序号，    
  顺序码的奇数分配给男性，偶数分配给女性。     
 
 5、校验码（第十八位数）    
  （1）十七位数字本体码加权求和公式 S = Sum(Ai * Wi), i = 0,  , 16 ，先对前17位数字的权求和    
  Ai:表示第i位置上的身份证号码数字值 Wi:表示第i位置上的加权因子 Wi: 7 9 10 5 8 4 2 1 6 3 7 9 10 5 8 4    
  2 （2）计算模 Y = mod(S, 11) （3）通过模得到对应的校验码 Y: 0 1 2 3 4 5 6 7 8 9 10 校验码: 1 0    
  X 9 8 7 6 5 4 3 2    
  功能：身份证的有效验证 
```

---

#  网易笔试：一个合法的括号匹配

   给一个合法的括号匹配序列s，找到同样长的所有的括号匹配序列t，
   1.t和s不同，但是长度相同
   2.t也合法
  3.LCS（s,t） 是满足上面两个条件的t中最大的。

 eg:
 s="(())()"
 匹配的有： "()(())"、"((()))"、"()()()"、"(()())"
 其中 LCS（"(())()","()(())"）为4，其余都是5，所以输出3

```java
package ProgrammingWritten;

import java.util.ArrayList;
import java.util.HashMap;

import java.util.Map.Entry;

import java.util.Scanner;

/**
 * Created by zdmein on 2017/9/10.
 */
public class NetEase1 {

        public static int cnt;
        public static HashMap<String,Integer> map = new HashMap<>();
        public static void main(String[] args) {

            HashMap<Integer,Integer> map2 = new HashMap<>();
            Scanner input = new Scanner(System.in);
            String str1 = input.nextLine();
            // int n = Integer.parseInt(str);

            int n2 = str1.length();
            int n = (int) n2 / 2;
            // StringBuffer bf = new StringBuffer("");
            // String str1 = bf.append(str).reverse().toString();
            // n += Integer.parseInt(str1);
            // System.out.println(n);

            ArrayList<String> list = getBracketsOfN(n);
            for (String str2 : list) {
                if (str1.equals(str2)) {
                    continue;
                }else {
                    int[][] re = longestCommonSubsequence(str1, str2);

                    cnt=0;
                    print(re, str1, str2, str1.length(), str2.length());
                    map.put(str2, cnt);
                }

            }
            for (Entry<String, Integer> entry : map.entrySet()) {

                if (map2.containsKey(entry.getValue())) {
                    int count=map2.get(entry.getValue());
                    map2.put(entry.getValue(), ++count);
                }
                else {
                    map2.put(entry.getValue(), 1);
                }


            }

            int result = 0;
            int max= 0;
            for(Entry<Integer, Integer> entry : map2.entrySet()) {

                if (max<entry.getKey()) {
                    result = entry.getValue();
                }
            }
            System.out.println(result);

        }

        /**
         * Given the number N, return all of the correct brackets.
         *
         * @param n
         * @return
         */
        @SuppressWarnings("unchecked")
        public static ArrayList<String> getBracketsOfN(int n) {
            @SuppressWarnings("rawtypes")
            ArrayList[] dp = new ArrayList[n + 1];
            for (int i = 0; i < dp.length; i++)
                dp[i] = new ArrayList<String>();
            dp[0].add("");
            dp[1].add("()");
            if (n == 0)
                return dp[0];
            if (n == 1)
                return dp[1];
            int count = 2;
            while (count < n + 1) {
                ArrayList<String> lcount = dp[count];
                for (int i = 0; i < count; i++) {
                    ArrayList<String> l1 = dp[i];
                    ArrayList<String> l2 = dp[count - i - 1];
                    for (int j = 0; j < l1.size(); j++) {
                        for (int k = 0; k < l2.size(); k++) {
                            StringBuffer sb = new StringBuffer();
                            sb.append(l1.get(j));
                            sb.append("(");
                            sb.append(l2.get(k));
                            sb.append(")");
                            lcount.add(sb.toString());
                        }
                    }
                }
                dp[count++] = lcount;
            }
            return dp[n];
        }



        // 假如返回两个字符串的最长公共子序列的长度
        public static int[][] longestCommonSubsequence(String str1, String str2) {
            int[][] matrix = new int[str1.length() + 1][str2.length() + 1];//建立二维矩阵
            // 初始化边界条件
            for (int i = 0; i <= str1.length(); i++) {
                matrix[i][0] = 0;//每行第一列置零
            }
            for (int j = 0; j <= str2.length(); j++) {
                matrix[0][j] = 0;//每列第一行置零
            }
            // 填充矩阵
            for (int i = 1; i <= str1.length(); i++) {
                for (int j = 1; j <= str2.length(); j++) {
                    if (str1.charAt(i - 1) == str2.charAt(j - 1)) {
                        matrix[i][j] = matrix[i - 1][j - 1] + 1;
                    } else {
                        matrix[i][j] = (matrix[i - 1][j] >= matrix[i][j - 1] ? matrix[i - 1][j]
                                : matrix[i][j - 1]);
                    }
                }
            }
            return matrix;
        }
        //根据矩阵输出LCS
        public static void print(int[][] opt, String X, String Y, int i, int j) {
            if (i == 0 || j == 0) {
                return;
            }
            if (X.charAt(i - 1) == Y.charAt(j - 1)) {
                print(opt, X, Y, i - 1, j - 1);
                cnt++;
//                System.out.print(X.charAt(i - 1));
            } else if (opt[i - 1][j] >= opt[i][j]) {
                print(opt, X, Y, i - 1, j);
            } else {
                print(opt, X, Y, i, j - 1);
            }
        }
    }


```



---

# 滴滴笔试：求解第n个丑数

我们把只包含因子2,3和5的数称为丑数（Ugly Number),求从小到大的顺序第n的丑数，例如6,8都是丑数，但14不是，因为它包含因子7。习惯上我们把1当作第1个丑数

```java
package ProgrammingWritten;

import java.util.Scanner;

/**
 * Created by zdmein on 2017/9/10.
 */
public class didi2 {
    public static void main(String args[])
    {
        Scanner cin = new Scanner(System.in);

         int    n = cin.nextInt();

            System.out.println(getugly(n));

    }
        public  static int getugly(int n) {
            if(n==0)
                return 0;
            int[] a = new int[n];
            int count = 1;
            a[0] = 1;
            int num2 = 0;
            int num3 = 0;
            int num5 = 0;
            while(count<n){
                int min = min(a[num2]*2,a[num3]*3,a[num5]*5);
                a[count] = min;
                while(a[num2]*2<=a[count]) num2++;
                while(a[num3]*3<=a[count]) num3++;
                while(a[num5]*5<=a[count]) num5++;
                count++;
            }
            int result = a[n-1];
            return result;
        }

        public static int min(int a,int b,int c){
            int tmp = a>b?b:a;
            return tmp>c?c:tmp;
        }
}
```



---











![](http://cdn01.wallconvert.com/_media/wp_200x125/1/4/30546.jpg)