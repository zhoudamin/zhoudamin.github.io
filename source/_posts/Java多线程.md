---
title: Java多线程
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-04-03 10:17:35
categories: Java
tags: 多线程
photos:
 - http://bruce.u.qiniudn.com/2013/11/27/reading/photos-0.jpg
 - http://bruce.u.qiniudn.com/2013/11/27/reading/photos-1.jpg

---
![](http://cdn01.wallconvert.com/_media/wp_200x125/1/5/41445.jpg)

<!--more-->

# 基础

创建线程：
1.通过继承线程类Thread来创建线程类；
2.建立一个实现Runnable接口的类来运行线程。

通过继承Thread创建一个子类，在主控程序中同时运行两个线程。
```
class Thread1{
	public static void main(String args[]){
 		testThread t1 = new testThread("thread1");
 		testThread t2 = new testThread("thread2");
 		t1.start();
 		t2.start();
   }	
}

class testThread extends Thread{
	public testThread(String str){
	super(str);
    }

    public void run(){
    	for(int i=0;i<3;i++){
    		System.out.println(getName()+"在运行")；
    		try{
    			sleep(1000);
    		}catch(InterruptedException e){}
    	}
    	System.out.println(getName()+"已结束")；
    }
}
```

``
线程的生命周期
``
·创建（New Thread）
·运行（Runnable）
·挂起（Not Runnable）
·结束（Dead）

``
以下三种情况会使线程转入Not Runnable状态
``
1.调用sleep方法时，延时结束，重新转入Runnable状态；
2.调用wait方法等待一个特定状态发生时，状态发生后，其他对象必须调用notify或notifyAll方法向等待中的线程发出通知，才能唤醒这个线程；
3.当线程被I/O阻塞时，I/O完成后可唤醒线程。

***

# 售票系统之多线程

买火车票是大家春节回家最为关注的事情，我们就简单模拟一下火车票的售票系统：有500张从广州到长沙的火车票，在8个窗口同时出售，保证系统的稳定性和数据的原子性。

show my code .

```java
class Service{
    private String ticketName;
    private int totalCount;
    private int remaining;

    public Service(String ticketName , int totalCount){
        this.ticketName=ticketName;
        this.totalCount=totalCount;
        this.remaining=totalCount;
    }

    public synchronized int saleTicket(int ticketNum){
        if(remaining>0){
            remaining-=ticketNum;
            try{
                Thread.sleep(1000);
            }catch (InterruptedException e){
                e.printStackTrace();
            }

            if(remaining>=0){
                return remaining;
            }else{
                remaining+=ticketNum;
                return -1;
            }

        }
        return -1;
    }

    public synchronized int getRemaining(){
        return remaining;
    }

    public String getTicketName(){
        return  this.ticketName;
    }
}

class TicketSaler implements Runnable{
    private String name;
    private Service service;

    public TicketSaler(String windowName , Service service){
        this.name=windowName;
        this.service=service;
    }

    @Override
    public void run(){
        while(service.getRemaining()>0){
            synchronized (this){
                System.out.println(Thread.currentThread().getName()+"出售第"+service.getRemaining()+"张票，");
                int remaining =service.saleTicket(1);
                if(remaining>=0){
                    System.out.println("出票成功！剩余"+remaining+"张票。");
                }else {
                    System.out.println("出票失败！ 该票已售完 。");
                }
            }
        }
    }
}

public class MyThreadTests{
    public static void main(String [] args){
        //testThread();
        Service service=new Service("广州 --> 长沙",500);
        TicketSaler ticketSaler=new TicketSaler("售票系统" , service);
        Thread threads []=new Thread[8];
        for(int i=0;i<threads.length;i++){
            threads[i]=new Thread(ticketSaler,"窗口"+(i+1));
            System.out.println("窗口"+(i+1)+"开始出售"+service.getTicketName()+"的票...");
            threads[i].start();
        }
    }
}
```

部分结果：

```java
窗口1开始出售广州 --> 长沙的票...
窗口2开始出售广州 --> 长沙的票...
窗口3开始出售广州 --> 长沙的票...
窗口4开始出售广州 --> 长沙的票...
窗口5开始出售广州 --> 长沙的票...
窗口6开始出售广州 --> 长沙的票...
窗口7开始出售广州 --> 长沙的票...
窗口8开始出售广州 --> 长沙的票...
窗口1出售第500张票，
出票成功！剩余499张票。
窗口3出售第499张票，
出票成功！剩余498张票。
窗口3出售第498张票，
......
窗口7出售第0张票，
出票失败！ 该票已售完 。
窗口2出售第0张票，
出票失败！ 该票已售完 。
窗口1出售第0张票，
出票失败！ 该票已售完 。
```




![](http://cdn01.wallconvert.com/_media/wp_200x125/1/5/41938.jpg)