---
title: 读书笔记之《深入理解Java虚拟机》
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-08-08 10:35:12
categories: 开发者手册
tags: 开发者手册
---





![](https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=2692886884,1328684177&fm=15&gp=0.jpg)
<!--more-->



# 第13章 线程安全与锁优化

## 分类

### 1.  不可变

不可变的对象一定是线程安全的

不可变带来的安全性是最简单和最纯粹的

1. 基本数据类型：final
2. 对象：java.lang.String

### 2. 绝对线程安全

在Java API 中标注自己是线程安全的类，大多数都不是绝对的线程安全。

Java.util.Vector是一个线程安全的容器，因为它的add()、get()、size()这类方法都是被synchronized修饰的。 在多线程中，还需在调用端做额外的同步措施。

### 3. 相对线程安全

通常意义上讲的线程安全

在Java中，大部分的线程安全类都属于这种类型，如Vector、HashTable、Collection、的synchronizedCollection()方法包装的集合等。

### 4. 线程兼容

指对象本身并不是线程安全的，但是可以在同步端正确使用同步手段保证对象在并发环境中可以安全地使用。

Java API 中大部分的类都是属于线程兼容的，如Vector 和 HashTable相对应的集合类ArrayList 和 HashMap 等。

### 5. 线程对立

指无论调用端是否采取了同步措施，都无法在多线程环境中并发使用的代码。

一个线程对立的例子是Thread类的 suspend() 和 resume() 方法。

## 线程安全实现方法

### 互斥同步

Mutual Exclusion & Synchronization

同步是指在多个线程并发访问共享数据时，保证共享数据在同一个时刻只被一个（或者是一些，使用信息量的时候）线程使用。

互斥是实现同步的一种手段，临界区（Critical Section）、互斥量（Mutex）和信息量（Semaphore）都是主要的互斥实现方式。

互斥是因，同步是果。互斥是方法，同步是目的。

互斥同步最主要的问题是进行线程阻塞和唤醒所带来的性能问题，因此这种同步也称为阻塞同步（Blocking Synchronization）。

#### Synchronized

在Java中，最基本的互斥同步手段就是synchronized关键字

API 层面的互斥锁（lock() 和 unlock() 方法配合 try/finally 语句块来完成）

#### ReentrantLock

重人锁

原生语法层面的互斥锁。

1. 等待可中断：是指当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。
2. 公平锁：是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。
3. 锁绑定多个条件：是指一个ReentrantLock对象可以同时绑定多个Condition对象。只需要多次调用newCondition()方法即可。

### 非阻塞同步

乐观的并发策略

先进行操作，如果没有其他线程争用共享数据，那操作就成功了；如果共享数据有争用，产生了冲突，那就再采取其他的补偿措施。

* 测试并设置
* 获取并增加
* 交换
* 比较并交换
* 加载链接/条件存储














```
Java代码实现
```
<br/>





![](https://ss1.bdstatic.com/70cFvXSh_Q1YnxGkpoWK1HF6hhy/it/u=1597620862,3199942303&fm=26&gp=0.jpg)