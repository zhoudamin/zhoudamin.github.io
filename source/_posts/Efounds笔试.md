---
title: Efounds笔试
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-06-24 11:38:56
categories: 算法与数学
tags: 
      - 笔试
      - Java
---

Efounds的笔试~

![](http://cdn01.wallconvert.com/_media/wp_400x250/1/5/41149.jpg)

<!--more-->
``
1.比较两个浮点数大小
``
```
一般不会直接用“==”或者“!=”对两个浮点数进行比较。
判断两个浮点数float a 与 float b 是否相等可以根据他们的差的绝对值是否大于0来进行判断。
判断相等：
if（Math.abs(a-b)<=0）{相等}
或者
if（!Math.abs(a-b)>0）{相等}

判断不等：
if（Math.abs(a-b)>0）{不相等}

```
``
2，用最快排序找到无序的数组里第K大的数
``
```
public class AlgCode {
    public static void main(String[] args) {
     // int [] nums={5, 7, 1, 8,3, 10};  //测试
        int [] nums=  {3,1,4,7,5,8,0};
        int k=5;
      int  res= findKth(nums,0,nums.length-1,k-1);
      System.out.print("The Kth is: "+res);
    }

    private static int findKth(int [] data ,int first , int last ,int k){
        int kth;
        if(first==last) kth=data[first];
        else {
            int pivot = data[first];
            int splitPoint = partition(data, first, last);
            if (k < splitPoint)
                kth = findKth(data, first, splitPoint - 1, k);
            else if(k>splitPoint)
                kth = findKth(data, splitPoint + 1, last, k);
            else
            kth = pivot;
           }
           return kth;
        }

        private static int partition(int [] data ,int first , int last){
            int pivot = data[first];
            while(first<last) {
                while (first < last && data[last] > pivot)
                    last--;
                data[first] = data[last];
                while (first < last && data[first] < pivot)
                    first++;
                data[last] = data[first];
            }
                data[first]=pivot;
                return first;
        }
    }
```
``
3，单链表是否有环并如何找到环入口
``
首先要了解什么叫环，如图，Join.next->Pos，Pos.next->Join，那么该链表有环
![](http://images.cnitblog.com/i/466768/201404/162209501501814.png)

```
public class LinkedLoop{
    //内部静态类定义结点类
	static class Node{
	int val;
	Node next;
	public Node(int val){
	this.val = val;
	}
}
	//判断单链表是否有环
	public static boolean hasLoop(Node head){
	Node p1=head;  //指向头节点
	Node p2=head.next;  //指向下一个节点

	while(p2!=null &&p2.next!=null){
		p1=p1.next;
		p2=p2.next.next;
		if(p2==null){
			return false;
		}
		int val1=p1.val;
		int val2=p2.val;
		if(val1==val2)
		return true;
	}
	return false;
}

	public static void main(String [] args){
		Node n1=new Node(1);
		Node n2=new Node(3);
		Node n3=new Node(6);
		Node n4=new Node(4);
		Node n5=new Node(5);
		Node n6=new Node(10);
		n1.next = n2;
		n2.next = n3;
		n3.next = n4;
		n4.next = n5;
		n5.next = n6;
		n6.next = n5;
		System.out.println(hasLoop(n1));
	}
}
```
``
4，快排第一次排列后的顺序
``
```
int [] nums=  {3,1,4,7,5,8,0};      
如果基准为3；
第一次快排后:
nums={0,1,3,7,5,8,4}; 
```

``
5，Java 数据输入
``
```
  int a =0;
  Scanner input = new Scanner(System.in);
  System.out.println("输入数字:");
  a = input.nextInt();
```

``
6，软件生命周期
``

软件生命周期(SDLC，Systems Development Life Cycle,SDLC)是软件的产生直到报废或停止使用的生命周期。
软件生存周期包括：

一，问题定义。要求系统分析员与用户进行交流，弄清“用户需要计算机解决什么问题”然后提出关于“系统目标与范围的说明”，提交用户审查和确认。

二，可行性研究。一方面在于把待开发的系统的目标以明确的语言描述出来，另一方面从经济、技术、法律等多方面进行可行性分析。

三，需求分析。弄清用户对软件系统的全部需求，编写需求规格说明书和初步的用户手册，提交评审。

四，开发阶段。开发阶段由三个阶段组成：
1，设计
2，实现：根据选定的程序设计语言完成源程序的编码。
3，测试

五，维护：维护包括四个方面
1，改正性维护：在软件交付使用后，由于开发测试时的不彻底、不完全、必然会有一部分隐藏的错误被带到运行阶段，这些隐藏的错误在某些特定的使用环境下就会暴露。
2，适应性维护：是为适应环境的变化而修改软件的活动。
3，完善性维护 ：是根据用户在使用过程中提出的一些建设性意见而进行的维护活动。
4，预防性维护：是为了进一步改善软件系统的可维护性和可靠性，并为以后的改进奠定基础。


``
7,多线程和多进程的区别
``
a.进程是资源分配的基本单位，线程是cpu调度，或者说是程序执行的最小单位。在Mac、Windows NT等采用微内核结构的操作系统中，进程的功能发生了变化：它只是资源分配的单位，而不再是调度运行的单位。在微内核系统中，真正调度运行的基本单位是线程。因此，实现并发功能的单位是线程。

b.进程有独立的地址空间，比如在linux下面启动一个新的进程，系统必须分配给它独立的地址空间，建立众多的数据表来维护它的代码段、堆栈段和数据段，这是一种非常昂贵的多任务工作方式。而运行一个进程中的线程，它们之间共享大部分数据，使用相同的地址空间，因此启动一个线程，切换一个线程远比进程操作要快，花费也要小得多。当然，线程是拥有自己的局部变量和堆栈（注意不是堆）的，比如在windows中用_begin threadex创建一个新进程就会在调用CreateThread的同时申请一个专属于线程的数据块（_tiddata)。

c.线程之间的通信比较方便。统一进程下的线程共享数据（比如全局变量，静态变量），通过这些数据来通信不仅快捷而且方便，当然如何处理好这些访问的同步与互斥正是编写多线程程序的难点。而进程之间的通信只能通过进程通信的方式进行。

d.由b，可以轻易地得到结论：多进程比多线程程序要健壮。一个线程死掉整个进程就死掉了，但是在保护模式下，一个进程死掉对另一个进程没有直接影响。

e.线程的执行与进程是有区别的。每个独立的线程有有自己的一个程序入口，顺序执行序列和程序的出口，但是线程不能独立执行，必须依附与程序之中，由应用程序提供多个线程的并发控制。

``
8,多线程同步和互斥有何异同，在什么情况下分别使用他们？举例说明
``
 所谓同步，表示有先有后，比较正式的解释是“线程同步是指线程之间所具有的一种制约关系，一个线程的执行依赖另一个线程的消息，当它没有得到另一个线程的消息时应等待，直到消息到达时才被唤醒。”

 所谓互斥，比较正式的说明是“线程互斥是指对于共享的进程系统资源，在各单个线程访问时的排它性。当有若干个线程都要使用某一共享资源时，任何时刻最多只允许一个线程去使用，其它要使用该资源的线程必须等待，直到占用资源者释放该资源。线程互斥可以看成是一种特殊的线程同步。”表示不能同时访问，也是个顺序问题，所以互斥是一种特殊的同步操作。

举个例子，设有一个全局变量global，为了保证线程安全，我们规定只有当主线程修改了global之后下一个子线程才能访问global，这就需要同步主线程与子线程，可用关键段实现。当一个子线程访问global的时候另一个线程不能访问global，那么就需要互斥。

``
9,网络协议：访问某个网址都经过了哪些协议
``
·域名解析协议DNS
应用层协议，网址相当于是域名，访问DNS服务器，这个过程有域名解析协议，解析出域名对应的IP地址。

·超文本传输协议HTTP
应用层协议，基于请求和响应的协议，通过请求行、消息报头、请求正文向目的地址发送请求。目的服务器在接受请求后，返回一个状态行、消息报头、响应正文的响应。

·传输控制协议TCP
传输层协议，HTTP协议是基于TCP协议的，也就是说HTTP无论是请求还是响应都是把HTTP的内容作为TCP的正文封装到TCP的报文中的。TCP协议是传输安全，面向连接的协议，在客户端和服务端建立TCP/IP五层模型的协议 连接的过程中需要经过三次握手，发送第一个SYN的一端将执行主动打开，接收这个SYN并发回下一个SYN的另一端执行被动打开，以及四次释放的过程才停止发送数据。

·网际协议IP协议
IP协议在整个传输过程中都起着重要的作用，网址通过DNS解析为IP地址，在TCP建立连接以及传输数据的整个过程中都在使用着IP协议。

![](http://cdn01.wallconvert.com/_media/wp_400x250/1/5/40750.png)