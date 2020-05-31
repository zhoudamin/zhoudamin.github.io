---
title: 剑指Offer(1-10)
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-06-28 16:33:26
categories: 算法与数学
tags: 算法与数学
---


剑指Offer题目解析！

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1498648976510&di=bffe34c0fbc10ab7a8a562901fd66243&imgtype=0&src=http%3A%2F%2Fimg2.a0bi.com%2Fupload%2Fttq%2F20140711%2F1405077864900.gif)

<!--more-->
# 题1：赋值运算符函数

 
# 题2：实现单例模式

 
# 题3：二维数组中的查找
题目：在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。
```
public class Main {
    public static void main(String[] args) {
      int arr[][] ={{1,3,5,7,9},
            {2,4,6,8,10},
            {3,5,7,9,12},
            {5,7,9,10,15}};
      int key=5;
      System.out.print("Key has in arr :" + find(arr,key));
    }

    public static boolean find(int [][] array,int key){
        if(array==null) return false;
        int clo=array[0].length-1;
        int row=0;
        while (clo>=0&&row<array.length){      
                if(array[row][clo]>key){
                    clo--;
                }else if (array[row][clo]<key){
                    row++;
                }else {
                    return true;
                }
            }
            return false;
        }
    }
```

 
# 题4：替换空格
`方法1：时间复杂度为 O(n) 的解法，思路是先数空格，再倒序插值!`
```
public class Main {
    public static void main(String[] args) {
        String str= "we are happy.";
        String res=ResplaceBlank(str);
        System.out.println(res);
    }

    public static String ResplaceBlank(String str){
        if(str==null) return str;
        int len=str.length();
        int k=0;
        for(int i=0;i<len;i++){
            if(str.charAt(i)==' ')
            k++;
        }
        char [] strnew = new char[len+k*3];
        for(int i=len+k*2-1,j=len-1;i>=0;i--,j--){
            if(str.charAt(j)!=' '){
                strnew[i]=str.charAt(j);
            }else{
                strnew[i--]='0';
                strnew[i--]='2';
                strnew[i]='%';
            }
        }
        return new String(strnew);
    }
}
```
`方法2：新数组，直接append`
```
public class Main {
    public static void main(String[] args) {

            String str= "we are happy.";
            String res=ResplaceBlank(str);
            System.out.println(res);
        }

    public static String ResplaceBlank(String str){
        if(str==null) return str;
        StringBuilder outputbuffer=new StringBuilder();
        for(int i=0;i<str.length();i++){
            if(str.charAt(i)!=' '){
                outputbuffer.append(str.charAt(i));

            }else {
                outputbuffer.append('%');
                outputbuffer.append('2');
                outputbuffer.append('0');
            }
        }
        return new String(outputbuffer);
    }
}
```

 
# 题5：从尾到头打印链表
`学会Stack的使用，Stack是一种先进后出的数据结构，pop出和push存`
```
import java.util.Stack;
public class Main {
    public static void main(String[] args) {
       ListNode node1=new ListNode(1);
       ListNode node2=new ListNode(2);
       ListNode node3=new ListNode(3);
       node1.next=node2;
       node2.next=node3;
       printReverseList(node1);
    }

    public static class ListNode{
        int val;
        ListNode next;
        ListNode(int val){
            this.val=val;
            this.next=null;
        }
    }

    public static void printReverseList(ListNode root){
        if(root==null ) return ;
        ListNode node=root;
        Stack<ListNode> stack = new Stack<ListNode>();
        while(node!=null){
            stack.push(node);
            node=node.next;
        }
        while(!stack.isEmpty()){
            int val=stack.pop().val;
            System.out.print(val+" ");
        }
    }
}
```
递归实现方法：
```
    public class Main {
        public static void main(String[] args) {
            ListNode node1=new ListNode(1);
            ListNode node2=new ListNode(2);
            ListNode node3=new ListNode(3);
            node1.next=node2;
            node2.next=node3;
            printReverseList(node1);
        }

        public static class ListNode{
            int val;
            ListNode next;
            ListNode(int val){
                this.val=val;
                this.next=null;
            }
        }

        public static void printReverseList(ListNode root){
          if(root!=null){
              if(root.next!=null){
                  printReverseList(root.next);
              }
          }
          System.out.println(root.val);
        }
    }
```

 
# 题6：重建二叉树
题目：输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如：前序遍历序列｛ 1, 2, 4, 7, 3, 5, 6, 8｝和中序遍历序列｛4, 7, 2, 1, 5, 3, 8,6}，重建出下图所示的二叉树并输出它的头结点。
![](http://upload-images.jianshu.io/upload_images/1361867-8773c0a120d3cb88.png?imageMogr2/auto-orient/strip " 根据前序和后序重建的二叉树")
`函数使用：Arrays.copyOfRange(preSort, 1, i + 1)`
```
import java.util.Arrays;

class BinaryTreeNode{
    public  int value=0;
    public BinaryTreeNode leftNode;
    public BinaryTreeNode rightNode;
}

public class Main {
    public static void main(String[] args) throws Exception{
        int[] preSort = {1, 2, 4, 7, 3, 5, 6, 8};
        int[] inSort = {4, 7, 2, 1, 5, 3, 8, 6};
        BinaryTreeNode root = constructCore(preSort, inSort);
        System.out.print(root.value+" "+root.leftNode.value+" "+root.rightNode.value);
    }

    public static BinaryTreeNode constructCore(int[] preSort, int[] inSort) throws Exception {
        if(preSort==null||inSort==null){
          return null;
        }
        if(preSort.length!=inSort.length){
              throw new  Exception("长度不一样，非法的输入！");
        }
        BinaryTreeNode root = new BinaryTreeNode();
        for (int i = 0; i < preSort.length; i++) {
            if (preSort[0] == inSort[i]) {
                root.value = preSort[0];
                root.leftNode = constructCore(Arrays.copyOfRange(preSort, 1, i + 1), Arrays.copyOfRange(inSort, 0, i));
                root.rightNode = constructCore(Arrays.copyOfRange(preSort, i + 1, preSort.length), Arrays.copyOfRange(inSort, i + 1, inSort.length));
            }
        }
        return root;
    }
}
```
 
# 题7：两个栈实现队列
题目：用两个栈实现一个队列。队列的生命如下，请实现它的两个函数appendTail和deleteHead，分别完成在队列尾部插入结点和在队列头部删除结点的功能。
```
import java.util.Stack;

public class Main<T> {
    private Stack<T> stack1=new Stack<T>();
    private Stack<T> stack2=new Stack<T>();
    public void appendTail(T t){
        stack1.push(t);
    }

    public T deleteHead() throws Exception{
        if(stack2.isEmpty()){
            if(!stack1.isEmpty()){
                stack2.push(stack1.pop());
            }
        }
        if(stack2.isEmpty()){
            throw new Exception("队列为空，不能删除！");
        }
        return stack2.pop();
    }

    public static void main(String[] args) throws Exception{
    Main<String> p7=new Main<>();
    p7.appendTail("1");
    p7.appendTail("2");
    p7.appendTail("3");
    System.out.println(p7.stack1);  //[1, 2, 3]
    p7.deleteHead();     
    System.out.println(p7.stack1);   //   [1, 2]
    System.out.println(p7.deleteHead()); // 2
    System.out.println(p7.stack1); //  [1]
    }
}
```
 
# 题8：旋转数组的最小数字
题目： 把一个数组最开始的若干个元素搬到数组的末尾，我们称之数组的旋转。输入一个递增排序的数组的一个旋转， 输出旋转数组的最小元素。例如数组{3 ,4,5, 1, 2 ｝为｛ 1,2,3, 4,5}的一个旋转，该数组的最小值为 1。
```
public class Main {
    public static void main(String[] args) {
  //  int [] array={3 ,4,5, 1, 2 };
     int [] array={1 ,1,1, 2, 0 };
     //   int [] array={1 ,1,1, 0, 1 };
    System.out.println(findMin(array));
    }

    public static Integer findMin(int [] arr){
        if(arr==null)
            return null;
        int start=0;
        int end=arr.length-1;

        while (start<end){
            int mid=(start+end)/2;
            if(arr[mid]>arr[end]){
                start=mid+1;
            }else if(arr[mid]<arr[end]){
                end=mid;
            }else{
                start++;
                end--;
            }
        }
        return arr[start];
    }
}
```
 
# 题9：斐波那契数列
写一个函数，输入n，求斐波那契（Fibonacci) 数列的第n项
```
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner sc=new Scanner(System.in);
        int n=sc.nextInt();
    System.out.println(Fibonacci(n));
    }

    public static long Fibonacci(int n){
    long res=0;
    long numb1=0;
    long numb2=1;
    if(n==0){
        return numb1;
    }
    if(n==1){
        return numb2;
    }
    while(n!=1){
        res=numb1+numb2;
        numb1=numb2;
        numb2=res;
        n--;
    }
    return res;
    }
}
```

 
# 题10：二进制中1的个数
请实现一个函数， 输入一个整数，输出该数二进制表示中1的个数。
例如把9表示成二进制是1001 ，有2位是1. 因此如果输入9，该出2。
```
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner sc=new Scanner(System.in);
        int n=sc.nextInt();
    System.out.println(numberOf1(n));
    }

    public static int numberOf1(int n){
        int count=0;
        while (n!=0){
            n=n&(n-1);
            count++;
        }
        return count;
    }
}
``` 

![](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1498648976508&di=0a707e43f7854a1b70038ba84df0be8e&imgtype=0&src=http%3A%2F%2Fs2.sinaimg.cn%2Fmw690%2F005DdmLAzy6JP9GXZQJa1%26690)