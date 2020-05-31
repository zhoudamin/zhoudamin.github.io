---
title: Java知识点锦集
categories:
  - 编程语言
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2019-04-13 17:33:55
tags: Java
---

知识锦集
<!--more-->

## hashMap原理，java8做的改变
从结构实现来讲，HashMap是数组+链表+红黑树（JDK1.8增加了红黑树部分）实现的。HashMap最多只允许一条记录的键为null，允许多条记录的值为null。HashMap非线程安全。ConcurrentHashMap线程安全。解决碰撞：当出现冲突时，运用拉链法，将关键词为同义词的结点链接在一个单链表中，散列表长m，则定义一个由m个头指针组成的指针数组T，地址为i的结点插入以T(i)为头指针的单链表中。Java8中，冲突的元素超过限制（8），用红黑树替换链表。

## String 和 StringBuilder 的区别
1）可变与不可变：String不可变，每一次执行“+”都会新生成一个新对象，所以频繁改变字符串的情况中不用String，以节省内存。
2）是否多线程安全：StringBuilder并没有对方法进行加同步锁，所以是非线程安全的。StringBuffer和String均线程安全。



{% asset_img 111.jpg Vha %}
