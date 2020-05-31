---
title: 读书笔记之《Java编程思想》
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-08-08 09:07:57
categories: 读书
tags: Java
---

![](https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=801804653,3756867112&fm=15&gp=0.jpg)
<!--more-->

# 17. 容器

## Set 

存入Set的每个元素都必须是唯一的，因为Set不保存重复元素。

Set接口不保证维护元素的次序

## Map

映射表（关联数组）的基本思想是维护的是键-值（对）关联，因此可以用键来查找值。

基本方法

```java
Map.put()
Map.get()
```



# 21. 并发

* 阻塞：

如果程序中的某个任务因为该程序控制范围之外的某些条件（通常是I/O）而导致不能继续执行，那么我们就说这个任务或线程阻塞了。

Java的线程机制是抢占式的。

一个线程就是在进程中的一个单一的顺序控制流，因此，单个进程可以拥有多个并发执行的任务。

***

LiftOff任务将显示发射之前的倒计时：

```java 
public class LiftOff implements Runnable{
	protected int countDown = 10;
	private static   int taskCount =0;
	private final int id=taskCount++;
	public LiftOff(){
		this.countDown=countDown;
	}

public String status(){
	return "#"+id +"("+(countDown>0 ? countDown:"LiftOff ！")+").";
}

public void run(){
	while(countDown-->0){
	System.out.println(status());
	Thread.yield();
		}
	}
}

/*************************************************************/

public class ThreadPro {
    public static void main(String [] args){
        for(int i=0;i<5;i++)
            new Thread(new LiftOff()).start();
        System.out.println("Waiting for LiftOff !");
    }
}
```

***

* 执行器（Executor）

Executor允许你管理异步任务的执行，而无需显式地管理线程的生命周期。

***

* 优先级

线程的优先级将该线程的重要性传递给了调度器。

尽管CPU处理现有线程集的顺序使不确定的，但是调度器将倾向于让优先权最高的线程先执行。

另外会让优先级较低的线程执行的频率较低，故而优先权不会导致死锁。





































![](https://ss0.bdstatic.com/70cFvHSh_Q1YnxGkpoWK1HF6hhy/it/u=1444271841,2904428413&fm=26&gp=0.jpg)