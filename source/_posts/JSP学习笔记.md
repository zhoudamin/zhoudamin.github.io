---
title: JSP学习笔记
字数统计: WordCount
阅读时长预计: Min2Read
总字数统计: TotalCount
date: 2017-04-02 10:03:23
categories: Java
tags: 框架
      JSP   
---
JSP : Java Server Pages ,是一个简化的Servlet设计，实现了在Java中使用HTML标签。
![](http://cdn01.wallconvert.com/_media/wp_200x125/1/2/18703.jpg)

<!--more-->

JSP基础语法：
``
1.JSP指令
``
·page指令：通常位于Jsp页面的顶端，同一个页面可以有多个page指令。
·include指令：将一个外部文件嵌入到当前JSP文件中，同时解析这个页面中的JSP语句。
·taglib指令：使用标签库定义新的自定义标签，在JSP页面中启动定制行为。


``
2.JSP注释
``
·HTML的注释：
< !--heml注释-->  //客户端可见
·JSP的注释：
<%--html注释--%>  //客户端不可见
·JSP脚本注释：
//单行注释
/**/多行注释

``
3.JSP脚本
``
<%Java代码%>

``
4.JSP声明
``
在JSP页面中定义变量或者方法。
<%!JAVA代码%>
``
5.Java 表达式
``
在JSP页面中执行的表达式。
<% =表达式 %>  //注意:表达式不以分号结束

``
6.JSP页面生命周期
``
·用户发出请求index.jsp
·是否是第一次请求--->
·是-->JSP引擎把该JSP文件转换成为一个Servlet，生成字节码文件，并执行jspInit()-->再访问生成的字节码文件
·否-->直接访问生成的字节码文件
·解析执行，jspService()

当用户第一次请求一个jsp页面时，首先被执行的方法是构造方法。

``
JSP内置对象
``
```
1.内置对象简介
JSP内置对象是Web容器创建的一组对象，不使用new关键就可以使用的内置对象。

2.四种作用域范围

3.out

4.request

5.session

6.application

7.其他内置对象

缓冲区：Buffer
缓冲区就是内存的一块区域用来保存临时数据。

```
``
8.ContextConfig 的 init 方法将会主要完成以下工作：
``
·创建用于解析 xml 配置文件的 contextDigester 对象
·读取默认 context.xml 配置文件，如果存在解析它
·读取默认 Host 配置文件，如果存在解析它
·读取默认 Context 自身的配置文件，如果存在解析它
·设置 Context 的 DocBase

``
9.ContextConfig 的 init 方法完成后，Context 容器的会执行 startInternal 方法，这个方法启动逻辑比较复杂，主要包括如下几个部分：
``
·创建读取资源文件的对象
·创建 ClassLoader 对象
·设置应用的工作目录
·启动相关的辅助类如：logger、realm、resources 等
·修改启动状态，通知感兴趣的观察者（Web 应用的配置）
·子容器的初始化
·获取 ServletContext 并设置必要的参数
·初始化“load on startup”的 Servlet


10.与 Servlet 主动关联的是三个类，分别是 ServletConfig、ServletRequest 和 ServletResponse。这三个类都是通过容器传递给 Servlet 的，其中 ServletConfig 是在 Servlet 初始化时就传给 Servlet 了，而后两个是在请求达到时调用 Servlet 时传递过来的。


![](http://img.hb.aicdn.com/2de6ab7af328bce602e7e547225a0866233860b321955-rcw4fB_fw658)